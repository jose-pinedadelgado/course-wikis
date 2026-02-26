# Chapter 21 — Multi-Agent Systems 101

## Why Multiple Agents?

!!! quote "The Core Insight"
    **Agent design ≈ organizational design.** Just as companies organize employees into roles and teams, complex AI systems organize agents into specialized units.

## The Org Chart Metaphor

Think about how you'd hire for a task:

1. **What roles are needed?** (analyst, writer, reviewer)
2. **What does each role do?** (job description = system prompt + tools)
3. **Who reports to whom?** (supervisor hierarchy)
4. **How do they communicate?** (handoffs, shared state)

## When to Use Multi-Agent

| Scenario | Single Agent | Multi-Agent |
|----------|-------------|-------------|
| Simple Q&A | ✅ | Overkill |
| Customer support with routing | ✅ (with tools) | ✅ |
| Complex research + writing | Struggles | ✅ |
| Cross-domain pipeline | Poor quality | ✅ |

!!! tip "Don't Default to Multi-Agent"
    Multi-agent systems add complexity. Start with a single agent + good tools. Only split into multiple agents when a single agent **demonstrably struggles** with the breadth of tasks.

??? question "Discussion: Agent vs. Human Teams"
    What organizational patterns (matrix, flat, hierarchical) map best to multi-agent architectures? Where does the analogy break down?
