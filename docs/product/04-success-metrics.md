# Success metrics

Status: proposed measurement framework

ChessScope should be judged by research outcomes, not by feature count or generated prose.

## North-star outcome

**Verified insight completion:** the share of serious research tasks in which a user reaches a useful conclusion and can inspect the supporting games or analysis without leaving ChessScope.

This requires qualitative task studies before it can be reduced to one numeric KPI.

For the first paid professional release, the canonical study task is preparing for an unfamiliar FIDE opponent within ten minutes using eligible OTB games, a saved analysis branch, and a source-backed coaching answer. Track corpus coverage and report trust alongside completion time.

## Product metrics

| Dimension | Candidate measure | Guardrail |
|---|---|---|
| Time to evidence | Median time from query/start position to opening a relevant source game | Fast answers must not hide incomplete corpora |
| Research completion | Percentage of benchmark tasks completed correctly | Correctness is manually sampled |
| Drill-down use | Supported claims opened into evidence | Low use may indicate trust or discoverability problems |
| Reuse | Saved searches, chapters, and preparation workspaces reopened | Avoid optimizing for empty saves |
| Import success | Games accepted, rejected, deduplicated, and partially preserved | Never count silent data loss as success |
| Retention | Serious users returning for another preparation/research session | Segment by player, coach, and analyst |

## Search service objectives

Initial targets are hypotheses until benchmarked with realistic hardware and corpora:

- game metadata search: P95 under 500 ms for interactive filters;
- warm exact-position lookup: P95 under 250 ms at the validated MVP scale;
- opening-tree aggregation: P95 under 750 ms for common filters;
- saved evidence links remain resolvable across compatible dataset versions;
- result completeness and corpus freshness are displayed, not inferred.

## Analytics quality

- 100% of displayed metrics identify version, cohort, filters, sample size, and uncertainty status;
- zero known temporal leakage in novelty and historical models;
- peer comparisons never mix rating pools without an explicit calibration policy;
- metric regressions are evaluated against frozen golden datasets;
- statistical claims are reviewed before being labeled product facts.

## Engine quality and cost

- 100% of persisted engine results record engine/version, options, position, and budget;
- duplicate jobs with identical identities reuse compatible cached work;
- cancellation stops or reclaims compute within a defined bound;
- cost per analyzed position and per imported game is observable;
- local analysis does not upload private positions unless the user selects a cloud path.

## AI coaching quality in the first paid release

Measure separately:

- tool-selection accuracy;
- argument accuracy;
- numeric factual accuracy;
- evidence precision and recall;
- unsupported-claim rate;
- ambiguity handling;
- reference accuracy for named variation branches and active board nodes;
- legality of proposed branch moves and resistance to stale board updates;
- unjustified use of “forced” for goal-oriented engine lines;
- refusal/insufficient-evidence correctness;
- latency and provider cost.

The target for deterministic numeric claims is exact agreement with verified tool output, not semantic similarity.

## Phase 0 success

The research phase succeeds when it produces:

1. a validated top-three workflow set;
2. a legal data-source matrix;
3. a repeatable position-index benchmark at 100k, 1M, and preferably 10M games;
4. accepted definitions for the first two analytics metrics;
5. an engine compute and privacy model;
6. a reviewed architecture baseline and kill list.

Writing application code before these conditions are met is activity, not progress.
