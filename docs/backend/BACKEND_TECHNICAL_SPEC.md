# Backend technical specification

Status: implementation baseline
Owner: backend/platform
Last updated: 2026-09-23

## 1. Purpose

This document is the executable technical specification for the ChessScope backend. It converts the product and architecture context into testable backend requirements. It does not authorize implementation yet; implementation begins phase by phase after the relevant entry gate is accepted.

ChessScope is a professional, AI-native ChessBase-class research platform. The backend must treat chess data, deterministic statistics, engine analysis, evidence, and AI explanations as separate layers with explicit provenance.

## 2. Fixed technical decisions

| Area | Decision |
|---|---|
| Backend language | Python 3.13+ after dependency compatibility verification |
| Web framework | Django + Django REST Framework (DRF) |
| Frontend contract | TypeScript/React client generated or validated against OpenAPI |
| Primary database | PostgreSQL; PostgreSQL remains the system of record |
| Background work | Celery workers with durable routing; RabbitMQ is the production broker |
| Cache/coordination | Redis for cache, rate-limit state, short-lived locks, and progress fan-out |
| Binary/large artifacts | S3-compatible object storage; MinIO is acceptable locally |
| Deployment shape | Modular monolith plus independently scaled worker pools |
| API style | Versioned REST JSON under `/api/v1/`; asynchronous jobs for expensive work |
| Live progress | Server-Sent Events where supported, polling fallback always available |
| Observability | OpenTelemetry-compatible traces, structured logs, metrics, error tracking |
| Position storage | PostgreSQL first; specialized store only after recorded benchmark trigger |
| AI boundary | AI calls typed deterministic tools; narratives are not source-of-truth records |

The system starts as a modular monolith, not a distributed microservice fleet. Module boundaries, queues, database ownership, and APIs are designed so ingestion, position search, analytics, or engine execution can be separated later without replacing domain identities.

## 3. Capacity envelope and service objectives

The first production architecture is sized for several hundred simultaneous users without pretending to be internet-scale.

### 3.1 Planning envelope

These are capacity assumptions to verify in load tests, not marketing promises:

- 5,000 registered accounts and 300–500 concurrently active web sessions;
- 50 API requests/second sustained and 200 requests/second for short bursts;
- 20 simultaneous imports and 50 simultaneous analytical jobs;
- 16–64 concurrent Stockfish processes, isolated from API compute;
- 1–10 million canonical games in the first professional deployment;
- 100–800 million position occurrences, depending on corpus and indexing policy;
- evolution toward 100 million games through storage specialization, not a day-one schema rewrite.

### 3.2 SLOs

Measured at the API boundary, excluding client network latency:

| Workload | Target |
|---|---|
| Auth/account reads | P95 <= 250 ms |
| Indexed metadata search | P95 <= 500 ms |
| Exact-position summary, warm cache | P95 <= 700 ms |
| Exact-position summary, cold database | P95 <= 1.5 s |
| Game document load | P95 <= 800 ms |
| Create import/analysis job | P95 <= 400 ms |
| Job progress visibility | update visible within 2 s |
| Monthly API availability after beta | >= 99.5% |
| Successful ingestion correctness | no silent move/tag loss; all rejected records explained |

Any request predicted to exceed two seconds should normally become an asynchronous job or a paginated/streaming query.

## 4. Backend modules

Each module owns its services and public contract. Cross-module writes go through application services, not direct model imports from arbitrary code.

1. **identity** — users, authentication, sessions, OAuth identities, roles, API credentials.
2. **workspaces** — tenancy, membership, invitations, shares, quotas, private/public boundaries.
3. **sources** — source policies, import batches, immutable raw artifacts, connector checkpoints.
4. **players** — canonical people/accounts, aliases, identity candidates, ratings, rankings, media rights.
5. **chess** — PGN parsing contract, canonical games, revisions, variations, events, sites, teams.
6. **positions** — canonical position identity, occurrences, move edges, opening aggregates.
7. **search** — normalized query model, filters, cursor pagination, saved searches.
8. **analytics** — cohorts, metric registry, materialized results, player snapshots, uncertainty.
9. **engines** — engine builds, profiles, jobs, leases, results, cancellation, compute budgets.
10. **evidence** — immutable evidence bundles, citations, claims, reproducibility manifests.
11. **studies** — studies, chapters, annotations, diagrams, ordered collaborative artifacts.
12. **integrations** — provider adapters, credentials, rate limits, conditional requests, sync state.
13. **jobs** — durable application job state, progress, attempts, errors, idempotency.
14. **audit** — security events, access logs, administrative actions, data deletion trail.

