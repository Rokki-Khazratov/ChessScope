# ADR 0003: Evidence-first AI over deterministic tools

Status: accepted architectural principle
Date: 2026-09-23

## Context

Natural language is valuable for multi-stage chess research, but language models can invent games, players, counts, moves, or historical claims. Exact position search and statistics are structured computation problems, not semantic retrieval problems.

## Decision

ChessScope's future AI layer will orchestrate typed, permission-scoped deterministic tools and explain their outputs. It will not be the source of truth for chess data, metrics, legal moves, or engine evaluations.

Material factual claims require evidence references. A claim-verification layer checks numeric and entity correspondence before rendering claims as facts. Interpretations remain visually distinct.

RAG is limited to appropriate unstructured material such as user notes, licensed text, annotations, concepts, and documentation. Exact structured questions use database/analytics tools.

The first database-and-analytics MVP does not depend on an LLM.

## Consequences

Positive:

- factual answers can be inspected and reproduced;
- AI provider changes do not redefine core chess semantics;
- deterministic workflows survive AI outages;
- permissions and corpus boundaries remain enforceable server-side;
- the evaluation target is measurable.

Negative:

- more product and backend work than a direct chatbot;
- evidence schemas and stable identifiers are required early;
- claim verification cannot prove all interpretations;
- answers may be more conservative when evidence is incomplete.

## Guardrails

- no unrestricted production SQL tool;
- no factual numeric claim without evidence ID;
- no external model receives private data without an allowed route;
- imported text is untrusted and cannot redefine tool policy;
- empty and insufficient results remain valid outcomes;
- provider/model/tool versions are part of the trace.

## Rejected alternatives

- **LLM-only chess expert:** unacceptable factual reliability.
- **RAG over all games:** poor fit for exact counts, filters, and position identity.
- **Train a custom foundation model first:** high cost without resolving data, tools, or verification.
- **Add evidence after the chatbot:** would force a later redesign of every result contract.
