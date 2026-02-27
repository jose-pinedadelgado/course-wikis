# Features

Bamboo Money ships with **8 core features** plus foundational capabilities for transaction management, budgeting, savings, and net worth tracking.

---

## Core Features

<div class="grid cards" markdown>

-   :material-download:{ .lg .middle } **CSV Export**

    ---

    Download filtered transactions as a CSV file. Respects all active filters.

    [:octicons-arrow-right-24: CSV Import & Export](csv-import-export.md)

-   :material-upload:{ .lg .middle } **CSV Import**

    ---

    Upload bank CSVs with auto-detection for Chase, Amex, Wells Fargo, and Bank of America.

    [:octicons-arrow-right-24: CSV Import & Export](csv-import-export.md)

-   :material-weather-night:{ .lg .middle } **Dark Mode**

    ---

    Light/dark theme toggle with localStorage persistence and Chart.js adaptation.

    [:octicons-arrow-right-24: Dark Mode](dark-mode.md)

-   :material-tag-multiple:{ .lg .middle } **Auto-Categorization Rules**

    ---

    Keyword-based rules with contains/starts-with/exact match and priority ordering.

    [:octicons-arrow-right-24: Categorization Rules](categorization-rules.md)

-   :material-bell-alert:{ .lg .middle } **Spending Alerts**

    ---

    Budget warnings at 80%, 100%, and unusual spending detection (>1.5× average).

    [:octicons-arrow-right-24: Spending Alerts](spending-alerts.md)

-   :material-repeat:{ .lg .middle } **Recurring Detection**

    ---

    Auto-detect subscriptions and recurring bills from transaction history patterns.

    [:octicons-arrow-right-24: Recurring Detection](recurring-detection.md)

-   :material-arrow-right-bold:{ .lg .middle } **Budget Rollover**

    ---

    Carry unused budget forward with optional caps. Per-category opt-in.

    [:octicons-arrow-right-24: Budget Rollover](budget-rollover.md)

-   :material-chart-line:{ .lg .middle } **Cash Flow Forecast**

    ---

    Spending pace projections with actual vs. budget line chart and upcoming bills.

    [:octicons-arrow-right-24: Cash Flow Forecast](cash-flow-forecast.md)

</div>

## AI & Analytics

<div class="grid cards" markdown>

-   :material-robot:{ .lg .middle } **AI Chatbot (Bamboo Assistant)**

    ---

    GPT-4o-mini powered assistant with inline visualizations, suggested questions, and conversation memory.

    [:octicons-arrow-right-24: Chatbot](chatbot.md)

-   :material-chart-sankey:{ .lg .middle } **Cash Flow Sankey Diagram**

    ---

    Interactive D3.js Sankey showing income → spending → savings flow with month navigation.

    [:octicons-arrow-right-24: Cash Flow Sankey](cashflow-sankey.md)

</div>

## Foundational Capabilities

<div class="grid cards" markdown>

-   :material-view-dashboard:{ .lg .middle } **Dashboard**

    ---

    Financial command center with charts, alerts, pace tracking, and projections.

    [:octicons-arrow-right-24: Dashboard](dashboard.md)

-   :material-swap-horizontal:{ .lg .middle } **Transactions**

    ---

    Full CRUD with search, filter, pagination, and HTMX inline editing.

    [:octicons-arrow-right-24: Transactions](transactions.md)

-   :material-chart-pie:{ .lg .middle } **Budget Management**

    ---

    Per-category spending limits with progress bars and color coding.

    [:octicons-arrow-right-24: Budgets](budgets.md)

-   :material-target:{ .lg .middle } **Savings Goals**

    ---

    Target-based saving with contributions, deadlines, and monthly projections.

    [:octicons-arrow-right-24: Savings Goals](savings-goals.md)

-   :material-scale-balance:{ .lg .middle } **Net Worth Tracker**

    ---

    Full financial dashboard with 8 key metrics (FI ratio, liquid NW, debt payoff), trend charts, allocation donut, stacked area, account groups, and formatted Excel export.

    [:octicons-arrow-right-24: Net Worth](net-worth.md)

</div>

## Design Philosophy

!!! abstract "Privacy First"
    No bank account linking required. All data stays on your machine. Import via CSV upload or manual entry.

!!! abstract "Simple Over Complex"
    Inspired by Monarch's lesson: every feature has one obvious way to use it. Clean UI, no clutter.

!!! abstract "Intelligence Without SaaS"
    Auto-categorization rules, recurring detection, and spending alerts work locally — no external API calls needed.
