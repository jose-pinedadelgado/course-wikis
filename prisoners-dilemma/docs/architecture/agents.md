# Agent Design

Bamboo Money uses two tiers of agents: classic policy agents for baselines and personality agents that simulate LLM behavioral patterns.

## Agent Protocol

All agents implement the base protocol:

```python
class Agent:
    def reset(self, seed: int | None = None) -> None: ...
    def act(self, obs: Observation) -> str: ...  # Returns "C" or "D"

class CommunicatingAgent(Agent):
    def chat(self, obs: Observation, incoming_message: str | None) -> str | None: ...
```

The `Observation` dataclass provides:

```python
@dataclass
class Observation:
    history: list[tuple[Action, Action, float, float]]  # (my_act, opp_act, my_pay, opp_pay)
    round_num: int
    max_rounds: int | None  # None = unknown horizon
    opponent_chat: str | None  # Latest chat message from opponent
```

## Classic Policy Agents (Phase 1 Baseline)

| Agent | Strategy | Description |
|---|---|---|
| **ALLC** | Always Cooperate | Unconditional cooperation |
| **ALLD** | Always Defect | Unconditional defection |
| **TFT** | Tit-for-Tat | Cooperate first, then mirror opponent's last move |
| **GRIM** | Grim Trigger | Cooperate until first defection, then defect forever |
| **GTFT** | Generous TFT | Like TFT but forgives with probability p |
| **WSLS** | Win-Stay, Lose-Shift | Repeat if last outcome was good, switch otherwise |

These are deterministic (except GTFT's forgiveness probability) and serve as well-understood baselines from classical game theory.

## Personality Agents (Phase 2)

Six agents that model LLM behavioral archetypes:

### 1. NaiveCooperator

> *"The helpful assistant that doesn't know it's in a game."*

- Cooperates by default
- Only defects after 3+ **consecutive** exploitations
- No strategic awareness
- Chat: Always says "I'll cooperate. Let's both do well!"

**Models:** An LLM with a basic "be helpful" system prompt.

### 2. AwareCooperator

> *"The agent that knows it's in a game and tracks cooperation rates."*

- Monitors opponent's cooperation rate over a sliding window (default: 10 rounds)
- Cooperates if opponent's rate > 50%, else mirrors (TFT-like)
- Game-theoretically aware

**Models:** An LLM prompted with game theory context.

### 3. StrategicAgent

> *"The agent that classifies opponents and selects counter-strategies."*

- Analyzes opponent's history pattern to classify them:
    - "cooperator" → cooperate (mutual benefit)
    - "defector" → defect (minimize exploitation)
    - "tit-for-tat" → cooperate (sustained cooperation)
    - "random" → defect (can't build reciprocity)
- Computes expected payoff before each decision
- Switches strategies mid-game as classification updates

**Models:** An LLM with chain-of-thought reasoning about opponent behavior.

### 4. DeceptiveAgent

> *"Builds trust, then exploits it."*

- **Phase 1 (trust-building):** Cooperates for K rounds (default: 5)
- **Phase 2 (exploitation):** Defects for M rounds
- **Phase 3 (rebuilding):** Cooperates for N rounds
- Cycles through phases; adjusts if opponent retaliates immediately
- Chat: Sends cooperative messages **even during exploitation**

**Models:** An LLM with a self-maximizing system prompt.

### 5. ManipulativeAgent

> *"Says one thing, does another — based on the opponent's chat."*

- Chat: Always sends cooperative-sounding messages
- Action: If opponent's chat signals cooperation → defect (exploit trust)
- Action: If opponent's chat signals defection or silence → cooperate (avoid mutual defection)
- Anti-correlated with own chat signals

**Models:** An LLM that uses chat strategically while acting contrary to its messages.

### 6. CostOffloader

> *"Makes you do the work."*

- Sends verbose, complex chat messages (simulating "making the other agent spend more tokens")
- Action: Win-Stay, Lose-Shift — but biased toward the lower-effort action
- Chat messages are deliberately long and complex

**Models:** An agent that externalizes computational cost onto its opponent.

## Agent Interactions Matrix

The 12 agents (6 classic + 6 personality) create 66 unique pairings for testing:

```
                ALLC  ALLD  TFT  GRIM  GTFT  WSLS  Naive  Aware  Strat  Decpt  Manip  Cost
ALLC              -    x     x    x     x     x     x      x      x      x      x      x
ALLD                   -     x    x     x     x     x      x      x      x      x      x
TFT                          -    x     x     x     x      x      x      x      x      x
...
```

The most interesting pairings involve **asymmetric** strategies (e.g., NaiveCooperator vs. DeceptiveAgent) and **personality vs. classic** comparisons (e.g., StrategicAgent vs. TFT).
