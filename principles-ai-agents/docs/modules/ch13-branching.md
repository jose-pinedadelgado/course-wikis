# Chapter 13 — Branching, Chaining, and Merging

## Workflow Patterns

### Chaining (Sequential)

Steps execute one after another. Output of step A becomes input to step B.

```
Input → Step A → Step B → Step C → Output
```

### Branching (Conditional)

Based on a condition, the workflow takes different paths:

```
Input → Check → [Path A] or [Path B] → Output
```

### Parallelization

Independent steps run **simultaneously**:

```
Input → [Step A, Step B, Step C] (parallel) → Merge → Output
```

### Merging

After parallel or branched paths, results are **combined** into a single output.

## Real-World Example: Support Ticket Routing

1. **Classify** the ticket (billing, technical, general)
2. **Branch** to the appropriate handler
3. Each handler runs its own sub-workflow
4. **Merge** results into a response template

!!! tip "Deterministic Where Possible"
    Use LLMs for classification and generation, but keep routing logic deterministic. Don't let the model decide control flow unless you have to.

??? question "Discussion: Parallel Processing"
    When is parallelization worth the complexity? What happens when one parallel branch fails?
