# Ch 2: Fairness & Proxy Features

This chapter introduces the foundational tools for measuring fairness in ML models: the confusion matrix, accuracy metrics, fairness concepts, and proxy feature detection.

## Topics

- [Fairness Concepts & Metrics](fairness-concepts.md) — Independence, separation, sufficiency, and 6 fairness metrics with Python code
- [Proxy Features](proxy-features.md) — Detecting hidden bias through proxy feature analysis

---

## Confusion Matrix

The confusion matrix is the foundation for all fairness metrics. It breaks down predictions into four categories:

|  | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

### Simple Confusion Matrix in Python

```python
from sklearn.metrics import confusion_matrix

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

cm = confusion_matrix(y_actual, y_pred)
print("Confusion Matrix:")
print(cm)
```

Output:
```
[[3 1]
 [2 2]]
```

### Extracting Individual Values

```python
from sklearn.metrics import confusion_matrix

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

tn, fp, fn, tp = confusion_matrix(y_actual, y_pred).ravel()

print(f"True Negative: {tn}")
print(f"False Positive: {fp}")
print(f"False Negative: {fn}")
print(f"True Positive: {tp}")
```

### Confusion Matrices for Protected Groups

Since we need separate confusion matrices for advantaged and disadvantaged groups:

```python
tn_adv, fp_adv, fn_adv, tp_adv = confusion_matrix(
    y_actual[X_test[protected_group_name] == adv_val],
    y_pred_binary[X_test[protected_group_name] == adv_val]
).ravel()

tn_disadv, fp_disadv, fn_disadv, tp_disadv = confusion_matrix(
    y_actual[X_test[protected_group_name] == disadv_val],
    y_pred_binary[X_test[protected_group_name] == disadv_val]
).ravel()
```

### Visualizing the Confusion Matrix

```python
import xgboost as xgb
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

cm = confusion_matrix(y_actual, y_pred)

disp = ConfusionMatrixDisplay(confusion_matrix=cm)
disp.plot(cmap=plt.cm.Blues)
plt.title("Confusion Matrix for Validation Data")
plt.show()
```

<!-- TODO: Add confusion matrix visualization image -->

---

## Common Accuracy Metrics

### False Positive Rate (FPR)

Probability that a false alarm will be raised (predicting positive when true value is negative).

$$
FPR = \frac{FP}{FP + TN}
$$

!!! note
    FP + TN represents ALL the actual negatives.

### False Negative Rate (FNR)

$$
FNR = \frac{FN}{FN + TP}
$$

!!! note
    FN + TP represents ALL the actual positives.

### True Positive Rate (TPR) — Sensitivity / Recall

$$
TPR = \frac{TP}{TP + FN}
$$

### True Negative Rate (TNR) — Specificity

Probability that an actual negative will test negative. Especially useful when negative outcomes are high cost.

$$
TNR = \frac{TN}{TN + FP}
$$

### Positive Predictive Value (PPV) — Precision

Fraction of positive cases correctly predicted out of all predicted positives. This is the classifier's ability **not to label as positive** a sample that **is negative**.

$$
PPV = \frac{TP}{TP + FP}
$$

### Accuracy

$$
Accuracy = \frac{TP + TN}{TP + FN + FP + TN}
$$

### F1 Score

$$
F1 = \frac{2 \times Precision \times Recall}{Precision + Recall}
$$

!!! tip "Key Relationships"
    - **TPR + FNR = 1** (always)
    - **FPR + TNR = 1** (always)
