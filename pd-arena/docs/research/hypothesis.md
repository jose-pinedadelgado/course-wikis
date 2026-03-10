# Hypothesis & Motivation

## Core Hypothesis

> In iterated Prisoner's Dilemma interactions between LLM-based agents, do protocol-level safeguards reduce vulnerability to adversarial exploitation, and does equipping agents with game-theoretic awareness further improve cooperative outcomes beyond what protocols alone provide?

## Why This Matters?

LLM-based agents are increasingly deployed in multi-agent systems where they interact with other agents — negotiating, trading, sharing resources. These interactions often have the structure of social dilemmas: each agent can benefit individually by defecting, but mutual cooperation produces better collective outcomes.

### The Governance Gap

Current AI safety research focuses on single-agent alignment (making one model behave well). But in multi-agent settings, a well-aligned agent can still be **exploited** by an adversarial one. The question isn't just "is this agent safe?" but **"is this agent safe when surrounded by agents that may not be?"**

### Why Prisoner's Dilemma?

The IPD is the canonical model for studying cooperation under conflict of interest:

- **Simple enough** to control experimentally (two players, two actions, known payoffs)
- **Rich enough** to exhibit complex dynamics (trust, betrayal, forgiveness, retaliation)
- **Well-studied** in classical game theory (provides theoretical baselines)
- **Directly analogous** to real multi-agent interactions (resource sharing, negotiation, API cost allocation)

## What's Novel About Our Approach?

| Gap in Literature | Our Contribution |
|---|---|
| Nobody tested **agent architecture** (memory, tools, MCP) as PD variables | We systematically vary identity, memory regime, and context pressure |
| Nobody tested **pre-decision communication** as a controlled variable in LLM IPD | We add an optional chat phase and measure its effect |
| Nobody tested **protocol safeguards** (MCP-like) in PD settings | We compare unstructured vs. structured communication channels |
| Nobody mapped **LLM personas** to classical strategies using canonical baselines | We test 5 persona types against all 6 canonical PD strategies |
| Nobody published this in **IS venues** | We target ICIS/AMCIS/JAIS with a governance + design science framing |

## The 2×2 Design

Our core experimental design varies two factors:

|  | No Chat | Chat |
|---|---------|------|
| **Unstructured** | Pure PD baseline | Communication without safeguards |
| **Structured (MCP)** | Protocol-only | Communication with protocol safeguards |

This produces four conditions, each revealing a different aspect:

1. **No Chat + Unstructured**: Baseline. How do agents behave with no communication and no protocol?
2. **Chat + Unstructured**: Does cheap talk help cooperation? Or enable deception?
3. **No Chat + Structured**: Do protocols alone (without communication) affect behavior?
4. **Chat + Structured**: The full governance stack. Does MCP + communication produce the best outcomes?

## Target Venues

- **ICIS** (International Conference on Information Systems) — governance & design science track
- **AMCIS** (Americas Conference on Information Systems) — AI & intelligent systems track
- **JAIS** (Journal of the Association for Information Systems) — full research paper

## Related Work

See the [Literature Review](literature.md) for detailed coverage of prior studies.
