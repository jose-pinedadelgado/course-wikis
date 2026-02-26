# Chapter 23 — Control Flow

## Handoffs vs. Delegation

| Pattern | Description | When to Use |
|---------|-------------|-------------|
| **Handoff** | Control passes completely to another agent | Agent A finishes, Agent B takes over |
| **Delegation** | Agent A asks Agent B for help, then resumes | Sub-task within a larger flow |
| **Broadcast** | Request goes to all agents simultaneously | Need multiple perspectives |

## Sequential vs. Parallel

- **Sequential**: Agent A → Agent B → Agent C (each builds on previous output)
- **Parallel**: Agents A, B, C run simultaneously (results merged at end)
- **Pipeline**: Like sequential, but with data transformation at each step

## Shared State

Agents in a multi-agent system often need **shared context**:

- **Message history** — what's been said so far
- **Artifacts** — documents, data structures being built
- **Metadata** — user info, session state

!!! warning "State Management Is Hard"
    The more agents share state, the more complex debugging becomes. Keep shared state minimal and well-defined.

??? question "Discussion: Concurrency in Multi-Agent Systems"
    How do you handle race conditions when multiple agents modify shared state? What patterns from distributed systems apply?
