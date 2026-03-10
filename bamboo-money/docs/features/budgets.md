# Budgets & Rollover

Bamboo Money uses **zero-based budgeting** — every dollar of income gets assigned to a category.

## Creating Budgets

Each budget has:

- **Category** — e.g., Groceries, Rent, Entertainment
- **Monthly Amount** — The planned spending limit
- **Month/Year** — Which month this budget applies to

## Automatic Rollover

When a month ends, any **unspent** budget automatically rolls forward:

```
March Groceries: $500 budgeted - $420 spent = $80 remaining
April Groceries: $500 budgeted + $80 rollover = $580 available
```

This means you don't lose money you didn't spend — it accumulates until you use it. Overspending in a category creates negative rollover that reduces next month's available amount.

## Budget vs. Actual View

The budget page shows a table with:

| Category | Budgeted | Rollover | Available | Spent | Remaining |
|----------|----------|----------|-----------|-------|-----------|
| Groceries | $500 | +$80 | $580 | $420 | $160 |
| Rent | $2,000 | $0 | $2,000 | $2,000 | $0 |
| Entertainment | $200 | -$50 | $150 | $100 | $50 |

## CRUD Operations

- Create, edit, and delete budgets for any month
- Copy last month's budget structure to the current month
- Bulk category management
