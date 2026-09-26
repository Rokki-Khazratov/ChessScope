# ChessScope UI/UX System

Status: canonical design direction; no production UI code

Audience: product design, frontend, backend, AI/evaluation, and future contributors

Last reviewed: 2026-09-23

## 1. Design thesis

ChessScope is a professional chess-intelligence workstation whose interface makes evidence feel closer than complexity.

Its central loop is:

```text
FIND -> FILTER -> COMPARE -> MEASURE -> ANALYZE -> EXPLAIN -> VERIFY
```

The interface should feel calm at first contact and exceptionally capable under pressure. It must support a player preparing on a 13-inch laptop, a coach comparing repertoires on a large monitor, and an analyst validating a statistical claim without turning every screen into a control room.

The visual character is **analytical editorialism**: quiet workspace surfaces, confident typography, compact data where the task demands it, and a visible chain from conclusion to evidence. ChessScope should look like a research instrument, not a game, a generic SaaS dashboard, or a chatbot.

### Signature element: the Evidence Rail

ChessScope's single distinctive visual idea is an Evidence Rail. Claims, metrics, engine evaluations, and saved conclusions can reveal a restrained vertical or horizontal rail that connects:

```text
claim -> sample -> filters -> method -> source games
```

Its indentation borrows from chess variations and its structure borrows from a research ledger. It is not decorative. It teaches the user that every conclusion has a path back to the underlying chess data. On narrow surfaces it collapses into evidence chips and a drawer.

### Design tension

ChessScope must hold two modes without becoming two products:

- **reading mode:** spacious, narrative, immediately understandable;
- **working mode:** compact, keyboard-driven, multi-pane, and information dense.

Density changes by surface and user preference; brand identity does not.

## 2. Product UX principles

### 2.1 Evidence before decoration

Visual emphasis follows epistemic importance. A verified sample count is more important than an ornamental illustration. Data provenance must be reachable from the claim without turning the default page into a methodology report.

### 2.2 Progressive disclosure in three depths

Every serious surface supports three coherent layers:

1. **Answer:** the immediately useful conclusion, position, game, or next action.
2. **Inspect:** filters, supporting games, trends, variations, and engine lines.
3. **Audit:** corpus snapshot, exact query, confidence, methodology, metric version, and engine provenance.

The layers are consistent across Player, Position, Opening, Engine, and Investigation experiences.

### 2.3 Position-centric continuity

Moving from player to opening to game to position must preserve context. Back navigation must restore filters, selected row, board position, and scroll state. Users should never feel that opening a source game destroyed their research session.

### 2.4 Deterministic core, conversational edge

Search, filters, data, metrics, legal moves, and engine results remain usable without AI. Natural language accelerates the same system; it never replaces or obscures it.

### 2.5 Density belongs to the task

Player summaries breathe. Game tables are efficient. Opening trees and engine lines are compact. Methodology drawers are dense but structured. One universal card spacing would be a design failure.

### 2.6 State is explicit

Loading, stale, incomplete, private, cached, locally computed, cloud computed, insufficient, and unverified are different states. The UI names them precisely.

### 2.7 Keyboard parity

Any repetitive expert workflow must be achievable by keyboard. Pointer interactions may enhance the experience but cannot be the only route to moves, rows, tabs, filters, or evidence.

### 2.8 Quiet confidence

Primary actions are rare. Motion explains spatial change. Color carries meaning sparingly. Empty space creates hierarchy, not spectacle.

## 3. Reference analysis

References are sources of principles, not visual templates.

| Reference | Borrow | Do not borrow |
|---|---|---|
| Apple | ruthless hierarchy, typography, whitespace, clear primary action, disciplined motion | marketing-scale whitespace or product spectacle inside analytical workspaces |
| Notion | stable sidebar, lightweight surfaces, object organization, low-chrome editing | treating every chess object as a document block or hiding structure behind slash commands |
| OpenAI / ChatGPT | approachable input, conversational clarity, understated controls, progressive response rendering | making chat the application shell or reducing research to a message stream |
| Hermes Agent | typographic confidence, decisive visual hierarchy, technical directness, one memorable gesture | electric-blue identity, all-caps everywhere, mythic imagery, or exact branding |
| ChessBase | professional database depth, reference workflows, saved layouts, keyboard utility | ribbon/menu sprawl, persistent panel overload, Windows-era chrome, hidden discoverability |
| Lichess | board usability, functional restraint, speed, keyboard behavior | analysis-board conventions when they conflict with research provenance |
| En Croissant | modern cross-platform chess workspace and resizable panes | carrying one pane layout across unrelated modes or exposing desktop complexity too early |
| OpeningTree | direct PGN-to-tree comprehension | narrow product assumptions at platform level |
| ChessMonitor / Aimchess | player-centered analytics and longitudinal framing | opaque scores or detached metric cards |
| DecodeChess / ChessAgine | explanation and tool-oriented AI workflows | unsupported prose or generic chatbot presentation |

Design research reviewed public Apple, Notion, Hermes, ChessBase, Lichess, and En Croissant surfaces in September 2026. The product spec must be revalidated against hands-on usability tests before visual implementation is considered final.

## 4. Information architecture

### 4.1 Global navigation

Use five permanent destinations:

- **Home** — resume work and start a search;
- **Explore** — games, positions, players, and openings through one query surface;
- **Workspace** — currently open games, positions, and analysis tabs;
- **Library** — imports, databases/corpora, studies, and saved material;
- **Players** — recent players, comparisons, and preparation snapshots.

Investigations are not a permanent top-level destination in MVP. A saved investigation appears in Home, Library, and contextual history. If research later shows investigations becoming a primary durable object, promote them without adding another permanent tab prematurely.

Professional-release amendment (2026-09-26): the board workspace includes a persistent coaching conversation tied to the selected variation node. The chat is contextual, so it does not require a sixth global destination. [Release scope](../product/05-professional-coach-release.md) and [variation-state contract](../architecture/10-coach-chat-variation-state.md) govern this interaction.

### 4.2 Contextual navigation

Object pages use a local tab row:

- Player: Overview, Repertoire, Trends, Games;
- Position: Continuations, Games, Similar, Analysis;
- Library item: Contents, Details, Imports/History;
- Study: Chapters, Search, Activity when collaboration exists.

Tabs represent stable peer views. Temporary tools do not become tabs.

### 4.3 Panels, drawers, dialogs, and pages

