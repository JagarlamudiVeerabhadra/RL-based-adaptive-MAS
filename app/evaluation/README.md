# Evaluation & Reward Layer (`app/evaluation/`)

The **Evaluation & Reward Layer** computes quantitative metrics, structural graph scores, cost benchmarks, and decoupled scalar rewards for offline Meta-RL training and online qualitative monitoring.

---

## 📌 Multi-Objective Reward & Evaluation System

### 1. Decoupled Dual-Objective Reward Formulation
To prevent high-capacity LLMs from masking bloated, redundant topologies, AMAS computes decoupled reward signals:
$$R_{\text{total}} = R_{\text{arch}} + R_{\text{resp}} + R_{\text{feedback}}$$

Where:
- **$R_{\text{arch}}$ (Architectural Objective)**:
  $$R_{\text{arch}} = w_1 \cdot C_{\text{cov}} + w_2 \cdot C_{\text{struct}} + B_{\text{simplification}} - P_{\text{bloat}}$$
  - $C_{\text{cov}} \in [0.0, 1.0]$: Capability coverage score against required capabilities.
  - $C_{\text{struct}} \in [0.0, 1.0]$: Topological compactness score.
  - $B_{\text{simplification}} = +0.20$: Bonus for excising redundant nodes while maintaining 100% coverage.
  - $P_{\text{bloat}} = -0.15$: Penalty for activating superfluous agents.
- **$R_{\text{resp}}$ (Response Objective)**:
  Evaluates factual completeness, structural soundness, and adherence to constraints ($0.0 - 1.0$).
- **$R_{\text{feedback}}$ (Human-in-the-Loop)**:
  Adjusted via UI thumbs-up / feedback ratings ($\pm 0.50$).

---

## 📂 Module Files

| File | Core Classes | Description |
| :--- | :--- | :--- |
| [`reward.py`](reward.py) | `DualObjectiveRewardCalculator`, `RewardCalculator` | Computes scalar rewards, penalties for invalid graph transitions, and multi-objective balances. |
| [`metrics_logger.py`](metrics_logger.py) | `MetricsLogger` | Dual logger streaming metrics to TensorBoard (`runs/session_...`), JSONL files, and MongoDB Atlas. |
| [`cost_tracker.py`](cost_tracker.py) | `CostTracker` | Estimates token consumption and USD expenses for baseline vs adapted architectures. |
| [`evaluator.py`](evaluator.py) | `ArchitectureEvaluator` | Computes graph connectivity, agent activity ratio, redundancy, and structural validity. |
| [`task_performance.py`](task_performance.py) | `TaskPerformanceEvaluator` | Deterministic offline proxy assessing capability-to-role matching without making LLM calls. |
| [`theoretical_evaluator.py`](theoretical_evaluator.py) | `TheoreticalArchitectureEvaluator` | Theoretical graph metrics including critical path length ($L_{\text{crit}}$) and Pareto utility. |
| [`multi_task_evaluation.py`](multi_task_evaluation.py) | `run_multi_task_evaluation()` | Demonstrates topological divergence across task families from identical initial baselines. |

---

## 📈 TensorBoard Integration & Telemetry

`MetricsLogger` streams real-time scalar metrics during training and live execution:
- `Reward/Total`, `Reward/Architectural`, `Reward/Response`
- `Architecture/ActiveAgentsCount`, `Architecture/PruningRate`
- `Efficiency/CriticalPathLength`, `Efficiency/TokenSavingsPct`
- `Cost/TotalTokens`, `Cost/EstimatedUSD`

Launch dashboard via:
```powershell
tensorboard --logdir runs
```
