---
layout: post
category:
tags:
tagline:
---

I've been thinking a lot about research and the attempt to solve practical problems (after all that is in a way the goal of _my_ PhD). The questions come around what is the easiest way to gain acceptance or to ensure other people _use_ what you are going to build?

Now part of this is undoubtedly my own world view and not necessarily the view of my supervisor or company, because personally I believe several facets determine whether **I** personally believe something is worthwhile.

- Is it _obvious_ to use and install?
- Is it easy to understand?

## Principle of Least Astonishment

Algorithms can be extremely complex. Perhaps the reason why libraries and wrappers such as Keras work so well is that it makes deep learning easy to understand. One of the greatest strengths about scikit learn library from my perspective is that the interface abstracts away so that any user will know that `fit` trains a model and `transform/predict` will provide the output.

Hence whatever research I provide must be easy to use; ideally leveraging existing frameworks to provide value to the user.

## On Algorithms

How important is an algorithm to the implementation? If we provide an MCMC approach, can we be sure that a general user can understand it? In some instances it doesn't matter. If we can provide a simple interface to an approach akin to LDA models or gaussian mixture models, it will be "obvious" how it is used.

In this setting it will break down to:

1.  Choose number of clusters
2.  Fit clusters (in some magical way)
3.  Return object

## Closing Thoughts

From a practical standpoint, so long as the interface is obvious, and the expected input and output of the algorithm is clear, then the research is probably worth while. From my perspective this is where reproducible search makes this complicated - how can we reuse algorithms and approaches from papers where the code isn't provided? Or if it is provided on non-free software?

Perhaps the forefront of research should not only be on expanding knowledge, but also pushing it towards acceptance to the wider world.

## Combining the Two
