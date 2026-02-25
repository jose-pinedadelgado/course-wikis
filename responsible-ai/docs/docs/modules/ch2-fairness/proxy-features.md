# Proxy Features

## Introduction

In the last chapter, the focus was on protected features for identifying bias and calculating fairness metrics. Independent features are ideally devoid of discriminatory information, but **bias can persist even after removing protected features** in complex problems.

It's a misconception to assume immunity from bias if protected features are not used — independent features can act as **proxy features** for protected ones.

### Relationship Between Proxy Features & Protected Features

Proxy features often relate to protected features due to **multicollinearity** — a strong linear relationship between multiple variables. Perfect multicollinearity occurs when the correlation between two independent variables is 1 or -1.

Common proxy features include:

- **Tax paid** → reflects income
- **Zip code** → indicates race/ethnicity
- **Shopping patterns** → suggests gender and marital status
- **Disposable income** → reveals gender and marital status
- **Salary and age** → uncovers gender and promotion trends

!!! warning
    Fairness efforts are futile if proxy features remain undetected — discrimination persists despite fairness definitions and measurements.

### Detecting Proxies

- Identifying proxy features is crucial to mitigating bias in datasets
- Business analysts and product owners, alongside data scientists, play pivotal roles
- Proxy features can emerge during feature engineering (e.g., disposable income may proxy gender)
- Even with protected features removed, engineered features can retain discriminatory information
- Proxies should be detected at various stages, including pre- and post-feature engineering

---

## Method 1: Linear Regression

When two features are related or correlated, a linear regression model can reveal their relationship.

$$
X_1 = a + \lambda \cdot X_2
$$

- $X_1$: The dependent variable (predicted feature)
- $a$: The intercept term
- $\lambda$: The coefficient representing the relationship
- $X_2$: The independent variable (potential proxy)

The standard error measures model fit:

$$
SE = \sqrt{1 - R^2_{adj}} \times \sigma_y
$$

!!! note
    When R² = 1 (two identical features), SE = 0, indicating a perfect fit and definite proxy.

### Python Code

```python
from sklearn.linear_model import LinearRegression
import statsmodels.api as sm

def test_proxy_correlation(feature, target='age'):
    print(f"Testing correlation between {feature} and {target}...\n")
    
    X_feat = X[[feature]]
    y_target = X[target]
    
    model = LinearRegression()
    model.fit(X_feat, y_target)

    X_const = sm.add_constant(X_feat)
    model_sm = sm.OLS(y_target, X_const).fit()

    print(model_sm.summary())
    print("\n" + "="*80 + "\n")

# Test each potential proxy
potential_proxies = ['Medu', 'Fedu', 'traveltime', 'studytime', 'absences']
for feature in potential_proxies:
    test_proxy_correlation(feature, target='age')
```

### Alternative: Using scipy.stats

```python
import numpy as np
from scipy import stats

proxy = np.array([
    [34, 34, 1156, 6.8, 8],
    [71, 71, 5041, 71.0, 8],
    [88, 88, 7744, 88.0, 8],
    [85, 85, 7225, 85.0, 11],
    [35, 35, 1225, 35.0, 9]
])

def adjusted_r2(r2, n, k):
    return 1 - ((1 - r2) * (n - 1) / (n - k - 1))

def standard_error(r2_adj, std_y):
    return np.sqrt(1 - r2_adj) * std_y

n = proxy.shape[0]
k = 1

for i in range(1, proxy.shape[1]):
    X = proxy[:, i]
    y = proxy[:, 0]
    res = stats.linregress(X, y)
    r2 = res.rvalue**2
    r2_adj = adjusted_r2(r2, n, k)
    std_y = np.std(y)
    se = standard_error(r2_adj, std_y)
    
    print(f"Testing Feature {i + 1} against Feature 1:")
    print(f"  R-squared: {r2:.6f}")
    print(f"  Adjusted R-squared: {r2_adj:.6f}")
    print(f"  Standard Error: {se:.6f}\n")
```

---

## Method 2: Variance Inflation Factor (VIF)

VIF is commonly used to identify multicollinearity. It uses the coefficient of determination (R²) to evaluate how features covariate.

$$
VIF = \frac{1}{1 - R^2}
$$

- **High VIF** → strong multicollinearity → likely proxy
- VIF serves as a more reliable metric than pairwise correlation when proxies involve multiple features

!!! warning "Division by Zero"
    When R² = 1 (perfect correlation), VIF = 1/(1-1) = 1/0, which is undefined. This indicates a perfect proxy.

### Python Code

```python
import pandas as pd
import numpy as np
from statsmodels.regression.linear_model import OLS
import statsmodels.api as sm

continuous_features = ['age', 'Medu', 'Fedu', 'traveltime', 'studytime', 'absences']
X_continuous = X[continuous_features]
protected_feature = 'age'

def calculate_proxy_vif(X, protected_feature):
    """
    Calculates VIF to detect if any feature is a proxy for the protected feature.
    """
    y = X[protected_feature]
    predictors = X.drop(columns=[protected_feature])

    vif_data = pd.DataFrame()
    vif_data['feature'] = predictors.columns

    vif_values = []
    for feature in predictors.columns:
        X_single = sm.add_constant(predictors[[feature]])
        model = OLS(y, X_single).fit()
        r_squared = model.rsquared
        vif = 1 / (1 - r_squared)
        vif_values.append(vif)

    vif_data['VIF'] = vif_values
    return vif_data

vif_results = calculate_proxy_vif(X_continuous, protected_feature)
print(vif_results)
```