| Container | Appropriate use |
|---|---|
| Page | A durable object or broad research mode with a URL |
| Workspace pane | Simultaneous manipulation: board, notation, reference, engine |
| Right context panel | Persistent coaching chat in the professional analysis workspace; contextual inspection of filters, engine details, or metadata elsewhere |
| Drawer | Evidence details, query method, import report, or secondary inspection |
| Modal | Short blocking decision, destructive confirmation, entity disambiguation |
| Popover | Small selection or explanation anchored to a control |
| Command palette | Navigation and actions known by name; never the only path to essential functionality |

### 4.4 Object continuity

Every Player, Game, Position, Opening, Study, Corpus, Query, and Evidence Bundle has a stable URL or shareable identifier where permissions allow. Context such as active corpus and filters is encoded safely without exposing private source data.

## 5. Application shell

### 5.1 Recommended shell

Use a collapsible left navigation rail and a flexible central canvas. The professional Game Workspace keeps the coaching panel visible on desktop, alongside a readable move tree and engine/database controls. Other screens may use an optional context panel. A minimal contextual toolbar sits above the canvas.

```text
+------------------------------------------------------------------+
| object breadcrumb / title        query state        actions       |
+----------+--------------------------------------+----------------+
| global   |                                      | optional       |
| nav      |            primary canvas            | context panel  |
|          |                                      |                |
|          |                                      |                |
+----------+--------------------------------------+----------------+
```

Why this shell:

- the left rail provides durable orientation;
- the center can be reading-width or full-canvas by page type;
- the right panel supports inspection without navigation loss;
- only one optional context surface prevents panel proliferation.

### 5.2 Dimensions and behavior

- Expanded sidebar: approximately 220–248 px; collapsed rail: 52–64 px.
- Context panel: 320–440 px, resizable within limits; closes to restore canvas.
- Reading pages: centered content with a practical max width around 1120–1280 px.
- Player analytics: max-width layout until a chart/table benefits from expansion.
- Workspace, Explore results, and comparison: full available canvas.
- Minimum supported desktop research width: 1024 px; 1280 px is the comfortable baseline.
- Large monitors do not merely stretch prose. They add comparison columns, wider tables, or an open Evidence Rail.

### 5.3 Layout modes

- **Focus:** central canvas only; sidebar collapsed; context closed.
- **Research:** sidebar plus canvas plus one context panel.
- **Compare:** two synchronized central columns; context panel usually closed.
- **Workspace:** board plus notation/reference panes with saved task-specific sizing.

Pane sizes persist by workspace mode, not globally. Annotation, engine analysis, and repertoire work require different proportions.

## 6. Navigation

The sidebar uses icon plus label when expanded and tooltips when collapsed. It contains only primary destinations, recent/favorite work below a divider, and account/settings at the bottom.

Navigation rules:

- one active destination at a time;
- no nested tree deeper than one expandable level in the global sidebar;
- recent items are capped and searchable;
- active engine/import jobs appear as a quiet status area, not a navigation destination;
- breadcrumbs describe object hierarchy, not browser history;
- open research objects appear as workspace tabs only when multitasking is genuinely useful.

## 7. Search and command palette

Search is a flagship component and the fastest way into the product.

### 7.1 Entry points

- prominent search field on Home;
- persistent compact trigger in the sidebar/header;
- `Cmd/Ctrl + K` opens universal search and commands;
- contextual “Search this position/player/corpus” actions preserve scope.

### 7.2 Query recognition

The input recognizes:

- player names and aliases;
- events and tournaments;
- openings and ECO codes;
- FEN;
- PGN fragments;
- game IDs;
- natural-language questions when the feature is enabled.

Recognition is visible as a resolved token, never hidden inference:

```text
[Player: Abdusattorov] [Color: Black] [Since: 2024] Catalan
```

Tokens are editable. Ambiguity opens an inline resolver with identity, federation/account, rating context, and source coverage.

### 7.3 Results

Default grouping:

1. exact entity matches;
2. likely actions or recognized query interpretation;
3. recent/saved items;
4. grouped Players, Games, Positions/Openings, Studies;
5. commands.

Arrow keys move selection; Right Arrow previews; Enter opens; modifier+Enter opens in a workspace tab. Results show only enough metadata to disambiguate.

### 7.4 Failure and history

Invalid FEN identifies the invalid field. Unknown player suggests aliases without silently selecting one. No result keeps the parsed filters visible and offers to relax one filter. Search history is private, removable, and separated from saved queries.

## 8. Page hierarchy and Home

Home is a launch surface, not a KPI dashboard.

Order:

1. large universal search/input;
2. Resume: up to three recent investigations, studies, or workspace objects;
3. active work: imports and engine jobs only when present;
4. recent players/games/queries in a restrained list;
5. first-run import/connect action when the library is empty.

Home should fit its meaningful content near one viewport on a laptop. It has no generic “games analyzed” cards unless such a number directly helps the next action.

## 9. Explore

Explore is one query model with result modes, not four unrelated applications.

Use a mode switch for **Games / Positions / Players / Openings** after a query is interpreted. A query can return grouped “All” results before a mode is chosen. Mode changes preserve compatible filters and visibly remove incompatible ones.

### 9.1 Filter hierarchy

- Always visible: query summary, corpus, active filter chips, result count.
- Basic filter bar: player, color, date, time control, result, opening.
- Advanced drawer: rating systems/ranges, events, annotations, exact source, data-quality constraints.
- Audit: normalized query and corpus snapshot in Evidence/Query details.

### 9.2 Results

- Games use a compact table/list with a miniature move/position preview on focus.
- Positions use board thumbnails only when they materially help; avoid grids of tiny unreadable boards.
- Players use identity rows with repertoire/trend preview.
- Openings use move-path rows and sample/performance microvisualizations.

Sorting, pagination/virtualization, column configuration, and saved queries are available without dominating first use. A shareable query always declares its corpus and permission limits.

## 10. Game workspace

### 10.1 Default structure

```text
+----------------------+-----------------------------------------+
|        BOARD         | game header / metadata                  |
| arrows / previews    | named move tree and nested variations   |
|                      | engine lines / database references      |
|                      | coaching chat + source-game cards       |
+----------------------+-----------------------------------------+
```

The user can save proportions per mode. At narrower widths, the chat and reference controls use a deliberate switcher while the active branch remains visible. Do not expose arbitrary docking in the first release.

### 10.2 Board behavior

