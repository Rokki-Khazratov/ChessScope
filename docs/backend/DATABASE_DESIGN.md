# PostgreSQL database design

Status: physical schema baseline; validate with benchmarks before migration code
Last updated: 2026-09-26

## 1. Principles

- PostgreSQL is the system of record; Redis and search/analytics projections are rebuildable.
- Raw source records, canonical records, and derived records remain distinguishable.
- Source provenance and visibility are never optional metadata.
- Query-critical fields use typed columns; `jsonb` is reserved for provider/raw extension data.
- Mutable user knowledge is versioned; published evidence and engine results are immutable.
- High-volume tables are append-oriented, partitionable, and independently rebuildable.
- Schema changes follow expand -> migrate/backfill -> switch -> contract.

## 2. Namespaces

Logical PostgreSQL schemas are optional operationally, but ownership boundaries are mandatory:

| Namespace | Responsibility |
|---|---|
| `iam` | users, credentials, sessions, workspace membership |
| `source` | providers, policies, imports, raw artifacts, checkpoints |
| `chess` | players, events, games, revisions, lines |
| `position` | canonical positions, occurrences, edges, aggregates |
| `analytics` | cohorts, metrics, results, player snapshots |
| `engine` | builds, profiles, jobs, immutable results |
| `evidence` | bundles, items, claims |
| `study` | studies, chapters, annotations, shares |
| `ops` | jobs, outbox, idempotency, audit, quotas |
| `billing` | provider events, subscriptions, entitlements |

## 3. Identity types and conventions

- External IDs: UUIDv7 generated in the application and stored as PostgreSQL `uuid`.
- Very high-volume internal rows: `bigint GENERATED ALWAYS AS IDENTITY`.
- Provider IDs: normalized `text` plus provider ID, unique together.
- Checksums: `bytea`, with algorithm stored or fixed by the contract.
- Position key: 16-byte `bytea` with `CHECK (octet_length(position_key) = 16)` plus canonical-state collision verification.
- Money/cost: integer minor units or exact `numeric`; never floating point.
- Engine/statistical floats: `double precision` with semantic validation and null for unavailable values.
- Timestamps: `timestamptz`; chess dates also carry a precision enum when day/month is unknown.
- Soft deletion is not a universal default; use explicit tombstone/status only where source correction or retention requires it.

## 4. Core relational model

### IAM and workspaces

```text
iam_user(id, email_normalized, display_name, status, locale, created_at, deleted_at)
iam_external_identity(id, user_id, provider, provider_subject, metadata, verified_at)
workspace(id, owner_user_id, name, plan_code, visibility, quota_policy_id, created_at)
workspace_member(workspace_id, user_id, role, status, joined_at)
share_grant(id, workspace_id, subject_type, subject_id, permission, token_hash, expires_at, revoked_at)
```

Constraints: unique normalized email when active; unique `(provider, provider_subject)`; unique membership pair. Private resource tables include `workspace_id NOT NULL` and policy indexes beginning with it.

### Sources and imports

```text
data_source(id, code, kind, base_url, policy_version_id, status)
source_policy_version(id, source_id, terms_url, license_code, allowed_purposes,
                      attribution_template, retention_days, reviewed_at, decision)
connector_account(id, workspace_id, source_id, external_subject, secret_ref, scopes, status)
connector_checkpoint(id, connector_account_id, stream, cursor, etag, last_modified, synced_at)
raw_artifact(id, workspace_id?, source_id, object_key, sha256, media_type, byte_size,
             visibility, policy_version_id, retrieved_at, retention_until)
import_batch(id, workspace_id?, source_id, artifact_id, parser_version, state,
             requested_by, started_at, finished_at, counters, error_artifact_id)
source_game(id, import_batch_id, provider_game_id, source_offset, raw_record_hash,
            original_tags, parse_state, canonical_game_id?)
```

Uniqueness: `(source_id, provider_game_id)` when provider IDs are stable; otherwise `(import_batch_id, source_offset)`. An artifact checksum may repeat across workspaces but visibility cannot be inferred from checksum deduplication.

