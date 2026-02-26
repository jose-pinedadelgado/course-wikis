# 🛡️ AI Prisoner's Dilemma

> **Do protocol safeguards and game-theoretic awareness protect AI agents from adversarial exploitation?**

This research project studies how LLM-based agents cooperate and compete in iterated Prisoner's Dilemma games, with a focus on how architecture features (memory, identity, tools, MCP protocols) change the dynamics.

## Research Question

In iterated Prisoner's Dilemma interactions between LLM-based agents:

1. Do **protocol-level safeguards** (MCP-like structured communication) reduce vulnerability to adversarial exploitation?
2. Does equipping agents with **game-theoretic awareness** further improve cooperative outcomes beyond what protocols alone provide?
3. How do **agent architecture features** (memory, identity persistence, tools) affect cooperation stability?

## What's Novel?

Previous work (e.g., "LLMs Are Nicer Than Humans" studies) tested basic PD with vanilla LLM prompts. Nobody has tested:

- **Agent architecture as a variable** — memory type, identity persistence, context pressure
- **Protocol safeguards** — MCP-style structured communication vs. ad-hoc
- **Pre-decision chat** — agents can negotiate before choosing actions
- **Tool access** — whether tools create new exploitation vectors
- **Personality agents** — 6 distinct behavioral archetypes that simulate LLM personas

## Project Structure

```
5_ai_open_claw/
├── src/pd_phase2/
│   ├── agents/         # 12 agent strategies (6 classic + 6 personality)
│   ├── core/           # Game engine, metrics, protocol validator
│   ├── runners/        # Experiment orchestration
│   ├── storage/        # Data persistence
│   └── ui/             # Streamlit dashboard
├── configs/            # YAML experiment definitions
├── data/runs/          # Experimental results
├── docs/               # Research documentation
└── tests/              # 52 unit tests
```

## Quick Start

```bash
cd 5_ai_open_claw
uv sync
uv run pytest -q                                    # Run tests
uv run pd2 validate configs/experiment_phase2.yaml  # Validate config
uv run pd2 run configs/experiment_phase2.yaml --replicates 2  # Run experiment
```

## Four Experimental Phases

| Phase | Focus | Conditions | Est. Runs |
|---|---|---|---|
| [Phase 1](research/phases.md#phase-1-baseline-social-pd) | Baseline PD (no tools, no memory) | 45 | 675 |
| [Phase 2](research/phases.md#phase-2-capability-asymmetry) | Memory, identity, context pressure | 54 | 810 |
| [Phase 3](research/phases.md#phase-3-tools--ill-intent) | Tool access + adversarial goals | 24 | 360 |
| [Phase 4](research/phases.md#phase-4-mcp-vs-non-mcp) | MCP vs unstructured protocols | 24 | 360 |
| **Total** | | **147** | **~2,200** |

**Estimated cost:** $2,000–3,000 (Phase 1 on Haiku, Phases 2–4 on Sonnet)
**Estimated wall-clock:** 2–3 days with 10-concurrent parallelism

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.12 |
| Package Manager | UV |
| Data Models | Pydantic |
| Data Analysis | Pandas + PyArrow |
| Visualization | Streamlit |
| Testing | pytest (52 tests) |
| LLM Providers | Pluggable (Mock, OpenAI, Anthropic) |

---

*Jose Pineda — Assistant Professor, California State University, Long Beach*
