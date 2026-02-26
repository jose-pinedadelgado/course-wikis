# Adding Agents

## Creating a New Personality Agent

1. Add a new class to `src/pd_phase2/agents/personality.py`:

```python
class MyNewAgent(PersonalityAgent):
    """Description of the agent's strategy."""
    
    def __init__(self, param: int = 5) -> None:
        self._param = param
    
    def reset(self, seed: int | None = None) -> None:
        # Reset internal state
        pass
    
    def act(self, obs: Observation) -> str:
        # Return "C" or "D"
        return "C"
    
    def chat(self, obs: Observation, incoming_message: str | None) -> str | None:
        # Return a message or None
        return "I'll cooperate!"
```

2. Register it in `src/pd_phase2/runners/registry.py`:

```python
PERSONALITY_AGENTS = {
    "naive_cooperator": NaiveCooperator,
    "aware_cooperator": AwareCooperator,
    # ... existing agents
    "my_new_agent": MyNewAgent,
}
```

3. Write tests in `tests/`:

```python
def test_my_new_agent_cooperates_initially():
    agent = MyNewAgent()
    obs = Observation(history=[], round_num=0, max_rounds=100)
    assert agent.act(obs) == "C"
```

4. Add to a YAML config:

```yaml
conditions:
  - name: "MyNew_vs_TFT"
    agent_a:
      ref: "agents/personality.yaml"
      overrides: {personality: "my_new_agent"}
    agent_b:
      ref: "agents/policy.yaml"
      overrides: {policy: "tft"}
```

## Creating a Policy Agent

Policy agents are simpler (no chat, fully deterministic):

```python
class MyPolicy(PolicyAgent):
    def act(self, obs: Observation) -> str:
        if not obs.history:
            return "C"
        # Your logic here
        return "D"
```

## Testing Requirements

Every new agent needs:

- [x] Unit test for initial action (no history)
- [x] Unit test for action after cooperation
- [x] Unit test for action after exploitation
- [x] Unit test for action after mutual defection
- [x] Chat test (if CommunicatingAgent)
- [x] Reset test (state clears properly)
