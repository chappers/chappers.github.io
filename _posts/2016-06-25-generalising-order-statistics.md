---
layout: post
category : web micro log
tags :
---

Order statistics problems are very common in 2nd year statistics courses. Often they come in the following form:

>  Suppose \\(X_1, X_2, X_3, ..., X_k\\) is iid by some distribution \\(F\\). What is the pdf of \\(Y = max(X_1, X_2, X_3, ..., X_k) \\)?

This problem is generally solved as follows:

$$ P(Y \leq x) = P(X_1 \leq x, ..., X_k \leq x) = F_X(x)^n $$

How can we generalise this?

The simple answer is using our understanding of combinatorics. If \\(X_(k)\\) is the \\(k\\) order of statistic,
then:

$$ P(X_{(k)} \leq x) = \sum_{n=k}^n {n \choose m} F(x)^m (1-F(x))^{n-m} $$

Since we can view the combinatorics, in the form that \\(P(\|\{i : X_i \leq m\}\| =m) = {n \choose m} F(x)^m (1-F(x))^{n-m} \\)

This then leads me to why I was thinking about this question in the first place...

---

Suppose we have

$$ A = X_{(m-p:m)} $$
$$ B = Y_{(n-p:n)} $$

Where \\(X, Y \\) are iid with some distribution \\(F\\), and we can assume without loss of generality \\(m \geq n \geq p\\).

Then what is \\(P(A \geq B)\\)?

Before I begin to answer this question, the motivation was quite simply this:

*  Many companies have diversity hiring policies, which follow a line of selecting the top \\(p\\) candidates of both genders. Yet on consistent (anecdotal) evidence is that the majority gender always seems to consistently be stronger than the minority gender in the workplace. How much of this can be attributed to mere chance? (Note that this is based on my small sample size of experiences)

## Convolutions

To answer \\(P(A \geq B)\\) one might already recognise this to be a convolution. We could simply rewrite this equation to be:

Let

$$Z = A - B$$

Then we are after \\(P(Z \geq 0)\\). Then what is the pdf of \\(Z\\)?

$$ f_Z(z) = \int_{-\infty}^{\infty} f_X(x) f_Y(z-x) dx $$

Which unfortunately can't be simplified anymore.

## Simulation

Using simulation has always been a quick way of getting a feel for these kind of probabilities.

```r
sim <- function(nx, ny, tailorder=c(3,2), rdist=rnorm) {
  xsample <- rdist(nx)
  ysample <- rdist(ny)

  xsample <- rev(xsample[order(xsample)])
  ysample <- rev(ysample[order(ysample)])

  return(xsample[tailorder[1]] > ysample[tailorder[2]])
}

table(unlist(Map(function(...) sim(24, 16), 1:10000)))/100
#
# FALSE  TRUE
# 52.95 47.05
table(unlist(Map(function(...) sim(6, 3, tailorder=c(3,1)), 1:10000)))/100
#
# FALSE  TRUE
# 76.16 23.84
```

How do we interpret the above results?

The first one, `table(unlist(Map(function(...) sim(24, 16), 1:10000)))/100` simulates
the number of times the "3rd best" in the sample of size 24 is higher than the "2nd best"
in the same of 16. Here we can see that it is better 47% of the time.
