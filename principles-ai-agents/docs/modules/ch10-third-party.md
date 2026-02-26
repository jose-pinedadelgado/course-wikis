# Chapter 10 — Popular Third-Party Tools

## Web Scraping & Browser Use

| Category | Examples | Notes |
|----------|----------|-------|
| **Cloud search APIs** | Exa, Browserbase, Tavily | Hosted, easy to integrate |
| **Low-level tools** | Playwright (Microsoft) | Pre-LLM era, powerful but manual |
| **Agentic search** | Stagehand (JS), Browser Use (Python) | Natural language APIs for scraping |

### Common Challenges

- **Anti-bot detection** — fingerprinting, WAFs, captchas
- **Fragile setups** — break when target websites change layout/CSS

!!! tip "Budget time for glue work — these challenges are solvable but not trivial."

## Third-Party Integrations

Most agents need a **core set of integrations**:

- Email (Gmail, Outlook)
- Calendar (Google Calendar)
- Documents (Google Docs, Notion)

Plus **domain-specific** integrations:

| Domain | Integrations |
|--------|-------------|
| Sales | Salesforce, Gong |
| HR | Rippling, Workday |
| Engineering | GitHub, Jira |

### Integration Platforms (iPaaS)

| Tier | Examples | Cost |
|------|----------|------|
| **Developer-friendly** | Composio, Pipedream, Apify | $10s–$100s/month |
| **Enterprise** | Specialized solutions | $1,000s/month |

??? question "Discussion: Build vs. Buy"
    When should you build your own integrations vs. use an iPaaS? What are the long-term implications of each approach?
