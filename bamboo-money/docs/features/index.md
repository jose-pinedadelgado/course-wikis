# Features Overview

Bamboo Money ships with **8 core features** plus foundational capabilities for transaction management, budgeting, savings, and net worth tracking.

## Feature Summary

| # | Feature | Status | Description |
|---|---------|--------|-------------|
| 1 | [CSV Export](csv-import-export.md#csv-export) | ✅ Complete | Download filtered transactions as CSV |
| 2 | [CSV Import](csv-import-export.md#csv-import) | ✅ Complete | Upload bank CSVs with auto-detection |
| 3 | [Dark Mode](dark-mode.md) | ✅ Complete | Light/dark theme toggle with persistence |
| 4 | [Auto-Categorization Rules](categorization-rules.md) | ✅ Complete | Keyword-based automatic categorization |
| 5 | [Spending Alerts](spending-alerts.md) | ✅ Complete | Budget warnings and unusual spending detection |
| 6 | [Recurring Detection](recurring-detection.md) | ✅ Complete | Auto-detect subscriptions and bills |
| 7 | [Budget Rollover](budget-rollover.md) | ✅ Complete | Carry unused budget to next month |
| 8 | [Cash Flow Forecast](cash-flow-forecast.md) | ✅ Complete | Spending pace projections and upcoming bills |

## Core Capabilities

These foundational features support the 8 core features above:

| Capability | Description |
|---|---|
| [Dashboard](dashboard.md) | Financial command center with charts, alerts, and projections |
| [Transactions](transactions.md) | Full CRUD with search, filter, pagination, and inline editing |
| [Budget Management](budgets.md) | Per-category spending limits with progress tracking |
| [Savings Goals](savings-goals.md) | Target-based saving with contributions and deadlines |
| [Net Worth Tracking](net-worth.md) | Manual asset/liability tracking with historical charts |

## Design Philosophy

Bamboo Money follows three principles informed by competitive analysis of Monarch Money and Copilot Money:

!!! abstract "Privacy First"
    No bank account linking required. All data stays on your machine. Import via CSV upload or manual entry.

!!! abstract "Simple Over Complex"
    Inspired by Monarch's lesson: every feature has one obvious way to use it. Clean UI, no clutter.

!!! abstract "Intelligence Without SaaS"
    Auto-categorization rules, recurring detection, and spending alerts work locally — no external API calls needed.
