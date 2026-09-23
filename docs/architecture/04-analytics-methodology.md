# Analytics methodology

Status: candidate metrics; independent statistical review required

## Rule

Metrics are computed by versioned deterministic code. LLMs may describe a result but may not calculate or invent it.

Every metric result includes:

- metric name and semantic version;
- corpus snapshot and cohort definition;
- player/entity resolution version;
- sample size and missing-data counts;
- point estimate and uncertainty;
- assumptions and warnings;
- source game/evidence IDs;
- generated timestamp.

## Repertoire entropy

For a position `s` with move probabilities `p_i`:

```text
H(s) = -sum(p_i * log(p_i))
H_normalized(s) = H(s) / log(K)
```

A player-level repertoire score aggregates relevant positions using explicit depth/frequency weights.

Requirements:

- Bayesian/Dirichlet smoothing or an accepted alternative;
- minimum effective sample;
- confidence or credible intervals;
- separate color, time control, era, and corpus filters;
- no interpretation of high entropy as strength by default.

## Opponent-adjusted performance

Baseline expected score for compatible ratings may begin with an Elo expectation:

```text
E_i = 1 / (1 + 10^((R_opp - R_player) / 400))
OAP = mean(actual_score_i - E_i)
```

Production methodology must account for color, time control, era, pool, and repeated-player effects. FIDE, Lichess, and Chess.com ratings cannot be mixed naively. A logistic mixed-effects or similarly reviewed model is a likely later direction.

## Novelty score

Novelty is empirical surprise relative to the corpus known before a game:

```text
P(move | position, before_time)
Novelty = -log(P)
```

Requirements:

- strict temporal cutoff to prevent future leakage;
- smoothing for unseen moves;
- position depth and baseline frequency;
- label wording: “novel relative to this corpus,” never “first ever” without stronger evidence;
- snapshot reproducibility.

This is later than the first two metrics because it needs a correct temporal data model.

## Conversion and recovery

These use engine-derived win/draw/loss probabilities rather than raw centipawns.

Concepts:

- qualifying advantage/disadvantage threshold;
- stability over multiple ply;
- one qualifying state per game to avoid repeated counting;
- result-based conversion/saving outcome;
- peer baseline by rating and time control;
- fixed analysis profile and calibrated engine/model version.

These metrics are expensive and remain later-phase work.

## Preparation surprise

Estimate how unexpected a move is relative to a player's prior repertoire, optionally conditioned on opponent class and weighted toward recent history.

It may indicate preparation, but rarity is not proof of preparation. Product wording must say “surprise” unless external evidence supports a stronger conclusion.

## Additional later metrics

- repertoire/style drift over time;
- time-pressure quality when clock data exists;
- practical difficulty by human-move model and rating cohort;
- opponent-specific deviations;
- endgame-family performance;
- decision complexity and move-choice stability.

## Confounder checklist

Before shipping a metric, review:

- opponent strength and repeated opponents;
- player color;
- time control and increment;
- rating system and era;
- tournament/online context;
- corpus coverage and duplicate handling;
- selection and survivorship bias;
- missing clocks/ratings/dates;
- engine depth/profile;
- multiple comparisons;
- temporal leakage;
- tiny or unbalanced samples.

## UI contract

Every metric card must provide:

```text
value and unit
comparison/baseline
sample size
uncertainty status
cohort/filter summary
time window
method version
open contributing games
open methodology
```

Avoid single composite “style scores” unless their construction is transparent and validated. Decorative precision is forbidden.

## Validation program

1. Freeze representative corpora with known edge cases.
2. Implement a slow, auditable reference calculation.
3. Add property and leakage tests.
4. Ask an independent statistician to try to invalidate the metric.
5. Compare stability across resampling, time windows, and corpora.
6. Run blind expert review of whether conclusions match source games.
7. Version any semantic change; never silently recalculate old evidence.

## MVP metric decision

The first candidates are repertoire entropy and opponent-adjusted performance. Novelty, conversion/recovery, and preparation surprise remain research items until temporal and engine pipelines are validated.
