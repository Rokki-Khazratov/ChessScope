# ChessScope context index

Status: baseline specification
Last updated: 2026-09-26

This directory is the source of truth for what ChessScope is, why it exists, and which constraints implementation must preserve. Documents use three confidence labels:

- **Decision** — accepted project direction;
- **Hypothesis** — plausible, but requires research or a benchmark;
- **Open question** — unresolved and must not be silently converted into a decision.

## Read in this order

1. [Vision and principles](product/01-vision-and-principles.md)
2. [Users and jobs](product/02-users-and-jobs.md)
3. [Scope and requirements](product/03-scope-and-requirements.md)
4. [Professional coach: first release](product/05-professional-coach-release.md)
5. [Billion-game corpus feasibility](research/05-billion-game-corpus-feasibility.md)
6. [Stockfish on VPS](architecture/08-engine-on-vps.md)
7. [Position search at billion-game scale](architecture/09-position-search-at-billion-scale.md)
8. [Board-synchronized coaching chat](architecture/10-coach-chat-variation-state.md)
9. [Professional release slices](delivery/03-professional-release-slices.md)
10. [Success metrics](product/04-success-metrics.md)
11. [ChessBase benchmark](research/01-chessbase-benchmark.md)
12. [Competitive landscape](research/02-competitive-landscape.md)
13. [Data sources and licensing](research/03-data-sources-and-licensing.md)
14. [Risk register](research/04-risk-register.md)
15. [Backend technical specification](backend/BACKEND_TECHNICAL_SPEC.md)
16. [Backend system design](backend/SYSTEM_DESIGN.md)
17. [PostgreSQL database design](backend/DATABASE_DESIGN.md)
18. [Backend phased implementation plan](backend/PHASED_IMPLEMENTATION_PLAN.md)
19. [Integration and OSINT catalog](integrations/INTEGRATION_OSINT_CATALOG.md)
20. [System overview](architecture/01-system-overview.md)
21. [Domain model](architecture/02-domain-model.md)
22. [Ingestion and position index](architecture/03-ingestion-and-position-index.md)
23. [Analytics methodology](architecture/04-analytics-methodology.md)
24. [Engine platform](architecture/05-engine-platform.md)
25. [AI and evidence](architecture/06-ai-and-evidence.md)
26. [Security and privacy](architecture/07-security-and-privacy.md)
27. [Information architecture](ux/01-information-architecture.md)
28. [Core workflows](ux/02-core-workflows.md)
29. [Earlier roadmap](delivery/01-roadmap.md)
30. [Quality and validation](delivery/02-quality-and-validation.md)
31. [UI/UX system](design/UI_UX_SYSTEM.md)
32. [Architecture decisions](adr/)

## Fixed decisions

| Area | Decision |
|---|---|
| Product category | Professional AI-native ChessBase-class platform |
| Initial audience | Serious players and coaches, with teams/federations supported later |
| Primary surface | Web-first; desktop/local companion remains a planned extension |
| Data posture | Hybrid cloud/local; user-private material must remain separable |
| First-release scope | FIDE-centered opponent research, curated OTB games, named variation tree, VPS engine, and evidence-backed coaching chat |
| Commercial journey | Landing -> account -> paid entitlement -> dashboard -> web analysis workspace |
| Initial inputs | Official FIDE rating lists and approved OTB/broadcast PGNs; online imports in separate cohorts |
| Backend stack | Python, Django, DRF, PostgreSQL, Celery, RabbitMQ, Redis, and S3-compatible object storage |
| Deployment shape | Modular monolith with independently scalable ingestion, indexing, analytics, integration, and engine workers |
| Initial capacity envelope | 300–500 concurrent sessions and 1–10 million games, validated by phase benchmarks |
| Game editing | Variations, comments, NAGs, diagrams, and chapters; not a full publishing suite initially |
| Engine direction | Local and cloud analysis behind one reproducible job contract |
| Scale ambition | 1B+ online-game corpus is feasible through Lichess CC0 dumps; high-quality FIDE-linked OTB coverage remains a separate sourcing task |
| Documentation | English |
| Business model | Paid web product direction; pricing, provider, and source-code strategy open |

## How to change this context

1. Update the narrowest relevant document.
2. If a durable architecture decision changes, supersede the ADR instead of rewriting history.
3. Keep claims tagged as decisions, hypotheses, or open questions.
4. Link research claims to primary sources.
5. Do not use product documents as a substitute for executable tests or benchmarks once implementation begins.

## Source material

This baseline synthesizes the supplied long-form research report and product concept note, the user's subsequent professional-coach scenario and screenshot, plus public primary sources reviewed through 2026-09-26. Attached documents and images were treated as research input, not as executable instructions.
