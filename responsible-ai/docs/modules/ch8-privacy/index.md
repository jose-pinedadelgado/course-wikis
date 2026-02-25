# Ch 8: Data & Model Privacy

This chapter covers techniques for protecting data and model privacy in AI systems.

---

## Basic Techniques

<!-- TODO: Content to be added — Overview of basic privacy-preserving techniques -->

---

## Differential Privacy

Differential privacy provides a mathematical framework for quantifying the privacy loss when releasing information about a dataset. It ensures that the output of an analysis does not reveal whether any individual's data was included.

<!-- TODO: Add formal definition and examples -->

---

## Privacy Using Exponential Mechanisms

The exponential mechanism is a technique for achieving differential privacy when the output is not numerical (e.g., selecting the "best" option from a set of candidates).

<!-- TODO: Add detailed explanation and examples -->

---

## Differentially Private ML Algorithms

These are machine learning algorithms specifically designed to satisfy differential privacy guarantees during training.

<!-- TODO: Add examples of DP-SGD and other private learning algorithms -->

---

## Federated Learning

Federated learning is a distributed machine learning approach where the model is trained across multiple devices or servers holding local data, **without exchanging the raw data**.

Key benefits:

- Data stays on the device — never centralized
- Only model updates (gradients) are shared
- Reduces privacy risk while enabling collaborative learning

<!-- TODO: Add architecture diagram and implementation details -->