- board size follows the smaller of available height and board-column width;
- preserve a minimum useful notation width before enlarging the board;
- coordinates can be hidden but default on for research;
- last move uses a low-saturation two-square treatment;
- selected square and legal destinations are distinct;
- arrows never cover essential piece identity;
- orientation is persistent per context and explicit in the toolbar;
- captured pieces are optional metadata, off by default in serious analysis.

### 10.3 Notation

- main line reads as a continuous sequence;
- variations indent one clear level at a time and can collapse;
- current move has a strong shape/weight state, not color alone;
- NAG symbols have tooltip/plain-language equivalents;
- comments use normal prose typography and do not visually merge with moves;
- deep variations switch to an outline/tree view on demand rather than wrapping endlessly.

### 10.4 Keyboard

Left/Right navigate moves; Up/Down can select variations when notation has focus; Enter enters a selected variation; Escape returns one level. Exact final shortcuts remain subject to conflict testing.

## 11. Position workspace

Position research is a signature surface. The default layout shows board, continuation summary, sample count, result distribution, and representative games before advanced controls.

### 11.1 Match types

Use three named modes with persistent labels and distinct icons/patterns:

- **Exact position:** identical canonical state; square-outline icon.
- **Similar position:** versioned similarity model; offset-overlap icon.
- **Pawn structure:** pawn-only/structure fingerprint; file-grid icon.

Never communicate match type by color alone. Every result header repeats the active match definition. Similarity results show the model/version and the dimensions that differ.

### 11.2 Layout

Board remains anchored while Continuations, Games, Similar, and Analysis switch in the research region. Filters open in the context panel. Evidence opens in a drawer or rail without replacing the position.

### 11.3 Position identity

FEN, position key, transposition path, and canonical rules details belong in Inspect/Audit layers. The default shows a human-readable opening/structure label only when confidently resolved.

## 12. Opening Explorer

Opening Explorer is a keyboard-navigable tree with analytical rows, not a spreadsheet by default.

Each row contains:

```text
move | games | frequency bar | W/D/L bar | score/performance | trend
```

Sample size remains visible. Direct labels replace legends. Expanding a move advances the board and query together. Back navigation restores the tree branch.

### 12.1 Default versus advanced

Default shows 5–8 leading moves, total sample, corpus/date summary, result bar, and representative games. “Show all” enters a denser table with columns for rating, performance, recency, and confidence.

### 12.2 Filters

Rating, time control, era, player, color, and source are summarized in one sentence above the tree. Advanced filter controls live in the context panel. Changing a filter keeps the board position and visibly refreshes counts.

### 12.3 Trends

Use tiny sparklines or a short textual delta such as “+8 pp since 2024.” Avoid up/down arrows without baseline and interval context. A trend badge opens the relevant time-window comparison.

## 13. Player Intelligence

Player pages are editorial profiles backed by evidence, not dashboards of independent cards.

### 13.1 Hierarchy

1. identity, rating context, country/accounts, and data coverage;
2. one-sentence evidence-backed summary with date/corpus;
3. repertoire distribution by color;
4. recent change timeline;
5. opponent-adjusted performance and uncertainty;
6. representative games;
7. deeper sections through local tabs.

### 13.2 Metric presentation

Metrics appear in narrative groups and tables with microvisualizations. A number must answer: compared with what, from how many games, and during which period?

Good:

> Najdorf score is **+4.2 percentage points above expected** · 147 games · 95% interval …

Bad:

> Opening strength 87.3

Repertoire entropy is explained as predictability/variety in context; it is not displayed as a gamified score ring.

### 13.3 Data coverage

Coverage appears near the header: sources, time span, time controls, missing ratings/dates, and last refresh. It is not hidden in settings because it changes how the profile should be read.

## 14. Player Comparison

Comparison is symmetrical without declaring a winner.

```text
+----------------------+----------------------+
| Player A context     | Player B context     |
+----------------------+----------------------+
| shared filters / corpus / sample coverage   |
+----------------------+----------------------+
| repertoire small multiples                  |
| opening frequencies and intervals           |
| performance vs expected                     |
| recent changes                              |
+---------------------------------------------+
| shared and representative games             |
+---------------------------------------------+
```

Use a common axis and consistent ordering. Never mirror charts in ways that reverse reading direction. Differences are shown as deltas with samples and intervals. Filter changes apply to both players unless explicitly unlocked and labeled.

## 15. Analytics visualization

### 15.1 Philosophy

- direct labels before legends;
- position and move names near data;
- uncertainty when the metric supports it;
- sample sizes adjacent to conclusions;
- restrained gridlines;
- no 3D, radial gauges, or decorative area fills;
- charts link to source games.

### 15.2 Choosing the form

| Need | Preferred form |
|---|---|
| exact values, many dimensions | table |
| compare a few categories | horizontal bar or dot plot |
| change over time | line/step chart with interval and event markers |
| W/D/L composition | accessible stacked bar plus labels |
| small trend inside a row | sparkline plus textual delta |
| distribution/uncertainty | interval plot, histogram, or box/violin when expert use justifies it |
| opening continuations | analytical tree rows |
| two-player comparison | aligned small multiples |

Charts must survive screenshots and exports with visible titles, units, filters, and source date. Patterns/labels provide color-independent interpretation.

## 16. Evidence UX

Evidence has four depths:

1. inline fact with sample text;
2. compact evidence chips;
3. Evidence Rail/drawer preview;
4. full evidence detail page or exported report.

Example:

```text
Abdusattorov played 4...dxc4 in 31 of 74 matching games.
[31 / 74 games] [Classical] [Black] [2025–2026] [View games] [Method]
```

Chip hierarchy:

- primary evidence count opens source games;
- scope chips summarize filters and are neutral, not colorful tags;
- Method opens query/metric details;
- warnings appear before optional scope chips.

The Evidence Rail shows claim type, verification status, corpus, sample, query, method, and sources in a stable order. Hover previews may summarize but never contain the only accessible path. On touch, click opens the drawer.

Evidence type—source, statistic, engine, interpretation—is communicated by label and icon, with semantic color as reinforcement.

## 17. AI Investigation UX

AI Investigation is a research document with dialogue capability, not a customer-support chat.

In the professional Game Workspace, this dialogue is the right-side coach. Its response can preview a legal line, highlight squares, cite source games, and propose a named branch. The selected move node and saved branch names stay in context as the user navigates; previews never silently rewrite the study.

### 17.1 Three layers

