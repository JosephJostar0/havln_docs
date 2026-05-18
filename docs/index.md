# HA-VLN Participant Documentation

HA-VLN provides the simulator stack, public data, and challenge workflow for human-aware vision-language navigation.

This documentation has two main paths.

## Use the HA-VLN Environment

Use these pages if you want to install HA-VLN, prepare data, and build an agent against the environment.

1. [Environment & Data](participant/install_and_data.md)
2. [Develop Your Agent](participant/develop_agent.md)
3. [Test Your Agent](participant/test_agent.md)

Reference pages for environment development:

- [Dependencies](quick_start/dependencies.md)
- [Installation Steps](quick_start/installation.md)
- [Data Download](quick_start/data.md)
- [Agent Integration Notes](quick_start/integration.md)
- [Simulator and metric APIs](#reference-material)

## Join the Challenge

Use these pages if you want to submit to the HA-VLN challenge.

1. [How To Participate](challenge/overview.md)
2. [Action Sequence Format](challenge/action_sequence_format.md)
3. [Agent Package And Local Validation](challenge/agent_package_validation.md)

The challenge has two phases:

- Phase 1 submits an `actions.json` action sequence file.
- Phase 2 submits an executable package mounted at `/app/agent`; `run.sh` writes the action sequence file.

Challenge homepage: [Codabench](https://www.codabench.org/).

## Reference Material

If you need more detail, the reference pages are still available:

- [Human State](api/human_state.md)
- [Scene Updates](api/scene_updates.md)
- [Collision Checks](api/collision_checks.md)
- [Evaluation Metrics](api/evaluation_metrics.md)
