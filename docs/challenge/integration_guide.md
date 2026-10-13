# HA-VLN Challenge: Agent Integration Guide

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This guide explains how to integrate your Vision-and-Language Navigation (VLN) agent with the HA-VLN environment to participate in the RoboWorld 2026 Track 2 Challenge.

---

## Overview

To participate in the challenge, you need to:
1. Configure your agent to run within the HA-VLN dynamic human environment (`HAVLNCE_task.yaml`).
2. Collect discrete string actions during inference on the required splits.
3. Format and bundle the predictions into a `submission.zip` containing `val_seen.json` and `val_unseen.json`.
4. Validate and submit your results on CodaBench.

---

## 1. Environment Configuration

### 1.1 Task Configuration

Point your agent launcher to the HA-VLN task configuration:

```yaml
BASE_TASK_CONFIG_PATH: HASimulator/config/HAVLNCE_task.yaml
```

### 1.2 Key Configuration Switches

Ensure dynamic human rendering and realistic physics are enabled:

```yaml
SIMULATOR:
  ADD_HUMAN: True
  ALLOW_SLIDING: True
  HUMAN_GLB_PATH: Data/HAPS2_0
  HUMAN_INFO_PATH: Data/Multi-Human-Annotations/human_motion.json
```

### 1.3 Environment Wrapper

Dynamic human motions are updated along a timeline. Ensure your environment step loop synchronizes signals:

```python
from habitat.core.env import Env
from HASimulator.environments import HAVLNCE

class HAVLNWrapper(Env):
    def __init__(self, config, dataset=None):
        super().__init__(config, dataset)
        self.use_dynamic_human = getattr(self._config.TASK_CONFIG.SIMULATOR, "ADD_HUMAN", False)
        if self.use_dynamic_human:
            self.havlnce_tool = HAVLNCE(self._config.TASK_CONFIG, self._sim)
            self.havlnce_tool._reset_signal_queue_and_counters()

    def step(self, action):
        if self.use_dynamic_human:
            self.havlnce_tool._handle_signals()
        return super().step(action)
```

---

## 2. Collecting Action Sequences

The official challenge evaluates discrete action strings. Here is a recommended action recording helper:

```python
import json
from collections import defaultdict
from typing import List, Dict, Any

class ActionCollector:
    def __init__(self, action_vocabulary: tuple):
        self.action_vocab = action_vocabulary
        self.traces = defaultdict(list)

    def record_step(self, episode_id: str, action_idx: int):
        """Record the string action literal for an episode step."""
        action_name = self.action_vocab[action_idx]
        self.traces[str(episode_id)].append(action_name)

    def export_json(self, output_path: str, split_name: str):
        """Export the formatted JSON predictions."""
        data = {
            "format_version": 1,
            "split": split_name,
            "episodes": [
                {"episode_id": ep_id, "actions": self.traces[ep_id]}
                for ep_id in sorted(self.traces)
            ],
        }
        with open(output_path, "w", encoding="utf-8") as f:
            json.dump(data, f, indent=2)
```

---

## 3. Official Action Vocabulary

The simulator environment recognizes six discrete string actions:

| Action Literal | Description |
|---|---|
| `"STOP"` | Terminates navigation and ends the episode (mandatory final action). |
| `"MOVE_FORWARD"` | Advance forward 0.25m. |
| `"TURN_LEFT"` | Rotate heading left 15°. |
| `"TURN_RIGHT"` | Rotate heading right 15°. |
| `"LOOK_UP"` | Pitch sensor upward 30°. |
| `"LOOK_DOWN"` | Pitch sensor downward 30°. |

*Planar policies using only `STOP`, `MOVE_FORWARD`, `TURN_LEFT`, and `TURN_RIGHT` are fully valid and supported.*

---

## 4. Validating Action Sequences

Verify your sequences locally before packaging:

```python
VALID_ACTIONS = {
    "STOP", "MOVE_FORWARD", "TURN_LEFT", "TURN_RIGHT", "LOOK_UP", "LOOK_DOWN"
}

def validate_episode_actions(actions: List[str]) -> bool:
    if not actions or len(actions) > 500:
        return False
    if actions[-1] != "STOP" and len(actions) < 500:
        return False
    for a in actions:
        if a not in VALID_ACTIONS:
            return False
    return True
```

---

## 5. Packaging Submissions

Export `val_seen.json` and `val_unseen.json`, then bundle into a flat ZIP:

```bash
zip -j submission.zip val_seen.json val_unseen.json

# Validate via Docker container (or 'havln-validate submission.zip' inside container):
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip
```

Submit `submission.zip` to the [RoboWorld 2026 Track 2 CodaBench Competition](https://www.codabench.org/competitions/18135/).

---

## Resources & Links

- [RoboWorld 2026 Track 2 CodaBench](https://www.codabench.org/competitions/18135/)
- [Participant Kit (roboworld2026-track2)](https://github.com/JostarXiong/roboworld2026-track2)
- [Submission Format Specification](submission_format.md)
- [Evaluation Metrics & Scoring Formula](../api/evaluation_metrics.md)
- [HA-VLN GitHub Repository](https://github.com/JostarXiong/HA-VLN)