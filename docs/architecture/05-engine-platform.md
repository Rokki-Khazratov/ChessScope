# Engine platform

Status: target architecture; not an MVP implementation plan

## Goals

- one analysis contract for local and cloud execution;
- reproducible results and transparent budgets;
- no orphaned engine processes;
- safe cancellation and resource isolation;
- reuse identical analysis where privacy allows;
- avoid making cloud CPU the default for every position.

## Analysis identity

An engine result is reusable only when the identity matches:

```text
canonical position
engine build and network
engine options
analysis budget type and value
threads/hash constraints where material
MultiPV
tablebase configuration
ruleset
result schema version
```

Time-only budgets can vary across hardware. Node budgets are more reproducible, though engine/version and thread behavior still matter. Store both requested and observed values.

## Execution routes

### Browser/local lightweight

Useful for instant analysis and privacy, potentially through WebAssembly or a local companion. Constraints include device performance, browser lifecycle, battery, memory, and engine packaging licenses.

### Local companion

Supports native UCI engines, private files, and stronger local hardware. It requires signed updates, sandboxing, loopback authentication, process supervision, and cross-platform support.

### Cloud workers

Support consistent hardware pools, deep analysis, batches, and team workflows. They require quotas, queueing, cost controls, multi-tenant isolation, and capacity planning.

## User-visible budgets

Prefer understandable modes backed by explicit profiles:

- **Cached/statistical** — no new engine work;
- **Quick** — low node/time budget for interaction;
- **Standard** — stable default analysis;
- **Deep** — higher budget with cost/time estimate;
- **Research** — queued batch or long-running job.

The UI displays engine identity and achieved budget, not only a depth number.

## Job state machine

```text
created -> queued -> leased -> starting -> running
    -> completed | failed | cancelled | expired
```

Requirements:

- idempotency key and deduplication;
- worker lease and heartbeat;
- start/run deadlines;
- cancellation propagated to the process group;
- bounded stdout/stderr capture;
- graceful `stop` followed by forced termination;
- cleanup verified after every terminal state;
- retry policy that distinguishes infrastructure from deterministic input failure;
- partial results explicitly marked.

## UCI boundary

Treat an engine binary as untrusted code:

- run with least privilege and resource limits;
- isolate filesystem and network where feasible;
- validate option names/values;
- cap memory, processes, threads, and output;
- never build shell commands from user text;
- retain binary checksum and license metadata;
- permit only approved cloud engine builds.

## Score model

Persist:

- side-to-move normalized score;
- centipawn or mate score without conflation;
- WDL when supported/configured;
- principal variations as legal moves;
- nodes, depth, selective depth, time, and NPS;
- engine-reported versus ChessScope-derived fields.

Do not convert centipawns to win probability without a versioned calibration model and context.

## Cost controls

- estimate positions and budget before batch submission;
- enforce per-user/workspace quotas;
- cache public-corpus analysis by full identity;
- never share private-position cache entries across security boundaries;
- prioritize interactive jobs over batch work;
- support pause/resume at job-set boundaries;
- record CPU/GPU time, queue time, and cost allocation.

## Failure scenarios to test

- engine ignores `stop` or never reaches `uciok`;
- binary crashes or emits malformed output;
- user cancels during startup and during deep search;
- worker disappears while holding a lease;
- tablebase/network file is missing;
- local companion disconnects;
- duplicate jobs arrive concurrently;
- a private job is accidentally offered to a shared cache;
- one analysis produces extreme output volume.

## Licensing gate

Stockfish and other engines have their own licenses and distribution obligations. WebAssembly/native bundles, neural networks, and tablebases require explicit dependency and redistribution review before packaging.
