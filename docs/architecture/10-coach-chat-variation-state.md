# Board-synchronized coaching chat and variation memory

Status: proposed interaction and backend contract
Updated: 2026-09-26

## Product behavior

The assistant is a coach operating inside an analysis workspace. It sees the active board position, the selected variation node, the current study, chosen player/corpus filters, and evidence already retrieved. It can ask for engine analysis, find historical games, propose a line, explain a strategic motif, and point to a square. It can refer to “the Najdorf branch we called B” after the user navigates elsewhere or returns later.

The durable context is stored as structured chess data. A language-model context window is a temporary view over that data; it is not the database of record.

## State model

```text
Workspace
  Study
    Chapter (root FEN, corpus snapshot, filters)
      Variation node (parent node, UCI move, SAN, resulting FEN/key, label)
        Annotation / arrow / evidence links / engine result refs
  Chat thread
    Message (text, author, timestamp, active node ID, tree revision)
    Tool call/result (typed arguments, evidence, cost, status)
    Proposed action (variation delta, preconditions, accepted/rejected)
```

Node IDs are stable. A branch name is mutable display metadata and is not its identity. A transposition may have the same position key as another node but a different path and commentary; the chat can mention both. Moves are stored in UCI plus generated SAN so text can be rerendered safely. The server validates the move against the parent position before saving it.

### Minimal turn envelope

```json
{
  "thread_id": "opaque-id",
  "study_id": "opaque-id",
  "active_node_id": "node-B-12",
  "tree_revision": 18,
  "position_key": "canonical-hash",
  "fen": "current legal FEN",
  "selected_branch_path": ["node-root", "node-B-1", "...", "node-B-12"],
  "corpus_snapshot_id": "otb-2026-09",
  "filters": {"color": "black", "time_control": "classical"},
  "user_text": "Why did branch B choose ...Nc6?"
}
```

`fen` is rederived/validated from the tree on the server. The client cannot assert an arbitrary position and gain access to unrelated private resources. If `tree_revision` has advanced, the server resolves the stable node or requests a conflict resolution rather than applying a move to the wrong branch.

## Agent tool contracts

```text
get_workspace_context(study_id, active_node_id, nearby_depth, named_branches)
resolve_fide_player(query)
search_games(player_id, position_key, color, corpus, filters)
get_opening_tree(position_key, cohort)
get_engine_result(position_key, profile) / enqueue_engine_analysis(...)
get_board_features(position_key)
check_goal_on_line(root_node, legal_moves, goal_predicate)
propose_variation(parent_node_id, legal_uci_moves, label, annotations)
get_evidence(bundle_id)
```

Each tool returns data with source, snapshot, method, warnings, and permission scope. The LLM receives compact summaries and selected games/PVs, not millions of rows. Factual numbers and move legality are verified before being shown.

## Memory construction

For each turn, assemble a bounded context packet from:

1. exact active position and path from chapter root;
2. named branch directory (`name -> node ID, root position, last updated`);
3. last relevant messages on this node and its ancestors;
4. user-pinned preparation goals and repertoire assumptions;
5. relevant evidence/engine references, fetched on demand;
6. a versioned summary of older chat, with links back to original messages.

Do not send the entire study or all source games on every turn. Retrieval uses node IDs, position keys, labels, and semantic text search for notes. Summaries are disposable accelerators; the underlying tree and messages remain authoritative. If the user says “this variant,” the active node resolves it; if several branches fit, the assistant asks which one inside the chat.

## Streaming and board control

The chat streams text and structured UI events separately:

```text
message_delta
tool_status
evidence_card
board_preview(line_id, node_id, moves, arrows, highlights)
variation_proposal(parent_node_id, legal_moves, label, precondition_revision)
engine_update(score, depth, nodes, PV)
completion / failure
```

Preview events are ephemeral and may animate pieces on a ghost/preview board state. They do not change the saved study. Accepted variation proposals run a transaction that checks parent node, revision, legality, and authorization, then creates nodes and emits a study update. A user can drag a piece or navigate while a response streams; every event remains bound to the node/revision at which it was generated. Stale events never overwrite the current board.

## Worked examples

### “How did Carlsen play this position as Black?”

Resolve Magnus Carlsen's FIDE ID -> exact position lookup in OTB corpus -> intersect with Black, relevant dates/time controls -> cite the game IDs, opponents, dates, counts, next moves. A separate section may show verified online games. If exact matches are absent, say so and offer similar pawn structures as a different query. The engine is unnecessary for the historical fact.

### “Trade knights and reach opposite-colored bishops”

Parse the target as a board-state goal and a practical/forced intent. Extract current material and legal moves, ask engine worker for bounded candidate lines, validate every move, test goal predicate at nodes, and inspect opponent alternatives. Preview promising branches with evaluations and label strength of support. The assistant must not claim a forced outcome from a single PV.

### “Use the open d-file, defend the c-file”

Compute whether files are open, which pieces control or invade squares, and candidate rook/queen maneuvers. Compare engine lines and human games in the same structure. Highlight d- and c-file squares and create two labeled branches: active plan and opponent counterplay. Explain with coordinates and exact node context.

## Report generation as agent workflow

For the tournament scenario:

```text
FIDE ID + user's color/repertoire + target date
-> resolve identity and corpus coverage
-> fetch indexed games/rating/event facts
-> repertoire distribution and recent changes
-> select candidate lines / representative games
-> spend bounded engine budget on a few critical positions
-> assemble evidence bundle
-> write checked report + saved study branches
```

The report job is resumable. User sees early deterministic sections while engine work continues. Its ten-minute target relies on pre-ingested OTB data; it cannot fetch and clean an entire new corpus during one request.

## Failure and safety semantics

- No valid position: explain which move/FEN is invalid and leave the tree intact.
- Missing player games: show corpus coverage and available source links; do not invent repertoire weaknesses.
- Ambiguous player identity: request FIDE ID or let user choose a candidate.
- Engine queue full: show cached/shallower analysis and ETA; the board remains usable.
- LLM timeout: keep saved study, engine, and search results available.
- Stale tree revision: regenerate or attach proposal to original node, never force-apply to the new one.
- Imported PGN comments and web text are treated as data, not instructions to the assistant.

## Quality evaluation

Build a professional test set with titled-player review: exact historical lookups, transpositions, ambiguous IDs, four-branch navigation, 20–30-ply recall, tactical claims, strategic file/structure explanations, stale board events, and weak sample warnings. Score move legality, citation accuracy, branch reference accuracy, unsupported “forced” claims, usability, latency, and engine/LLM cost.
