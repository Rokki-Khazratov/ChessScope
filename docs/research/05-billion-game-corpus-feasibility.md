# Billion-game corpus: availability, rights, and cost

Status: research findings plus proposed ingestion plan
Reviewed: 2026-09-26

## Answer in one paragraph

One billion games are obtainable as **online games**: the official Lichess open database currently lists 8,130,696,420 standard rated games under CC0 and roughly 90 million new games per recent month. This is an excellent source for scale and broad position statistics. It does not solve the professional **FIDE opponent-preparation** problem by itself. OTB game scores, reliable FIDE IDs, recent open-tournament coverage, time controls, and legal reuse must be built as a separate curated corpus. We should not claim that a billion online games equals a billion FIDE-attributed tournament games. [Source: Lichess database](https://database.lichess.org/).

## Verified source inventory

| Source | Published scale / content | Appropriate use | Critical gap |
|---|---|---|---|
| [Lichess standard dump](https://database.lichess.org/) | 8.13B rated standard games on review date; Aug 2026: 91.9M games / 30.1 GB compressed | CC0 online corpus, position frequencies, rated online play | account names are not automatically FIDE identities; speed mix differs from OTB classical |
| [Lichess broadcast dump](https://database.lichess.org/#broadcasts) | 1.235M PGN games on review date; CC BY-SA 4.0, separate from CC0 standard games | recent event/broadcast candidate corpus | player IDs and metadata vary; attribution/share-alike policy must be implemented |
| [FIDE rating-list downloads](https://ratings.fide.com/download_lists.phtml) | monthly official player/rating records, FIDE ID, title, federation, standard/rapid/blitz | identity anchor and rating timeline | rating-list file is not a bulk PGN archive of those players' games |
| [ChessBase Mega Database 2026](https://shop.chessbase.com/en/products/mega_database_2026) | commercial curated product advertises >11.7M games and >114k annotated games | market benchmark or licensed partnership | a retail license is not a right to put its dataset behind our SaaS |
| [Lichess master explorer](https://github.com/lichess-org/lila-openingexplorer) | public `/masters` opening statistics and game references | comparison/limited API lookup per terms | public query API is not proof of bulk corpus redistribution rights or complete FIDE coverage |
| [Chess.com PubAPI](https://support.chess.com/en/articles/9650547-what-is-the-pubapi-and-how-do-i-use-it) | read-only public player/game archives | conservative requested profile/account lookup | not a bulk dataset grant; parallel access can return 429 |
| [TWIC archive](https://theweekinchess.com/twic) | weekly downloadable PGN and event reporting | partner candidate, research reference | site explicitly says personal use only and all rights reserved |

Numbers are a dated snapshot and must be refreshed before business promises. Lichess broadcast games have a different license from its standard-game dump.

## The professional corpus problem

Professional opponent reports need an eligible OTB game record with:

```text
game score + event/date/round + time control + source + license/policy
+ white/black identity candidates + ideally FIDE IDs + provenance confidence
```

FIDE ratings alone cannot reconstruct moves. An imported PGN with `WhiteFideId` and `BlackFideId` is strong identity evidence; names, titles, federation, rating, event, and date are weaker signals. Store uncertain matches as candidates, not facts. Account-to-FIDE linking needs explicit verification or corroboration and can remain unknown.

Recommended acquisition ladder:

1. Start with Lichess broadcasts as a license-separated, source-attributed OTB/broadcast corpus. Audit which records actually represent rated OTB games and which contain FIDE IDs.
2. Sign direct agreements with tournament organizers, arbiters, broadcasters, federations, and PGN/data providers. Request historical files plus incremental round feeds, stable player IDs, correction notices, and display/derived-statistics rights.
3. Add user or coach PGN uploads as private, workspace-scoped material. A user upload does not grant public redistribution rights.
4. Consider licensed professional databases only through a specific SaaS/data contract. Do not mirror retail products or ingest TWIC into a shared commercial corpus under its personal-use terms.
5. Use Lichess standard dumps as a separate online corpus and possibly a verified-account cohort. Never merge online and OTB counts into one unlabeled “professional” statistic.

Source coverage should be displayed as a measured property, e.g. “178 eligible classical OTB games found, 94 since 2023; last source update 2026-09-24.” “All games” should mean all games *within the selected corpus*, not all games ever played by the person.

## Scale arithmetic: facts versus estimates

Published fact: August 2026 Lichess standard PGN archive is 30.1 GB compressed for 91,912,325 games, about 327 compressed bytes per game. The page says uncompressed archives are approximately 7.1 times larger. [Source](https://database.lichess.org/).

Engineering estimates for **one billion** games, to replace with a pilot measurement:

| Component | Calculation | Planning range |
|---|---:|---:|
| compressed raw PGN | ~327 B/game × 1B | ~327 GB, highly source-dependent |
| uncompressed PGN | ~7.1 × compressed | ~2.3 TB |
| complete position occurrences | 60–90 plies/game × 1B | 60–90B rows/entries |
| compact posting payload | 50–150 B/occurrence including indexing overhead, **assumption** | ~3–13.5 TB before replicas, backups, compaction headroom |
| full Stockfish pass at 1 second/occurrence | 80B seconds, **illustrative** | ~2,535 single-core years |

The occurrence-byte range is deliberately broad. It is not a published Lichess storage figure or a PostgreSQL benchmark. A real pilot must measure index bytes, WAL, compaction amplification, compression ratio, rebuild time, hot-key skew, and hardware costs. Lichess's own opening-explorer README describes an SSD/RAM-intensive index, low live import rate, and a historical 2.90 TB RocksDB state on its then-current dataset; that is an architectural signal, not a quote for our future bill. [Source: Lichess explorer README](https://github.com/lichess-org/lila-openingexplorer).

### Why “analyze all games” needs two meanings

- **Index and summarize:** parse each move once, derive positions, openings, results, player and time-control dimensions, quality flags, and aggregate counts. This can be batch processed at corpus scale.
- **Engine evaluate:** run a deep search on a position. Running it on every ply of one billion games is not a first-release plan. Analyze selected critical positions, representative games, new variations, and cached popular positions; deduplicate identical positions first.

## Ingestion and quality pipeline

```text
licensed monthly/event file -> immutable object + checksum
-> streaming parse and legality check
-> canonical game + original PGN link
-> source-aware deduplication
-> FIDE identity candidate resolution
-> position generation / opening classification
-> corpus-specific index and aggregates
-> quality reconciliation -> publish immutable snapshot
```

For Lichess, import the `.pgn.zst` archives in bounded monthly batches and checkpoint at a stable file/record offset. For OTB sources, preserve provenance and corrections per event. Bad PGNs and unresolved identities are quarantined with counts. Reimports are idempotent.

## Milestones for a defensible corpus

| Gate | Corpus ambition | Proof required |
|---|---|---|
| C0 | 100k–1M curated OTB/broadcast games | source rights, FIDE matching precision, opponent report examples |
| C1 | 5–20M curated/licensed OTB games if agreements permit | coverage against a professional test set; correction and freshness pipeline |
| C2 | 100M–1B selected online games | import throughput, position index economics, separate cohort semantics |
| C3 | 1B+ online games and wider OTB portfolio | sharded index, multi-TB storage, proven query and rebuild SLOs |

The 5–20M OTB target is an ambition, not an available open dump. C0 should test at least 50 real opponent-preparation cases across events and federations. A database of millions can still fail the job if the next opponent's recent games are absent.

## Unanswered sourcing questions

- Which event organizers can provide historical PGN and round-by-round updates with SaaS rights?
- How many broadcast PGNs carry reliable FIDE IDs, time controls, and full moves?
- Can a commercial OTB supplier license searchable moves, cards, derived metrics, and evidence links at an affordable price?
- What coverage and freshness do titled players consider credible for Swiss pairing preparation?
- Which online identities can be verified for a FIDE person without guesswork?
