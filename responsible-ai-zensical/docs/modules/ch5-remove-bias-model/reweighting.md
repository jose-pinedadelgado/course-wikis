# Reweighting the Data

## How It Works

Reweighting assigns **sample weights** based on the intersection of protected feature values and outcomes. The weights counter-balance discrimination without changing any data.

### Weight Formula

For a dataset $D$ with protected feature $S$ (binary: $S_a$ advantaged, $S_d$ disadvantaged) and outcome $Y$ (binary: $Y^+$ favourable, $Y^-$ unfavourable):

$$W(S, Y) = \frac{P_{expected}(S) \times P_{expected}(Y)}{P_{observed}(S, Y)}$$

This gives four weights — one for each combination:

| Group | Outcome | Weight Direction |
|-------|---------|-----------------|
| $S_d$ (disadvantaged) | $Y^+$ (favourable) | ⬆️ **Increased** |
| $S_a$ (advantaged) | $Y^-$ (unfavourable) | ⬆️ **Increased** |
| $S_d$ (disadvantaged) | $Y^-$ (unfavourable) | ⬇️ **Decreased** |
| $S_a$ (advantaged) | $Y^+$ (favourable) | ⬇️ **Decreased** |

!!! info "Intuition"
    We **up-weight** disadvantaged people who deserve favourable outcomes (they were historically under-served) and **down-weight** advantaged people who unfairly received favourable outcomes.

### Python Implementation

```python
import numpy as np
import pandas as pd

def compute_reweighting(df, protected_col, privileged_val, target_col, fav_val):
    """Compute sample weights for bias mitigation via reweighting."""
    n = len(df)
    
    # Marginal probabilities
    p_priv = (df[protected_col] == privileged_val).mean()
    p_unpriv = 1 - p_priv
    p_fav = (df[target_col] == fav_val).mean()
    p_unfav = 1 - p_fav
    
    # Joint probabilities (observed)
    p_priv_fav = ((df[protected_col] == privileged_val) & (df[target_col] == fav_val)).mean()
    p_priv_unfav = ((df[protected_col] == privileged_val) & (df[target_col] != fav_val)).mean()
    p_unpriv_fav = ((df[protected_col] != privileged_val) & (df[target_col] == fav_val)).mean()
    p_unpriv_unfav = ((df[protected_col] != privileged_val) & (df[target_col] != fav_val)).mean()
    
    # Expected (under independence)
    weights = {
        (True, True):   (p_priv * p_fav) / p_priv_fav,      # priv + fav
        (True, False):  (p_priv * p_unfav) / p_priv_unfav,   # priv + unfav
        (False, True):  (p_unpriv * p_fav) / p_unpriv_fav,   # unpriv + fav
        (False, False): (p_unpriv * p_unfav) / p_unpriv_unfav # unpriv + unfav
    }
    
    # Assign weights
    df['weight'] = df.apply(
        lambda row: weights[(
            row[protected_col] == privileged_val,
            row[target_col] == fav_val
        )], axis=1
    )
    
    return df

# Usage
df = compute_reweighting(df, 'gender', 'Male', 'default', 0)

# Train with weights
from sklearn.ensemble import GradientBoostingClassifier
model = GradientBoostingClassifier()
model.fit(X_train, y_train, sample_weight=df.loc[X_train.index, 'weight'])
```

### Verifying Discrimination Reduction

After reweighting, the **Statistical Parity Difference** should approach 0:

```python
def verify_reweighting(df, protected_col, privileged_val, target_col, fav_val):
    """Check SPD before and after reweighting."""
    priv = df[df[protected_col] == privileged_val]
    unpriv = df[df[protected_col] != privileged_val]
    
    # Unweighted SPD
    spd_before = (
        (unpriv[target_col] == fav_val).mean() - 
        (priv[target_col] == fav_val).mean()
    )
    
    # Weighted SPD
    p_fav_priv_w = np.average(
        priv[target_col] == fav_val, weights=priv['weight']
    )
    p_fav_unpriv_w = np.average(
        unpriv[target_col] == fav_val, weights=unpriv['weight']
    )
    spd_after = p_fav_unpriv_w - p_fav_priv_w
    
    print(f"SPD before reweighting: {spd_before:.4f}")
    print(f"SPD after reweighting:  {spd_after:.4f}")
```

## Composite Features

Reweighting handles **one protected feature at a time**. To handle multiple features simultaneously, create a **composite feature**:

```python
# Create composite of gender and marital status
df['gender_marital'] = df['gender'] + '_' + df['marital_status']

# Now reweight on the composite
df = compute_reweighting(df, 'gender_marital', 'Male_Married', 'default', 0)
```

!!! warning "Composite Feature Explosion"
    Combining too many features creates many small groups with unreliable weight estimates. Stick to 2–3 features per composite.

---

*Next: [Advanced Techniques →](advanced.md)*
