# Chapter 33 — Code Generation

## The Biggest Agent Use Case

Code generation is arguably the **most successful AI agent application** as of 2025:

| Tool | Approach |
|------|----------|
| **Cursor** | IDE with AI-native editing, chat, and codebase understanding |
| **Windsurf** | Similar to Cursor, focused on "flow" state |
| **Replit** | Cloud IDE with agent that builds full apps |
| **bolt.new** | Browser-based, generates and deploys apps from prompts |
| **v0** | Vercel's UI generation tool |

## Why Code Gen Works Well

1. **Clear success criteria** — code either runs or it doesn't
2. **Rich feedback** — compiler errors, test results, runtime behavior
3. **Iterative** — agents can try, fail, and retry quickly
4. **Large training corpus** — models have seen billions of lines of code

## Challenges

- **Hallucinated APIs** — model invents functions that don't exist
- **Context limits** — large codebases don't fit in one prompt
- **Security** — generated code may have vulnerabilities
- **Dependency management** — keeping generated code compatible

!!! tip "Code Review Is Essential"
    Never deploy AI-generated code without review. Use it as a **draft** that a human refines, not a finished product.

??? question "Discussion: The Future of Programming"
    If AI can generate code, what skills do developers still need? How does the role of a software engineer change?
