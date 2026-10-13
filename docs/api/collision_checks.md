## Collision Checks

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This module provides finer-grained collision traceability than binary collision flags, with emphasis on separating human collisions from environment collisions and enabling strict evaluation.

### collisions_detail

- Purpose: Returns object-level collision details for the current step.
- Prerequisite: Enable `COLLISIONS_DETAIL` in task measurements.

```python
observations, reward, done, info = env.step(action)

if "collisions_detail" in info:
	details = info["collisions_detail"]
	if any("human" in str(obj).lower() for obj in details):
		print("Warning: Human collision detected")
		done = True
```

### Calculate_Metric

- Purpose: Uses Oracle baseline collision statistics to compute net new collision rate (TCR), CR, and strict SR.
- When to call: Offline evaluation after an episode ends.

```python
from HASimulator.metric import Calculate_Metric

metric_calc = Calculate_Metric(split="val_unseen")
metric_calc(info, episode_id)

tcr = info["TCR"]
cr = info["CR"]
strict_sr = info["SR"]
```
