# Chapter 6 — Tool Calling

## What Are Tools?

Tools are **functions that agents can call** to perform specific tasks — fetching weather data, querying a database, processing calculations.

The key to effective tool use is **clear communication** with the model about what each tool does and when to use it.

## Best Practices

- **Detailed descriptions** in the tool definition and system prompt
- **Specific input/output schemas** with clear types
- **Semantic naming** — `multiplyNumbers` not `doStuff`
- Describe both **what it does** and **when to call it**

!!! warning "The Most Important Step"
    **Tool design is the single most important thing** when creating an AI application. Before coding, write out:
    
    1. What is the full list of tools you'll need?
    2. What will each of them do?

## Case Study: Alana's Book Recommendation Agent

**Goal:** Intelligent recommendations from a corpus of investor book recommendations.

### First Attempt (Failed)
Dropped all books into the agent's knowledge window → agent couldn't reason about the data in a structured way.

### Better Approach
Broke the problem into **specific tools**:

| Tool | Purpose |
|------|---------|
| Access investor corpus | Get the list of investors |
| Book recommendations | Get recommendations per investor |
| Books by genre | Filter by category |
| Sort by recommender type | Founders, investors, etc. |

### Result
The agent could now intelligently analyze the corpus, answer nuanced questions, and provide useful recommendations — like a skilled human analyst.

!!! quote "Key Takeaway"
    **Think like an analyst.** If a human would follow specific operations or queries, write those operations as tools your agent can use.

??? question "Discussion: Tool Granularity"
    How fine-grained should your tools be? What's the tradeoff between many small tools vs. fewer large ones? When does an agent have *too many* tools?
