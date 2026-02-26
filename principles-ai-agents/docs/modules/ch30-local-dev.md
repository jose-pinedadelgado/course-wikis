# Chapter 30 — Local Development

## The Dev Loop

Building AI agents requires fast iteration:

1. **Change** a prompt, tool, or workflow
2. **Test** against a few examples
3. **Evaluate** the results
4. **Repeat**

## Local Dev Challenges

| Challenge | Why It's Hard |
|-----------|---------------|
| **API costs** | Every test call costs money |
| **Latency** | LLM calls take seconds, not milliseconds |
| **Non-determinism** | Same test, different result each run |
| **State management** | Memory, conversation history, tool state |

## Tips for Faster Iteration

!!! tip "Cache LLM Responses"
    During development, cache API responses so you're not paying (or waiting) for the same call twice. Most frameworks support this.

### Environment Setup

- **Use environment variables** for API keys (never hardcode)
- **Separate dev/staging/prod** configurations
- **Mock expensive tools** during early development
- **Use smaller models** for testing, larger for validation

### Playground & REPL

Most frameworks provide a way to interact with your agent in a **playground** — a chat interface for manual testing:

- Mastra: built-in dev playground
- LangChain: LangServe playground
- Custom: simple CLI or web UI

!!! note "The Playground Is Your Best Friend"
    Before writing evals, manually test your agent in a playground. Get the "feel" right, then automate.

??? question "Discussion: Development Workflow"
    How does AI agent development compare to traditional web development? What tools and practices are missing?
