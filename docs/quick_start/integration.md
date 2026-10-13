## Agent Integration

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

The agent sits between the environment layer and API layer, making this page the bridge from runnable setup to custom integration.

This page extends the minimum setup with practical details from the original integration notes, including config switching, dynamic clock synchronization, vocabulary updates, strict metrics, and optional visualization hooks.

### 1. Redirect Base Task Config

Redirect `BASE_TASK_CONFIG_PATH` in agent configuration to the HA-VLN task config:

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

- `HAVLNCE_task.yaml` often contains relative paths such as `../Data/...`.
- These paths are resolved against the process CWD where your agent starts.
- If your training/eval launcher runs in another directory, switch those paths to absolute paths to avoid silent file-not-found failures.

### 2. Select the Experimental Baseline

For ablation studies, switch only `BASE_TASK_CONFIG_PATH` while keeping the rest of your agent code unchanged:

- `HAVLNCE_task.yaml`: Dynamic environment + HA-R2R instructions.
- `HAVLNCE_R2R_task.yaml`: Dynamic environment + original R2R instructions.
- `VLNCE_task.yaml`: Static environment + original R2R instructions.
- `VLNCE_HAR2R_task.yaml`: Static environment + HA-R2R instructions.

### 3. Implement Environment Wrapper Synchronization

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

- HA-VLN advances human motions using a child-thread clock.
- Without `_handle_signals()` before each step, policy actions may be evaluated on stale geometry / stale NavMesh.

### 4. Expand Vocabulary (if needed)

If you use a fixed vocabulary model (not an end-to-end tokenizer), run vocabulary expansion to avoid OOV tokens introduced by HA-R2R.

Minimal expansion logic:

```python
import json
import gzip
import re
from pathlib import Path

def clean_text(text: str):
    text = text.lower()
    text = re.sub(r'([.?!,;:/\\()\[\]"\'\-])', r' \1 ', text)
    text = re.sub(r'\s+', ' ', text).strip()
    return text.split()

def update_vocabulary(ha_r2r_dir: str, existing_vocab_path: str, output_vocab_path: str):
    with open(existing_vocab_path, 'r', encoding='utf-8') as f:
        existing_words = [line.strip() for line in f.readlines()]

    vocab_set = set(existing_words)
    new_words = []

    for split in ['train', 'val_seen', 'val_unseen']:
        json_file = Path(ha_r2r_dir) / split / f"{split}.json.gz"
        if not json_file.exists():
            continue

        with gzip.open(json_file, 'rt', encoding='utf-8') as f:
            data = json.load(f)

        for item in data:
            for instruction in item.get('instructions', []):
                for token in clean_text(instruction):
                    if token not in vocab_set:
                        vocab_set.add(token)
                        new_words.append(token)

    with open(output_vocab_path, 'w', encoding='utf-8') as f:
        for word in existing_words + new_words:
            f.write(f"{word}\n")
```

### 5. Integrate Strict Metrics in Eval Loop

After each episode ends, inject TCR/CR/SR and strip high-dimensional fields to avoid aggregation errors:

```python
from HASimulator.metric import Calculate_Metric

metric_calc = Calculate_Metric(split="val_unseen")
metric_calc(stats_episodes[ep_id], ep_id)

for key in ["distance_to_human", "collisions_detail"]:
    if key in stats_episodes[ep_id]:
        stats_info.setdefault(ep_id, {})[key] = stats_episodes[ep_id].pop(key)
```

Metric semantics:

- `TCR`: Net new trajectory collisions after excluding Oracle baseline scene collisions.
- `CR`: Collision flag for the episode.
- `SR`: Strict success rate under the extra condition `TCR == 0`.

### 6. Optional Human Counting and Visualization Hook

If `HUMAN_COUNTING: True`, ensure GroundingDINO is installed and the absolute weight path is correctly configured in `detector.py` before evaluation.

You can optionally stitch observation frames and instruction text inside the evaluation loop, then export MP4 videos for debugging and comparison.

```python
from copy import deepcopy
from habitat_extensions.utils import observations_to_image, generate_video
from habitat.utils.visualizations.utils import append_text_to_image

rgb_frames = [[] for _ in range(envs.num_envs)]

for i in range(envs.num_envs):
    if len(config.VIDEO_OPTION) > 0:
        if config.TASK_CONFIG.SIMULATOR.HUMAN_COUNTING:
            observations_ = deepcopy(observations)
            observations_[i]['rgb'] = detected_img[i]
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

- Human distance / angle and counting hooks: `api/human_state.md`
- Clock sync and forced frame controls: `api/scene_updates.md`
- Collision detail and strict TCR metrics: `api/collision_checks.md`
