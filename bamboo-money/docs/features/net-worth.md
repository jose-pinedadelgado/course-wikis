# Net Worth Tracking

Track your total financial picture — assets, liabilities, and net worth over time.

## Overview

The net worth page (`/networth/`) displays:

- **Assets** — savings accounts, investments, property, vehicles
- **Liabilities** — loans, credit card debt, mortgages
- **Net Worth** — assets minus liabilities
- **Historical chart** — net worth over time from monthly snapshots

## Adding Entries

Click **Add Entry** and fill in:

| Field | Description | Example |
|---|---|---|
| Name | Entry name | "401(k)", "Student Loan", "Honda Civic" |
| Type | Asset or Liability | Asset |
| Amount | Current value | $45,000 |
| Category | Optional grouping | "Retirement", "Real Estate", "Debt" |

## Historical Snapshots

`NetWorthSnapshot` records store monthly totals for the historical chart:

```python
class NetWorthSnapshot(models.Model):
    user = ForeignKey(User)
    date = DateField()
    total_assets = DecimalField(max_digits=14, decimal_places=2)
    total_liabilities = DecimalField(max_digits=14, decimal_places=2)
    net_worth = DecimalField(max_digits=14, decimal_places=2)

    class Meta:
        unique_together = ["user", "date"]
```

The chart displays these snapshots as a line graph over time, giving you a long-term view of financial progress.

## Dashboard Integration

The dashboard shows a condensed net worth summary:

```python
def _calc_net_worth(user):
    entries = NetWorthEntry.objects.filter(user=user)
    assets = entries.filter(entry_type="asset").aggregate(t=Sum("amount"))["t"] or 0
    liabs = entries.filter(entry_type="liability").aggregate(t=Sum("amount"))["t"] or 0
    return {"assets": assets, "liabilities": liabs, "net": assets - liabs}
```

## Use Case

> *Priya has a 401(k) worth $120,000, a home valued at $450,000, and a mortgage of $350,000. Her net worth page shows: Assets $570,000, Liabilities $350,000, Net Worth $220,000. Each month she updates the values and watches the chart trend upward.*

## Data Model

```python
class NetWorthEntry(models.Model):
    ENTRY_TYPES = [
        ("asset", "Asset"),
        ("liability", "Liability"),
    ]
    user = ForeignKey(User)
    name = CharField(max_length=100)
    entry_type = CharField(max_length=10, choices=ENTRY_TYPES)
    amount = DecimalField(max_digits=14, decimal_places=2)
    category = CharField(max_length=50, blank=True)
    updated_at = DateTimeField(auto_now=True)
```

!!! tip "Manual updates"
    Net worth entries are **manually maintained** — update them monthly when you check account balances. A future enhancement could auto-sync from account balances if bank linking is added.