- **Answer:** concise conclusion with charts/tables only when useful.
- **Evidence:** embedded claims, source games, samples, and engine results.
- **Investigation:** resolved entities, filters, method, tool outcomes, and rerun controls.

Raw chain-of-thought or internal agent traces are never shown. The audit layer shows safe structured actions: “Resolved player,” “Queried 74 games,” “Compared two periods.”

### 17.2 Interaction

The question remains visible as the title. Resolved entities and filters appear directly below it. Follow-up input is contextual and supports actions:

- open player/position/game;
- change a filter;
- compare another period/player;
- rerun against latest corpus;
- save to Study;
- export evidence.

Partial verification is visible at the claim level. If AI is unavailable, saved evidence and deterministic results remain readable.

## 18. Engine UX

Default engine panel shows:

- current evaluation/WDL;
- 1–3 best lines;
- engine route and identity summary;
- progress as nodes/time/depth where meaningful;
- Start/Stop and analysis profile.

Advanced settings expand into engine, MultiPV, nodes/time, threads, hash, tablebases, version, and local/cloud route. Changing an identity-defining setting clearly starts a different analysis result.

Engine-positive/negative orientation is always stated from White or side-to-move perspective. Mate scores and WDL are not visually conflated with centipawns. Cached results carry a timestamp and profile; running results never appear complete.

## 19. Studies and Library

Library groups objects by what they are, not by arbitrary folders:

- Imports and corpora;
- Studies;
- Saved queries/investigations;
- Repertoires when implemented.

Tags, search, favorites, and recent activity prevent folder hell. A Study contains ordered chapters that may hold game lines, positions, prose, diagrams, and evidence links. In MVP this architecture is lightweight: create, rename, reorder, search, and add research. Collaboration and activity history are later.

## 20. Typography

### 20.1 Philosophy

Typography provides most of the identity. Use one highly readable variable sans for interface and analytical prose, plus a mono face only for machine-readable chess/data strings. Avoid a decorative display face inside the application; confidence comes from scale, weight, and composition.

Recommended implementation candidates:

- UI/prose: **Geist Sans Variable** or a licensed equivalent after glyph and licensing review;
- mono/data: **Geist Mono** or **IBM Plex Mono**;
- system fallback: `-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`;
- chess symbols: a tested Unicode symbol fallback, not a brand font dependency.

The exact family remains a design-system choice subject to licensing and rendering tests. Roles are fixed even if the family changes.

### 20.2 Type roles

| Role | Size / line height | Weight | Use |
|---|---|---|---|
| Display | 36–48 / 1.05–1.12 | 520–620 | Home search thesis or major empty state only |
| H1 | 28–34 / 1.15 | 600 | page/object title |
| H2 | 22–26 / 1.2 | 600 | major page section |
| H3 | 17–20 / 1.3 | 600 | subsection or panel title |
| Section title | 13–14 / 1.3 | 600 | compact groups; sentence case |
| Body | 15–16 / 1.5 | 400–450 | narrative and explanations |
| Body compact | 13–14 / 1.4 | 400–450 | dense panels and tables |
| Label | 12–13 / 1.25 | 520–600 | controls and field names |
| Caption/metadata | 11–12 / 1.35 | 400–500 | provenance, timestamps, secondary facts |
| Button/input | 13–15 / 1.2 | 520–600 | actions and fields |
| Code/FEN | 12–14 / 1.4 | 400–500 mono | FEN, hashes, engine options |
| Chess notation | 14–16 / 1.55 | 450; current 650 | moves and variations |
| Numeric/data | context size | 500–650, tabular | metrics, counts, ratings |

Use tabular figures for tables, clocks, ratings, percentages, and engine numbers. Avoid uppercase metadata except very short technical markers such as ECO; sentence case is the default.

## 21. Color token architecture

Final brand colors are intentionally unresolved. Implementation must use semantic tokens so the founder's palette can be introduced without rewriting components.

### 21.1 Direction

- Neutral direction: quiet, slightly cool-to-neutral rather than sepia, blue-black, or pure gray.
- Saturation: low across structural UI; concentrated only in accent, status, chess board, and charts.
- Accent count: one primary brand accent plus semantic colors; no “AI gradient.”
- Contrast: WCAG 2.2 AA minimum, with AAA where practical for body text and critical data.
- Light and dark modes are separate mappings of the same semantics, not inverted hex values.

### 21.2 Core semantic tokens

| Token | Light mapping | Dark mapping | Purpose |
|---|---|---|---|
| `color.bg.primary` | `TBD_NEUTRAL_LIGHT_0` | `TBD_NEUTRAL_DARK_0` | application background |
| `color.bg.secondary` | `TBD_NEUTRAL_LIGHT_1` | `TBD_NEUTRAL_DARK_1` | sidebars and quiet regions |
| `color.bg.elevated` | `TBD_NEUTRAL_LIGHT_ELEVATED` | `TBD_NEUTRAL_DARK_ELEVATED` | popovers/dialogs |
| `color.surface.default` | semantic neutral surface | semantic dark surface | controls and panels |
| `color.surface.hover` | relative contrast step | relative contrast step | hover feedback |
| `color.surface.selected` | `accent.muted` | `accent.muted` | selected row/tab |
| `color.text.primary` | highest readable neutral | high but non-glowing neutral | primary content |
| `color.text.secondary` | medium neutral | medium-light neutral | supporting content |
| `color.text.tertiary` | lower contrast AA where required | lower contrast AA where required | metadata |
| `color.text.disabled` | disabled neutral | disabled neutral | unavailable controls |
| `color.border.subtle` | near-background contrast | near-background contrast | large-region division |
| `color.border.default` | control boundary | control boundary | inputs, tables |
| `color.border.strong` | active structural boundary | active structural boundary | focus-adjacent emphasis |
| `color.accent.primary` | `TBD_BRAND_HEX` | `TBD_BRAND_DARKMODE_HEX` | selected navigation and primary action |
| `color.accent.hover` | derived accessible accent | derived accessible accent | hover/pressed |
| `color.accent.muted` | low-chroma accent surface | low-chroma accent surface | selection background |
| `color.status.success` | `TBD_SUCCESS_HEX` | `TBD_SUCCESS_DARK_HEX` | completed/verified success |
| `color.status.warning` | `TBD_WARNING_HEX` | `TBD_WARNING_DARK_HEX` | incomplete/stale/caution |
| `color.status.danger` | `TBD_DANGER_HEX` | `TBD_DANGER_DARK_HEX` | destructive/error |
| `color.status.info` | `TBD_INFO_HEX` | `TBD_INFO_DARK_HEX` | neutral system information |

