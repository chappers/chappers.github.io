---
layout: post
category:
tags:
tagline:
---

Are feature stores really that hard (yes they actually are). But putting something with a deployment bias should be plausible in a range of programming languages and higher level architecture infra perspective. In a way it is a bit of a shame feature stores are not more common (from a BYO perspective). 

Feature stores really aren't as difficult as they sound, but it is definitely true that building an effective feature store for pure streaming pipelines is indeed a massive challenge. 

High Level Approach
-------------------

Imagine you're in an environment where you don't have access to Python (for arguments sake, suppose you are building a feature store solution in Rust). How would you build this?

1. Make key assumptions, by presuming there is an ETL pipeline which creates denormalised snapshots at event time
2. When these snapshots arrive, immediately update a key-value store (e.g. Redis) with the whole record

_Note: this approach follows most closely with the kappa architecture rhetoric_

Notice that this makes no assumption on the libraries we would use, or the tooling, besides a generic ETL approach, and a generic key-value store. 

Dealing with Training Data Sets
-------------------------------

The challenge then becomes how do we manage training datasets? 

Since we have a way to pump denormalised data, the key which follows is to consider _conventions_ we can use (such as everything is snapshot daily) to make time-travel joins easier to manage. 

Another convention we could include is explicit avoidance of combining multiple feature tables, instead preferring to defer this task explicitly in the ETL pipeline. This would make the ETL much more computationally heavy, but could be worthwhile in terms of pushing the complexity to more mature ETL tools and teams rather than relying on data scientists to manage this workflow appropriately.

Unfortunately the lack of (general) tooling means that the model development process drastically slows down, as you're reliant on data engineering to solve the problem rather than the data scientists. 

This kind of approach is probably only sensible for training models and then taking it to production in a real-time setting. 

### Dealing With Data Science Workflows

Theoretically this approach enables feature store with batch features only. This could be a sensible gateway between dev and prod as a data science team grows in maturity.