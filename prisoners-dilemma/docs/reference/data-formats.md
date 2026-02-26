# Data Formats

## rounds.jsonl

One JSON object per line, one line per round:

```json
{
    "round": 1,
    "action_a": "C",
    "action_b": "D",
    "payoff_a": 0,
    "payoff_b": 5,
    "chat_a": "Let's cooperate!",
    "chat_b": null,
    "protocol_violations": 0
}
```

| Field | Type | Description |
|---|---|---|
| `round` | int | Round number (1-indexed) |
| `action_a` | "C" \| "D" | Agent A's action |
| `action_b` | "C" \| "D" | Agent B's action |
| `payoff_a` | float | Agent A's payoff |
| `payoff_b` | float | Agent B's payoff |
| `chat_a` | string \| null | Agent A's pre-decision message |
| `chat_b` | string \| null | Agent B's pre-decision message |
| `protocol_violations` | int | Violations in this round |

## aggregates.parquet

Per-run summary metrics in Apache Parquet format:

| Column | Type | Description |
|---|---|---|
| `experiment` | string | Experiment name |
| `condition` | string | Condition name |
| `replicate` | int | Replicate number |
| `seed` | int | Random seed used |
| `cooperation_rate_a` | float | Agent A cooperation rate |
| `cooperation_rate_b` | float | Agent B cooperation rate |
| `mutual_cooperation_rate` | float | Fraction of (C,C) outcomes |
| `avg_payoff_a` | float | Agent A mean payoff |
| `avg_payoff_b` | float | Agent B mean payoff |
| `payoff_gap` | float | avg_payoff_a − avg_payoff_b |
| `deception_success_rate` | float | Chat-action inconsistency exploitation |
| `chat_consistency_a` | float | Agent A chat-action correlation |
| `chat_consistency_b` | float | Agent B chat-action correlation |
| `exploitation_window_a` | int | Max consecutive exploitations by A |
| `exploitation_window_b` | int | Max consecutive exploitations by B |
| `trust_recovery_time` | float | Avg rounds to recover cooperation |
| `protocol_violations` | int | Total violations in the run |
| `total_rounds` | int | Actual rounds played |

## run_manifest.json

```json
{
    "experiment": "phase2_baseline",
    "conditions": ["Naive_vs_Deceptive", "Strategic_vs_Manipulative"],
    "replicates": 15,
    "base_seed": 42000,
    "started_at": "2026-02-25T15:30:00Z",
    "completed_at": "2026-02-25T15:45:00Z",
    "total_runs": 30,
    "config_hash": "sha256:abc123..."
}
```
