# Evaluation Metrics & Composite Score

Official HA-VLN repository: https://github.com/UWMILab/HA-VLN

This document explains the evaluation metrics used in the HA-VLN benchmark and the multi-objective scoring formula.

---

## 1. Core Navigation & Social Metrics

HA-VLN evaluates agents on both navigation task completion and social safety in human-populated environments:

### Success Rate (SR)

- **Definition**: Percentage of episodes completed successfully.
- **Goal Criterion**: The agent terminates navigation (`STOP`) within a 3.0-meter radius of the target goal location.
- **Strict Success Criterion**: In strict evaluation, an episode is successful if the goal criterion is met AND zero collisions with dynamic humans occurred ($s_i \mathbf{1}[e_i = 0]$).
- **Optimization**: Higher is better ($\max = 1.0$ or $100\%$).

### Navigation Error (NE)

- **Definition**: Mean Euclidean distance (in meters) between the agent's final stopping position and the nearest goal location.
- **Measurement**: Computed via `DistanceToGoal` ($d_i$).
- **Optimization**: Lower is better ($\min = 0.0\,\mathrm{m}$).
- *Note on Progress Monitor*: During agent training (e.g., in the HA-VLN-CMA baseline), the progress monitor module is supervised using normalized *geodesic distance* to goal via `VLNOracleProgressSensor`. Final evaluation NE is strictly the *Euclidean distance* at episode termination.

### Total Collision Rate (TCR)

- **Definition**: Average number of collisions with dynamic humans per episode.
- **Calculation**: Counts contacts with human 3D meshes within a 1.0-meter proximity envelope, adjusting for unavoidable collisions:
  $$
  \mathrm{TCR} = \frac{1}{L}\sum_{i=1}^{L}e_i
  $$
- **Optimization**: Lower is better ($\min = 0.0$).

### Collision Rate (CR)

- **Definition**: Percentage of human-influenced episodes in which at least one collision occurred:
  $$
  \mathrm{CR} = \frac{\sum_{i=1}^{L}\min(e_i, 1)}{\beta L}
  $$
  where $\beta L$ is the number of human-influenced episodes in the split.
- **Optimization**: Lower is better ($\min = 0.0$).

---

## 2. Multi-Objective Composite Score

To comprehensively evaluate embodied agents across navigation effectiveness and human safety, HA-VLN adopts a multi-objective **Composite Score**:

### Sub-Score Formulations

$$
\begin{aligned}
U_{\mathrm{NE}} &= \frac{3}{3 + \mathrm{NE}} \\
U_{\mathrm{TCR}} &= \frac{1}{1 + \mathrm{TCR}} \\
\mathrm{Navigation} &= 0.80 \times \mathrm{SR} + 0.20 \times U_{\mathrm{NE}} \\
\mathrm{Social} &= 0.75 \times (1 - \mathrm{CR}) + 0.25 \times U_{\mathrm{TCR}}
\end{aligned}
$$

### Final Composite Score

$$
\mathrm{Score} = 100 \times \mathrm{Navigation} \times (0.70 + 0.30 \times \mathrm{Social})
$$

- **Range**: $0.0 \le \mathrm{Score} \le 100.0$.
- **Evaluation Ranking**: Models are evaluated and ranked primarily by **Composite Score** (descending), followed by individual component metrics ($\mathrm{SR}$, $\mathrm{NE}$, $\mathrm{CR}$, $\mathrm{TCR}$).

---

## 3. Baseline Validation Benchmark (HA-VLN-CMA)

Validation benchmark evaluations of the public CMA checkpoint produced:

| Split | SR ↑ | NE (m) ↓ | CR ↓ | TCR ↓ | Score ↑ |
|---|---|---|---|---|---|
| `val_seen` | 0.165 | 6.230 | 0.638 | 13.271 | 15.469585 |
| `val_unseen` | 0.114 | 6.502 | 0.689 | 22.352 | 11.944822 |

*Note: Score is calculated from unrounded metrics; displayed component metrics are rounded. Values may differ slightly from other reported CMA runs because of checkpoint, runtime, or evaluation details.*

---

## 4. Related Guides & References

- [HA-VLN GitHub Repository](https://github.com/UWMILab/HA-VLN)
- [Environment Setup & Installation](../quick_start/installation.md)
- [Dataset Download & Preparation](../quick_start/data.md)
- [Agent Integration](../quick_start/integration.md)