### 21.3 Chess and analytical tokens

| Token family | Tokens |
|---|---|
| Board | `board.light`, `board.dark`, `board.selected`, `board.lastMove`, `board.legalMove`, `board.annotation`, `board.engineArrow` |
| Engine | `engine.whiteAdvantage`, `engine.blackAdvantage`, `engine.equal`, `engine.mate`, `engine.pending` |
| Results | `result.white`, `result.draw`, `result.black` with labels/pattern alternatives |
| Confidence | `confidence.high`, `confidence.medium`, `confidence.low`, `confidence.insufficient` |
| Evidence | `evidence.source`, `evidence.statistic`, `evidence.engine`, `evidence.interpretation`, `evidence.warning` |
| Charts | `chart.series.1` through `.8`, plus `chart.grid`, `chart.axis`, `chart.selection`, `chart.interval` |

Board colors are chosen after piece-set contrast testing. “White/Black result” colors must not assume literal white and black fills in every theme.

### 21.4 Dark mode

Dark mode uses deep neutral surfaces with small luminance steps. Primary text should not be pure white over pure black. Borders become slightly more visible because shadow cues weaken. Accents lose saturation if they bloom. Charts use theme-specific series colors with equivalent distinguishability. The board may use a slightly lighter surface than the application background to remain legible without glowing.

## 22. Spacing and density

Use a 4 px base unit with a deliberately small set:

| Token | Nominal size | Use |
|---|---:|---|
| `space.0` | 0 | reset |
| `space.1` | 4 | icon/text micro gaps |
| `space.2` | 8 | compact inline gaps |
| `space.3` | 12 | control groups and dense padding |
| `space.4` | 16 | default component padding |
| `space.5` | 20 | panel padding |
| `space.6` | 24 | section-internal spacing |
| `space.8` | 32 | major groups |
| `space.10` | 40 | page section gap on laptops |
| `space.12` | 48 | spacious page separation |
| `space.16` | 64 | Home/empty-state rhythm only |

### Density modes

- **Comfortable:** reading, Home, Player Overview, Investigation answer.
- **Default:** normal forms, filters, position results, studies.
- **Compact:** game tables, opening trees, engine lines, audit details.

Density changes row height, vertical padding, and some metadata visibility—not type family, hierarchy, or target size. User preference may set a default, but each surface retains a sensible minimum/maximum.

## 23. Radius, border, and shadow

### Radius

- `radius.xs`: 3–4 px for compact cells and board-adjacent controls;
- `radius.sm`: 6 px for buttons, inputs, chips;
- `radius.md`: 8–10 px for popovers and small panels;
- `radius.lg`: 12–14 px for dialogs and prominent empty-state surfaces;
- `radius.full`: only avatars, status dots, and true pills.

Do not place each section inside a rounded card. Panels are usually defined by layout, surface change, and one divider.

### Borders

- subtle dividers separate large regions;
- default borders clarify interactive controls;
- strong borders are reserved for focus, selected structures, and high-contrast mode;
- nested surfaces should not each add a border.

### Shadows

- `shadow.none` for normal page/panel structure;
- `shadow.popover` for floating menus;
- `shadow.dialog` for modal separation;
- `shadow.drag` for a dragged object only.

Dark mode relies more on border/luminance than shadow.

## 24. Iconography

Use one outline system with an optical 1.5–2 px stroke and rounded joins. Lucide, Phosphor, or a similar set may be evaluated; mixing families is prohibited. Default sizes are 16 px in compact controls, 18–20 px in navigation, and 24 px only for large empty states.

Rules:

- text accompanies unfamiliar or consequential actions;
- icon-only controls require an accessible label and tooltip;
- filled variants indicate state only when the base system supports them consistently;
- chess pieces are domain content, not general navigation icons;
- exact/similar/structure matching receives custom but simple semantic icons;
- status is never icon-only.

## 25. Motion

Motion communicates containment and continuity.

| Token | Range | Use |
|---|---:|---|
| `motion.instant` | 0–80 ms | selection/pressed feedback |
| `motion.fast` | 120–160 ms | hover, small disclosure |
| `motion.normal` | 180–240 ms | drawer, sidebar, panel |
| `motion.slow` | 280–360 ms | rare workspace transition |

Use an ease-out curve for entering, ease-in for leaving, and a balanced standard curve for position changes. Board pieces may animate quickly between squares when the move is user-triggered; rapid move navigation suppresses animation. Live engine numbers cross-fade or update in place without jumping layout.

Forbidden: bouncing, decorative springs, parallax, animated gradients, long page transitions, or motion on every chart refresh. Reduced motion removes spatial animation and keeps immediate opacity/state changes.

## 26. Chess board design

The board is flat, readable, and subordinate to the research task.

- Piece set: modern, classical proportions, high silhouette differentiation, no ornamental medieval detail.
- Squares: moderate contrast; comfortable for hours; tested in both themes.
- Coordinates: outside or inset subtly; orientation-aware; hideable.
- Selection: clear outline/surface state separate from last move.
- Legal moves: small center mark for empty destinations and ring for captures; optional for expert mode.
- Arrows: distinguish user annotation, engine, and evidence paths by line style plus semantic token.
- Multiple arrows fade by priority; selected arrow becomes dominant.
- Flip: short spatial transition or instant under reduced motion.
- Drag/drop: piece remains aligned, legal targets visible; click-click movement supported.
- Keyboard/screen reader: board exposes square, piece, legal moves, side to move, and last move textually.

The board never uses photo-real wood, beveled pieces, 3D perspective, or ambient glow as the default.

## 27. Component inventory

### Foundation

`ThemeProvider`, `Typography`, `Icon`, `Divider`, `FocusRing`, `VisuallyHidden`, `DensityProvider`.

### Navigation and layout

`AppShell`, `Sidebar`, `ContextToolbar`, `Breadcrumbs`, `WorkspaceTabs`, `SplitPane`, `ContextPanel`, `CommandPalette`.

### Input

`GlobalSearch`, `QueryComposer`, `PlayerAutocomplete`, `OpeningAutocomplete`, `FenInput`, `PgnInput`, `DateRange`, `RatingRange`, `FilterBuilder`, `FilterChip`, `CorpusSelector`.

### Data display

