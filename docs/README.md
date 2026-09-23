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
9. [Backend technical specification](backend/BACKEND_TECHNICAL_SPEC.md)
10. [Backend system design](backend/SYSTEM_DESIGN.md)
11. [PostgreSQL database design](backend/DATABASE_DESIGN.md)
12. [Backend phased implementation plan](backend/PHASED_IMPLEMENTATION_PLAN.md)
13. [Integration and OSINT catalog](integrations/INTEGRATION_OSINT_CATALOG.md)
14. [System overview](architecture/01-system-overview.md)
15. [Domain model](architecture/02-domain-model.md)
16. [Ingestion and position index](architecture/03-ingestion-and-position-index.md)
17. [Analytics methodology](architecture/04-analytics-methodology.md)
18. [Engine platform](architecture/05-engine-platform.md)
19. [AI and evidence](architecture/06-ai-and-evidence.md)
20. [Security and privacy](architecture/07-security-and-privacy.md)
21. [Information architecture](ux/01-information-architecture.md)
22. [Core workflows](ux/02-core-workflows.md)
23. [Roadmap](delivery/01-roadmap.md)
24. [Quality and validation](delivery/02-quality-and-validation.md)
25. [UI/UX system](design/UI_UX_SYSTEM.md)
26. [Architecture decisions](adr/)

## Fixed decisions

| Area | Decision |
|---|---|
| Product category | Professional AI-native ChessBase-class platform |
| Initial audience | Serious players and coaches, with teams/federations supported later |
| Primary surface | Web-first; desktop/local companion remains a planned extension |
| Data posture | Hybrid cloud/local; user-private material must remain separable |
| First-release scope | Database exploration and advanced player analytics |
| Initial inputs | PGN plus Lichess and Chess.com user imports |
| Backend stack | Python, Django, DRF, PostgreSQL, Celery, RabbitMQ, Redis, and S3-compatible object storage |
| Deployment shape | Modular monolith with independently scalable ingestion, indexing, analytics, integration, and engine workers |
| Initial capacity envelope | 300–500 concurrent sessions and 1–10 million games, validated by phase benchmarks |
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
