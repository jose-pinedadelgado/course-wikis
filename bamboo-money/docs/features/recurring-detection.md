# Recurring Detection

Bamboo Money automatically identifies recurring transactions (subscriptions, bills, regular payments).

## Detection Algorithm

The system analyzes transaction history looking for:

- **Same merchant** — Normalized merchant name matching
- **Similar amount** — Within a tolerance range (handles price changes)
- **Regular interval** — Monthly, bi-weekly, weekly, or quarterly patterns

## Model

```python
class RecurringTransaction(models.Model):
    merchant = models.CharField(max_length=200)
    expected_amount = models.DecimalField(max_digits=10, decimal_places=2)
    frequency = models.CharField(choices=FREQUENCY_CHOICES)
    next_expected_date = models.DateField()
    is_active = models.BooleanField(default=True)
```

## Usage

- View all detected recurring transactions on the recurring page
- Mark transactions as recurring or not-recurring to train the detector
- Get alerts when expected recurring transactions are missing
- See monthly recurring commitments total
