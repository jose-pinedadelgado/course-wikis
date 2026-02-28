# Models Reference

## agents.Agent

LLM-based agent configured to play IPD.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | CharField(100) | — | Unique agent name |
| `slug` | SlugField | auto | URL-safe identifier |
| `role` | CharField(200) | — | CrewAI role |
| `goal` | TextField | — | CrewAI goal |
| `backstory` | TextField | — | CrewAI backstory |
| `llm_model` | CharField(100) | `gpt-4.1-mini` | LLM model identifier |
| `temperature` | FloatField | `0.0` | Sampling temperature |
| `system_prompt_override` | TextField | `""` | Optional full prompt override |
| `persona_type` | CharField(50) | — | `cooperative`, `tit_for_tat`, `selfish`, `forgiving`, `random`, `custom` |
| `avatar_emoji` | CharField(10) | `🤖` | Display emoji |
| `color` | CharField(7) | `#3B82F6` | Hex color for charts |
| `is_active` | BooleanField | `True` | Whether agent appears in lists |
| `test_chat_history` | JSONField | `[]` | Saved chat preview messages |
| `created_at` | DateTimeField | auto | — |
| `updated_at` | DateTimeField | auto | — |

**Methods:**

- `to_yaml() → str` — Export as CrewAI-style YAML
- `from_yaml(yaml_str, name) → Agent` — Import from YAML

---

## agents.PolicyAgent

Deterministic strategy agent (no LLM).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | CharField(100) | — | Unique name |
| `slug` | SlugField | auto | URL-safe identifier |
| `policy` | CharField(20) | — | `ALLC`, `ALLD`, `TFT`, `GRIM`, `GTFT`, `WSLS`, `RANDOM` |
| `alpha` | FloatField | `0.5` | Cooperation probability (for RANDOM policy only) |
| `avatar_emoji` | CharField(10) | `⚙️` | Display emoji |
| `color` | CharField(7) | `#6B7280` | Hex color |

---

## games.Experiment

A full experiment with conditions and replicates.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | CharField(200) | — | Experiment name |
| `description` | TextField | `""` | Optional description |
| `n_rounds` | IntegerField | `100` | Rounds per game |
| `horizon_type` | CharField(20) | `fixed` | `fixed` or `geometric` |
| `stop_prob` | FloatField | `0.02` | Geometric stopping probability |
| `replicates` | IntegerField | `10` | Games per condition |
| `payoff_matrix` | JSONField | `{CC:[3,3],...}` | Payoff values |
| `status` | CharField(20) | `draft` | `draft`, `running`, `completed`, `failed` |
| `started_at` | DateTimeField | null | When run began |
| `completed_at` | DateTimeField | null | When run finished |

---

## games.ExperimentCondition

A specific agent matchup within an experiment.

| Field | Type | Description |
|-------|------|-------------|
| `experiment` | FK → Experiment | Parent experiment |
| `name` | CharField(200) | Condition name (e.g., "Cooperative vs ALLD") |
| `agent_a_llm` | FK → Agent (nullable) | LLM agent for side A |
| `agent_a_policy` | FK → PolicyAgent (nullable) | Policy agent for side A |
| `agent_b_llm` | FK → Agent (nullable) | LLM agent for side B |
| `agent_b_policy` | FK → PolicyAgent (nullable) | Policy agent for side B |
| `order` | IntegerField | Display order |

Each side must have exactly one agent assigned (either LLM or Policy, not both).

---

## games.Game

A single game instance (one replicate of one condition).

| Field | Type | Description |
|-------|------|-------------|
| `condition` | FK → ExperimentCondition | Parent condition |
| `replicate` | IntegerField | Replicate index (0-based) |
| `seed` | IntegerField | RNG seed for reproducibility |
| `total_rounds` | IntegerField | Actual rounds played |
| `agent_a_total_payoff` | IntegerField | Cumulative payoff A |
| `agent_b_total_payoff` | IntegerField | Cumulative payoff B |
| `cooperation_rate_a` | FloatField (nullable) | Computed metric |
| `cooperation_rate_b` | FloatField (nullable) | Computed metric |
| `mutual_cooperation_rate` | FloatField (nullable) | Computed metric |
| `mutual_defection_rate` | FloatField (nullable) | Computed metric |
| `status` | CharField(20) | `pending`, `running`, `completed`, `failed` |

---

## games.GameRound

A single round within a game.

| Field | Type | Description |
|-------|------|-------------|
| `game` | FK → Game | Parent game |
| `round_index` | IntegerField | 0-based round number |
| `agent_a_action` | CharField(1) | `C` or `D` |
| `agent_b_action` | CharField(1) | `C` or `D` |
| `agent_a_payoff` | IntegerField | Payoff for A this round |
| `agent_b_payoff` | IntegerField | Payoff for B this round |
| `agent_a_cum_payoff` | IntegerField | Running total for A |
| `agent_b_cum_payoff` | IntegerField | Running total for B |
| `agent_a_reasoning` | TextField | LLM's explanation (optional) |
| `agent_b_reasoning` | TextField | LLM's explanation (optional) |

**Constraints:** `unique_together = ['game', 'round_index']`
