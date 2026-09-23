# Security and privacy

Status: baseline threat model; formal review required before implementation

## Protected assets

- private games, annotations, repertoires, and preparation targets;
- player/team membership and sharing relationships;
- OAuth/API credentials for external chess platforms;
- local filesystem and engine access through a companion;
- licensed or restricted datasets;
- engine and AI compute budgets;
- evidence integrity and audit history;
- user identity and billing data if introduced.

## Trust boundaries

```text
browser <-> application API
application <-> database/object storage/queue
workers <-> untrusted PGN and engine binaries
cloud <-> local companion
ChessScope <-> Lichess/Chess.com/data providers
ChessScope <-> AI providers
workspace <-> workspace/tenant
```

## Data classification

| Class | Examples | Default |
|---|---|---|
| Public | Open corpus games and published metadata | Searchable within license/policy |
| User-private | Personal PGN, notes, saved preparation | Owner only |
| Workspace-private | Team studies and shared analysis | Explicit role-based access |
| Restricted-source | Commercial/licensed datasets | Contract and tenant constrained |
| Secret | Tokens, encryption keys, local companion credentials | Dedicated secret storage; never logged |

## Primary threats and controls

### Cross-tenant access

- authorization on every object and derived query;
- tenant-aware database policies or equivalent defense in depth;
- evidence bundles cannot reference objects the viewer cannot access;
- automated isolation tests with adversarial IDs and search filters.

### Malicious imports

- treat PGN/comments/tags as untrusted text;
- limit file size, nesting depth, token length, and decompression ratio;
- streaming parsing and isolated workers;
- malware/content scanning for uploaded containers where applicable;
- no active HTML/script rendering from comments.

### Local companion compromise

- signed binaries and updates;
- narrow authenticated protocol with origin checks;
- no unauthenticated localhost control surface;
- explicit grants for files, folders, engines, and cloud synchronization;
- process and filesystem sandboxing where platforms permit;
- revocable device identity.

### Engine execution

- approved binaries in cloud environments;
- isolated process/user/container boundary;
- CPU, memory, process, time, output, and filesystem limits;
- termination of complete process trees;
- no arbitrary shell arguments.

### External identity and API tokens

- OAuth where supported;
- encrypted token storage and least scopes;
- CSRF/state/PKCE protections as applicable;
- token revocation and connection audit;
- never expose tokens to analytics or AI providers.

### AI and retrieval

- permission filtering before retrieval/tool execution;
- imported text cannot redefine system/tool policy;
- output encoding and link allowlisting;
- evidence IDs are server-issued and authorized;
- model/provider route selected by data classification;
- configurable retention and provider no-training terms.

## Privacy model

- private by default for user imports and studies;
- explicit visibility transition: private -> workspace -> shared link/public;
- raw and derived data deletion paths;
- export of user-owned content and machine-readable metadata;
- disclose which work runs locally versus in cloud;
- no global metric contribution without explicit approved policy;
- saved AI traces follow the same access rules as their source evidence.

## Hybrid synchronization

Synchronization should operate on versioned domain objects, not raw database files. Requirements:

- end-to-end authenticated transport;
- conflict detection rather than last-write-wins for chess variations/notes;
- resumable encrypted transfer;
- local queue visibility and cancellation;
- tombstone propagation;
- per-object sync policy;
- no upload of raw restricted data for local-only workflows.

## Audit events

Record at minimum:

- login, connection, and device changes;
- import/export/delete operations;
- sharing and permission changes;
- access to restricted-source data;
- cloud engine and AI submissions containing private data;
- administrative actions;
- security-relevant local companion events.

Audit logs must avoid storing raw secrets, full private PGNs, or unnecessary prompt content.

## Security gates

Before public alpha:

- threat-model review;
- dependency/license inventory;
- tenant-isolation tests;
- upload parser fuzzing and resource-limit tests;
- engine sandbox/cancellation tests;
- secret scanning and secure deployment baseline;
- backup/restore and deletion verification.

Before team or local companion release:

- external security review;
- protocol and update-chain review;
- role/permission matrix tests;
- synchronization conflict and leak tests;
- incident response and audit access process.
