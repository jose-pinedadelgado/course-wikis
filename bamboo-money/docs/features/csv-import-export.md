# CSV Import & Export

## Import

Bamboo Money auto-detects bank CSV formats for seamless transaction import.

### Supported Banks

The CSV parser recognizes 10+ bank formats by analyzing header columns:

- Chase (credit & checking)
- Bank of America
- Wells Fargo
- Capital One
- Citi
- American Express
- Discover
- USAA
- Navy Federal
- Generic (fallback: Date, Description, Amount)

### Import Flow

1. **Upload** — Select a CSV file from your bank's export
2. **Auto-detect** — Parser identifies the bank format from column headers
3. **Preview** — See parsed transactions before importing
4. **Deduplication** — Smart matching prevents duplicate imports (same date + amount + merchant within a window)
5. **Auto-categorize** — Categorization rules auto-assign categories based on merchant patterns
6. **Import** — Confirmed transactions are saved

### Deduplication Logic

A transaction is considered a duplicate if all three match within a 3-day window:

- Same merchant name (normalized)
- Same amount
- Date within ±3 days of an existing transaction

## Export

- **CSV Export** — Download filtered transactions as CSV
- **Excel Export** — See [Excel Export](excel-export.md) for the formatted .xlsx option
