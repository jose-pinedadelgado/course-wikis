# Chapter 24 — Workflows as Tools

## The Hybrid Pattern

Instead of giving agents raw tool calls, wrap **entire workflows** as tools. The agent decides *when* to run a workflow, but the workflow itself is deterministic.

!!! example "Research Agent"
    The agent has a `deep_research` workflow-tool that:
    
    1. Searches multiple sources (parallel)
    2. Extracts key findings (LLM)
    3. Cross-references facts (deterministic)
    4. Generates a summary (LLM)
    
    The agent calls it like any other tool, but gets a reliable multi-step process.

## Benefits

| Benefit | Explanation |
|---------|-------------|
| **Reliability** | Workflow steps are predictable |
| **Composability** | Build complex tools from simple steps |
| **Testability** | Test the workflow independently of the agent |
| **Reusability** | Same workflow, multiple agents |

!!! tip "This Is Often the Sweet Spot"
    Agent for decision-making + workflow for execution = best of both worlds.

??? question "Discussion: Abstraction Level"
    How do you decide what goes inside a workflow-tool vs. what the agent handles directly? What's the right level of abstraction?
