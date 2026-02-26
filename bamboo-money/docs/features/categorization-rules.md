# Auto-Categorization Rules

Categorization rules automatically assign budget categories to transactions based on keyword matching. Instead of manually categorizing every Starbucks charge, create a rule once and it handles the rest.

## How Rules Work

Each rule has:

| Field | Description | Example |
|---|---|---|
| **Keyword** | Text to match against | `STARBUCKS` |
| **Match Type** | How to match | Contains, Starts With, Exact Match |
| **Category** | Category to assign | Food & Dining |
| **Priority** | Order of evaluation (higher = first) | 10 |

### Match Types

| Type | Behavior | Example |
|---|---|---|
| **Contains** | Keyword appears anywhere in text | "STARBUCKS" matches "STARBUCKS #12345 LOS ANGELES CA" |
| **Starts With** | Text begins with keyword | "NETFLIX" matches "NETFLIX.COM" but not "HD NETFLIX" |
| **Exact Match** | Full text must equal keyword | "RENT" matches only "RENT", not "CAR RENTAL" |

All matching is **case-insensitive**.

### Priority

When multiple rules could match the same transaction, the highest priority rule wins. Rules are evaluated against both the `description` and `merchant` fields.

## Three Ways to Create Rules

### 1. Rules Page

Navigate to **Rules** → **New Rule**. Fill in keyword, match type, category, and priority.

### 2. From a Transaction (One-Click)

On the Transactions page, click the :material-tag-plus: icon on any categorized transaction. This creates a "contains" rule using the transaction's merchant as the keyword and its current category.

```python
# Creates a rule like:
# keyword="STARBUCKS", match_type="contains", category=Food & Dining
CategorizationRule.objects.get_or_create(
    user=request.user,
    keyword=txn.merchant,
    defaults={"match_type": "contains", "category": txn.category}
)
```

### 3. Bulk Apply

Click **Apply All Rules** on the Rules page to run every rule against all uncategorized transactions:

```python
rules = CategorizationRule.objects.filter(user=user).order_by("-priority")
uncategorized = Transaction.objects.filter(user=user, category__isnull=True)
for txn in uncategorized:
    for rule in rules:
        if rule.matches(txn.description) or rule.matches(txn.merchant):
            txn.category = rule.category
            txn.save()
            break  # First matching rule wins
```

A success message shows: *"Applied rules: X transactions categorized."*

## Use Case

> *Every Starbucks charge lands as "uncategorized." Maria creates a rule: keyword "STARBUCKS", match type "contains", category "Food & Dining." She clicks "Apply All" — all 23 Starbucks transactions get categorized instantly. Future imports will benefit from the same rule.*

## Data Model

```python
class CategorizationRule(models.Model):
    MATCH_TYPES = [
        ("contains", "Contains"),
        ("starts_with", "Starts With"),
        ("exact", "Exact Match"),
    ]
    user = ForeignKey(User)
    keyword = CharField(max_length=100)
    match_type = CharField(max_length=20, choices=MATCH_TYPES, default="contains")
    category = ForeignKey(BudgetCategory)
    priority = IntegerField(default=0)
    created_at = DateTimeField(auto_now_add=True)

    def matches(self, text):
        text_lower = text.lower()
        kw_lower = self.keyword.lower()
        if self.match_type == "contains":
            return kw_lower in text_lower
        elif self.match_type == "starts_with":
            return text_lower.startswith(kw_lower)
        elif self.match_type == "exact":
            return text_lower == kw_lower
        return False
```

## Future: ML-Based Categorization

The current rule system is deterministic and user-controlled. A future enhancement could add machine learning:

1. Train a classifier on the user's historical categorizations
2. Use as fallback when no rule matches
3. Present suggestions with confidence scores ("Is this Food & Dining? 92% confident")

This is listed as a roadmap item — see [Roadmap](../product/roadmap.md).
