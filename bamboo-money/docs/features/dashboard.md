# Dashboard

The main dashboard provides a real-time financial overview with interactive charts and budget tracking.

## Summary Cards

Four cards at the top show key metrics for the current month:

- **Total Income** — Sum of all income transactions
- **Total Expenses** — Sum of all expense transactions
- **Net Savings** — Income minus expenses
- **Budget Remaining** — Unspent budget across all categories

## Spending Donut Chart

A Chart.js donut chart breaks down spending by category. Each segment is clickable and shows the dollar amount and percentage. The center displays total spending.

## Spending Trends

A line chart showing daily or weekly spending over the selected period. Helps identify spending spikes and patterns.

## Budget Progress Bars

Each budget category shows:

- Category name and icon
- Amount spent vs. budgeted
- Visual progress bar (green → yellow → red as you approach/exceed the limit)
- Rollover amount from previous month (if applicable)

## Recent Transactions

A quick-view list of the 10 most recent transactions with merchant, category, date, and amount. Click any row to edit.

## Collapsible Sidebar

The sidebar can be collapsed for more screen space. State persists via `localStorage`. An expand tab appears on the left edge when collapsed. Smooth CSS animation on toggle.

## HTMX Integration

Dashboard updates use HTMX for partial page refreshes — changing the month filter or dismissing alerts doesn't reload the entire page. CSRF tokens are automatically injected via an `htmx:configRequest` event listener.
