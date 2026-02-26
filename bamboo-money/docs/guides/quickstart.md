# Quick Start

Get Bamboo Money running in under 2 minutes.

## Prerequisites

- **Python 3.12+**
- **[UV](https://docs.astral.sh/uv/)** package manager
- **Git**

## Setup

```bash
# Clone the repository
git clone https://github.com/jose-pinedadelgado/bamboo-money-combined.git
cd bamboo-money-combined

# Install dependencies
uv sync

# Create database and apply migrations
uv run python manage.py migrate

# Load demo data (306 transactions, 12 categories, 3 accounts)
uv run python manage.py seed_demo_data

# Detect recurring transactions from demo data
uv run python manage.py detect_recurrings

# Calculate budget rollovers
uv run python manage.py calculate_rollovers

# Start the server
uv run python manage.py runserver 8400
```

## First Login

Open [http://localhost:8400](http://localhost:8400) and log in:

| Field | Value |
|---|---|
| Username | `demo` |
| Password | `demo` |

You'll land on the **Dashboard** with 6 months of realistic demo data already loaded.

## What to Explore

1. **Dashboard** — See your spending summary, budget progress, alerts, and cash flow forecast
2. **Transactions** — Browse, filter, search, and export your transaction history
3. **Budgets** — View per-category progress bars and remaining amounts
4. **Rules** — Set up auto-categorization rules for merchants
5. **Recurring** — See detected subscriptions and upcoming bills
6. **Goals** — Track progress toward savings goals
7. **Net Worth** — View assets, liabilities, and historical net worth chart
8. **Dark Mode** — Toggle the :material-weather-night: button in the sidebar

## Create Your Own Account

Click **Register** on the login page. New accounts automatically get 12 default budget categories (Housing, Food & Dining, Transportation, etc.).

## Next Steps

- [Installation Guide](installation.md) — detailed setup for development and production
- [Demo Walkthrough](demo-walkthrough.md) — guided tour of every feature
- [CSV Import](../features/csv-import-export.md) — import your real bank data
