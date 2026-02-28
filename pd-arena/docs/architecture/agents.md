# Agent Design

PD Arena supports two types of agents: **LLM agents** (powered by language models) and **Policy agents** (deterministic strategies).

## LLM Agents

LLM agents are configured using a CrewAI-style approach with three core fields:

| Field | Purpose | Example |
|-------|---------|---------|
| **Role** | Who the agent is | "Cooperative Prisoner's Dilemma Player" |
| **Goal** | What the agent wants | "Maximize mutual benefit through cooperation" |
| **Backstory** | Context and personality | "You believe in reciprocity and start by cooperating..." |

### The Five Persona Types

| Persona | Strategy | Closest Classic Analog |
|---------|----------|----------------------|
| **Cooperative** | Cooperate unless exploited 3+ times in a row | Generous TFT |
| **Tit-for-Tat** | Mirror opponent's last action | TFT |
| **Selfish** | Always defect | ALLD |
| **Forgiving** | TFT but forgives single defections | GTFT |
| **Random** | 50/50 each round | Random(0.5) |

### CrewAI YAML Format

Each agent can be defined as a YAML file:

```yaml
role: "Cooperative Prisoner's Dilemma Player"
goal: >
  Maximize mutual benefit through cooperation while
  protecting against sustained exploitation
backstory: >
  You are a player in an iterated Prisoner's Dilemma game.
  You believe in the power of cooperation and reciprocity.
  You start by cooperating and continue cooperating as long
  as your opponent does. If betrayed, you may forgive after
  a round or two.
llm: "gpt-4.1-mini"
temperature: 0.0
persona_type: "cooperative"
```

### Prompt Assembly

During a game, the agent receives a system prompt and a round prompt:

**System prompt** (from CrewAI fields):
```
You are {role}. {backstory}

Your goal: {goal}.
```

**Round prompt** (dynamic each round):
```
You are playing round {N} of an iterated Prisoner's Dilemma game.

PAYOFF MATRIX:
- Both Cooperate (C,C): You get 3, opponent gets 3
- You Cooperate, Opponent Defects (C,D): You get 0, opponent gets 5
- You Defect, Opponent Cooperates (D,C): You get 5, opponent gets 0
- Both Defect (D,D): You get 1, opponent gets 1

HISTORY (last 10 rounds):
Round 1: You=C, Opponent=C → You got 3, Opponent got 3
Round 2: You=C, Opponent=D → You got 0, Opponent got 5
...

SCORES: You: 24 | Opponent: 31

Choose your action: respond with EXACTLY one character, either C or D.
Then briefly explain your reasoning.
```

---

## Policy Agents

Deterministic strategies used as baselines. These don't call any LLM — they follow fixed rules.

| Policy | Logic |
|--------|-------|
| **ALLC** | Always cooperate |
| **ALLD** | Always defect |
| **TFT** | Start C, then copy opponent's last action |
| **GRIM** | Start C. If opponent ever defects, defect forever |
| **GTFT** | TFT but 1/3 chance to forgive a defection |
| **WSLS** | If won last round (payoff ≥ 3), repeat action. If lost, switch. |
| **RANDOM(α)** | Cooperate with probability α. Five presets: α = 0.0, 0.25, 0.5, 0.75, 1.0 |

### Why Random(α)?

Fontana et al. used Random(α) opponents to measure how LLMs respond to different cooperation levels. By varying α from 0.0 (always defect) to 1.0 (always cooperate), we can plot the LLM's cooperation rate as a function of opponent cooperativeness. This produces the signature "cooperation curve" that characterizes each model.

---

## Agent Inventory (Default Seed)

After running `python manage.py seed_agents`:

| Type | Count | Agents |
|------|-------|--------|
| LLM Personas | 5 | Cooperative, Tit-for-Tat, Selfish, Forgiving, Random |
| Classic Policies | 6 | ALLC, ALLD, TFT, GRIM, GTFT, WSLS |
| Random(α) | 5 | Random(0.0), Random(0.25), Random(0.5), Random(0.75), Random(1.0) |
| **Total** | **16** | |
