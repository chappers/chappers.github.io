---
layout: post
category : 
tags : 
tagline: 
---


Is it really that hard to teach a software engineer machine learning?

This is a thought experiment that I had; that implementing simple variations of popular algorithms which are commonly used is sufficient for a software engineer to build out pipelines and help the wider team be more effective. 

Gradient Descent - Or how to iteratively find the "line of best fit"
--------------------------------------------------------------------

One of the simplest algorithms you could use for line of best fit is simply doing an interval halving method, or a binary search method. Then to find the line of best fit, becomes a minimisation problem using "mean squared error". 

In some kind of Pseudo-code this would look like

```
Input:  data `x`, output `y`
        maximum iterations `tol`
        tolerance `e`
Assume: allowed values in the range of `[low, high]`

a0, a1 = [low, high]
b0, b1 = [low, high]

def perform_binary_search(low, high, mid, y):
    if score(low, y) < score(mid, y):
        return index low, index mid
    else:
        return index mid, index high

while number of iterations <= tol:
    a = (a0 + a1)/2
    b = (b0 + b1)/2

    y0, y1, y_mid = a0*x + b, a1*x + b, a*x + b
    a0, a1 = perform_binary_search(y0, y1, y_mid)

    y0, y1, y_mid = a*x + b0, a*x + b1, a*x + b
    b0, b1 = perform_binary_search(y0, y1, y_mid)

    if score(y_mid, y) < e:
        output a, b

output a, b
```

Then to program this:

```py
import numpy as np
from sklearn.datasets import make_regression
from sklearn.datasets import make_classification
from sklearn.metrics import mean_squared_error, log_loss, accuracy_score
from scipy.special import expit


class BisectionModel(object):
    def __init__(
        self,
        low=-999,
        high=999,
        max_iter=1000,
        tol=1e-6,
        score_fn=mean_squared_error,
        predict_fn=lambda x: x,
    ):
        self.low = low
        self.high = high
        self.max_iter = max_iter
        self.tol = tol
        self._score = score_fn
        self._predict = predict_fn  # this is so we can transform it to a classifier

    def binary_search_islow(self, low, high, y, sample_weight):
        if self._score(y, low, sample_weight) < self._score(y, high, sample_weight):
            return True
        else:
            return False

    def _single_iter(
        self, X, y, coef_low, coef_high, inter_low, inter_high, sample_weight
    ):
        feats = X.shape[1]
        coef_mid = [(x + y) / 2 for x, y in zip(coef_low, coef_high)].copy()
        inter_mid = (inter_low + inter_high) / 2
        for f in range(feats):
            coef_temp_low = np.array(coef_mid.copy())
            coef_temp_low[f] = coef_low[f]
            coef_temp_high = np.array(coef_mid.copy())
            coef_temp_high[f] = coef_high[f]

            low = self._predict(X.dot(coef_temp_low) + inter_mid)
            high = self._predict(X.dot(coef_temp_high) + inter_mid)

            if self.binary_search_islow(low, high, y, sample_weight):
                coef_high[f] = coef_mid[f]
            else:
                coef_low[f] = coef_mid[f]

        # check for intercept
        coef_mid = np.array([(x + y) / 2 for x, y in zip(coef_low, coef_high)])
        low = self._predict(X.dot(coef_mid) + inter_low)
        high = self._predict(X.dot(coef_mid) + inter_high)

        if self.binary_search_islow(low, high, y, sample_weight):
            inter_high = inter_mid
        else:
            inter_low = inter_mid

        return coef_low, coef_high, inter_low, inter_high

    def fit(self, X, y, sample_weight=None):
        feats = X.shape[1]
        self.coef_low = [self.low for _ in range(feats)]
        self.coef_high = [self.high for _ in range(feats)]
        self.inter_low = self.low
        self.inter_high = self.high
        return self.partial_fit(X, y, sample_weight)

    def partial_fit(self, X, y, sample_weight=None):
        # determine the number of inputs we need etc...
        feats = X.shape[1]
        coef_low = self.coef_low.copy()
        coef_high = self.coef_high.copy()
        inter_low = self.inter_low
        inter_high = self.inter_high

        for _ in range(self.max_iter):
            coef_low, coef_high, inter_low, inter_high = self._single_iter(
                X, y, coef_low, coef_high, inter_low, inter_high, sample_weight
            )

            # calculate the score of both low and high and check tol.
            low = self._predict(X.dot(np.array(coef_low)) + inter_low)
            high = self._predict(X.dot(np.array(coef_high)) + inter_high)
            if (
                self._score(y, low, sample_weight) < self.tol
                and self._score(y, high, sample_weight) < self.tol
            ):
                break

        self.coef = np.array([(x + y) / 2 for x, y in zip(coef_low, coef_high)])
        self.inter = (inter_low + inter_high) / 2
        self.coef_low = coef_low.copy()
        self.coef_high = coef_high.copy()
        self.inter_low = inter_low
        self.inter_high = inter_high
        return self

    def predict(self, X):
        return self._predict(X.dot(self.coef) + self.inter)


X, y = make_regression()
br = BisectionModel()
br.fit(X, y)

mean_square_error(br.predict(X), y)

X, y = make_classification()
br = BisectionModel(score_fn=log_loss, predict_fn=expit)
br.fit(X, y)
log_loss(y, br.predict(X))
accuracy_score(y, np.round(br.predict(X)))

```

