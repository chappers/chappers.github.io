---
layout: post
category: code dump
tags: [python]
tagline:
---

Some notes on Bayesian Optimization using Matern Kernel as per NIPS Practical Bayesian Optimization paper .

```python
# see http://scikit-learn.org/stable/auto_examples/gaussian_process/plot_gpr_prior_posterior.html

import numpy as np

import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import numpy as np
%matplotlib inline

from sklearn.gaussian_process import GaussianProcessRegressor
from sklearn.gaussian_process.kernels import Matern

kernel = Matern(nu=2.5)
gp = GaussianProcessRegressor(kernel=kernel)
```

```python
# suppose we are fitting this function..

X = np.linspace(0, 5, 100)
y = np.sin((X - 2.5) ** 2)
plt.plot(X, y)
```

![png](/img/BayesOpt_files/BayesOpt_1_1.png)

```python
# Generate data and fit GP
rng = np.random.RandomState(4)
# take 10 points...
X = rng.uniform(0, 5, 10)[:, np.newaxis]
y = np.sin((X[:, 0] - 2.5) ** 2)
gp.fit(X, y)

X_ = np.linspace(0, 5, 100)
y_mean, y_std = gp.predict(X_[:, np.newaxis], return_std=True)
```

```python
plt.plot(X_, np.sin((X_ - 2.5) ** 2))
plt.plot(X, y, 'ro')
plt.grid(True)
plt.show()
```

![png](/img/BayesOpt_files/BayesOpt_3_0.png)

```python
# how should we approach this? One curve?
y_samples = gp.sample_y(X_[:, np.newaxis], 1)
plt.plot(X_, y_samples, lw=1)
plt.grid(True)
plt.plot(X, y, 'ro')

```

![png](/img/BayesOpt_files/BayesOpt_4_1.png)

```python
# how should we approach this? 10 curves?
y_samples = gp.sample_y(X_[:, np.newaxis], 10)
plt.plot(X_, y_samples, lw=1)
plt.grid(True)
plt.plot(X, y, 'ro')

```

![png](/img/BayesOpt_files/BayesOpt_5_1.png)

```python
# how should we approach this? 100 curves?
y_samples = gp.sample_y(X_[:, np.newaxis], 100)
plt.plot(X_, y_samples, lw=1)
plt.grid(True)
plt.plot(X, y, 'ro')

```

![png](/img/BayesOpt_files/BayesOpt_6_1.png)

```python
# we can show what it looks like 1 standard deviation away
plt.plot(X_, y_mean, 'b', X, y, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - y_std, y_mean + y_std,
                 alpha=0.5, color='k')
```

![png](/img/BayesOpt_files/BayesOpt_7_1.png)

```python
# how should we approach this? 100 curves?
y_samples = gp.sample_y(X_[:, np.newaxis], 5)
n_std = 1.20
plt.plot(X_, y_samples, lw=1)
plt.plot(X, y, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - n_std*y_std, y_mean + n_std*y_std,
                 alpha=0.95, color='k')

```

![png](/img/BayesOpt_files/BayesOpt_8_1.png)

From here there are several way to pick the next point. Two common approaches are around:

- Upper confidence bound (exploration vs exploitation)
- Expected improvement

```python
n_std = 1.0
plt.plot(X_, np.sin((X_ - 2.5) ** 2), 'b')
plt.plot(X, y, 'ro')
plt.grid(True)
plt.plot(X_, y_mean - n_std*y_std)
plt.plot(X_, y_mean)
```

![png](/img/BayesOpt_files/BayesOpt_10_1.png)

```python
n_std = 1.0
plt.plot(X, y, 'ro')
plt.grid(True)
plt.plot(X_, y_mean - n_std*y_std)
plt.plot(X_, y_mean)
```

![png](/img/BayesOpt_files/BayesOpt_11_1.png)

### GP Upper/Lower Confidence Band

This strategy aims to get a balance between exploration (search based on $\sigma$) and exploitation (search based on $\mu$), controlled by some parameter $\kappa$.

Then the next point $\mathbf{x}$ is the one which minimizes:

$$\mu(\mathbf{x}; ...) - \kappa \sigma(\mathbf{x}; ...)$$

```python
# GP - LCB

# this parameter can be automatically selected or selected manually!
# selecting to something really large (say 10000)
# will suggest different search space to say 1.0
def gplcb(x, gp=gp, kappa=1.0):
    mu, sigma = gp.predict(x, return_std=True)
    return mu-kappa*sigma

plt.grid(True)
plt.plot(X_, np.vectorize(gplcb)(X_, kappa=1.0))
plt.title("GP-LCB (minimize regret)")
```

