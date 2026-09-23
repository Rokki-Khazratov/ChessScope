# Domain model

Status: conceptual model; physical baseline is defined separately

The PostgreSQL tables, keys, indexes, partition strategy, retention, and migration rules are defined in [PostgreSQL database design](../backend/DATABASE_DESIGN.md). This document remains the storage-independent domain vocabulary.

## Design rules

- separate source records from normalized entities;
- make uncertain identity explicit;
- version mutable chess artifacts;
- keep immutable evidence references;
- distinguish a chess position from its occurrences in games;
- distinguish statistical facts, engine results, and interpretations.

## Core entities

### Source and corpus

**DataSource** describes an origin such as a user upload, Lichess dump, Chess.com archive, or licensed feed.

**ImportBatch** records retrieval, checksums, parser version, policy/license class, visibility, and completion state.

**CorpusSnapshot** names a stable set of eligible game revisions used for statistics. Results never refer only to “the database.”

### Game

**SourceGame** is the source-faithful record, including original tags and raw payload reference.

**CanonicalGame** is the normalized chess game used for search and analysis.

**GameRevision** captures corrections to moves, tags, identities, comments, or visibility without rewriting historical evidence.

**GameLine** represents the main line and nested analysis variations for editable user material. Public reference games and private user annotations may have different revisions and ownership.

### People and events

**Player** is a canonical person or platform account identity.

**PlayerAlias** maps source spellings/account names to a player with provenance and confidence.

**IdentityCandidate** records unresolved or competing matches. Low-confidence matches must not be silently merged.

**Event**, **Site**, **Round**, and **Team** retain source-specific metadata while allowing canonical grouping.

**RatingObservation** stores system, value, date, and source; ratings from different pools are never implicitly interchangeable.

### Position graph

**Position** is canonical chess state: pieces, side to move, castling rights, and relevant en-passant state. Move counters may be stored for interchange but do not normally define repetition-equivalent identity.

**PositionKey** is a stable 128-bit hash or equivalent key plus a collision-verification representation.

**PositionOccurrence** connects position, game revision, ply, next move, mover, and result context.

**MoveEdge** is a legal transition from one canonical position to another. It may be materialized or derived.

**StructureFingerprint** is a versioned derived representation for pawn/material/piece similarity and is not the same as exact position identity.

### Analytical entities

**CohortDefinition** specifies corpus, players, date window, rating system/bucket, time control, color, and other filters.

**MetricDefinition** contains semantic version, methodology, inputs, assumptions, and uncertainty method.

**MetricResult** contains value, units, confidence/credible interval, sample size, cohort, generated time, and evidence bundle.

**PlayerSnapshot** is a materialized, time-bounded set of metric results, not a permanent label on a person.

### Engine entities

**EngineBuild** identifies engine name, version/commit, binary provenance, license, network file, architecture, and supported options.

**AnalysisProfile** names a reproducible budget: nodes or time, threads, hash, MultiPV, tablebases, and option set.

**AnalysisJob** contains requester, position/game targets, privacy route, budget, state, lease, and cancellation state.

**AnalysisResult** stores score/WDL, PVs, depth, nodes, time, and the exact engine/profile identity.

### Evidence and claims

**EvidenceBundle** is an immutable manifest of query, corpus snapshot, filters, source IDs, metric/engine results, and generation time.

**Claim** has text, type, evidence references, confidence, and verification state.

Claim types:

- database fact;
- statistic;
- engine result;
- interpretation;
- recommendation.

Only the first three can normally be machine-verified for exact correspondence.

### User knowledge

**Workspace** is a private or shared boundary.

**Study** contains ordered **Chapters**, saved game lines, positions, notes, diagrams, and evidence links.

**SavedQuery** stores normalized search input plus corpus policy.

**ShareGrant** records subject, resource, permissions, expiry, and revocation.

## Key relationships

```text
DataSource -> ImportBatch -> SourceGame -> CanonicalGame -> GameRevision
Player <- PlayerAlias -> SourceGame
GameRevision -> PositionOccurrence -> Position -> MoveEdge
CorpusSnapshot -> CohortDefinition -> MetricResult -> EvidenceBundle
Position -> AnalysisJob -> AnalysisResult -> EvidenceBundle
Workspace -> Study -> Chapter -> GameLine / Position / EvidenceBundle
EvidenceBundle -> Claim
```

## Identity and deduplication

Game deduplication should use multiple signals:

- normalized legal move sequence;
- player candidates;
- date/event/round;
- result and starting position;
- source identifiers;
- annotation and variation differences.

Identical moves do not imply identical source records. The model may cluster duplicate game records while preserving every source and annotation set.

## Versioning rules

- raw source objects are immutable;
- canonical entities receive explicit revisions after correction;
- derived indexes identify the source revision range or snapshot;
- saved evidence does not silently rebind to newer data;
- UI may offer “refresh with latest corpus” as a new result;
- metric semantic changes require a new metric version.

## Open modeling questions

- whether `MoveEdge` should be stored globally or aggregated per position/corpus;
- how much source annotation can be structurally merged without rights ambiguity;
- how to model games with unknown dates and inconsistent time controls;
- whether player accounts and real people should be separate first-class entities;
- how to express partial visibility when one evidence bundle includes multiple workspaces.