## 5. API contract

### 5.1 Conventions

- Base path: `/api/v1/`.
- Media type: JSON for commands and metadata; PGN/NDJSON/ZIP for documented exports.
- Identifiers exposed to clients: opaque UUIDv7 values; internal high-volume rows may use `bigint`.
- Time: RFC 3339 UTC; date precision is represented explicitly when the source is incomplete.
- Lists: cursor pagination for games, occurrences, jobs, and ratings; bounded page pagination for administration.
- Filters: explicit typed fields; no raw SQL-like query language from clients.
- Errors: stable machine code, localized-safe message, trace ID, field details, retryability.
- Versioning: additive changes within `v1`; breaking semantic changes require a new version or a versioned resource representation.
- Documentation: generated OpenAPI 3.1 checked in CI for breaking changes.

### 5.2 Command safety

- `POST` commands that create imports, exports, engine runs, or paid external calls require an `Idempotency-Key`.
- Mutations of versioned content use optimistic concurrency (`If-Match` or explicit revision).
- Resource representations use `ETag` where useful; source connectors honor upstream conditional-request headers.
- Delete requests create auditable deletion jobs when derived data must also be removed.
- A successful asynchronous command returns `202 Accepted`, `job_id`, state, status URL, and estimated queue class; it never pretends work is already complete.

### 5.3 Endpoint families

```text
/api/v1/auth/*
/api/v1/me
/api/v1/workspaces/*
/api/v1/sources/*
/api/v1/imports/*
/api/v1/games/*
/api/v1/positions/*
/api/v1/search/*
/api/v1/players/*
/api/v1/ratings/*
/api/v1/analytics/*
/api/v1/engine-builds/*
/api/v1/analysis-jobs/*
/api/v1/evidence/*
/api/v1/studies/*
/api/v1/integrations/*
/api/v1/jobs/*
/api/v1/admin/*
```

Detailed serializers must expose provenance, freshness, coverage, and visibility fields wherever a result could otherwise look more authoritative than its source permits.

### 5.4 Authentication and authorization

- Browser authentication uses secure, `HttpOnly`, same-site cookies with CSRF protection.
- Personal API access uses short-lived scoped tokens; long-lived secrets are hashed and revocable.
- OAuth connectors use least-privilege scopes and encrypted refresh tokens.
- DRF permission classes enforce coarse endpoint access; domain policy services enforce resource/tenant access.
- Workspace roles start with `owner`, `admin`, `analyst`, `member`, and `viewer`.
- Object-level access is checked before serialization and before a job is queued.
- DRF throttling protects fair use and cost budgets but is not treated as DDoS protection.

## 6. Asynchronous job contract

Expensive work is modeled as a durable application job in PostgreSQL. Celery delivery is an execution mechanism, not the source of job truth.

Every job contains:

- public ID, type, schema version, tenant/requester, and priority;
- idempotency key and normalized input fingerprint;
- queued/started/heartbeat/finished timestamps;
- state: `queued`, `leased`, `running`, `cancelling`, `succeeded`, `partially_succeeded`, `failed`, `cancelled`;
- progress units, total when known, stage, retry count, and safe user-facing message;
- lease owner and expiry;
- cost budget and actual usage;
- result/error artifact references;
- trace ID and parent job ID.

Queue classes:

