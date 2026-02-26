# Chapter 16 — Observability and Tracing

## The Challenge

AI agents are **non-deterministic**. The same input can produce different outputs. Traditional debugging (set breakpoint, inspect state) doesn't work well.

## Tracing

A **trace** captures the full execution path of an agent interaction:

- Each LLM call with prompts and responses
- Tool invocations with inputs and outputs
- Timing data for each step
- Token usage and cost

## Observability Platforms

| Platform | Notes |
|----------|-------|
| **LangSmith** | LangChain's platform, popular |
| **Langfuse** | Open-source alternative |
| **Braintrust** | Focus on evals + observability |
| **Custom** | Most frameworks support OpenTelemetry-style hooks |

!!! warning "Do This From Day 1"
    Don't wait until something breaks. Set up tracing early — it's the only way to understand what your agent is actually doing.

## What to Monitor

- **Latency** per step and overall
- **Token usage** and cost per interaction
- **Error rates** by step and tool
- **User satisfaction** signals

??? question "Discussion: Debugging Non-Determinism"
    How do you write tests for a system that gives different answers each time? What does "correct" mean for an agent?
