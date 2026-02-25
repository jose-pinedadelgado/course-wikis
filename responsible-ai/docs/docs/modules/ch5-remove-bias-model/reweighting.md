# Reweighting the Data

## Step 1: Build a Frequency Table

Consider a protected feature (marriage) and target (loan default):

|  | Default (0) | Not Default (1) |
|---|---|---|
| Not Married ($S_a$ = 0) | 42,709 | 12,316 |
| Married ($S_d$ = 1) | 4,043 | 2,253 |

From this we need:

- Total observations: $n = 61{,}321$
- Total disadvantaged: $S_d = 6{,}296$
- Total favorable: $Y_{fav} = 46{,}752$
- Disadvantaged + favorable: $S_d \cap Y_{fav} = 4{,}043$

## Step 2: Calculate Observed Probability

The observed probability is the number of times a combination appears divided by total observations:

$$
P(\text{Observed}_{S_dY_{fav}}) = \frac{S_d \cap Y_{fav}}{n} = \frac{4{,}043}{61{,}321} = 0.0659 \quad (6.59\%)
$$

## Step 3: Calculate Expected Probability

Assuming independence between protected feature and target, the expected probability is the product of individual probabilities:

$$
P(\text{Expected}_{S_dY_{fav}}) = \frac{Y_{fav}}{n} \times \frac{S_d}{n} = \frac{46{,}752}{61{,}321} \times \frac{6{,}296}{61{,}321} = 0.0782 \quad (7.82\%)
$$

The gap between expected (7.82%) and observed (6.59%) suggests discrimination.

## Step 4: Calculate the Weight

$$
Weight = \frac{\text{Expected Probability}}{\text{Observed Probability}}
$$

$$
W_{S_dY_{fav}} = \frac{0.0782}{0.0659} = 1.1047
$$

All four weights for our example:

| Weight | Value |
|--------|-------|
| $W_{S_aY_{fav}}$ | 0.98227 |
| $W_{S_aY_{unfav}}$ | 1.18728 |
| $W_{S_dY_{fav}}$ | 1.06148 |
| $W_{S_dY_{unfav}}$ | 0.66393 |

---

## Implementing in Python

### Setup: Count instances

```python
dummy = np.repeat(1, len(data))
data['dummy'] = dummy
n = np.sum(data['dummy'])

sa = np.sum(data['dummy'][data[choice]==pval])            # Total privileged
sd = np.sum(data['dummy'][data[choice]==upval])           # Total unprivileged
ypos = np.sum(data['dummy'][data[target_feature]==fav])   # Total favorable
yneg = np.sum(data['dummy'][data[target_feature]==unfav]) # Total unfavorable

print(f"""Total Advantageous: {sa}, Total Disadvantageous: {sd}, 
Total Favourable: {ypos}, Total Unfavourable: {yneg}""")
```

### Create subsets for each combination

```python
data_sa_ypos = data[(data[choice]==pval) & (data[target_feature]==fav)]
data_sa_yneg = data[(data[choice]==pval) & (data[target_feature]==unfav)]
data_sd_ypos = data[(data[choice]==upval) & (data[target_feature]==fav)]
data_sd_yneg = data[(data[choice]==upval) & (data[target_feature]==unfav)]
```

### Count each combination

```python
sa_ypos = np.sum(data_sa_ypos['dummy'])
sa_yneg = np.sum(data_sa_yneg['dummy'])
sd_ypos = np.sum(data_sd_ypos['dummy'])
sd_yneg = np.sum(data_sd_yneg['dummy'])

print(f"Advantaged + Favorable: {sa_ypos}")
print(f"Advantaged + Unfavorable: {sa_yneg}")
print(f"Disadvantaged + Favorable: {sd_ypos}")
print(f"Disadvantaged + Unfavorable: {sd_yneg}")
```

### Calculate weights

```python
w_sa_ypos = (ypos*sa) / (n*sa_ypos)
w_sa_yneg = (yneg*sa) / (n*sa_yneg)
w_sd_ypos = (ypos*sd) / (n*sd_ypos)
w_sd_yneg = (yneg*sd) / (n*sd_yneg)

print(f"W(Sa, Y+): {w_sa_ypos}")
print(f"W(Sa, Y-): {w_sa_yneg}")
print(f"W(Sd, Y+): {w_sd_ypos}")
print(f"W(Sd, Y-): {w_sd_yneg}")
```

### Verify bias reduction

```python
DiscriminationBefore = (sa_ypos/sa) - (sd_ypos/sd)
DiscriminationAfter = (sa_ypos/sa * w_sa_ypos) - (sd_ypos/sd * w_sd_ypos)

print(f"Discrimination Before: {abs(DiscriminationBefore)}")
print(f"Discrimination After: {abs(DiscriminationAfter)}")
```

### Apply weights to dataset

```python
datatest = data.copy()
datatest['Weights'] = np.repeat(999, len(data))

datatest.loc[(datatest[choice]==pval) & (datatest[target_feature]==fav), 'Weights'] = w_sa_ypos
datatest.loc[(datatest[choice]==pval) & (datatest[target_feature]==unfav), 'Weights'] = w_sa_yneg
datatest.loc[(datatest[choice]==upval) & (datatest[target_feature]==fav), 'Weights'] = w_sd_ypos
datatest.loc[(datatest[choice]==upval) & (datatest[target_feature]==unfav), 'Weights'] = w_sd_yneg

datatest['Weights'].head()
```
