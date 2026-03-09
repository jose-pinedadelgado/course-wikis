# Ch 2: Fairness and Proxy Features

> Humans have an innate sense of fairness — studies show even 3-year-old children consider merit when sharing rewards. But fairness doesn't always translate into our algorithms.

## Chapter Overview

This chapter introduces the foundational tools for measuring and achieving fairness in ML:

- **[Confusion Matrix & Accuracy Metrics](confusion-matrix.md)** — The building blocks
- **[Fairness Metrics](fairness-metrics.md)** — Independence, separation, sufficiency, and 6+ metrics
- **[Proxy Features](proxy-features.md)** — Detecting hidden bias through correlated features

## Key Concepts

### Favourable vs Unfavourable Outcomes

| Notation | Meaning |
|----------|---------|
| $\hat{Y}_{fav}$ | Predicted favourable outcome |
| $\hat{Y}_{unfav}$ | Predicted unfavourable outcome |
| $Y_{fav}$ | Actual favourable outcome |
| $Y_{unfav}$ | Actual unfavourable outcome |

!!! example "Loan Default Prediction"
    - **Favourable**: Applicant predicted as non-defaulter (approved)
    - **Unfavourable**: Applicant predicted as defaulter (rejected)

### Protected Features & Privileged Classes

Features are divided into:

- **Independent features ($X$)** — No personal, racial, or socio-economic indicators
- **Protected features ($S$)** — May contain information that leads to discrimination

For a given protected feature, classes are:

- **Privileged class ($S_a$)** — More likely to receive favourable outcomes
- **Unprivileged class ($S_d$)** — Less likely to receive favourable outcomes

!!! tip "Determining Privileged Classes"
    Always determine privileged/unprivileged classes **from the data**, not from assumptions. Plot heat maps of frequency for each protected feature by target outcome. A privileged class in one problem may be unprivileged in another.

### Widely Recognized Protected Features

- Race (Civil Rights Act of 1964)
- Sex including gender, pregnancy, sexual orientation (Equal Pay Act of 1963)
- Religion or creed
- National origin or ancestry
- Age (Age Discrimination in Employment Act of 1967)
- Citizenship (Immigration Reform and Control Act)
- Physical or mental disability status (Rehabilitation Act of 1973)
- Veteran status
- Genetic information
- Familial status

---

*Next: [Confusion Matrix & Accuracy Metrics →](confusion-matrix.md)*
