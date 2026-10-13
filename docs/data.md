# Public Data

The [HA-VLN 2.0 repository](https://github.com/UWMILab/HA-VLN) contains
the released code and a helper for downloading the public HA-R2R and HAPS 2.0
archives. From the repository root, run:

```bash
bash scripts/download_data.sh
```

This helper downloads only those two archives. Other repository data,
including `Data/HA-R2R-tools/` and
`Data/Multi-Human-Annotations/human_motion.json`, is part of the source
checkout; do not expect the download helper to create it.

Matterport3D is separately licensed. Request access from the
[official Matterport3D site](https://niessner.github.io/Matterport/), then use
the HA-VLN repository's `download_mp.py` according to your authorized access
and the repository instructions. The downloaded scenes are not supplied by
this documentation repository.

Some baseline configurations also require pretrained depth-encoder weights.
The HA-VLN README links to the
[DD-PPO weights archive](https://dl.fbaipublicfiles.com/habitat/data/baselines/v1/ddppo/ddppo-models.zip)
and describes extraction under `Data/ddppo-models/`.

For dataset layout and model-specific run commands, consult the
[upstream README](https://github.com/UWMILab/HA-VLN#-download-dataset).
