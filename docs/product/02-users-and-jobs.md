# Users and jobs

Status: initial product hypothesis

## Audience priority

The selected audience is between two levels:

1. serious tournament players and coaches are the first design target;
2. teams, clubs, federations, and professional preparation groups are a later expansion.

Advanced amateurs should be able to use the product, but simplifying for beginners must not erase professional depth.

## Primary personas

### Competitive player

Needs to prepare against opponents, understand personal weaknesses, maintain annotated games, and make decisions under limited preparation time.

Core jobs:

- find what an opponent actually plays under relevant conditions;
- discover repertoire changes and likely preparation surprises;
- compare practical outcomes with rating-adjusted expectations;
- inspect representative games and critical positions;
- preserve private notes and analysis.

### Coach

Needs to study several players, explain findings, assign material, and distinguish stable weaknesses from noisy samples.

Core jobs:

- build evidence-backed player reports;
- compare a student with a peer cohort;
- identify recurring decision patterns;
- annotate games with variations, comments, NAGs, diagrams, and chapters;
- share selected material without exposing all private analysis.

### Analyst or second

Needs deeper filters, batch research, reproducible engine work, and traceability.

Core jobs:

- run position and structure investigations across large corpora;
- reproduce statistics and engine results;
- save queries and evidence bundles;
- monitor changes in a target's repertoire;
- export preparation artifacts.

### Team or federation administrator

Later-phase persona. Needs controlled workspaces, roles, retention rules, audit trails, shared corpora, and compute budgets.

## Canonical jobs to be done

| Job | Desired outcome | Evidence of success |
|---|---|---|
| Opponent preparation | Understand likely openings and recent deviations | A report whose claims open into source games |
| Player intelligence | Understand strengths, weaknesses, and change over time | Metrics include cohort, uncertainty, and methodology |
| Position research | Find every relevant occurrence of a position | Exact definition, filters, complete result count |
| Opening research | Compare continuations by cohort and era | Move tree, W/D/L, performance, representative games |
| Game analysis | Combine annotations, engines, and references | Saved analysis remains reproducible |
| Knowledge capture | Turn research into durable chapters and notes | Searchable, linkable, privately controlled material |
| Natural-language investigation | Express a multi-step question without manual filter choreography | Correct tools, correct arguments, supported answer |

## High-value questions

- How has this player changed their response to `1.e4` over three years?
- Which Catalan structures does this opponent enter as Black in classical games?
- Find games where Carlsen reached this pawn structure and won after an equal evaluation.
- Compare two players' repertoire breadth and opponent-adjusted performance.
- Which positions in my games produce the largest performance gap versus peers?
- Build a low-theory Black repertoire using master games, my history, and 1800-rated practical statistics.

## User constraints

- preparation time may be measured in minutes, not hours;
- source data can be incomplete, duplicated, or incorrectly attributed;
- private preparation has high confidentiality value;
- many users cannot interpret raw statistical uncertainty;
- engine compute and battery are finite;
- users may move between browser, desktop, and offline environments.

## Research questions

- Which three workflows justify switching from existing tools?
- Do players value novel metrics when underlying games are one click away?
- How much setup will users tolerate before first useful insight?
- Which private-data guarantees are required by titled players and coaches?
- Is the first team buyer a coach, club, academy, or federation?
