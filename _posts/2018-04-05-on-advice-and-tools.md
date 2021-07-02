---
layout: post
category : 
tags : 
tagline: 
---


How did I learn programming and machine learning? What tools should you learn? What tools should I use?

A lot of this advice is difficult to give - simply because the landscape has changed in such a short time, and also the path that I decided to take. 

For example:

*  Previously, I would recommend workflows using tools like `luigi`, or `airflow` or even rolling your own using `doit` or `go` - now I think plain old `make` is better. However, who is to say that I won't recomment something else in the future?
*  I've gone from thinking on workflows, ranging from `R`, `Scala`, `Python`, `SQL` and the associated orchestration, to thinking we should just stick with plain old `Python` and leverage something well engineered over something that is "fast" (Spark)
*  Perhaps I would have recommended learning functional programming at one stage, because I felt it was going to take over - now I'm not too sure (though I still think you should for pedagogy reasons)
*  Once I felt C++ was going to be key, leveraging tools like `Rcpp` or `Cython`. Now I think learning plain old `Java` makes more sense from an enterprise perspective - or even plain old C language instead. Though maybe all of it isn't needed when you have abstractions through Tensorflow or Spark.

What does this all mean?

Either the landscape is changing before me, or I'm becoming old. Old to trust in enterprise grade tools which others have used and recommended. The value now is more in stability and ability to integrate over what is new and flashy. Which leads to, why am I even writing this post?

Simply as a reminder in the future where I've come from, what I've learnt and where I'm going. 

Perhaps the ultimate lesson after all is:

*  Learn all that you can
*  Rely on old and trusted technologies (spirit of anti-fragility)
*  Value integration over all things else

With this tooling should be reliant on:

*  `make` - for orchestration of all things (especially for batch processes and testing)
*  Use *something* for environment management. `conda` or `virtualenv` for simple projects, `docker` for industrial grade production
*  Learn DevOps tools, such as ansible - you will thank yourself

With Data Science:

*  Focus on building reproducible pipelines. This is what makes it to production
*  The act of machine learning is becoming more and more commoditized. Personally I feel using AutoML from H2O solves almost all problems with limited configuration. This does not disregard the need to learn and understand good analytics - however it'll be almost silly not use tools like AutoML. 

How to study:

*  Read code: no reason to at least familiarise yourself with [The Architecture of Open Source Applications](https://github.com/aosabook/500lines)  
*  Reimplement algorithms and understand engineering considerations (e.g. neural networks and tree models)
*  Have your mind on deployment - a data scientist who can't deploy wouldn't have demonstrated their ability!






