# Data Drift

Data drift (also called **dataset shift**) occurs when the statistical properties of the model's input data change over time. The model was trained on one distribution but is now seeing a different one.

---

## Covariate Drift

Covariate drift occurs when the distribution of **input features** ($X$) changes, but the relationship between inputs and outputs ($P(Y|X)$) stays the same.

**Example:** A loan model trained mostly on applicants aged 30-50. Over time, more applicants aged 20-25 apply. The model hasn't seen much of this demographic → predictions may be unreliable.

<!-- TODO: Add detection methods, visualization, and Python code -->

---

## Jensen-Shannon Distance

The Jensen-Shannon (JS) distance measures the **similarity between two probability distributions**. It's a symmetric, bounded version of KL divergence.

$$
JSD(P \| Q) = \frac{1}{2} D_{KL}(P \| M) + \frac{1}{2} D_{KL}(Q \| M)
$$

Where $M = \frac{1}{2}(P + Q)$ is the average distribution.

**Properties:**

- Bounded: $0 \leq JSD \leq 1$ (when using log base 2)
- Symmetric: $JSD(P \| Q) = JSD(Q \| P)$
- $JSD = 0$ means identical distributions
- $JSD = 1$ means completely different distributions

<!-- TODO: Add Python implementation and interpretation thresholds -->

---

## Stability Index (PSI)

The Population Stability Index measures how much a variable's distribution has shifted between two time periods (e.g., training vs. production).

$$
PSI = \sum_{i=1}^{n} (P_i^{actual} - P_i^{expected}) \times \ln\left(\frac{P_i^{actual}}{P_i^{expected}}\right)
$$

### Interpreting PSI

| PSI Value | Interpretation |
|-----------|---------------|
| < 0.1 | No significant shift |
| 0.1 – 0.25 | Moderate shift — investigate |
| > 0.25 | Significant shift — action required |

<!-- TODO: Add Python implementation and visualization -->

---

## Summary

| Method | Measures | Best For |
|--------|----------|----------|
| Covariate Drift detection | Feature distribution changes | Catching demographic shifts |
| Jensen-Shannon Distance | Distribution similarity | Comparing any two distributions |
| Population Stability Index | Distribution shift over time | Monitoring production models |
