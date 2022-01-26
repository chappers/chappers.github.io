---
layout: post
category:
tags:
tagline:
---

In this post we'll look at an implementation of [ABDM (Association-Based Distance Metric)](http://www.jaist.ac.jp/~bao/papers/N26.pdf) and [MVDM (Modified Value Difference Metric)](https://link.springer.com/article/10.1023/A:1022664626993) for categorical encoding which can be used in k-nearest-neighbours. There currently isn't a [paper on this but is forthcoming at the time of writing](https://docs.seldon.io/projects/alibi/en/stable/methods/CFProto.html). This is a quick implementation based on the [Seldon Alibi](https://github.com/SeldonIO/alibi/blob/master/alibi/utils/distance.py) implementation; mainly because their implementation wasn't very "sklearn-eqsue". This is also for my own edification!

# High Level View of Shared Components

**Whats the Algorithm?**

The heart of both algorithms is the usage of multi-dimensional scaling; whereby it takes an array of ordinal values, and some data and magically transforms it into a mapping from the ordinal values to a multi-dimensional scale version of it.

To do this, it computes several steps:

1.  Compute the conditional probability of the values with respect to the provided data
2.  Calculate the distance among all ordinal values based on the conditional probability
3.  Use multi-dimensional scaling to conver the distance matrix to a 1-D vector, so that we have a mapping between the ordinal values to an embedding.

**Conditional (Empirical) Probability**
The easiest way to represent this is if we one-hot encode our data, and perform a pivot to calculate the empircal mean by group (i.e. conditional probability).

The treatment for continuous variables in our reference data is then to discretise them through binning or some other mechanism.

This ends up being a one-line:

```py
mean_over = one_hot_dataframe.groupby(series_array).agg("mean")
```

**Distance Calculation**
We can then calculate the distance using any pair-wise distance measure like so:

```py
d_pair_col = manhattan_distances(mean_over)
```

In ABDM the distance metric is also additionally transformed by KL-divergence, as an improved measure of dissimilarity; This is just a preprocessing step before we apply a distance calculation.

This ends up looking like something here:

```py
from scipy.special import rel_entr  # this is KL Divergence

eps = 1e-16
kl_div_dist = lambda u, v: rel_entr(u+eps, v+eps) + rel_entr(v+eps, u+eps)


def iterable_wrapper(X, iterable_transform):
    X_dist = np.zeros(X.shape)
    for row in range(X.shape[0]):
        for col in range(X.shape[1]):
            if col < row:
                X_dist[row, col] = iterable_transform(row, col)
    X_dist += X_dist.T
    return X_dist
```

**Multi-dimensional Scaling**
For multi-dimensional scaling, this is computed off scikit-learn's algorithms:

```py
mds = MDS(
    n_components=2,
    max_iter=5000,
    eps=1e-9,
    random_state=0,
    n_init=4,
    dissimilarity="precomputed",
    metric=True,
)
mds.fit(d_pair_col)
emb = mds.embedding_
origin = emb[np.argsort(np.linalg.norm(emb, axis=1))[-1]].reshape(1, -1)
x_mapping = np.linalg.norm(emb - origin, axis=1)

final_mapping = zip([list(mean_over.index), x_mapping.tolist()])
```

# High Level View of Split Components

The difference in the two algorithms can be summaried by how the comparitive dataset is produced, as well as the preprocessing which is used in both cases.

MVDM: the comparison dataset is a one-hot encoded version of the label
ABDM: the comparison dataset is the modelling dataset itself, with the numeric variables discretised. In addition it uses a symmetric KL-Divergence as a preprocessor before applying manhattan distance.

From here we can apply some preprocessing for both approaches and wrap it up. Would I use the code below in production? Maybe; just may need some config/others so that parts like KBin and MDS portions are easily configurable.

````py
# ABDM and MVDM
import numpy as np
from typing import Dict, Tuple
from sklearn.manifold import MDS
from itertools import product
import numpy as np
import pandas as pd
from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.preprocessing import OneHotEncoder
from sklearn.metrics.pairwise import manhattan_distances

from itertools import combinations
from sklearn.base import TransformerMixin
from sklearn.preprocessing import StandardScaler, KBinsDiscretizer
from sklearn.pipeline import make_pipeline

from scipy.special import rel_entr

eps = 1e-16
kl_div_dist = lambda u, v: rel_entr(u + eps, v + eps) + rel_entr(v + eps, u + eps)


def iterable_wrapper(X, iterable_transform):
    X_dist = np.zeros(X.shape)
    for row in range(X.shape[0]):
        for col in range(X.shape[1]):
            if col < row:
                X_dist[row, col] = iterable_transform(row, col)
    X_dist += X_dist.T
    return X_dist


def multidim_scaling(
    series_array,
    one_hot_dataframe,
    distance=manhattan_distances,
    iterable_transform=None,
):
    mean_over = one_hot_dataframe.groupby(series_array).agg("mean")
    output_index = mean_over.index
    if iterable_transform is not None:
        mean_over = iterable_wrapper(mean_over, iterable_transform)
    d_pair_col = distance(mean_over)

    mds = MDS(
        n_components=2,
        max_iter=5000,
        eps=1e-9,
        random_state=0,
        n_init=4,
        dissimilarity="precomputed",
        metric=True,
    )

    mds.fit(d_pair_col)
    emb = mds.embedding_
    origin = emb[np.argsort(np.linalg.norm(emb, axis=1))[-1]].reshape(1, -1)
    x_mapping = np.linalg.norm(emb - origin, axis=1)
    return [list(output_index), x_mapping.tolist()]


class MVDMTransformer(TransformerMixin):
    """
    Usage:

    ```
    X = pd.DataFrame({'x0': np.random.choice(range(2), 100), 'x1': np.random.choice(range(2), 100), 'x2':np.random.choice(range(3), 100)})

    y_cats = 4
    y = np.random.choice(range(y_cats), 100)

    # alibi also suggests centering...
    mvdm = MVDMTransformer(['x0', 'x1', 'x2'])
    X_trans = mvdm.fit_transform(X, y)

    pipeline = make_pipeline(MVDMTransformer(['x0', 'x1', 'x2']), StandardScaler())
    X_trans = pipeline.fit_transform(X, y)

    # now apply k-nn for counterfactuals
    ```
    """

    def __init__(self, col=[]):
        """
        For numeric columns, please pre-process using
        `sklearn.preprcoessing.KBinsDiscretizer`; or keep as is.
        """
        self.col = col
        self.y_ohe = None
        self.x_mds = None
        self.mapping = None

    def fit(self, X, y):
        mapping = {}
        X_sel = X[self.col]
        y_ohe = OneHotEncoder(sparse=False).fit_transform(y.reshape(-1, 1))
        y_ohe = pd.DataFrame(y_ohe)

        for idx, col in enumerate(self.col):
            mapping[col] = multidim_scaling(X_sel[col], y_ohe)

        self.mapping = mapping.copy()
        return self

    def transform(self, X, y=None):
        # transforms dataset to one with categorical mapping
        X_sel = X[self.col].copy()
        for col in self.col:
            X_sel[col] = X_sel[col].replace(*self.mapping[col])
        return X_sel

    def fit_transform(self, X, y):
        return self.fit(X, y).transform(X)


class ABDMTransformer(TransformerMixin):
    """
    This is an unsupervised approach. We have two inputs:
    *  categorical columns, whereby the transformation is done
    *  numeric columns, whereby this is first binned and then used as a category to
       calculate statistics

    Usage:

    ```
    X = pd.DataFrame({'x0': np.random.choice(range(2), 100),
    'x1': np.random.choice(range(2), 100),
    'x2':np.random.choice(range(3), 100),
    'x3': np.random.normal(size=100), 'x4': np.random.normal(size=100)})

    y_cats = 4
    y = np.random.choice(range(y_cats), 100)

    # alibi also suggests centering...
    abdm = ABDMTransformer(['x0', 'x1', 'x2'], ['x3', 'x4'])
    X_trans_all = abdm.fit_transform(X, y)

    abdm = ABDMTransformer(['x0', 'x1', 'x2'], [])
    X_trans_cats = abdm.fit_transform(X, y)

    pipeline = make_pipeline(ABDMTransformer(['x0', 'x1', 'x2']), StandardScaler())
    X_trans = pipeline.fit_transform(X, y)

    # See if there is a linear transformation betwene the two
    from sklearn.linear_model import SGDRegressor
    mod = SGDRegressor()

    X_in = manhattan_distances(X_trans_all)[np.argwhere(manhattan_distances(X_trans_all) != 0)].flatten()
    y_in = manhattan_distances(X_trans_cats)[np.argwhere(manhattan_distances(X_trans_cats) != 0)].flatten()

    mod.fit(X_in.reshape(-1, 1), y_in)
    # calculate mse

    from sklearn.metrics import mean_squared_error
    mean_squared_error(y_in, mod.predict(X_in.reshape(-1, 1)))

    # now apply k-nn for counterfactuals
    ```
    """

    def __init__(self, col=[], num_col=[]):
        """
        For numeric columns, please pre-process using
        `sklearn.preprcoessing.KBinsDiscretizer`
        """
        self.col = col
        self.num_col = num_col
        self.kbins_discrete = None
        self.x_mds = None
        self.mapping = None

    def fit(self, X, y=None):
        mapping = {}
        X_sel = X[self.col]

        if len(self.num_col) > 0:
            X_num = X[self.num_col]
            self.kbins_discrete = KBinsDiscretizer(encode="ordinal")
            X_num = pd.DataFrame(
                self.kbins_discrete.fit_transform(X_num).astype(int),
                columns=self.num_col,
            )
            X_sel = pd.concat([X_sel, X_num], axis=1)

        all_cols = list(X_sel.columns)
        for idx, col in enumerate(self.col):

            all_other_cols = [x for x in all_cols if x != col]
            X_other = pd.DataFrame(
                OneHotEncoder(sparse=False).fit_transform(X_sel[all_other_cols])
            )
            mapping[col] = multidim_scaling(
                X_sel[col], X_other, iterable_transform=kl_div_dist
            )

        self.mapping = mapping.copy()
        return self

    def transform(self, X, y=None):
        # transforms dataset to one with categorical mapping
        X_sel = X[self.col].copy()
        for col in self.col:
            X_sel[col] = X_sel[col].replace(*self.mapping[col])
        return X_sel

    def fit_transform(self, X, y):
        return self.fit(X, y).transform(X)
````
