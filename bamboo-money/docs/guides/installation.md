# Installation Guide

## Development Setup

### 1. Clone and Install

```bash
git clone https://github.com/jose-pinedadelgado/bamboo-money-combined.git
cd bamboo-money-combined
uv sync
```

This installs Django 6.0, django-htmx, whitenoise, and all dependencies into a `.venv/` virtual environment.

### 2. Database Setup

```bash
uv run python manage.py migrate
```

This creates a SQLite database (`db.sqlite3`) with all 6 migration files:

| Migration | What It Creates |
|---|---|
| `0001_initial` | Core models: BudgetCategory, Account, Transaction, SavingsGoal, GoalContribution, NetWorthEntry, NetWorthSnapshot |
| `0002_importbatch_transaction_import_batch` | ImportBatch model + FK on Transaction |
| `0003_budgetalert` | BudgetAlert model |
| `0004_categorizationrule` | CategorizationRule model |
| `0005_recurringtransaction` | RecurringTransaction model |
| `0006_budgetcategory_rollover_cap_and_more` | Rollover fields + MonthlyBudgetSnapshot |

### 3. Load Demo Data (Optional)

```bash
uv run python manage.py seed_demo_data
uv run python manage.py detect_recurrings
uv run python manage.py calculate_rollovers
```

The seed command creates:

- 1 demo user (`demo/demo`)
- 12 budget categories with monthly limits
- 3 accounts (Chase Checking, Amex Gold, Cash)
- 306 transactions over 6 months
- 3 savings goals with contributions
- 9 net worth entries and 6 monthly snapshots

### 4. Run the Server

```bash
uv run python manage.py runserver 8400
```

!!! tip "Port Assignment"
    Bamboo Money uses port **8400** by convention in this project. You can use any available port.

### 5. Verify

```bash
uv run python smoke_test.py
```

This runs 12 automated checks covering all pages (dashboard, transactions, budgets, goals, net worth, rules, recurring, CSV upload/export).

## Project Structure

```
bamboo-money-combined/
├── bamboo_site/              # Django project settings
│   ├── settings.py           # Configuration
│   ├── urls.py               # Root URL routing
│   └── wsgi.py
├── interface/                # Main Django app
│   ├── models.py             # 11 models
│   ├── views.py              # 35+ views
│   ├── forms.py              # 7 form classes
│   ├── urls.py               # URL patterns
│   ├── csv_parser.py         # Bank CSV detection & parsing
│   ├── admin.py              # Django admin registration
│   └── management/commands/
│       ├── seed_demo_data.py
│       ├── detect_recurrings.py
│       └── calculate_rollovers.py
├── templates/interface/      # Django templates
│   ├── base.html             # Sidebar layout + dark mode
│   ├── dashboard.html        # Main dashboard
│   ├── transactions.html     # Transaction list
│   └── ...                   # 18 total templates
├── docs/                     # Project documentation
│   ├── IMPROVEMENT_PLAN.md
│   ├── SECURITY_PRODUCTION_READINESS.md
│   └── TEST_FIRST_BUGFIX_POLICY.md
├── test_files/               # Sample CSV files for testing
├── smoke_test.py             # Automated smoke tests
├── pyproject.toml            # UV/pip dependencies
└── uv.lock                   # Locked dependency versions
```

## Environment Variables

For development, the defaults in `settings.py` work out of the box. For production:

| Variable | Purpose | Default |
|---|---|---|
| `SECRET_KEY` | Django secret key | Hardcoded (⚠️ change for production) |
| `DEBUG` | Debug mode | `True` (⚠️ set `False` for production) |
| `ALLOWED_HOSTS` | Permitted hostnames | `["*"]` (⚠️ restrict for production) |

!!! warning "Production Deployment"
    The current settings are configured for **development only**. See the [Security](../architecture/security.md) page for production hardening requirements.

## Troubleshooting

??? question "Migration errors after pulling new code"
    ```bash
    uv run python manage.py migrate
    ```
    If migrations conflict, reset with:
    ```bash
    rm db.sqlite3
    uv run python manage.py migrate
    uv run python manage.py seed_demo_data
    ```

??? question "Port 8400 already in use"
    Use a different port: `uv run python manage.py runserver 8500`

??? question "Static files not loading"
    Bamboo Money uses WhiteNoise for static file serving. Ensure `whitenoise` is in your dependencies:
    ```bash
    uv sync
    ```
