---
layout: post
category : 
tags : 
tagline: 
---

There are of course several ways one can approximate correlation. In this post I thought I would outline the use of kernel approximation and how to relate that to correlation measures. 

Rough outline:

*  Realise Cosine similarity is the same as correlation when centered
*  Use kernel approximation method (Nystroem)

**Cosine Similarity**

The link to cosine similarity is best described in [this post](https://stats.stackexchange.com/questions/235673/is-there-any-relationship-among-cosine-similarity-pearson-correlation-and-z-sc).

The important aspect is that 

$$ \rho_{xy} = \frac{1}{n} \sum_i z_x z_y$$

where \\(z_i\\) is the z-score for the \\(i\\)th variable. As the vector in Cosine is a z-score if it was already centered, then assuming that the vector was "pre-centered", the results would be equivalent. 

In order to approximate it quickly, then the approach could be

```py
from sklearn import preprocessing
from sklearn.kernel_approximation import Nystroem
from sklearn.metrics.pairwise import pairwise_kernels
import numpy as np

X_scaled = preprocessing.scale(X_train)
X_approx_cor = pairwise_kernels(X_scaled.T, metric='cosine')
np.corrcoef(X_train.T) # this will be the same as above
```
