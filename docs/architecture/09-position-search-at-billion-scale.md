# Position search from curated OTB to a billion games

Status: target design; physical stores require benchmark decisions
Updated: 2026-09-26

## Query to answer

The board is at a legal position in a Najdorf branch. A user asks, “How did this FIDE player respond here as Black?” The system should return exact matching games and continuations in the selected eligible OTB corpus quickly, then optionally show a separate online comparison or engine result. Searching every PGN at request time is prohibited.

## Data separation

```text
PostgreSQL system of record
  players/FIDE identities, games, provenance, rights, studies, jobs, payments
Object storage
  immutable raw PGN, exports, snapshots, compact game archives
Position serving index
  position key -> postings, next-move counts, representative game IDs
Analytical store/projections
  player/time-control/date/opening cohorts and heavy aggregations
Redis
  disposable hot query and report cache
```

The existing [database design](../backend/DATABASE_DESIGN.md) uses partitioned PostgreSQL occurrences for the first 1–10M games. At a billion games, tens of billions of position occurrences require a separate sharded/compact position store or a demonstrated equivalent. PostgreSQL remains authoritative for identity, source policy, user work, and billing. [PostgreSQL partitioning reference](https://www.postgresql.org/docs/current/ddl-partitioning.html).

## Ingest once, query by key

For each accepted game:

1. Parse PGN, validate moves, preserve original source and rights metadata.
2. Resolve white/black FIDE identity candidates; never infer verified IDs solely from names.
3. Generate canonical positions after each ply. Position identity includes pieces, side to move, castling rights, and relevant en-passant state; store full state to verify hash collisions.
4. Write compact game metadata/moves and `position_key -> (game_id, ply, next_move, result, player IDs, date/cohort references)` postings.
5. Aggregate common position/next-move counts by immutable corpus snapshot and selected cohorts.
6. Publish a snapshot only after reconciliation of source counts, accepted games, indexed positions, and correction/tombstone records.

Transpositions that reach the same exact board state share a position key. A move-order query can additionally filter by path/PGN prefix when that distinction matters. Castling rights and en-passant cannot be discarded merely to get more matches.

## Request algorithm

```text
input: active node FEN, desired player FIDE ID, black, classical OTB, date range
validate FEN -> canonical state -> 128-bit position key
resolve player ID and corpus snapshot
retrieve posting list for exact position (or aggregate)
intersect with player/color/date/time-control eligibility
verify canonical state, deduplicate same game/position policy
compute next-move counts and representative games
return coverage + evidence IDs + bounded page of game references
```

For a popular opening position, the posting list may be huge. Do not materialize it into an API response or feed it to an LLM. Use precomputed counts for common dimensions, compressed posting lists, and bounded intersections. For rare positions, read the short posting list directly. A query planner chooses between `position-first` and `player-first` access based on estimated cardinality; a player with a few hundred OTB games should often be read from a compact player index, with positions checked only in those games.

Lichess's open [opening-explorer implementation](https://github.com/lichess-org/lila-openingexplorer) demonstrates that multi-billion-game position lookup is feasible with a purpose-built RocksDB-backed index, caches, and substantial storage/RAM. Its public `/masters`, `/lichess`, and `/player` endpoints are useful architectural references, but its API is not ChessScope's own professional corpus or entitlement system.

## Index tiers

| Tier | Data | Use |
|---|---|---|
| T0 | curated/licensed OTB games, full-game positions | professional opponent preparation and exact historical answers |
| T1 | selected online games, full moves + first N plies indexed | openings and high-frequency statistics |
| T2 | all approved online games, compact archive and metadata | broad corpus, player/time-control lookup |
| T3 | full arbitrary-position index over all online games | long-term goal after storage/compute benchmarks |

The tiered policy must be visible in query results. A T1 opening index cannot claim complete endgame coverage. To promise arbitrary position search across all one billion games, T3 must actually exist and be tested. To promise all games of a FIDE player, the OTB source portfolio must be sufficiently complete and identity-linked.

## Latency budgets for the interactive board

Proposed, not yet measured:

| Work | Target |
|---|---:|
| validate and normalize FEN | < 10 ms server CPU |
| hot exact position summary | P95 < 300 ms |
| cold exact position + common filters | P95 < 1.5 s |
| player report indexed counts | P95 < 3 s |
| deep filtered scan | asynchronous with progress |

The search response carries `corpus_snapshot`, `coverage_tier`, `filters`, `sample_size`, `source_policy`, `freshness`, and stable evidence references. These fields survive into AI answers. The performance figures are gates to test at 1M, 10M, 100M, and 1B; do not extrapolate the 1M result linearly.

## Hotspots and correctness

- The initial position and common openings are extreme hot keys. Cache their aggregates; avoid reading billions of postings for a simple move-frequency chart.
- Repeated occurrences of one position in a game follow an explicit counting policy so a repetition does not inflate “number of games.”
- Correction or removal of a game invalidates derived postings and snapshot eligibility, while existing evidence remains historically tied to the old snapshot with freshness warning.
- If the same game arrives from multiple sources, preserve each source record but count the canonical game once per corpus policy.
- Separate online, broadcast, licensed OTB, and private cohorts. Comparable-looking percentages can come from very different populations.
- Player identity corrections can change a report even when the position index does not; version the player-to-game mapping and cache key.

## Decision on storage

Use the existing PostgreSQL schema for the first OTB corpus and evaluate a purpose-built KV/RocksDB position service or an analytical store for very large online indexing. Choice requires reproducible measurements of bytes/occurrence, ingestion throughput, query P95/P99, write amplification, rebuild/deletion time, operational complexity, and cost. Avoid a full billion-row or billion-game PostgreSQL commitment before those measurements.
