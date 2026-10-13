## Dynamic Scene Updates

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

The dynamic scene is driven by a child-thread clock. This module provides APIs for timeline synchronization, forced frame jumps, and NavMesh recomputation.

### _handle_signals()

- Purpose: Consumes clock signals and triggers a full update cycle (remove old humans, insert new humans, update NavMesh).
- Call convention: Execute before each `sim.step()`.

```python
self.havlnce_tool._handle_signals()
```

### refresh_human_model(frame_id)

- Purpose: Forces the simulator to switch to a specific human motion frame.
- Parameter: `frame_id`, commonly in the range `0~119`.

```python
env.havlnce_tool.refresh_human_model(60)
```

### recompute_navmesh(...)

- Purpose: Low-level manual NavMesh recomputation.
- Note: CPU intensive. Prefer cached navmesh files during training.

```python
import habitat_sim

navmesh_settings = habitat_sim.nav.NavMeshSettings()
navmesh_settings.set_defaults()
navmesh_settings.agent_radius = 0.1
navmesh_settings.agent_height = 1.5

success = env.havlnce_tool._sim.recompute_navmesh(
	env.havlnce_tool._sim.pathfinder,
	navmesh_settings,
	include_static_objects=True,
)
```
