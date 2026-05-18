# Test Your Agent

This page is for participants who already have a runnable HA-VLN agent and want to validate it on the public workflow before submission.

The challenge-specific check depends on the phase:

- for Phase 1, your method should produce a valid primitive action file
- for Phase 2, your `/app/agent` package should run from `bash /app/agent/run.sh` and produce that same action file

## What to Test

Before considering your agent ready, test the following:

- the agent can start correctly in the HA-VLN environment
- required data paths are resolved correctly
- the agent behaves correctly in dynamic human scenes
- the agent can export CE primitive actions
- the official action replay scorer can replay those actions and write metrics

## Recommended Testing Path

### 1. Test on public development splits

Run your agent on a public split such as `val_unseen` first. This catches:

- path issues
- configuration mistakes
- broken environment synchronization
- missing dependencies
- invalid action export

### 2. Inspect collision-aware behavior

Because HA-VLN is human-aware, test not only navigation success but also collision-related behavior. Use:

- [Collision Checks](../api/collision_checks.md)
- [Evaluation Metrics](../api/evaluation_metrics.md)

### 3. Validate action replay

After your method writes `/app/result/actions.json`, run:

```bash
havln-score \
  --exp-config /app/official_scoring/config/challenge_submission.yaml \
  --actions-path /app/result/actions.json \
  --output-dir /app/result \
  RESULTS_DIR /app/result \
  LOG_FILE /app/result/challenge_eval.log
```

This is the official local check for whether the submitted action sequence can be scored.

## `actions.json` Check

Use [Action Sequence Format](../challenge/action_sequence_format.md) for the full schema. Before replay, confirm that:

- `format_version` is `1`
- `metadata.split` matches the config split
- each episode has `episode_id`, `trajectory_id`, `scene_id`, and `actions`
- all actions are `STOP`, `MOVE_FORWARD`, `TURN_LEFT`, or `TURN_RIGHT`
- `STOP`, when present, is the final action
- each action sequence is able to finish the simulator episode

## Expected Outputs

After successful replay, `/app/result` should contain:

- `actions.json`
- `score_summary.json`
- `episode_metrics.json`
- `run_metadata.json`
- `challenge_eval.log`

## Common Failure Patterns

### Config or path mismatch

If the environment boots but evaluation fails quickly, check:

- the task config path used by your method
- checkpoint paths inside the mounted package
- host-side absolute paths that should be container paths
- whether `metadata.split` matches `EVAL.SPLIT`

### Invalid action sequence

The scorer accepts only:

- `STOP`
- `MOVE_FORWARD`
- `TURN_LEFT`
- `TURN_RIGHT`

Action count must be between `1` and `500`.

### Episode identity mismatch

If scoring reports an episode mismatch, check the `episode_id`, `trajectory_id`, and `scene_id` fields in `actions.json`.

### Replay does not finish

The replayed simulator episode must reach `done=True`. In practice, ensure the action sequence ends with `STOP`.

### Metrics look surprising

If plain success looks reasonable but challenge-facing `SR` looks low, revisit:

- [Evaluation Metrics](../api/evaluation_metrics.md)
- [Collision Checks](../api/collision_checks.md)

`SR` is stricter than plain environment `success` because it also requires zero adjusted human-collision count.

## If You Need More Detail

- [How To Participate](../challenge/overview.md)
- [Action Sequence Format](../challenge/action_sequence_format.md)
- [Agent Package And Local Validation](../challenge/agent_package_validation.md)
- [Agent Integration Notes](../quick_start/integration.md)
- [Evaluation Metrics](../api/evaluation_metrics.md)
