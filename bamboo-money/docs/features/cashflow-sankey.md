# 📊 Cash Flow (Sankey Diagram)

An interactive Sankey diagram visualizing how money flows from income sources through spending categories, savings, and remaining balance.

## Overview

The Cash Flow page (`/cashflow/`) renders a D3.js-powered Sankey diagram that shows:

```
Income Sources → 💰 Total Income → Spending Categories
                                 → Savings Goals
                                 → 💵 Remaining (or 🔴 Deficit)
```

## Data Flow

### Nodes

| Node Type | Source | Color |
|---|---|---|
| 📥 Income sources | Transaction merchant/description (grouped) | Green `#66BB6A` |
| 💰 Total Income | Aggregated hub | Green `#4CAF50` |
| 📦 Expense categories | BudgetCategory name + icon | Category color |
| 🎯 Savings goals | SavingsGoal name + icon | Blue `#42A5F5` |
| 💵 Remaining | Income - Expenses - Savings | Light green `#81C784` |
| 🔴 Deficit | When expenses exceed income | Red `#EF5350` |

### Links

Links represent money flow with width proportional to the amount. Hover to see exact values.

## Features

### Month Navigation
- **Previous/Next** buttons to browse any month
- Summary cards update with each month showing: Income, Expenses, Savings, Remaining

### Summary Cards
Four cards above the diagram provide at-a-glance totals:
- 💚 **Income** — total income for the month
- 🧡 **Expenses** — total spending
- 💙 **Savings** — goal contributions
- **Remaining** — green if positive, red if deficit

### Interactivity
- **Hover nodes** — tooltip with name and total amount
- **Hover links** — tooltip with source → target and exact flow amount
- **Responsive** — redraws on window resize

## Technical Details

| Component | Technology |
|---|---|
| Chart library | D3.js v7 + d3-sankey v0.12.3 |
| Data endpoint | `/cashflow/data/?month=M&year=Y` (JSON) |
| Rendering | SVG with CSS tooltips |
| Template | `templates/interface/cashflow.html` |

### API Response Format

```json
{
    "nodes": [
        {"name": "💰 Total Income", "color": "#4CAF50"},
        {"name": "🍕 Food & Dining", "color": "#FF7043"}
    ],
    "links": [
        {"source": 0, "target": 1, "value": 450.00}
    ],
    "summary": {
        "month": 2,
        "year": 2026,
        "month_name": "February",
        "total_income": 8605.73,
        "total_expenses": 3220.52,
        "total_savings": 1479.15,
        "remainder": 3906.06
    }
}
```

## Chatbot Integration

Ask the Bamboo Assistant about cash flow:
- "Show me my cash flow"
- "Sankey diagram"
- "How does my money flow?"

The chatbot auto-injects a **"📊 View Cash Flow Diagram →"** button linking to the full interactive page.
