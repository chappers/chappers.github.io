---
layout: post
category : 
tags : 
tagline: 
---

_This post was more to get myself thinking, nothing here is rigorous or necessarily makes sense - leaving it here for historical reasons_

How would you train a decision tree via stochastic gradient descent? This idea was covered in [Efficient Non-greedy Optimization of Decision Trees](https://papers.nips.cc/paper/5886-efficient-non-greedy-optimization-of-decision-trees.pdf).

## What does it mean to be "non-greedy"

When we consider CART or similar algorithms, they are often _greedy_, that is when a split is found it cannot be changed. This obviously violates or affects the ability to have partial updates. 

The general gist of the algorithm is to relax or allow decision stumps to be oblique instead of parallel to the axis; in other words instead of a single feature being compared at each node, a comparision based on a linear combination is made:

$$\sum_{i=1}^p a_i X_i + a_{p+1} > 0$$

Where $p$ is the number of features, and $a_i$ is the learnt weights. This can intuitively be learnt through SGD approaches that is common in linear models



**Stochastic Gradient Descent algorithm (Duchi et al., 2010; McMahan and Streeter, 2010)**

* Input: Invariance update function $s$, $\mathbf{w}=0$, $\mathbf{G}=\mathbf{I}$
* _FOR_ every $(\mathbf{x}, y)$ pair in training set
    1. $\mathbf{g} \leftarrow \nabla_{\mathbf{w}} \mathit{l} (\mathbf{w}^\top \mathbf{x}; y)$
    2. $\mathbf{w} \leftarrow \mathbf{w} - s(\mathbf{w}, \mathbf{x}, y) \mathbf{G}^{-1/2} \mathbf{g}$
    3. $G_{jj} \leftarrow G_{jj} + g_j^2$ for all $j=1, ..., d$
* _END FOR_


## Learning Nodes and Probabilities

The additional difference to the oblique decision boundaries, is the routing through the decision tree is now stochastic (probabilistic) where a decision is _weighted_ throughout the nodes. This allows the weights of the model to be updated incrementally throughout the whole model. 

This results in the optimization problem learning two things:

1.  The weights to determine each of the node decisions
2.  The weights associated with the leaf nodes to determine the final prediction.

Thus finally putting it altogether, without proof, the final algorithm is presented in the paper:

**Stochastic Gradient Descent algorithm for non-greedy decision tree learning**


* _FOR_ $t=0$ to $\tau$
    1. For minibatch $(\mathbf{x}, y)$ from $\mathcal{D}$
    2. $\hat{h} \leftarrow \text{sgn}(W^{(t)} \mathbf{x})$
    3. $\hat{g} \leftarrow \underset{g \in \mathcal{H}^m}{\operatorname{arg\max}} \left( \mathbf{g}^\top W^{(t)} \mathbf{x} + \mathit{l}(\Theta^\top f(\mathbf{g}), y) \right)$
    4.  $W^{(tmp)} \leftarrow W^{(t)} - \eta \hat{g}\mathbf{x}^\top + \eta \hat{h}\mathbf{x}^\top$
    5. _FOR_ $i=1$ to $m$:
        * Update via: $W^{(t+1)} \leftarrow \min (1, \sqrt{\nu}/\lvert W^{(tmp)} \rvert_2 ) W^{(tmp)}$
    6. _ENDFOR_
    7. $\Theta^{(t+1)} \leftarrow \Theta^{(t)} - \eta \nabla_\Theta \mathit{l} (\Theta^\top f(\hat{g}), y)$
* _ENDFOR_


