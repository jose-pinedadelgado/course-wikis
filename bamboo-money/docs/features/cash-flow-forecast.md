# Cash Flow Forecast

The cash flow forecast projects your end-of-month spending based on your current pace, visualized as a line chart on the dashboard.

## How It Works

### Daily Average Projection

```python
days_elapsed = max(now.day, 1)
daily_avg_spend = total_spent / days_elapsed
projected_total = daily_avg_spend * days_in_month
pace_status = "under" if projected_total <= total_budget else "over"
pace_diff = abs(projected_total - total_budget)
```

**Example:** On day 15 of a 30-day month, you've spent $1,500. Daily average = $100. Projected total = $3,000. If your total budget is $3,200, you're projected to be $200 **under** budget.

### Spending Pace Chart

A Chart.js line chart with two series:

| Line | Style | Meaning |
|---|---|---|
| **Actual** | Solid | Cumulative daily spending (what you actually spent each day) |
| **Budget Pace** | Dashed | Linear budget pace (total budget ÷ days in month × day number) |

```python
# Build daily cumulative spending
daily_cumulative = []
running_total = 0
for d in range(1, now.day + 1):
    day_spent = month_txns.filter(is_income=False, date__day=d)
                           .aggregate(t=Sum("amount"))["t"] or 0
    running_total += float(day_spent)
    daily_cumulative.append(round(running_total, 2))

# Build budget pace line
daily_budget = total_budget / days_in_month
budget_pace = [round(daily_budget * d, 2) for d in range(1, now.day + 1)]
```

When the actual line is **below** the budget pace line, you're spending slower than average. When it crosses **above**, you're on pace to exceed your budget.

### Pace Badge

A color-coded badge on the dashboard:

- :material-check-circle:{ style="color: green" } **Green badge** — "Projected $X under budget"
- :material-alert:{ style="color: red" } **Red badge** — "Projected $X over budget"

With subtext: *"At current pace: $X by month end"*

## Upcoming Bills

Below the chart, upcoming recurring charges are listed:

- Pulls from [RecurringTransaction](recurring-detection.md) records
- Shows charges expected **later this month** (date > today)
- Up to 5 items with merchant, amount, and expected date
- Running total of upcoming charges

This helps answer: *"Even if I stop spending, how much more is already committed?"*

## Use Case

> *It's the 15th and Jose has spent $1,800. The dashboard shows: "Projected $3,600 by month end — $200 over budget." The pace chart shows his spending line crossing above the budget line around day 20. Below, it lists $450 in upcoming bills (rent, Netflix, gym).*

## Competitive Inspiration

| App | Approach |
|---|---|
| **Copilot Money** | "Follow the line" — daily spending vs. budget pace. Badge: "You're $125 under budget at this pace." |
| **Monarch Money** | Cash flow report with income vs. expenses. Projection based on recurring transactions. |
| **Bamboo Money** | Combines both: pace chart (Copilot-style) + upcoming bills (Monarch-style), all on the dashboard. |

## Limitations

!!! note "Linear projection"
    The current forecast uses a simple **daily average** projection. It doesn't account for spending patterns (e.g., rent hits on the 1st, groceries cluster on weekends). A future enhancement could use weighted projections based on historical day-of-month patterns.

!!! note "Recurring not subtracted from pace"
    Upcoming recurring bills are **shown separately** but not factored into the projected total. A smarter model would add known upcoming bills to the projection.
