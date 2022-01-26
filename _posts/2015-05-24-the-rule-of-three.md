---
layout: post
category: web micro log
tags:
---

# or what in the world is \\(-log(0.05)\\)

If you type into a calculator \\(-log(0.05)\\), you will realise this number is approximately three. This is basically where the rule of three within statistics comes from.

This rule comes from the binomial proportion confidence interval, where we may want to calculate the confidence interval around the hyperparameter \\(p\\). However in the case where \\(\hat{p} = 0 \\), what can we do?

Since we have 0 actual cases, we then know the probability of this occuring is:

$$ (1-p)^n = 0.05 $$

If we based on experiments we have \\(n\\) trials, and can safely assume a distribution \\(B(n, p)\\). Rearranging and using the information above, we would then have:

$$ n log(1-p) = 3 $$

or in other words:

$$ log(1-p) = \frac{3}{n} $$

which then leads to the confidence interview for \\(p\\) being, \\( [0, \frac{3}{n}] \\).
