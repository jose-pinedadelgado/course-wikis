# Roadmap

Bamboo Money follows a milestone-based roadmap. Each milestone is a shippable product that delivers real value.

## Current State: Prototype Complete ✅

The working prototype includes:

- User auth (register, login, logout)
- Full transaction management with search, filter, pagination
- CSV import (Chase, Amex, Wells Fargo, BoA auto-detection)
- CSV export with filter support
- Budget categories with progress tracking
- Budget rollover with caps
- Auto-categorization rules (keyword matching)
- Spending alerts (80%, 100%, unusual)
- Recurring transaction detection
- Cash flow forecast with pace visualization
- Savings goals with contribution tracking
- Net worth tracking with historical snapshots
- Dark mode with persistence

## Milestone Roadmap

### 🟢 M1: Production-Ready MVP

**Goal:** Harden the prototype for real daily use.

| Task | Priority | Status |
|---|---|---|
| Fix security findings (see [Security](../architecture/security.md)) | P0 | 🔴 Not started |
| Split settings (dev/prod) | P0 | 🔴 Not started |
| Fix cross-user category assignment bug | P0 | 🔴 Not started |
| Convert logout to POST-only | P0 | 🔴 Not started |
| Add authorization regression tests | P0 | 🔴 Not started |
| Re-enable password validators | P0 | 🔴 Not started |
| Login rate limiting | P1 | 🔴 Not started |
| Atomic goal contribution updates | P1 | 🔴 Not started |
| Docker deployment | P1 | 🔴 Not started |
| Auto-apply categorization rules on CSV import | P1 | 🔴 Not started |

### 🟡 M2: Smart Categorization + Analytics

**Goal:** The app feels intelligent and provides "aha" moments.

| Task | Priority |
|---|---|
| AI auto-categorization (OpenAI fallback when no rule matches) | P0 |
| Learn from user corrections (auto-create rules) | P0 |
| Budget vs. actual bar chart | P0 |
| Per-category spending trends | P0 |
| Suggested budget limits based on actual spending | P1 |
| Bulk category reassignment | P1 |

### 🔵 M3: AI Assistant + PDF Parsing

**Goal:** The app proactively tells you things you didn't know.

| Task | Priority |
|---|---|
| PDF statement parsing (AMEX, Chase, Wells Fargo, BoA) | P0 |
| Monthly AI spending insights (auto-generated) | P0 |
| Subscription list with annual cost calculator | P0 |
| Natural language queries ("How much on dining in Jan?") | P1 |
| Weekly spending recap | P1 |

### 🟣 M4: Household + Polish

**Goal:** Multi-user support and production polish.

| Task | Priority |
|---|---|
| Household model + invite flow | P0 |
| Shared budget view | P0 |
| Transaction attribution (who spent what) | P1 |
| Onboarding wizard | P1 |
| Responsive design audit (mobile/tablet) | P1 |
| Accessibility audit (WCAG 2.1 AA) | P2 |

### ⚪ M5: Platform

**Goal:** Bamboo Money as a platform, not just an app.

| Task | Priority |
|---|---|
| Open-source release | P0 |
| Plugin/extension system | P1 |
| Plaid integration (optional) | P1 |
| Mobile PWA | P1 |
| Multi-currency support | P2 |
| Financial coaching via LLM | P2 |

## Timeline Visualization

```
Prototype     M1: Harden    M2: Smart    M3: AI       M4: Polish    M5: Platform
  (done)      (2-3 weeks)  (2-3 weeks)  (3-4 weeks)  (2-3 weeks)  (ongoing)
────●─────────────●────────────●────────────●────────────●────────────●────→
   Now        Security +    AI cats +    PDF parsing  Household    Open source
              tests         analytics    NLQ          mobile       plugins
```

## Design Principles (from Competitive Research)

1. **Simple > complex** — Monarch's lesson. One obvious way to use each feature.
2. **Polish matters** — Copilot's lesson. Micro-interactions and consistent spacing.
3. **Auto-categorization is table stakes** — Both agree. Not a premium feature.
4. **Recurring detection builds trust** — "This app understands my finances."
5. **The dashboard drives daily habit** — Make people *want* to open the app.
6. **Privacy messaging is important** — Lead with "your data, your server."
