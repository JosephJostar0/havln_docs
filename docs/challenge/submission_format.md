# Submission Format Specification

Official HA-VLN repository: https://github.com/JostarXiong/HA-VLN

This document specifies the official format required for the RoboWorld 2026 Track 2 (HA-VLN) Challenge submissions.

---

## 1. Submission Bundle Overview

Submissions are packaged as a single flat ZIP archive (e.g. `submission.zip`) containing two JSON prediction files:

```text
submission.zip
├── val_seen.json
└── val_unseen.json
```

Do not nest files within subdirectories inside the ZIP archive. Create the archive using:

```bash
zip -j submission.zip val_seen.json val_unseen.json
```

---

## 2. Action Vocabulary

The official simulator replay environment accepts six discrete string action literals:

| Action Literal | Action Name | Description |
|---|---|---|
| `"STOP"` | Stop | Terminates navigation and ends the episode. |
| `"MOVE_FORWARD"` | Move Forward | Moves the agent forward by 0.25 meters. |
| `"TURN_LEFT"` | Turn Left | Rotates agent heading left by 15.0 degrees. |
| `"TURN_RIGHT"` | Turn Right | Rotates agent heading right by 15.0 degrees. |
| `"LOOK_UP"` | Look Up | Pitches agent sensor upward by 30.0 degrees. |
| `"LOOK_DOWN"` | Look Down | Pitches agent sensor downward by 30.0 degrees. |

*Note: Baselines that only use the four planar actions (`STOP`, `MOVE_FORWARD`, `TURN_LEFT`, `TURN_RIGHT`) are completely valid, as the four planar actions form a legal subset of the six-action vocabulary.*

---

## 3. JSON Schema Specification

Both `val_seen.json` and `val_unseen.json` must adhere to format version 1:

```json
{
  "format_version": 1,
  "split": "val_seen",
  "episodes": [
    {
      "episode_id": "0",
      "actions": [
        "MOVE_FORWARD",
        "TURN_LEFT",
        "MOVE_FORWARD",
        "STOP"
      ]
    },
    {
      "episode_id": "1",
      "actions": [
        "TURN_RIGHT",
        "MOVE_FORWARD",
        "STOP"
      ]
    }
  ]
}
```

### Episode Constraints

1. **`episode_id`**: String representing the unique episode ID matching the evaluation split.
2. **`actions`**: Array of strings from the 6-action vocabulary.
3. **Length**: Between 1 and 500 actions inclusive (`1 <= len(actions) <= 500`).
4. **Termination**:
   - If the sequence length is `< 500`, it must end with `"STOP"`.
   - `"STOP"`, if present, must appear exactly once as the final element of the array.
   - If the sequence reaches the maximum budget of 500 steps, `"STOP"` may be omitted if terminated by the step limit.

### Split Coverage Requirements (Phase 1)

Every submission must contain predictions for all episodes in both validation splits:

- `val_seen`: exactly **778** episodes (across 259 distinct trajectories).
- `val_unseen`: exactly **1,839** episodes (across 613 distinct trajectories).

---

## 4. Local Validation

Before submitting to CodaBench, validate your ZIP bundle locally using the official verification CLI in the evaluation Docker container:

```bash
IMAGE=ghcr.io/jostarxiong/havln-challenge-2026@sha256:78a62cd176d2fd7d0e2825f4cb5be2488ebc5f1a354649b7b4f536a98f1054f4

# Run validation from host via Docker:
docker run --rm -v "$(pwd):/workspace" "$IMAGE" havln-validate /workspace/submission.zip
```

*(Note: If working directly inside an interactive evaluation container, you can execute `havln-validate submission.zip` directly).*

The validator verifies:
- Archive structure and valid JSON syntax
- `format_version == 1` and correct `split` tags
- Valid action string values within the 6-action vocabulary
- Exact episode ID coverage for all required episodes

---

## 5. Submission to CodaBench

Once validated, submit your `submission.zip` to the [RoboWorld 2026 Track 2 CodaBench Competition](https://www.codabench.org/competitions/18135/).