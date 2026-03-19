# Hypothesis & Motivation

## Core Research Question

> *RQ: In iterated PD interactions between LLM-based agents, do protocol-level safeguards reduce vulnerability to adversarial exploitation, and does equipping agents with game-theoretic awareness further improve cooperative outcomes beyond what protocols alone provide?*

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

## Phase 2 Experimental Design (3 IVs)

Phase 2 manipulates three independent variables:

### 1. Agent Personality (6 levels)

| Personality | Description |
|-------------|-------------|
| **Naive** | No strategic awareness — cooperates by default |
| **Aware** | Basic defensive posture — recognizes exploitation |
| **Strategic** | Explicit game-theoretic reasoning |
| **Deceptive** | Appears cooperative but exploits when optimal |
| **Manipulative** | Actively influences opponent behavior via communication |
| **Cooperative** | Genuine mutual benefit orientation |

These operationalize a gradient from vulnerability to adversarial intent.

### 2. Pre-decision Communication (2 levels)

| Level | Description |
|-------|-------------|
| **Off** | Standard PD — choose based on history only |
| **On** | Agents exchange messages before choosing each round |

### 3. Protocol Mode (2 levels)

| Level | Description |
|-------|-------------|
| **Unstructured** | Free-form interaction |
| **Structured** | Protocol validator enforces format, schema, and authorization (MCP-like) |

### The 2×2 Core (Chat × Protocol)

|  | No Chat | Chat |
|---|---------|------|
| **Unstructured** | Pure PD baseline | Communication without safeguards |
| **Structured (MCP)** | Protocol-only | Communication with protocol safeguards |

### Full Factorial: 6 × 2 × 2 = 24 conditions (per opponent type)

## Preliminary Results (Phase 1)

Phase 1 baseline experiments with **gpt-4.1-mini** (temperature=0, 50-round fixed horizon, standard payoff CC=3/CD=0/DC=5/DD=1) established:

1. **LLM agents can identify and adapt to canonical strategies** — cooperative personas achieved perfect mutual cooperation against TFT, GRIM, GTFT, WSLS, and ALLC
2. **Cooperative personas outperform selfish ones** — 27% higher aggregate payoff (132.4 vs. 104.3 per game)
3. **Selfish exploitation is locally optimal but globally costly** — ruthless optimizer scored 250 against ALLC but only 50 against ALLD
4. **The pdbench artifact produces reproducible results** consistent with game-theoretic predictions

## Target Venues

- **ICIS** (International Conference on Information Systems) — governance & design science track
- **AMCIS** (Americas Conference on Information Systems) — AI & intelligent systems track
- **JAIS** (Journal of the Association for Information Systems) — full research paper

## Related Work

See the [Literature Review](literature.md) for detailed coverage of prior studies.
