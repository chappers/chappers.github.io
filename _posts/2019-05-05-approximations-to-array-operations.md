---
layout: post
category:
tags:
tagline:
---

When you think about Machine Learning operations, often the complexity arises when the operations which are used operate over arrays rather than over records.

For example, elementary operations which are row-based generally have simple SQL query analogues; whether you are doing something like addition, subtraction or even calculating the mean via a group by. However if you are doing group by comparisons or operations over a sequential data, you will essentially be performing an operation which is like an inner loop (or an all-pairs operation).

**Example**

One could normalise prediction outputs for recommendation across multiple items by using percentile, another approach is simply to shift the location of the logistic function and scale it to the desired output.

```py
import numpy as np

def scale_logx(x, mean, variance):
    """n.b. stronger versions are typically implemented in neural network libraries"""
    return (x-mean)/np.std(variance)
```

We should take care when performing this; as the transformation would have impact when applying softmax function (infact performing these transformations have direct analogues to gumbel-softmax/concrete distribution approaches). To understand what happens:

- Shifting by the mean (or location) would move the midpoint of the distribution to the "middle" which would align all other log-odds distributions as well; whereas the shift to variance would control the scale or the steepness to how it approaches either extreme points (i.e. $P(y) = \{0, 1\}$).

Example code which demonstrates this:

```py
import numpy as np
import pandas as pd
from scipy.special import expit, logit
from scipy.stats import truncnorm
from collections import Counter
from sklearn.preprocessing import normalize, StandardScaler

def get_params(my_mean=None, my_std=None, low=0, high=1):
    a, b = (low - my_mean) / my_std, (high - my_mean) / my_std
    return a, b, my_mean, my_std

a0, b0, c0, d0 = get_params(0.5, 0.3)
a1, b1, c1, d1 = get_params(0.2, 0.4)
a2, b2, c2, d2 = get_params(0.1, 0.3)
a3, b3, c3, d3 = get_params(0.7, 0.3)

x_proba = np.clip(np.stack([truncnorm(a0, b0, c0, d0).rvs(size=500),
                    truncnorm(a1, b1, c1, d1).rvs(size=500),
                    truncnorm(a2, b2, c2, d2).rvs(size=500),
                    truncnorm(a3, b3, c3, d3).rvs(size=500)], axis=1), 1e-11, 1-1e-11)

x_logit = logit(x_proba)
df = pd.DataFrame(x_proba, columns=['m1', 'm2', 'm3', 'm4'])

# if we normalise first then it should look a lot more random across all levels
df_scaled = pd.DataFrame(expit(StandardScaler().fit_transform(x_logit)), columns=['m1', 'm2', 'm3', 'm4'])

# let's redo by calculating percentile explicitly.
df_rank = pd.DataFrame(x_proba.copy(), columns=['m1', 'm2', 'm3', 'm4'])
df_rank['m1'] = df_rank['m1'].rank()
df_rank['m2'] = df_rank['m2'].rank()
df_rank['m3'] = df_rank['m3'].rank()
df_rank['m4'] = df_rank['m4'].rank()

print("Raw Values:\t", Counter(np.argmax(df.values, 1).tolist()).most_common())
print("Scaled logit:\t", Counter(np.argmax(df_scaled.values, 1).tolist()).most_common())
print("Ranking:\t", Counter(np.argmax(df_rank.values, 1).tolist()).most_common())
```

Output:

```
Raw Values:	 [(3, 270), (0, 148), (1, 61), (2, 21)]
Scaled logit:	 [(2, 130), (1, 129), (3, 122), (0, 119)]
Ranking:	 [(1, 128), (0, 125), (2, 124), (3, 123)]
```
