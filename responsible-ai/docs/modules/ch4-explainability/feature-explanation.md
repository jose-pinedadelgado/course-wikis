# Feature Explanation

Feature explanation techniques help us understand how individual features impact the model's output.

---

## Information Value Plots

Information Value (IV) measures the predictive power of a feature on a categorical target. It relies on **Weight of Evidence (WOE)**, which explores the proportion of positive and negative outcomes within each bin or class of a feature.

### Weight of Evidence (WOE)

$$
WOE = \ln\left(\frac{\text{Proportion of Positives}}{\text{Proportion of Negatives}}\right)
$$

### Information Value (IV)

$$
IV = \sum (\text{Proportion of Positives} - \text{Proportion of Negatives}) \times WOE
$$

### How to Interpret IV

| IV Range | Predictive Power |
|----------|-----------------|
| < 0.02 | Very weak |
| 0.02 – 0.1 | Weak |
| 0.1 – 0.3 | Medium |
| 0.3 – 0.5 | Strong |
| > 0.5 | Suspiciously strong |

### Why Use Natural Logarithm?

1. **Symmetry** — WOE of 0 means equal odds; positive = event more likely; negative = less likely
2. **Stabilizing variance** — Bins with extreme odds ratios don't disproportionately affect the model
3. **Handling extremes** — Reduces impact of outliers
4. **Intuitive interpretation** — Magnitude indicates strength of evidence

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
3. **Make predictions** at each grid point (other features held fixed)
4. **Average predictions** across all instances

### Python Code

```python
from sklearn.inspection import PartialDependenceDisplay
from sklearn.inspection import partial_dependence

# Get individual prediction lines
pdp_lines = partial_dependence(rf, X, ["car_age"], 
                               percentiles=(0,1), 
                               grid_resolution=100,
                               kind='individual')
```

### Single Instance PDP

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(8,4))
plt.plot(pdp_lines['values'][0], pdp_lines['individual'][0][0], 
         linewidth=2, color='black')
plt.plot(car_0['car_age'], y_pred[0], 'ro', markersize=5, label='Actual car_age')
plt.ylabel('Predicted price', size=12)
plt.xlabel('car_age', size=12)
plt.legend(loc='upper right')
plt.show()
```

### ICE Plot (Individual Conditional Expectation)

An ICE plot displays one line per instance, showing how each instance's prediction changes when a feature changes:

```python
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(8,4))
PartialDependenceDisplay.from_estimator(
    rf, X, ['car_age'], 
    kind='both',
    ice_lines_kw={"color": "black"},
    ax=ax,
    pd_line_kw={"color": "red", "lw": 3, 'linestyle': '--'}
)
plt.show()
```

<!-- TODO: Add PDP and ICE plot images -->

---

## Accumulated Local Effects (ALE)

<!-- TODO: Content to be expanded — ALE plots address limitations of PDPs when features are correlated -->

---

## Sensitivity Analysis

<!-- TODO: Content to be expanded — Sensitivity analysis measures how changes in input features affect model output -->
