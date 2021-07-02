---
layout: post
category : proofs
tags : 
---



Following on from a previous [post](http://charliec443.github.io/proofs/2013/11/07/a-glimpse-into-bayes-probability/), lets look into
Bayesian inference.

## Recap

Recall that under Bayesian probability which have this formula.

$$ P(A \cap B) = P(A)\times P(B | A) $$ 

We can rearrange this:

$$ P(A | B) = P(A)\times \frac{P(B|A)}{P(B)} $$ 

Now we have the equation for Bayesian inference.

---

The central idea for Bayesian inference is this:

$$ posterior \propto prior \times likelihood $$

Right now all these things don't really make sense, so lets revisit the equation we have above. Instead of just having
events \(A\) and \(B\), we can instead use _something_ more meaningful.

$$ P(Truth | Evidence) = P(Truth)\times \frac{P(Evidence | Truth)}{P(Evidence)} $$ 

Here we can say that \(P(Truth\) is the _prior_, \(\frac{P(Evidence | Truth)}{P(Evidence)} \) is the _likelihood_, and
\(P(Truth | Evidence)\) is the _posterior_.

The idea in Bayesian inference is that the _posterior_ is proportional to combining _prior_ and _likelihood_. When making comparisons we don't really care about absolute amounts, but instead only the relative difference when making inference.

## A Simple Example

Lets suppose we have a medical test. 

-  The probability of having a disease is 10% (this is our _prior_)
-  Having a **positive** test and having disease is 95% (this is our _likelihood_)
-  Having a **positive** test and not having disease is 20% (this is our _likelihood_)

So then for our _posterior_ (which is the probability of having the disease given evidence of a positive test)

<table>
<tr><th>Prior Scenario</th><th>Prior</th><th>Likelihood</th><th>Posterior (proportional to)</th></tr>
<tr><td>Have Disease</td><td>0.1</td><td>0.95</td><td>0.095</td></tr>
<tr><td>Don't Have Disease</td><td>0.9</td><td>0.2</td><td>0.18</td></tr>
</table>

Looking at these numbers we can see that it is still more likely that you don't have a disease after getting a positive result (since \(0.18 > 0.095\)).




