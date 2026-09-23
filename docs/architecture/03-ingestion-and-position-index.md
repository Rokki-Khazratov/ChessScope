# Ingestion and position index

Status: benchmark hypothesis

## Pipeline

```text
acquire source
-> checksum and policy classification
-> parse source-faithful record
-> validate legal move tree
-> normalize metadata and identity candidates
-> deduplicate/cluster
-> persist canonical game revision
-> generate position occurrences
-> update search/index projections
-> schedule analytics and optional engine work
-> publish corpus freshness
```

Each stage is idempotent and records errors without discarding the rest of a batch.

## PGN fidelity

The parser must account for:

- tag pairs, custom tags, missing or partial dates;
- standard and non-standard starting positions;
- comments, NAGs, nested RAV variations, and escaped text;
- clock/evaluation annotations embedded in comments;
- encoding and line-ending variation;
- malformed SAN and recoverable truncation;
- duplicate games with different annotations;
- unsupported variants, which must be rejected or quarantined explicitly.

The first product may index only standard chess while retaining unsupported raw source files.

## Canonical position identity

FEN is the interchange and display representation, not the primary large-scale key.

Identity includes:

- piece placement;
- side to move;
- castling rights;
- en-passant state only when relevant to legal move equivalence;
- variant/ruleset.

Halfmove and fullmove counters remain available but are normally excluded from search-equivalent identity.

Use a 128-bit position key or a 64-bit key plus secondary collision verification. Exact matches must verify canonical bytes before returning evidence. A hash collision may never silently produce a false game match.

## Position occurrence projection

Conceptual fields:

```text
position_key
verification_fingerprint
game_revision_id
ply
next_move
mover_player_id
mover_color
result_from_mover_perspective
played_at
time_control_class
rating observations
corpus/visibility dimensions
```

Not every dimension must be duplicated physically. The benchmark should compare joins, denormalization, partitioning, and pre-aggregates.

## Scale model

Illustrative estimate:

```text
10,000,000 games * 80 ply/game = 800,000,000 occurrences
```

This is not a capacity promise. It explains why “parse PGN and add an index” is insufficient as a long-term plan.

## Baseline storage candidates

### PostgreSQL

Use for the first 1–10 million-game experiments because it keeps transactions, metadata, and position lookup in one operational system. Risks include row/index overhead, write amplification, maintenance, and analytical interference.

### ClickHouse

Candidate for later position occurrences and analytical facts when multidimensional aggregations or write volume justify another store. It should not be added before a measured trigger.

### RocksDB-style serving store

Candidate for compact position-key lookup and a required reference prototype because Lichess has demonstrated the architecture at very large scale. It changes operational and query flexibility tradeoffs.

### Parquet and DuckDB

Preferred research path for bulk experiments, feature engineering, metric validation, and portable benchmark outputs. It is not the sole interactive exact-position backend.

## Required benchmark

Use the same legal game stream at:

- 100,000 games;
- 1 million games;
- 10 million games if resources permit.

Measure:

- games and positions parsed per second;
- end-to-end positions ingested per second;
- bytes per game and position occurrence;
- exact-position P50/P95/P99, warm and cold;
- opening-tree aggregation;
- player/date/time-control filtered lookup;
- common versus rare positions using Zipf-like workloads;
- rebuild, correction, and deletion cost;
- memory, CPU, disk, compaction, and write amplification;
- behavior while ingest and reads run concurrently;
- operational recovery from interrupted imports.

## Data-quality outputs

Every batch reports:

- source records seen;
- parsed, rejected, and partially recovered games;
- legal/illegal move-tree counts;
- exact and probable duplicates;
- unresolved player/event identities;
- indexed position occurrences;
- lost/unsupported annotation features;
- source/corpus policy classification;
- time and resource usage.

## Correctness tests

- known transpositions resolve to the same position identity;
- castling and legal en-passant distinctions are preserved;
- illegal or variant positions cannot pollute standard indexes;
- hash collision simulation exercises secondary verification;
- duplicate source records do not inflate selected corpus statistics;
- future corrections can rebuild all derived projections;
- exact search matches a slow reference implementation on golden corpora.

## Migration contract

Application services query an abstract position service returning a stable response contract. Storage-specific cursors and keys must not leak into user evidence IDs. This allows the physical index to move without invalidating saved research.

## Primary reference

The [Lichess Opening Explorer README](https://github.com/lichess-org/lila-openingexplorer/blob/master/README.md) documents RocksDB, compressed PGN imports, hardware/cache observations, and production traffic. It is a reference case, not an automatic technology decision.
