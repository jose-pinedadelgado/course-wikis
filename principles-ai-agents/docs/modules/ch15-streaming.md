# Chapter 15 — Streaming Updates

## Why Streaming?

Long-running workflows can take **seconds or minutes**. Without streaming, users stare at a spinner. With streaming, they see **progress in real time**.

## What to Stream

| Update Type | Example |
|------------|---------|
| **Step transitions** | "Now processing payment..." |
| **Partial results** | Token-by-token LLM output |
| **Progress indicators** | "Step 3 of 5 complete" |
| **Debug info** | Tool call results (for dev mode) |

## Implementation

Most frameworks support streaming via:

- **Server-Sent Events (SSE)** — simple, HTTP-based
- **WebSockets** — bidirectional, more complex
- **Async iterators** — language-level streaming

!!! tip "UX Impact"
    Streaming makes agents feel **faster** even when they're not. Perceived latency drops dramatically when users see incremental progress.

??? question "Discussion: Streaming Tradeoffs"
    When might streaming actually hurt the user experience? (Think: error handling, partial results that change, information overload.)
