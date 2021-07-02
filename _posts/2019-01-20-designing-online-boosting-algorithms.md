---
layout: post
category : 
tags : 
tagline: 
---

This post revolves around several papers on online boosting including:

*  Optimal and Adaptive Algorithms for Online Boosting - ICML 2015
*  Online Gradient Boosting - NIPS 2015

The ideas presented here are not the original algorithms, but seek to gain an understanding into the intuition when switching from batch to online for boosting algorithms. Such as the implication of moving to regression variant of online adaboost as done in "Improving Regressors using Boosting Techniques" (1997).

When we think of gradient boosting and adaboost, the relationships between the two can be shown as:

*  AdaBoost revolves around reweighting the training samples for subsequent learners
*  Gradient boosting is equivalent to AdaBoost under exponential loss, and be interpreseted as subsequent learners fitting the residuals of the previous span of weak learners with respect to the chosen loss function

This can be shown analytically as in Gradient boosting, we seek to choose an $h$ which maximise $-\nabla L(F) \cdot h$ where $F$ is the span of previous weak learners and $h$ is the new proposed weak learner. This goal of maximisation is shown through Taylor series approximation:

$$L(F + \alpha h) = L(F) + \alpha \nabla L (F) \cdot h$$

Under exponential loss, we can see that

$$-\nabla L (F) \cdot h = yh e^{-hF}$$

Where 

$$e^{-hF} = e^{-y \sum \alpha h} = \prod e^{-y \alpha h} \propto D(i)$$

Where $D(i) represents the weight of instances based on the previous error. Hence

$$-\nabla L (F) \cdot h \propto D(i) yh$$

In Adaboost, due to the construction of exponential loss, the subsequent update of $\alpha$ can be solved analytically, whereas in gradient boosting, this is typically updated again through gradient descent. If gradient descent with exponential loss did use the analytically derived $\alpha$ as an update rather than an approximation - we would in theory yield the same model.

Intuition Behind Online Boosting
================================

To construct online AdaBoost, there are two components which we need to look at:

1.  Incremental update of instances weights through the "partial" errors for training each weak learner
2.  Incremental update of the weights $\alpha$ for the final boosted model

In the analogous online gradient boosting we would need

1.  Incremental update of the partial derivatives with respect to the weak learner
2.  Incremental update of a shrinkage factor by gradient descent for the final model

There are a number of things we need to be careful here:

*  in either case, the loss functions need to be bounded. If it is not, it can explode and "blow up" the classifier e.g. if it had infinite error, the incremental update would blow out the instance weight which is fed into the weak learner, or the partial derivatives for the weak learner - whereby the downstream effects would mean tha the subsequent learners (through either back prop or feedforward) would get "no information"
*  Online AdaBoost suggests using surrogate loss function to further bound the update of $\alpha$, Online gradient boosting can use carefully chosen surrogate functions (e.g. squared loss for "Online Gradient Boosting on Stochastic Data Streams") or simply clips the allowable values of the shrinkage

In terms of implementation strategies:

*  Online AdaBoost relies on reweighting instances, this can be done in frameworks such as `scikit-learn`; making use of minibatch updates
*  Online Gradient Boosting relies updating through partial loss gradients which means that we either need to construct custom loss functions or make use of auto-differentiation to feed this in using Tensorflow/PyTorch and the like


Online AdaBoost
---------------

The pseudo-code for this would act as follows:

```
1.  Calculate the prediction for all weak learners
2.  Calculate the "error contribution" for each weak learner 
3.  Use (2) to update $\alpha$ for each weak learner (in paper, this was done via a surrogate loss function with properties around being bounded)
4.  Using (2) to re-weight instances for each weak learner and feed (minibatch or one-by-one) to each weak learner
5.  Go to (1)
```

In this instance the prediction would be $\sum \alpha h$

Online Gradient Descent
-----------------------

The pseudo-code for this would act as follows:

```
1.  Feed forward through the model
2.  Calculate and feed the partial loss for each weak learner 
3.  Use (2) to update shrinkage for the $span(F)$
4.  Go to (1)
```

In this instance the prediction would be determined by some linear combination of $h$ which is calculated in an iterative feedforward manner. Carefully chosen surrogate loss functions could be employed to update $\alpha$ so that the prediction would be $\sum \alpha h$ - however it is important to note that in the online gradient descent formulation this is not necessarily needed, and the gradient updates could simply be clipped instead. 



