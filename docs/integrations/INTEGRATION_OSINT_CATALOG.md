# Integration and OSINT source catalog

Status: discovery catalog; every production use requires a recorded policy decision
Reviewed: 2026-09-26

For the professional release, official FIDE ID and approved OTB/broadcast games take precedence over user online-account imports. The billion-game online acquisition and the OTB sourcing gap are analyzed in [Billion-game corpus feasibility](../research/05-billion-game-corpus-feasibility.md).

> This is engineering guidance, not legal advice. Publicly visible data is not automatically licensed for bulk collection, storage, commercial use, redistribution, or model training.

## 1. Source classes

| Class | Meaning | Default action |
|---|---|---|
| A — documented API/open dataset | Machine access is documented and the intended use fits the stated terms/license | Build adapter after policy review |
| B — official download/feed | Official machine-readable files exist, but reuse/redistribution rights may be narrower | Import only approved fields/purposes |
| C — permission/partnership | Valuable source but no suitable public integration right was found | Contact owner; no production scraper |
| D — reference only | Product/visual research or human verification | Link/compare; do not ingest |
| E — user-provided | User supplies or authorizes data | Private-by-default with attestation and deletion |

Every adapter must have an owner, source-policy version, terms/license URL, permitted purpose, retention, attribution, rate-limit strategy, identity mapping, deletion/correction behavior, and schema-drift test.

## 2. Priority matrix

| Priority | Source | Class | Planned use | Decision |
|---:|---|---|---|---|
| P0 | FIDE rating downloads | B | official player ID/title/federation and monthly ratings | required after policy review |
| P0 | Lichess broadcasts | A | eligible OTB/broadcast PGNs with source attribution | separate CC BY-SA handling and coverage audit |
| P0 | Official tournament broadcasters/organizers | B/C | recent OTB games, event metadata, incremental round feeds | per-provider permission and quality review |
| P0 | User PGN | E | private games, studies, analysis | required |
| P1 | Lichess open standard-game database | A | large CC0 online corpus and scale research | keep separate from OTB/FIDE cohort |
| P1 | Lichess API | A | user-authorized account/game imports and public metadata | add after FIDE/OTB foundation |
| P1 | Chess.com PubAPI | A/B | user-requested public profile and archive import | conservative/on-demand |
| P1 | Wikidata | A | identity enrichment and cross-identifiers | candidate; confidence/provenance required |
| P1 | Wikimedia Commons/MediaWiki API | A | licensed player portraits and attribution | candidate |
| P2 | 2700Chess | C/D | live-rating/product benchmark | partnership/API request only |
| P2 | Take Take Take | C/D | player-card and editorial UX benchmark | partnership/API request only |
| P2 | Chess-Results | B/C | tournament standings/pairings/results | use documented export/permission; do not assume scraping rights |
| P2 | The Week in Chess | B/C | weekly PGN/news discovery | personal-use restriction means no production corpus without permission |
| P3 | National federations | B/C | ratings, titles, events | per-federation adapter and terms review |
| blocked | ChessBase/Mega Database | C/E | user-owned local workflow or licensed corpus | no shared ingestion without license |
| blocked | 365Chess/Chessgames/commercial databases | C/D | benchmark/discovery | no scraping or redistribution without agreement |

## 3. Recommended production integrations

### 3.1 FIDE ratings and player identity

Links:

