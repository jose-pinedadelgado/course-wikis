# Ch 7: Accountability in AI

Accountability ensures that AI systems remain **reliable and fair over time** — not just at deployment. This chapter covers techniques for detecting when your model's environment or performance has changed.

## Topics

- [Data Drift](data-drift.md) — When the input data distribution changes
- [Concept Drift](concept-drift.md) — When the relationship between inputs and outputs changes

## Why Accountability Matters

A model that was fair at deployment can become unfair as:

- **Demographics shift** — the population changes over time
- **Behavior changes** — user patterns evolve
- **External factors** — economic conditions, policy changes, etc.

Monitoring for drift is essential to maintaining fairness guarantees.
