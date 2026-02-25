# Ch 6: Remove Bias from ML Output

This chapter covers post-processing techniques that adjust model **outputs** (rather than the model itself) to reduce bias.

## Reject Option Classifier (ROC)

The Reject Option Classifier modifies predictions in the **uncertainty region** — where the model is least confident — to favor the disadvantaged group.

### How It Works

1. Train a standard classifier
2. Identify the **critical region** — predictions close to the decision boundary (low confidence)
3. Within the critical region:
   - If the instance belongs to the **disadvantaged group** → flip prediction to **favorable**
   - If the instance belongs to the **advantaged group** → flip prediction to **unfavorable**
4. Outside the critical region, keep the original prediction

### Key Parameters

- **Critical region threshold** — How close to the boundary counts as "uncertain"
- Wider threshold → more predictions are flipped → stronger fairness correction but lower accuracy
- Narrower threshold → fewer flips → less fairness impact but preserves accuracy

<!-- TODO: Add Python implementation code -->

---

## Optimizing the ROC

Finding the optimal critical region threshold requires balancing fairness metrics against accuracy.

<!-- TODO: Add optimization approach, code, and visualizations -->

---

## Handling Multiple Features in ROC

When multiple protected features exist (e.g., race AND gender), the ROC must account for intersectional bias.

<!-- TODO: Add multi-feature ROC approach and implementation -->

---

## Summary

| Approach | When to Use |
|----------|-------------|
| Reject Option Classifier | When you can't retrain the model but need fairer outputs |
| Optimized ROC | When you need to balance fairness-accuracy tradeoff precisely |
| Multi-feature ROC | When multiple protected attributes intersect |

!!! note "Pre-processing vs. Post-processing"
    Ch 5 (reweighting) modifies the **training data**. Ch 6 (ROC) modifies the **predictions**. Both can be combined for stronger bias mitigation.
