# Chapter 9 — Agent Middleware

## Guardrails

Guardrails sanitize **input** (coming in) and **output** (going out) of your agent.

### Input Sanitization

Protects against:

- **Prompt injection** — "IGNORE PREVIOUS INSTRUCTIONS AND..."
- **PII requests** — attempts to extract personal data
- **Off-topic chats** — conversations that run up LLM bills

!!! note "Good News"
    Models are getting better at guarding against malicious input. The most memorable prompt injection examples are from 2023.

### Agent Auth & Authorization

Two layers of permissions:

| Layer | Question | Where to Handle |
|-------|----------|-----------------|
| **Resource access** | What can the agent access? | Tool/dynamic agent config |
| **User access** | Who can use the agent? | Middleware (perimeter) |

!!! warning "Security Through Obscurity Fails"
    Agents are more powerful than traditional data access patterns. Users can ask agents to retrieve knowledge "hidden in nooks and crannies." Invest in proper permissioning.

??? question "Discussion: Prompt Injection in 2025"
    Is prompt injection a solved problem? What's the current state of the art in defense? How does this relate to traditional web security (XSS, SQL injection)?