### Alternative Implementation

```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score

def calculate_vif(df, target_feature):
    """Calculate VIF for each feature with respect to the target."""
    if not isinstance(df, pd.DataFrame):
        raise TypeError("Input should be a pandas DataFrame")
    if target_feature not in df.columns:
        raise ValueError(f"Target feature '{target_feature}' not found")

    y = df[target_feature]
    vif_data = []

    for feature in df.columns:
        if feature != target_feature:
            X = df[[feature]]
            model = LinearRegression()
            model.fit(X, y)
            y_pred = model.predict(X)
            r2 = r2_score(y, y_pred)
            vif = 1 / (1 - r2) if r2 < 1 else 'perfect correlation'
            vif_data.append({'Feature': feature, 'VIF': vif})

    return pd.DataFrame(vif_data)
```

---

## Method 3: Linear Association Using Variance

This method computes the linear association between variables using the square of the Pearson correlation coefficient to highlight stronger associations.

```python
import pandas as pd
import numpy as np

data = {
    'Feature 1': [34, 71, 88, 85, 35],
    'Feature 2': [34, 71, 88, 85, 35],
    'Feature 3': [1156, 5041, 7744, 7225, 1225],
    'Feature 4': [6.8, 71.0, 88.0, 85.0, 35.0],
    'Feature 5': [8, 8, 8, 11, 9]
}
proxy = pd.DataFrame(data)

for i in np.arange(1, 5):
    var1 = np.var(proxy['Feature 1'])
    var2 = np.var(proxy.iloc[:, i])
    cov = np.cov(proxy['Feature 1'], proxy.iloc[:, i])[0][1]
    asso = np.square(cov) / (var1 * var2)
    print(f'Association of Feature {i + 1} with Feature 1: {asso}')
```

---

## Method 4: Cosine Similarity/Distance

Linear methods may not capture non-linear relationships. Cosine similarity measures the cosine of the angle between two vectors to assess if they point in the same direction.

Key properties:

- Determines similarity based on **vector direction** rather than magnitude
- Useful for identifying similarity in multidimensional space
- Applicable to various data types, including categorical proxies
- Unlike correlation (which measures variability), cosine similarity assesses similarity in values

```python
from scipy import spatial
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

data = {
    'Feature 1': [34, 71, 88, 85, 35],
    'Feature 2': [34, 71, 88, 85, 35],
    'Feature 3': [1156, 5041, 7744, 7225, 1225],
    'Feature 4': [6.8, 71.0, 88.0, 85.0, 35.0],
    'Feature 5': [8, 8, 8, 11, 9]
}
proxy = pd.DataFrame(data)

for i in np.arange(1, 5):
    result = 1 - spatial.distance.cosine(proxy['Feature 1'], proxy.iloc[:,i])
    print('Cosine distance between Feature 1 and Feature {}:'.format(i+1), result)
    
    r = 1
    d = 10 * r * (1 - result)
    
    circle1 = plt.Circle((0, 0), r, alpha=.5)
    circle2 = plt.Circle((d, 0), r, alpha=.5)
    
    plt.ylim([-1.1, 1.1])
    plt.xlim([-1.1, 1.1 + d])
    fig = plt.gcf()
    fig.gca().add_artist(circle1)
    fig.gca().add_artist(circle2)
    plt.show()
```

---

## Method 5: Mutual Information

Mutual information measures the information gained from one random variable given another. It's a **generalized form of the correlation coefficient**, particularly useful for discrete features.

Key differences from covariance:

- Covariance focuses on the impact on the **product** of features
- Mutual information considers the impact on **distribution**
- Mutual information is **not limited to linear relationships**
- Makes it suitable for nonlinear tree-based algorithms

```python
import pandas as pd
from sklearn.metrics import mutual_info_score
import numpy as np

data = {
    'Cat 1': [1, 0, 1, 1, 0, 1],
    'Cat 2': [0, 1, 1, 0, 1, 0],
    'Cat 3': [1, 1, 0, 0, 1, 1],
    'Cat 4': [0, 0, 1, 1, 0, 1],
    'Cat 5': [1, 1, 1, 0, 0, 0],
    'Cat 6': [0, 1, 0, 1, 0, 1],
    'Cat 7': [1, 0, 1, 1, 0, 0],
    'Cat 8': [0, 1, 0, 1, 1, 1]
}
proxy = pd.DataFrame(data)

for i in np.arange(1, 8):
    mi = mutual_info_score(proxy['Cat 1'], proxy.iloc[:, i])
    print(f'Mutual Info between Cat 1 and {proxy.iloc[:, i].name}: {mi}')
```

!!! note
    Sample data is small (only a few records), so meaningful determinations require a representative dataset.

---

## Summary of Detection Methods

| Method | Best For | Limitation |
|--------|----------|------------|
| Linear Regression | Continuous features, initial screening | Computationally intensive with large feature spaces |
| VIF | Multicollinearity detection | Limited to linear relationships |
| Linear Association | Quick variance-based check | Only captures linear associations |
| Cosine Similarity | Non-linear relationships, categorical data | Requires careful interpretation |
| Mutual Information | Discrete features, non-linear relationships | Needs representative dataset size |
