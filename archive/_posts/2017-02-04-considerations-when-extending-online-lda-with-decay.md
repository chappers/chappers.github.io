---
layout: post
category: web micro log
tags:
---

[Gensim](http://radimrehurek.com/gensim/models/ldamodel.html) has a lovely online LDA module which works extremely well. One of the challenges when placing it in a modelling environment is the assumption that the topics shouldn't change _too_ much over time. This assumption when you have a massive dataset is more than reasonable, however sometimes when you are just starting to build the model out you don't really have "good" data, hence the need to retrain and refit the dictionary.

This means that it might be ideal to keep a history of all the documents fitted. Overtime this number might get a bit ridiculous, as you will have too many documents to keep in memory. This means you will either keep the last X number of documents, where X is either fix or a percentage.

On a practical note, what does having a fix or a percentage mean? If it is for a single application and the model code will not be reused for other use cases perhaps having a fixed number will be beneficial; in this sense you will always know how this should or shouldn't work.

On the other hand, if you want to reuse this model for many different applications perhaps having a decay percentage is ideal. In this setting we have to be careful of the maximum possible number of documents we will keep in memory. As a rule of thumb this number would be $\frac{1}{1-x}$ where $x$ is a percentage of the documents you wish to keep in each batch update. In other words if we have a rate of $0.9$ then we will probably expect in the long run $10\times n$ documents, where $n$ is our average number of documents in every batch.

For the actuaries this is precisely the perpetuity annuity formula, for others the intuition behind this formula is we are aiming to see what the steady state is; that is where the number of new documents will cover the number of documents lost. If the decay rate is $0.9$, then we will lost $10\%$ of our documents on every batch run, which means we will have $1/0.1 = 10$ times the number of documents in the long run. I thought this was pretty neat!
