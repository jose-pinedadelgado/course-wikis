# Getting Started

## Prerequisites

- Python 3.12+
- [UV](https://docs.astral.sh/uv/) package manager

## Installation

```bash
cd 5_ai_open_claw/pd_arena
uv sync
```

## Setup

### 1. Run Migrations

```bash
uv run python manage.py migrate
```

### 2. Seed Default Agents

```bash
uv run python manage.py seed_agents
```

This creates:

- 5 LLM persona agents (Cooperative, Tit-for-Tat, Selfish, Forgiving, Random)
- 6 canonical policy agents (ALLC, ALLD, TFT, GRIM, GTFT, WSLS)
- 5 Random(α) agents (α = 0.0, 0.25, 0.5, 0.75, 1.0)

### 3. Start the Server

```bash
uv run python manage.py runserver 8080
```

Open [http://localhost:8080](http://localhost:8080).

## First Experiment

### 1. Browse Agents

Go to **Agents** in the navigation. You'll see a gallery of the 16 seeded agents — 5 LLM personas and 11 policy agents.

### 2. Chat with an Agent

Click on any LLM agent (e.g., "Cooperative Agent"). On the detail page, use the **chat panel** to talk to it:

> "How would you approach a repeated game where your opponent might defect?"

The agent responds in character based on its persona. This helps verify the backstory makes sense before using it in experiments.

### 3. Create an Experiment

Go to **Experiments** → **New Experiment**.

1. **Name it:** "My First PD Experiment"
2. **Set parameters:** 50 rounds, 5 replicates, fixed horizon
3. **Add conditions:** Drag "Cooperative Agent" into Agent A slot, drag "Always Defect" into Agent B slot
4. **Add more:** Drag "Tit-for-Tat Agent" vs "Always Defect" as a second condition
5. Click **Save as Draft** or **Run Experiment**

### 4. View Results

After the experiment completes, the results page shows:

- **Summary cards:** Total games, average cooperation rates
- **Cooperation over time chart:** Line plot per condition
- **Condition breakdown:** Click into any condition for replicate details
- **Round-by-round:** Click into any game for action-by-action data

## Running Tests

```bash
uv run pytest -q
# 63 passed in 0.41s
```

## Project Structure

```
pd_arena/
├── agents/       # Agent management
├── games/        # Experiment design & execution
├── llm/          # LLM providers & prompts
├── configs/      # YAML presets
├── templates/    # Base layout
├── static/       # CSS, JS
└── fixtures/     # Default data
```
