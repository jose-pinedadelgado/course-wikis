# Configuration

## Experiment YAML

Full schema for experiment configuration files.

```yaml
experiment:
  name: string          # Unique experiment identifier
  description: string   # Human-readable description

game:
  max_rounds: int       # Maximum rounds per game (default: 100)
  chat_phase: bool      # Enable pre-decision chat (default: false)
  protocol_mode: string # "unstructured" | "structured" | "structured_validated"
  payoff:
    R: int              # Reward — mutual cooperation (default: 3)
    T: int              # Temptation — unilateral defection (default: 5)
    S: int              # Sucker — unilateral cooperation (default: 0)
    P: int              # Punishment — mutual defection (default: 1)
  horizon:
    type: string        # "fixed" | "geometric" | "unknown"
    param: float        # For geometric: continuation probability

conditions:
  - name: string        # Unique condition name
    agent_a:
      ref: string       # Path to agent YAML
      overrides: dict   # Parameter overrides
    agent_b:
      ref: string
      overrides: dict
```

## Agent YAML

### Policy Agent

```yaml
type: "policy"
policy: "tft"  # allc | alld | tft | grim | gtft | wsls
params:
  forgiveness_prob: 0.1  # For GTFT only
```

### Personality Agent

```yaml
type: "personality"
personality: "naive_cooperator"
params:
  exploitation_threshold: 3  # Agent-specific params
```

## Available Agents

### Policy Agents

| Key | Strategy |
|---|---|
| `allc` | Always Cooperate |
| `alld` | Always Defect |
| `tft` | Tit-for-Tat |
| `grim` | Grim Trigger |
| `gtft` | Generous TFT (param: `forgiveness_prob`) |
| `wsls` | Win-Stay, Lose-Shift |

### Personality Agents

| Key | Archetype | Key Params |
|---|---|---|
| `naive_cooperator` | Helpful, unaware | `exploitation_threshold` |
| `aware_cooperator` | Game-aware, tracking | `window_size`, `threshold` |
| `strategic` | Opponent classifier | — |
| `deceptive` | Trust-then-exploit | `trust_rounds`, `exploit_rounds`, `rebuild_rounds` |
| `manipulative` | Chat exploiter | — |
| `cost_offloader` | Verbose chat, low effort | — |
