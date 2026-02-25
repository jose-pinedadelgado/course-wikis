# Common Accuracy Metrics

The following metrics are derived from the confusion matrix. Each measures a different aspect of model performance.

## False Positive Rate (FPR)

The probability that a false alarm will be raised (predicting positive when the true value is negative).

$$FPR = \frac{FP}{FP + TN}$$

!!! note
    FP + TN represents ALL the actual negatives.

## False Negative Rate (FNR)

$$FNR = \frac{FN}{FN + TP}$$

!!! note
    FN + TP represents ALL the actual positives.

## True Positive Rate (TPR) — Sensitivity / Recall

$$TPR = \frac{TP}{TP + FN}$$

## True Negative Rate (TNR) — Specificity

The probability that an actual negative will be predicted as negative. Especially useful when negative outcomes have high cost.

$$TNR = \frac{TN}{TN + FP}$$

## Positive Predictive Value (PPV) — Precision

The fraction of positive predictions that are actually correct. This measures the classifier's ability **not to label as positive** a sample that **is negative**.

$$PPV = \frac{TP}{TP + FP}$$

## Accuracy

$$(TP + TN) / (TP + FN + FP + TN)$$

## F1 Score

$$F1 = \frac{2 \times Precision \times Recall}{Precision + Recall}$$

## Key Relationships

!!! info "Important"
    - **TPR + FNR = 1** (always)
    - **FPR + TNR = 1** (always)

These complementary relationships mean you can use either metric in each pair — they convey the same information from different perspectives.
