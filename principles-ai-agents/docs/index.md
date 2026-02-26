# Principles of Building AI Agents

**Author:** Sam Bhagwat (Cofounder & CEO, Mastra.ai)  
**Edition:** 2nd Edition (May 2025)

---

## About This Wiki

This is a **teaching companion** for *Principles of Building AI Agents* by Sam Bhagwat. It distills each chapter into structured notes with key concepts, takeaways, and discussion prompts for classroom use.

!!! info "About the Book"
    Written by the co-founder of Gatsby and Mastra (an open-source JavaScript agent framework), this book covers the practical fundamentals of building AI agents — from prompting and tool calling to RAG, multi-agent systems, and deployment. It's intentionally short and code-focused.

## Book Structure

| Part | Chapters | Focus |
|------|----------|-------|
| **I** — Prompting LLMs | 1–3 | History, model selection, prompt engineering |
| **II** — Building an Agent | 4–9 | Agent architecture, tools, memory, middleware |
| **III** — Tools & MCP | 10–11 | Third-party tools, Model Context Protocol |
| **IV** — Graph-Based Workflows | 12–16 | Workflows, branching, streaming, observability |
| **V** — RAG | 17–20 | Retrieval-augmented generation pipeline |
| **VI** — Multi-Agent Systems | 21–26 | Supervisors, control flow, A2A protocol |
| **VII** — Evals | 27–29 | Testing and evaluation strategies |
| **VIII** — Dev & Deployment | 30–31 | Local development, serverless challenges |
| **IX** — Everything Else | 32–34 | Multimodal, code gen, future directions |

## Key Themes

- **Start simple, iterate** — Prototype with hosted models before optimizing
- **Tool design is the most important step** — Think like an analyst, break problems into reusable operations
- **Agents are AI employees, not contractors** — They maintain context, have roles, and use tools
- **Fight over-engineering** — Especially with RAG; try full context loading first
- **Organizational design ≈ Multi-agent design** — Group tasks into job descriptions, manage hierarchy

## How to Use This Wiki

1. **Before class:** Read the chapter summary for the week's topic
2. **During class:** Use discussion questions and "Think About It" prompts
3. **After class:** Try the code patterns with Mastra or your framework of choice

!!! tip "Framework-Agnostic"
    While the book uses Mastra (JavaScript) for code examples, the concepts apply to any agent framework: LangChain, CrewAI, AutoGen, etc.
