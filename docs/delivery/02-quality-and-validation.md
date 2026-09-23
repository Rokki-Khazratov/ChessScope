# Quality and validation strategy

Status: required quality gates

## Quality model

ChessScope correctness has multiple independent dimensions:

1. chess legality and notation fidelity;
2. identity and data provenance;
3. search completeness and isolation;
4. statistical validity;
5. engine reproducibility and lifecycle safety;
6. AI factual grounding;
7. security, privacy, accessibility, and usability.

A green unit-test suite in one layer does not prove the product is trustworthy.

## Golden corpora

Maintain small, reviewable datasets containing:

- standard games and transpositions;
- alternate starting FENs;
- castling/en-passant edge cases;
- promotions, mates, repetitions, and truncated games;
- nested variations, comments, NAGs, clocks, evaluations, and encodings;
- duplicates with different metadata/annotations;
- ambiguous player names;
- missing dates, ratings, events, and time controls;
- malformed but recoverable and unrecoverable PGN;
- private/public visibility mixtures.

Expected normalized output and search results are versioned.

## Test layers

### Parser and chess core

- unit and property tests for move legality, SAN/UCI/FEN/PGN;
- round-trip tests with a documented loss model;
- fuzzing for parser crashes, nesting, and resource exhaustion;
- differential tests against a trusted reference library/tool.

### Data and indexing

- import idempotency;
- source-to-canonical provenance;
- duplicate and identity review fixtures;
- position-key collision simulation;
- exact-position results compared with a slow reference scan;
- correction, deletion, rebuild, and snapshot consistency;
- tenant/visibility filters in every query path.

### Analytics

- formula fixtures and property tests;
- temporal leakage tests;
- bootstrap/Bayesian reproducibility with fixed seeds/config;
- missing-data and minimum-sample behavior;
- rating-pool separation;
- stability across corpus resamples;
- independent methodology review.

### Engines

- UCI conformance and malformed-output tests;
- process crash, hang, timeout, and cancellation;
- orphan process detection;
- job lease loss and duplicate execution;
- cache-identity correctness;
- legal PV validation;
- privacy-route and quota enforcement.

### AI and evidence

- frozen question/evidence evaluation set;
- tool and argument scoring;
- exact numeric and entity verification;
- unsupported claim detection;
- prompt injection from comments/documents;
- permission-scoped retrieval;
- provider regression comparison;
- no-evidence and ambiguity behavior.

### Application and UX

- contract and integration tests for core workflows;
- end-to-end tests for import, position research, player profile, annotation, and job cancellation;
- accessibility automation plus manual keyboard/screen-reader checks;
- usability studies using real preparation tasks;
- responsive checks for supported workflows.

## Performance methodology

- publish hardware, software, dataset checksum, and configuration;
- separate cold and warm cache;
- use common and rare positions with realistic skew;
- benchmark concurrent ingest and read workload;
- report P50/P95/P99, throughput, resource use, and errors;
- retain raw benchmark output and scripts once coding begins;
- do not compare backends with unequal durability or result semantics.

## Release gates

### Before private alpha

- golden corpus passes;
- exact search matches reference implementation;
- import failures are visible and recoverable;
- tenant isolation and deletion are tested;
- engine jobs cannot leave known orphan processes;
- first metrics pass methodology review.

### Before public alpha

- load/cost model validated;
- backup and restore exercised;
- observability and incident ownership defined;
- external API failure behavior tested;
- security threat model reviewed;
- accessibility baseline passes.

### Before AI answers are labeled factual

- deterministic claims require evidence by schema;
- verifier and permission checks cannot be bypassed by model output;
- eval thresholds are documented and passing;
- provider/model versions are recorded;
- corpus incompleteness and uncertainty survive answer generation.

## Production observability

Track:

- import lag, failures, and data-quality categories;
- corpus snapshot freshness and coverage;
- query latency/error by class and corpus;
- job queue age, cancellation, retries, and orphan detection;
- engine and AI cost per workflow;
- evidence verification failures;
- unauthorized-access denials and suspicious import activity;
- user-visible degradation state.

## Review culture

Architecture changes require an ADR. Statistical features require a methodology review. Data-source changes require a rights/provenance review. AI provider changes require the domain eval suite. User-facing claims are reviewed as trust surfaces, not just copy.
