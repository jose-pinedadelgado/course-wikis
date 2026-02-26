# Chapter 26 — Multi-Agent Standards

## The Interoperability Challenge

When agents are built by **different teams or companies**, they need a shared protocol to communicate. MCP handles tool-calling; **A2A** handles agent-to-agent communication.

## Agent-to-Agent Protocol (A2A)

Proposed by **Google** in April 2025 (shortly after MCP gained traction):

| Feature | MCP | A2A |
|---------|-----|-----|
| **Purpose** | Connect agents to tools | Connect agents to agents |
| **Metaphor** | USB-C for tools | HTTP for agents |
| **Unit of work** | Tool call | Task |
| **State** | Stateless | Stateful (tasks have lifecycle) |

### A2A Concepts

- **Agent Card** — JSON description of what an agent can do (like an MCP tool description, but for agents)
- **Task** — A unit of work with states: submitted → working → completed/failed
- **Message** / **Artifact** — Communication between agents

!!! note "Early Days"
    A2A was proposed April 2025. It's conceptually promising but adoption is still nascent. Watch this space.

## MCP + A2A Together

- **MCP** = how agents use tools
- **A2A** = how agents talk to each other

They're complementary, not competing.

??? question "Discussion: Standards Adoption"
    Will A2A achieve the same adoption as MCP? What factors determine success? (Consider: Google vs. Anthropic backing, complexity, developer experience.)
