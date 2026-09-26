# Information architecture

Status: product-structure hypothesis

## UX objective

Build the easiest serious chess analysis environment to use. “Easy” means users can enter through their task and progressively reveal depth; it does not mean hiding professional capabilities permanently.

## Primary navigation

Keep the stable top level small:

1. **Home** — recent work, imports, saved investigations, and active jobs.
2. **Explore** — universal search across players, games, positions, openings, events, and saved queries.
3. **Workspace** — board, notation, references, analysis, comments, chapters, and evidence.
4. **Players** — profiles, comparisons, trends, and preparation.
5. **Library** — imported corpora, studies, chapters, and data-source status.

The first paid professional workspace has a persistent right-side coaching chat alongside the board, notation, and engine/database evidence. Investigation is also reachable from player, position, and game contexts. The exact behavior is in [Professional coach release](../product/05-professional-coach-release.md) and [variation-aware chat](../architecture/10-coach-chat-variation-state.md).

## Universal object model

Every major screen should support moving among:

```text
Player <-> Games <-> Positions <-> Moves/Openings
   \           \        |          /
    \           -> Evidence <- Statistics
     \                         /
      --------> Studies <-----
```

Users should never have to remember which database window owns a fact.

## Core views

### Explore

One input accepts player names/FIDE IDs, game metadata, FEN/PGN, opening names, and natural-language questions. Type detection remains visible and correctable.

Results are grouped by object type and show corpus/visibility. Advanced filters expand without replacing the simple entry point.

### Player

Sections:

- identity and data coverage;
- recent activity and time range;
- repertoire by color;
- trends and change points;
- opponent-adjusted performance;
- representative games;
- saved preparation and comparisons.

Every chart opens the filtered games behind it. Style descriptions are hypotheses, not immutable personality labels.

### Position

Sections:

- board and move history/transpositions;
- continuation tree;
- W/D/L and performance by selected cohort;
- top, recent, and representative games;
- local/cloud engine panel;
- notes and chapter links;
- exact versus similar position mode.

The selected corpus and filters remain visible.

### Game workspace

Default professional desktop layout:

```text
+----------------------+--------------------------+
|        board         | notation / named branches|
| arrows / preview     | engine + source evidence |
|                      | coaching chat             |
+----------------------+--------------------------+
```

Panels are contextual and rearrangeable later. The initial product should avoid making layout configuration a prerequisite.

### Study

A study organizes chapters containing annotated game lines, positions, prose, diagrams, and evidence references. It is the durable output of exploration and preparation.

### Evidence drawer

Opens from any statistic, engine evaluation, or later AI claim. It shows:

- normalized claim/query;
- corpus snapshot and visibility;
- filters and sample size;
- metric/engine method;
- warnings and uncertainty;
- source games and export link;
- refresh-as-new-result action.

## Progressive disclosure

### Level 1: answer the immediate task

Board, result, top moves, key filters, representative games.

### Level 2: inspect

Full continuation tree, cohort controls, uncertainty, engine lines, annotations.

### Level 3: research

Saved queries, versioned evidence, batch jobs, method details, exports, and comparisons.

## Interaction principles

- keyboard-first board and notation navigation;
- command palette for object creation and context actions;
- URLs represent shareable view state without leaking secrets;
- every background job exposes progress, cancellation, and outcome;
- filters use human-readable summaries and can be reset individually;
- “no data” differs from “not loaded,” “not permitted,” and “insufficient sample”;
- local versus cloud processing is visible before submission;
- destructive edits and share changes require clear scope.

## Visual semantics

- facts, engine results, and AI interpretations use distinct treatments;
- W/D/L colors are accessible and never the only encoding;
- exact position, transposition, and similarity matches are labeled differently;
- uncertainty is shown through intervals/status, not false decimal precision;
- private/workspace/public data classes are consistently marked.

## Responsive strategy

Web-first does not mean every professional panel fits a phone. Mobile priorities:

- open shared evidence and games;
- review a chapter;
- inspect a player summary;
- annotate lightly;
- monitor a job.

Dense multi-panel research remains optimized for desktop-class screens until mobile research is validated.

## Accessibility

- WCAG 2.2 AA target;
- full keyboard paths for moves, variations, search, filters, and tabs;
- focus follows chess navigation predictably;
- board squares and arrows have textual equivalents;
- notation is screen-reader navigable;
- reduced-motion and high-contrast behavior;
- no essential meaning encoded only by color or hover.
