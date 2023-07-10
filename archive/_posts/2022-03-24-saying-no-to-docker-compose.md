---
layout: post
category:
tags:
tagline:
---

I'm saying no to docker compose. Simply because I can't take it to production on the seemingly industry standard of Kubernetes. 

To break down "why", I've increasingly found that the standard ML deployment consists of something like:

* Celery worker - for async tasks
* Celery beats - for cron jobs
* Redis - as a broker
* Web server - for triggering and serving models

Each of these components (in production) have to be scaled up and down, and whilst Docker Compose gives you an environment to spin these up easily, you end up with two sets of things to maintain (sort of). This "hand-off" becomes problematic when you don't have a devops team to handle alot of the complexity for you.

(there is a bit of irony where DevOps exist to remove developer's control of end to end rather than being able to manage infra and code altogether - discussion for another day)

What has been increasingly clear to me is also the complexity which arises in working with infra, where Helm is the defacto standard. `skaffold` is a really cool tool which has helped alleviate some of the pain, I just wish there was something _more_. 

I'm still going to search and think about it a bit more, and maybe I'll post my actual set up in a future writing.