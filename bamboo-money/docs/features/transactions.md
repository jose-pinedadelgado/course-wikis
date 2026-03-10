# Transactions

Transactions are the core data unit in Bamboo Money. Every income and expense flows through the transaction system.

## Fields

| Field | Type | Description |
|-------|------|-------------|
| Date | date | When the transaction occurred |
| Merchant | text | Who you paid / who paid you |
| Amount | decimal | Dollar amount (positive = expense, negative = income, or vice versa depending on bank) |
| Category | FK | Budget category this belongs to |
| Account | FK | Which bank account |
| Notes | text | Optional notes |
| Is Recurring | bool | Flagged by recurring detection |

## Search & Filter

- **Text search** across merchant name and notes
- **Date range** filter
- **Category** filter (dropdown)
- **Account** filter
- Pagination with configurable page size

## Manual Entry

Add transactions directly through a form. Auto-suggests categories based on merchant name (using categorization rules).

## Bulk Operations

- Select multiple transactions for bulk category assignment
- Bulk delete with confirmation
- CSV export of filtered results
