# Ch 8: Data & Model Privacy

This chapter covers techniques for protecting data and model privacy in AI systems.

---

## Basic Techniques

Foundational approaches to data privacy include:

- **Anonymization** — Removing personally identifiable information
- **Pseudonymization** — Replacing identifiers with artificial ones
- **Data masking** — Obscuring specific data within a dataset
- **Aggregation** — Reporting data at group level rather than individual level

<!-- TODO: Content to be expanded -->

---

## Differential Privacy

Differential privacy provides mathematical guarantees that individual data points cannot be identified from the output of a computation. It adds controlled noise to queries or data to protect individual privacy while maintaining statistical utility.

<!-- TODO: Content to be expanded with epsilon parameter, noise mechanisms, and examples -->

---

## Privacy Using Exponential Mechanisms

The exponential mechanism is a technique within differential privacy for selecting outputs from a set of possible options while preserving privacy. It's particularly useful when the output is not numerical.

<!-- TODO: Content to be expanded -->

---

## Differentially Private ML Algorithms

Applying differential privacy directly to machine learning training processes, such as:

- **DP-SGD** (Differentially Private Stochastic Gradient Descent)
- Gradient clipping and noise addition during training
- Privacy budget management across training epochs

<!-- TODO: Content to be expanded -->

---

## Federated Learning

Federated learning enables model training across multiple decentralized devices or servers without sharing raw data. Each device trains locally and only shares model updates (gradients), keeping data private.

Key benefits:

- Data never leaves the source device
- Reduced privacy risk compared to centralized training
- Applicable to mobile devices, hospitals, financial institutions

<!-- TODO: Content to be expanded with architecture diagrams and implementation details -->
