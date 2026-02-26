# Management Commands

Bamboo Money includes three custom Django management commands.

## seed_demo_data

Load realistic demo data for testing and development.

```bash
uv run python manage.py seed_demo_data
```

### What It Creates

| Data | Count | Details |
|---|---|---|
| Users | 1 | `demo` / `demo` |
| Budget Categories | 12 | With monthly limits and icons |
| Accounts | 3 | Chase Checking, Amex Gold, Cash |
| Transactions | 306 | 6 months of realistic spending |
| Savings Goals | 3 | With contributions |
| Net Worth Entries | 9 | Assets and liabilities |
| Net Worth Snapshots | 6 | Monthly history |

### Notes

- Running twice will error on duplicate user — delete the database first if re-seeding
- Transactions include realistic merchants (Starbucks, Trader Joe's, Netflix, etc.)
- Amount ranges are calibrated per category (e.g., groceries $30-150, subscriptions $5-30)

---

## detect_recurrings

Analyze transaction history to find recurring charges (subscriptions, bills, memberships).

```bash
# All users
uv run python manage.py detect_recurrings

# Specific user
uv run python manage.py detect_recurrings --user demo

# Require more occurrences (default: 3)
uv run python manage.py detect_recurrings --min-occurrences 5
```

### Algorithm

1. Group transactions by normalized merchant name
2. Calculate intervals between consecutive charges
3. Classify frequency using median interval + standard deviation
4. Create/update `RecurringTransaction` records

### Output

```
demo: 8 new recurring transactions detected
```

### Options

| Flag | Default | Description |
|---|---|---|
| `--user` | All users | Username to analyze |
| `--min-occurrences` | 3 | Minimum charges to flag as recurring |

---

## calculate_rollovers

Calculate budget rollover amounts for the previous month.

```bash
uv run python manage.py calculate_rollovers
```

### What It Does

For each user with rollover-enabled categories:

1. Looks at the **previous month's** spending
2. Calculates how much budget was unused
3. Applies rollover cap if set
4. Creates/updates a `MonthlyBudgetSnapshot` record

### Output

```
  Food & Dining: spent $450, rollover out $150
  Entertainment: spent $80, rollover out $70
  Travel: spent $0, rollover out $200
Rollovers calculated.
```

### When to Run

- **Monthly** — at the start of each month to snapshot the previous month
- **On demand** — anytime you want to recalculate
- **After seeding** — to populate rollover data for demo accounts

!!! note "Idempotent"
    Running multiple times for the same month is safe — it uses `update_or_create` to avoid duplicates.
