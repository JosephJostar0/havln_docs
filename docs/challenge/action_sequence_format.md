# Action Sequence Format

This page defines the action sequence file used by HA-VLN challenge evaluation.

In Phase 1, participants submit this file directly. In Phase 2, `run.sh` should generate the same file during inference.

The expected file name is:

```text
actions.json
```

For local validation, the default path is:

```text
/app/result/actions.json
```

## JSON Schema

`actions.json` must be a JSON object:

```json
{
  "format_version": 1,
  "episodes": [
    {
      "episode_id": "0",
      "trajectory_id": "5732",
      "scene_id": "mp3d/example_scene",
      "actions": ["MOVE_FORWARD", "TURN_LEFT", "MOVE_FORWARD", "STOP"]
    }
  ],
  "metadata": {
    "split": "val_unseen",
    "agent_name": "ExampleAgent"
  }
}
```

Required root fields:

- `format_version`: must be `1`
- `episodes`: non-empty list of episode action records
- `metadata`: object containing at least `split`

Required episode fields:

- `episode_id`: episode id from the official HA-VLN split
- `trajectory_id`: trajectory id from the official HA-VLN split
- `scene_id`: scene id from the official HA-VLN split
- `actions`: ordered list of primitive actions

Required metadata fields:

- `split`: the evaluated split, such as `val_unseen`

## Primitive Actions

Use action names in challenge submissions:

| Name | Code | Meaning |
|---|---:|---|
| `STOP` | `0` | stop the episode |
| `MOVE_FORWARD` | `1` | move forward by the official CE step size, currently 0.25m |
| `TURN_LEFT` | `2` | rotate left by the official CE turn angle, currently 15 degrees |
| `TURN_RIGHT` | `3` | rotate right by the official CE turn angle, currently 15 degrees |

Recommended format:

```json
"actions": ["MOVE_FORWARD", "TURN_LEFT", "STOP"]
```

The local loader also accepts integer codes:

```json
"actions": [1, 2, 0]
```

Action names are preferred because they are easier to inspect and less likely to be confused with boolean or framework-specific values.

## Validation Rules

For each episode:

- `actions` must be a list.
- the action count must be between `1` and `500`.
- `STOP`, if present, must be the final action.
- the replayed episode must reach simulator completion.
- actions after simulator completion are invalid.

For the file:

- `metadata.split` must match the evaluated split.
- every expected episode must be present.
- extra episode ids are invalid for official scoring.
- `episode_id`, `trajectory_id`, and `scene_id` must match the official dataset metadata.

## Convert Method Outputs to Primitive Actions

Your method may use poses, continuous trajectories, waypoints, high-level macro actions, or learned latent actions internally. The submitted file should contain the four primitive actions used by the current HA-VLN-CE challenge scorer.

## Validate An Action File Locally

To replay a local action file with the official scorer, mount or copy it to a container-visible path such as:

```text
/app/result/actions.json
```

Then run:

```bash
havln-score \
  --exp-config /app/official_scoring/config/challenge_submission.yaml \
  --actions-path /app/result/actions.json \
  --output-dir /app/result \
  RESULTS_DIR /app/result \
  LOG_FILE /app/result/challenge_eval.log
```

This is the local validation path for Phase 1 action files. It is also the replay step used after a Phase 2 package generates `actions.json`.

Successful replay writes score artifacts under the output directory.
