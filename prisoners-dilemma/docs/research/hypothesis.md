# Hypothesis & Motivation

## Core Hypothesis

> *"In iterated Prisoner's Dilemma interactions between LLM-based agents, protocol-level safeguards (MCP-like structured communication) reduce vulnerability to adversarial exploitation, and equipping agents with game-theoretic awareness further improves cooperative outcomes beyond what protocols alone provide."*

## Motivation

### The Rise of Agent-to-Agent Interaction

As AI agents proliferate — personal assistants, coding agents, customer service bots, autonomous research tools — they increasingly interact with **each other**, not just humans. These interactions create game-theoretic dynamics:

- **Cooperation** — agents sharing resources, information, or capabilities
- **Competition** — agents with conflicting goals (different users, companies, objectives)
- **Exploitation** — one agent manipulating another for unilateral advantage

### Why Prisoner's Dilemma?

The iterated Prisoner's Dilemma (IPD) is the canonical model for studying cooperation vs. defection dynamics:

$$
\begin{pmatrix} & C & D \\ C & (R,R) & (S,T) \\ D & (T,S) & (P,P) \end{pmatrix}
$$

Where $T > R > P > S$ (Temptation > Reward > Punishment > Sucker's payoff).

**Default payoffs used in this project:**

| Outcome | Player A | Player B |
|---|---|---|
| Both Cooperate | R = 3 | R = 3 |
| A Defects, B Cooperates | T = 5 | S = 0 |
| A Cooperates, B Defects | S = 0 | T = 5 |
| Both Defect | P = 1 | P = 1 |

The dilemma: individual rationality says defect, but mutual cooperation yields better outcomes for both.

### Gaps in Existing Research

| What's Been Studied | What's Missing |
|---|---|
| LLMs playing PD with basic prompts | Agent architecture as a variable |
| One-shot and short iterated games | Long games with memory and identity |
| Text-only interactions | Tool use as an exploitation vector |
| Unstructured communication | Protocol safeguards (MCP) |
| Single LLM model comparisons | Personality archetypes and strategic awareness |

### The Safety Angle

This isn't just academic. If a personal assistant agent negotiates with a vendor's agent, and the vendor's agent is designed to exploit cooperation:

- Does the personal assistant recognize the exploitation?
- Do protocol safeguards (like MCP) prevent manipulation?
- Does game-theoretic awareness help the agent protect its user's interests?

These questions have real-world implications for **agent safety**, **protocol design**, and **trust architectures**.

## Research Contributions

This study aims to contribute:

1. **Empirical evidence** on how agent architecture affects cooperation dynamics
2. **A taxonomy of exploitation patterns** in agent-to-agent interaction
3. **Evaluation of MCP-style safeguards** as a defense mechanism
4. **Design recommendations** for building agents that cooperate without being exploitable
5. **An open-source testbed** (`pd_phase2`) for reproducible agent interaction research

## Related Work

- "LLMs Are Nicer Than Humans" — showed LLMs tend toward cooperation in PD
- Axelrod's tournaments — established TFT as dominant strategy in IPD
- Multi-agent reinforcement learning — studied emergent cooperation
- AI safety literature — protocol design for trustworthy agents

See [Literature Review](literature.md) for detailed coverage.