`DataTable`, `VirtualList`, `GameRow`, `PlayerRow`, `PositionRow`, `Metric`, `MetricDelta`, `ResultBar`, `Sparkline`, `IntervalPlot`, `CoverageSummary`, `FreshnessBadge`.

### Chess

`ChessBoard`, `BoardToolbar`, `MoveTree`, `NotationPanel`, `VariationOutline`, `PositionIdentity`, `OpeningTree`, `GamePreview`, `AnnotationPalette`.

### Analytics

`ChartFrame`, `CohortSummary`, `TrendTimeline`, `ComparisonMatrix`, `SampleIndicator`, `ConfidenceIndicator`, `MethodLink`.

### Engine

`EnginePanel`, `EngineLine`, `EvalBar`, `AnalysisProfile`, `EngineProgress`, `RouteBadge`, `EngineSettings`.

### AI and evidence

`InvestigationComposer`, `InvestigationAnswer`, `ResolvedEntity`, `QuerySummary`, `Claim`, `EvidenceChip`, `EvidenceRail`, `EvidenceDrawer`, `VerificationStatus`, `ToolSummary`.

### Library and feedback

`StudyCard`, `ChapterList`, `ImportJob`, `JobStatus`, `EmptyState`, `ErrorState`, `InsufficientState`, `Toast`, `InlineNotice`, `Progress`.

Each component owns one responsibility. Components consume semantic tokens and domain states; they do not invent local colors, radii, or evidence semantics.

## 28. Tables

Tables are first-class and must feel authored, not like a generic data-grid package.

Capabilities:

- sticky header and optional pinned identity column;
- sortable columns with explicit priority;
- resize, hide/show, and restore defaults;
- row selection and batch actions;
- full keyboard grid navigation;
- compact/default density;
- virtualization for large result sets;
- column configuration persisted per table purpose;
- contextual row preview and open-in-workspace action.

Games table baseline columns: players, result, date, event, ratings, opening/ECO, source. Dense metadata moves into preview rather than creating 20 default columns.

Responsive degradation: below tablet width, tables become prioritized rows with two metadata lines; horizontal scrolling is allowed for expert tables and clearly signaled. Do not convert every row into a giant card.

## 29. Forms, controls, and filters

### Actions

- Primary: one per decision region, reserved for the main forward action.
- Secondary: common alternative with visible border/surface.
- Tertiary/ghost: contextual actions.
- Danger: explicit destructive label; never color alone.
- Icon button: only for established, repeated actions.

### Selection controls

- Segmented control: 2–4 mutually exclusive peer views.
- Tabs: stable content destinations within one object.
- Checkbox: independent choices.
- Radio: one choice in a visible group.
- Toggle: immediate on/off setting, never a submit choice.
- Chips: active filters or compact evidence scopes, not generic decoration.

### Inputs

All inputs support label, help text, validation, loading, disabled, resolved, and ambiguous states. FEN/PGN inputs use mono only for source text and show a parsed board/summary before commit. Rating and date ranges permit direct entry and bounded sliders only where the scale is meaningful.

The filter builder begins with common fields and adds advanced clauses progressively. A natural-language-looking query summary remains visible above raw controls.

## 30. States

| State | Required response |
|---|---|
| Loading | preserve layout; use skeleton only when shape is known; announce progress accessibly |
| Empty | explain what belongs here and provide one relevant action |
| No results | preserve query; name active constraints; suggest specific relaxation |
| Insufficient sample | show available n, threshold/method, and source games without producing a strong conclusion |
| Permission denied | identify the inaccessible resource/scope and legitimate next step |
| Offline | show cached/local capability and queued actions; do not imply cloud freshness |
| Stale data | show snapshot date and refresh action |
| Engine unavailable | retain cached results and offer another route/profile |
| Engine queued | position, profile, route, estimate/progress, cancel |
| Analysis failed | specific failure category, retry path, preserved inputs |
| Corrupted PGN | partial/rejected counts, line/location where safe, downloadable report |
| Ambiguous player | candidate identities with source evidence; no silent merge |
| Dataset incomplete | coverage warning adjacent to affected conclusion |
| AI unavailable | deterministic tools remain usable; saved answer/evidence readable |
| Partially verified | mark individual claims; summarize verification coverage |
| Verification failed | suppress factual styling; expose evidence mismatch and safe retry |

Error copy is direct: “The PGN ends during move 37. We imported 82 complete games and skipped 1 incomplete game.” Avoid “Oops” and blame language.

## 31. Accessibility

- Target WCAG 2.2 AA; aim for AAA on core reading text.
- Every interactive element has a visible focus indicator.
- Focus order follows visual and task order, including resizable panes.
- Skip links reach canvas, notation, and context panel.
- Notation is a semantic ordered structure with current move and variation depth.
- Board has grid semantics or an equivalently tested pattern, square labels, piece labels, legal moves, and textual state.
- Charts include summary, data table, and keyboard-accessible points where appropriate.
- W/D/L, confidence, engine status, and match type never rely on color alone.
- Text supports 200% zoom without loss of core function.
- Pointer targets target 44 px where possible; compact expert rows retain an accessible activation area.
- Reduced motion and high-contrast modes are respected.
- ARIA supplements native semantics; it does not replace them.

## 32. Responsive behavior

### Desktop 1440+

Full shell and two-column comparisons; the professional analysis workspace shows board, move tree, engine/database controls, and coach simultaneously. Large monitors gain simultaneous context, not oversized typography.

### Laptop 1024–1439

Sidebar may default collapsed. Board and move tree remain side by side where at least a useful 420–480 px board and readable notation fit. The coach and reference controls can switch within the right column; filters become a drawer.

### Tablet 768–1023

Navigation becomes a temporary drawer. Board/notation use a two-region layout or stacked switcher by orientation. Comparison becomes aligned sections. Dense tables allow controlled horizontal scroll.

### Mobile below 768

Supported for viewing, lightweight search, game replay, player summary, evidence reading, job monitoring, and brief follow-ups. The shell becomes bottom/temporary navigation with one primary surface. Board, notation, engine, and evidence are mutually selected views. Complex annotation, multi-pane comparison, table configuration, and deep engine settings are deferred to larger screens.

## 33. Performance UX

Performance expectations are part of the design contract:

- search feedback begins within 100 ms; cached/entity results feel immediate;
- metadata and warm position results target the service objectives in product docs;
- player pages progressively render identity/coverage, then core metrics, then expensive sections;
- opening tree preserves the current board and shows scoped loading at the changed branch;
- engine and AI are explicit background work with cancellable progress;
- cached evidence renders before optional refresh.

