# Chapter 25 — Combining the Patterns

## Real-World Systems Use Everything

Production AI systems rarely use just one pattern. They combine:

- **Agents** for open-ended reasoning
- **Workflows** for reliable pipelines
- **RAG** for knowledge retrieval
- **Multi-agent** for task distribution
- **Tools** (including workflow-tools) for actions

## Architecture Examples

### Customer Support Platform

```
User → Routing Agent (classifier)
  ├── FAQ Workflow (RAG + template)
  ├── Billing Agent (tools: Stripe, refund workflow)
  └── Technical Agent (tools: Jira, docs RAG, escalation workflow)
```

### Content Generation Pipeline

```
User brief → Research Agent (web search, RAG)
  → Writing Agent (structured output)
  → Editor Agent (review, fact-check)
  → Publishing Workflow (format, schedule, deploy)
```

!!! note "Start Simple, Add Complexity"
    Begin with a single agent. Add workflows when you need reliability. Add more agents when breadth becomes a problem. Add RAG when context is insufficient.

??? question "Discussion: Architecture Decisions"
    Given a new AI product idea, how do you decide which patterns to use? What's your decision framework?
