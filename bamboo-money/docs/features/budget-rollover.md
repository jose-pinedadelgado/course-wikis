# Budget Rollover

Budget rollover carries unused budget from one month to the next. This is inspired by **Copilot Money's signature feature** — their implementation won Apple Design Awards recognition.

## How It Works

When rollover is enabled for a category:

```
Effective Budget = Monthly Limit + Rollover from Previous Month
```

**Example:**

| Month | Base Limit | Rollover In | Effective | Spent | Rollover Out |
|---|---|---|---|---|---|
| January | $200 | $0 | $200 | $120 | $80 |
| February | $200 | $80 | $280 | $150 | $130 |
| March | $200 | $130 | $330 | $400 | $0 |

In March, spending exceeded the effective budget, so no rollover carries forward.

## Rollover Cap

To prevent unlimited accumulation, each category has an optional **rollover cap**:

```python
rollover_out = max(0, min(remaining, rollover_cap or infinity))
```

- **Cap = 0** → Unlimited rollover (default)
- **Cap = $500** → Maximum $500 carries forward, even if more was unspent

## Enabling Rollover

1. Navigate to **Budgets** → click **Edit** on a category
2. Check **Rollover enabled**
3. Optionally set a **Rollover cap**
4. Save

## Use Case

> *Maria budgets $200/month for Travel but didn't travel in January or February. With rollover enabled, her March travel budget is $600. She sets a rollover cap of $500 so it doesn't grow forever.*

## Monthly Snapshots

Rollover calculations are stored in `MonthlyBudgetSnapshot` records:

```python
class MonthlyBudgetSnapshot(models.Model):
    user = ForeignKey(User)
    category = ForeignKey(BudgetCategory)
    year = IntegerField()
    month = IntegerField()
    base_limit = DecimalField()
    rollover_in = DecimalField(default=0)
    actual_spent = DecimalField(default=0)

    @property
    def effective_budget(self):
        return self.base_limit + self.rollover_in

    @property
    def rollover_out(self):
        remaining = self.effective_budget - self.actual_spent
        if remaining <= 0:
            return Decimal("0")
        cap = self.category.rollover_cap
        if cap > 0:
            return min(remaining, cap)
        return remaining
```

## Management Command

Run monthly (or on-demand) to calculate rollovers:

```bash
uv run python manage.py calculate_rollovers
```

This command:

1. Iterates all users with rollover-enabled categories
2. For each category, snapshots the previous month's spending
3. Chains rollover amounts: previous month's `rollover_out` → current month's `rollover_in`

## Dashboard Integration

Budget progress bars on the dashboard use `effective_budget()` — which includes rollover — not just the base `monthly_limit`. This means a category showing 50% spent might actually have more room than you'd expect if rollover has accumulated.
