# Bamboo Money

Your finances, your server, your data. Smart budgeting without giving your bank credentials to anyone.

---

Bamboo Money is a **privacy-first personal finance app** built with Django. It helps you understand and control your spending through CSV import, intelligent categorization, and actionable budget insights — all without requiring bank account linking.

!!! tip "Quick Start"
    ```bash
    git clone https://github.com/jose-pinedadelgado/bamboo-money-combined.git
    cd bamboo-money-combined
    uv sync && uv run python manage.py migrate
    uv run python manage.py seed_demo_data
    uv run python manage.py runserver 8400
    ```
    Open [localhost:8400](http://localhost:8400) — login: **demo / demo**

## Get Started

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } **Quick Start**

    ---

    Get Bamboo Money running in under 2 minutes with demo data.

    [:octicons-arrow-right-24: Quick Start](guides/quickstart.md)

-   :material-download:{ .lg .middle } **Installation**

    ---

    Detailed setup for development and production environments.

    [:octicons-arrow-right-24: Installation](guides/installation.md)

-   :material-compass:{ .lg .middle } **Demo Walkthrough**

    ---

    Guided tour of every feature with the demo account.

    [:octicons-arrow-right-24: Walkthrough](guides/demo-walkthrough.md)

-   :material-view-dashboard:{ .lg .middle } **Features**

    ---

    Overview of all 8 core features and foundational capabilities.

    [:octicons-arrow-right-24: Features](features/index.md)

</div>

## Core Features

<div class="grid cards" markdown>

-   :material-upload:{ .lg .middle } **CSV Import**

    ---

    Auto-detect Chase, Amex, Wells Fargo, BoA. 3-step upload → preview → confirm flow.

    [:octicons-arrow-right-24: Learn more](features/csv-import-export.md)

-   :material-tag-multiple:{ .lg .middle } **Auto-Categorization**

    ---

    Keyword rules with priority, bulk apply, and one-click rule creation from transactions.

    [:octicons-arrow-right-24: Learn more](features/categorization-rules.md)

-   :material-chart-line:{ .lg .middle } **Cash Flow Forecast**

    ---

    Daily spending pace visualization with end-of-month projections.

    [:octicons-arrow-right-24: Learn more](features/cash-flow-forecast.md)

-   :material-bell-alert:{ .lg .middle } **Spending Alerts**

    ---

    Automatic warnings at 80%, 100%, and unusual spending (>1.5× average).

    [:octicons-arrow-right-24: Learn more](features/spending-alerts.md)

-   :material-repeat:{ .lg .middle } **Recurring Detection**

    ---

    Auto-detect subscriptions and recurring bills from transaction patterns.

    [:octicons-arrow-right-24: Learn more](features/recurring-detection.md)

-   :material-arrow-right-bold:{ .lg .middle } **Budget Rollover**

    ---

    Carry unused budget to next month with optional caps. Inspired by Copilot Money.

    [:octicons-arrow-right-24: Learn more](features/budget-rollover.md)

</div>

## Why Bamboo Money?

| Pain Point | Bamboo Money's Answer |
|---|---|
| "I don't trust Plaid with my bank credentials" | **No bank linking required** — upload CSVs or enter manually |
| "Mint is dead, YNAB costs $99/year" | **Self-hosted and free** — run it on your own machine |
| "I have accounts at banks Plaid doesn't support" | **Universal CSV import** — works with any bank's export |
| "I want AI features without a subscription" | **AI-ready architecture** — smart features without SaaS lock-in |

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 6.0, Python 3.12 |
| Frontend | Bootstrap 5, HTMX, Chart.js |
| Database | SQLite (dev) / PostgreSQL (prod) |
| Package Manager | UV |

## Architecture

<div class="grid cards" markdown>

-   :material-sitemap:{ .lg .middle } **System Overview**

    ---

    Architecture diagram, request flow, and design decisions.

    [:octicons-arrow-right-24: Overview](architecture/overview.md)

-   :material-database:{ .lg .middle } **Data Model**

    ---

    11 models with ERD, relationships, and migration history.

    [:octicons-arrow-right-24: Data Model](architecture/data-model.md)

-   :material-shield-lock:{ .lg .middle } **Security**

    ---

    Current posture, known findings, and remediation plan.

    [:octicons-arrow-right-24: Security](architecture/security.md)

-   :material-map:{ .lg .middle } **Roadmap**

    ---

    5 milestones from prototype to platform. Competitive analysis included.

    [:octicons-arrow-right-24: Roadmap](product/roadmap.md)

</div>

---

*Built by [Jose Pineda](https://github.com/jose-pinedadelgado) — Assistant Professor, California State University, Long Beach*
