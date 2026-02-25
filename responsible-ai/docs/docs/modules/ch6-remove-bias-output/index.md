# Ch 6: Remove Bias from ML Output

This chapter covers post-processing techniques to remove bias from model predictions after the model has been trained.

---

## Reject Option Classifier (ROC)

The Reject Option Classifier works by modifying predictions that fall within an **uncertainty region** near the decision boundary. In this region, the classifier is least confident, and adjustments can be made to favor the disadvantaged group.

### How It Works

1. Identify predictions near the decision boundary (low confidence)
2. For uncertain predictions, flip the outcome to favor the disadvantaged group
3. This post-processing approach doesn't require retraining the model

---

## Optimizing the ROC

<!-- TODO: Content to be expanded — Techniques for finding the optimal rejection threshold -->

---

## Handling Multiple Features in ROC

<!-- TODO: Content to be expanded — Extending ROC to work with multiple protected features simultaneously -->
