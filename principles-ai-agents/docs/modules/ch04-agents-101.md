# Chapter 4 — Agents 101

## What Is an Agent?

Direct LLM calls work for **one-shot transformations** ("given a transcript, write a description"). For **ongoing, complex interactions**, you need an **agent**.

!!! quote "Core Metaphor"
    Think of agents as **AI employees rather than contractors** — they maintain context, have specific roles, and can use tools to accomplish tasks.

## Levels of Autonomy

Like self-driving cars, agents exist on a spectrum:

| Level | Capabilities | Examples |
|-------|-------------|----------|
| **Low** | Binary choices in a decision tree | Simple chatbot routing |
| **Medium** | Memory, tool calling, retry failed tasks | Customer support agent |
| **High** | Planning, task decomposition, queue management | Code-generation agents (Replit, Cursor) |

!!! note "This Book's Focus"
    Mostly covers low-to-medium autonomy. High-autonomy agents are still rare in production (as of 2025).

## Agent Components (in Mastra)

An agent in Mastra has:

- **Persistent memory** — maintains context across interactions
- **Consistent model configuration** — fixed provider/model settings
- **Tool access** — suite of callable functions and workflows

??? question "Discussion: Autonomy Tradeoffs"
    What are the risks of giving agents higher autonomy? When is low autonomy preferable, even if the model is capable of more?
