---
layout: post
category:
tags:
tagline:
---

Kedro is an interesting project which I've kept tabs on here and there.

It is about time that I jot down some initial impressions and things to consider now and the future.

Things I don't like:

- Not a fan of the Kedro templates. I understand why its done (and probably has immense value in large teams) - I personally wouldn't use it except to provide an opinionated boilerplate when we can't do better or when we can't decide
- The CLI hides a bit too much complexity. Its a bit magical and difficult to debug.
- Documentation - the documentation is geared towards citizen data scientist, not to engineers. I wish there was a greater focus on how to "pick and choose" parts of Kedro to keep and not to keep
- Data Catalog - mixed feelings about it. I understand why and can see the benefits, the actual implementation leaves much to be desired from an ML Ops perspective.

Things I like:

- The `scikit-learn`-esque interface. I believe it does do a really good job bridging the data science "feature store to model" gap, particularly when things don't quite fit in memory and require some kind of preprocessing.
- The pipeline runners is a really great idea and the implementation makes perfect sense

Things I need to explore more of:

- Kubernetes integration. This is probably related my lack of knowledge in the k8s space rather than Kedro itself
- Dask/cluster compute integration. Is this possible? Or is the only cluster integration the accessing of data, and it defers to in-memory Pandas for everything else?
