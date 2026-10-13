## Human State Queries

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

This section provides APIs for querying dynamic human states during navigation. Typical use cases include safety-aware rewards, interaction policies, and behavior analysis.

**Simulator-state warning:** The direct distances, angles, and global human
coordinates below are privileged simulator state. They can be useful for
debugging and controlled experiments, but are not ordinary egocentric sensor
observations. Check the rules of any downstream benchmark before using them as
policy inputs.

### distance_to_human

- Purpose: Returns privileged distance and relative-angle diagnostics for
  visible humans.
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

- Purpose: Directly reads global absolute human coordinates and rotations, not
  limited by the agent FoV.
- Prerequisite: `ADD_HUMAN: True` and an initialized HAVLNCE helper.

```python
global_positions = env.havlnce_tool._sim._human_posisions
```

### human_counting

- Purpose: Uses GroundingDINO to count humans in the current egocentric view
  and returns rendered images with bounding boxes. Unlike the simulator-state
  fields above, this detector processes rendered RGB observations.
- Prerequisites:
  1. `HUMAN_COUNTING: True`
  2. Correct model weight path configured in `detector.py`

```python
from HASimulator.detector import Detector

detector = Detector().to(device)
stats_info = {}
detected_imgs = detector(observations, "human", current_episodes, stats_info)
```
