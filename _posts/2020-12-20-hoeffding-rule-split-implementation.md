---
layout: post
category : 
tags : 
tagline: 
---

In this post we're going to look at Hoeffding rule splits, which are the basis for incremental decision tree algorithms like "Very Fast Decision Tree". 

In this post we will look at streaming patterns to enable this from a machine learning perspective (in the spirit of the $P^2$ percentile algorithm).

The Hoeffding bound provides a probabilistic error based on the number of samples provided for a given metric. Let's suppose that there is random variable $x$, where the range is $R$ (probability of range is 1, and for information gain the range is $\log c$ for $c$ classes).


The bound is given, that with probability $1-\delta$, the true mean of a variable is at least $\bar{r} - \epsilon$ where 

$$\epsilon = \sqrt{\frac{R^2 \log(1/\delta)}{2n}}$$

What we actually interested is the metric (gini or otherwise) of a particular attribute $X_a$ split $G(X_a)$, and we want to choose the particular attribute where $G$ has the _smallest_ difference across all candidate splits; that is we want to choose $\Delta \bar{G} > \epsilon$ condition to hold with probability $1-\delta$. 

By the definition of gini impurity, it can hold a value between $0$ and $1$ (actually can bound it tigher dependent of the number of classes $c$: $R = \frac{c-1}{c}$), where it is:

$$H(X_m) = \sum_{k} p_{mk} ( 1- p_{mk})$$

Based on this, the target bound would be in the form:

$$\epsilon = \sqrt{\frac{\log(1/\delta)}{2n}}$$

So if we choose $\delta = 0.05$, then with $n = 25$ observed instances, we'll have $\epsilon = \sqrt(\log(20)/50) = 0.1613$, if $n = 100$ then $\epsilon = 0.08$ which suggests the more data (or evidence) that we "looser" the difference needs to be for us to be reasonably comfortable with the choice of split. 

Figuring out the split
----------------------

The easiest variation to learning a split in an online manner, is to presume that the node is modeled by a gaussian naive bayes, since the naive examples makes the split meaningful. 

```py
import numpy as np
from sklearn.naive_bayes import GaussianNB

stream = SEAGenerator()

model = GaussianNB(priors=[0.5, 0.5])


def find_split(
    model, n_seen=0, proposal_scores=None, delta=0.05, min_seen=100, *args, **kwargs
):
    """
    Attempts to find splits.

    Returns:
    {
        'is_split' - bool,
        'n_seen' - int
        'proposal_scores' - array
        'model' - naive bayes model
        'split info' - None or {feat_idx:val}
    }
    """
    X, y = stream.next_sample(5)
    hoeffding_bound = lambda n: np.sqrt(0.25 * np.log(1 / delta) / (2 * n))
    if n_seen == 0:
        model.fit(X, y)
    else:
        model.partial_fit(X, y)
    proposed_splits = np.mean(model.theta_, 0)
    n_seen += X.shape[0]

    if proposal_scores is None:
        proposal_scores = np.zeros(X.shape[1])
    for feat_idx in range(X.shape[1]):
        y_indx = X[:, feat_idx] < proposed_splits[feat_idx]
        proposal_scores[feat_idx] += np.sum(y[y_indx])

    impurity = [
        2 * (x / n_seen) * (1 - x / n_seen) for x in proposal_scores
    ]  # this isn't 100% correct, I'm just lazy...
    vals = sorted(impurity.copy())[:2]
    is_split = False
    split_info = None
    print("Seen: ", n_seen, ". Minimum before making a decision: ", min_seen)
    print(
        "\tCurrently diff between top two scores are ",
        np.abs(vals[0] - vals[1]),
        ". hoeffding bound: ",
        hoeffding_bound(n_seen),
    )
    if np.abs(vals[0] - vals[1]) < hoeffding_bound(n_seen) and n_seen >= min_seen:
        is_split = True
        split_info = {
            "feat": np.argmin(impurity),
            "val": proposal_scores[np.argmin(impurity)],
            "impurity": np.min(impurity),
        }

    return dict(
        is_split=is_split,
        model=model,
        n_seen=n_seen,
        min_seen=min_seen,
        proposal_scores=proposal_scores,
        split_info=split_info,
    )


split_not_found = True
info = {
    "is_split": False,
    "model": model,
    "n_seen": 0,
    "proposal_scores": None,
    "split_info": None,
}
while not info["is_split"]:
    info = find_split(**info)

import pprint

pprint.pprint(info)


```
