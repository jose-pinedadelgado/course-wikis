# Transactions

Transactions are the foundation of Bamboo Money. Every financial record — whether imported from CSV, entered manually, or detected as recurring — is stored as a Transaction.

## Transaction List

The main transactions page (`/transactions/`) provides:

### Filtering

| Filter | Type | Description |
|---|---|---|
| Search | Text | Matches against description and merchant name |
| Category | Dropdown | Filter by budget category |
| Date From | Date | Start of date range |
| Date To | Date | End of date range |
| Type | Dropdown | All, Income, or Expense |

Filters are passed as query parameters and work with both the full page view and HTMX partial updates.

### Pagination

Transactions display **25 per page**. This was specifically tuned to keep page sizes manageable (~62KB vs ~626KB for all transactions at once).

### HTMX Integration

Two HTMX-powered interactions:

1. **Inline category editing** — Each transaction row has a category dropdown. Changing it sends a POST to `/transactions/<pk>/category/` and re-renders just that row.

2. **Create rule from transaction** — Click the tag icon to instantly create a [categorization rule](categorization-rules.md) from the transaction's merchant and current category. Returns a "Rule created" badge via HTMX.

## Adding Transactions

Click **Add Transaction** to open the form:

| Field | Required | Description |
|---|---|---|
| Date | Yes | Transaction date |
| Description | Yes | What was purchased (e.g., "Starbucks Coffee") |
| Merchant | No | Merchant name (e.g., "Starbucks") |
| Amount | Yes | Dollar amount (always positive) |
| Category | No | Budget category assignment |
| Account | No | Which account (Chase Checking, Amex, etc.) |
| Is Income | No | Check if this is income, not an expense |
| Notes | No | Free-form notes |

## Editing and Deleting

- **Edit** — Click the edit icon on any transaction to modify all fields
- **Delete** — Click delete, then confirm. Deletion is permanent.

## Data Model

```python
class Transaction(models.Model):
    user = ForeignKey(User)
    account = ForeignKey(Account, null=True)
    category = ForeignKey(BudgetCategory, null=True)
    import_batch = ForeignKey(ImportBatch, null=True)
    date = DateField()
    description = CharField(max_length=255)
    merchant = CharField(max_length=200, blank=True)
    amount = DecimalField(max_digits=10, decimal_places=2)
    is_income = BooleanField(default=False)
    notes = TextField(blank=True)
    created_at = DateTimeField(auto_now_add=True)
```

!!! info "Amount Convention"
    Amounts are always stored as **positive numbers**. The `is_income` boolean determines whether the transaction is income or expense. This simplifies display logic and aggregation.

## Shared Filter Helper

Transaction list and CSV export share the same filter logic via `_get_filtered_transactions()`:

```python
def _get_filtered_transactions(request):
    """Returns (queryset, filter_dict) for reuse in list and export views."""
    txns = Transaction.objects.filter(user=request.user)
    # Apply q, category, date_from, date_to, type filters
    return txns, filters
```

This ensures exports always match what you see on screen.
