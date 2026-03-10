# Net Worth Tracker

The net worth tracker monitors all your accounts over time, providing a comprehensive view of your financial health.

## Account Groups

Every account belongs to one of 5 groups:

| Group | Examples | Color |
|-------|----------|-------|
| **Liquid** | Checking, savings, money market | Blue |
| **Investment** | Brokerage, 529, HSA | Green |
| **Retirement** | 401k, IRA, Roth IRA | Purple |
| **Liability** | Credit cards, loans, mortgage | Red |
| **Other Asset** | Property, vehicles | Gray |

## Data Model

```mermaid
erDiagram
    NetWorthEntry {
        string name
        decimal balance
        string account_group
        date date
        FK user
    }
    NetWorthEntryHistory {
        decimal amount
        date date
        FK entry
    }
    NetWorthEntry ||--o{ NetWorthEntryHistory : "tracked over time"
```

- **`NetWorthEntry`** — Each account (e.g., "Chase Checking", "Vanguard 401k") with current balance and group
- **`NetWorthEntryHistory`** — Per-account snapshots over time for historical tracking

## 8 Key Metrics

1. **Liquid Months** — Cash reserves ÷ monthly expenses. How many months could you survive without income?
2. **Debt-to-Asset Ratio** — Total liabilities ÷ total assets. Lower is better (< 0.5 is healthy).
3. **FI Ratio** — Net worth ÷ annual expenses. Target: 25x (the "4% rule" for financial independence).
4. **Savings Rate** — Month-over-month net worth growth as a percentage.
5. **Invested Capital** — (Investments + Retirement) as a percentage of total net worth.
6. **Liquid Net Worth** — Cash and liquid assets minus all debts.
7. **Debt Payoff Timeline** — Months to pay off all debt at current paydown rate.
8. **Retirement %** — Retirement accounts as a percentage of total net worth.

## Visualizations

### Trend Chart
Three solid lines tracking **Assets** (blue), **Net Worth** (green), and **Liabilities** (red) over time. Built with Chart.js.

### Asset Allocation Donut
A donut chart showing the breakdown by account group. Uses `chartjs-plugin-datalabels` for permanent labels showing both dollar amount ($XXK) and percentage (XX.X%).

### Stacked Area Chart
Shows net worth composition over time — how the proportion of liquid/investment/retirement/liability changes month to month.

### Collapsible Group Breakdown
Expandable cards for each of the 5 groups showing individual account balances within that group.

## Demo Data

The `demo2` account contains Jose's real financial snapshot:

- **34 accounts** across all 5 groups
- **204 history records** (6 time periods × 34 accounts)
- **$470,476.11 net worth** (verified against source spreadsheet)
- Breakdown: $68K liquid, $176K investments, $230K retirement, $16K liabilities, $12K other
