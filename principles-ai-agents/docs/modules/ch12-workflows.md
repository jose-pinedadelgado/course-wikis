# Chapter 12 — Workflows 101

## Why Workflows?

Sometimes you need **reliable, repeatable steps** rather than open-ended agent behavior. Workflows are **graph-based, step-by-step processes** that can include LLM calls alongside deterministic logic.

## Agent vs. Workflow

| Feature | Agent | Workflow |
|---------|-------|----------|
| **Control** | LLM decides what to do next | Developer defines the path |
| **Reliability** | Variable — depends on model | High — deterministic control flow |
| **Flexibility** | High — handles novel situations | Lower — follows defined paths |
| **Use case** | Open-ended tasks | Business processes, pipelines |

!!! note "Not Either/Or"
    Agents can **call workflows as tools**, and workflows can include agent steps. The best systems combine both.

## Workflow Basics

A workflow is a **directed acyclic graph (DAG)** of steps:

1. Define steps with inputs/outputs
2. Connect them in sequence or parallel
3. Each step can be deterministic code, an LLM call, or a tool invocation

??? question "Discussion: When to Use Each"
    Your company processes insurance claims. Some are straightforward (auto-approve), others need human review. How would you combine agents and workflows?
