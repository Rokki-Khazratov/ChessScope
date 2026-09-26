# Risk register

Status: active; review at every phase boundary

Scoring uses likelihood and impact from 1 (low) to 5 (critical). Score is their product, not a probability.

| Risk | L | I | Score | Leading signal | Mitigation / gate |
|---|---:|---:|---:|---|---|
| Insufficient professional data | 4 | 5 | 20 | Preparation queries miss known OTB games | Measure coverage; pursue licensing only after workflow validation |
| Position index does not scale economically | 4 | 5 | 20 | Storage/latency curves worsen at 1M–10M games | Reproducible multi-backend benchmark before architecture lock-in |
| Analytics are statistically misleading | 4 | 5 | 20 | Results change wildly with cohort/filter | Method review, uncertainty, leakage tests, minimum samples |
| AI invents chess facts | 5 | 4 | 20 | Unsupported names, counts, games, or evaluations | Typed tools, claim verifier, evidence-required rendering |
| Scope expansion prevents a coherent release | 5 | 4 | 20 | Roadmap accumulates training/social/play features | Explicit non-goals and phase exit criteria |
| Private preparation leaks | 3 | 5 | 15 | Cross-tenant result or cloud upload without intent | Tenant isolation, local mode, threat model, audit trails |
| Engine compute cost is unbounded | 4 | 4 | 16 | Batch analysis queue/cost grows faster than usage | Budgets, quotas, cache identity, local/browser paths |
| User imports contain restricted material | 4 | 4 | 16 | Commercial annotations appear in shared corpus | Private default, terms, classifiers, legal review, no silent pooling |
| Player identity merges are wrong | 4 | 4 | 16 | One profile combines different people | Probabilistic candidates, manual review, reversible merges |
| Web-first model fails professional offline needs | 3 | 4 | 12 | Target users reject upload/cloud dependency | Prototype local companion and offline boundary early |
| Third-party API access changes | 3 | 4 | 12 | Rate limits, blocked requests, changed terms | Adapter isolation, caching, user export fallback |
| Engine/worker lifecycle failures | 4 | 3 | 12 | Orphan processes, stuck jobs, ignored cancellation | Supervisor, deadlines, leases, heartbeats, process groups |
| PGN round trips lose analysis | 3 | 4 | 12 | Variations/comments disappear after export | Golden corpus and explicit fidelity report |
| Metrics become vanity scores | 3 | 4 | 12 | Users optimize a number without useful decisions | Always show method, comparison, uncertainty, source games |
| Product is only “prettier ChessBase” | 3 | 5 | 15 | Interviews cite aesthetics but no unique outcome | Validate evidence-first analytical jobs, not preference surveys |
| Vendor lock-in for AI | 3 | 3 | 9 | Prompts/tools depend on one provider feature | Provider-neutral tool/evidence contracts and eval set |
| Premature distributed architecture | 4 | 3 | 12 | Multiple stores before measured need | Postgres-first baseline with migration triggers |

## Top risk narratives

### Data quality and rights

The product can be technically excellent and still fail professional preparation if its corpus is incomplete, poorly normalized, or legally unusable. Coverage and provenance need product visibility. Licensing is a strategic workstream, not a deployment detail.

### Position scale

At an illustrative 10 million games and 80 ply per game, ingestion creates around 800 million position occurrences. Exact lookup, filtered aggregation, compaction, cache behavior, and rebuild time need measurement with realistic skew.

### Statistical validity

Raw win rates confound opponent strength, color, era, time control, rating pool, and selection. Novelty calculations can leak future games. Engine-derived conversion metrics depend on depth and calibration. A polished chart does not cure a weak estimator.

### AI trust

An LLM should never be the system of record for player identity, game counts, legal moves, statistics, or engine evaluations. The interface must distinguish fact from interpretation and refuse when evidence is insufficient.

### Infinite product surface

Chess naturally invites openings, analysis, training, puzzles, play, teams, courses, media, and community. The selected first paid release includes FIDE-centered database research, the board, engine, and constrained coaching chat. Other areas require an explicit phase decision; see [professional release scope](../product/05-professional-coach-release.md).

## Risk handling protocol

Each high-score risk must have:

- a named owner before implementation;
- an observable leading signal;
- a prevention or containment control;
- a phase gate or acceptance threshold;
- a dated review note;
- an ADR when the mitigation changes architecture.

## Kill criteria

Pause or reshape the product if research shows any of the following:

- target users cannot name a repeated task that is materially better than existing tools;
- a legally usable corpus cannot support the chosen first workflow;
- proposed signature metrics fail independent statistical review;
- position serving at the target scale is economically incompatible with the product model;
- hybrid privacy cannot meet the expectations of the professional audience.