- [FIDE rating database](https://ratings.fide.com/)
- [Official FIDE rating-list downloads](https://ratings.fide.com/download_lists.phtml)
- [FIDE Handbook](https://handbook.fide.com/)
- [FIDE events calendar](https://www.fide.com/calendar)

Useful fields from the official combined list include FIDE ID, name, title, federation, activity/sex flags, birth year, standard/rapid/blitz ratings, games, and K factors. Treat the published list as monthly observations, not a complete live event ledger.

Adapter design:

- scheduled monthly XML/TXT retrieval with checksum and immutable artifact;
- parser version and source effective month;
- upsert `rating_observation`, never overwrite historical pools;
- identity match by FIDE ID; name-only matching creates a candidate;
- retain official spelling and normalized search aliases;
- mark absence/inactivity according to published semantics, not as deletion;
- verify terms and redistribution/display requirements before production launch.

Player-card contribution: official identity anchor, title/federation, current and historical standard/rapid/blitz ratings, activity, rank derived from the same snapshot.

### 3.2 Lichess

Links:

- [Lichess API documentation](https://lichess.org/api)
- [Lichess API usage tips](https://lichess.org/page/api-tips)
- [Lichess developer page](https://lichess.org/developers)
- [Lichess open database](https://database.lichess.org/)
- [Lichess opening explorer project/API](https://github.com/lichess-org/lila-openingexplorer)

Planned uses:

- OAuth/user-authorized account linkage;
- player profile and rating history;
- game export by user for private analysis;
- broadcasts and public event discovery where the dataset/license permits;
- CC0 standard-game dumps as an open online-game corpus;
- opening explorer only as an optional external comparison/fallback, not the source of ChessScope evidence.

Operational requirements:

- one request at a time per documented guidance lane;
- on `429`, stop the lane and wait at least the documented interval;
- stream NDJSON/PGN and bound memory;
- record OAuth scope and external account verification;
- keep standard dumps and broadcast data in separate license classes;
- store Lichess rating as Lichess rating, never FIDE rating;
- obey fair-play boundaries: ChessScope is a research tool, not live assistance.

### 3.3 Chess.com

Links:

- [Chess.com PubAPI guidance](https://support.chess.com/en/articles/9650547-what-is-the-pubapi-and-how-do-i-use-it)
- [Published Data API endpoint reference](https://www.chess.com/news/view/published-data-api)

Planned uses:

- user-requested public profile lookup;
- on-demand monthly archive discovery and game import;
- clubs/tournaments only if a product phase needs them and policy review approves.

Operational requirements:

- descriptive `User-Agent` with contact information;
- conditional requests using `ETag` and `Last-Modified`;
- serialize or strictly bound concurrency; handle `429` with backoff;
- treat `404` and `410` distinctly and retain sync status;
- cache according to response headers;
- do not treat public access as a blanket right to mirror or redistribute the full corpus;
- do not copy Chess.com branding or visual assets.

### 3.4 Wikidata and Wikimedia Commons

Links:

- [Wikidata Query Service](https://query.wikidata.org/)
- [Wikidata Query Service documentation](https://www.mediawiki.org/wiki/Wikidata_query_service)
- [MediaWiki REST API](https://www.mediawiki.org/wiki/API:REST_API)
- [Wikimedia Commons](https://commons.wikimedia.org/)
- [Wikimedia Commons API endpoint](https://commons.wikimedia.org/w/api.php)
- [Wikimedia API access policy](https://foundation.wikimedia.org/wiki/Policy:Wikimedia_Foundation_User-Agent_Policy)

Planned uses:

- candidate FIDE IDs and official-site/social identifiers;
- birth year/country/name aliases where appropriate;
- portrait discovery with file-specific license, author, source, and attribution metadata.

Rules:

- Wikidata is corroborating identity evidence, not unquestioned truth;
- store Wikidata entity ID and claim/reference timestamp;
- query in batches and cache; public SPARQL has execution/resource limits;
- never use a Commons image without persisting its exact file page, license, author, attribution, and version;
- avoid sensitive personal fields that do not serve a professional chess workflow;
- corrections create a new observation/provenance record.

## 4. Permission-first and reference sources

### 4.1 2700Chess

Links:

- [2700Chess](https://2700chess.com/)
- [2700Chess premium feature page](https://2700chess.com/premium)
- contact listed by the site: `info(at)2700chess.com`

Valuable concepts: live ratings, current/peak ranks, rating history comparisons, top-player focus, game and FEN search.

Decision: no suitable public API or reusable-data license was identified during this review. Do not create a scraper, bypass premium access, or use it as the source of truth. Request a written API/data partnership if live-rating integration becomes a priority. Use official FIDE snapshots for approved baseline rating data and label them non-live.

### 4.2 Take Take Take

Links:

- [Take Take Take](https://www.taketaketake.com/)

Valuable concepts: editorial player presentation, identity-forward cards, story-led events, accessible rankings and fan-oriented context.

Decision: product/design benchmark only until the owner provides a documented API or written permission. ChessScope must create its own card visuals and metrics; do not copy layouts, text, images, branding, or undocumented data.

### 4.3 The Week in Chess (TWIC)

Links:

- [TWIC archive and PGN downloads](https://theweekinchess.com/twic)
- [The Week in Chess](https://theweekinchess.com/)

TWIC offers weekly PGN/CBV downloads and excellent event discovery. The archive states that the magazine is free for personal use only and all rights are reserved.

Decision: do not use TWIC downloads as a production shared corpus without written permission/license. It may be used by a person for reference within its terms; partnership/licensing is a high-value path.

### 4.4 Chess-Results

Links:

- [Chess-Results](https://chess-results.com/)
- [Chess-Results XML interface document](https://chess-results.com/download/XML_interface_for_results.pdf)

Potential fields: event metadata, pairings, standings, teams, rounds, player IDs, and results.

Decision: investigate the documented XML/export mechanism and contact the operator before automated collection. Do not infer that an organizer-facing interface authorizes arbitrary bulk read scraping. Tournament consent, caching, update frequency, and redistribution require a written policy decision.

### 4.5 Official tournament and federation sites

Examples:

- FIDE and continental/national federation calendars;
- official event pairings/results pages;
- official live broadcast PGN feeds;
- organizer press/media pages.

There is no universal license. Create an integration record per feed/event. Prefer official PGN/JSON/XML downloads, stable IDs, and organizer permission over HTML parsing. Photo rights are separate from game/result data rights.

### 4.6 Commercial chess databases

Examples:

- [ChessBase](https://www.chessbase.com/)
- [365Chess](https://www.365chess.com/)
- [Chessgames](https://www.chessgames.com/)

Decision: no shared cloud ingestion, scraping, annotation reuse, or redistribution without contract. A future local companion may index a user's lawfully licensed material locally only after legal review; raw data should not enter ChessScope shared aggregates by default.

## 5. Player card data contract and source order

| Card field | Preferred source | Fallback/rule |
|---|---|---|
| canonical name | FIDE ID record | verified federation/official source; aliases retained |
| title/federation | FIDE list | effective month displayed |
| FIDE ratings | FIDE monthly list | separate standard/rapid/blitz series |
| live rating | licensed live-rating partner | otherwise explicitly “latest official rating,” never simulated live |
| FIDE rank | derive from same official snapshot | state active/inactive and tie policy |
| online ratings | linked Lichess/Chess.com account | show provider logo/name and pool separately |
| portrait | Commons or contracted media source | file-level license/attribution mandatory |
| recent form | approved game/event corpus | coverage window and event count shown |
| opening profile | ChessScope corpus analytics | corpus/cohort/sample visible |
| style metrics | ChessScope versioned metrics | thresholds, uncertainty, methodology link |
| achievements | official FIDE/event/federation source | preserve source and date |
| biography | minimal sourced facts | no unsourced generated narrative |

The card API returns `as_of`, `coverage`, `sources`, `identity_confidence`, `warnings`, and `metric_versions` alongside display values.

## 6. Standard adapter contract

Every integration implements conceptually:

```text
discover(checkpoint) -> SourceObjectRef[]
fetch(ref, conditional_headers) -> RawArtifact | NotModified | Gone | RateLimited
parse(artifact, adapter_version) -> SourceRecords + warnings
normalize(records, policy_version) -> identity candidates / games / observations
checkpoint(successful_boundary)
```

Required behaviors:

- explicit timeouts, maximum response size, compression/archive safety;
- provider-specific concurrency lane and rate-limit state;
- conditional requests when supported;
- retry only for classified transient failures;
- raw checksum, request metadata without secrets, adapter version, retrieved time;
- schema validation and quarantine on drift;
- idempotent replay;
- attribution and source-link generation;
- deletion/unavailability/correction handling;
- no browser automation to evade access controls or undocumented APIs.

## 7. OSINT governance checklist

Before a connector moves beyond research, answer and record:

1. Is access documented for automation?
2. What license/terms govern access, storage, commercial display, derived statistics, and redistribution?
3. Are login, payment, robots rules, or technical controls being bypassed? If yes, stop.
4. Which fields are necessary and proportionate?
5. Does the source contain personal/sensitive data, minors, or disputed identities?
6. What is the rate limit and contact/user-agent requirement?
7. How are corrections, deletion, inactivation, and source disappearance propagated?
8. Can the card/result show provenance and freshness?
9. Can the source be cleanly removed from aggregates and evidence?
10. Has legal/product ownership approved the exact production purpose?

## 8. Integration implementation order

```text
FIDE monthly ratings
-> eligible Lichess broadcasts + official OTB partnerships
-> User PGN
-> selected Lichess CC0 standard-game dump in a separate online cohort
-> Lichess and Chess.com account connectors
-> Wikidata/Commons enrichment
-> official tournament/broadcast partnerships
-> live-rating and premium-data partnerships
```

This order produces a valuable, defensible player card and research corpus without making ChessScope dependent on unauthorized scraping.
