# Fairness Metrics

## Three Core Concepts

Fairness revolves around the relationships between the protected feature ($S$), actual outcome ($Y$), and predicted outcome ($\hat{Y}$).

### Independence

$$\hat{Y} \perp S$$

The predicted outcome is independent of the protected feature. The probability of receiving a favourable outcome has nothing to do with group membership.

### Separation

$$\hat{Y} \perp S \mid Y$$

Given the actual outcome, the predicted outcome is independent of the protected feature. Your actual class determines your prediction — not your group.

### Sufficiency

$$Y \perp S \mid \hat{Y}$$

Given the predicted outcome, the actual outcome is independent of the protected feature. The prediction's accuracy should not depend on group membership.

## Fairness Metrics in Detail

### Equal Opportunity

Both privileged and unprivileged groups have **equal FNR** (False Negative Rate):

$$P(\hat{Y} = 0 \mid Y = 1, S = S_a) = P(\hat{Y} = 0 \mid Y = 1, S = S_d)$$

!!! info "Interpretation"
    In a loan scenario: the probability of an actual non-defaulter being incorrectly predicted as a defaulter should be the **same** for both groups. No group should suffer a higher miss rate.

Since $TPR + FNR = 1$, equal FNRs also means equal TPRs (recall).

### Predictive Equality

Both groups have **equal FPR** (False Positive Rate):

$$P(\hat{Y} = 1 \mid Y = 0, S = S_a) = P(\hat{Y} = 1 \mid Y = 0, S = S_d)$$

!!! info "Interpretation"
    The probability of a defaulter being incorrectly labelled as a non-defaulter should be equal across groups.

### Equalized Odds (Disparate Mistreatment)

Both groups have **equal TPR and equal FPR**:

$$P(\hat{Y} = i \mid Y = 1, S = S_a) = P(\hat{Y} = i \mid Y = 1, S = S_d), \quad i \in \{0, 1\}$$

This combines equal opportunity and predictive equality — the strictest of the three.

### Predictive Parity (Outcome Test)

Both groups have **equal PPV** (precision):

$$P(Y = 1 \mid \hat{Y} = 1, S = S_a) = P(Y = 1 \mid \hat{Y} = 1, S = S_d)$$

!!! tip "Advantage"
    A perfectly predictive model automatically satisfies predictive parity (both groups have PPV = 1).

!!! warning "Limitation"
    Predictive parity doesn't necessarily reduce bias — it only ensures errors are spread homogeneously.

### Demographic Parity

Protected class membership has **no correlation** with receiving a favourable outcome:

$$P(\hat{Y} = 1 \mid S = S_a) = P(\hat{Y} = 1 \mid S = S_d)$$

!!! example
    If 40% of loan applicants are female, then 40% of predicted non-defaulters must be female.

### Average Odds Difference

Average of difference in FPR and TPR between groups:

$$\frac{1}{2}\left[(FPR_{S_d} - FPR_{S_a}) + (TPR_{S_d} - TPR_{S_a})\right]$$

A value of **0** means equal benefit for both groups.

## Summary Table

| Metric | What's Equal | Concept |
|--------|-------------|---------|
| Equal Opportunity | FNR (and TPR) | Separation |
| Predictive Equality | FPR | Separation |
| Equalized Odds | TPR and FPR | Separation |
| Predictive Parity | PPV / Precision | Sufficiency |
| Demographic Parity | P(Ŷ=1) | Independence |
| Average Odds Diff | Avg of FPR & TPR diff | Combined |

## Example Comparison

| Metric | Male | Female | Difference |
|--------|------|--------|------------|
| Statistical Parity | 0.733 | 0.500 | 0.233 |
| Demographic Parity | 0.667 | 0.300 | 0.333 |
| True Positive Rate | 0.727 | 0.300 | 0.427 |
| True Negative Rate | 0.500 | 0.700 | −0.167 |
| FPR | 0.500 | 0.333 | 0.167 |
| Equal Opportunity (FNR) | 0.273 | 0.667 | −0.394 |
| Predictive Parity (PPV) | 0.800 | 0.500 | 0.300 |

!!! warning "Metrics Can Conflict"
    Improving one fairness metric may worsen another. You **must prioritize** metrics based on your use case.

## Prioritizing Fairness Metrics

!!! note "Demographic Parity vs Equal Opportunity"
    - **Demographic Parity** works well with sufficient data for both groups. Without enough data, it may force favourable outcomes for likely defaulters, reducing accuracy.
    - **Equal Opportunity** takes a merit-based approach but can make disadvantaged groups sparse in the favourable-outcome set, perpetuating historical discrimination.
    
    Choose based on your context: sometimes positive discrimination (demographic parity) is needed to correct systemic wrongs.

## Python Implementation

```python
import numpy as np
from sklearn.metrics import confusion_matrix

def fairness_metrics(y_true, y_pred, protected, privileged_value):
    """Compute fairness metrics for a binary classifier."""
    priv_mask = protected == privileged_value
    unpriv_mask = ~priv_mask
    
    metrics = {}
    for label, mask in [('privileged', priv_mask), ('unprivileged', unpriv_mask)]:
        tn, fp, fn, tp = confusion_matrix(
            y_true[mask], y_pred[mask]
        ).ravel()
        metrics[label] = {
            'tpr': tp / (tp + fn) if (tp + fn) > 0 else 0,
            'fpr': fp / (fp + tn) if (fp + tn) > 0 else 0,
            'fnr': fn / (fn + tp) if (fn + tp) > 0 else 0,
            'ppv': tp / (tp + fp) if (tp + fp) > 0 else 0,
            'stat_parity': (tp + fp) / len(y_true[mask]),
        }
    
    p = metrics['privileged']
    u = metrics['unprivileged']
    
    print("=== Fairness Metrics ===")
    print(f"Equal Opportunity (FNR diff): {u['fnr'] - p['fnr']:.4f}")
    print(f"Predictive Equality (FPR diff): {u['fpr'] - p['fpr']:.4f}")
    print(f"Predictive Parity (PPV diff): {u['ppv'] - p['ppv']:.4f}")
    print(f"Demographic Parity diff: {u['stat_parity'] - p['stat_parity']:.4f}")
    avg_odds = 0.5 * ((u['fpr'] - p['fpr']) + (u['tpr'] - p['tpr']))
    print(f"Average Odds Difference: {avg_odds:.4f}")
    
    return metrics
```

---

*Next: [Proxy Features →](proxy-features.md)*
