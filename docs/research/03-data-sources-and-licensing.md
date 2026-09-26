# Data sources and licensing

Status: engineering guidance; legal review still required
Reviewed: 2026-09-26

The actionable provider/API link register and integration priority are maintained in [Integration and OSINT source catalog](../integrations/INTEGRATION_OSINT_CATALOG.md).

This document is not legal advice. Access to data, rights to process it, rights to redistribute it, and rights to train models on it are separate questions.

## Data principles

1. Every imported record carries source, retrieval time, original identifier, license/terms classification, and transformation history.
2. Publicly accessible does not mean freely redistributable.
3. User authorization does not automatically grant ChessScope the right to publish or aggregate licensed material.
4. Raw source retention and derived-data retention need separate policies.
5. A corpus version is part of every statistical result.
6. Deletion, correction, and reimport must be possible without corrupting derived evidence.

## Initial source matrix

| Source | Intended use | Known posture | Decision |
|---|---|---|---|
| User PGN | Private import and analysis | Rights depend on content and user | Accept with user attestation and private-by-default handling |
| FIDE rating downloads | Player identity, titles, federation, monthly rating observations | Official machine-readable downloads; downstream use still requires policy/legal review | Preferred identity/rating anchor after review |
| Lichess standard games | Open reference and research corpus | Lichess exports state CC0 | Preferred open foundation |
| Lichess broadcasts | OTB/broadcast research | Listed separately as CC BY-SA 4.0 | Keep license class separate; attribution/share-alike review required |
| Lichess evaluations | Engine-labelled research | Export page states database exports are CC0; verify each dataset note | Candidate for experiments, with snapshot provenance |
| Chess.com PubAPI | User-linked/on-demand imports | Public read-only API; not a blanket CC0 grant | Integrate conservatively; do not mass mirror without review |
| Wikidata / Wikimedia Commons | Identity corroboration and licensed portraits | Open-data/media projects with claim/file-specific provenance and attribution | Candidate enrichment source; never treat identity matches as certain by default |
| 2700Chess / Take Take Take | Live-rating and player-card benchmark | No suitable public production API/license identified in this review | Reference or partnership only; no undocumented scraping |
| The Week in Chess | Event discovery and weekly PGN reference | Archive states personal use only and all rights reserved | Permission/license required for a shared production corpus |
| Commercial databases | User-owned local workflows only until licensed | Proprietary | No public corpus ingestion or redistribution |
| Licensed OTB corpus | Professional reference database | Contract-dependent | Later strategic partnership |
| Coach/user annotations | Private knowledge and collaboration | User/contract-dependent | Explicit ownership, sharing, and deletion rules |

## Lichess

The official Lichess database page states that database exports are released under CC0 and may be used, modified, and redistributed. It also separately states that broadcast games are CC BY-SA 4.0. Those classes must not be collapsed into one generic “Lichess” license.

Engineering requirements:

- store dump filename, checksum, month, dataset category, and retrieval timestamp;
- retain source game ID when available;
- make monthly ingestion idempotent;
- support tombstone/correction workflows;
- track whether clock, evaluation, and rating fields were present;
- avoid implying Lichess ratings are directly comparable with FIDE or Chess.com ratings.

Primary source: [Lichess open database](https://database.lichess.org/).

## Chess.com

The official PubAPI is a read-only API for public player, game, club, and tournament data. Its documentation says serial access is unlimited while parallel access may trigger `429` responses; it also documents caching through `ETag` and `Last-Modified`.

Engineering requirements:

- import only user-requested accounts/archives when the online-account connector is introduced;
- use conditional requests and a descriptive user agent;
- serialize or carefully limit concurrency;
- retain response and archive provenance;
- respect deletion/unavailability signals;
- do not copy Chess.com visual assets or product identifiers;
- obtain legal review before any large-scale corpus or redistribution plan.

Primary source: [Chess.com PubAPI guidance](https://support.chess.com/en/articles/9650547-what-is-the-pubapi-and-how-do-i-use-it).

## User PGN

PGN is a format, not a license. A file can include copyrighted annotations, proprietary curation, personal data, or games sourced from a commercial database.

Default policy:

- imports are private;
- raw files are encrypted at rest in cloud storage;
- user selects whether processing is local, cloud, or hybrid;
- private records are excluded from global statistics unless an explicit, legally reviewed contribution mechanism exists;
- export/delete operations cover raw and derived private artifacts;
- parsing preserves unsupported content where practical and reports semantic loss.

## Commercial databases

Mega Database 2026 is a separately sold proprietary product with more than 11.7 million games and a large annotated subset. ChessScope must not ingest it into a shared service without an appropriate license.

Possible later models:

- local-only indexing of a user's licensed copy, after legal review;
- bring-your-own-data connectors that never upload raw content;
- a direct licensing or distribution partnership;
- licensed metadata while full annotations remain local.

None is currently approved.

## Provenance record

Every source item should be able to answer:

```text
source system and dataset class
source object identifier
retrieved/imported at
original checksum
license/terms policy identifier
import batch and parser version
normalization decisions
canonical object identifiers
visibility and tenant
deletion/correction status
```

## Legal review gates

Legal review is required before:

- redistributing Chess.com-derived game archives;
- offering cloud processing of commercial databases;
- using annotations or books for model training/RAG;
- pooling private user games into product-wide analytics;
- publishing derived datasets that could reconstruct restricted sources;
- changing from private import to team/shared/public visibility;
- choosing a commercial/open-source dual model that affects dependencies and datasets.

## Open questions

- What exact corpus is sufficient for the first professional preparation workflow?
- Can a useful OTB corpus be licensed within the expected budget?
- Which derived aggregates are safe to retain after a source game is deleted?
- How should ChessScope respond to identity corrections across snapshots?
- Does the MIT repository license remain appropriate once product and dependency decisions are made?
