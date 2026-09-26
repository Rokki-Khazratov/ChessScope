# Roadmap

Status: earlier outcome-based sequence; professional first-release ordering is in 03-professional-release-slices.md

The roadmap follows validated outcomes, not a promise to ship every listed feature. The long-term scale target is ChessBase-class capability, reached through measured stages.

Backend delivery phases B0–B8 provide infrastructure detail in [Backend phased implementation plan](../backend/PHASED_IMPLEMENTATION_PLAN.md). The user's subsequent professional-release decision brings FIDE-centered OTB research, board-synchronized AI coaching, and paid access into the first release. The updated delivery sequence is [Professional release slices](03-professional-release-slices.md), which controls ordering where this earlier roadmap differs.

## Phase 0 — de-risk the foundation (0–3 months)

No production feature sprint should replace this work.

### Research outcomes

- interview serious players, coaches, and analysts;
- benchmark five workflows against existing tools;
- complete data-source and licensing review;
- audit En Croissant and Lichess Opening Explorer at code/issue level;
- define and challenge the first two metrics;
- produce an AI-agent evaluation design, not a chat demo.

### Disposable technical experiments

- PGN parser and fidelity corpus;
- canonical position/key reference implementation;
- position index at 100k, 1M, and if feasible 10M games;
- PostgreSQL, analytical-columnar, and key-value comparisons;
- isolated UCI worker lifecycle experiment;
- typed tool plus claim-verification experiment.

### Product prototypes

- Explore;
- Player;
- Position;
- Game workspace;
- evidence drawer/investigation.

### Exit criteria

- three repeated high-value user jobs are validated;
- a legally usable first corpus is identified;
- storage choice is supported by reproducible results;
- entropy and opponent-adjusted performance definitions pass review;
- privacy boundary for local/cloud work is accepted;
- architecture ADRs are updated with evidence.

## Phase 1 — private technical alpha (3–6 months)

### Outcome

A small group can import games, search players/positions, inspect an opening tree, and verify player metrics against source games.

### Scope

- accounts and private workspaces;
- PGN, Lichess, and conservative Chess.com user imports;
- canonical game and player identity pipeline;
- game/player/exact-position search;
- opening tree and representative games;
- basic game workspace and annotation;
- repertoire entropy and opponent-adjusted performance;
- evidence drawer;
- one local/lightweight and one cloud engine path if safe.

### Scale target

Prove the chosen architecture on 1–10 million games; exact corpus size follows rights and benchmark cost.

### Exit criteria

- target users complete benchmark workflows correctly;
- no known silent PGN data loss;
- query SLOs meet the accepted target;
- metrics remain stable and explainable;
- tenant isolation and engine lifecycle tests pass;
- operating cost per active research workflow is understood.

## Phase 2 — public alpha and investigations (6–9 months)

### Outcome

Users can perform a constrained set of natural-language investigations whose factual claims are evidence-backed.

### Scope

- trusted typed tools;
- player comparison and temporal queries;
- claim verification and evidence-linked answers;
- saved investigations and studies;
- basic novelty model if leakage validation passes;
- import and compute quotas;
- public reliability/coverage indicators.

### Exit criteria

- AI eval targets pass across factual accuracy and unsupported-claim rate;
- non-AI workflows remain fully usable;
- evidence links remain stable across data refreshes;
- abuse/cost controls operate under load.

## Phase 3 — professional beta (9–18 months)

Only pursue after demonstrated repeat use.

### Scale

- grow 10M -> 25M -> 50M -> 100M games as rights, cost, and value justify;
- split position/analytics storage only at benchmark trigger;
- incremental correction, deletion, and rebuild operations;
- broader OTB/curated corpus through licensing or partnership.

### Intelligence

- conversion and recovery;
- preparation surprise and style drift;
- structure similarity;
- human-likelihood/practical difficulty models;
- deeper opponent-specific preparation.

### Product depth

- local desktop companion and offline/private workflows;
- repertoire artifacts and selective training;
- coach/team workspaces, roles, reports, and collaboration;
- richer annotation and publishing only where demanded.

## Deferred until evidence exists

- native mobile research client;
- generic puzzle platform;
- online play;
- tournament operations;
- social/community feed;
- course marketplace;
- custom language-model training.

## Business model track

Commercial SaaS versus open-source remains unresolved. Architecture should track real cost centers—storage, deep engine jobs, large imports, AI inference, collaboration—without inventing artificial limits before product value is validated.

Resolve before public beta:

- source-code strategy;
- dependency and dataset compatibility;
- hosted/free/pro/team boundaries;
- local companion distribution;
- contribution and governance model if open source.

## Long-term moat checkpoint

By month 18, evaluate whether ChessScope has accumulated defensible value in:

- canonical identity and provenance;
- position graph and derived aggregates;
- validated player metrics;
- reusable analysis and evidence;
- professional workflow history;
- licensed/curated or user-private data relationships.

If not, adding more UI features will not fix the strategy.
