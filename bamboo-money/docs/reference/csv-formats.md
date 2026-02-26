# Bank CSV Formats

Bamboo Money auto-detects CSV formats from four major US banks. This page documents the expected formats and how to add support for new banks.

## Supported Banks

### Chase

| Column | Header Name |
|---|---|
| Date | `Transaction Date` |
| Post Date | `Post Date` |
| Description | `Description` |
| Category | `Category` |
| Type | `Type` |
| Amount | `Amount` |

- **Date format:** `MM/DD/YYYY`
- **Amount convention:** Negative = charge, Positive = credit/refund
- **Detection:** Headers contain "Transaction Date" and "Post Date"

### American Express

| Column | Header Name |
|---|---|
| Date | `Date` |
| Reference | `Reference` |
| Description | `Description` |
| Amount | `Amount` |

- **Date format:** `MM/DD/YYYY`
- **Amount convention:** ⚠️ **Positive = charge** (opposite of most banks!)
- **Detection:** Headers contain "Date" and "Reference"

!!! warning "Inverted Signs"
    Amex is the only major bank that uses positive numbers for charges. The parser automatically inverts the sign so that expenses appear as positive amounts with `is_income=False`.

### Wells Fargo

| Column | Header Name |
|---|---|
| Date | First column |
| Amount | Second column |
| Unknown | Third column |
| Unknown | Fourth column |
| Description | Fifth column |

- **Date format:** `MM/DD/YYYY`
- **Amount convention:** Negative = charge
- **Detection:** Specific header pattern matching

### Bank of America

| Column | Header Name |
|---|---|
| Date | `Date` |
| Description | `Payee` |
| Amount | `Amount` |

- **Date format:** `MM/DD/YYYY`
- **Amount convention:** Negative = charge
- **Detection:** Headers contain "Date" and "Payee"

## Adding a New Bank

Edit `interface/csv_parser.py` and add to the `BANK_PRESETS` dictionary:

```python
BANK_PRESETS = {
    # ... existing banks ...
    "your_bank": {
        "date_col": "Date",           # Header name for date column
        "description_col": "Description",  # Header for description
        "amount_col": "Amount",        # Header for amount
        "date_formats": ["%m/%d/%Y", "%Y-%m-%d"],  # Accepted formats
        "amount_sign": "native",       # "native" or "inverted"
    },
}
```

Then update the `detect_bank()` function to recognize the new bank's header pattern:

```python
def detect_bank(headers):
    headers_lower = [h.lower().strip() for h in headers]
    if "your_header" in headers_lower:
        return "your_bank"
    # ... existing checks ...
```

### Amount Sign Options

| Value | Meaning | Banks |
|---|---|---|
| `native` | Negative = expense, Positive = income | Chase, Wells Fargo, BoA |
| `inverted` | Positive = expense, Negative = income | Amex |

## Manual Column Mapping

For unrecognized banks, the import preview page shows a manual mapping interface:

1. Select which column contains the **date**
2. Select which column contains the **description**
3. Select which column contains the **amount**
4. Choose the **amount sign convention**

The parser then uses these mappings instead of a preset.

## Common Issues

??? question "My dates aren't parsing correctly"
    Check your bank's date format. The parser tries multiple formats (`%m/%d/%Y`, `%Y-%m-%d`, `%m/%d/%y`). If yours isn't supported, add it to the preset's `date_formats` list.

??? question "All my amounts show as income"
    Your bank might use the opposite sign convention. Set `amount_sign: "inverted"` in the preset.

??? question "Duplicate detection is too aggressive"
    Duplicates match on: same amount + same description + date within ±1 day. If you have legitimate duplicate charges (e.g., two coffees on the same day), use "Import anyway" on the preview page.
