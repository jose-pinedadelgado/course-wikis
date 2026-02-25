# Ch 5: Remove Bias from ML Model

This chapter covers techniques for removing bias at the model level, primarily through **reweighting** — assigning weights to data based on protected features to reduce discrimination without altering the data itself.

## Topics

- [Reweighting](reweighting.md) — Calculating and implementing weights to correct bias
- [Advanced Techniques](advanced.md) — Calibrating decision boundaries, composite features, and Additive Counterfactual Fairness

## Key Properties of Reweighting

1. Handles **one protected feature** at a time
2. **Only works for classification** problems
3. Can create composite features to handle multiple features together
4. Maintains accuracy with minimal impact

## How It Works

Weights are based on frequency counts comparing the **expected probability** (assuming independence between protected feature and outcome) with the **observed probability** in the data.

$$
SPD = P(S=S_a \mid Y=Y^+) - P(S=S_d \mid Y=Y^+)
$$

If SPD ≈ 0, bias is unlikely. If significant, we apply weights:

$$
SPD_{corrected} = P(S=S_a \mid Y=Y^+) \times W_{S_aFav} - P(S=S_d \mid Y=Y^+) \times W_{S_dFav}
$$

We need 4 weights — one for each combination:

| Weight | Protected Class | Outcome |
|--------|----------------|---------|
| $W_{S_aY^+}$ | Advantaged | Favorable |
| $W_{S_aY^-}$ | Advantaged | Unfavorable |
| $W_{S_dY^+}$ | Disadvantaged | Favorable |
| $W_{S_dY^-}$ | Disadvantaged | Unfavorable |
