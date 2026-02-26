# Chapter 11 — Model Context Protocol (MCP)

## What Is MCP?

Proposed by Anthropic in **November 2024**, MCP is an open protocol for connecting AI agents to tools, models, and each other.

!!! quote "The USB-C Analogy"
    Think of MCP like a **USB-C port for AI applications**. If your tool or agent "speaks" MCP, it can plug into any MCP-compatible system — regardless of who built it or what language it's in.

### Timeline

| Date | Event |
|------|-------|
| Nov 2024 | Anthropic proposes MCP |
| Mar 2025 | Critical mass after Shopify CEO Tobi Lutke champions it |
| Apr 2025 | OpenAI and Google Gemini announce MCP support → becomes the default |

## MCP Primitives

| Primitive | Role |
|-----------|------|
| **Servers** | Wrap sets of tools; can be written in any language; communicate over HTTP |
| **Clients** | Models or agents that query servers for available tools, then request execution |

MCP is essentially a **standard for remote code execution**, like OpenAPI or RPC.

## The MCP Ecosystem

- **Vendors** (Stripe, etc.) ship MCP servers for their APIs
- **Independent devs** publish servers on GitHub (browser use, etc.)
- **Registries** (Smithery, PulseMCP, mcp.run) catalogue and validate servers
- **Frameworks** (Mastra) ship client/server abstractions

## When to Use MCP

- **Building an agent** that needs many third-party integrations → build an MCP client
- **Building a tool** that other agents should use → ship an MCP server

## Open Challenges

1. **Discovery** — No centralized way to find MCP tools (registries are fragmented)
2. **Quality** — No equivalent of NPM scoring or verification badges (yet)
3. **Configuration** — Each provider has its own schema; clients don't always fully implement the spec

!!! warning "Don't Roll Your Own"
    You could spend a weekend debugging subtle differences between Cursor and Windsurf's MCP clients. Use a good framework or library.

??? question "Discussion: Protocol Wars"
    How does MCP's adoption path compare to other standards (USB, HTTP, OAuth)? What determines whether a protocol becomes the default?
