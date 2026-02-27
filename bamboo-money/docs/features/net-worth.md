# Net Worth Tracking

Track your total financial picture — assets, liabilities, and net worth over time with rich visualizations and key financial health metrics.

## Overview

The net worth page (`/networth/`) is a full financial dashboard with:

- **Hero cards** — Total Assets, Total Liabilities, Net Worth, Monthly Change
- **8 key metrics** — Liquid Months, Debt-to-Asset Ratio, FI Ratio, Savings Rate, Invested Capital, Liquid Net Worth, Debt Payoff Timeline, Retirement total
- **Trend chart** — Assets, Liabilities, and Net Worth plotted together over time
- **Asset allocation donut** — Visual breakdown by account group
- **Stacked area chart** — Composition over time by group
- **Collapsible group breakdowns** — Individual accounts organized by type

## Account Groups

Every net worth entry belongs to one of five account groups, enabling automatic organization and reporting:

| Group | Icon | Color | Examples |
|---|---|---|---|
| **Liquid** | 💧 | Blue | Checking, savings, emergency fund, HSA, cash |
| **Investments** | 📈 | Green | Brokerage accounts, crypto, index funds |
| **Retirement** | 🏦 | Purple | 401(k), 403(b), IRA, Roth IRA, pension, 457 |
| **Other Assets** | 📦 | Gray | Vehicles, property, equipment |
| **Liabilities** | 💳 | Red | Credit cards, loans, medical bills |

```python
class NetWorthEntry(models.Model):
    ACCOUNT_GROUPS = [
        ("liquid", "Liquid (Cash & Equivalents)"),
        ("liability", "Liabilities"),
        ("investment", "Investments & Brokerage"),
        ("retirement", "Retirement"),
        ("other_asset", "Other Assets"),
    ]
    name = CharField(max_length=100)
    entry_type = CharField(choices=[("asset", "Asset"), ("liability", "Liability")])
    account_group = CharField(max_length=20, choices=ACCOUNT_GROUPS, default="liquid")
    amount = DecimalField(max_digits=14, decimal_places=2)
```

## Key Metrics

### Row 1: Core Health Indicators

| Metric | Formula | What it tells you |
|---|---|---|
| **Liquid Months** | Cash reserves ÷ avg monthly expenses | How long you can survive on cash alone |
| **Debt-to-Asset Ratio** | Total liabilities ÷ total assets × 100 | How leveraged you are (<10% = Excellent) |
| **FI Ratio** | Net worth ÷ annual expenses | Progress toward financial independence (target: 25×) |
| **Savings Rate** | Month-over-month net worth growth % | How fast your wealth is growing |

### Row 2: Wealth Composition

| Metric | Formula | What it tells you |
|---|---|---|
| **Invested Capital** | Investments + Retirement totals | How much of your money is working for you (shown as % of NW) |
| **Liquid Net Worth** | Liquid assets − all liabilities | What's left if you paid off all debts today from cash |
| **Debt Payoff** | Total liabilities ÷ avg monthly paydown rate | Months until debt-free at current pace |
| **Retirement** | Total in retirement accounts | % of net worth locked in retirement vehicles |

## Historical Tracking

### Per-Account History

`NetWorthEntryHistory` tracks each account's balance at each point in time, enabling per-account trends and the stacked area chart:

```python
class NetWorthEntryHistory(models.Model):
    entry = ForeignKey(NetWorthEntry, related_name="history")
    date = DateField()
    amount = DecimalField(max_digits=14, decimal_places=2)

    class Meta:
        unique_together = ["entry", "date"]
```

### Monthly Snapshots

`NetWorthSnapshot` records store monthly totals for the main trend chart:

```python
class NetWorthSnapshot(models.Model):
    date = DateField()
    total_assets = DecimalField(max_digits=14, decimal_places=2)
    total_liabilities = DecimalField(max_digits=14, decimal_places=2)
    net_worth = DecimalField(max_digits=14, decimal_places=2)
```

## Visualizations

### Net Worth Trend (Chart.js)
Three solid lines plotted together — **Assets** (blue), **Net Worth** (green), **Liabilities** (red) — so you can see all three dimensions at once.

### Asset Allocation Donut
Current breakdown by account group with percentages. Hover for dollar amounts.

### Stacked Area: Composition Over Time
Shows how Liquid, Investments, Retirement, and Other Assets contribute to total wealth over time.

### Collapsible Group Cards
Each account group renders as a card with the group total in the header. Click to expand/collapse and see individual accounts with edit/delete controls.

## Excel Export

Click **"Export to Sheets"** to download a formatted `.xlsx` workbook with 3 tabs:

### Dashboard Tab
- Bamboo Money branded header
- Summary cards (Assets / Liabilities / Net Worth)
- 6 key metrics table with explanations
- Full account breakdown table, color-coded by group
- Embedded line chart (trend) and pie chart (allocation)

### Net Worth Tracker Tab
- **Accounts as columns**, **dates as rows** (scales well in Google Sheets)
- Color-coded group headers (blue/green/purple/gray/red)
- Group subtotals, Total Assets, Total Liabilities, Net Worth
- Computed columns: MoM $ Change, MoM % Change, Debt/Asset %, Liquid Months
- Frozen panes, alternating row stripes, number formatting

### Charts Tab
- Stacked area chart showing composition over time
- Bar chart comparing current holdings by group

!!! tip "Google Sheets compatible"
    The `.xlsx` file opens directly in Google Sheets with formatting, colors, and charts preserved.

## Dashboard Integration

The main dashboard shows a condensed net worth card using:

```python
def _calc_net_worth(user):
    entries = NetWorthEntry.objects.filter(user=user)
    assets = entries.filter(entry_type="asset").aggregate(t=Sum("amount"))["t"] or 0
    liabs = entries.filter(entry_type="liability").aggregate(t=Sum("amount"))["t"] or 0
    return {"assets": assets, "liabilities": liabs, "net": assets - liabs}
```

## Use Case

> *Jose tracks 34 accounts across 5 groups: 14 liquid accounts ($68K), 10 investments ($176K), 5 retirement accounts ($230K), a car ($12K), and 4 liabilities ($16K). His net worth page shows $470K with an FI ratio of 5.6×, 86% invested, and 4.2 liquid months. He exports monthly to Google Sheets to track progress toward financial independence.*

## Demo Account

Run `python manage.py seed_demo2` to create a **demo2/demo2** account with realistic data:

- 34 net worth entries across all 5 groups
- 6 months of per-account historical data (204 records)
- 12 monthly snapshots showing growth trajectory
- ~70 sample transactions for expense metrics
- 3 savings goals
