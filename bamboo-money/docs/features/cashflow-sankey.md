# Sankey Cash Flow

The Sankey diagram visualizes how money flows from income sources through spending categories, providing an intuitive view of where your money goes.

## How It Works

```
Income Sources → Spending Categories → Accounts
   Salary ──────→ Groceries ─────────→ Chase Checking
   Freelance ───→ Rent ──────────────→ Wells Fargo
                → Entertainment ─────→ Credit Card
                → Savings ───────────→ Vanguard
```

## Technology

- **D3.js** — Core rendering engine
- **d3-sankey** — Sankey layout algorithm
- Node widths proportional to dollar amounts
- Hover tooltips show exact amounts and percentages

## Navigation

- **Month selector** — Navigate forward/backward through months
- **Summary cards** — Total income, total expenses, net savings for the selected month
- Responsive layout adapts to screen width

## Colors

- Income nodes: green tones
- Expense category nodes: colored by category
- Account nodes: blue tones
- Links inherit source node color with reduced opacity

## Implementation

The Sankey page lives at `/cashflow/` and queries the transaction model for the selected month, grouping by category and account to build the node/link data structure for D3.
