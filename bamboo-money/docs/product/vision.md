# Vision & Strategy

## Product Vision

> **"The privacy-first budget app that reads your statements so you don't have to."**

Bamboo Money is a self-hosted personal finance app that helps users understand and control their spending through file-based import, intelligent categorization, and actionable budget insights — all without requiring bank account linking.

## The Problem

Personal finance apps face a fundamental tension:

| Approach | Benefit | Cost |
|---|---|---|
| **Bank linking (Plaid)** | Automatic transaction sync | Users surrender bank credentials to a third party |
| **Manual entry (YNAB)** | Full privacy and control | High friction, low adherence |
| **Bamboo Money** | Upload CSVs/statements | Privacy preserved, moderate effort, automatable |

Most users want the **convenience** of automatic categorization without the **privacy cost** of bank linking.

## Target Users

| Persona | Description | Pain Point |
|---|---|---|
| 🔒 **Privacy-Conscious Professional** | 30-45, tech-savvy, doesn't trust Plaid | Wants budgeting without linking bank accounts |
| 🌍 **Multi-Bank User** | Has accounts at institutions Plaid doesn't support | Can't use Monarch/YNAB effectively |
| 🎓 **Budget-Conscious Student** | 18-25, limited income, needs free tools | YNAB costs $99/yr, EveryDollar is opinionated |
| 👫 **Couple Managing Finances** | Partners who want shared visibility | Current free options are buggy |
| 📊 **Financial Optimizer** | High earner wanting full picture + AI insights | Wants comprehensive dashboard without subscription |

## Strategic Positioning

### Identity

Bamboo Money sits in a unique space:

```
Privacy ──────────────────────── Convenience
  │                                    │
  YNAB          Bamboo Money     Monarch Money
  (manual)      (file upload)    (Plaid-linked)
```

### Differentiators

1. **No bank linking required** — privacy-first, upload-based
2. **Self-hosted** — your data stays on your machine
3. **AI-ready architecture** — smart features without SaaS dependency
4. **Open-source potential** — community-driven, transparent
5. **PDF statement parsing** — a capability no major competitor offers

### Lessons from Competitors

| Lesson | Source | Application |
|---|---|---|
| Simple > complex | Monarch's clean UI | One obvious way to do everything |
| Polish matters | Copilot's Apple Design Award | Animations, spacing, consistency |
| Daily habit = retention | YNAB's engagement model | Dashboard must make you *want* to open it |
| Auto-categorization is table stakes | All competitors | Ship it early, not as premium |
| Privacy messaging builds trust | Market gap | Lead with "your data, your server" |

## Value Proposition Canvas

### Customer Jobs
- Know where money goes each month
- Stay within budget limits
- Track progress toward financial goals
- Understand spending patterns over time
- Detect forgotten subscriptions

### Pains
- Don't trust bank-linking services (Plaid)
- Too lazy for full manual entry (YNAB)
- Free tools are ad-supported or limited
- Switching between bank websites is tedious
- Subscription fatigue ($99/yr for YNAB, $99/yr for Monarch)

### Gains
- Privacy preserved (no third-party data access)
- Smart categorization (rules + future AI)
- Beautiful dashboard for daily check-ins
- Free to self-host
- CSV import works with any bank
