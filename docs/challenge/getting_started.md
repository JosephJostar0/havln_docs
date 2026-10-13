# HA-VLN Challenge: Getting Started

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This guide provides a step-by-step walkthrough for setting up your environment, acquiring data, recording action sequences with your agent, and submitting to the RoboWorld 2026 Track 2 Challenge.

---

## 1. Environment Setup

Participants can choose between:
- **Docker Container (Recommended)**:
  ```bash
  IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
  docker pull "$IMAGE"
  ```
- **Native Conda Environment (Python 3.8 / CUDA 11.8)**:
  Follow the [Installation Guide](../quick_start/installation.md) to set up Habitat-Sim 0.1.7, Habitat-Lab 0.1.7, and PyTorch 2.0.1.

---

## 2. Dataset Acquisition

Download the HA-R2R episodes, HAPS 2.0 human models, and licensed Matterport3D scene meshes:

```bash
# 1. Download HA-VLN data from Hugging Face
pip install huggingface-hub
hf download fly1113/HA-VLN --repo-type dataset --local-dir Data

# 2. Download and unzip Matterport3D Habitat scenes
python3 download_mp.py -o Data/scene_datasets --task_data habitat
unzip Data/scene_datasets/v1/tasks/mp3d_habitat.zip -d Data/scene_datasets
```

See [Data Download](../quick_start/data.md) for full instructions and directory layout.

---

## 3. Agent Integration & Action Recording

The challenge evaluates action sequences rather than raw agent positions. Your agent must record the discrete action string literals executed at each environment step:

```python
import json
from collections import defaultdict

# Map action indices to the official 6-action string vocabulary
action_names = tuple(config.TASK_CONFIG.TASK.POSSIBLE_ACTIONS)
action_traces = defaultdict(list)

# In your inference loop:
for episode, action_id in zip(current_episodes, actions):
    action_traces[str(episode.episode_id)].append(action_names[action_id])
```

Export each split to a JSON file:

```python
split = config.INFERENCE.SPLIT
with open(f"{split}.json", "w", encoding="utf-8") as f:
    json.dump({
        "format_version": 1,
        "split": split,
        "episodes": [
            {"episode_id": ep_id, "actions": action_traces[ep_id]}
            for ep_id in sorted(action_traces)
        ]
    }, f, indent=2)
```

---

## 4. Packaging and Validating Submissions

For Phase 1 validation evaluation, create a ZIP file containing both split files:

```bash
zip -j submission.zip val_seen.json val_unseen.json
```

Verify your submission format locally before uploading using the evaluation Docker container:

```bash
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4

# Run validation from host via Docker:
docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip
```

*(Note: If you are already working inside the interactive evaluation container, you can run `havln-validate submission.zip` directly).*

The validator confirms:
- Both `val_seen.json` and `val_unseen.json` are present at the root of the archive.
- `format_version == 1` and all actions belong to the 6-action vocabulary.
- Exactly 778 episodes for `val_seen` and 1,839 episodes for `val_unseen` are included.

---

## 5. Submitting to CodaBench

Upload your validated `submission.zip` to the [RoboWorld 2026 Track 2 CodaBench Competition](https://www.codabench.org/competitions/18135/).

---

## Helpful Links

- [RoboWorld 2026 Track 2 CodaBench](https://www.codabench.org/competitions/18135/)
- [Participant Kit (roboworld2026-track2)](https://github.com/JostarXiong/roboworld2026-track2)
- [Agent Integration Guide](integration_guide.md)
- [Submission Format Specification](submission_format.md)
- [Evaluation Metrics & Scoring Formula](../api/evaluation_metrics.md)