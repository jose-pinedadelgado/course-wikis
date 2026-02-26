# Budget Management

Budgets are organized by category. Each category has a monthly spending limit, and Bamboo Money tracks your progress throughout the month.

## Default Categories

When a new user registers, 12 default categories are created automatically:

| Icon | Category | Default Limit |
|---|---|---|
| 🏠 | Housing | $1,500 |
| 🍔 | Food & Dining | $600 |
| 🚗 | Transportation | $400 |
| 🛒 | Groceries | $500 |
| 💊 | Health | $200 |
| 🎬 | Entertainment | $150 |
| 👔 | Shopping | $200 |
| 📱 | Subscriptions | $100 |
| 💡 | Utilities | $250 |
| 📚 | Education | $100 |
| ✈️ | Travel | $200 |
| 💰 | Income | $0 (income category) |

## Budget View

The budget page (`/budgets/`) shows:

- **Total budget** — sum of all category limits
- **Total spent** — sum of all expenses this month
- **Remaining** — total budget minus total spent

Each category displays:

- Icon + name
- Progress bar (color-coded green/yellow/red)
- Spent amount vs. limit
- Effective budget (if [rollover](budget-rollover.md) is enabled)

## Creating & Editing Categories

| Field | Description |
|---|---|
| Name | Category name (unique per user) |
| Icon | Emoji icon (e.g., 🍔) |
| Color | Hex color for charts (color picker) |
| Monthly Limit | Target spending cap |
| Rollover Enabled | Carry unused budget forward |
| Rollover Cap | Max rollover amount (0 = unlimited) |

## How Spending Is Calculated

Each category has a `spent_this_month()` method that queries:

```python
def spent_this_month(self):
    now = timezone.now()
    total = self.transactions.filter(
        date__year=now.year,
        date__month=now.month,
        is_income=False
    ).aggregate(total=Sum("amount"))["total"]
    return total or Decimal("0.00")
```

## Progress Bar Logic

```python
def progress_percent(self):
    effective = self.effective_budget()  # base + rollover
    if effective <= 0:
        return 0
    return min(int((self.spent_this_month() / effective) * 100), 100)

def progress_color(self):
    pct = self.progress_percent()
    if pct >= 100: return "danger"    # red
    elif pct >= 80: return "warning"  # yellow
    return "success"                   # green
```

## Data Model

```python
class BudgetCategory(models.Model):
    user = ForeignKey(User)
    name = CharField(max_length=100)
    icon = CharField(max_length=10, default="📦")
    color = CharField(max_length=7, default="#6c757d")
    monthly_limit = DecimalField(max_digits=10, decimal_places=2, default=0)
    is_income = BooleanField(default=False)
    rollover_enabled = BooleanField(default=False)
    rollover_cap = DecimalField(max_digits=10, decimal_places=2, default=0)

    class Meta:
        unique_together = ["user", "name"]
```
