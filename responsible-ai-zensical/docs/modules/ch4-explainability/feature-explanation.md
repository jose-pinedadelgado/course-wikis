# Feature Explanation

Feature explanation helps us understand **how individual features impact the output** — critical for feature selection, bias detection, and explaining decisions.

## Information Value (IV) Plots

IV Plots combine **Weight of Evidence (WoE)** and **Information Value (IV)** to quantify a feature's predictive power.

### Weight of Evidence (WoE)

$$WoE_i = \ln\left(\frac{\text{Distribution of Events}_i}{\text{Distribution of Non-Events}_i}\right)$$

Where events = favourable outcomes, non-events = unfavourable outcomes, and $i$ represents a bin of the feature.

!!! info "WoE Interpretation"
    - **WoE > 0**: Bin has more events than expected → favourable
    - **WoE < 0**: Bin has fewer events than expected → unfavourable  
    - **WoE = 0**: No predictive power in this bin

### Information Value (IV)

$$IV = \sum_{i=1}^{n} (D_{events_i} - D_{nonevents_i}) \times WoE_i$$

| IV Range | Predictive Power |
|----------|-----------------|
| < 0.02 | Not useful |
| 0.02 – 0.1 | Weak |
| 0.1 – 0.3 | Medium |
| 0.3 – 0.5 | Strong |
| > 0.5 | Suspicious (too good — check for overfitting) |

```python
import numpy as np
import pandas as pd

def compute_woe_iv(df, feature, target, bins=10):
    """Compute WoE and IV for a feature."""
    # Bin continuous features
    if df[feature].nunique() > bins:
        df['bin'] = pd.qcut(df[feature], q=bins, duplicates='drop')
    else:
        df['bin'] = df[feature]
    
    grouped = df.groupby('bin')[target].agg(['sum', 'count'])
    grouped.columns = ['events', 'total']
    grouped['non_events'] = grouped['total'] - grouped['events']
    
    # Distribution
    total_events = grouped['events'].sum()
    total_non_events = grouped['non_events'].sum()
    
    grouped['dist_events'] = grouped['events'] / total_events
    grouped['dist_non_events'] = grouped['non_events'] / total_non_events
    
    # WoE and IV
    grouped['woe'] = np.log(
        grouped['dist_events'].clip(0.0001) / 
        grouped['dist_non_events'].clip(0.0001)
    )
    grouped['iv'] = (grouped['dist_events'] - grouped['dist_non_events']) * grouped['woe']
    
    iv_total = grouped['iv'].sum()
    df.drop('bin', axis=1, inplace=True)
    
    return grouped, iv_total
```

## Partial Dependence Plots (PDP)

PDPs show the **marginal effect** of a feature on the predicted outcome, averaging over all other features.

$$\hat{f}_{PDP}(x_s) = \frac{1}{n}\sum_{i=1}^{n} \hat{f}(x_s, x_{c}^{(i)})$$

Where $x_s$ is the feature of interest and $x_c$ are all other features.

```python
from sklearn.inspection import PartialDependenceDisplay
import matplotlib.pyplot as plt

# After training a model
PartialDependenceDisplay.from_estimator(
    model, X_test, features=[0, 1, 2],
    kind='average', grid_resolution=50
)
plt.tight_layout()
plt.show()
```

## SHAP Values

SHAP (SHapley Additive exPlanations) assigns each feature an importance value based on game theory — how much does each feature contribute to moving the prediction from the baseline?

$$\phi_i = \sum_{S \subseteq F \setminus \{i\}} \frac{|S|!(|F|-|S|-1)!}{|F|!} [f(S \cup \{i\}) - f(S)]$$

```python
import shap

# Train your model first
explainer = shap.Explainer(model, X_train)
shap_values = explainer(X_test)

# Summary plot
shap.summary_plot(shap_values, X_test)

# Individual prediction
shap.waterfall_plot(shap_values[0])
```

!!! tip "SHAP for Fairness"
    Compare SHAP values across protected groups. If a protected feature (or its proxy) has high SHAP importance, your model may be learning bias.

---

*Next: [Model Explanation →](model-explanation.md)*
