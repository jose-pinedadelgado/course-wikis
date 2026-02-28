# Comparison: PD Arena vs pd_phase2

PD Arena (Django) and pd_phase2 (CLI) are complementary tools for the same research project. Here's how they compare.

## Feature Comparison

| Feature | pd_phase2 (CLI) | PD Arena (Django) |
|---------|:---:|:---:|
| **Interface** | CLI (`pd2 run`) | Web UI (HTMX + Tailwind) |
| **Config format** | YAML files | Database + YAML import/export |
| **Agent creation** | Edit YAML | Web form + drag-and-drop |
| **Agent testing** | — | Chat preview |
| **Policy agents** (6 canonical) | ✅ | ✅ |
| **Random(α) opponents** | ❌ | ✅ |
| **Personality agents** (6 types) | ✅ (hardcoded) | ✅ (mock, 5 types) |
| **Real LLM prompts** | ❌ | ✅ (CrewAI + OpenAI) |
| **Chat phase in games** | ✅ | ✅ |
| **Protocol validator** (MCP-like) | ✅ | ✅ (3 levels) |
| **Deception/manipulation metrics** | ✅ | ✅ (5 metrics) |
| **Geometric horizon** | ✅ | ✅ |
| **Results visualization** | Streamlit viewer | Chart.js (built-in) |
| **Test count** | 52 | 63 |

## When to Use Which

### Use pd_phase2 when:

- Running batch experiments from the command line
- You need the chat phase or protocol validator (Phase 2 features)
- Automated CI/CD pipelines
- JSONL output for custom analysis scripts

### Use PD Arena when:

- Designing experiments visually (drag-and-drop)
- Creating and testing LLM agents interactively
- Presenting results to collaborators
- Running experiments with real LLM API calls (once wired)
- Sharing the platform with research assistants

## Migration Plan

Phase 2 features from pd_phase2 will be ported into PD Arena:

| Feature | pd_phase2 Source | PD Arena Target | Priority |
|---------|-----------------|-----------------|:---:|
| Chat phase | `runners/run_experiment.py` | `games/engine.py` | High |
| Protocol validator | `core/protocol.py` | New: `games/protocol.py` | High |
| Deception metrics | `core/metrics.py` | `games/metrics.py` | High |
| Personality agents (6 types) | `agents/personality.py` | LLM personas replace these | Medium |
| Experiment configs | `configs/experiment_phase2.yaml` | Database-driven | Already done |

## Architecture Comparison

### pd_phase2

```
YAML Config → CLI → Agent Factory → Game Loop → JSONL Log → Aggregates
```

Everything is file-based. Agents are instantiated from YAML, results written to JSONL, aggregated into Parquet.

### PD Arena

```
Web UI → Django Models → Engine → Database → Chart.js
```

Everything is database-backed. Agents are DB records, results stored as Django models, visualized with Chart.js.

## Shared Concepts

Both platforms share the same core game theory:

- Same payoff matrix (CC=3, CD=0, DC=5, DD=1)
- Same canonical strategies (ALLC, ALLD, TFT, GRIM, GTFT, WSLS)
- Same history window (10 rounds)
- Same metric definitions (cooperation rate, retaliation, forgiveness)
- Same experimental phases roadmap (Phase 1 → 4)
