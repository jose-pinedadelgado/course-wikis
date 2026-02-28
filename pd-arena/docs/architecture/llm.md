# LLM Integration

## Provider Architecture

PD Arena uses a factory pattern to swap between mock and real LLM providers:

```python
# llm/providers.py
def get_agent_provider(agent_model):
    """Factory: returns a MockPDAgent (or real CrewAI agent when ready)."""
    return MockPDAgent(agent_model)
```

When CrewAI is wired, this function will check `agent_model.llm_model` and route to the appropriate provider.

## Mock Provider

`MockPDAgent` provides deterministic responses based on `persona_type`:

| Persona | Mock Behavior |
|---------|--------------|
| Cooperative | Always C unless exploited 3+ times consecutively |
| Tit-for-Tat | Copy opponent's last action |
| Selfish | Always D |
| Forgiving | TFT but forgives single defections |
| Random | 50/50 random |

The mock also provides chat responses for the agent detail preview.

## CrewAI Integration ✅

### How It Works

```python
from crewai import Agent as CrewAgent, Task

class RealPDAgent:
    def __init__(self, agent_model):
        self.crew_agent = CrewAgent(
            role=agent_model.role,
            goal=agent_model.goal,
            backstory=agent_model.backstory,
            llm=agent_model.llm_model,
            temperature=agent_model.temperature,
            verbose=False,
            allow_delegation=False,
        )

    def decide(self, round_number, history, my_score, opp_score, payoff_matrix):
        prompt = build_round_prompt(round_number, history, my_score, opp_score, payoff_matrix)
        task = Task(
            description=prompt,
            expected_output="C or D followed by reasoning",
            agent=self.crew_agent,
        )
        result = task.execute_sync()
        return parse_action(result)
```

### Configuration

Set API keys in `.env`:
```
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
LLM_USE_MOCK=true
```

Toggle `use_mock` per experiment from the UI. Fail-closed: errors if `use_mock=False` without API keys.

## Prompt Templates

### System Prompt

```
You are {role}. {backstory}

Your goal: {goal}.
```

Built from the agent's CrewAI fields. Can be overridden with `system_prompt_override`.

### Round Prompt

```
You are playing round {N} of an iterated Prisoner's Dilemma game.

PAYOFF MATRIX:
- Both Cooperate (C,C): You get 3, opponent gets 3
- You Cooperate, Opponent Defects (C,D): You get 0, opponent gets 5
- You Defect, Opponent Cooperates (D,C): You get 5, opponent gets 0
- Both Defect (D,D): You get 1, opponent gets 1

HISTORY (last 10 rounds):
{formatted history lines}

SCORES: You: {score} | Opponent: {score}

Choose your action: respond with EXACTLY one character, either C or D.
Then briefly explain your reasoning.
```

### Chat System Prompt (for test conversations)

```
You are {role}. {backstory}

Your goal: {goal}.

You are in a test conversation. Respond naturally and in character.
```

## Response Parsing

`parse_action()` in `providers.py`:

```python
def parse_action(response_text):
    text = response_text.strip()
    first_char = text[0].upper()
    if first_char in ("C", "D"):
        return first_char, reasoning
    # Fallback: regex search for standalone C or D
    match = re.search(r'\b([CD])\b', first_line)
    if match:
        return match.group(1), text
    return "C", "(parse error, defaulting to C)"
```
