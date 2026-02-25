# Reweighting the Data

## Overview

When we detect bias via Statistical Parity Difference, we can use **weights** to mitigate discrimination. The corrected formula with weights:

$$
P(S=S_a \mid Y=Y^+) \times W_{S_aFav} - P(S=S_d \mid Y=Y^+) \times W_{S_dFav}
$$

We need 4 weights — one for each combination of protected feature class and outcome:

| Weight | Description |
|--------|-------------|
| $W_{S_aY_{fav}}$ | Advantageous class + favorable outcome |
| $W_{S_aY_{unfav}}$ | Advantageous class + unfavorable outcome |
| $W_{S_dY_{fav}}$ | Disadvantageous class + favorable outcome |
| $W_{S_dY_{unfav}}$ | Disadvantageous class + unfavorable outcome |

---

## Step 1: Frequency Table

Build a frequency count for each combination. Example (Marriage as protected feature, Default as target):

|  | Default (0) | Not Default (1) |
|---|---|---|
| Not Married ($S_a$ = 0) | 42,709 | 12,316 |
| Married ($S_d$ = 1) | 4,043 | 2,253 |

From this table:

- Total observations ($n$) = 61,321
- Total unprivileged ($S_d$) = 6,296
- Total favorable ($Y_{fav}$) = 46,752
- Unprivileged + favorable ($S_d \cap Y_{fav}$) = 4,043

---

## Step 2: Observed Probability

The observed probability of a combination = count of that combination / total observations.

$$
P(Observed_{S_dY_{fav}}) = \frac{S_d \cap Y_{fav}}{n} = \frac{4043}{61321} = 0.0659
$$

So 6.59% of instances are disadvantaged AND favorable.

---

## Step 3: Expected Probability

If the protected feature is truly independent of the target, we can compute the expected probability as:

$$
P(Expected_{S_dY_{fav}}) = \frac{Y_{fav}}{n} \times \frac{S_d}{n} = \frac{46752}{61321} \times \frac{6296}{61321} = 0.0728
$$

The difference between expected (7.28%) and observed (6.59%) suggests discrimination.

---

## Step 4: Calculate the Weight

The weight is the **ratio of expected to observed probability**:

$$
Weight = \frac{P(Expected)}{P(Observed)}
$$

$$
W_{S_dY_{fav}} = \frac{0.0728}{0.0659} = 1.1047
$$

All four weights:

| Weight | Value |
|--------|-------|
| $W_{S_aY_{fav}}$ | 0.98227 |
| $W_{S_aY_{unfav}}$ | 1.18728 |
| $W_{S_dY_{fav}}$ | 1.06148 |
| $W_{S_dY_{unfav}}$ | 0.66393 |

---

## Implementation in Python

### Setting Up Counts

```python
import numpy as np

dummy = np.repeat(1, len(data))
data['dummy'] = dummy
n = np.sum(data['dummy'])  # Total instances

sa = np.sum(data['dummy'][data[choice] == pval])            # Privileged count
sd = np.sum(data['dummy'][data[choice] == upval])           # Unprivileged count
ypos = np.sum(data['dummy'][data[target_feature] == fav])   # Favorable count
yneg = np.sum(data['dummy'][data[target_feature] == unfav]) # Unfavorable count

print(f"""Total Advantageous: {sa}, Total Disadvantageous: {sd}, 
Total Favourable: {ypos}, Total Unfavourable: {yneg}""")
```

### Creating Subsets

```python
data_sa_ypos = data[(data[choice] == pval) & (data[target_feature] == fav)]
data_sa_yneg = data[(data[choice] == pval) & (data[target_feature] == unfav)]
data_sd_ypos = data[(data[choice] == upval) & (data[target_feature] == fav)]
data_sd_yneg = data[(data[choice] == upval) & (data[target_feature] == unfav)]

sa_ypos = np.sum(data_sa_ypos['dummy'])
sa_yneg = np.sum(data_sa_yneg['dummy'])
sd_ypos = np.sum(data_sd_ypos['dummy'])
sd_yneg = np.sum(data_sd_yneg['dummy'])
```

### Calculating Weights

```python
w_sa_ypos = (ypos * sa) / (n * sa_ypos)  # Privileged + favorable
w_sa_yneg = (yneg * sa) / (n * sa_yneg)  # Privileged + unfavorable
w_sd_ypos = (ypos * sd) / (n * sd_ypos)  # Unprivileged + favorable
w_sd_yneg = (yneg * sd) / (n * sd_yneg)  # Unprivileged + unfavorable

print(f"Weight Advantaged + Favorable: {w_sa_ypos}")
print(f"Weight Advantaged + Unfavorable: {w_sa_yneg}")
print(f"Weight Disadvantaged + Favorable: {w_sd_ypos}")
print(f"Weight Disadvantaged + Unfavorable: {w_sd_yneg}")
```

### Verifying Bias Reduction

```python
DiscriminationBefore = (sa_ypos / sa) - (sd_ypos / sd)
DiscriminationAfter = (sa_ypos / sa * w_sa_ypos) - (sd_ypos / sd * w_sd_ypos)

print(f"Discrimination Before: {abs(DiscriminationBefore)}")
print(f"Discrimination After: {abs(DiscriminationAfter)}")
```

### Applying Weights to the Dataset

```python
datatest = data.copy()
datatest['Weights'] = np.repeat(999, len(data))  # Placeholder

datatest.loc[(datatest[choice] == pval) & (datatest[target_feature] == fav), 'Weights'] = w_sa_ypos
datatest.loc[(datatest[choice] == pval) & (datatest[target_feature] == unfav), 'Weights'] = w_sa_yneg
datatest.loc[(datatest[choice] == upval) & (datatest[target_feature] == fav), 'Weights'] = w_sd_ypos
datatest.loc[(datatest[choice] == upval) & (datatest[target_feature] == unfav), 'Weights'] = w_sd_yneg

datatest['Weights'].head()
```

!!! tip
    After applying weights, the Statistical Parity Test should be much closer to 0, indicating reduced bias while maintaining prediction accuracy.
