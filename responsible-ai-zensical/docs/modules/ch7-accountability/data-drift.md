# Data Drift

## What Is Data Drift?

Data drift occurs when the **statistical distribution of features or predictions** changes over time compared to the training data.

| Drift Type | What Shifts | Notation |
|-----------|------------|----------|
| **Feature drift** | Distribution of input features | $P(X)$ changes |
| **Prior probability drift** | Distribution of target variable | $P(Y)$ changes |

!!! example
    A credit model trained pre-COVID sees very different income distributions, employment patterns, and default rates post-COVID. The model's predictions become unreliable.

## Detection Methods

### Population Stability Index (PSI)

PSI measures how much a distribution has shifted between two time periods:

$$PSI = \sum_{i=1}^{n} (A_i - E_i) \times \ln\left(\frac{A_i}{E_i}\right)$$

Where $A_i$ = actual proportion in bin $i$, $E_i$ = expected proportion in bin $i$.

| PSI Value | Interpretation |
|-----------|---------------|
| < 0.1 | No significant change |
| 0.1 – 0.2 | Moderate shift — investigate |
| > 0.2 | Significant shift — action required |

```python
import numpy as np

def population_stability_index(expected, actual, bins=10):
    """Compute PSI between two distributions."""
    # Bin the expected distribution
    breakpoints = np.percentile(expected, np.linspace(0, 100, bins + 1))
    breakpoints[0] = -np.inf
    breakpoints[-1] = np.inf
    
    expected_counts = np.histogram(expected, bins=breakpoints)[0]
    actual_counts = np.histogram(actual, bins=breakpoints)[0]
    
    # Convert to proportions (avoid zero)
    expected_pct = np.clip(expected_counts / len(expected), 0.0001, None)
    actual_pct = np.clip(actual_counts / len(actual), 0.0001, None)
    
    psi = np.sum((actual_pct - expected_pct) * np.log(actual_pct / expected_pct))
    return psi

# Usage
psi = population_stability_index(X_train['income'], X_live['income'])
print(f"PSI for income: {psi:.4f}")
```

### Kolmogorov-Smirnov (KS) Test

The KS test measures the maximum distance between two cumulative distribution functions:

```python
from scipy.stats import ks_2samp

def detect_drift_ks(train_data, live_data, features, alpha=0.05):
    """Detect drift using KS test for each feature."""
    results = []
    for feature in features:
        stat, p_value = ks_2samp(train_data[feature], live_data[feature])
        results.append({
            'Feature': feature,
            'KS Statistic': stat,
            'p-value': p_value,
            'Drift?': '⚠️' if p_value < alpha else '✅'
        })
    return pd.DataFrame(results).sort_values('p-value')
```

### Chi-Square Test (for Categorical Features)

```python
from scipy.stats import chi2_contingency
import pandas as pd

def detect_drift_categorical(train_data, live_data, feature):
    """Detect drift in categorical features using chi-square test."""
    train_counts = train_data[feature].value_counts()
    live_counts = live_data[feature].value_counts()
    
    # Align categories
    all_cats = set(train_counts.index) | set(live_counts.index)
    contingency = pd.DataFrame({
        'train': [train_counts.get(c, 0) for c in all_cats],
        'live': [live_counts.get(c, 0) for c in all_cats]
    }, index=list(all_cats))
    
    chi2, p_value, dof, expected = chi2_contingency(contingency.T)
    print(f"Chi-square: {chi2:.4f}, p-value: {p_value:.6f}")
    return p_value
```

## Monitoring Dashboard

```python
import matplotlib.pyplot as plt

def drift_dashboard(train_data, live_data, features, top_n=10):
    """Visual drift dashboard."""
    psi_scores = {}
    for feat in features:
        psi_scores[feat] = population_stability_index(
            train_data[feat].values, live_data[feat].values
        )
    
    # Sort by PSI
    sorted_feats = sorted(psi_scores.items(), key=lambda x: x[1], reverse=True)[:top_n]
    
    feats, scores = zip(*sorted_feats)
    colors = ['red' if s > 0.2 else 'orange' if s > 0.1 else 'green' for s in scores]
    
    plt.figure(figsize=(10, 6))
    plt.barh(feats, scores, color=colors)
    plt.axvline(x=0.1, color='orange', linestyle='--', label='Moderate')
    plt.axvline(x=0.2, color='red', linestyle='--', label='Significant')
    plt.xlabel('PSI')
    plt.title('Feature Drift Dashboard')
    plt.legend()
    plt.tight_layout()
    plt.show()
```

---

*Next: [Concept Drift →](concept-drift.md)*
