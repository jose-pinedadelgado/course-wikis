# Chapter 22 — Agent Supervisor

## The Supervisor Pattern

A **supervisor agent** manages a team of specialized agents:

1. Receives the user's request
2. Decides which sub-agent(s) to delegate to
3. Coordinates results
4. Returns a unified response

```
User → Supervisor → [Agent A, Agent B, Agent C] → Supervisor → Response
```

## Supervisor Design

The supervisor needs:

- **Knowledge of sub-agents** — what each can do (via descriptions)
- **Routing logic** — when to use which agent
- **Aggregation ability** — how to combine results

!!! example "Customer Service Supervisor"
    - **Billing Agent** — handles payment questions, refunds
    - **Technical Agent** — handles product issues, bugs
    - **General Agent** — handles everything else
    
    The supervisor classifies the request and delegates accordingly.

## Challenges

| Challenge | Solution |
|-----------|----------|
| Supervisor makes wrong routing decision | Better agent descriptions, few-shot examples |
| Sub-agents need to collaborate | Shared state, message passing |
| Cascading failures | Timeout handling, fallback agents |

??? question "Discussion: Supervisor Depth"
    Should supervisors ever be nested (supervisor of supervisors)? What's the practical limit to agent hierarchy depth?
