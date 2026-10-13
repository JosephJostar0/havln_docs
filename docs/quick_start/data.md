# Data Download

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

This page explains how to acquire and prepare all datasets, scene meshes, and model weights required for HA-VLN.

---

## 1. Directory Layout

All scene meshes, human activities, episodes, and baseline checkpoints reside under the `Data/` directory:

```text
Data/
├── HA-R2R/
│   ├── train/train_bertidx.json.gz
│   ├── val_seen/val_seen_bertidx.json.gz
│   └── val_unseen/val_unseen_bertidx.json.gz
├── HA-R2R-tools/
│   ├── collision_num_val_seen.json
│   └── collision_num_val_unseen.json
├── Multi-Human-Annotations/
│   └── human_motion.json
├── HAPS2_0/
│   └── <dynamic_human_motion_glbs>
├── scene_datasets/
│   └── mp3d/<scan>/<scan>.glb
└── ddppo-models/
    └── gibson-2plus-resnet50.pth  # optional for depth encoder
```

---

## 2. Matterport3D Scene Meshes (`Data/scene_datasets`)

Matterport3D is licensed separately by Matterport and cannot be redistributed.

1. Request access on the [Matterport3D Project Page](https://niessner.github.io/Matterport/) and submit the signed Terms of Use to receive your personal `download_mp.py` script.
2. Download and extract the Habitat scene meshes:

```bash
python3 /path/to/download_mp.py -o Data/scene_datasets --task_data habitat
# After task-data download finishes, press Ctrl-C at the prompt for the main dataset.
unzip Data/scene_datasets/v1/tasks/mp3d_habitat.zip -d Data/scene_datasets
```

Verify that scene meshes reside at `Data/scene_datasets/mp3d/<scan>/<scan>.glb`.

---

## 3. HA-VLN Simulation Assets & Annotations

Simulation assets (HAPS 2.0 dynamic human motion models, HA-R2R episodes, and baseline weights) are officially hosted on [**Hugging Face (fly1113/HA-VLN)**](https://huggingface.co/datasets/fly1113/HA-VLN). Multi-human placement metadata (`human_motion.json`) and collision evaluation baselines are fetched from the repository.

### Source 1: Hugging Face Hub (Recommended)

The verified downloader automatically fetches all assets, unzips and normalizes HAPS 2.0 meshes, and retrieves collision baselines and human annotations:

```bash
python scripts/download_hf.py --destination Data --target all
```

<details>
<summary><b>Manual Hugging Face CLI Download</b></summary>
<br>

```bash
pip install huggingface-hub
hf download fly1113/HA-VLN --repo-type dataset --local-dir Data
```

*Note: Raw HF Hub download leaves `HAPS2_0.zip` unextracted. If using this option, you must manually unpack `HAPS2_0.zip` (flattening `human_motion_glbs_v3` into `Data/HAPS2_0`), and retrieve `Multi-Human-Annotations/human_motion.json` and `Data/HA-R2R-tools/collision_num_val_*.json` from the repository.*
</details>

### Source 2: Google Drive (Legacy Compatibility)

<details>
<summary><b>Google Drive Mirror (Legacy Fallback)</b></summary>
<br>

```bash
pip install gdown
bash scripts/download_data.sh
```

*Note: Google Drive mirror is maintained strictly for backward compatibility. For reproducible downloads with checksum verification and automated HAPS 2.0 extraction, use the Hugging Face source above.*
</details>

---

## 4. Pretrained Depth Encoder Weights (Optional for CMA)

If training or evaluating CMA from scratch with depth observations, download the pretrained DD-PPO ResNet-50 weights:

```bash
curl -fL https://dl.fbaipublicfiles.com/habitat/data/baselines/v1/ddppo/ddppo-models.zip -o ddppo-models.zip
# Extract directly into ddppo-models without creating nested directories:
unzip -j ddppo-models.zip "data/ddppo-models/gibson-2plus-resnet50.pth" -d Data/ddppo-models
rm ddppo-models.zip
```

---

## 5. GroundingDINO Weights (Optional for Human Counting)

If `HUMAN_COUNTING` is enabled in `HASimulator/detector.py`:

```bash
mkdir -p HASimulator/GroundingDINO/weights
curl -fL --retry 3 \
  https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth \
  -o HASimulator/GroundingDINO/weights/groundingdino_swint_ogc.pth
```

---

## 6. Vocabulary Expansion (for static vocab agents)

If your agent uses a static vocabulary file, expand it from HA-R2R corpora to avoid OOV tokens:

```python
import json
import gzip
import re
from pathlib import Path

def clean_text(text: str) -> list:
    text = text.lower()
    text = re.sub(r'([.?!,;:/\\()\[\]"\'\-])', r' \1 ', text)
    text = re.sub(r'\s+', ' ', text).strip()
    return text.split()

def update_vocabulary(ha_r2r_dir: str, existing_vocab_path: str, output_vocab_path: str):
    ha_r2r_path = Path(ha_r2r_dir)

    with open(existing_vocab_path, 'r', encoding='utf-8') as f:
        existing_words = [line.strip() for line in f.readlines()]

    vocab_set = set(existing_words)
    new_words = []

    for split in ['train', 'val_seen', 'val_unseen']:
        json_file = ha_r2r_path / split / f"{split}.json.gz"
        if not json_file.exists():
            continue

        with gzip.open(json_file, 'rt', encoding='utf-8') as f:
            data = json.load(f)

        for item in data:
            for instruction in item.get('instructions', []):
                tokens = clean_text(instruction)
                for token in tokens:
                    if token not in vocab_set:
                        vocab_set.add(token)
                        new_words.append(token)

    with open(output_vocab_path, 'w', encoding='utf-8') as f:
        for word in existing_words + new_words:
            f.write(f"{word}\n")
```
