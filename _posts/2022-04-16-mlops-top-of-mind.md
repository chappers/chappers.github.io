---
layout: post
category:
tags:
tagline:
---

I've been thinking about "production grade" things recently, specifically the libraries we use. In no particular order:

* [FastAPI-users](https://github.com/fastapi-users) is really cool, but I wish [Piccolo](https://github.com/piccolo-orm/piccolo_api) was more popular, because wrapping any kind of Starlette application is really powerful.
* [k9s](https://k9scli.io/) and [skaffold](https://skaffold.dev/) are truly my daily go-tos
* I wish sqlite was used more in production with [litestream](https://litestream.io/) as examples. (More specifically, could we just have litestream baked in as default databases rather than Postgres for Helm charts?) It would just make dev to production much quicker.
* Vector databases are still very young, but I really like [Weaviate](https://weaviate.io/), in particular how similar it is to Elastic. Milvus on the other hand is really painful to develop on.
* There isn't enough resources on enterprise software development that is accessible. I wish the field was more developed so people would talk the same language at least. Domain driven developments with Service Layers at the forefront really should be the standard go-to pattern for Machine Learning applications.

One other topic that isn't broached enough is hosting model APIs in a sensible way in Kubernetes when a centralised decision platform is required. I don't think there is a one-size-fits all solution (and indeed, there are many vendor specific solutions), what I'm concerned about is the lack of open-source or "common language" approaches for this problem, despite being very common. One reason for this is due to the mixing of statistical concepts (e.g. experimental design) with software concepts (feature flags) often make it difficult to have some common ground. Solutions are typically too biased on direction or the other. 


