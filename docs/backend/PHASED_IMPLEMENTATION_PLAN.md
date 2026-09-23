# Backend phased implementation plan

Status: proposed delivery contract
Last updated: 2026-09-23

## 1. How to use this plan

This plan decomposes the backend technical specification into vertical, testable phases. Phase numbers here are backend delivery phases (`B0`–`B8`) and map onto the broader product roadmap; they do not replace product discovery gates.

Rules:

- phases are sequential at their critical foundations, but UI and research can proceed alongside them;
- no phase starts merely because a date arrived; its entry conditions must be true;
- every phase ships documentation, observability, security, and tests with the feature;
- migrations and APIs must remain compatible with the previous deployed version;
- source rights are a hard dependency, not a cleanup task;
- estimates are team planning ranges, not commitments; re-estimate after B0 benchmarks.

## 2. Phase map

| Phase | Outcome | Indicative duration | Product roadmap mapping |
|---|---|---:|---|
| B0 | risks, contracts, and benchmarks resolved | 3–5 weeks | Product Phase 0 |
| B1 | deployable platform foundation | 3–4 weeks | Product Phase 0/1 |
| B2 | trustworthy PGN and canonical chess data | 4–6 weeks | Product Phase 1 |
| B3 | position index, search, and opening explorer | 5–7 weeks | Product Phase 1 |
| B4 | player identity, ratings, and player cards | 4–6 weeks | Product Phase 1 |
| B5 | analytics, evidence, and reproducible metrics | 5–7 weeks | Product Phase 1/2 |
| B6 | isolated engine analysis platform | 4–6 weeks | Product Phase 1/2 |
| B7 | typed investigation tools and evidence-first AI | 5–7 weeks | Product Phase 2 |
| B8 | collaboration, scale hardening, and beta operations | 6–10 weeks | Product Phase 3 |

The smallest serious alpha is B0–B5. B6 can run partly in parallel after stable position identity exists. B7 must not precede deterministic evidence APIs.

## 3. B0 — foundation decisions and disposable benchmarks

### Goal

Remove decisions that would otherwise force expensive rewrites: legal corpus, parser fidelity, position identity, storage economics, engine isolation, and API/job contracts.

### Scope

- approve the backend, system, database, and integration specifications;
- choose supported Python/Django/DRF/PostgreSQL/Celery versions from current official documentation;
- produce ADRs for broker, authentication, position key, object storage, and tenancy;
- create golden PGN corpus: valid, malformed, Chess960/variant, nested variations, clocks, comments, NAGs, Unicode, incomplete dates, and custom FEN;
- prototype position generation and compare PostgreSQL partition/index designs at 100k and 1M games; extrapolate only with recorded assumptions;
- exercise 10M games if hardware and legal/open corpus permit;
- prove UCI process lifecycle, timeouts, cancellation, and result normalization;
- complete a source-policy decision for the initial public and user-authorized corpus;
- define OpenAPI conventions, error envelope, job schema, visibility classes, and trace propagation.

### Deliverables

- accepted ADR set and benchmark report;
- position-key test vectors shared across backend and future local companion;
- capacity model with storage and cost per one million games;
- initial threat model and data-flow diagram;
- source/license register with owners and review dates;
- phase B1 backlog with acceptance tests.

### Exit criteria

- no known silent loss across the golden PGN set;
- canonical positions reproduce expected keys on all test vectors;
- exact-position and filtered-query plans are measured, not guessed;
- initial corpus has an approved usage basis;
- engine process cannot escape its resource/network policy in the prototype;
- open decisions blocking schema creation have named owners and due dates.

### Explicit non-deliverable

No production UI or production data ingestion is required from B0. Benchmark code may be disposable.

## 4. B1 — platform skeleton and operational baseline

### Goal

Create a deployable, observable, secure backend shell capable of serving authenticated tenants and durable background jobs.

### Scope

- Django/DRF modular project boundaries;
- environment/config/secret handling;
- PostgreSQL migrations, PgBouncer-ready connection settings;
- user, workspace, membership, role, and quota-policy model;
- secure cookie authentication, CSRF, password/MFA integration decision;
- RabbitMQ, Celery routing, Redis cache, and S3-compatible storage;
- durable `job`, `outbox_event`, and idempotency implementation;
- health/readiness, structured logging, tracing, metrics, error reporting;
- OpenAPI publication and TypeScript client contract pipeline;
- CI: lint/type/test/migration/security/dependency checks;
- staging deployment and backup/restore rehearsal.

### API milestone

`auth`, `me`, `workspaces`, `members`, `jobs`, `health`, and administration foundations.

### Acceptance criteria

- cross-workspace authorization test matrix passes;
- duplicate idempotent command produces one logical job;
- broker outage leaves a recoverable outbox record;
- worker crash retries an idempotent fixture without double effect;
- trace connects HTTP request, outbox, task, and database operation;
- staging restore meets the provisional RPO/RTO exercise;
- 500 simulated active sessions do not exhaust database connections.

### Deferred

