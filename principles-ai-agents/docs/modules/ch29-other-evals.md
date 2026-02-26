# Chapter 29 — Other Evals

## Beyond Text: Evaluating Agent Behavior

### Tool Use Evals

Did the agent:

- Call the **right tools** in the **right order**?
- Pass **correct parameters**?
- Handle **tool errors** gracefully?

### Workflow Evals

Did the workflow:

- Complete all steps?
- Produce valid intermediate results?
- Stay within time/cost budgets?

### Multi-Turn Evals

Over a conversation:

- Did the agent maintain context?
- Did it ask clarifying questions when needed?
- Did it avoid contradicting itself?

## Classification Evals

For routing/classification tasks, use standard ML metrics:

| Metric | Formula | Best For |
|--------|---------|----------|
| **Accuracy** | Correct / Total | Balanced classes |
| **Precision** | TP / (TP + FP) | Minimizing false positives |
| **Recall** | TP / (TP + FN) | Minimizing false negatives |
| **F1** | Harmonic mean of P & R | Imbalanced classes |

!!! note "Evals Are a Product"
    Building good evals is an ongoing investment, not a one-time setup. As your agent evolves, your evals must evolve too.

??? question "Discussion: Eval ROI"
    How do you decide how much to invest in evals vs. features? When are evals most valuable — early in development or late?
