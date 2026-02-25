# Ch 8: Data and Model Privacy

This chapter covers techniques for protecting individual privacy in machine learning systems — from basic anonymization to advanced differential privacy and federated learning.

---

## Basic Techniques

Traditional privacy-preserving approaches before the era of differential privacy.

### Anonymization

Remove personally identifiable information (PII) — names, SSNs, addresses.

!!! warning "Anonymization is not enough"
    Research has shown that "anonymized" datasets can often be re-identified through **linkage attacks** (combining multiple datasets) or **inference attacks** (using known attributes to narrow down individuals).

### k-Anonymity

Each record is indistinguishable from at least $k-1$ other records on quasi-identifier attributes.

### l-Diversity

Each equivalence class (group of k-anonymous records) has at least $l$ distinct values for the sensitive attribute.

### t-Closeness

The distribution of sensitive attributes within each equivalence class is close to the overall distribution (within threshold $t$).

<!-- TODO: Add Python examples of anonymization techniques -->

---

## Differential Privacy

Differential privacy provides **mathematically provable** privacy guarantees. A mechanism $M$ satisfies $\epsilon$-differential privacy if:

$$
P[M(D_1) \in S] \leq e^{\epsilon} \cdot P[M(D_2) \in S]
$$

For all datasets $D_1$ and $D_2$ differing by a single record, and all possible outputs $S$.

### Key Concepts

- **$\epsilon$ (epsilon)** — the privacy budget. Smaller = more private, but noisier results
- **Sensitivity** — how much a single record can change the query result
- **Noise** — random values added to query results to mask individual contributions

### Laplace Mechanism

For numeric queries, add noise drawn from a Laplace distribution:

$$
M(D) = f(D) + \text{Laplace}\left(\frac{\Delta f}{\epsilon}\right)
$$

Where $\Delta f$ is the sensitivity of function $f$.

### Gaussian Mechanism

An alternative using Gaussian noise, providing $(\epsilon, \delta)$-differential privacy:

$$
M(D) = f(D) + \mathcal{N}\left(0, \frac{2 \ln(1.25/\delta) \cdot (\Delta f)^2}{\epsilon^2}\right)
$$

<!-- TODO: Add Python implementation of Laplace and Gaussian mechanisms -->

---

## Privacy Using Exponential Mechanisms

The exponential mechanism is used for **non-numeric queries** (categorical outputs) where adding noise directly doesn't make sense.

Instead of adding noise, it **selects an output** with probability proportional to its utility score:

$$
P[M(D) = r] \propto \exp\left(\frac{\epsilon \cdot u(D, r)}{2 \Delta u}\right)
$$

Where:
- $u(D, r)$ = utility score for output $r$ given dataset $D$
- $\Delta u$ = sensitivity of the utility function

Higher utility outputs are more likely to be selected, but randomness preserves privacy.

<!-- TODO: Add Python implementation and examples -->

---

## Differentially Private ML Algorithms

Applying differential privacy during model **training** rather than just query time.

### DP-SGD (Differentially Private Stochastic Gradient Descent)

1. **Clip gradients** — bound the influence of any single training example
2. **Add noise** — inject calibrated Gaussian noise to clipped gradients
3. **Track privacy budget** — use the moments accountant to monitor cumulative $\epsilon$

### Key Parameters

| Parameter | Effect |
|-----------|--------|
| **Clipping norm** | Limits per-example gradient influence |
| **Noise multiplier** | Controls privacy-utility tradeoff |
| **Batch size** | Larger batches = better privacy amplification |
| **Epochs** | More epochs = more privacy budget consumed |

<!-- TODO: Add DP-SGD implementation example -->

---

## Federated Learning

Federated learning trains models **without centralizing data**. Each participant trains locally and shares only model updates (gradients), not raw data.

### How It Works

1. **Server** distributes the current global model to participants
2. **Each participant** trains the model on their local data
3. **Participants** send model updates (not data) back to the server
4. **Server** aggregates updates (e.g., FedAvg) and updates the global model
5. Repeat until convergence

### Advantages

- Data never leaves the device
- Works across organizations with different privacy regulations
- Reduces data transfer bandwidth

### Challenges

- **Non-IID data** — participants may have very different data distributions
- **Communication overhead** — model updates can be large
- **Byzantine participants** — some may send malicious updates
- **Privacy leakage** — gradients can still reveal information (→ combine with differential privacy)

### Federated Learning + Differential Privacy

The gold standard: each participant applies DP-SGD locally, then shares differentially private gradients. This provides both:

- **Data locality** (federated learning)
- **Mathematical privacy guarantees** (differential privacy)

<!-- TODO: Add federated learning simulation code -->

---

## Summary

| Technique | Protects | Guarantee |
|-----------|----------|-----------|
| Anonymization | Direct identifiers | Weak (vulnerable to re-identification) |
| k-Anonymity | Quasi-identifiers | Moderate |
| Differential Privacy | Any query/model output | Strong (mathematical) |
| Exponential Mechanism | Non-numeric outputs | Strong (mathematical) |
| DP-SGD | Model training | Strong (mathematical) |
| Federated Learning | Raw data locality | Moderate (combine with DP for strong) |
