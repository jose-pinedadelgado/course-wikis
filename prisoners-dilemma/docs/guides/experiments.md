# Running Experiments

## Experiment Configuration

Experiments are defined in YAML files in the `configs/` directory.

### Basic Structure

```yaml
experiment:
  name: "phase2_baseline"
  description: "Personality agent baseline comparisons"
  
game:
  max_rounds: 100
  chat_phase: true
  protocol_mode: "unstructured"  # or "structured"
  payoff:
    R: 3  # Reward (mutual cooperation)
    T: 5  # Temptation (defect while opponent cooperates)
    S: 0  # Sucker (cooperate while opponent defects)
    P: 1  # Punishment (mutual defection)

conditions:
  - name: "Naive_vs_Deceptive"
    agent_a:
      ref: "agents/personality.yaml"
      overrides: {personality: "naive_cooperator"}
    agent_b:
      ref: "agents/personality.yaml"
      overrides: {personality: "deceptive"}
  
  - name: "Strategic_vs_Manipulative"
    agent_a:
      ref: "agents/personality.yaml"
      overrides: {personality: "strategic"}
    agent_b:
      ref: "agents/personality.yaml"
      overrides: {personality: "manipulative"}
```

## Running

```bash
# Validate config first
uv run pd2 validate configs/experiment_phase2.yaml

# Run with default replicates
uv run pd2 run configs/experiment_phase2.yaml

# Specify replicates
uv run pd2 run configs/experiment_phase2.yaml --replicates 15

# Run specific condition only
uv run pd2 run configs/experiment_phase2.yaml --condition "Naive_vs_Deceptive"
```

## Output

Results are saved to `data/runs/<experiment-name>/`:

```
data/runs/phase2_baseline/
├── rounds.jsonl          # Every round of every game
├── aggregates.parquet    # Per-run summary statistics
└── run_manifest.json     # Experiment metadata
```

## Interpreting Results

### Key Questions per Condition

1. **Who won?** — Compare average payoffs
2. **How did cooperation evolve?** — Plot cooperation rate over rounds
3. **Was there exploitation?** — Check exploitation window metric
4. **Did chat help or hurt?** — Compare deception success rate with chat consistency
5. **Did the protocol matter?** — Compare violation counts across protocol modes

### Quick Analysis

```python
import pandas as pd

df = pd.read_parquet("data/runs/phase2_baseline/aggregates.parquet")
print(df.groupby("condition")[["cooperation_rate_a", "cooperation_rate_b", "avg_payoff_a", "avg_payoff_b"]].mean())
```
