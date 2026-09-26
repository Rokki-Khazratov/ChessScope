# ChessScope

ChessScope is a web-first, AI-native chess intelligence platform for serious players, coaches, teams, and researchers. It connects game databases, position search, player analytics, engine analysis, and verifiable natural-language investigation in one workspace.

> Ask questions. Explore games. Understand chess.

## Current phase

The repository is in the **product-definition and architecture phase**. It intentionally contains no application code yet. The immediate goal is to remove the expensive unknowns around data rights, position indexing, statistical validity, engine compute, and evidence-grounded AI before implementation begins.

## Confirmed direction

- professional ChessBase-class ambition, designed from an AI-native starting point;
- web-first product, with a desktop/local companion considered after the web foundation;
- hybrid data model: cloud services plus private/local user data and compute;
- first paid professional release centered on FIDE player research, an OTB game corpus, board-synchronized coaching chat, and reproducible engine analysis;
- approved OTB/broadcast PGN and official FIDE ratings first; online game imports remain a separate cohort;
- local and cloud engine execution as the long-term engine model;
- English as the repository and product-specification language;
- landing -> account -> payment -> dashboard -> analysis workspace as the target commercial journey; source-code strategy remains open.

## Product boundaries

ChessScope is not an online chess server, generic puzzle site, or chatbot wrapped around Stockfish. Its core is a position-centric evidence system:

```text
Games and positions
        -> deterministic search and statistics
        -> reproducible engine analysis
        -> evidence bundles
        -> human-readable explanations
```

Every important result should be inspectable down to source games, filters, sample size, metric version, position identity, and engine budget.

## Documentation map

Start with [docs/README.md](docs/README.md). The context is organized into:

- `docs/product/` — vision, users, requirements, boundaries, and success criteria;
- `docs/research/` — ChessBase benchmark, competitors, data rights, and risks;
- `docs/architecture/` — system, data, analytics, engines, AI, and security;
- `docs/backend/` — executable backend specification, system design, PostgreSQL schema, and implementation phases;
- `docs/integrations/` — approved, permission-first, and reference-only API/OSINT sources;
- `docs/design/` — canonical UI/UX direction, design tokens, page blueprints, and interaction rules;
- `docs/ux/` — information architecture and core workflows;
- `docs/delivery/` — roadmap and validation strategy;
- `docs/adr/` — durable architectural decisions and unresolved decisions.

## Decision rule

The project follows this order:

```text
DATA -> INDEX -> STATISTICS -> EVIDENCE -> TOOLS -> AI -> POLISH
```

The current execution sequence is [professional release slices](docs/delivery/03-professional-release-slices.md). The [backend phased plan](docs/backend/PHASED_IMPLEMENTATION_PLAN.md) supplies component-level acceptance detail. The initial corpus, position index, and VPS engine capacity need benchmark evidence before scale claims are made.

## License

The repository is currently licensed under the [MIT License](LICENSE). This does not grant rights to third-party chess databases, annotations, branding, or engine binaries. Dataset and dependency licensing must be evaluated separately.
