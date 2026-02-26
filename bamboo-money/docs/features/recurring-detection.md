# Recurring Transaction Detection

Bamboo Money automatically detects subscriptions, memberships, and recurring bills by analyzing patterns in your transaction history.

## Detection Algorithm

The `detect_recurrings` management command:

### 1. Normalize Merchant Names

Raw bank descriptions are messy. The normalizer strips noise:

```python
def normalize_merchant(description):
    desc = description.strip().upper()
    # Remove store/location numbers: "STARBUCKS #12345" → "STARBUCKS"
    desc = re.sub(r'[#*]\s*\d+', '', desc)
    # Remove trailing long numbers
    desc = re.sub(r'\s+\d{4,}', '', desc)
    # Remove state codes: "SHELL OIL CA" → "SHELL OIL"
    desc = re.sub(r'\s+(CA|NY|TX|FL|...)\b', '', desc)
    return desc[:60]
```

### 2. Group by Merchant

All transactions with the same normalized merchant name are grouped together.

### 3. Calculate Intervals

For each merchant group with 3+ transactions, calculate the days between consecutive charges:

```python
dates = sorted([t.date for t in merchant_txns])
intervals = [(dates[i+1] - dates[i]).days for i in range(len(dates)-1)]
```

### 4. Classify Frequency

Using median interval and standard deviation:

| Frequency | Median Range | Max Std Dev |
|---|---|---|
| Weekly | 5–9 days | 3 |
| Bi-weekly | 12–16 days | 4 |
| Monthly | 25–35 days | 10 |
| Quarterly | 85–100 days | 15 |
| Yearly | 350–380 days | 20 |

If the intervals don't fit any pattern, the merchant is skipped.

### 5. Create Records

For each detected recurring:

- **Expected amount** = median of all transaction amounts
- **Next expected date** = last seen date + frequency interval
- **Category** = category of the most recent transaction

## Running Detection

```bash
# All users
uv run python manage.py detect_recurrings

# Specific user
uv run python manage.py detect_recurrings --user demo

# Require more occurrences (default: 3)
uv run python manage.py detect_recurrings --min-occurrences 5
```

You can also trigger detection from the web UI: **Recurring** → **Detect Recurring** button.

## Recurring Page

The recurring transactions page (`/recurring/`) shows:

### Active Tab
- Merchant name with icon
- Expected amount and frequency
- Next expected date
- Category assignment
- **Confirm** — verify a detection is correct
- **Pause** — hide without deleting
- **Delete** — remove entirely

### Paused Tab
- Previously paused recurring transactions
- **Resume** — reactivate

### Summary Metrics
- **Monthly total** — estimated total recurring cost per month (adjusts for frequency)
- **Upcoming** — charges expected in the next 14 days
- **Overdue** — charges past their expected date

## Dashboard Integration

The dashboard shows up to 5 upcoming recurring transactions for the current month:

```python
upcoming_recurring = RecurringTransaction.objects.filter(
    user=user, is_active=True,
    next_expected_date__year=now.year,
    next_expected_date__month=now.month,
    next_expected_date__day__gt=now.day,
).order_by("next_expected_date")[:5]
```

## Use Case

> *Jose runs "Detect Recurring" and discovers he's still paying $14.99/month for a streaming service he forgot about. The recurring page shows it's due in 3 days. He cancels it and saves $180/year.*

## Data Model

```python
class RecurringTransaction(models.Model):
    FREQUENCIES = [
        ("weekly", "Weekly"),
        ("biweekly", "Bi-weekly"),
        ("monthly", "Monthly"),
        ("quarterly", "Quarterly"),
        ("yearly", "Yearly"),
    ]
    user = ForeignKey(User)
    merchant = CharField(max_length=200)
    category = ForeignKey(BudgetCategory, null=True)
    expected_amount = DecimalField(max_digits=10, decimal_places=2)
    frequency = CharField(max_length=20, choices=FREQUENCIES)
    next_expected_date = DateField(null=True)
    last_seen_date = DateField(null=True)
    is_confirmed = BooleanField(default=False)
    is_active = BooleanField(default=True)
    icon = CharField(max_length=10, default="🔄")
```

## Limitations

!!! note "Detection requires history"
    The algorithm needs at least **3 occurrences** of a merchant to detect a pattern. New subscriptions won't appear until enough data exists.

!!! note "Same-day transactions excluded"
    Intervals of 0 days (multiple charges on the same day) are filtered out to avoid false positives.

!!! note "Amount variation"
    The algorithm uses **median** amount, not exact match. This handles slight price changes (e.g., tax adjustments) but may group genuinely different charges from the same merchant.
