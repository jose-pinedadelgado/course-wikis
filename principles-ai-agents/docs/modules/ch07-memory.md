# Chapter 7 — Agent Memory

## Why Memory Matters

LLMs process individual messages effectively but need help managing **longer-term context** and historical interactions.

## Types of Memory

### Working Memory

Stores **persistent, long-term user characteristics**. Example: ChatGPT's "memory" feature that remembers facts about you across conversations.

!!! example "Fun Anecdote"
    The author's ChatGPT thinks he's "a five year old girl who loves squishmellows" — because his children use his devices.

### Hierarchical Memory

Combines **recent messages** with **relevant long-term memories**. The process:

1. User sends a query
2. Search long-term memory for relevant past events
3. Include the last few messages (sliding window)
4. Join both into the context window
5. Formulate a response

In Mastra, this is configured with:

- **`lastMessages`** — sliding window of recent messages
- **`semanticRecall`** — RAG-based search through past conversations
- **`topK`** — number of past messages to retrieve
- **`messageRange`** — context around each match

!!! note "Context Window Management"
    Instead of overwhelming the model with entire conversation history, selectively include the most pertinent past interactions. This prevents context window overflow while maintaining relevance.

## Memory Processors

Sometimes you want to **deliberately prune** the context window:

| Processor | Purpose |
|-----------|---------|
| **`TokenLimiter`** | Prevents exceeding context window limits; removes oldest messages first |
| **`ToolCallFilter`** | Removes tool call results from memory; saves tokens when past tool outputs aren't needed |

!!! tip "As Context Windows Grow..."
    Many developers start by throwing everything into the context window and set up memory later. This is fine for prototyping!

??? question "Discussion: Memory Architecture"
    How does agent memory compare to human memory (working memory, short-term, long-term)? What are the limitations of the current approach?