### Players, identities, ratings, and media

```text
player(id, canonical_name, normalized_name, fide_id?, birth_year?, identity_state,
       title_code?, federation_code?, activity_state, created_at)
player_alias(id, player_id, source_id, alias, normalized_alias, source_subject_id,
             confidence, valid_from, valid_to)
platform_account(id, player_id?, source_id, handle, provider_subject_id, verification_state)
identity_candidate(id, subject_type, subject_id, candidate_player_id, score,
                   evidence, state, reviewed_by?, reviewed_at?)
rating_observation(id, player_id, source_id, rating_system, time_control,
                   effective_date, value, games_count?, rank?, is_active?, import_batch_id)
player_media(id, player_id, kind, asset_url/object_key, source_url, license_code,
             author, attribution_text, valid_from, status)
player_snapshot(id, player_id, corpus_snapshot_id, as_of_date, metric_set_version,
                coverage, generated_at, state)
```

Indexes: unique partial `fide_id` when present; alias lookup on normalized alias; rating lookup `(player_id, rating_system, time_control, effective_date DESC)`; ranking `(rating_system, time_control, effective_date, rank)`.

Birth date is not required for a useful card. Prefer year/precision available from an approved source and record provenance.

### Games and revisions

```text
event(id, canonical_name, normalized_name, start_date?, end_date?, site_id?, source_confidence)
site(id, canonical_name, country_code?, geo_point?)
team(id, canonical_name, federation_code?)
canonical_game(id bigint, public_id uuid, variant, start_fen?, current_revision_id?, created_at)
game_revision(id bigint, game_id, revision_no, white_player_id?, black_player_id?,
              event_id?, played_on?, date_precision, round_text?, result,
              time_control_code?, white_rating?, black_rating?, eco?, ply_count,
              move_blob, tag_projection, visibility, workspace_id?, content_hash,
              supersedes_revision_id?, created_at)
game_source_link(game_id, game_revision_id, source_game_id, relation, confidence)
game_line(id, workspace_id, game_revision_id?, parent_line_id?, parent_ply?, order_key,
          move_blob, comment_doc?, version, created_by)
analysis_workspace(id, workspace_id, owner_user_id, title, corpus_snapshot_id?, version)
variation_node(id, analysis_workspace_id, chapter_id?, parent_node_id?, root_fen?,
               uci_move?, san?, position_key, node_fen, label?, order_key,
               tree_revision, created_by, created_at)
node_annotation(id, variation_node_id, body, arrows, highlighted_squares,
                evidence_bundle_id?, version)
```

`move_blob` is a versioned compact canonical sequence. The application can generate SAN/PGN; a lossless source payload remains in object storage/source records. `canonical_game` is stable identity; `game_revision` captures corrections and user-owned variation content is not merged into public source content by default.

`variation_node` is the durable addressable unit for the coaching chat. A node stores both its move path identity and resulting exact position; transpositions may share a position key but remain distinct nodes. `game_line` can remain a compact import/export representation or be migrated to the node tree after benchmarking and an ADR; do not maintain two independently editable truths.

Indexes are chosen from measured queries: `(white_player_id, played_on DESC)`, `(black_player_id, played_on DESC)`, event/date, ECO/date, and a dedup content-hash index. Avoid separate low-selectivity indexes on every PGN tag.

### Positions and occurrences

```text
position(id bigint, position_key bytea, canonical_state bytea, variant,
         side_to_move, castling_mask, ep_square?, created_at)
position_occurrence(id bigint, position_key bytea, game_revision_id bigint, ply smallint,
                    next_move_code?, mover_player_id?, played_on?, result,
                    white_rating?, black_rating?, time_control_class?, corpus_id bigint)
move_edge(from_position_key, move_code, to_position_key, legality_version)
position_move_aggregate(position_key, corpus_snapshot_id, cohort_bucket_id,
                        move_code, games, white_wins, draws, black_wins,
                        avg_rating?, last_played_on?, generated_at)
structure_fingerprint(position_key, algorithm_version, fingerprint bytea, features)
```

