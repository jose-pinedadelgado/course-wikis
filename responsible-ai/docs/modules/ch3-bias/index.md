# Ch 3: Bias in Data

This chapter covers how to detect and measure bias in your dataset using Statistical Parity Difference (SPD) and Disparate Impact (DI).

---

## Statistical Parity Difference (SPD)

SPD is **the difference** between the probability that a random individual from the disadvantaged group is labelled positive and the probability that a random individual from the advantaged group is labelled positive.

$$
SPD = P(Y=1 \mid S = S_a) - P(Y=1 \mid S = S_d) = 0
$$

- A bigger value indicates a higher level of disadvantage
- A small difference is called "statistical parity"
- A fair scenario: if 10% of males get a loan, we expect roughly 10% of females to also get a loan

!!! tip "Threshold"
    What's an acceptable SPD depends on your feature. A common threshold is **0.1** for statistical parity.

---

## Disparate Impact (DI)

Disparate Impact measures adverse effects on the disadvantaged group. It's a **ratio** between the probability of favorable outcomes for each group.

The California FEPC uses an **80% rule**:

$$
\frac{P(Y=1 \mid S = S_d)}{P(Y=1 \mid S = S_a)} \geq 0.8
$$

A higher percentage (close to 1) represents a more fair outcome.

Or equivalently, looking at how much advantage the advantaged group has:

$$
\frac{P(Y=1 \mid S = S_a)}{P(Y=1 \mid S = S_d)} \leq 1.25
$$

A value closer to 1 is preferred. The optimal DI is 1.

---

## Calculating SPD & DI

We can calculate both by looking at predictions and counting favorable outcomes for each class.

- **SPD** = difference in positive prediction rates
- **DI** = ratio of positive prediction rates

```python
def statistical_parity_test(data_frame, protected_group, Sa_label, Sd_label, Y, fav_label):
    """
    Calculate Statistical Parity Difference and Disparate Impact.
    
    Parameters
    ----------
    data_frame : DataFrame with Y and S
    protected_group : Column name of protected feature
    Sa_label : label for advantageous group
    Sd_label : label for disadvantageous group
    Y : the target label column
    fav_label : label for favorable outcome
    
    Returns
    -------
    statistical_parity, disparate_impact
    """
    Sa = data_frame[data_frame[protected_group] == Sa_label]
    Fav_Sa = Sa[Sa[Y] == fav_label]
    Fav_Sa_count = len(Fav_Sa)
    
    Sd = data_frame[data_frame[protected_group] == Sd_label]
    Fav_Sd = Sd[Sd[Y] == fav_label]
    Fav_Sd_count = len(Fav_Sd)
    
    Advantageous = len(Sa)
    dis_Advantageous = len(Sd)
    
    statistical_parity = (Fav_Sd_count / dis_Advantageous) - (Fav_Sa_count / Advantageous)
    disparate_impact = (Fav_Sd_count / dis_Advantageous) / (Fav_Sa_count / Advantageous)
    
    return statistical_parity, disparate_impact
```

---

## Visualizing SPD & DI

A combined bar chart with dual Y-axes showing both metrics with threshold lines:

```python
import matplotlib.pyplot as plt
import numpy as np

categories = [
    'Male', 'HigherEd', 'AgeGroup', 'HouseOwner', 'Mortgage', 
    'Entrepreneur', 'Tenant', 'Estonian', 'English', 'NoOfDependantsLessThan3', 
    'Married', 'Single', 'Divorced', 'WorkExLess10', 'WorkExLess5'
]
stat_parity_diff = [0.05, 0.10, 0.15, 0.12, 0.18, 0.08, 0.10, 0.15, 0.09, 0.14, 
                    0.10, 0.11, 0.16, 0.07, 0.08]
disparate_impact = [1.1, 1.2, 1.25, 1.15, 1.3, 1.1, 1.05, 1.2, 1.1, 1.3, 
                    1.25, 1.15, 1.35, 1.2, 1.25]

bar_width = 0.35
x = np.arange(len(categories))

fig, ax1 = plt.subplots(figsize=(12, 6))

# Statistical Parity Difference (Red Bars)
ax1.bar(x - bar_width/2, stat_parity_diff, bar_width, color='red', label='Stat Parity Diff')

# Disparate Impact (Black Bars) on second Y-axis
ax2 = ax1.twinx()
ax2.bar(x + bar_width/2, disparate_impact, bar_width, color='black', label='Disparate Impact')

ax1.set_xticks(x)
ax1.set_xticklabels(categories, rotation=45, ha='right')
ax1.set_xlabel('Sensitive / Protected Features')
ax1.set_ylabel('Statistical Parity Difference', color='red')
ax2.set_ylabel('Disparate Impact', color='black')
plt.title('Statistical Parity Difference and Disparate Impact')

# Threshold lines
ax1.axhline(y=0.1, color='red', linestyle='--', linewidth=1)
ax2.axhline(y=0.8, color='black', linestyle='--', linewidth=1)
ax2.axhline(y=1.2, color='black', linestyle='--', linewidth=1)

fig.legend(loc='upper right', bbox_to_anchor=(0.9, 0.9))
fig.tight_layout()
plt.show()
```

<!-- TODO: Add SPD & DI visualization image -->

---

## Special Cases

### When Y is Continuous & Feature (S) is Binary

<!-- TODO: Content to be added — this scenario requires different statistical tests -->

### When Y is Binary & S is Continuous

<!-- TODO: Content to be added — this scenario requires binning or different approaches -->
