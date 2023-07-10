---
layout: post
category: web micro log
tags:
---

What about regression trees?

There are implementations in [SciPy](http://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeRegressor.html) and different R libraries (for example [rpart](https://cran.r-project.org/web/packages/rpart/rpart.pdf) but how do they actually work intuitively compared with decision trees?

In general the implementation is based on the famous CART (classification and **regression** trees) and work through using [recursive partitions](http://www.stat.cmu.edu/~cshalizi/350-2006/lecture-10.pdf) to seperate your continuous response. Once the stopping criterion is reached it will use [local regression](http://scikit-learn.org/stable/auto_examples/tree/plot_tree_regression.html) techniques to finally predict your answer. Essentially they created through piece-wise linear models.

In this manner, decision trees can be modified for a continuous response.