`position` has unique `(variant, position_key, canonical_state)` and a lookup index on `(variant, position_key)`. Hash matches are verified against `canonical_state`.

`position_occurrence` is hash-partitioned by `position_key` (initially 32 partitions; benchmark 16/32/64). Primary lookup index inside partitions begins `(position_key, game_revision_id, ply)`. Only proven filter combinations receive additional covering indexes, for example `(position_key, mover_player_id, played_on DESC) INCLUDE (next_move_code, result)`. Partition count is selected before large loading because repartitioning is an operational event.

At one million 80-ply games, approximately 80 million occurrences are expected; at ten million, approximately 800 million. Both storage and WAL amplification must be measured from real encodings. A full 100-million-game occurrence corpus is not assumed to remain in PostgreSQL.

### Corpus and analytics

```text
corpus(id, code, name, eligibility_policy_version, visibility)
corpus_snapshot(id, corpus_id, version, manifest_object_key, game_count,
                position_count, source_cutoffs, published_at, state)
cohort_definition(id, definition_hash, schema_version, normalized_definition)
metric_definition(id, code, semantic_version, implementation_version,
                  methodology_url, uncertainty_method, status)
metric_result(id, subject_type, subject_id, corpus_snapshot_id, cohort_definition_id,
              metric_definition_id, value_numeric?, value_json?, unit,
              sample_size, interval_low?, interval_high?, generated_at,
              evidence_bundle_id?)
snapshot_metric(player_snapshot_id, metric_result_id, display_order)
```

Unique metric identity includes subject, snapshot, cohort, and metric definition/version. JSON values must have a registered schema; searchable scalar metrics also use typed numeric columns.

### Engine, evidence, studies, and operations

```text
engine_build(id, name, version, binary_sha256, network_sha256?, license_code,
             architecture, provenance_url, enabled)
analysis_profile(id, code, version, budget_type, budget_value, threads, hash_mb,
                 multipv, syzygy_version?, options_hash, options)
analysis_job(id, job_id, workspace_id, position_key/game_revision_id, build_id,
             profile_id, privacy_route, cache_key, state)
analysis_result(id, position_key, build_id, profile_id, result_hash, score_type,
                score_value, wdl, depth, seldepth, nodes, time_ms, pv_lines,
                hardware_class, completed_at, workspace_scope?)
evidence_bundle(id, public_id, workspace_id?, manifest_hash, corpus_snapshot_id?,
                schema_version, object_key?, created_at)
evidence_item(bundle_id, ordinal, item_type, object_type, object_id, object_version,
              excerpt_locator?, content_hash)
claim(id, bundle_id, claim_type, text, confidence, verification_state)
study(id, workspace_id, title, current_version, created_by, visibility)
chapter(id, study_id, order_key, title, root_fen, current_version)
job(id, public_id, workspace_id?, type, schema_version, state, priority,
    idempotency_key?, input_hash, progress_current, progress_total?, stage,
    lease_owner?, lease_expires_at?, attempt_count, trace_id, created_at, finished_at?)
outbox_event(id bigint, aggregate_type, aggregate_id, event_type, schema_version,
             payload, created_at, published_at?, attempts)
idempotency_record(scope, key_hash, request_hash, response_ref, expires_at)
audit_event(id bigint, occurred_at, actor_user_id?, workspace_id?, event_type,
            resource_type?, resource_id?, reason?, ip_hash?, metadata)
chat_thread(id, workspace_id, analysis_workspace_id, title, created_by, created_at)
chat_message(id, thread_id, author_kind, body, active_node_id, tree_revision,
             model_id?, status, created_at)
coach_tool_result(id, message_id, tool_name, input_hash, evidence_bundle_id?,
                  result_object_key?, status, cost_units)
coach_action(id, message_id, parent_node_id, expected_tree_revision,
             action_type, proposed_moves, status, accepted_by?, accepted_at?)
opponent_report(id, workspace_id, subject_player_id, requester_id,
                corpus_snapshot_id, filter_hash, job_id, evidence_bundle_id?,
                generated_at?, status)
billing_customer(id, workspace_id, provider, provider_customer_id, created_at)
billing_subscription(id, billing_customer_id, provider_subscription_id,
                     plan_code, provider_status, current_period_end?, version, updated_at)
billing_event(id, provider, provider_event_id, event_type, payload_object_key,
              received_at, processed_at?, processing_state)
entitlement(id, workspace_id, code, status, starts_at, ends_at?, source_subscription_id?,
            version, updated_at)
```

