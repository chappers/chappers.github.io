---
layout: post
category : 
tags : 
tagline: 
---

Human beings are always lazy, and nature always has in it really interesting and random phenomemon. In this post we will explore a few key ideas:

*  What if the Pareto Principle holds?
*  Rule of 5

Pareto Principle
================

Pareto Principle is around the idea that life and nature generally adheres to a 80:20 ruleset. That 80% of the effect can be explained by 20% of the cause. But what if we want to know 90%, 70% or 50% of the effect - what is the corresponding proportion of cause?

To understand this, we will approximate this through a power-law distribution

```r
pareto_principle <- function(x){
  return (x ^ (log(0.2)/log(0.8)))
}

sprintf("%.2f of the effort comes from %.2f of the causes", 0.9, pareto_principle(0.9))
sprintf("%.2f of the effort comes from %.2f of the causes", 0.75, pareto_principle(0.75))
sprintf("%.2f of the effort comes from %.2f of the causes", 0.5, pareto_principle(0.5))
```

Roughly speaking we would see 90% effect explained by 50% of the effort, 75% explained by 15% and 50% explained by 1%. What does this all mean? Roughly speaking you probably need to work far less than you think to come up with a reasonable estimation. 

Rule of 5
=========

How little, you might ask? In Hubbard's book, "How to measure anything", he explains the "Rule of 5", where if you have a random sample of 5 numbers, you have sufficient data to provide a 93% confidence interval around the median! How does this work?

When choosing the median, it means that we have 50% chance being over or under the number. This is a binomial distribution! Recognising this, we can quickly construct the appropriate confidence interval and understand the bounds.

```r
dbinom(0, 5, 0.5)   # 0.03125
1-dbinom(5, 5, 0.5) # 0.96875
```

Now with these two ideas in tandem you can never say you can't measure anything! Though we still have to be wary of induction fallacy...

>  Consider a turkey that is fed every day. Every single feeding will firm up the bird's belief that it is the general rule of life to be fed every day by friendly members of the human race "looking out for its best interests," as a politician would say. On the afternoon of the Wednesday before Thanksgiving, something unexpected will happen to the turkey. It will incur a revision of belief. - Nassim Taleb, The Black Swan: The Impact of the Highly Improbable