Chess data, public corpus, engines, and AI.

## 5. B2 — ingestion and canonical chess model

### Goal

Import user PGNs and approved source files into a loss-aware, provenance-complete canonical model.

### Scope

- data-source/policy registry, raw artifact upload, checksum, retention;
- safe archive validation and object-storage lifecycle;
- staged streaming PGN parser and legality validation;
- source game, canonical game, revision, event/site, initial player alias model;
- deterministic normalization and content fingerprints;
- deduplication clustering while retaining all source records;
- import progress, warnings, rejected record artifact, retry, cancel, delete;
- private-by-default workspace visibility;
- export round-trip test for supported PGN constructs.

### API milestone

`sources`, `imports`, `artifacts`, `games`, basic `players`, and import job events.

### Scale milestone

- 1 million game import rehearsal;
- bounded worker memory and configurable backpressure;
- no unbounded transaction or task payload.

### Acceptance criteria

- golden corpus has zero silent semantic loss;
- every canonical revision resolves to source/import/policy/checksum;
- repeated import is idempotent and does not create duplicate canonical games;
- deletion removes or invalidates private derived records and leaves an audit trail;
- malformed records do not abort unrelated valid games;
- import metrics identify throughput and top rejection causes.

## 6. B3 — position index, deterministic search, and explorer

### Goal

Deliver the first core ChessBase-class workflow: find a position, filter occurrences, inspect next moves, and open source games.

### Scope

- canonical FEN/state normalization and collision-verifiable position key;
- position generation from canonical revisions;
- partitioned occurrence storage and rebuild checkpoints;
- exact position lookup, next-move aggregate, representative game selection;
- filters for player/color/date/result/rating/time control/event/corpus;
- metadata search and cursor pagination;
- corpus and corpus-snapshot publication;
- freshness, coverage, sample size, and source-game drill-down;
- saved query foundation;
- query budgets, cache keys, slow-query observability.

### API milestone

`positions`, `position occurrences`, `opening explorer`, `search`, `corpora`, and `corpus snapshots`.

### Scale milestone

Run the accepted B0 load shape at 1–10 million games. Verify concurrent import does not break interactive SLOs; otherwise isolate/throttle/re-architect before exit.

### Acceptance criteria

- position key passes all fixtures including castling and en-passant distinctions;
- collision verification is exercised by an artificial collision test path;
- warm/cold P95 targets are met at accepted corpus size;
- filters produce independently verified counts;
- result exposes corpus snapshot and can list contributing games;
- interrupted rebuild resumes and never publishes a partial snapshot as current.

## 7. B4 — canonical players, ratings, and professional player cards

### Goal

Create reliable FIFA/Take Take Take-style player cards backed by provenance, separate rating systems, identity confidence, and permitted media.

### Scope

- real-person/player versus platform-account decision and schema;
- FIDE list ingestion with monthly snapshots and standard/rapid/blitz observations;
- user-authorized Lichess/Chess.com account connectors;
- aliases and identity candidate/review workflow;
- current/peak ratings and ranks by source/system/time control;
- recent eligible games/form and event coverage;
- licensed portrait/attribution workflow using approved sources;
- player snapshot projection and cache invalidation;
- card coverage, freshness, dispute/correction, and missing-data states;
- first opening preference panels from B3 aggregates.

### API milestone

`players`, `aliases`, `accounts`, `ratings/history`, `rankings`, `player snapshots`, and connector status.

### Acceptance criteria

- FIDE/online rating values never overwrite or masquerade as each other;
- monthly FIDE sync is idempotent and reports additions/changes/inactivation;
- uncertain matches remain candidates, not automatic merges;
- portrait response includes license/author/source/attribution or no portrait;
- provider rate-limit, 304, 404/410, and schema-change cases are tested;
- player card states its effective date, source coverage, and identity confidence.

### Legal gate

2700Chess and Take Take Take are visual/product references only unless a written integration agreement or documented public API grants the intended use.

## 8. B5 — analytics and evidence platform

### Goal

Produce reproducible player analytics whose inputs, sample sizes, uncertainty, versions, and supporting games are inspectable.

### Scope

- normalized cohort definitions and stable hashes;
- metric definition/version registry;
- scheduled/on-demand analytics jobs;
- repertoire entropy and opponent-adjusted performance as first validated metrics;
- sample thresholds, confidence/credible intervals, and not-enough-data behavior;
- player snapshot composition and time-window comparisons;
- immutable evidence bundle manifests and source-game citations;
- refresh-as-new-result behavior;
- metric validation fixtures and independent notebook/reference calculations;
- cost/latency budgets and aggregate caching.

### API milestone

`cohorts`, `metric definitions`, `metric results`, `player comparisons`, `evidence bundles`, and evidence drill-down.

### Acceptance criteria

- independently computed fixtures match within declared tolerance;
- result is reproducible from snapshot, cohort, and metric version;
- private games do not enter shared aggregates without explicit approved policy;
- small/biased samples return warnings or no score, not false precision;
- correction creates a new result/evidence bundle and marks prior freshness;
- evidence links remain stable after a new corpus snapshot is published.

