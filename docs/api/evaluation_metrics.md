# Evaluation Metrics

This document explains the evaluation metrics used by the current HA-VLN action replay scorer. Understanding these metrics will help you interpret your agent's behavior during development and validation.

## Overview

The current evaluator exposes four core challenge-facing metrics:

1. **Strict Success (`SR`)**
2. **Trajectory Collision Rate (`TCR`)**
3. **Navigation Error (`NE`)**
4. **Collision Rate (`CR`)**

The evaluator also keeps the underlying environment `success` signal. In the current HA-VLN challenge workflow, `SR` is stricter than plain `success`.

## Current Evaluator Outputs

In the current output schema, `episode_metrics.json` contains one row per replayed episode. Rows include the primitive action trace and metric fields such as:

- `success`
- `distance_to_goal`
- `steps_taken`
- `actions`
- `path_length`
- `collisions`
- `spl`
- `ndtw`
- `sdtw`
- `TCR`
- `CR`
- `SR`

At the summary level, the current evaluator writes:

- `score_summary.json`, which stores aggregated metrics under `metrics`

In the current schema, the canonical metric key is `distance_to_goal`. `NE` remains a useful shorthand for discussion.

## Metric Definitions

### 1. Strict Success (`SR`)

**Definition**: Mean strict-success value across episodes.

At the episode level, the current implementation defines:

```text
SR_episode = success * int(TCR == 0)
```

This means an episode contributes `1` only if:

1. the environment reports `success == 1`
2. the adjusted trajectory collision count `TCR` is zero

The dataset-level summary is then:

```text
SR = Σ(SR_episode) / (Total episodes)
```

Higher is better.

Important note:

- `SR` is not the same as plain environment `success`
- in the current evaluator, `SR` is a strict success metric
- the exported summary value is a mean in `[0, 1]`, not a percentage unless you multiply it by `100` for presentation

### 2. Trajectory Collision Rate (`TCR`)

**Definition**: Average adjusted human-collision count per episode after subtracting the precomputed unavoidable collision component used by the metric implementation.

At the episode level, the current implementation defines:

```text
TCR_episode = max(0, collisions.count - unavoidable_collision_baseline)
```

The dataset-level summary is then:

```text
TCR = Σ(TCR_episode) / (Total episodes)
```

Lower is better.

Interpretation:

- `TCR = 0`: no counted human-collision events after adjustment
- `TCR > 0`: some counted human-collision events occurred after adjustment

### 3. Navigation Error (`NE`)

**Definition**: Mean final distance-to-goal value across episodes.

**Units**: meters

In the current evaluator implementation, the canonical metric key is `distance_to_goal`. `NE` remains a useful shorthand for discussion, but the JSON summary now keeps the underlying metric name.

```text
NE = Σ(distance_to_goal at episode end) / (Total episodes)
```

Lower is better.

### 4. Collision Rate (`CR`)

**Definition**: Mean episode-level collision indicator across episodes.

At the episode level, the current implementation defines:

```text
CR_episode = min(TCR_episode, 1)
```

So in the current evaluator:

- an episode contributes `0` if its adjusted collision count is zero
- an episode contributes `1` if its adjusted collision count is one or more

The dataset-level summary is then:

```text
CR = Σ(CR_episode) / (Total episodes)
```

Lower is better.

Important note:

- the exported summary value is a mean in `[0, 1]`
- it is often interpreted like a rate, but the current evaluator does not multiply it by `100`

## Human-Aware Interpretation

### What Counts as a Collision Here?

In the current HA-VLN challenge workflow, the human-aware metrics are meant to reflect interaction with dynamic humans rather than only static-scene collisions.

### Why `TCR` and `CR` Both Matter

- `TCR` measures how much adjusted human-collision behavior accumulates across episodes
- `CR` measures how often an episode has at least one adjusted collision event

Together they help distinguish frequency from severity.

### Why `success` and `SR` Both Matter

- `success` tells you whether the episode satisfied the environment success condition
- `SR` tells you whether the episode was both successful and collision-clean under the strict HA-VLN rule

This is why two agents with similar plain success can still differ on the challenge-facing `SR` metric.

## Practical Implications for Participants

When iterating on your own agent, use these metrics together rather than optimizing only one of them.

Useful questions to ask are:

- does the agent reach goals reliably under plain `success`?
- does it also preserve strict success under `SR`?
- are failures caused more by navigation error or by adjusted human collisions?

## Notes

- this page follows the current metric code in `HASimulator/metric.py` and the challenge action replay scorer
- final scoring is based on replaying `actions.json` in the official CE environment
- if future evaluator code changes the exported metric definitions, the documentation should be updated to match the code

## Further Reading

- [How To Participate](../challenge/overview.md)
- [Action Sequence Format](../challenge/action_sequence_format.md)
- [Collision Checks](collision_checks.md)
