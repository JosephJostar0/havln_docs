## Agent Integration

Official HA-VLN repository: https://github.com/F1y1113/HA-VLN

This page explains how to connect your own agent to the HA-VLN environment for local method development outside the Docker challenge packaging flow.

The goal of this page is practical integration, not to force a specific model architecture. The repository `agent/` directory is reference material only. For challenge scoring, the final behavior must be exportable as HA-VLN-CE primitive actions.

### 1. Redirect Base Task Config

Redirect `BASE_TASK_CONFIG_PATH` in your agent configuration to the HA-VLN task config:

```yaml
BASE_TASK_CONFIG_PATH: path/to/HASimulator/config/HAVLNCE_task.yaml
```

Also verify the following key switches:

```yaml
SIMULATOR:
  ADD_HUMAN: True
  ALLOW_SLIDING: True
  HUMAN_COUNTING: False

TASK:
  MEASUREMENTS:
    - COLLISIONS_DETAIL
    - DISTANCE_TO_HUMAN
```

Important path note:

- `HAVLNCE_task.yaml` often contains relative paths such as `../Data/...`
- these paths are resolved against the process CWD where your agent starts
- if your launcher runs in another directory, switch those paths to absolute paths to avoid silent file-not-found failures

### 2. Choose the Task Variant You Need

If you are comparing settings or running ablations, switch only `BASE_TASK_CONFIG_PATH` while keeping the rest of your agent code unchanged:

- `HAVLNCE_task.yaml`: dynamic environment + HA-R2R instructions
- `HAVLNCE_R2R_task.yaml`: dynamic environment + original R2R instructions
- `VLNCE_task.yaml`: static environment + original R2R instructions
- `VLNCE_HAR2R_task.yaml`: static environment + HA-R2R instructions

### 3. Synchronize with the Dynamic Human Timeline

Consume dynamic-scene clock signals before `step()` so each agent action runs on the latest physical state.

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

    def reset(self):
        if self.use_dynamic_human:
            self.havlnce_tool.reset()
        return super().reset()

    def step(self, action):
        if self.use_dynamic_human:
            self.havlnce_tool._handle_signals()
        return super().step(action)
```

Why this matters:

- HA-VLN advances human motions using a child-thread clock
- without `_handle_signals()` before each step, policy actions may be evaluated on stale geometry or stale NavMesh

### 4. Use the Provided Text Resources First

Before building custom text preprocessing, check the text resources already bundled with HA-R2R:

- the main dataset files such as `Data/HA-R2R/train/train.json.gz` include `instruction_vocab.word_list`
- the public splits also provide pre-tokenized BERT-format files such as `train_bertidx.json.gz`, `val_seen_bertidx.json.gz`, and `val_unseen_bertidx.json.gz`

Recommended guidance:

- if your method uses a classic word-level vocabulary, first try the vocabulary bundled in the HA-R2R dataset files
- if your method uses a BERT-style text encoder, first try the provided `*_bertidx.json.gz` files
- only build or extend your own text pipeline if your method truly requires it

### 5. Integrate Human-Aware Metrics in Eval

After each episode ends, you can inject TCR, CR, and strict SR and strip high-dimensional fields before aggregation:

```python
from HASimulator.metric import Calculate_Metric

metric_calc = Calculate_Metric(split="val_unseen")
metric_calc(stats_episodes[ep_id], ep_id)

for key in ["distance_to_human", "collisions_detail"]:
    if key in stats_episodes[ep_id]:
        stats_info.setdefault(ep_id, {})[key] = stats_episodes[ep_id].pop(key)
```

Metric semantics:

- `TCR`: net new trajectory collisions after excluding the pre-computed unavoidable collision component
- `CR`: collision flag for the episode
- `SR`: strict success rate under the extra condition `TCR == 0`

### 6. Optional Human Counting and Visualization Hook

If `HUMAN_COUNTING: True`, ensure GroundingDINO is installed and the weight path is configured correctly in `detector.py` before evaluation.

You can also stitch observation frames and instruction text inside the evaluation loop for debugging videos.

```python
from copy import deepcopy
from habitat_extensions.utils import observations_to_image, generate_video
from habitat.utils.visualizations.utils import append_text_to_image

rgb_frames = [[] for _ in range(envs.num_envs)]

for i in range(envs.num_envs):
    if len(config.VIDEO_OPTION) > 0:
        if config.TASK_CONFIG.SIMULATOR.HUMAN_COUNTING:
            observations_ = deepcopy(observations)
            observations_[i]["rgb"] = detected_img[i]
            frame = observations_to_image(observations_[i], infos[i])
        else:
            frame = observations_to_image(observations[i], infos[i])

        frame = append_text_to_image(frame, current_episodes[i].instruction.instruction_text)
        rgb_frames[i].append(frame)

    if dones[i] and len(config.VIDEO_OPTION) > 0:
        ep_id = current_episodes[i].episode_id
        generate_video(
            video_option=config.VIDEO_OPTION,
            video_dir=config.VIDEO_DIR,
            images=rgb_frames[i],
            episode_id=ep_id,
            checkpoint_idx=checkpoint_index,
            metrics={"spl": stats_episodes[ep_id]["spl"]},
            tb_writer=writer,
        )
        rgb_frames[i] = []
```

### 7. API Cross-Reference

Use the API pages together with this integration guide:

- human distance, angle, and counting hooks: `api/human_state.md`
- clock sync and forced frame controls: `api/scene_updates.md`
- collision detail and strict human-aware metrics: `api/collision_checks.md`