Use optimistic UI only for reversible local actions such as starring or renaming before conflict. Never optimistically show completed import, analysis, verification, sharing, or deletion. Skeletons must not create fake chart shapes. Long jobs survive navigation and appear in global job status.

## 34. Trust and statistical UX

Trust is a visual system, not a disclaimer.

Every analytical conclusion answers:

- What happened?
- Compared with what?
- How many observations?
- What period and corpus?
- How uncertain/incomplete is it?
- Can I open the contributing games?

Confidence uses plain language first: Strong evidence, Directional, Low sample, Insufficient. Intervals and methodology are available for experts. Decimal precision follows measurement quality. A metric that is a model estimate is labeled as such.

Engine results show engine build/profile and analysis budget. AI claims show verified/interpretive status. Corpus freshness and incomplete coverage sit beside the affected result, not in a distant global banner.

## 35. Microcopy

Voice: precise, concise, neutral, active, and technically honest.

| Situation | Preferred copy |
|---|---|
| No result | “No matching games in the current corpus.” |
| Low sample | “Only 6 games match these filters. Treat the trend as directional.” |
| Stale | “This profile uses data through 12 Sep 2026.” |
| Engine cached | “Cached Stockfish result · 5M nodes · generated 2 days ago.” |
| Ambiguous | “We found three players named A. Petrov. Choose the identity to continue.” |
| Partial import | “Imported 1,284 games. 12 need review.” |
| AI uncertainty | “The data shows a change, but the sample is too small to attribute a stable repertoire shift.” |

Buttons use verbs: Open games, Save investigation, Run analysis, Change filters, Import PGN. Avoid “Submit,” “Magic,” “Unlock insights,” and celebratory filler.

## 36. Keyboard UX

Proposed strategy, subject to platform and assistive-technology testing:

- `Cmd/Ctrl + K`: global search/command palette;
- `/`: focus local query when not editing text;
- Left/Right: previous/next move in board context;
- Up/Down: variation or row movement depending focus;
- `E`: toggle engine panel when focus is outside text input;
- `F`: open filters in research views;
- `G` then a destination key: optional navigation chord for power users;
- `Esc`: close topmost transient UI or return one variation level;
- `?`: shortcut reference.

Shortcuts are scope-aware, shown in tooltips/menus, customizable later, and never override browser/OS expectations without strong reason. All have discoverable menu/button paths.

## 37. Page blueprints

### Home

```text
+------------------------------------------------------+
| sidebar |                                            |
|         |  Search players, games, positions…         |
|         |  [____________________________________]    |
|         |                                            |
|         |  Resume                                    |
|         |  recent investigation / study / position   |
|         |                                            |
|         |  Active work (only when present)            |
+------------------------------------------------------+
```

### Explore — Games / Positions

```text
+------------------------------------------------------+
| query summary                         corpus · count  |
| Games | Positions | Players | Openings       Filters |
+------------------------------------------------------+
| result table/list                    | preview        |
| selected row                         | board/details  |
+------------------------------------------------------+
```

### Opening Explorer / Position

```text
+--------------------+---------------------------------+
| board              | position label · sample         |
|                    | move tree / W-D-L / trend        |
|                    |                                 |
+--------------------+---------------------------------+
| representative games / evidence                      |
+------------------------------------------------------+
```

### Player Overview / Repertoire

```text
+------------------------------------------------------+
| identity · rating context · coverage · actions        |
| Overview | Repertoire | Trends | Games                |
+------------------------------------------------------+
| evidence-backed summary                              |
| repertoire distribution      recent change timeline  |
| performance vs expected      representative games    |
+------------------------------------------------------+
```

### Player Comparison

```text
+--------------------------+---------------------------+
| Player A                 | Player B                  |
+--------------------------+---------------------------+
| shared filters · coverage · common axis               |
| aligned repertoire / performance / trend sections     |
+------------------------------------------------------+
```

### Game Workspace

```text
+--------------------+---------------------------------+
| board              | notation / variations / comments|
|                    |                                 |
+--------------------+---------------------------------+
| Reference | Engine | Evidence | Notes                 |
+------------------------------------------------------+
```

### Investigation

```text
+------------------------------------------------------+
| question                                             |
| resolved player · opening · dates · corpus            |
+------------------------------------------------------+
| concise answer                                       |
| claim [evidence]                                     |
| comparison / representative games                    |
+------------------------------------------------------+
| follow-up input · save · change filters · audit       |
+------------------------------------------------------+
```

### Library / Study

```text
+------------------------------------------------------+
| Library search · Import                               |
| Imports | Studies | Saved research                    |
| compact objects and status                            |
+------------------------------------------------------+

+---------------+--------------------------------------+
| Study chapters| chapter: prose, board, game, evidence |
+---------------+--------------------------------------+
```

## 38. Core user flows

### A. Search player -> inspect repertoire

Entry: Home/global search. Resolve identity; open Player; coverage loads first; choose Repertoire; adjust color/time window; inspect move distribution; open evidence games. Ambiguous names require selection. Keyboard: `Cmd/Ctrl+K`, type, arrows, Enter, local tabs.

### B. Player -> opening -> games

Select an opening row; preserve player/cohort filters; open opening explorer; choose continuation; open Games; selected game opens at occurrence. Completion is a source game without losing the original player context.

### C. Paste FEN -> exact position -> continuation -> game

Paste into search; validate/preview; open Exact Position; review corpus/sample; select continuation; open representative game. Invalid FEN explains the field; zero matches offers Similar/Pawn Structure as explicit alternatives.

### D. Position -> engine -> evidence

Open engine panel; select quick/standard/deep route; start/cancel; inspect PV; attach completed result to evidence/study. Cached results are labeled. Failure retains position and profile.

### E. Natural-language question -> investigation -> evidence -> game

Parse question; display resolved entities/filters; user corrects ambiguity; execute; render answer; expand evidence; open source game; return with state preserved. Unsupported claims are suppressed or marked interpretive.

### F. Compare players

Choose first player; Add comparison; resolve second player; set shared color/opening/time filters; inspect aligned metrics; open differential evidence. Never compute an overall winner.

### G. Import PGN -> browse games

Select file and privacy route; preview; start background job; show accepted/partial/rejected; open completed collection; filter games. Corrupt items have a review report; valid games remain available.

### H. Save research to Study

