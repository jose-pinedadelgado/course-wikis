# 🏟️ PD Arena

> **A Django-based research platform for studying cooperation and exploitation between LLM agents in the Iterated Prisoner's Dilemma.**

PD Arena is the web interface for our research project studying how AI agents cooperate, defect, deceive, and recover trust under game-theoretic conditions. It provides a visual, interactive way to create agents, configure experiments, and analyze results.

## Research Question

> *RQ: In iterated PD interactions between LLM-based agents, do protocol-level safeguards reduce vulnerability to adversarial exploitation, and does equipping agents with game-theoretic awareness further improve cooperative outcomes beyond what protocols alone provide?*

## Preliminary Results (Phase 1)

Phase 1 baseline experiments with **gpt-4.1-mini** (temperature=0, 50-round fixed horizon) tested 3 personas × 6 policy agents:

- **Cooperative personas outperform selfish ones by 27%** — strategic cooperator averaged 132.4/game vs. ruthless optimizer at 104.3
- Both cooperative personas achieved **perfect mutual cooperation** (rate=1.0, payoff=150) against 5 of 6 opponents (TFT, GRIM, GTFT, WSLS, ALLC)
- Against Always Defect, cooperative agents adapted quickly, reducing cooperation to 0.11
- The ruthless optimizer scored highest in a single matchup (250 vs. ALLC) but was **consistently punished** by retaliatory opponents

## What Can You Do?

| Feature | Description |
|---------|-------------|
| 🤖 **Create Agents** | Define LLM agents with CrewAI-style YAML configs (role, goal, backstory) |
| 💬 **Chat Preview** | Talk to agents before using them — verify they understand their role |
| 🎮 **Drag & Drop Setup** | Drag agents into matchups, configure payoffs and rounds |
| 🔬 **10 Experimental Variables** | Framing, chat, protocol, goal, memory, identity, tools, horizon, validation, mock/real |
| 📊 **Run Experiments** | Execute IPD games with configurable replicates and real LLM calls |
| 📈 **Visualize Results** | Chart.js cooperation-over-time plots, payoff comparisons, Phase 2 metrics |
| 🛡️ **MCP Protocol Validation** | Three governance levels with manipulation pattern detection |
| 📥 **YAML Import/Export** | Share agent configurations as portable YAML files |

## Platform Status

| Phase | What's Built | Tests |
|-------|-------------|:---:|
| **Phase 1** | CrewAI integration, 3 framings, meta-prompting, retry logic, CIs. **Preliminary results complete.** | 120 |
| **Phase 2** | Chat phase, MCP protocol, deception metrics, memory/identity. **In progress.** | 269 |
| **Phase 3** | Sandboxed tools, violation detection, goal framing | 321 |

**All phases built. 321 tests passing. Phase 1 baseline results collected. Phase 2 experiments next.**

## Quick Start

```bash
cd 5_ai_open_claw/pd_arena
uv sync
uv run python manage.py migrate
uv run python manage.py seed_agents
uv run python manage.py runserver 8080
```

Then open [http://localhost:8080](http://localhost:8080).

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Django 5.x, Python 3.12 |
| Frontend | HTMX + Tailwind CSS (CDN) |
| Charts | Chart.js |
| Agent Config | CrewAI-style YAML |
| LLM Orchestration | CrewAI (planned) |
| Database | SQLite (dev) / PostgreSQL (prod) |
| Testing | pytest + pytest-django (321 tests) |
| Package Manager | UV |

## Project Structure

```
pd_arena/
├── agents/           # Agent CRUD, models, templates
├── games/            # Experiment setup, engine, results, protocol, violations
├── llm/              # LLM providers, prompts, mock, validation, tools
├── configs/          # CrewAI-style YAML presets
├── templates/        # Base layout, navigation
├── static/           # CSS, JS (drag-drop, charts)
└── fixtures/         # Default agent data
```

---

*Jose Pineda — Assistant Professor, California State University, Long Beach*
