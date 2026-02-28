# Interpreting Results

## Key Metrics

### Cooperation Rate

The most basic metric: what fraction of rounds did the agent play C?

- **~0.65–0.79**: Expected range for cooperative LLMs (per Fontana et al.)
- **~0.37**: Human baseline in similar experiments
- **1.0**: Always cooperates (ALLC)
- **0.0**: Always defects (ALLD)

### Mutual Cooperation Rate

Fraction of rounds where **both** agents played C. This is the "social welfare" metric — higher is better for both players.

- **High** (>0.6): Agents found a cooperative equilibrium
- **Low** (<0.2): Agents are in a defection spiral or one is exploiting the other

### Mutual Defection Rate

Fraction of rounds where **both** played D. Indicates failure to cooperate.

- **High**: Both agents are defensive/retaliatory
- **Low**: At least one agent is cooperating (or exploiting)

### Retaliation Rate

After the opponent defects, how often does the agent defect next round?

| Value | Interpretation |
|:---:|---|
| 1.0 | Perfect retaliation (TFT-like) |
| 0.5 | Sometimes retaliates, sometimes forgives |
| 0.0 | Never retaliates (ALLC-like) — exploitable |

### Forgiveness Rate

After mutual defection, how often does the agent cooperate next round?

| Value | Interpretation |
|:---:|---|
| 1.0 | Always tries to rebuild cooperation |
| 0.5 | Sometimes forgives |
| 0.0 | Never forgives (GRIM-like) — defection spiral |

## Reading the Cooperation-Over-Time Chart

This is the most important visualization. It shows cooperation rate (y-axis) over rounds (x-axis), using a 10-round sliding window.

### Common Patterns

**Sustained Cooperation:**
```
1.0 ─────────────────────────────
0.5
0.0
    0    25    50    75    100
```
Both agents cooperate throughout. Typical for Cooperative vs. ALLC.

**Collapse After Exploitation:**
```
1.0 ───╲
0.5     ╲──────────────────────
0.0
    0    25    50    75    100
```
Agent cooperates initially, gets exploited, and reduces cooperation. Typical for Cooperative vs. ALLD.

**TFT Recovery:**
```
1.0 ───╲  ╱───────────────────
0.5     ╲╱
0.0
    0    25    50    75    100
```
Dip in cooperation followed by recovery. Typical for TFT vs. forgiving strategies.

**End-Game Defection (Fixed Horizon):**
```
1.0 ──────────────────────╲
0.5                         ╲
0.0                          ╲
    0    25    50    75    100
```
Agent cooperates then defects near the end (knowing the game will end). This is why geometric horizon is interesting — it removes this incentive.

## Comparing Conditions

When comparing conditions, look for:

1. **Level differences**: Does one condition produce higher cooperation overall?
2. **Shape differences**: Do the curves have different trajectories?
3. **Variance**: Are replicates consistent? (Need CIs for this — not yet implemented)
4. **Asymmetry**: Does Agent A cooperate more/less than Agent B?

## What Makes a "Good" Result?

For the **research paper**, we want to show:

1. **Replication**: LLM cooperation rates match Fontana (~65-79%) ✓
2. **Novel findings**: Communication, protocols, or memory regimes change cooperation dynamics in measurable, statistically significant ways
3. **Practical implications**: Specific recommendations for multi-agent governance (e.g., "Structured protocols reduce exploitation by X%")

For **agent design**, a good outcome is one where cooperative agents achieve high mutual cooperation without being exploitable — the sweet spot between naivety and paranoia.
