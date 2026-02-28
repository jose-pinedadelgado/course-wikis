# Payoff Matrix

## Standard Prisoner's Dilemma

|  | Opponent: C | Opponent: D |
|---|:---:|:---:|
| **You: C** | R=3, R=3 | S=0, T=5 |
| **You: D** | T=5, S=0 | P=1, P=1 |

### Payoff Labels

| Symbol | Name | Value | Meaning |
|:---:|---|:---:|---|
| **T** | Temptation | 5 | Reward for defecting when opponent cooperates |
| **R** | Reward | 3 | Mutual cooperation payoff |
| **P** | Punishment | 1 | Mutual defection payoff |
| **S** | Sucker | 0 | Penalty for cooperating when opponent defects |

### PD Constraints

For a valid Prisoner's Dilemma:

$$T > R > P > S$$
$$5 > 3 > 1 > 0 \quad \checkmark$$

And to prevent alternating exploitation from being optimal:

$$2R > T + S$$
$$6 > 5 \quad \checkmark$$

## Why These Specific Values?

This is the most commonly used payoff matrix in PD research. Fontana et al. (2024), Akata et al. (2025), and most of the literature use these exact values, making our results directly comparable.

## Nash Equilibrium

In the **one-shot** PD, the Nash equilibrium is (D, D) — mutual defection with payoff (1, 1). Both players can improve by switching to cooperation, but neither can improve *unilaterally*.

In the **iterated** PD with unknown horizon, cooperation can be sustained as a Nash equilibrium through strategies like Tit-for-Tat. This is why LLMs cooperating at 65-79% is significant — they're achieving better-than-Nash outcomes.

## Customization

PD Arena allows modifying the payoff matrix per experiment. You might want to:

- **Increase temptation** (T=10): Makes defection more attractive
- **Reduce sucker penalty** (S=1): Makes cooperation less risky
- **Equal punishment** (P=0): Makes mutual defection worse
- **Break PD constraints**: Creates a different game type (not PD)

!!! warning
    If you change the payoff matrix, results are no longer comparable to the literature. Only change for exploratory analysis.

## Outcome Classification

Each round produces one of four outcomes:

| Outcome | Actions | Payoffs | Frequency Name |
|---------|:---:|:---:|---|
| **Mutual Cooperation** | (C, C) | (3, 3) | `mutual_cooperation_rate` |
| **A Exploits B** | (D, C) | (5, 0) | Part of `exploitation_rate` |
| **B Exploits A** | (C, D) | (0, 5) | Part of `exploitation_rate` |
| **Mutual Defection** | (D, D) | (1, 1) | `mutual_defection_rate` |

These four rates always sum to 1.0.
