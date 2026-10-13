## Collision Checks

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

This module provides finer-grained collision traceability than binary collision flags, with emphasis on separating human collisions from environment collisions and enabling strict evaluation.

**Evaluator-state warning:** Collision annotations and adjusted counts are
diagnostic outputs, not ordinary sensor observations. Use them for
post-episode analysis and debugging; consult any downstream benchmark's rules
before using privileged fields in a policy.

### collisions_detail

- Purpose: Returns privileged object-level collision details for evaluation and
  debugging.
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

- Purpose: Uses the metric implementation's pre-computed unavoidable collision
  component to compute the adjusted episode collision count, episode collision
  indicator, and strict success.
- When to call: Offline evaluation after an episode ends.

```python
from HASimulator.metric import Calculate_Metric

metric_calc = Calculate_Metric(split="val_unseen")
metric_calc(info, episode_id)

tcr = info["TCR"]
collision_indicator = info["CR"]
strict_sr = info["SR"]
```

The episode-level `CR` field is an indicator, not a dataset-level collision
rate. Dataset aggregation depends on the evaluation protocol being used.
