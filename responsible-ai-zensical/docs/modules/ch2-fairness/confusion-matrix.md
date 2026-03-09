# Confusion Matrix & Accuracy Metrics

## The Confusion Matrix

The confusion matrix is the **foundation** for all fairness metrics. It breaks predictions into four categories:

|  | **Predicted Positive** ($\hat{Y}_{fav}$) | **Predicted Negative** ($\hat{Y}_{unfav}$) |
|---|---|---|
| **Actual Positive** ($Y_{fav}$) | True Positive (TP) | False Negative (FN) |
| **Actual Negative** ($Y_{unfav}$) | False Positive (FP) | True Negative (TN) |

!!! note "Definition"
    A confusion matrix $M$ where $\sum_{i=1}^{n}\sum_{j=1}^{n} M$ equals the number of observations known to be in group $Y_i$ and predicted to be in group $Y_j$.

### Building a Confusion Matrix in Python

```python
from sklearn.metrics import confusion_matrix

y_actual = [0, 1, 0, 1, 0, 1, 1, 0]
y_pred   = [0, 0, 0, 1, 1, 1, 0, 0]

cm = confusion_matrix(y_actual, y_pred)
print("Confusion Matrix:")
print(cm)
```

Output:
```
[[3 1]
 [2 2]]
```

### Extracting Values

```python
tn, fp, fn, tp = confusion_matrix(y_actual, y_pred).ravel()

print(f"True Positive:  {tp}")  # 2
print(f"True Negative:  {tn}")  # 3
print(f"False Positive: {fp}")  # 1
print(f"False Negative: {fn}")  # 2
```

## Common Accuracy Metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **False Positive Rate (FPR)** | $\frac{FP}{FP + TN}$ | Probability of a false alarm |
| **False Negative Rate (FNR)** | $\frac{FN}{FN + TP}$ | Probability of missing a true positive (miss rate) |
| **True Positive Rate (TPR)** | $\frac{TP}{TP + FN}$ | Sensitivity / Recall |
| **True Negative Rate (TNR)** | $\frac{TN}{TN + FP}$ | Specificity |
| **Precision (PPV)** | $\frac{TP}{TP + FP}$ | Fraction of positive predictions that are correct |

!!! tip "Key Relationships"
    - $TPR + FNR = 1$
    - $FPR + TNR = 1$
    - A classifier with equal FNRs across groups will also have equal TPRs

### Computing Metrics in Python

```python
from sklearn.metrics import (
    confusion_matrix, accuracy_score, 
    precision_score, recall_score, f1_score
)

y_actual = [0, 1, 0, 1, 0, 1, 1, 0]
y_pred   = [0, 0, 0, 1, 1, 1, 0, 0]

tn, fp, fn, tp = confusion_matrix(y_actual, y_pred).ravel()

fpr = fp / (fp + tn)
fnr = fn / (fn + tp)
tpr = tp / (tp + fn)  # recall
tnr = tn / (tn + fp)
ppv = tp / (tp + fp)  # precision

print(f"FPR: {fpr:.3f}")
print(f"FNR: {fnr:.3f}")
print(f"TPR (Recall): {tpr:.3f}")
print(f"TNR (Specificity): {tnr:.3f}")
print(f"PPV (Precision): {ppv:.3f}")
```

### Per-Group Confusion Matrices

To assess fairness, compute confusion matrices **separately for each protected group**:

```python
import pandas as pd
import numpy as np

# Sample data
data = pd.DataFrame({
    'gender': ['M','M','M','M','M','M','M','M','M','M','M','M','M','M','M',
               'F','F','F','F','F','F','F','F','F','F'],
    'y_actual': [1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
                 1,1,1,0,0,0,0,0,0,0],
    'y_pred':   [1,1,1,1,1,1,1,1,0,0,0,1,1,0,0,
                 1,0,0,1,0,0,0,0,0,0]
})

for group in ['M', 'F']:
    subset = data[data['gender'] == group]
    tn, fp, fn, tp = confusion_matrix(
        subset['y_actual'], subset['y_pred']
    ).ravel()
    print(f"\n--- Gender = {group} ---")
    print(f"TP={tp}, FP={fp}, FN={fn}, TN={tn}")
    print(f"FNR: {fn/(fn+tp):.3f}")
    print(f"FPR: {fp/(fp+tn):.3f}")
```

---

*Next: [Fairness Metrics →](fairness-metrics.md)*
