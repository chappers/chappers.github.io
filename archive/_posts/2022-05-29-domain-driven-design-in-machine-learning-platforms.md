---
layout: post
category:
tags:
tagline:
---

One of the critical challenges in actually deploying machine learning applications is the development of a suitable framework and platform around it. How do we ensure each part of the model _development_ process is adequately replicated in the model _deployment_ process?

Domain-driven design is such solution to this issue. In this particular, domain-driven design enables deployments through a well-defined bounded context between data science model, and the additional auxiliary servicess, such as the data engineering or feature engineering, integration with data lake for auditing purposes and a decision platform when combining and optimising business processes.

This enables greater separation across the different parts of an AI application:
* feature stores
* model deployments
* decision engines

The alternative approach to deploy a "big bang" application where a single person (or team) owns the whole pipeline, which typically slows down the process and prevents multiple users deploy models or reuse components as there is no well defined interface or contract to enable multiple models, features or decision rules to co-exist.

We can separate an a service deployed in this context via:

* api interfaces/handlers
* services
* entities

where enforcement of the so-called onion architecture can be enforced through imports and inheritance. Although verbose and takes more time to setup this approach enables adding and removing machine learning models in a way that we can deploy with a high degree of confidence and reuse (so long as various conventions around reuse are followed).

...more to come in the future