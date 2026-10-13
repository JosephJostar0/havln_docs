## Human State Queries

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This section provides APIs for querying dynamic human states during navigation. Typical use cases include safety-aware rewards, interaction policies, and behavior analysis.

### distance_to_human

- Purpose: Returns distance and relative angle between the agent and visible humans.
- Prerequisite: Enable `DISTANCE_TO_HUMAN` in task measurements.
- Return format: `[{"distance": float, "angle": float}, ...]`.

```python
observations, reward, done, info = env.step(action)

if "distance_to_human" in info:
    human_states = info["distance_to_human"]
    min_distance = min([h["distance"] for h in human_states]) if human_states else float("inf")
    if min_distance < 0.5:
        reward -= 5.0
```

### _human_posisions

- Purpose: Directly reads global absolute human coordinates and rotations, not limited by the agent FoV.
- Prerequisite: `ADD_HUMAN: True` and an initialized HAVLNCE helper.

```python
global_positions = env.havlnce_tool._sim._human_posisions
```

### human_counting

- Purpose: Uses GroundingDINO to count humans in the current view and returns rendered images with bounding boxes.
- Prerequisites:
  1. `HUMAN_COUNTING: True`
  2. Correct model weight path configured in `detector.py`

```python
from HASimulator.detector import Detector

detector = Detector().to(device)
stats_info = {}
detected_imgs = detector(observations, "human", current_episodes, stats_info)
```
