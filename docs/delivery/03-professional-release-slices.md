# Professional release slices and billion-game expansion

Status: updated execution order for the user-directed professional scope
Updated: 2026-09-26

This plan refines the [B0–B8 backend plan](../backend/PHASED_IMPLEMENTATION_PLAN.md). The new first paid release includes FIDE-centered opponent research, a saved board/variation tree, VPS Stockfish, and an evidence-backed coaching chat. Billion-game online coverage is an expansion milestone and does not stand in for OTB quality.

## P0 — proof of the hard dependencies

Deliverables:

- audit usable OTB/broadcast games, FIDE IDs, time controls, source rights, and freshness;
- select 50 real professional opponent-preparation cases and measure existing corpus coverage;
- benchmark 100k/1M/10M-game position indexes and representative filtered queries;
- benchmark one intended VPS with quick/standard/deep Stockfish jobs under concurrency;
- prototype a saved four-branch analysis tree and stale-response handling;
- establish citations, goal-checker semantics, and AI evaluation cases;
- document paid entitlement and cancellation behavior before payment integration.

Exit: the team can answer how many relevant OTB games are actually found for the test opponents, how many engine slots a host sustains, and what the position index costs per million games. If coverage is poor, sourcing or the promise changes before building the paid UI.

## P1 — professional corpus and identity

Deliverables:

- official FIDE ID/rating import and player-card basics;
- approved OTB/broadcast PGN ingestion with source-policy separation;
- identity candidates and reviewed merges;
- exact game/player/event search and source-game drill-down;
- initial 1M-game corpus with snapshot/freshness/coverage indicators;
- first opponent repertoire queries by color, date, time control, and opening.

Exit: a FIDE ID returns an attributable card and source games; uncertain names are not silently attached. At least 50 prepared test cases reveal known data gaps honestly.

## P2 — analysis workspace and engine

Deliverables:

- responsive web board, legal moves, PGN import/export, comments, named nested variations, arrows, and move tree;
- persistent node IDs and tree revisions;
- position explorer with continuation counts and representative games;
- Stockfish worker pool on CPU VPS with quick/standard/deep profiles, cache, queue, cancellation, and progress;
- evidence links from board nodes to games and engine results.

Exit: four named branches of 20–30 plies remain navigable after reload, engine work does not slow ordinary search, and the source of each engine score is visible.

## P3 — paid professional coach release

Deliverables:

- landing, account creation, hosted payment, verified webhook, entitlement, dashboard, billing portal;
- right-side coaching chat bound to active node and named branches;
- typed historical-search, position, engine, board-feature, goal-check, and variation-proposal tools;
- streaming text, source-game chips, board previews, variation proposals, and accepted branch creation;
- ten-minute opponent briefing job with progressive deterministic results and bounded engine checks;
- citation/unsupported-claim evaluation and coach review.

Exit: paid-user access is correct under duplicate/out-of-order events; an unfamiliar opponent with adequate corpus coverage receives a sourced briefing; all three representative chat prompts in [the release spec](../product/05-professional-coach-release.md) work within declared limitations. A titled-player evaluator can trace key claims to games or engine jobs.

## P4 — broader coverage and scale

Deliverables:

- direct OTB source partnerships and incremental round feeds;
- selected CC0 Lichess online imports in separately labeled cohorts;
- specialized position serving store when measured PostgreSQL limits are reached;
- multi-host engine fleet and autoscaling/admission control;
- advanced opponent trends, repertoire change detection, and batch reports;
- 100M then 1B online game gates with explicit index coverage tiers.

Exit at each corpus gate: rights, bytes/game, bytes/occurrence, query P95/P99, rebuild/correction time, and spend are measured; professional OTB report quality is maintained.

## Hackathon demonstration within this strategy

The hackathon prototype should prove the professional product loop with a small *real and permitted* OTB sample: FIDE ID -> opponent card -> indexed recent games -> one board position -> historical lookup -> Stockfish line -> coaching explanation -> saved named branch. A checkout may run in provider test mode. A prototype must label its limited corpus coverage and simulated steps. The intended production architecture remains as above; a hackathon demo is not a claim that a billion-game index or ten-minute report is already operational.

## Phase dependencies

```text
OTB source rights + identity -> opponent report truthfulness
position identity + game index -> historical board questions
saved variation tree -> persistent coaching context
isolated engine workers -> concrete move evaluation
evidence API + above -> AI coach
entitlement state -> paid app access
measured storage economics -> billion-game rollout
```
