# ADR 0002: Web-first product with a hybrid local boundary

Status: accepted product direction
Date: 2026-09-23

## Context

The product must be easily accessible and collaborative, while serious chess work may involve private preparation, user-owned databases, local UCI engines, offline use, and large files. A pure desktop system increases distribution and synchronization complexity; a pure cloud system creates privacy, compute, and data-rights problems.

## Decision

The browser is the primary application surface and cloud services provide shared search, public/open corpora, synchronization, and queued compute.

A local companion is a planned extension, not an MVP requirement. It will use shared domain contracts to provide:

- explicit local file access;
- native local engine execution;
- local-only indexing/processing for approved workflows;
- offline queue/cache where justified;
- controlled synchronization of selected artifacts.

Private imports remain private by default. Users can see and choose the execution route for sensitive or expensive work.

## Consequences

Positive:

- immediate cross-platform access and simpler updates;
- shareable evidence and research workflows;
- a path for privacy and local compute;
- one primary UX rather than unrelated web and desktop products.

Negative:

- local/cloud synchronization becomes a significant later system;
- browser limitations affect large imports and local compute;
- the companion creates a high-value security boundary;
- some professional users may reject the alpha before local support exists.

## Guardrails

- no hidden upload of local/private positions;
- no unauthenticated localhost API;
- no raw database-file synchronization as the collaboration model;
- explicit per-workflow privacy and compute route;
- user research must test whether deferred local support is acceptable.

## Revisit when

Private-alpha interviews show that the first validated workflow cannot succeed without local engines/files, or browser performance makes the planned corpus workflow infeasible.
