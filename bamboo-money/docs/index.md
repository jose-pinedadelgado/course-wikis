# 🎋 Bamboo Money

> **Your finances, your server, your data. Smart budgeting without giving your bank credentials to anyone.**

Bamboo Money is a **privacy-first personal finance app** built with Django. It helps you understand and control your spending through CSV import, intelligent categorization, and actionable budget insights — all without requiring bank account linking.

## Why Bamboo Money?

| Pain Point | Bamboo Money's Answer |
|---|---|
| "I don't trust Plaid with my bank credentials" | **No bank linking required** — upload CSVs or enter manually |
| "Mint is dead, YNAB costs $99/year" | **Self-hosted and free** — run it on your own machine |
| "I have accounts at banks that Plaid doesn't support" | **Universal CSV import** — works with any bank's export |
| "I want AI features but not a monthly subscription" | **AI-ready architecture** — smart features without SaaS lock-in |

## Feature Highlights

- :material-upload: **CSV Import** with auto-detection for Chase, Amex, Wells Fargo, Bank of America
- :material-tag-multiple: **Auto-Categorization Rules** with keyword matching and bulk apply
- :material-chart-line: **Cash Flow Forecast** with spending pace visualization
- :material-bell-alert: **Spending Alerts** at 80%, 100%, and unusual spending detection
- :material-repeat: **Recurring Transaction Detection** for subscriptions and bills
- :material-arrow-right-bold: **Budget Rollover** — unused budget carries to next month
- :material-target: **Savings Goals** with contribution tracking and projections
- :material-scale-balance: **Net Worth Tracking** with historical snapshots
- :material-weather-night: **Dark Mode** with localStorage persistence

## Quick Start

```bash
git clone https://github.com/jose-pinedadelgado/bamboo-money-combined.git
cd bamboo-money-combined
uv sync
uv run python manage.py migrate
uv run python manage.py seed_demo_data
uv run python manage.py runserver 8400
```

Open [http://localhost:8400](http://localhost:8400) — login with **demo / demo**.

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 6.0, Python 3.12 |
| Frontend | Bootstrap 5, HTMX, Chart.js |
| Database | SQLite (dev) / PostgreSQL (prod) |
| Package Manager | UV |

## Project Status

Bamboo Money is a **working prototype** with 8 implemented features. It was built as part of a research project exploring AI-assisted software development and privacy-first personal finance.

---

*Built by [Jose Pineda](https://github.com/jose-pinedadelgado) — Assistant Professor, California State University, Long Beach*
