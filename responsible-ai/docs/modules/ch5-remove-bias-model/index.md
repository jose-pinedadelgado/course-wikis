# Ch 5: Remove Bias from ML Model

This chapter covers techniques to remove bias **during model training** — specifically by reweighting the data to correct for discriminatory patterns.

## Key Concepts

**Reweighting** assigns weights to data based on a protected feature. These weights are used in the loss function optimization. By reweighting, we do **not** need to alter the data to achieve a reduction in bias.

Key features of reweighting:

1. Handles **1 protected feature** at a time
2. **Only works for Classification** problems
3. Can create **composite features** to handle multiple features together
4. Maintains accuracy with minimal impact

## Topics

- [Reweighting the Data](reweighting.md) — Calculating and implementing weights
- [Advanced Techniques](advanced.md) — Calibrating decision boundary, composite features, and additive counterfactual fairness
