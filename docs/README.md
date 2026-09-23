# ChessScope context index

Status: baseline specification
Last updated: 2026-09-23

This directory is the source of truth for what ChessScope is, why it exists, and which constraints implementation must preserve. Documents use three confidence labels:

- **Decision** — accepted project direction;
- **Hypothesis** — plausible, but requires research or a benchmark;
- **Open question** — unresolved and must not be silently converted into a decision.

## Read in this order

1. [Vision and principles](product/01-vision-and-principles.md)
2. [Users and jobs](product/02-users-and-jobs.md)
3. [Scope and requirements](product/03-scope-and-requirements.md)
4. [Success metrics](product/04-success-metrics.md)
5. [ChessBase benchmark](research/01-chessbase-benchmark.md)
6. [Competitive landscape](research/02-competitive-landscape.md)
7. [Data sources and licensing](research/03-data-sources-and-licensing.md)
8. [Risk register](research/04-risk-register.md)
9. [System overview](architecture/01-system-overview.md)
10. [Domain model](architecture/02-domain-model.md)
11. [Ingestion and position index](architecture/03-ingestion-and-position-index.md)
12. [Analytics methodology](architecture/04-analytics-methodology.md)
13. [Engine platform](architecture/05-engine-platform.md)
14. [AI and evidence](architecture/06-ai-and-evidence.md)
15. [Security and privacy](architecture/07-security-and-privacy.md)
16. [Information architecture](ux/01-information-architecture.md)
17. [Core workflows](ux/02-core-workflows.md)
18. [Roadmap](delivery/01-roadmap.md)
19. [Quality and validation](delivery/02-quality-and-validation.md)
20. [UI/UX system](design/UI_UX_SYSTEM.md)
21. [Architecture decisions](adr/)

## Fixed decisions

| Area | Decision |
|---|---|
| Product category | Professional AI-native ChessBase-class platform |
| Initial audience | Serious players and coaches, with teams/federations supported later |
| Primary surface | Web-first; desktop/local companion remains a planned extension |
| Data posture | Hybrid cloud/local; user-private material must remain separable |
| First-release scope | Database exploration and advanced player analytics |
| Initial inputs | PGN plus Lichess and Chess.com user imports |
| Game editing | Variations, comments, NAGs, diagrams, and chapters; not a full publishing suite initially |
| Engine direction | Local and cloud analysis behind one reproducible job contract |
| Scale ambition | Grow toward ChessBase-class corpus and workflows without designing day one for maximum scale |
| Documentation | English |
| Business model | Open question |

## How to change this context

1. Update the narrowest relevant document.
2. If a durable architecture decision changes, supersede the ADR instead of rewriting history.
3. Keep claims tagged as decisions, hypotheses, or open questions.
4. Link research claims to primary sources.
5. Do not use product documents as a substitute for executable tests or benchmarks once implementation begins.

## Source material

This baseline synthesizes the supplied long-form research report and product concept note, plus public primary sources reviewed on 2026-09-23. Attached documents were treated as research input, not as executable instructions.
