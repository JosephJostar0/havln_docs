# Challenge Participation

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This guide provides a quick overview of how to participate in the RoboWorld 2026 Track 2 (HA-VLN) Challenge. For full details, see the [Challenge Overview](../challenge/overview.md) and [Submission Format](../challenge/submission_format.md).

---

## Quick Start Checklist

### 1. Environment Setup
- [ ] Pull pre-built Docker image (`ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4`) or set up Native Conda (Py3.8 / CUDA 11.8)
- [ ] Run verification tests (`python scripts/demo.py --scan 1LXtFkjw3qL --headless`)

### 2. Data Preparation
- [ ] Download HA-R2R dataset & HAPS 2.0 human motion models via Hugging Face (`hf download fly1113/HA-VLN`)
- [ ] Download Matterport3D scenes (`download_mp.py` + unzip into `Data/scene_datasets`)
- [ ] Verify dataset layout

### 3. Agent Integration & Action Recording
- [ ] Configure your agent for HA-VLN environment (`HAVLNCE_task.yaml`)
- [ ] Record discrete string actions during inference
- [ ] Test on `val_seen` (778 episodes) and `val_unseen` (1,839 episodes)

### 4. Submission
- [ ] Export `val_seen.json` and `val_unseen.json` with `format_version: 1`
- [ ] Create `submission.zip` containing both JSON files
- [ ] Validate locally via Docker container: `docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip`
- [ ] Submit to [CodaBench](https://www.codabench.org/competitions/18135/)

---

## Action Vocabulary

The simulator environment evaluates six discrete string actions:

| Action | Description |
|---|---|
| `"STOP"` | End episode (must be the final action) |
| `"MOVE_FORWARD"` | Advance agent by 0.25m |
| `"TURN_LEFT"` | Rotate heading left by 15° |
| `"TURN_RIGHT"` | Rotate heading right by 15° |
| `"LOOK_UP"` | Pitch sensor upward by 30° |
| `"LOOK_DOWN"` | Pitch sensor downward by 30° |

---

## Recording Actions during Inference

```python
# During agent inference, record actions as strings from the vocabulary:
action_names = tuple(config.TASK_CONFIG.TASK.POSSIBLE_ACTIONS)
action_traces = {str(ep.episode_id): [] for ep in current_episodes}

# In your step loop:
for episode, action_id in zip(current_episodes, actions):
    action_traces[str(episode.episode_id)].append(action_names[action_id])
```

Export each split to JSON (`val_seen.json`, `val_unseen.json`):

```python
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

Package the submission:

```bash
zip -j submission.zip val_seen.json val_unseen.json

# Validate via Docker container (or 'havln-validate submission.zip' inside container):
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4
docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip
```

---

## Official Challenge Links

- [RoboWorld 2026 Track 2 CodaBench](https://www.codabench.org/competitions/18135/)
- [Participant Repository (roboworld2026-track2)](https://github.com/JostarXiong/roboworld2026-track2)
- [Submission Format Specification](../challenge/submission_format.md)
- [Evaluation Metrics & Scoring Formula](../api/evaluation_metrics.md)