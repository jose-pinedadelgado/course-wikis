# Getting Started

## Prerequisites

- Python ≥3.10, <3.13
- [UV](https://docs.astral.sh/uv/) package manager
- OpenAI API key

## Setup

```bash
cd 3_Pineapple_Course_Assistant
uv sync
```

## Configure API Key

Create a `.env` file:

```bash
OPENAI_API_KEY=sk-your-key-here
OPENAI_MODEL_NAME=gpt-4o
```

## Run

```bash
uv run pineapple run
```

The crew will:

1. Research the configured topic
2. Generate a comprehensive report
3. Save output to `report.md`

## Other Commands

| Command | Description |
|---|---|
| `uv run pineapple run` | Execute the crew (default) |
| `uv run pineapple train` | Train with performance tracking |
| `uv run pineapple replay <task-id>` | Replay a previous task |
| `uv run pineapple test` | Test with evaluation metrics |

## Customization

- **Agents:** Edit `src/pineapple/config/agents.yaml`
- **Tasks:** Edit `src/pineapple/config/tasks.yaml`
- **Logic:** Edit `src/pineapple/crew.py`
- **Knowledge:** Add files to `knowledge/`

## Project Locations

- **Desktop:** `C:\Users\coche\Documents\Research_Projects\3_Pineapple_Course_Assistant\`
- **Laptop:** `~/Documents/githubprojects/3_Pineapple_Course_Assistant/`
