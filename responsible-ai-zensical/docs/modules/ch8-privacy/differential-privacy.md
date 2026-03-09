# Differential Privacy

## Core Concept

Differential privacy guarantees that the **output of a query or model doesn't change significantly** whether or not any single individual's data is included.

$$P[\mathcal{M}(D) \in S] \leq e^\epsilon \times P[\mathcal{M}(D') \in S]$$

Where:

- $\mathcal{M}$ is the mechanism (query, model)
- $D$ and $D'$ differ by one record
- $\epsilon$ is the **privacy budget** (lower = more private)
- $S$ is any set of possible outputs

!!! info "Privacy Budget ($\epsilon$)"
    - $\epsilon = 0$: Perfect privacy (useless output)
    - $\epsilon < 1$: Strong privacy
    - $\epsilon = 1-3$: Moderate privacy (common in practice)
    - $\epsilon > 10$: Weak privacy

## Mechanisms

### Laplace Mechanism

Add noise drawn from a Laplace distribution to query results:

$$\mathcal{M}(D) = f(D) + \text{Laplace}\left(\frac{\Delta f}{\epsilon}\right)$$

Where $\Delta f$ is the **sensitivity** — the maximum change in $f$ from adding/removing one record.

```python
import numpy as np

def laplace_mechanism(true_value, sensitivity, epsilon):
    """Apply Laplace mechanism for differential privacy."""
    scale = sensitivity / epsilon
    noise = np.random.laplace(0, scale)
    return true_value + noise

# Example: private mean income
true_mean = df['income'].mean()
# Sensitivity = (max - min) / n
sensitivity = (df['income'].max() - df['income'].min()) / len(df)
epsilon = 1.0

private_mean = laplace_mechanism(true_mean, sensitivity, epsilon)
print(f"True mean: ${true_mean:,.2f}")
print(f"Private mean (ε={epsilon}): ${private_mean:,.2f}")
```

### Exponential Mechanism

For non-numeric outputs (e.g., selecting the best category), the exponential mechanism selects outputs with probability proportional to their utility:

$$P[\mathcal{M}(D) = r] \propto \exp\left(\frac{\epsilon \cdot u(D, r)}{2\Delta u}\right)$$

```python
def exponential_mechanism(utilities, epsilon, sensitivity):
    """Select output using exponential mechanism."""
    scores = np.array(utilities)
    probabilities = np.exp((epsilon * scores) / (2 * sensitivity))
    probabilities /= probabilities.sum()
    return np.random.choice(len(utilities), p=probabilities)
```

### Gaussian Mechanism

For $(\epsilon, \delta)$-differential privacy:

$$\mathcal{M}(D) = f(D) + \mathcal{N}\left(0, \frac{2 \ln(1.25/\delta) \cdot \Delta f^2}{\epsilon^2}\right)$$

```python
def gaussian_mechanism(true_value, sensitivity, epsilon, delta=1e-5):
    """Apply Gaussian mechanism for (ε,δ)-differential privacy."""
    sigma = np.sqrt(2 * np.log(1.25 / delta)) * sensitivity / epsilon
    noise = np.random.normal(0, sigma)
    return true_value + noise
```

## Differentially Private ML

### Private Stochastic Gradient Descent (DP-SGD)

Train models with differential privacy by clipping gradients and adding noise:

```python
# Using opacus (PyTorch)
from opacus import PrivacyEngine

model = YourModel()
optimizer = torch.optim.SGD(model.parameters(), lr=0.1)

privacy_engine = PrivacyEngine()
model, optimizer, train_loader = privacy_engine.make_private(
    module=model,
    optimizer=optimizer,
    data_loader=train_loader,
    noise_multiplier=1.0,
    max_grad_norm=1.0,
)

# Training loop as usual — privacy is handled automatically
```

## Privacy-Fairness Connection

!!! tip "Privacy Helps Fairness"
    Adding differential privacy to protected features **reduces bias** by limiting how much the model can learn from sensitive attributes. It's a win-win: better privacy *and* better fairness.

---

*Next: [Federated Learning →](federated-learning.md)*
