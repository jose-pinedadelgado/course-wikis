# Chapter 2 — Choosing a Provider and Model

## Decision Framework

### 1. Hosted vs. Open-Source

!!! tip "Start with Hosted"
    Always prototype with cloud APIs (OpenAI, Anthropic, Google Gemini) — even if you plan to use open-source later. Otherwise you'll debug infra instead of iterating on your product.

Use a **model routing library** so you can swap providers without rewriting code.

### 2. Model Size: Accuracy vs. Cost/Latency

| Model Tier | Characteristics | When to Use |
|-----------|-----------------|-------------|
| **Large** (e.g., GPT-4o, Claude Opus) | Expensive, accurate, slower | Prototyping, complex reasoning |
| **Small** (e.g., GPT-4o-mini, Haiku) | Cheap, fast, less accurate | Production at scale, simple tasks |

!!! tip "Prototype Expensive, Optimize Later"
    Start with larger models to validate your approach. Once it works, downgrade to reduce cost.

### 3. Context Window Size

The **context window** determines how many tokens a model can process. Larger windows let you feed more data without complex retrieval.

- **Gemini Flash 1.5 Pro**: 2M tokens (~4,000 pages of text)
- Use case: feed an entire codebase to a support assistant

### 4. Reasoning Models

Reasoning models do extensive internal logic before responding. They may take seconds or minutes but deliver higher-quality analysis.

**Key advances:**

- **Chain-of-thought prompting** — models show their work step by step
- **Chain of draft** / **chain of preference optimization** — stay focused, skip fluff

!!! warning "Reasoning Models Need Context"
    Think of them as "report generators." Give lots of context via many-shot prompting. Without it, they go off the rails.

    **Suggested reading:** "o1 isn't a chat model" by Ben Hylak

??? question "Discussion: Model Selection Tradeoffs"
    Your startup has a $500/month LLM budget. You need a customer support agent that handles 10,000 queries/day. How do you approach model selection? What's your routing strategy?
