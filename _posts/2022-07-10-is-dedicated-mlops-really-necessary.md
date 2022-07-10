---
layout: post
category:
tags:
tagline:
title: Is MLOps Specialisation Really Necessary?
---

One observation is MLOps seems to be overly complicated. Its difficult to self-host the "open-source" tooling that is availabe, and most of them are just overly complicated. CML actually is the closest to getting things right, but depends on DVC which is "okay" but not great depending on the enterprise environment you're in.

Maybe the thought is to just let data scientist own DevOps - their own deployments! Rather than the current pattern which is to stand up architecture, applications which abstract things out for them. I think there is value is relying on SaaS solutions - but only if that is the path to production. Otherwise emphasising skills and ideas around ownership will take the team further. 

At the end of the day - what is actually needed?

1. A DevOps pipeline that triggers a "job" on a PR (e.g. train a model in staging)
2. A DevOps pipeline that publishes (i.e. writes the model) to an artifact store upon merge/push to production
3. A method for logging model serving requests (e.g. to a task queue)
4. A method for performing audit and evaluating model performance

These are all readily achieveable, if only we trust our data scientists to own their own production workflows. 