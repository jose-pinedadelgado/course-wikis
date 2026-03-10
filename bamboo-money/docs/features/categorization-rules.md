# Categorization Rules

Automatically assign categories to transactions based on merchant name patterns.

## How Rules Work

Each rule maps a **pattern** to a **category**:

| Pattern | Category | Type |
|---------|----------|------|
| `walmart` | Groceries | contains |
| `netflix` | Entertainment | contains |
| `shell` | Gas | contains |
| `starbucks` | Dining Out | contains |

When a transaction is imported (via CSV) or manually created, the system checks the merchant name against all rules. First match wins.

## Rule Management

- **Create** rules manually or from existing transactions ("Always categorize transactions from this merchant as...")
- **Edit** patterns and target categories
- **Priority ordering** — Rules are evaluated in order; first match wins
- **Delete** rules that are no longer needed

## Auto-Suggestion

When you categorize a transaction, the system offers to create a rule:

> "You categorized 'STARBUCKS #1234' as Dining Out. Create a rule for all Starbucks transactions?"