## 9. B6 — engine analysis platform

### Goal

Run reproducible, resource-isolated Stockfish analysis without degrading interactive workloads.

### Scope

- engine build and analysis profile registries;
- dedicated engine queue/hosts and UCI supervisor;
- node/time/depth budgets, MultiPV, score/WDL normalization;
- cache key and immutable result format;
- batch game/position analysis, priority, quota, cost accounting;
- cancellation, retry, crash quarantine, hard resource limits;
- optional Syzygy asset/version handling;
- local-companion-compatible job/result contract, without requiring the companion yet.

### API milestone

`engine builds`, `profiles`, `analysis jobs`, `analysis results`, and progress/cancel.

### Acceptance criteria

- identical build/profile/position produces a reusable deterministic result within defined limits;
- illegal PV, mate score, terminal position, and engine crash fixtures pass;
- engine saturation does not violate API/database SLOs;
- quotas prevent one workspace from consuming the pool;
- cancelled job releases process/resources and records partial state honestly;
- binary/network licenses and checksums are recorded.

## 10. B7 — typed investigation tools and evidence-first AI

### Goal

Answer constrained natural-language chess research questions using typed deterministic tools and verified evidence.

### Entry gate

B3 and B5 evidence APIs are stable; B6 is required only for questions that invoke fresh engine analysis.

### Scope

- typed tool registry for player, game, position, metric, and engine queries;
- planner/executor budgets and permission-aware tool calls;
- claim extraction, claim type, evidence binding, and verification state;
- prompt/response versioning without storing private content unnecessarily;
- citation rendering contract and unsupported-claim refusal;
- provider abstraction, timeouts, quotas, and fallback;
- adversarial evaluation set for hallucination, identity confusion, stale data, prompt injection, and private-data leakage.

### Acceptance criteria

- database facts/statistics/engine claims cite valid immutable evidence;
- unsupported factual claims are rejected or explicitly labeled interpretation;
- tool authorization cannot be overridden by prompt content;
- AI outage leaves all deterministic workflows usable;
- evaluation thresholds for factual accuracy and unsupported-claim rate are met;
- per-investigation cost and trace are observable.

## 11. B8 — team workflows, scale hardening, and professional beta

### Goal

Operate the platform reliably for professional teams and a growing corpus, and extract specialized storage only where measured.

### Scope

- studies, chapters, revisions, comments, shares, roles, invitations;
- optimistic concurrency and conflict handling;
- team audit/export/deletion administration;
- scheduled reports and saved investigations;
- disaster recovery, load/failure/soak testing, capacity alarms;
- read replica and workload isolation when measurements require them;
- position/analytics datastore extraction only through accepted ADR/benchmark;
- local/private companion foundations if product validation supports them;
- broader licensed corpus connectors and operational data-quality tooling.

### Scale checkpoints

- 10M -> 25M -> 50M -> 100M games are separate approval gates;
- each gate requires rights, value, storage cost, rebuild time, and SLO evidence;
- no corpus-size milestone is accepted if deletion/correction and provenance cannot operate at that scale.

### Acceptance criteria

- team authorization and sharing matrix passes under concurrent edits;
- production-like 500-user load and mixed background workload meet SLOs;
- restore, connector outage, queue backlog, and failed migration drills pass;
- cost per active user/import/engine hour is known and alertable;
- support can diagnose a result from trace ID through corpus/metric/engine evidence;
- beta runbooks and on-call ownership exist.

## 12. Cross-phase workstreams

These never become “later” work:

- **data governance:** policy versions, source rights, retention, corrections, deletions;
- **security:** threat-model updates, dependency scanning, tenant tests, secret rotation;
- **quality:** golden fixtures, contract tests, load corpus, regression benchmarks;
- **operations:** dashboards, alerts, runbooks, restore drills, cost attribution;
- **documentation:** OpenAPI, ADRs, schema dictionary, operational procedures;
- **product validation:** observed professional workflows and usability evidence.

## 13. Recommended implementation order inside each phase

```text
contract and threat review
-> schema/API design
-> migration and fixture design
-> smallest end-to-end vertical slice
-> negative/failure behavior
-> scale and observability
-> documentation and runbook
-> acceptance review
```

## 14. Backlog rule

A ticket is implementation-ready only when it states:

- user/system outcome and phase;
- API/event/schema effect;
- permissions and visibility;
- idempotency/retry/cancellation behavior;
- provenance/deletion effect;
- acceptance examples and performance budget;
- telemetry and rollout/rollback plan;
- dependencies and explicit non-goals.

## 15. Decisions intentionally deferred

- commercial SaaS versus open-source packaging;
- native mobile client;
- full desktop/local companion scope;
- public social features;
- generic training/puzzle product;
- custom foundation-model training;
- proprietary corpus licensing deals.

Deferred decisions must not be smuggled into schema or product behavior without an ADR and the appropriate legal/product gate.
