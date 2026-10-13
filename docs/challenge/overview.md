# HA-VLN Challenge: Overview

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

The Human-Aware Vision-and-Language Navigation (HA-VLN) Challenge (RoboWorld 2026 Track 2) evaluates an embodied agent's ability to navigate in dynamic human-populated indoor environments while following natural language instructions.

---

## Challenge Objective

The goal is to develop autonomous agents that can:
1. **Navigate** from a starting position to a goal location in photo-realistic 3D environments.
2. **Follow** natural language instructions that describe both spatial paths and dynamic human activities.
3. **Avoid** collisions with dynamic humans performing various activities.
4. **Demonstrate** socially-aware navigation behaviors.

---

## Key Features

### 1. Dynamic Human Activities
- Real-time rendering of human motions and interactions using HAPS 2.0.
- Humans perform diverse daily activities (walking, talking, reading, exercising, searching).
- Human positions and poses update dynamically along the timeline.

### 2. Complex Instructions
- Natural language descriptions referencing both physical landmarks and dynamic people.
- Examples: *"Exit the library and turn left. Continue moving forward, ensuring you do not disturb the person searching for a lost item."*

### 3. Human-Aware Evaluation & Official Composite Score
- Evaluation balances navigation task completion and social safety.
- The official leaderboard ranks agents primarily by **Composite Score**:

$$
\mathrm{Score} = 100 \times \mathrm{Navigation} \times (0.70 + 0.30 \times \mathrm{Social})
$$

where:

$$
\mathrm{Navigation} = 0.80 \times \mathrm{SR} + 0.20 \times \frac{3}{3 + \mathrm{NE}}, \quad \mathrm{Social} = 0.75 \times (1 - \mathrm{CR}) + 0.25 \times \frac{1}{1 + \mathrm{TCR}}
$$

---

## Dataset Splits & Benchmark Scope

The HA-R2R benchmark extends R2R with 910 placed human models across 90 scenes:

| Split | Trajectories | Episodes | Human-Influenced Episodes |
|---|---|---|---|
| `train` | 3,474 | 10,422 | 8,970 |
| `val_seen` | 259 | 778 | 682 |
| `val_unseen` | 613 | 1,839 | 1,593 |

For Phase 1 validation evaluation, participants submit predictions covering all 778 `val_seen` and 1,839 `val_unseen` episodes.

---

## Action Vocabulary

Agents output sequences composed of six discrete string actions:

1. `"STOP"`: Finish navigation and terminate the episode.
2. `"MOVE_FORWARD"`: Translate forward 0.25 meters.
3. `"TURN_LEFT"`: Rotate left 15 degrees.
4. `"TURN_RIGHT"`: Rotate right 15 degrees.
5. `"LOOK_UP"`: Pitch camera upward 30 degrees.
6. `"LOOK_DOWN"`: Pitch camera downward 30 degrees.

Planar-only policies (using only the first four actions) are fully valid and supported.

---

## Submission & Evaluation Flow

1. **Inference**: Run your agent on the target splits and record executed string actions.
2. **Export**: Save `val_seen.json` and `val_unseen.json` with `format_version: 1`.
3. **Package**: Bundle them into `submission.zip`:
   ```bash
   zip -j submission.zip val_seen.json val_unseen.json
   ```
4. **Validation**: Check format locally via the evaluation Docker container:
   ```bash
   IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
   docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip
   ```
   *(or `havln-validate submission.zip` directly inside the container).*
5. **Submit**: Upload the validated zip to the [RoboWorld 2026 Track 2 CodaBench Competition](https://www.codabench.org/competitions/18135/).

---

## Resources & Links

- **CodaBench Challenge**: [RoboWorld 2026 Track 2 Competition](https://www.codabench.org/competitions/18135/)
- **Participant Starter Kit**: [GitHub - roboworld2026-track2](https://github.com/JostarXiong/roboworld2026-track2)
- **HA-VLN Codebase**: [GitHub - JostarXiong/HA-VLN](https://github.com/JostarXiong/HA-VLN)
- **Hugging Face Data**: [fly1113/HA-VLN](https://huggingface.co/datasets/fly1113/HA-VLN)
- **Detailed Specifications**:
  - [Getting Started Guide](getting_started.md)
  - [Agent Integration Guide](integration_guide.md)
  - [Submission Format Specification](submission_format.md)
  - [Evaluation Metrics & Scoring Formula](../api/evaluation_metrics.md)