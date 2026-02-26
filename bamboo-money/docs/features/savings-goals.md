# Savings Goals

Track progress toward financial goals with target amounts, deadlines, and contribution history.

## Creating a Goal

Navigate to **Goals** → **New Goal**:

| Field | Required | Description |
|---|---|---|
| Name | Yes | Goal name (e.g., "Emergency Fund") |
| Icon | No | Emoji icon (default: 🎯) |
| Target Amount | Yes | How much you want to save |
| Deadline | No | Target date to reach the goal |

## Goal Detail Page

Click any goal to see its detail view:

### Progress
- **Progress bar** — current amount as percentage of target
- **Current amount** — total contributed so far
- **Remaining** — target minus current
- **Progress percent** — capped at 100%

### Projections (with deadline)
- **Days remaining** — countdown to deadline
- **Monthly needed** — amount per month to reach target on time

```python
def monthly_needed(self):
    days = self.days_remaining()
    if not days or days <= 0:
        return self.remaining()
    months = max(days / 30, 1)
    return (self.remaining() / Decimal(str(months))).quantize(Decimal("0.01"))
```

### Contribution History

A chronological list of all contributions with dates, amounts, and optional notes.

## Adding Contributions

On the goal detail page, use the contribution form:

| Field | Description |
|---|---|
| Amount | Contribution amount |
| Date | Date of contribution (defaults to today) |
| Note | Optional description |

When a contribution is saved, the goal's `current_amount` is automatically incremented.

## Goals Overview

The goals list page shows all goals with:

- Icon and name
- Progress bar
- Current / target amounts
- **Total saved** across all goals
- **Total target** across all goals

## Use Case

> *Ana and David want to save $50,000 for a house down payment by December 2027. They create a goal with that target and deadline. Each month, they add a $2,000 contribution. The goal page shows they need $2,083/month to stay on track — they're slightly behind but close.*

## Data Models

```python
class SavingsGoal(models.Model):
    user = ForeignKey(User)
    name = CharField(max_length=100)
    target_amount = DecimalField(max_digits=12, decimal_places=2)
    current_amount = DecimalField(max_digits=12, decimal_places=2, default=0)
    deadline = DateField(null=True, blank=True)
    icon = CharField(max_length=10, default="🎯")

class GoalContribution(models.Model):
    goal = ForeignKey(SavingsGoal, related_name="contributions")
    amount = DecimalField(max_digits=10, decimal_places=2)
    date = DateField(default=timezone.now)
    note = CharField(max_length=200, blank=True)
```

!!! warning "Non-atomic update"
    The current implementation uses a read-modify-write pattern for `current_amount`. Under concurrent requests, this could lose updates. A production fix would use Django's `F()` expression:
    ```python
    goal.current_amount = F('current_amount') + contrib.amount
    goal.save(update_fields=['current_amount'])
    ```
    See [Security](../architecture/security.md) for more details.
