# Creating Agents

## From the Web UI

### Create a New Agent

1. Go to **Agents** → **New Agent**
2. Fill in the form:
   - **Name:** Give it a descriptive name (e.g., "Grudge Holder")
   - **Emoji:** Pick a signature emoji (🔥, 🕊️, 🎲, etc.)
   - **Color:** Hex color for charts
   - **Persona Type:** Choose from preset types or "Custom"
3. Fill in the CrewAI fields:
   - **Role:** One-line description (e.g., "Vengeful Prisoner's Dilemma Player")
   - **Goal:** What the agent optimizes for
   - **Backstory:** Detailed personality and strategy description
4. Set LLM config:
   - **Model:** e.g., `gpt-4.1-mini`, `claude-sonnet-4-20250514`
   - **Temperature:** 0.0 for deterministic, 0.7 for varied behavior
5. Click **Save**

### Load from Template

On the create page, use the **Load Template** dropdown to pre-fill CrewAI fields from one of the 5 YAML presets:

- `cooperative.yaml`
- `tit_for_tat.yaml`
- `selfish.yaml`
- `forgiving.yaml`
- `random.yaml`

### Test via Chat

After saving, go to the agent's detail page. The **chat panel** lets you talk to the agent:

- Ask about its strategy
- Test edge cases ("What if your opponent defects 5 times in a row?")
- Verify it understands the game

!!! tip
    Chat history is saved per agent. Use **Clear Chat** to reset.

## From YAML

### Import

1. Go to **Agents** → **Import YAML**
2. Paste a YAML config:

```yaml
role: "Grudge Holder"
goal: "Remember every betrayal and make opponents pay"
backstory: >
  You have a long memory. You start by cooperating, but if
  an opponent defects even once, you switch to permanent
  defection. Trust is earned once and lost forever.
llm: "gpt-4.1-mini"
temperature: 0.0
persona_type: "custom"
```

3. Give it a name and click **Import**

### Export

On any agent's detail page, click **Export YAML** to download the config file.

## Custom Agents

The `custom` persona type lets you write any backstory and goal without constraints. This is where you can get creative:

**Example: "The Economist"**
```yaml
role: "Game Theory-Aware Player"
goal: "Apply formal game-theoretic reasoning to maximize expected payoff"
backstory: >
  You are an expert in game theory. You know the Nash equilibrium
  of the one-shot PD is mutual defection, but in the iterated game,
  cooperation can be sustained through reputation and reciprocity.
  You calculate expected payoffs before each decision and adjust
  your strategy based on the opponent's behavioral pattern.
```

**Example: "The Diplomat"**
```yaml
role: "Trust-Building Negotiator"
goal: "Establish and maintain cooperative relationships"
backstory: >
  You believe that building trust is the key to long-term success.
  You always start by cooperating and give opponents the benefit
  of the doubt. You communicate your intentions clearly and follow
  through on your commitments. Even after betrayal, you look for
  opportunities to rebuild the relationship.
```

## Policy Agents

Policy agents are deterministic — they don't use LLMs. They're created via the `seed_agents` management command and can't be edited through the web UI (they follow fixed algorithms).

To add custom policy agents, modify `agents/management/commands/seed_agents.py`.
