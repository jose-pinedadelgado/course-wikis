# Concept Drift

Concept drift occurs when the **relationship between inputs and outputs** changes over time — i.e., $P(Y|X)$ shifts, even if the input distribution stays the same.

**Example:** A fraud model trained on pre-pandemic data. During the pandemic, legitimate transaction patterns changed drastically, making the model's learned patterns obsolete.

---

## Kolmogorov-Smirnov (KS) Test

The KS test compares two cumulative distribution functions (CDFs) to determine if they come from the same distribution.

$$
D = \max_x |F_1(x) - F_2(x)|
$$

Where $F_1$ and $F_2$ are the empirical CDFs of two samples.

- **Null hypothesis:** Both samples come from the same distribution
- **If p-value < α** → reject null → distributions differ → drift detected

<!-- TODO: Add Python implementation with scipy.stats.ks_2samp -->

---

## Brier Score

The Brier Score measures the **accuracy of probabilistic predictions**. It's the mean squared difference between predicted probabilities and actual outcomes.

$$
BS = \frac{1}{N} \sum_{i=1}^{N} (p_i - o_i)^2
$$

Where:
- $p_i$ = predicted probability of event
- $o_i$ = actual outcome (0 or 1)

- **BS = 0** → perfect calibration
- **BS = 1** → worst possible calibration
- Track Brier Score over time; a rising score indicates concept drift

<!-- TODO: Add Python implementation and monitoring approach -->

---

## Page-Hinkley Test (PHT)

A sequential analysis technique for detecting **abrupt changes** in the average of a sequence.

The test monitors the cumulative sum of deviations from the running mean:

$$
m_T = \sum_{t=1}^{T} (x_t - \bar{x}_T - \delta)
$$

$$
M_T = \max(m_1, m_2, ..., m_T)
$$

Drift is detected when: $M_T - m_T > \lambda$

Where:
- $\delta$ = minimum magnitude of change to detect
- $\lambda$ = detection threshold

<!-- TODO: Add Python implementation -->

---

## Early Drift Detection Method (EDDM)

EDDM improves on DDM (Drift Detection Method) by monitoring the **distance between classification errors** rather than just the error rate. It's particularly effective at detecting **gradual drift**.

<!-- TODO: Add EDDM algorithm steps and Python implementation -->

---

## Hierarchical Linear Four Rate (HLFR)

HLFR monitors four rates from the confusion matrix (TPR, TNR, FPR, FNR) hierarchically to detect concept drift while accounting for class imbalance.

<!-- TODO: Add HLFR methodology and Python implementation -->

---

## Summary

| Method | Detects | Type of Drift |
|--------|---------|---------------|
| KS Test | Distribution changes | Any |
| Brier Score | Calibration degradation | Gradual |
| Page-Hinkley Test | Mean shifts | Abrupt |
| EDDM | Error pattern changes | Gradual |
| HLFR | Confusion matrix rate changes | Both |
