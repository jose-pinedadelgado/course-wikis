# Bias Metrics & Visualization

## Complete Bias Assessment Pipeline

This page provides a complete, runnable pipeline for assessing bias across all protected features in a dataset.

### Step 1: Load and Prepare Data

```python
import pandas as pd
import numpy as np

# Load your dataset
df = pd.read_csv('loan_data.csv')

# Define protected features and their privileged values
protected_config = {
    'gender': 'Male',
    'education': 'University',
    'age_group': '30-50',
    'home_ownership': 'Owner',
    'employment_status': 'Employed',
    'marital_status': 'Married',
    'dependants': 'less_than_3',
}

target_col = 'default'
favourable_value = 0  # non-defaulter
```

### Step 2: Compute All Metrics

```python
def full_bias_assessment(df, protected_config, target_col, fav_val):
    """Complete bias assessment across all protected features."""
    results = []
    
    for feature, priv_val in protected_config.items():
        priv_mask = df[feature] == priv_val
        
        p_priv = (df.loc[priv_mask, target_col] == fav_val).mean()
        p_unpriv = (df.loc[~priv_mask, target_col] == fav_val).mean()
        
        spd = p_unpriv - p_priv
        di = p_unpriv / p_priv if p_priv > 0 else np.inf
        
        n_priv = priv_mask.sum()
        n_unpriv = (~priv_mask).sum()
        
        results.append({
            'Feature': feature,
            'Privileged': priv_val,
            'N_Priv': n_priv,
            'N_Unpriv': n_unpriv,
            'P_Fav_Priv': p_priv,
            'P_Fav_Unpriv': p_unpriv,
            'SPD': spd,
            'DI': di,
            'Bias?': '⚠️' if abs(spd) > 0.05 or di < 0.8 else '✅'
        })
    
    return pd.DataFrame(results)

results = full_bias_assessment(df, protected_config, target_col, favourable_value)
print(results.to_string(index=False))
```

### Step 3: Visualize

```python
import matplotlib.pyplot as plt
import seaborn as sns

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# SPD bar chart
colors = ['red' if x < -0.05 else 'green' if abs(x) < 0.05 else 'orange' 
          for x in results['SPD']]
axes[0].barh(results['Feature'], results['SPD'], color=colors)
axes[0].axvline(x=0, color='black', linestyle='--')
axes[0].set_title('Statistical Parity Difference')

# DI bar chart
colors = ['red' if x < 0.8 else 'green' for x in results['DI']]
axes[1].barh(results['Feature'], results['DI'], color=colors)
axes[1].axvline(x=0.8, color='red', linestyle='--', label='80% threshold')
axes[1].axvline(x=1.0, color='green', linestyle='--', label='Parity')
axes[1].set_title('Disparate Impact Ratio')
axes[1].legend()

plt.tight_layout()
plt.show()
```

## Intersectional Analysis

!!! tip "Don't Forget Intersections"
    Bias may be hidden when looking at features individually. Check **intersections** of protected features (e.g., gender × race).

```python
def intersectional_bias(df, feat1, val1, feat2, val2, target_col, fav_val):
    """Check bias at the intersection of two protected features."""
    # Four groups
    groups = {
        f'{feat1}={val1}, {feat2}={val2}': (df[feat1]==val1) & (df[feat2]==val2),
        f'{feat1}={val1}, {feat2}≠{val2}': (df[feat1]==val1) & (df[feat2]!=val2),
        f'{feat1}≠{val1}, {feat2}={val2}': (df[feat1]!=val1) & (df[feat2]==val2),
        f'{feat1}≠{val1}, {feat2}≠{val2}': (df[feat1]!=val1) & (df[feat2]!=val2),
    }
    
    for name, mask in groups.items():
        n = mask.sum()
        p_fav = (df.loc[mask, target_col] == fav_val).mean()
        print(f"{name}: n={n}, P(fav)={p_fav:.4f}")
```

---

*Back to: [Chapter 3 Overview ←](index.md)*