| Queue | Examples | Isolation rule |
|---|---|---|
| `ingestion` | fetch, parse, normalize | network and parser limits |
| `indexing` | position generation, rebuild | high I/O, separate concurrency |
| `analytics` | cohort metrics, snapshots | CPU/SQL budgeted |
| `engine` | Stockfish/UCI work | dedicated hosts/process quotas |
| `integrations` | FIDE/Lichess/Chess.com sync | provider rate-limit lanes |
| `exports` | PGN/ZIP/report generation | low priority, bounded storage |
| `maintenance` | cleanup, compaction, verification | off-peak policy |

Requirements:

- tasks are idempotent or explicitly non-retriable;
- late acknowledgement is used only for idempotent tasks;
- long tasks use low prefetch and periodic heartbeats;
- retries use bounded exponential backoff with jitter and classify permanent errors;
- cancellation is cooperative and checked between bounded work units;
- poison messages terminate in a dead-letter path with an operator-visible reason;
- task payloads contain identifiers, not large PGN files or secrets;
- workers re-authorize private resources at execution time.

## 7. Ingestion contract

The pipeline is staged and restartable:

```text
discover/fetch -> immutable raw artifact -> parse -> validate legality
-> normalize -> identity resolution -> deduplicate/cluster -> persist revision
-> generate positions -> update aggregates -> publish corpus snapshot
```

Each stage records code/parser version, input/output checksum, counts, rejected records, warnings, and timing. A later parser version creates new normalized output without overwriting the raw source.

Required behavior:

- streaming PGN parsing with bounded memory;
- preservation of comments, NAGs, clocks, variations, unknown tags, and starting FEN where policy permits;
- explicit variant and illegal-game rejection;
- safe decompression with archive size/member limits and path traversal protection;
- deduplication that clusters source records but never deletes provenance;
- deterministic canonicalization and position-key test vectors;
- per-source license/policy classification and visibility before data enters a corpus;
- deletion and correction propagation to derived indexes and evidence freshness.

## 8. Position search and analytics

- Exact position identity includes piece placement, side to move, castling rights, and legally relevant en-passant state.
- Hash identity is verified against canonical state before returning a match; collisions must be detectable.
- Position occurrence is distinct from position aggregate.
- Opening explorer aggregates are snapshot-bound and include game count, result distribution, next moves, rating/time-control/date coverage, and last refresh.
- Filters are normalized into a stable cohort identity.
- Every metric declares semantic version, input corpus snapshot, cohort, sample size, uncertainty method, and evidence references.
- User-private games do not enter global aggregates by default.
- Recomputations produce new result versions; historical evidence never silently points to a new answer.

## 9. Engine platform

- Engine binaries never run inside API processes.
- Each result binds engine name, binary checksum/version, NNUE/network checksum, options, threads, hash, tablebase version, hardware class, and deterministic budget.
- Nodes are preferred for comparable reproducible budgets; time-based analysis may be offered but is marked hardware-dependent.
- Workers enforce process, memory, CPU, wall-time, file, and network boundaries.
- Analysis is cached by canonical position, build, profile, option set, and privacy scope.
- MultiPV, score perspective, mate scores, WDL, PV legality, and terminal positions have canonical representations.
- Cloud analysis requires quotas and admission control; private/local routing remains a future compatible contract.

## 10. Player card backend

The player card is a versioned projection, not a single scraped record. It combines:

- canonical identity, FIDE ID, verified platform accounts, aliases, title, federation, and activity state;
- licensed/attributed portrait metadata, never an untracked copied image;
- current and historical standard/rapid/blitz ratings by rating system;
- current/peak ranking with effective date and source;
- recent form and event results where licensed data exists;
- opening preferences by color and time window;
- preparation diversity, style indicators, conversion/recovery, and trend metrics only when sample thresholds are met;
- source coverage, last refresh, identity confidence, and warnings.

FIDE, Lichess, and Chess.com rating pools remain separate. A synthetic cross-platform score, if ever introduced, is a named metric with methodology—not a field called “rating.” Sensitive or disputed biographical fields are omitted or corrected through provenance-aware workflows.

## 11. Caching and consistency

