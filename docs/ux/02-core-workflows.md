# Core workflows

Status: workflow specification for prototypes and usability studies

## 1. Import a PGN collection

### Entry

User drops/selects PGN, chooses private workspace and local/cloud processing policy.

### Flow

1. Show file checksum, size, expected privacy route, and supported features.
2. Stream upload or hand off to local companion.
3. Parse and validate in background.
4. Show running counts: accepted, partial, rejected, duplicate, unresolved identity.
5. Present a completion report and searchable collection.
6. Allow download of detailed errors without exposing raw private content to logs.

### Success

The user can open imported games and understands any loss or ambiguity. Reimport is idempotent.

## 2. Research a position

### Entry

Paste FEN/PGN, play moves on a board, or open a position from a game.

### Flow

1. Validate the position and show its move path/transpositions.
2. Select corpus and filters.
3. Load exact continuation statistics immediately.
4. Show representative/top/recent games with their selection method.
5. Open any game at the occurrence.
6. Optionally run local or cloud engine analysis.
7. Save the query or add the position/evidence to a study.

### Failure behavior

Invalid FEN identifies the problem. Zero matches remains useful and never triggers fabricated examples.

## 3. Build a player profile

### Entry

Search for player name/account.

### Flow

1. Resolve identity and show aliases, sources, and ambiguity.
2. Choose color, date, time control, corpus, and rating context.
3. Load repertoire and performance overview.
4. Compare time windows and identify changes.
5. Drill every metric into games and method.
6. Save a preparation snapshot with corpus version.

### Success

The user can distinguish “what this player does” from “how confident the data is.”

## 4. Prepare for an opponent

### Entry

Select opponent and intended color/opening context.

### Flow

1. Confirm identity and expected match conditions.
2. Summarize recent and long-term repertoire.
3. Highlight high-frequency lines, change points, and rare deviations.
4. Compare performance against expected score and relevant peers.
5. Present representative games and critical positions.
6. Add selected lines, notes, and evidence to a private study.
7. Optionally request deeper engine jobs with explicit budget.

### Non-goal

The system does not claim a rare move was prepared without supporting evidence.

## 5. Annotate a game

### Flow

1. Open/import/create the game or line.
2. Navigate board and notation by keyboard or pointer.
3. Add/promote/delete variations with undo history.
4. Add comments, NAGs, arrows/squares, diagrams, and chapter metadata.
5. Attach reference searches, engine results, and evidence snapshots.
6. Save a new revision automatically; show sync/local status.
7. Export PGN with a fidelity report.

### Conflict behavior

Concurrent edits create an explicit conflict or merge workflow; they never silently discard a variation.

## 6. Run engine analysis

### Flow

1. Choose local or cloud route.
2. Choose profile: quick, standard, deep, research.
3. Review estimate and privacy implications.
4. Start job; show queue/running state and cancellation.
5. Stream compatible partial PVs where possible.
6. Persist the completed result with engine provenance.
7. Reuse only a compatible cached result.

## 7. Ask an analytical question in the coaching chat

### Flow

1. User asks in the current player/game/position context.
2. System displays interpreted entities and filters.
3. Agent executes permission-scoped typed tools.
4. Claim verifier checks facts and evidence.
5. Answer renders with evidence objects and caveats.
6. User opens games, modifies filters, or saves the investigation.

The assistant turn is bound to the selected variation node and tree revision. It can preview legal lines on the board and propose a named branch; saved branch creation checks the current revision and user action. See [variation-aware coaching contract](../architecture/10-coach-chat-variation-state.md).

### Recovery

Ambiguous player or opening definitions trigger a focused clarification. Insufficient evidence produces a constrained answer, not speculation.

## Prototype test set

Build clickable prototypes for only five surfaces initially:

- Explore;
- Player;
- Position;
- Game workspace;
- evidence/AI investigation.

Test with tournament players and coaches using real tasks. Measure task completion, time to first evidence, errors, trust calibration, and which existing tool they would otherwise use.