![png](/img/BayesOpt_files/BayesOpt_13_1.png)

### Expected Improvement

Expected improvement makes use of the gaussain assumptions to calculate the expected improvement over the best current point.

Let:

$$\gamma(\mathbf{x}) = \frac{f(\mathbf{x}_\text{best}) - \mu(\mathbf{x}; ...)}{\sigma(\mathbf{x};...)}$$

Expected improvement is the point which maximizes (where to remove symbols $f^*:=f(\mathbf{x}_\text{best}), \mu:=\mu(\mathbf{x}; ...), \sigma:=\sigma(\mathbf{x};...), \gamma:=\gamma(\mathbf{x})$

$$(f^*-\mu)\phi(\gamma)+\sigma(\Phi(\gamma)) $$

```python
def ei(x, gp=gp, y_start={'y':y_samples}):
    # this calculates the expected improvmeent of a single point
    from scipy.stats import norm
    x = np.array(x)
    mu, sigma = gp.predict(x, return_std=True)
    loss_optimum = np.min(y_start['y'])
    z = (loss_optimum - mu + x)/sigma
    return (loss_optimum-mu+x)*norm.cdf(z) + sigma*norm.pdf(z)
```

```python
plt.plot(X_, np.vectorize(ei)(X_))
plt.grid(True)
plt.title("Expected Improvement (maximize)")
```

![png](/img/BayesOpt_files/BayesOpt_16_1.png)

```python
### now complete a few iterations of GP-LCB and EI

def next_gplcb(X, gp=gp, kappa=1.0):
    gplcb_points = np.vectorize(gplcb)(X, gp=gp, kappa=1.0)
    return X[np.argmin(gplcb_points)]

X_next = X.copy()
gp = GaussianProcessRegressor(kernel=kernel)
gp.fit(X_next, y)

X_ = np.linspace(0, 5, 100)
y_mean, y_std = gp.predict(X_[:, np.newaxis], return_std=True)
x_next = next_gplcb(X_, gp)

X_next = np.vstack([X_next, np.array([x_next])])
y_next = np.sin((X_next - 2.5) ** 2).reshape(-1,1)


```

```python
# we can show what it looks like 1 standard deviation away
plt.plot(X_, y_mean, 'b', X.flatten(), y, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - y_std, y_mean + y_std,
                 alpha=0.5, color='k')
```

![png](/img/BayesOpt_files/BayesOpt_18_1.png)

```python
# we can show what it looks like 1 standard deviation away
gp.fit(X_next, y_next.flatten())

X_ = np.linspace(0, 5, 100)
y_mean, y_std = gp.predict(X_[:, np.newaxis], return_std=True)

plt.plot(X_, y_mean, 'b', X_next, y_next, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - y_std, y_mean + y_std,
                 alpha=0.5, color='k')
```

![png](/img/BayesOpt_files/BayesOpt_19_1.png)

```python
plt.grid(True)
plt.plot(X_, np.vectorize(gplcb)(X_, gp=gp, kappa=1.0))
plt.title("GP-LCB (minimize regret)")
```

![png](/img/BayesOpt_files/BayesOpt_20_1.png)

```python
### for expected improvement...

### now complete a few iterations of GP-LCB and EI

def next_ei(X, gp=gp, y_start=y):
    ei_points = np.vectorize(ei)(X, gp=gp, y_start={'y':y})
    return X[np.argmax(ei_points)]

X_next = X.copy()
gp = GaussianProcessRegressor(kernel=kernel)
gp.fit(X_next, y)

X_ = np.linspace(0, 5, 100)
y_mean, y_std = gp.predict(X_[:, np.newaxis], return_std=True)
x_next = next_ei(X_, gp, y)

X_next = np.vstack([X_next, np.array([x_next])])
y_next = np.sin((X_next - 2.5) ** 2).reshape(-1,1)


```

```python
# we can show what it looks like 1 standard deviation away
plt.plot(X_, y_mean, 'b', X.flatten(), y, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - y_std, y_mean + y_std,
                 alpha=0.5, color='k')

```

![png](/img/BayesOpt_files/BayesOpt_22_1.png)

```python
# we can show what it looks like 1 standard deviation away
gp.fit(X_next, y_next.flatten())

X_ = np.linspace(0, 5, 100)
y_mean, y_std = gp.predict(X_[:, np.newaxis], return_std=True)

plt.plot(X_, y_mean, 'b', X_next, y_next, 'ro')
plt.grid(True)
plt.fill_between(X_, y_mean - y_std, y_mean + y_std,
                 alpha=0.5, color='k')
```

![png](/img/BayesOpt_files/BayesOpt_23_1.png)
