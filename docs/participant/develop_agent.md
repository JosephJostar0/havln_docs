# Develop Your Agent

This page is for participants who want to build their own agent on top of HA-VLN.

The main idea is straightforward:

- HA-VLN provides the simulator and the public data
- you provide your own navigation agent
- your agent interacts with the HA-VLN environment through the simulator/task configuration and available APIs

The repository `agent/` directory is reference code and baseline material. You may build your own agent code as long as final challenge scoring can replay the produced HA-VLN-CE primitive actions.

## What Participants Usually Need from the Simulator

When developing an agent, participants typically need to understand three things:

1. how to point their code to the HA-VLN task configuration
2. which simulator or task APIs expose dynamic-human information
3. which metrics and runtime behaviors matter during evaluation

## Start from the HA-VLN Task Configuration

The first integration step is to switch your agent to the HA-VLN task configuration.

See the detailed integration notes here:

- [Agent Integration Notes](../quick_start/integration.md)

In particular, check:

- `BASE_TASK_CONFIG_PATH`
- `SIMULATOR.ADD_HUMAN`
- `SIMULATOR.ALLOW_SLIDING`
- measurement fields used during evaluation

## Basic Environment Interaction

HA-VLN follows the usual Habitat-style episode loop: create the environment, call `reset()`, choose one primitive action at a time, and stop when the episode is done.

The current HA-VLN-CE challenge primitive actions are:

- `STOP`
- `MOVE_FORWARD`
- `TURN_LEFT`
- `TURN_RIGHT`

A minimal loop has this shape:

```python
observations = env.reset()
done = False

while not done:
    action = agent.act(observations)
    observations, reward, done, info = env.step(action)

metrics = info
```

`observations` contains the configured sensor outputs for the current episode. Physical simulator sensors are observations such as RGB and depth. Task sensors are navigation-task fields such as instruction or progress-related signals. Choose the observation set in your local task config according to what your method uses.

Oracle-style task sensors, such as shortest-path or oracle-progress sensors, are useful for debugging and some baselines. Treat them as privileged signals when choosing challenge-facing inputs.

The `info` dictionary contains episode metrics after each step. At the end of an episode, use it to inspect success, distance, collision-aware measurements, and other configured metrics. For metric details, see [Evaluation Metrics](../api/evaluation_metrics.md).

## API Categories You Will Likely Use

### 1. Human State APIs
Use these when your agent needs information related to nearby humans, distances, angles, or human-aware observations.

- [Human State Queries](../api/human_state.md)

### 2. Dynamic Scene Update APIs
Use these when your agent or wrapper needs to stay synchronized with the dynamic human timeline.

- [Dynamic Scene Updates](../api/scene_updates.md)

### 3. Collision and Safety APIs
Use these when you want to inspect collision-related behavior or build human-aware evaluation logic.

- [Collision Checks](../api/collision_checks.md)

### 4. Evaluation Metrics
Use these when you want to understand how success and collision-aware performance are measured.

- [Evaluation Metrics](../api/evaluation_metrics.md)

## Typical Development Workflow

A practical participant workflow is:

1. start from an existing VLN agent or your own new implementation
2. switch it to the HA-VLN task configuration
3. make sure it can run in the dynamic human environment
4. use the simulator APIs above to inspect human-aware state when needed
5. validate your behavior and metrics on public splits

## Important Development Notes

- HA-VLN is not limited to one specific agent architecture
- participants are expected to develop their own agents
- the repository `agent/` directory is optional reference material rather than required submission code
- the important challenge requirement is compatibility with the HA-VLN environment and the primitive action contract

If you need low-level integration details such as wrapper synchronization, provided text resources, or metric injection, read:

- [Agent Integration Notes](../quick_start/integration.md)

If your eventual goal is the current Docker challenge workflow, treat this page as the method-development step in a local environment. After your method runs locally, continue to the challenge pages and make sure your method can export `actions.json`.

## Next Step

After your agent can run inside HA-VLN, move on to:

- [Test Your Agent](test_agent.md)
- [Action Sequence Format](../challenge/action_sequence_format.md)
- [Agent Package And Local Validation](../challenge/agent_package_validation.md)
