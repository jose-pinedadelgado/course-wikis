# Chapter 8 — Dynamic Agents

## The Problem with Static Agents

The simplest agent configuration is static: a fixed system prompt, model, and tool set. But what if you need to change these **at runtime**?

## What Are Dynamic Agents?

A dynamic agent's properties — **instructions, model, and available tools** — are determined at runtime based on user input, environment, or other context.

!!! example "Dynamic Support Agent"
    A support agent that adjusts behavior based on:
    
    - **Subscription tier** — Premium users get more detailed responses, access to advanced tools
    - **Language preferences** — System prompt switches language
    - **Time of day** — Different routing during business hours vs. after-hours

## Tradeoff: Predictability vs. Power

| Approach | Predictability | Flexibility |
|----------|---------------|-------------|
| **Static** | High — same behavior every time | Low — can't adapt |
| **Dynamic** | Lower — behavior varies | High — context-aware |

!!! note "When to Go Dynamic"
    Use dynamic agents when you need to serve different user segments, adapt to changing contexts, or A/B test different configurations.

??? question "Discussion: Dynamic Agent Risks"
    What could go wrong if an agent changes its own tools or instructions at runtime? How do you test dynamic agents?
