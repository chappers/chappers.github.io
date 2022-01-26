---
layout: post
category:
tags:
tagline:
---

Sometimes the best way to understand what is going on is to read the code rather than the pseudo-code. For example, in SAC how do you calculate the log probability for the actions in continuous space? This is important from the perspective that the underlying _implementation_ is not represented of what is in the pseudo-code because of numeric stability challenges.

The general idea is use an unbounded Gaussian as the action distribution (see Appendix C of the SAC paper "Soft Actor-Critic Algorithms and Applications"). As it needs to fit into a finite interval, the invertible squashing function (tanh) is used to compute the likelihood of the bounded actions. This then looks like the below:

$$\log \pi(a | s) = \log \mu(u | s) - \sum_{i=1}^D \log (1- \text{tanh}^2(u_i))$$

Where $a = \text{tanh}(u)$ and is applied elementwise.

In more specific implementation code this is implemented as follows:

1.  estimate $\mu$ and $\sigma$ of the Gaussian distribution from the input observation
2.  sample from Gaussan distribution given $\mu$ and $\sigma$ to yield $\log \mu(u \| s)$ to yield $\log \mu(u \| s)$
3.  calculate $\log (1- \text{tanh}^2(u))$ element-wise using the equivalent form shown below (from rlkit).
4.  add the results from (2) and (3) to yield $\log \pi (a \| s)$

This then gives the appropriate engineering implementation with numeric stability.

Note the derivation for $\log(1- \text{tanh}^2(x))$ is shown below (from rlkit documentation, in `torch/distributions.py`):

```
log(1 - tanh(x)^2)
= log(sech(x)^2)
= 2 * log(sech(x))
= 2 * log(2e^-x / (e^-2x + 1))
= 2 * (log(2) - x - log(e^-2x + 1))
= 2 * (log(2) - x - softplus(-2x))
```

How do we handle this in the deterministic setting?
