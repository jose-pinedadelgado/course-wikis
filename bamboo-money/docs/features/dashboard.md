# Dashboard

The dashboard is Bamboo Money's home screen — a financial command center designed for daily check-ins.

## Layout

The dashboard displays seven information sections:

### Summary Cards

Four metric cards at the top:

- **Total Spent** — sum of all expenses this month
- **Total Income** — sum of all income this month
- **Total Budget** — sum of all category monthly limits
- **Budget Progress** — overall spending as a percentage of total budget

### Budget Progress by Category

Each non-income category shows a horizontal progress bar:

| Color | Meaning |
|---|---|
| :material-check-circle:{ style="color: green" } Green | Under 80% spent |
| :material-alert:{ style="color: orange" } Yellow | 80–99% spent |
| :material-close-circle:{ style="color: red" } Red | Over budget (100%+) |

If [budget rollover](budget-rollover.md) is enabled for a category, the progress bar reflects the **effective budget** (base limit + rollover amount).

### Spending Alerts

Alert banners from the [spending alerts](spending-alerts.md) system. Shows up to 5 unread alerts with dismiss controls.

### Cash Flow Forecast

The **Spending Pace** section includes:

- A **pace badge** — "Projected $X under/over budget by month end"
- A **Chart.js line chart** with:
    - Solid line = actual cumulative daily spending
    - Dashed line = linear budget pace (even spending over the month)

See [Cash Flow Forecast](cash-flow-forecast.md) for details.

### Upcoming Bills

Lists up to 5 [recurring transactions](recurring-detection.md) expected later this month, with a total.

### Category Breakdown (Pie Chart)

A Chart.js doughnut chart showing spending distribution by category. Uses each category's custom color.

### 6-Month Trend (Bar Chart)

Side-by-side bars showing monthly spending and income for the last 6 months.

### Recent Transactions

The 10 most recent transactions with date, description, and amount.

## Technical Details

The dashboard view (`dashboard()` in `views.py`) performs several calculations on each load:

```python
# Key computations:
# 1. Monthly totals (income + spending)
# 2. Per-category spending via spent_this_month()
# 3. 6-month trend via monthly aggregates
# 4. Alert generation via _check_budget_alerts()
# 5. Cash flow projection via daily average extrapolation
# 6. Upcoming recurring from RecurringTransaction model
```

!!! note "Performance"
    The dashboard makes multiple database queries (one per category, 6 for monthly trends). For users with many categories, consider caching monthly aggregates.
