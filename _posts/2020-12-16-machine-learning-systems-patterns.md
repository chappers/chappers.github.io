---
layout: post
category : 
tags : 
tagline: 
---


In this post we'll provide some heuristics for productionising machine learning patterns. 

What Latency?
-------------

The first question to ask is what is the latency that we're after?

From an "API" perspective

1.  If sub 50ms, we probably need to use a deployment method which is built-in a vendor product. This could be pre-computed scores which are injected into the product, or a model deployment mechanism that is in the product.
2.  If 50ms - 1 second, you can probably get away with an "inline" model, which presumes the caller provides all the necessary information to make a decision or make use of precomputed scores which are retrieved when called. If an inline model is used, then the machine learning system needs to be coupled with the calling system which is typically less than desireable. 
3.  If 1 second to minutes, we should make use of asynchronous patterns, whether it be polling or callback. In this scenario, the setup would be that the caller provides an entity reference identifier, which we can then orchestrate to receive the "latest information" of that entity before making a decision. If implemented correctly, this approach can be decoupled from the callilng system.
4.  If results are measured in minutes to hours, generally we may prefer a cron-job approach (e.g. Apache Airflow), rather than coupling our machine learning platform with the integrating system, as this approach is generally the cheapest.

From a "stream" perspective, the differences then lies with the event-driven approach which we expect data and model outputs to flow. One could overload a stream (and maybe desireable from a message queue setting) to use the asynchronous pattern above, as it completely decouples the systems - though this kind of approach isn't always possible. Let's break down some scenarios here:

*  In a consumer-producer pattern, this aligns most closely with the inline-model pattern, as an event needs to happen which then triggers scoring. However the complexity typically in this setting is that the streaming platforms do not completely support all kinds of ML deployments. This approach is most similar to item (2) in the API patterns
*  The other approach is setup a separate scoring platform (e.g. PySpark) which then hooks into the streaming platform. This is similar to the async approach (depending on the setup), primarily because then suddenly this cluster could theoretically do anything you want it to, beyond just looking at things in the immediate stream. 

Both of these patterns presuppose that one is scoring things one entity at a time - but there are still instances where batch scoring is preferable; in which case, a better design pattern is treating the batch scoring as an ETL job in the workflow. 

What Price or Coupling?
-----------------------

Coupling systems generally increases ownership and cost. This is typically the limiting factor in deployments. Infact a closer examination of platforms like Tecton, Michelangelo, Hopsworks, reveals a trend that is to prefer precomputed features as a way of decoupling data from machine learning systems and their downstream integrations. 

The assumption here is that "if the feature data can be produces and arrives fast enough - then the machine learning system can retrieve the latest data based on 'best efforts' and it will be good enough for scoring". This assumption; if implemented correctly, can indeed dramatically reduce costs (or rather shift all the cost to a feature store); however it may not solve all problems all at once (though it would arguably solve over 80% of all problems). 

Generally the greater the coupling the lower the latency but the price would also increase exponentially; that too is a consideration in what we do. 

Is there some kind of half-way mark?
------------------------------------

In some deployments, there are in-betweens which we need to consider. For example, if you were to deploy an information retrieval problem, you may process a bunch of information "offline", and do comparisons "online" only with the pre-processed data; rather than trying to do all processing at once for retrieving similar results as an example. 

Let's play out this deployment example

>  How would I retrieve similar questions in stack overflow?

1.  Compute the vector of: tags, questions, body of the question on stack overflow and save periodically
2.  When a new question comes in, measure the simularity of the tags, questions, bodies with what is computed in 1), weighted by other factors (such as number of votes), and rank the results accordingly

In this kind of deployment, similar questions which have been asked all at once, would not be flagged together. We get around this from a practical standpoint, by weighing similarity based on popularity of the question anyway, so from a design perspective it would look "real-time" all the time. 

This is an example of where pre-computed information and inline scoring are done in conjunction. 

