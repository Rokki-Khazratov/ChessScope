# Professional coach: first product release

Status: user-directed product decision and implementation hypothesis
Updated: 2026-09-26

## Decision

“ChessScope Light” means a narrower first release of a professional chess research product. Its reference user is a titled or near-GM over-the-board player preparing for an unfamiliar opponent at a Swiss tournament. FIDE identity and high-quality tournament games are central. Lichess and Chess.com are supplementary data sources only when accounts can be linked with sufficient confidence.

The first release must contain the differentiator: a board-synchronized coaching chat with persistent, named variations and evidence-linked answers. The sequence `landing -> account -> paid entitlement -> dashboard -> research workspace` is the desired commercial journey. Exact pricing and payment provider remain open decisions.

This document supersedes the “AI later” and “online account import first” assumptions in [earlier scope](03-scope-and-requirements.md), [roadmap](../delivery/01-roadmap.md), and [backend phases](../backend/PHASED_IMPLEMENTATION_PLAN.md) for the professional release. Those files remain useful for domain and infrastructure detail; the [professional release slices](../delivery/03-professional-release-slices.md) define the new order.

## Professional user story

A player rated around 2500 FIDE learns the next opponent's name after round three of a large open tournament. They have about ten minutes for the first briefing and limited time until the next round. They search by name or FIDE ID, confirm the correct person, set their own color and likely repertoire, and request a report.

The report answers:

- which eligible over-the-board games are in the corpus and how recent they are;
- the opponent's current official identity, title, federation, rating history, and notable achievements with sources;
- what the opponent has played in the relevant color and time control;
- which choices are frequent, recent, surprising, and well supported by games;
- which candidate preparation lines fit the user's own repertoire;
- which claims are uncertain because games, IDs, or time controls are missing.

The user opens a suggested line on the board, branches it, asks a follow-up, renames variations, changes pieces manually, and resumes the same conversation. Every answer refers to the selected branch and exact position.

## Product surfaces

### Landing and conversion

- State the professional value through an opponent-preparation example with visible source games.
- Explain what corpus is included, freshness, and which claims can be checked.
- Support account creation, hosted checkout, entitlement confirmation, receipts/billing portal, and account cancellation.
- A payment return URL is a navigation event; access changes only after verified provider confirmation and an idempotent entitlement update.
- Dashboard loads without claiming that a new user's FIDE ID has been verified; users may research any FIDE player by ID.

### Dashboard

- Resume recent studies, opponent reports, and analysis boards.
- Search a FIDE player, game, event, or position.
- Show report/job progress, corpus freshness, available engine budget, and saved branches.

### Analysis workspace

Functional benchmark: the board, engine lines, notation, variation navigation, and game list of [Chess.com Analysis](https://www.chess.com/analysis). The supplied screenshot illustrates the information density and board/analysis relationship. ChessScope uses its own artwork, components, sounds, terminology, and interaction design.

Core layout on a desktop screen:

```text
┌──────────────────────────┬───────────────────────────────────────┐
│ board + arrows/highlights│ move tree with named branches         │
│                          │ engine / database evidence controls   │
│                          │ coaching chat and interactive cards   │
└──────────────────────────┴───────────────────────────────────────┘
```

The right area must allow the player to inspect moves while continuing the conversation. Move tree and chat cannot each be an isolated full-screen mode. A selected move always identifies a branch and node; the AI knows both.

### Player profile and search

- Search by FIDE ID, name aliases, federation, title, rating range, event, date, color, result, time control, ECO/opening, and exact position.
- Present a player card with official FIDE observations, verified portraits, source coverage, recent games, opening repertoire, and versioned ChessScope metrics.
- Explain identity confidence. A matching name alone is insufficient for automatically linking a PGN or online account to a FIDE person.
- Statistics default to the relevant OTB cohort; online games appear in a separately named cohort.

## Coaching chat behavior

### Three answer classes

1. **Historical:** “How did Magnus Carlsen play this position as Black?” Resolve the FIDE person, exact board state, color, corpus, and filters. Return matching games, next moves, dates, opponents, and source links. If no exact matches exist, offer separately labeled transpositions or similar structures.
2. **Engine and plan:** “How can I trade knights and reach opposite-colored bishops?” Generate candidate legal lines with the engine and a goal checker. Test reasonable defensive replies. Label a line as an example, plausible plan, or forced only when the search supports that strength of claim.
3. **Explanation:** “How do I exploit the open d-file and defend the c-file?” Combine legal move candidates, engine evaluations, board geometry, concrete threats, and relevant model games. Explain coordinates in the currently selected position and show arrows/highlights with cited lines.

The LLM drafts explanations and chooses typed tools. It does not supply move legality, position counts, FIDE identity, or engine evaluation from memory.

### Interactive output

An answer may contain text, linked source-game chips, an engine line, branch proposal, diagram, arrows/highlights, an assumption, and a follow-up action. The user can:

- preview a candidate line on the board;
- add it as a named variation without rewriting the current main line;
- compare two saved branches;
- jump to the cited node or source game;
- ask “why 12...Nc6 in branch B?” without repeating all moves;
- move a piece manually and ask from the new board state.

The chat must never silently move the study's authoritative line. A user action or explicit approved workspace mode commits a proposed branch. Streaming explanation and board previews can be automatic.

## Ten-minute opponent briefing contract

The report is assembled from indexed, already ingested data. It is an asynchronous job with progressive output:

| Target elapsed time | Visible result |
|---|---|
| < 2 seconds | identity candidates, FIDE card, coverage warning |
| < 15 seconds | recent games and opening distribution from existing indexes |
| < 60 seconds | first evidence-backed summary and candidate lines |
| < 10 minutes | deeper engine checks, representative games, final report |

These are proposed service targets, not measured performance. Missing source data cannot be repaired by waiting ten minutes. The report records corpus snapshot, search filters, sources, generated time, game count, and unresolved identity matches.

## First release acceptance scenarios

1. A user opens a FIDE player by ID, sees official ratings and which OTB games were actually found, and can open every game used by a repertoire claim.
2. On an arbitrary legal position, a user asks for historical play by a named player. The answer distinguishes exact position, transposition, similar structure, and no match.
3. The user creates four named variations of 20–30 plies each, moves between them, closes the browser, returns, and asks about one specific branch. The assistant refers to the correct node and line.
4. A user manually changes the board during a streaming response. The stale response remains attached to the old node and cannot overwrite the new board state.
5. A requested strategic outcome that cannot be demonstrated receives a bounded answer with alternatives; it is not called “forced.”
6. A subscribed user can access the app after the payment event is verified; a duplicated or out-of-order webhook cannot grant two entitlements or revoke a newer one.

## First release boundaries

The product is intended for professional depth, but the first release can index a curated OTB corpus and selected online corpus. Its UI must state coverage. It does not promise every FIDE game in the world, live pairing discovery from every tournament, exhaustive proof of arbitrary plans, or deep engine analysis of every game in a billion-game archive.

## Decisions still requiring product evidence

- paid plan structure and trial policy;
- minimum OTB coverage that professional users will accept;
- whether cloud AI/engine work is included in subscription or metered;
- allowed online-account verification methods;
- publication rights for opponent reports and event metadata;
- which coaching interactions require an explicit branch-add action versus an automatic preview.
