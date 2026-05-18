# HA-VLN Challenge

Challenge homepage: [Codabench](https://www.codabench.org/).

HA-VLN evaluates instruction-following navigation in indoor scenes with dynamic humans. Participants may use any agent architecture, planner, memory system, world model, or runtime. The challenge scoring input is an ordered sequence of HA-VLN-CE primitive actions.

## Challenge Phases

### Phase 1: Development And Offline Evaluation

Use Phase 1 to learn the environment, develop an agent, and compare methods quickly.

In this phase, participants submit an action sequence file. The scorer replays the submitted actions and writes the score artifacts.

Read [Action Sequence Format](action_sequence_format.md) for the file accepted by offline evaluation.

### Phase 2: Final Submission

Use Phase 2 for the final executable submission.

In this phase, participants submit an executable agent package. The package must include `run.sh`, inference code, required weights, and any setup needed by the method. During evaluation, the package is mounted at:

```text
/app/agent
```

The organizer runs:

```bash
bash /app/agent/run.sh
```

`run.sh` should prepare the runtime environment, start inference, and produce the same action sequence format used in Phase 1.

Read [Agent Package And Local Validation](agent_package_validation.md) for the Phase 2 package and Docker workflow.

## Fixed Action Interface

Both phases of the current HA-VLN-CE challenge workflow use the same primitive action interface:

- `STOP`
- `MOVE_FORWARD`
- `TURN_LEFT`
- `TURN_RIGHT`

If your method uses poses, continuous trajectories, waypoints, or high-level planner actions internally, export the final behavior as the primitive action sequence described above.

## Reading Path

- Start here to understand the two phases.
- Read [Action Sequence Format](action_sequence_format.md) before submitting Phase 1 results.
- Read [Agent Package And Local Validation](agent_package_validation.md) before preparing a Phase 2 package.
