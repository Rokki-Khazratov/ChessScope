# Vision and principles

Status: accepted direction  
Owners: product and architecture

## Vision

ChessScope is an AI-native chess intelligence platform that turns games, positions, engine analysis, and player history into searchable, explainable, and verifiable insights.

The product should become the easiest serious chess analysis environment to use without giving up the depth expected by professionals.

## Product promise

A user can ask a chess research question and move continuously between the answer, its statistics, the underlying positions, and the source games.

Example:

> How has Abdusattorov responded to the Catalan as Black in classical games since 2024?

A valid answer is not prose alone. It includes:

- resolved player identity;
- explicit position/opening definition;
- color, date, and time-control filters;
- matching-game count and corpus identity;
- continuation distribution and uncertainty;
- representative games;
- links from every material claim to its evidence.

## Four product pillars

### Explore

Find games, positions, players, events, openings, structures, and trends. Position navigation is first-class rather than an incidental feature of a game viewer.

### Analyze

Combine reference statistics with reproducible local or cloud engine analysis. Statistical facts and engine judgments remain distinct.

### Understand

Explain what the evidence means in clear language. AI may orchestrate and interpret; it may not invent deterministic chess facts.

### Prepare

Reveal an opponent's repertoire, changes, preferences, surprises, practical strengths, and representative games.

## Principles

1. **Evidence before eloquence.** A modest supported answer is better than a confident unsupported one.
2. **Position-centric by design.** Users may enter through a player, game, opening, or position and traverse all related objects.
3. **Progressive disclosure.** Common actions remain simple; professional depth appears when requested.
4. **Deterministic core, probabilistic interface.** Search, metrics, legality, and arithmetic are code; the LLM plans and explains.
5. **Reproducibility is a feature.** Engine version, node budget, dataset version, filters, and metric version travel with results.
6. **Hybrid privacy.** Private preparation should not require surrendering all data or engine work to a cloud service.
7. **Scale by evidence.** Start with simple infrastructure, benchmark realistic workloads, and split storage only when measured constraints justify it.
8. **Uncertainty is visible.** Small samples, missing metadata, model confidence, and corpus bias are part of the result.

## Differentiation

ChessScope is not differentiated by possessing a chessboard UI, running Stockfish, or adding a chat panel. Its intended moat is the combination of:

```text
canonical chess data
+ position graph
+ statistically defensible player models
+ provenance and evidence contracts
+ accumulated analysis
+ low-friction professional workflows
```

## Non-goals

- operating a chess-playing server;
- building a social network or news portal;
- generic puzzle and course catalogs;
- training a foundation language model in the first phases;
- copying ChessBase's full historical feature surface;
- presenting unverified AI explanations as authoritative chess knowledge.

## Long-term standard

The phrase “ChessBase-class” describes professional depth and corpus ambition, not a requirement to clone ChessBase's interface or proprietary data. ChessScope should preserve the value of reference research, preparation, annotation, engines, and repertoire work while rethinking navigation around evidence and questions.
