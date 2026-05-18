# Agent Package And Local Validation

This page describes the executable package used in Phase 2 and the local validation workflow.

For Phase 1, submit the action file described in [Action Sequence Format](action_sequence_format.md). The executable package described here is for Phase 2 final submission.

## Phase 2 Package

In Phase 2, the submitted package is mounted inside the official Docker image at:

```text
/app/agent
```

The package must provide:

- `run.sh`
- participant inference code
- model weights or other required resources
- dependency installation steps, if needed
- a way to write an action sequence file

The organizer starts inference with:

```bash
bash /app/agent/run.sh
```

`run.sh` is the participant-owned startup script. It should prepare the runtime environment, start inference, and write:

```text
/app/result/actions.json
```

Typical responsibilities include:

- activating the expected Python or Conda environment
- installing participant-specific dependencies, if needed
- setting method-specific environment variables or paths
- loading checkpoints or other required resources
- launching the inference script
- writing the final action sequence file

`run.sh` may call any participant-owned code path as long as it completes inference and produces the expected action file.

The action file must follow [Action Sequence Format](action_sequence_format.md).

## Recommended Package Layout

```text
agent/
|-- run.sh
|-- submission/
|   `-- your_method_files.py
|-- checkpoints/
`-- requirements.txt
```

The package layout may be different if your method needs another structure. The required entry point is `/app/agent/run.sh`, and the expected action file is `/app/result/actions.json`.

## Docker Mounts

For local validation, the Docker workflow mounts:

- local challenge data to `/app/Data:rw`
- local agent package to `/app/agent:rw`
- local output directory to `/app/result:rw`

Pull the official image:

```bash
docker pull ghcr.io/josephjostar0/havln-eval-image:latest
```

Create a local compose file from the repository template:

```bash
cp docker-compose-template.yml docker-compose.yml
```

Edit the host-side paths in `docker-compose.yml`, then start the container:

```bash
docker compose up -d
docker compose exec evaluator bash
```

## Local Validation

### Phase 1: Replay an Existing Action File

Place or mount the action file at:

```text
/app/result/actions.json
```

Replay it:

```bash
python /app/official_scoring/eval_offline.py \
  --exp-config /app/official_scoring/config/challenge_submission.yaml \
  --actions-path /app/result/actions.json \
  --output-dir /app/result \
  RESULTS_DIR /app/result \
  LOG_FILE /app/result/challenge_eval.log
```

### Phase 2-Style: Run the Package, Then Replay

Run the package:

```bash
bash /app/agent/run.sh
```

Confirm that the action file exists:

```bash
ls /app/result/actions.json
```

Then replay it:

```bash
python /app/official_scoring/eval_offline.py \
  --exp-config /app/official_scoring/config/challenge_submission.yaml \
  --actions-path /app/result/actions.json \
  --output-dir /app/result \
  RESULTS_DIR /app/result \
  LOG_FILE /app/result/challenge_eval.log
```

## Expected Outputs

A successful replay writes:

- `actions.json`
- `score_summary.json`
- `episode_metrics.json`
- `run_metadata.json`
- `challenge_eval.log`

## Common Issues

### `actions.json` Is Missing

For Phase 1 local validation, check that the file is mounted or copied to the path passed through `--actions-path`.

For Phase 2 package validation, check that `run.sh` writes `/app/result/actions.json`.

### Split Or Episode Metadata Mismatch

Make sure `metadata.split`, `episode_id`, `trajectory_id`, and `scene_id` match the official split being evaluated.

### Invalid Action

Only `STOP`, `MOVE_FORWARD`, `TURN_LEFT`, and `TURN_RIGHT` are accepted.

### Replay Does Not Finish

The replayed simulator episode must reach completion. In normal HA-VLN-CE evaluation, each episode should end with `STOP`.

### Docker Or GPU Startup Fails

If you switch from Docker Compose to manual `docker run`, carry over the graphics-related environment variables from `docker-compose-template.yml`. `--gpus all` alone is not enough for reliable Habitat/EGL startup.
