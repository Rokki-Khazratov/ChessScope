# AI and evidence

Status: platform contract; constrained board-synchronized coaching chat is required for the first paid professional release

The active board node, named variations, stale-response handling, and structured chat memory are specified in [Board-synchronized coaching chat](10-coach-chat-variation-state.md). The first release only needs the typed tools that support historical position questions, bounded engine plans, board features, and opponent reports; the broader tool list below remains the long-term surface.

## Core separation

```text
structured facts -> deterministic tools
engine judgment -> reproducible engine service
documents and notes -> retrieval where appropriate
planning and explanation -> LLM
verification -> policy and deterministic checks
```

RAG is useful for notes, books with valid rights, annotations, concepts, and documentation. It is not the correct mechanism for exact player/game/position counts.

## Typed tool surface

Candidate contracts:

```text
resolve_player(query, source_hints)
search_games(filters, corpus)
get_game(game_id, revision)
position_stats(position, filters, corpus)
opening_tree(position, filters, corpus)
player_profile(player_id, cohort)
compare_players(player_ids, dimensions, cohort)
compute_metric(metric_version, subject, cohort)
engine_analyze(position, analysis_profile, privacy_route)
find_similar_positions(position, similarity_version, filters)
get_evidence_bundle(evidence_id)
```

The production agent never receives unrestricted SQL or arbitrary process execution.

## Tool result envelope

Every factual tool result includes:

```json
{
  "data": {},
  "evidence_id": "...",
  "corpus_snapshot": "...",
  "filters": {},
  "sample_size": 0,
  "method_version": "...",
  "generated_at": "...",
  "warnings": []
}
```

Large source ID sets live in the referenced evidence bundle rather than the prompt.

## Claim model

The LLM proposes structured claims:

```text
text
type: database | statistic | engine | interpretation | recommendation
evidence_ids[]
confidence
scope qualifiers
```

Verification rules:

- numeric database/statistical values must equal cited tool output under formatting tolerance;
- named games and players must resolve to cited IDs;
- chess moves and PVs must be legal from the cited position;
- engine claims must cite the analysis result and budget;
- interpretations must be visibly labeled and cannot masquerade as facts;
- claims without sufficient evidence are rejected or rewritten as uncertainty.

## Example investigation

Question:

> How has Abdusattorov responded to the Catalan as Black in classical games since 2024?

Plan:

```text
resolve_player
-> resolve opening/position definition
-> search_games with color/date/time control
-> opening_tree
-> compare time windows
-> select representative games
-> build evidence bundle
-> draft claims
-> verify claims
-> render answer and drill-down controls
```

The interface shows matched count, corpus, definition, filters, continuation distribution, uncertainty, and representative games. It does not imply completeness beyond the selected corpus.

## Anti-hallucination requirements

1. Deterministic facts require tool output.
2. Numerical claims require evidence IDs.
3. Entity ambiguity is surfaced to the user or resolved by a deterministic policy.
4. FEN, positions, and move sequences are validated before tool execution.
5. “No results” is a valid result; the model must not fill the gap.
6. Insufficient sample and incomplete corpus warnings survive summarization.
7. Raw tool traces and provider/model identity are retained according to privacy policy.
8. Prompt/tool data is treated as untrusted input against injection.
9. Private evidence is never sent to an external model without an allowed route.

## Provider strategy

Do not train a language model initially. Define a provider-neutral orchestration boundary and select models through a ChessScope-specific evaluation suite. “Our AI coach” describes our orchestration, chess tools, memory, validation, and UX; it does not require training a foundation model from scratch. Support a commercial API path for quality and a local/open-weight path for privacy only when each meets measurable requirements.

Provider portability does not mean the lowest common denominator. Tool/evidence semantics remain stable while adapters may use provider-specific structured-output capabilities.

## Evaluation set

Build 200–500 versioned questions covering:

- exact factual lookup;
- filtered aggregation;
- player comparison;
- position and transposition search;
- engine analysis;
- temporal questions;
- ambiguous identities;
- empty/impossible queries;
- adversarial instructions inside PGN/comments/documents;
- unsupported causal or historical claims.

Metrics:

- tool selection and argument accuracy;
- final numeric accuracy;
- evidence precision/recall;
- unsupported-claim rate;
- ambiguity and insufficient-evidence behavior;
- latency and monetary/compute cost.

## Evidence UX contract

Prefer object chips over detached footnotes:

```text
Abdusattorov used ...dxc4 in 31 of 74 matching games.
[31 / 74 games] [Classical · Black · 2025–2026] [Method] [Open games]
```

Facts and interpretations use different visual treatment. Opening an evidence object reveals the normalized query and source set.

## Threats

- prompt injection in comments, annotations, imported documents, or web results;
- tool argument escalation beyond user permissions;
- inference of private preparation from shared caches;
- fabricated evidence IDs;
- stale answers after corpus updates;
- provider retention of sensitive prompts;
- confident interpretation of statistically weak data.

All are architecture concerns, not prompt-writing problems.
