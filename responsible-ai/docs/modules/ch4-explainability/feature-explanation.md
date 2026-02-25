# Feature Explanation

Feature explanation techniques help us understand how individual features impact the model's output.

---

## Information Value Plots

**Information Value (IV)** measures the predictive power of a feature on a categorical target. It relies on **Weight of Evidence (WOE)**, which explores the proportion of positive and negative outcomes within each class or bin of a feature.

### Weight of Evidence (WOE)

$$
WOE = \ln\left(\frac{\text{Proportion of Positives}}{\text{Proportion of Negatives}}\right)
$$

The natural logarithm provides:

- **Symmetry** around zero (WOE = 0 means equal odds)
- **Scaling** that linearizes the relationship for logistic regression
- **Variance stabilization** across bins
- **Handling of extreme values** by reducing their impact

### Information Value (IV)

$$
IV = \sum (\text{Proportion of Positives} - \text{Proportion of Negatives}) \times WOE
$$

### How to Read IV Values

| IV Range | Predictive Power |
|----------|-----------------|
| < 0.02 | Weak |
| 0.02 – 0.1 | Weak |
| 0.1 – 0.3 | Medium |
| 0.3 – 0.5 | Strong |
| > 0.5 | Suspiciously strong |

### Python Code

```python
import pandas as pd
import numpy as np

def calculate_woe_iv(data, feature, target):
    df = data[[feature, target]].copy()
    df['bin'] = pd.qcut(df[feature], q=10, duplicates='drop')

    df_agg = df.groupby('bin')[target].agg(['count', 'sum'])
    df_agg.columns = ['total', 'bad']
    df_agg['good'] = df_agg['total'] - df_agg['bad']
    
    df_agg['bad_pct'] = df_agg['bad'] / df_agg['bad'].sum()
    df_agg['good_pct'] = df_agg['good'] / df_agg['good'].sum()
    df_agg['woe'] = np.log(df_agg['bad_pct'] / df_agg['good_pct'])
    df_agg['iv'] = (df_agg['bad_pct'] - df_agg['good_pct']) * df_agg['woe']
    
    iv = df_agg['iv'].sum()
    return df_agg[['woe', 'iv']], iv

def plot_woe_iv(data, feature, target):
    woe_iv, iv = calculate_woe_iv(data, feature, target)
    woe_iv[['woe']].plot(kind='bar', title=f'WOE plot for {feature} (IV = {iv:.2f})')
```

---

## Partial Dependency Plots (PDP)

PDPs show the relationship between a feature and the predicted outcome by averaging out all other features.

### How PDPs Work

1. **Select a feature** of interest
2. **Create a grid** of values for that feature
3. **Make predictions** for each grid value (other features held constant)
4. **Average predictions** across all instances

### Python Code

```python
from sklearn.inspection import PartialDependenceDisplay
from sklearn.inspection import partial_dependence

# Get individual prediction lines
pdp_lines = partial_dependence(
    rf, X, ["car_age"],
    percentiles=(0, 1),
    grid_resolution=100,
    kind='individual'
)
```

### Single Instance PDP

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 4))
plt.plot(pdp_lines['values'][0], pdp_lines['individual'][0][0],
         linewidth=2, color='black')
plt.plot(car_0['car_age'], y_pred[0], 'ro', markersize=5, label='Actual car_age')
plt.ylabel('Predicted price', size=12)
plt.xlabel('car_age', size=12)
plt.legend(loc='upper right')
```

<!-- TODO: Add PDP single instance image -->

### ICE Plot (Individual Conditional Expectation)

ICE plots display one line per instance, showing how each instance's prediction changes when a feature changes:

```python
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(8, 4))
PartialDependenceDisplay.from_estimator(
    rf, X, ['car_age'],
    kind='both',
    ice_lines_kw={"color": "black"},
    ax=ax,
    pd_line_kw={"color": "red", "lw": 3, 'linestyle': '--'}
)
```

<!-- TODO: Add ICE plot image -->

---

## Accumulated Local Effects (ALE)

<!-- TODO: Content to be added — ALE provides an alternative to PDP that accounts for feature correlations -->

---

## Sensitivity Analysis

<!-- TODO: Content to be added — Sensitivity analysis examines how changes in input features affect model output -->
