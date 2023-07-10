---
layout: post
category:
tags:
tagline:
---

This post is to quickly go through linear SVMs and decision boundaries.

**Decision Rules in Binary Classification**

Consider the simple case where we have one predictor and it is a binary classification problem. Then the formulation for linear SVM would be:

$$\hat{y} = wx + b$$

Where if $\hat{y} \geq 0$, then the positive class is predicted, and would be assigned to the negative class otherwise. In this simple formulation, the decision rule surrounding this model would be:

$$wx + b >= 0$$

Or in other words, if treated as a decision boundary:

$$x >= -\frac{b}{w}$$

In the logistic regression scenario, as the linear component subsequently gets the softmax function applied. This would yield the logistic regression $p = \frac{1}{1+\exp(-wx - b)}$. then the decision boundary is in fact the same if the threshold is set to $p >= 0.5$.

Another approach is to use softmax function with temperature. This will naturally move towards a one hot encoding formulation for the decision boundary.

$$\frac{1}{1+e^{-x/\tau}}$$

Whereby, if $\tau < 1$ and approached zero, the boundaries would become more steep, and further apart, whereas if $\tau > 1$ was large, the probabilities would move closer and closer together.

The consequences of this is if we compute axis parallel splits in decision trees for neural networks (see neural network decision forests), then we can recover the decision rule that is used to build the decision tree.

How we implement this would be a discussion for another day.

For the multiple predictors, the decision boundary interpretation becomes more complicated; as the splits are no longer axis parallel, even if we simply bring in one additional predictor; as the decision boundary is then a hyperplane.

**Decision Rules in Multi-class Classification**

Take for example, if we build a model in tensorflow/keras with `categorical_crossentropy` loss. For arguments sake the code may look like this:

```py
import pandas as pd
import numpy as np
from sklearn.datasets import make_classification
import tensorflow as tf
from tensorflow.keras import layers, Input,

X, y_ = make_classification()
y = tf.keras.utils.to_categorical(y_)

input1 = Input(shape=(20,))
pred_out = layers.Dense(2, activation='softmax')(input1)
#pred_out = layers.Activation('softmax')(pred_layer)
model = Model(inputs=[input1], outputs=[pred_out])
model.compile(optimizer=tf.train.AdamOptimizer(),
            loss='categorical_crossentropy',
            metrics=['accuracy'])
model.fit(X, y, batch_size=32, epochs=50, verbose=1)
print(model.evaluate(X, y))
```

Then recovering the weights via:

```py
weight_list = [K.eval(x) for x in model.layers[1].weights]
w, b = weight_list
```

Will naturally yield `w` to be of shape `(20, 2)`, and `b` to be of shape `(2, )`; due to the nature of `categorical_crossentropy`. Note that this formulation can easily generalise to classes greater than 2, especially if we are using formulation of one versus rest, which is typical for situations where we are using logistic regression formulation.

In this case, the calculation for the decision boundary can be computed using [softmax](https://en.wikipedia.org/wiki/Softmax_function):

$$\sigma(x_i) = \frac{e^{-Wx_i}}{\sum_{\forall j} e{-Wx_j}}$$

where $x_i$ represents probability of instance $x$ belonging to class $i$, with $W$ being design matrix for purpose of clarity.

Again, assuming only a single predictor is chosen, we can make use of the max norm, as it would be an indication of discriminatory power (assuming that predictors are normalised).

Now since we care only for the decision boundary, and assuming that the threshold we choose is based highest probability (i.e. not a custom threshold), we can determine which category is chosen through

$$w_i x + b_i > w_j x + b_j$$

selecting whatever value yields the highest value. Since in all these equations, the usage of $x$ is consistent, then for the two class setting, we would use the decision boundary for class $i$ over class $j$ if $w_i - w_j > 0$

$$x > - \frac{b_i - b_j}{w_i - w_j}$$

and the inequality sign would be flipped otherwise. To yield the decision boundary as shown below

```py
def get_decision_boundary(w, b, verbose=False):
    # assumes binary case only!
    w_norm = np.sum(np.abs(w), axis=-1)
    w_max = np.argmax(w_norm) # this selects the best predictor based on the norm.

    boundary_value = -(b[0] - b[1])/(w[w_max, 0] - w[w_max, 1])
    if w[w_max, 0] > w[w_max, 1]:
        return "x[{}] >= {}".format(w_max, boundary_value)
    else:
        return "x[{}] < {}".format(w_max, boundary_value)

print(get_decision_boundary(w, b, True))
```