From Player/Position/Game/Investigation choose Save to Study; select/create Study and chapter; choose snapshot versus live reference where applicable; confirm. Keyboard path uses command palette. Saved evidence retains corpus/version.

## 39. MVP design scope

### Design system foundation — required before implementation

- semantic color architecture for both themes;
- typography, spacing, radius, borders, focus, motion, and density;
- AppShell, navigation, search, table, form, feedback, overlay primitives;
- board, notation, evidence, metric, and query-summary semantics;
- accessibility and responsive behavior.

### MVP required

- Home and global search;
- Explore Games and Exact Positions;
- Player Overview and Repertoire;
- Opening Explorer;
- Game Workspace with core annotation;
- Position Workspace;
- evidence chips/drawer and corpus/query summaries;
- PGN/Lichess/Chess.com import states;
- basic local/cloud engine contract and panel;
- Library and lightweight Study/Chapter save path;
- light and dark modes using unresolved-to-finalized semantic palette.

### V1 after MVP validation

- Player Comparison;
- saved queries and richer Study organization;
- novelty/change timelines;
- configurable columns/density;
- constrained natural-language investigations with claim verification;
- expanded engine profiles and batch jobs.

### Later

- similar-position and pawn-structure search;
- team collaboration and permissions UX;
- local desktop companion/offline states;
- repertoire training;
- human-move models and advanced analytics;
- mobile-specific optimized workflows.

The design system anticipates later objects, but MVP navigation and component scope remain small.

## 40. Anti-patterns and non-goals

Reject:

- a dashboard of unrelated KPI cards;
- chat as the home screen and only navigation model;
- card-inside-card layouts;
- permanent three/four-pane complexity on every page;
- every control in pill form;
- glassmorphism, neon, cyberpunk, wood, crowns, heraldry, and chess-piece logo clichés;
- decorative gradients or shadows used to signal “AI”;
- gamification, streaks, badges, arbitrary player scores, and “winner” comparisons;
- hidden samples, corpora, filters, or uncertainty;
- false precision;
- engine settings exposed by default;
- 14 top-level destinations;
- mobile parity that damages desktop research;
- online-play lobby, social feed, puzzle-first navigation, streaming, course marketplace, or NFT surfaces.

## 41. Open design questions

These are recommendations or unresolved topics, not established scope:

1. **Typeface licensing:** validate Geist or alternatives across operating systems, chess glyphs, tabular figures, and commercial distribution.
2. **Final palette:** founder supplies brand direction/HEX; test semantic mappings, piece sets, charts, and both themes before acceptance.
3. **Sidebar model:** test collapsed-default behavior on 13-inch laptops versus discoverability for new users.
4. **Workspace tabs:** validate whether serious users need browser-like object tabs in MVP or whether browser history plus recent items is enough.
5. **Evidence Rail placement:** compare right-side rail, inline expansion, and drawer through claim-to-game tasks.
6. **Board/notation breakpoint:** determine the smallest width at which side-by-side remains superior to a switcher.
7. **Study depth:** product docs include Chapters, but the alpha may need only “Save to collection.” Validate before building a document editor.
8. **AI presence:** investigation is later scope. Test whether its entry belongs in global search, contextual action, or both without making Home chat-first.
9. **Player summary language:** determine which interpretations professionals trust and which should remain raw metrics.
10. **Density preference:** decide whether user-selectable global density adds value or surface-specific density is sufficient.
11. **Comparison mobile behavior:** likely read-only stacked sections; validate whether mobile comparison is needed at all.
12. **Local/cloud route:** design the selection only after engine privacy and companion timing are finalized.

## Design risks

| Risk | Failure mode | Mitigation |
|---|---|---|
| Too much density | first-time users cannot identify the task | three-depth disclosure, low-density Home/Overview, task testing |
| Too many panels | the product becomes layout management | one optional context panel, task-specific persisted workspace modes |
| AI dominates | deterministic workflows feel secondary or unavailable | AI stays contextual; search/filter/pages remain complete without it |
| Evidence clutter | every sentence becomes a row of badges | four evidence depths; show count and warning first, audit on demand |
| Opening tree complexity | rows become an unreadable spreadsheet | default leading moves; advanced dense mode; direct labels and keyboard control |
| Engine complexity | basic analysis resembles a cockpit | named profiles and 1–3 lines by default; advanced settings collapsed |
| Analytics intimidation | charts obscure decisions | narrative order, direct labels, source-game drill-down, restrained chart set |
| Desktop over-orientation | tablet/mobile becomes unusable | supported viewing flows, single-surface mobile model, no false parity promise |
| Generic SaaS appearance | ChessScope loses identity | Evidence Rail, variation-derived structure, domain content, no generic KPI/card shell |
| ChessBase with nicer CSS | legacy complexity is reproduced | workflow-based IA, universal search, progressive disclosure, usability benchmarks |
| False trust | polished UI overstates weak data | sample/corpus/interval adjacent to claim; insufficient state; audit path |
| Palette delay | components hard-code temporary colors | semantic tokens only; automated contrast/theme checks |

## Implementation handoff rules

When UI implementation begins:

1. Build and review foundation tokens before feature-local styling.
2. Prototype Home/Search, Position/Opening Explorer, Player, and Game Workspace at 1280 and 1440 widths first.
3. Test one complete evidence path from claim to games before building AI presentation.
4. Validate keyboard and screen-reader models with the first board/notation prototype.
5. Use real chess content and long names, variations, filters, missing data, and low-sample states in every design review.
6. No component may hard-code brand color, local radius, or unversioned evidence semantics.
7. Any deviation from this canonical direction should update this document or receive a design decision record.

## Sources and relationship to existing context

This specification elaborates, and does not supersede, the product, architecture, and UX documents in `docs/`. It preserves the accepted web-first hybrid direction, database-and-analytics MVP, evidence contracts, professional audience, and later AI scope.

Primary public references reviewed:

- [Apple](https://www.apple.com/)
- [Notion product](https://www.notion.so/product)
- [OpenAI](https://openai.com/)
- [Hermes Agent](https://hermes-agent.nousresearch.com/)
- [ChessBase 26 guide](https://en.chessbase.com/post/chessbase-2026-a-players-guide-2)
- [Lichess analysis](https://lichess.org/analysis)
- [En Croissant](https://github.com/franciscoBSalgueiro/en-croissant)

The canonical product requirements remain in [Scope and requirements](../product/03-scope-and-requirements.md); this file defines how those requirements should look and behave.