Analysis result uniqueness is the complete reproducibility/cache key. Private results are never reused across workspaces unless the policy explicitly proves the inputs are public and the cache scope is safe.

Constraints: unique `(provider, provider_event_id)` on billing events; unique provider subscription ID; entitlement updates are monotonic/versioned under a workspace lock; `coach_action` requires the expected tree revision and its parent node in the same workspace; chat messages retain the historical node ID even after navigation; billing payloads are private, retained under provider policy, and never included in AI context.

## 5. Partition and retention policy

| Table | Strategy | Retention |
|---|---|---|
| `position_occurrence` | hash by position key | follows corpus/source eligibility; rebuildable |
| `rating_observation` | range by effective year after scale threshold | durable approved history |
| `audit_event` | monthly range partitions | policy-dependent, immutable |
| `outbox_event` | monthly range or archived after publish | compact after reconciliation window |
| `job` | range by creation month after scale threshold | summary retained; verbose logs expire |
| `raw_artifact` | object lifecycle, metadata in DB | source/workspace policy |

Dropping an expired partition is preferred to mass row deletes when the legal/data model permits it. Source-linked removal still produces tombstones/audit data and rebuild instructions.

## 6. Query projections

Use explicit aggregate/projection tables for:

- opening explorer by corpus/cohort bucket;
- player card current ratings, rank, form, and metric set;
- recent games by player/event;
- import/job dashboards;
- evidence manifest summaries.

PostgreSQL materialized views are acceptable for coarse refreshes. Incremental, high-frequency projections use ordinary tables updated from outbox events so freshness and rebuild state are visible.

## 7. Integrity rules

- Source record cannot exist without an import batch and policy version.
- Private canonical revision cannot exist without a workspace.
- Corpus membership excludes data whose policy/visibility is ineligible.
- Metric result cannot exist without metric and corpus/cohort versions.
- Evidence item references immutable object version/content hash.
- Current revision pointers are updated transactionally and cannot point across incompatible visibility scopes.
- Job terminal state requires result or structured error metadata.
- Rating source/system/time-control values are explicit; no cross-pool overwrite.
- Player merges are reversible through an audited merge map; hard identity collapse is prohibited without review.

## 8. Database security

- Separate application, migration, read-only analytics, worker, and backup roles.
- Application role cannot alter schema or bypass RLS.
- Connector secrets are references to an external secret store, not plaintext columns.
- RLS covers workspace studies, shares, private games, imports, and evidence as defense in depth.
- Administrative queries use an audited elevated path.
- Backups are encrypted and restore tests verify tenant and evidence integrity.

## 9. Migration and backfill rules

1. Add nullable/new structure without breaking old application instances.
2. Deploy dual-read/write or compatibility code where required.
3. Backfill in resumable, rate-limited chunks with progress and validation.
4. Switch reads after reconciliation counts/checksums pass.
5. Enforce constraints using low-lock techniques where possible.
6. Remove old structure in a later release.

No migration may perform an unbounded table rewrite during ordinary production deployment. High-volume index creation uses concurrent/partition-aware procedures and rehearsal against production-like cardinality.

## 10. Benchmark acceptance gates

Before Phase 3 position loading, record:

- encoded bytes per game and per occurrence, including indexes;
- insert positions/second and WAL generated;
- exact-position P50/P95/P99 at common and rare positions;
- filtered position query latency by player/date/rating/time control;
- aggregate rebuild and corpus deletion time;
- vacuum/analyze behavior and replica lag;
- API workload impact during import/indexing.

If the accepted SLO cannot be met economically, update the storage ADR and preserve the logical contracts while moving the measured workload.