This approach still isn't very good for many reasons; but it is a step in the right direction in terms of understanding how algorithms may iteratively improve over time with addition of new data without delving into calculus. Of course directly optimising with respect to the loss function rather than treating it as a black box will serve to do better; or even using simulated annealing or genetic algorithms! Adding a form of regulariser or clipping will help with convergence and the coefficients which are provided.


Simple Boost
------------

One of the easiest way to understand boosting is to understand that it consists of iteratively reweighting observations which are passed into the "weak learner". We can use a rather naive approach to complete this:

```
Input:  data `X`, output `y`
        reweighting scheme
        number of weak learners `n`
        learning rate or step size `v`

w = uniform weights

for learner in n:
    X' = weighted variant of X with weights w
    m <- emit learner trained on (X', y)
    construct M = M + v*m

Output final model M
```

To write this in code, we'll first come up with decision stump model to demonstrate that this ensembling approach does better than the weak learner


```py
class SimpleBoost(object):
    def __init__(
        self,
        model=lambda _: BisectionModel(
            max_iter=100, score_fn=log_loss, predict_fn=expit
        ),
        n_models=10,
        step_size=0.1,
    ):
        self.model = model
        self.boost_model = []
        self.n_models = n_models
        self.step_size = step_size

    def weight_scheme(self, y_true, y_pred):
        """
        Takes in true labels and prediction, uses this information
        to determine what are the new (perturbed) weights. 

        These weights are calculates to be:
        w = w * _new weight scheme_
        """
        y_pred = np.round(y_pred)
        y_weights = y_true == y_pred

        # make it so that if it gets it right then *0.9, otherwise *1.1
        y_weights = 1 + (y_weights * (-2 * self.step_size) + self.step_size)
        return y_weights

    def fit(self, X, y, sample_weight=None):
        if sample_weight is None:
            sample_weight = np.array([1.0 / X.shape[0] for _ in range(X.shape[0])])
        for _ in range(self.n_models):
            self.boost_model.append(self.model(None))
            Xw = np.random.choice(
                range(X.shape[0]), size=X.shape[0], replace=True, p=sample_weight
            )
            self.boost_model[-1].fit(X[Xw, :], y[Xw])
            sample_weight = sample_weight * self.weight_scheme(X, y)
            sample_weight = sample_weight / np.sum(sample_weight)

            # we assume step size is constant - no need to do line search here

    def predict(self, X):
        # get average of all predictions
        return np.mean([m.predict(X) for m in self.boost_model], 0)


X, y = make_classification()
br = BisectionModel(max_iter=100, score_fn=log_loss, predict_fn=expit)
br.fit(X, y)
print(accuracy_score(y, np.round(br.predict(X))))

sb = SimpleBoost()
sb.fit(X, y)
print(accuracy_score(y, np.round(sb.predict(X))))
```

There are a number of challenges with this algorithm - for example, we don't compute the step-size to determine what or how we want to ensemble the models (this is done typically using line search). The weighting scheme, in this scenario is based on accuracy and would not work for regression (in fact we need some kind of bounded loss function so that it can be weighted appropriate - this is why we typically use exponential loss or similar function). For an arbitrary scheme, one could use AnyBoost (i.e. Gradient Boosting) which allows residual boosting to kick in. Nevertheless this `SimpleBoost` algorithm provides a starting point for an almost "black box" approach to boosting relatively simple models.

Wrap Up
-------

We've demonstrated how we can provide pseudo algorithms for building Machine Learning models iteratively, as well as a rough sketch for a Boosting algorithm. Both algorithms are purely for pedagogical reasons, but do not have much clout or weight from a theoretical perspective. It may be interesting to review theoretical properties another time and think about where these could be used in terms of having a black box optimiser. 

