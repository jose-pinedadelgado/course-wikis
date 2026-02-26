# CSV Import & Export

Bamboo Money's privacy-first approach means your data comes in through file uploads, not bank API connections. CSV import is the primary way to get real transaction data into the app.

## CSV Export

### How It Works

Click **Export CSV** on the Transactions page to download a file with all currently filtered transactions.

The export:

- Respects all active filters (search, category, date range, type)
- Downloads as `bamboo_transactions_YYYY-MM-DD.csv`
- Includes columns: Date, Description, Merchant, Amount, Type, Category, Account, Notes

### Use Case

> *Maria wants to share her Q1 expenses with her accountant. She filters transactions from Jan–Mar, clicks "Export CSV," and emails the file.*

---

## CSV Import

### Three-Step Flow

#### Step 1: Upload

Navigate to **Import CSV** in the sidebar (or `/import/`).

- Drag and drop a `.csv` file or use the file picker
- Maximum 500 rows per import (stored in Django session)
- Only `.csv` files accepted

#### Step 2: Preview

The `csv_parser.py` engine analyzes your file:

**Auto-Detection:** The parser checks column headers against known bank patterns:

| Bank | Key Headers | Amount Convention |
|---|---|---|
| Chase | "Transaction Date", "Post Date", "Description" | Negative = charge |
| American Express | "Date", "Reference", "Description" | Positive = charge (reversed!) |
| Wells Fargo | Specific column order pattern | Negative = charge |
| Bank of America | "Date", "Payee" | Negative = charge |

!!! warning "Amex Convention"
    American Express uses **positive** numbers for charges (opposite of most banks). The parser handles this automatically by inverting the sign.

**Manual Mapping Fallback:** For unrecognized bank formats, you'll see dropdowns to map columns:

- Date column
- Description column
- Amount column
- Amount sign convention (native or inverted)

**Preview Display:**

- Parsed transactions in a table
- :material-alert: Duplicate warnings (yellow highlight) for transactions matching existing records
- Error count for unparseable rows

**Duplicate Detection:**

```python
# A transaction is flagged as duplicate if it matches an existing record:
# - Same amount
# - Same description (case-insensitive)
# - Date within ±1 day
```

#### Step 3: Confirm

- Select/deselect individual transactions
- Optionally force-import flagged duplicates
- Click **Import** to create the transactions

All imported transactions are linked to an `ImportBatch` record for audit trail:

```python
class ImportBatch(models.Model):
    user = ForeignKey(User)
    filename = CharField(max_length=255)
    source_bank = CharField(max_length=50)
    row_count = IntegerField()
    imported_count = IntegerField()
    skipped_duplicates = IntegerField(default=0)
    created_at = DateTimeField(auto_now_add=True)
```

### Use Case

> *Jose switches from Mint. He downloads 6 months of Chase statements as CSV. He uploads to Bamboo Money — it auto-detects "Chase" format, parses 306 transactions, flags 4 duplicates, and lets him review before confirming. All imported in 30 seconds.*

### Adding Support for New Banks

To add a new bank format, edit `csv_parser.py` and add an entry to `BANK_PRESETS`:

```python
BANK_PRESETS = {
    "chase": {
        "date_col": "Transaction Date",
        "description_col": "Description",
        "amount_col": "Amount",
        "date_formats": ["%m/%d/%Y"],
        "amount_sign": "native",  # negative = expense
    },
    # Add your bank here:
    "your_bank": {
        "date_col": "Date",
        "description_col": "Description",
        "amount_col": "Amount",
        "date_formats": ["%m/%d/%Y", "%Y-%m-%d"],
        "amount_sign": "native",
    },
}
```
