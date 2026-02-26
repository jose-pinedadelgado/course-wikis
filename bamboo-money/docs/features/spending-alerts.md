# Spending Alerts

Spending alerts proactively warn you when budget categories are approaching or exceeding their limits, or when spending patterns look unusual.

## Alert Types

| Type | Trigger | Icon | Color |
|---|---|---|---|
| **Approaching** | Category reaches 80% of limit | :material-alert: | Yellow |
| **Exceeded** | Category reaches 100% of limit | :material-close-circle: | Red |
| **Unusual** | Spending is >1.5× the 3-month average | :material-lightning-bolt: | Blue |

## How Alerts Are Generated

Alerts are checked on every **dashboard page load** via the `_check_budget_alerts()` function. This means:

- No background jobs or cron needed
- Alerts appear naturally when the user checks in
- Each alert type is generated **once per category per month** (deduplicated)

### Approaching & Exceeded

```python
for category in user.categories.filter(is_income=False, monthly_limit__gt=0):
    pct = category.progress_percent()
    existing = BudgetAlert.objects.filter(
        user=user, category=category,
        month=now.month, year=now.year
    )
    if pct >= 100 and not existing.filter(alert_type="exceeded").exists():
        # Create "exceeded" alert
    elif pct >= 80 and not existing.filter(alert_type="approaching").exists():
        # Create "approaching" alert
```

### Unusual Spending

For each category with spending this month:

1. Calculate the **3-month rolling average** (looking back 1, 2, and 3 months)
2. If current month spending > **1.5× the average** → flag as unusual
3. Only flags once per category per month

```python
# Example: Food & Dining
# Sept: $400, Oct: $380, Nov: $420 → average = $400
# December: $650 → 650/400 = 1.625 → exceeds 1.5× threshold
# → Alert: "Unusual spending on Food & Dining: $650 this month vs $400 average"
```

## Dashboard Display

Alerts appear as dismissible banners at the top of the dashboard:

- Up to **5 unread alerts** shown
- Total alert count badge visible
- **Dismiss individual** — HTMX DELETE marks as read, removes the banner
- **Dismiss all** — marks all unread alerts as read

## Use Case

> *Jose opens his dashboard mid-month and sees a yellow banner: "You're at 85% of your Food & Dining budget ($510 of $600)." He adjusts his eating-out plans for the rest of the month.*

## Data Model

```python
class BudgetAlert(models.Model):
    ALERT_TYPES = [
        ("approaching", "Approaching Limit"),
        ("exceeded", "Budget Exceeded"),
        ("unusual", "Unusual Spending"),
    ]
    user = ForeignKey(User)
    category = ForeignKey(BudgetCategory, null=True)
    alert_type = CharField(max_length=20, choices=ALERT_TYPES)
    message = TextField()
    is_read = BooleanField(default=False)
    month = IntegerField()   # Prevents re-triggering
    year = IntegerField()    # Same month/year/type/category = skip
    created_at = DateTimeField(auto_now_add=True)
```

## Future Enhancements

- **Email notifications** — weekly digest of alerts
- **Push notifications** — browser push for exceeded budgets
- **Large transaction alerts** — configurable threshold (e.g., anything over $500)
- **Recurring change alerts** — "Netflix charged $22.99 (was $15.99)"
- **Recurring missing alerts** — "Gym membership expected 3 days ago — not seen"
