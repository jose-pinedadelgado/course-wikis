# Game Engine

The game engine orchestrates PD matches: pairing agents, running rounds, computing payoffs, and managing horizons.

## Payoff Matrix

Default payoff values (classic PD with standard ordering):

| | Opponent: C | Opponent: D |
|---|---|---|
| **Player: C** | R=3, R=3 | S=0, T=5 |
| **Player: D** | T=5, S=0 | P=1, P=1 |

**Constraint:** $T > R > P > S$ and $2R > T + S$ (ensures mutual cooperation is optimal over time).

```python
class PayoffMatrix:
    def __init__(self, R=3, T=5, S=0, P=1):
        self.matrix = {
            (Action.COOPERATE, Action.COOPERATE): (R, R),
            (Action.COOPERATE, Action.DEFECT): (S, T),
            (Action.DEFECT, Action.COOPERATE): (T, S),
            (Action.DEFECT, Action.DEFECT): (P, P),
        }
```

## Horizon Types

| Type | Behavior | Purpose |
|---|---|---|
| **Fixed(N)** | Exactly N rounds | Baseline — known endpoint creates "endgame effect" |
| **Geometric(p)** | Each round has probability p of being the last | Unknown endpoint — no endgame effect |
| **Unknown** | Agent doesn't know the max (but there is one) | Tests information asymmetry |

The **endgame effect** is a key behavioral phenomenon: agents may defect in the last round (or last few rounds) because there's no future punishment. Unknown horizons remove this incentive.

## Seeded RNG

All randomness is controlled via `SeededRNG`:

```python
class SeededRNG:
    def __init__(self, seed: int):
        self._rng = random.Random(seed)
    
    def should_continue(self, p: float) -> bool:
        return self._rng.random() < p
```

Each run gets a unique seed. This ensures:

- Full reproducibility given a seed
- Different runs explore different random paths
- Statistical analysis can account for randomness

## Experiment Runner

The runner iterates through experiment conditions from YAML config:

```python
for condition in experiment.conditions:
    for replicate in range(num_replicates):
        agent_a = registry.create(condition.agent_a)
        agent_b = registry.create(condition.agent_b)
        protocol = create_protocol(condition.protocol_mode)
        horizon = create_horizon(condition.horizon)
        
        run_game(agent_a, agent_b, protocol, horizon, seed=base_seed + replicate)
```

## Data Output

Each run produces three files:

### rounds.jsonl

```json
{"round": 1, "action_a": "C", "action_b": "C", "payoff_a": 3, "payoff_b": 3, "chat_a": "Let's cooperate!", "chat_b": "Agreed!"}
{"round": 2, "action_a": "C", "action_b": "D", "payoff_a": 0, "payoff_b": 5, "chat_a": "Same again?", "chat_b": "Sure..."}
```

### aggregates.parquet

Per-run summary metrics in columnar format for fast analysis.

### run_manifest.json

```json
{
    "experiment": "phase2_baseline",
    "condition": "Naive_vs_Deceptive_chat_unstructured",
    "replicate": 3,
    "seed": 42003,
    "agent_a": {"type": "personality", "name": "naive_cooperator"},
    "agent_b": {"type": "personality", "name": "deceptive"},
    "protocol_mode": "unstructured",
    "horizon": {"type": "fixed", "max_rounds": 100},
    "started_at": "2026-02-25T15:30:00Z",
    "completed_at": "2026-02-25T15:34:23Z"
}
```
