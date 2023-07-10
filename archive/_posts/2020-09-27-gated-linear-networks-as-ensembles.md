---
layout: post
category:
tags:
tagline:
---

Can we just treat gated linear networks as a fancy ensemble model? Let's have a look at the construction and see if it works as expected...

The general idea, is that the inputs to a subsequent layer is the predicted probabilities of the previous layer. We use the "context" or original data to determine the weights of the geometric mixing operation.

```py
class SimpleGLN(ClassifierMixin):
    """
    This is a one layer GLN, just to understand how it does the ensembling...
    only support binary for now
    """
    def __init__(self, base_estimator=None, layers=[5]):
        assert len(layers) == 1  # more than one layer not implemented yet...
        n_estimators = layers[0]

        if base_estimator is None:
            base_estimator = GaussianNB()
        self._neurons = [deepcopy(base_estimator) for _ in range(n_estimators)]
        self._mixer = MultiOutputRegressor(SGDRegressor())

    def _neuron_output(self, X):
        return np.stack([mod.predict_proba(X)[:, 1] for mod in self._neurons], -1)

    def fit(self, X, y):
        for m in self._neurons:
            m.fit(X, y)

        y_hat = self._neuron_output(X)
        self._mixer.fit(X, y_hat-np.expand_dims(1-y, 1))
        return self

    def predict_proba(self, X):
        # calculate the logits...
        logits = logit(self._neuron_output(X))
        w = np.clip(self._mixer.predict(X), 0, 1)

        y_proba = expit(np.squeeze(np.expand_dims(w, 1) @ np.expand_dims(logits, -1)))
        return np.stack([1-y_proba, y_proba], -1)

    def predict(self, X):
        return self.predict_proba(X)[:, 1] > 0.5
```

A rather crude implementation is shown above, whereby the original neurons are learned off the input data, and the mixing operation is formed by the context (i.e. input data), where the label is based on the "error" or performance of the neurons.

In the GLN scenario, they use purely half space type neurons (and lots of them!) rather than an (expensive) classifier algorithm. Nevertheless, this is an interesting way to "ensemble" a set of decisions in a neural network and is worth thinking about in a differentiable framework when combining outputs from multiple sources.