- PostgreSQL is authoritative for users, permissions, jobs, canonical identities, and evidence manifests.
- Redis entries are disposable and version-keyed; cache loss must affect latency, not correctness.
- Cache keys include tenant/visibility, corpus snapshot, metric version, normalized filters, and representation version.
- Writes use transactions plus an outbox record; workers update derived state after commit.
- Read-after-write is guaranteed for user edits from PostgreSQL; indexes and analytics expose `pending`/freshness when eventually consistent.
- Stampede protection uses short leases and stale-while-revalidate only for non-sensitive deterministic results.

## 12. Security, privacy, and compliance requirements

- TLS everywhere; encryption at rest for managed storage and object artifacts.
- Secrets live in a secret manager, never in source, task payloads, or logs.
- Raw uploads are private by default, malware/archive checked, size limited, and content-type verified.
- Tenant ID is mandatory in private-domain queries; selected workspace tables use PostgreSQL row-level security as defense in depth.
- Structured logs redact PGNs, tokens, email addresses, connector payloads, and AI prompts containing private study data.
- Audit records cover login/security events, role changes, shares, exports, connector access, and deletion.
- Account/workspace deletion is an asynchronous verified workflow across raw, canonical, cached, index, analytics, and export data.
- Source terms, licenses, attribution, permitted purposes, and retention are data records with enforcement hooks.
- Backoffice actions require elevated role, MFA policy, reason capture, and audit trail.

## 13. Observability and operations

Required service telemetry:

- request rate, latency, status, database time, cache hit rate, and saturation;
- queue depth/age, task duration, retries, failures, heartbeats, and dead letters;
- import throughput and rejection reasons;
- exact-position latency by corpus size and filter;
- engine node throughput, queue time, crash rate, and cost;
- connector rate-limit status, freshness, and schema drift;
- PostgreSQL locks, connections, replication lag, WAL, table/index growth, and slow queries;
- per-workspace cost and quota signals without exposing private content.

Every API request and background job carries a trace ID. Health endpoints distinguish process liveness from dependency readiness. Runbooks must exist before beta for database outage, queue backlog, corrupt import, bad migration, object-storage loss, leaked connector credential, and runaway engine cost.

## 14. Quality gates

No phase is complete without relevant automated checks:

- unit tests for rules and normalization;
- golden PGN and position-key fixtures;
- property tests for move/application round-trips;
- API contract and authorization matrix tests;
- migration forward/backward compatibility checks;
- integration tests with PostgreSQL, broker, Redis, and object storage;
- retry/idempotency/failure-injection tests for jobs;
- deterministic analytics fixtures with independently calculated expected values;
- engine protocol and resource-limit tests;
- performance tests at the phase corpus target;
- security tests for tenant escape, IDOR, archive bombs, SSRF, injection, and secret leakage.

## 15. Definition of backend done

A backend capability is done only when it has:

1. accepted API/domain contract and authorization policy;
2. reversible or expand/migrate/contract database migration plan;
3. idempotency, retry, cancellation, and failure behavior where asynchronous;
4. provenance, visibility, deletion, and audit behavior;
5. tests including negative and tenant-isolation cases;
6. metrics, alerts, dashboard, and runbook entries;
7. measured performance against its phase target;
8. OpenAPI and operator/developer documentation;
9. explicit out-of-scope items and known limitations.

## 16. Non-goals for the first backend release

- internet chess play or anti-cheat adjudication;
- social feed, chat, course marketplace, or puzzle platform;
- mass mirroring of sites without a documented API/license/contract;
- live assistance during rated games;
- model training on proprietary annotations;
- multi-region active-active writes;
- Kubernetes solely for appearance of scale;
- premature microservice extraction before measurements identify a boundary.

## 17. Open decisions before Phase 1 implementation

- final supported Python/Django/DRF versions after compatibility and security review;
- cloud provider and managed-service choices;
- identity provider versus first-party Django authentication;
- exact public corpus and its legal basis;
- RabbitMQ from first deploy versus a shorter Redis-broker prototype;
- initial engine compute business/quota policy;
- whether player accounts and real-world persons become separate physical tables from day one;
- accepted RPO/RTO and paid backup/replica budget.
