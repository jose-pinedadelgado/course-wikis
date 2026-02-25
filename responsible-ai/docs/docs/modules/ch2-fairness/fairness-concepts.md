# Fairness Concepts & Metrics

## Fairness Concepts

There are three fundamental notions of fairness:

### Independence

A model satisfies independence if the prediction $\hat{Y}$ is statistically independent of the protected attribute $S$.

### Separation

A model satisfies separation if the prediction $\hat{Y}$ is independent of the protected attribute $S$, **conditional on the actual outcome** $Y$.

### Sufficiency

A model is fair if the prediction $\hat{Y}$ **does not** systematically over- or under-predict $Y$ for members of any group.

!!! info "Further Reading"
    [An Introduction to Fairness in Machine Learning](https://medium.com/analytics-vidhya/an-introduction-to-fairness-in-machine-learning-62ef827e0020)

---

## Fairness Metrics

To calculate fairness metrics, we need a **confusion matrix for each subclass** of a protected feature. Then we compare results across subgroups.

For all metrics below, we start by creating separate confusion matrices:

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

---

### 1. Equal Opportunity

**True Positive Rate (TPR) should be equal across different subgroups of a protected feature.**

$$
P(\hat{Y} = 0 \mid Y = 1, S = S_a) = P(\hat{Y} = 0 \mid Y = 1, S = S_d)
$$

$$
\text{Equal Opportunity} \implies TPR_{\text{Group A}} = TPR_{\text{Group B}}
$$

Where:

$$
TPR = \frac{TP}{TP + FN}
$$

Since TPR and FNR are complements (TPR + FNR = 1), we can equivalently use FNR:

$$
FNR = \frac{FN}{TP + FN}
$$

**Goal:** Ensure that qualified instances from different subgroups that should be categorized as positive are actually categorized as positive at the same rate.

```python
# Equal Opportunity — compare FNR (or TPR) across groups
FNR_adv = fn_adv / (fn_adv + tp_adv)
FNR_disadv = fn_disadv / (fn_disadv + tp_disadv)

EOpp_diff = abs(FNR_disadv - FNR_adv)
```

---

### 2. Predictive Equality

**Both privileged and unprivileged groups have equal FPR.**

$$
P(\hat{Y} = 1 \mid Y = 0, S = S_a) = P(\hat{Y} = 1 \mid Y = 0, S = S_d)
$$

Equal FPR means the chance of an actual defaulter being wrongly predicted as a non-defaulter should be identical across all subsets within a protected class.

$$
FPR = \frac{FP}{FP + TN}
$$

```python
# Predictive Equality — compare FPR across groups
FPR_adv = fp_adv / (fp_adv + tn_adv)
FPR_disadv = fp_disadv / (fp_disadv + tn_disadv)

pred_eq_diff = abs(FPR_disadv - FPR_adv)
```

---

### 3. Equalized Odds

**Also known as disparate mistreatment. Requires both privileged and unprivileged groups to have equal TPR AND FPR.**

$$
P(\hat{Y} = 1 \mid Y = i, S = S_a) = P(\hat{Y} = 1 \mid Y = i, S = S_d), \quad i \in \{0, 1\}
$$

A classifier meets this criterion if it ensures both advantageous and disadvantageous groups have identical TPR and FPR. The core principle is that applicants, regardless of their protected class status, should receive similar classification outcomes based on their actual creditworthiness.

```python
# Equalized Odds — compare TPR + FPR across groups
TPR_adv = tp_adv / (tp_adv + fn_adv)
TPR_disadv = tp_disadv / (tp_disadv + fn_disadv)

FPR_adv = fp_adv / (fp_adv + tn_adv)
FPR_disadv = fp_disadv / (fp_disadv + tn_disadv)

EOdds_diff = abs((TPR_disadv - TPR_adv) + (FPR_disadv - FPR_adv))
```

---

### 4. Predictive Parity

**Also known as outcome test. Requires both groups to have equal Precision (PPV).**

$$
P(Y = 1 \mid \hat{Y} = 1, S = S_a) = P(Y = 1 \mid \hat{Y} = 1, S = S_d)
$$

Given a positive prediction, the probability of the actual outcome being positive should be the same for both protected groups. A model with perfect precision (1 for all groups) is deemed fair.

!!! warning
    Achieving predictive parity doesn't necessarily imply bias reduction or elimination.

```python
# Predictive Parity — compare Precision/PPV across groups
prec_adv = tp_adv / (tp_adv + fp_adv)
prec_disadv = tp_disadv / (tp_disadv + fp_disadv)

prec_diff = abs(prec_disadv - prec_adv)
```

---

### 5. Demographic Parity

**Membership in a protected class should have no correlation with being predicted a favorable outcome.**

$$
P(\hat{Y} = 1, S = S_a) = P(\hat{Y} = 1, S = S_d)
$$

We use the **Predicted Positive Rate (PPR)** as our point of comparison.

Achieving demographic parity requires adjustments in confusion matrices:

- **Privileged group:** Reduce false positives, increase true negatives
- **Unprivileged group:** Decrease false negatives, increase true positives

```python
# Demographic Parity — ratio of favorable predictions to total instances
demo_parity_adv = (tp_adv + fp_adv) / (tn_adv + fp_adv + fn_adv + tp_adv)
demo_parity_disadv = (tp_disadv + fp_disadv) / (tn_disadv + fp_disadv + fn_disadv + tp_disadv)

demo_parity_diff = abs(demo_parity_disadv - demo_parity_adv)
```

---

### 6. Average Odds Difference

**Average of difference in FPR and TPR for advantageous and disadvantageous groups.**

$$
AOD = \frac{1}{2} \left[ (FPR_{S_d} - FPR_{S_a}) + (TPR_{S_d} - TPR_{S_a}) \right]
$$

This metric incorporates both **predictive equality** (FPR) and **equal opportunity** (TPR). A lower or zero difference indicates equal benefits for both groups.

```python
FPR_disadv = fp_disadv / (fp_disadv + tn_disadv)
FPR_adv = fp_adv / (fp_adv + tn_adv)

TPR_disadv = tp_disadv / (tp_disadv + fn_disadv)
TPR_adv = tp_adv / (tp_adv + fn_adv)

# Average Odds Difference
AOD = 0.5 * ((FPR_disadv - FPR_adv) + (TPR_disadv - TPR_adv))
```

---

## Summary

| Metric | What It Compares | Ideal Value |
|--------|-----------------|-------------|
| Equal Opportunity | TPR across groups | 0 difference |
| Predictive Equality | FPR across groups | 0 difference |
| Equalized Odds | TPR + FPR across groups | 0 difference |
| Predictive Parity | Precision across groups | 0 difference |
| Demographic Parity | Predicted Positive Rate | 0 difference |
| Average Odds Difference | Avg of FPR + TPR diff | 0 |

Each metric focuses on a different aspect of fairness. The choice depends on the specific use case and business context.

For each metric, the approach is consistent:

1. **Describe** the concept/theory
2. **Calculate** the formula
3. **Program** it in Python
4. **Interpret** the results
