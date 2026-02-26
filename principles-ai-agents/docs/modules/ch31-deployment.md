# Chapter 31 — Deployment

## Serverless Challenges

AI agents are **poor fits for traditional serverless** (AWS Lambda, Vercel Functions):

| Constraint | Why It's a Problem |
|-----------|-------------------|
| **Execution time limits** | Agent interactions can take minutes |
| **Cold starts** | Slow startup + LLM latency = bad UX |
| **Statelessness** | Agents need persistent memory/state |
| **Connection limits** | Streaming requires long-lived connections |

## Deployment Options

### Long-Running Servers

Traditional server deployment (EC2, Railway, Fly.io) — the simplest approach:

- No time limits
- Easy to maintain WebSocket/SSE connections
- Can hold state in memory

### Serverless with Workarounds

If you must use serverless:

- **Background jobs** for long-running tasks
- **External state store** (Redis, database) for memory
- **Queues** (SQS, Bull) for async processing

### Managed Agent Platforms

Emerging platforms that handle agent-specific infrastructure:

- Scaling based on concurrent conversations
- Built-in memory persistence
- Streaming support out of the box

!!! tip "Start Simple"
    Deploy to a single server (Railway, Fly.io, or a VPS). Only add complexity when you hit actual scaling issues.

## Production Considerations

- **Rate limiting** — protect against runaway costs
- **Timeouts** — kill stuck agent loops
- **Logging** — capture every interaction for debugging
- **Cost monitoring** — alerts for unexpected spend spikes

??? question "Discussion: Scaling AI Agents"
    What's different about scaling AI agents vs. scaling a traditional web API? What bottlenecks appear first?
