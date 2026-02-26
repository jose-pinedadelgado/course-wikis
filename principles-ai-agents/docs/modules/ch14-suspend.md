# Chapter 14 — Suspend and Resume

## The Problem

Some workflow steps require **human input** or **external events** before continuing. You can't just block a server thread waiting for someone to respond.

## Solution: Suspend and Resume

When a workflow hits a point that needs human interaction:

1. **Serialize** the workflow state (current step, accumulated data)
2. **Persist** it to a database
3. **Suspend** execution
4. When the human responds, **resume** from the saved state

!!! example "Use Cases"
    - **Approval workflows** — "Does this look right before I proceed?"
    - **Clarification** — "Which of these three options did you mean?"
    - **External data** — waiting for a third-party API callback

## Implementation Considerations

- Workflow state must be **serializable** (no closures, no live connections)
- Need a **persistence layer** (database, Redis, etc.)
- Resume must handle **stale state** (what if the world changed while suspended?)

??? question "Discussion: Human-in-the-Loop Design"
    How do you design the "pause points" in a workflow? What's the user experience when a workflow suspends — how do you communicate what's happening and what's needed?
