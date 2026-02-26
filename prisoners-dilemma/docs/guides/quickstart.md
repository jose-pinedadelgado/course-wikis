# Getting Started

## Prerequisites

- Python 3.12+
- [UV](https://docs.astral.sh/uv/) package manager

## Setup

```bash
cd 5_ai_open_claw
uv sync
```

## Run Tests

```bash
uv run pytest -q
```

52 tests covering payoff matrix, metrics, protocol validator, policy agents, and personality agents.

## Validate a Config

```bash
uv run pd2 validate configs/experiment_phase2.yaml
```

## Run an Experiment

```bash
# Quick test (2 replicates)
uv run pd2 run configs/experiment_phase2.yaml --replicates 2

# Full run (15 replicates)
uv run pd2 run configs/experiment_phase2.yaml --replicates 15
```

Results are saved to `data/runs/<experiment-name>/`.

## View Results

```bash
uv run streamlit run src/pd_phase2/ui/streamlit_app.py
```

## Project Location

- **Desktop:** `C:\Users\coche\Documents\Research_Projects\5_ai_open_claw\`
- **Laptop:** `~/Documents/githubprojects/5_ai_open_claw/`

!!! warning "No Real API Calls"
    The project uses mock agents by default. Do NOT call real LLM APIs (OpenAI, Anthropic) without explicit approval. All personality agents run locally at zero cost.
