# 🤖 Bamboo Assistant (AI Chatbot)

A conversational AI assistant powered by OpenAI GPT-4o-mini that understands your financial data and provides personalized insights, visualizations, and recommendations.

## Overview

The Bamboo Assistant lives as a floating widget (💬) in the bottom-right corner of every page. It can answer natural language questions about your spending, budgets, goals, and cash flow.

## Architecture

```
User Question → chatbot_llm.py → Build Financial Context → OpenAI GPT-4o-mini → Structured JSON → Render
```

### Context Builder

Every query sends your real financial data to the LLM:

- **Current month spending** by category (with transaction counts)
- **Income** totals and net calculation
- **Budget status** — spent vs. limit for each category
- **Recent transactions** (last 15)
- **Savings goals** — progress percentages
- **Accounts** — balances by type
- **Unread alerts**
- **Net worth** — assets vs. liabilities
- **Recurring transactions** — active subscriptions
- **Summary statistics** — averages, largest transaction

### Response Format

The LLM returns structured JSON with three components:

| Component | Description |
|---|---|
| `message` | Conversational text response |
| `visualization` | Optional chart (bar, pie, table, sankey) |
| `suggested_questions` | 3 follow-up question buttons |

## Visualizations

The chatbot can render inline visualizations:

- **Bar charts** — horizontal bars with labels and amounts
- **Pie charts** — rendered as bar charts (CSS-only, no canvas)
- **Tables** — headers + rows for detailed data
- **Sankey link** — green button linking to the Cash Flow diagram

## Chat Window Modes

The chatbot supports 4 size modes via header buttons:

| Icon | Mode | Description |
|---|---|---|
| ▫ | Small | Default floating panel (380px) |
| ◧ | Half-vertical | Right 50% of screen |
| ⬓ | Half-horizontal | Bottom 50% of screen |
| ⬜ | Full screen | Entire viewport |

## Thinking Animation

While waiting for the LLM response (~2-5 seconds), a **sprouting seed** animation plays:

1. 🌱 Seed appears
2. Stem grows upward
3. Leaves sprout on both sides
4. "Growing your answer..." text pulses
5. Animation loops until response arrives

## Conversation Memory

The assistant maintains conversation history (last 10 exchanges) per user session. This allows follow-up questions like:

> "How am I doing on my budget?" → "Which category should I cut back on?" → "Show me the transactions for that category"

Memory resets on server restart (in-memory storage).

## Fallback

If the OpenAI API key is missing or the API call fails, the chatbot falls back to the original **keyword-based NLU** (`chatbot.py`) which handles common queries via regex pattern matching.

## Example Queries

- "How much did I spend this month?"
- "Show me my spending breakdown"
- "Am I over budget on anything?"
- "What are my biggest expenses?"
- "Show me my cash flow" *(triggers Sankey button)*
- "How are my savings goals going?"
- "What's my net worth?"

## Configuration

| Setting | Value |
|---|---|
| Model | `gpt-4o-mini` |
| Temperature | 0.7 |
| Max tokens | 1,000 |
| History depth | 10 exchanges |
| API key | `OPENAI_API_KEY` environment variable |

## Future Options

See `docs/TODO_CHATBOT.md` for planned alternatives:

1. Gemini Flash (free tier)
2. Local Ollama model
3. Enhanced local NLU (no API)
4. Anthropic Claude Haiku
5. Hybrid local-first + LLM fallback
