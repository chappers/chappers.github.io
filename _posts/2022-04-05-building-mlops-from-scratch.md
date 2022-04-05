---
layout: post
category:
tags:
tagline:
---

What would you need to do to build a functional MLOps pipeline, from scratch?

What would it look like, if you took away all the organisational "niceities" that you're used to?

Here is my rough run down:

1. You need to have access to a production-grade kubernetes environment. If its airgapped, you're going to have a bad time. Maybe that is a topic for another discussion - how do I deploy things in an airgapped environment?
2. You would probably need to repackage your deployments to the minimal level infrastructure components, which you can confidently manage yourself! For me this probably means: a `python` webserver with a database (`postgresql`) and task queue for asynchronous workflows (`celery` and `redis`). Of course you may want to have `elasticsearch` rather than `postgresql` depending on the problem at hand, this setup is generic enough that you could even deploy Feast as-is, with `postgresql` as the online and offline store as an example.
3. You would probably require some kind of ETL workflow, I've been thinking `dagster` is a good candidate for this, as it enabled separation between the scheduler and the worker images in a clean manner. There are other alternatives such as `airflow`, `luigi`, `prefect` that could fit the bill here as well.

You could do model training via Celery with Feast + Postgres as the online and offline store. This is something I hope would mature in the long run (fingers crossed)