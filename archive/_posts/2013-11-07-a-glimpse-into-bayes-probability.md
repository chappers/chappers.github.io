---
layout: post
category: proofs
tags:
---

Let's firstly think what is the probability of two events happening. Lets say

- Coin toss is heads, \\(P(Heads)\\)
- It will rain, \\(P(Rain)\\)

Now these two events are **independent** of each other, so the probability of both occurring is:

$$P(Heads \cap Rain) = P(Heads)P(Rain)$$

**But**!! The probability of two events happening is _not_ always this simple! Lets take another two events:

- Next person you meet is male, \\(P(Male)\\)
- Next person you meet is wearing a dress, \\(P(Dress)\\)

The probability of this happening is not independent, since we know (or at least have a _prior belief_) that there are not too many males who wear dresses. That is the essense of Baysian probability; that you have a prior belief (or evidence) on how an underlying should behave.

Baye's theorem is typically expressed like this

$$ P(Male \cap Dress) = P(Male)\times P(Dress | Male) $$

or by symmetry

$$ P(Male \cap Dress) = P(Dress)\times P(Male | Dress) $$

#### Side Note

$$P(A | B) \neq P(B | A)$$

Lets say (using a now defunct wikipedia example, thanks to [Think Bayes](http://www.greenteapress.com/thinkbayes/))

- We have two bowls of cookies
  1.  Bowl 1 :
      - 30 Vanilla
      - 10 Chocolate
  2.  Bowl 2 :
      - 20 Vanilla
      - 20 Chocolate

If we choose one of the bowls at random, and then one cookie at random, then we can then see

$$ P(Vanilla | Bowl 1) = 3/4 $$

But if we move it around \\( P(Bowl 1 | Vanilla) \\), not only is it not obvious how to compute this, but it would not be \\(3/4\\).

$$ P(Bowl 1 | Vanilla) = \frac{P(Bowl 1) P(Vanilla | Bowl 1)}{P(Vanilla)} = 0.5 \times 0.75/0.625 = 0.6 $$

_next time: bayesian inference_
