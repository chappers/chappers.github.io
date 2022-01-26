---
layout: post
category: web micro log
tags:
---

Causal inference is concerned with _what would happen_ to an outcome if we _hypothetically_ did something else to it (i.e. treatment). This idea is commonly used in medicine to determine the effect that medicine might have to a person.

It is important to remember that you cannot substantiate causal claims from associations alone - behind any causal conslusion there must lie causal assumptions that is **not** testable in observational studies (i.e. correlation does not imply causation). Often we may miss various important piece of information, which leads to confounding information. This is known as [Simpson's Paradox](http://en.wikipedia.org/wiki/Simpson%27s_paradox).

# In the ideal world...

we already know how to solve this problem! We can rephrase this question by simply saying, what we really want to know is the average difference between performing treatment on one _random_ group of people compared to another. In this instance we simply can perform the [`t-test`](http://en.wikipedia.org/wiki/Student%27s_t-test).

However things are not always that simple in practise, after all, a sample of people could have unique traits. For example, in medical trials you are normally conducting trials on volunteers who possess particular traits. How could you then use this information to generalise to the whole population?

## The easy solution

Indeed the easiest way is simply treat these predictors in a regression model, thus following the general principles of regression modelling.

# Matching

Matching is a statistical technique used to evaluate the effect of a treatment. The goal of matching is, for every treated unit, to find one (or more) non-treated unit(s) with similar observable characteristics against wholm the effect of the treatment can be assessed. Through matching, it enables a comparison of outcomes among treated and non-treated units to estimate the effect of the treatment without reducing bias due to confounding.

## A motivating example

_...from [wikipedia](http://en.wikipedia.org/wiki/Rubin_causal_model)_

Suppose we wanted to know the effects that a drug has on patients, however we can't both run a test case on the patient when they take and don't take the drug, and hence we have imperfect information. Let's say that the drug is for blood pressure, then our data might look like the following:

| Subject | Change under treatment | Change under no treatment |
| ------- | ---------------------- | ------------------------- |
| Joe     | ?                      | -5                        |
| Bob     | 0                      | ?                         |
| James   | -50                    | ?                         |
| Mary    | -10                    | ?                         |
| Sally   | -20                    | ?                         |
| Susie   | ?                      | -20                       |

From the table above, we can then quite simply, assuming that every subject is the same, compute and determine the average causal effect of this drug, we can use the following `R` code to do this:

````r
drug <- as.data.frame(list(change.blood.pressure=c(-5,10,10,-20,5,15),
                           treatment=c(1,1,0,1,0,0)))
ddply(drug, ~ treatment, summarize,
      mean.blood.pressure = mean(blood.pressure))
```

By inspection we would say that the causal effect of taking the drug is a change in blood pressue of \\(-5 - 10 = -15\\).

However we might realise that sex and the starting blood pressure might be important. This is where we would performing matching to perhaps remove any bias.

## Perfectly Matched

Now if we have a dataset which is perfectly matched, we can still create the same conclusions! It would be our goal to somehow create a dataset which looks like the one below:

Subject|Gender|Blood Pressure | Change under treatment | Change under no treatment
-------|------|---------------|------------------------|--------------------------
Joe|male|180|?|-15|?
Bob|male|180|-20|?|?
James|male|160|-10|?|?
Paul|male|160|?|-5|?
Mary|female|180|5|?|?
Sally|female|180|?|10|?
Susie|female|160|5|?|?
Jen|female|160|?|10|?

Again, since it is perfectly matched, we can then simply take the average the same way that we did it above.

```r
drug <- as.data.frame(list(
  sex = rep(c("m", "f"), each=4),
  blood.pressure = c(180,180,160,160,180,180,160,160),
  change.in.bp = c(-15,-20,-10,-5,5,10,5,10),
  treatment = c(0,1,1,0,1,0,1,0)
  ))

ddply(drug, ~treatment, summarize,
      mean.blood.pressure = mean(blood.pressure),
      mean.change.in.bp = mean(change.in.bp))
```

This would assume that the matched units are homogeneous, and that they have the same causal effect. We could think of it as being we have two similar people, what if one was treated and the other was not. This is the idea behind matching. We must be careful though, because we are **not** trying to ensure that each _pair of matched observations_ is similar in terms of their covariate values, but rather that the matched groups are similar _on average_ across all their covariate values.


## Imperfect Match

What about in the case where matching isn't perfect? How would we then approach the matching problem?

We could implement a simple matching algorithm as follows:

1. Take an observations that received treatment
2. Find the closest match from the control group
3. Go back to step 1, removing its match from list of closest candidates

We must first imagine the situation where our datasets may not match. This means that there could be less treatments than control, for example. Lets consider a control dataset with values:

```r
X <- list(x = c(0.1,0.12,0.14,0.17,0.2,0.2,0.25,0.34,0.36), treat = 0)
Y <- list(x = c(0.15,0.19,0.22,0.23,0.31,0.32), treat=1)
```

We can manually match it to the closest value without replacement yielding a group which looks like the following

```
0.15 -> 0.14
0.19 -> 0.2
0.22 -> 0.2
0.23 -> 0.25
0.31 -> 0.34
0.32 -> 0.36
```

Or we can also matched with replacement, which would yield

```
0.15 -> 0.14
0.19 -> 0.2
0.22 -> 0.2
0.23 -> 0.25
0.31 -> 0.34
0.32 -> 0.34
```

```r
X <- as.data.frame(list(x = c(0.1,0.12,0.14,0.17,0.2,0.2,0.25, 0.34,0.36), treat = 0))
Y <- as.data.frame(list(x = c(0.15,0.19,0.22,0.23,0.31,0.32), treat=1))

xdf <- rbind(X,Y)

# manually matching
library(arm)
matched <- matching(xdf$treat, xdf$x)
xdf.m <- xdf[matched$matched,]

matched <- matching(xdf$treat, xdf$x, replace=TRUE)
xdf.m <- xdf[as.vector(matched$pairs),]
```

With these matched groups between treatment and control, we can then go ahead and perform our inference on the now corrected groups.

The common approach is to use propensity score matching in order to build groups of covariates, matching on propensity score in order to have groups with similar odds, which in turn signifies comparable relationships. Of course this all depends on the underlying assumptions which you were to place on each group.


````
