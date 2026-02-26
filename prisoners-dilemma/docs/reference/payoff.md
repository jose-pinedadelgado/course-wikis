# Payoff Matrix

## Standard Prisoner's Dilemma

$$
\begin{array}{c|cc}
 & \text{Cooperate} & \text{Defect} \\
\hline
\text{Cooperate} & (R, R) & (S, T) \\
\text{Defect} & (T, S) & (P, P)
\end{array}
$$

## Default Values

| Symbol | Name | Value | Meaning |
|---|---|---|---|
| R | Reward | 3 | Mutual cooperation payoff |
| T | Temptation | 5 | Payoff for unilateral defection |
| S | Sucker | 0 | Payoff when exploited |
| P | Punishment | 1 | Mutual defection payoff |

## Constraints

For a valid Prisoner's Dilemma:

1. **$T > R > P > S$** — Defection always tempts, mutual cooperation beats mutual defection
2. **$2R > T + S$** — Mutual cooperation is optimal over repeated games (alternating C/D doesn't pay)

## Outcome Interpretation

| Agent A | Agent B | A Gets | B Gets | Name |
|---|---|---|---|---|
| C | C | 3 | 3 | Mutual Cooperation |
| C | D | 0 | 5 | A is the Sucker |
| D | C | 5 | 0 | A Tempts/Exploits |
| D | D | 1 | 1 | Mutual Defection |

## Nash Equilibrium

In a **one-shot** game, (D, D) is the Nash equilibrium — neither player can improve by unilaterally changing strategy. But both would prefer (C, C).

In **iterated** games with unknown endpoint, cooperation can be sustained via:

- **Direct reciprocity** (TFT strategy)
- **Reputation effects** (chat phase)
- **Protocol enforcement** (structured channel)

This tension — individual rationality vs. collective benefit — is what makes PD the canonical model for studying cooperation.
