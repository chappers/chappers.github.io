---
layout: post
category : 
tags : 
tagline: 
---

In this post we'll talk briefly about the `P2` algorithm, which will allow one to calculate percentiles on a stream (approximately)! 

This post is more about implementing the algorithm, and I'll leave the discussion around what it does to a later stage. 

Implementation
==============

For percentile, which isn't the minimum of maximum, we require keeping tabs of 5 markers:

*  The minimum
*  $p/2$ quantile
*  $p$ quantile
*  $(1+p)/2$ quantile
* The maximum

```py
```

In this format, we need to retain a few items:

*  marker heights
*  marker positions
*  desired marker positions

Because we have 5 marker points, then the algorithm only kicks in when we have at least 5 points. 

```py
import numpy as np

class ApproxQuantile1D(object):
    # applies approximate quantile assuming the input object is a 1d list

    def __init__(self, p=50):
        self.p = p
        self._p = p/100  # this is so it is the same definition in numpy

        ## initialisation is set for when count is < 5 numbers
        self.n = None  # marker position
        self.ns = None  # desired marker position
        self.dns = None
        self.q = None  # marker heights
        self.reset()

    def reset(self):
        self.n = [0, 1, 2, 3, 4]  # initialise marker positions
        self.ns = [0, 2*self._p, 4*self._p, 2+2*self._p, 4]  # initialise desired marker position
        self.dns = [0, self._p/2, self._p, (1+self._p) /2, 1]
        self.q = []

    def fit(self, X):
        self.reset()
        return self.partial_fit(X)

    def partial_fit(self, X):
        for x in X:
            self.partial_fit_(x)
        return self

    def partial_fit_(self, x):
        # assume x is a single number
        # part A initialization
        if len(self.q) < 5:
            self.q.append(x)
            self.q = sorted(self.q)
            return self
        
        # part B
        if x <= self.q[0]:
            self.q[0] = x  # new min
            k = 0
        if x >= self.q[4]:
            self.q[4] = x
            k = 3
        elif x < self.q[1]:
            k = 0
        elif x < self.q[2]:
            k = 1
        elif x < self.q[3]:
            k = 2
        elif x < self.q[4]:
            k = 3
        else:
            raise Exception("All cases should have been covered?")
        
        # item 2 in algorithm - increment markers
        for i in range(k+1, 5):
            self.n[i] += 1
        for i in range(5):
            self.ns[i] += self.dns[i]

        for i in [1, 2, 3]:
            d = self.ns[i] - self.n[i]
            if (d >= 1 and self.n[i+1] - self.n[i] > 1) or (d <= -1 and self.n[i-1] - self.n[i] < -1):  # item 3 in algorithm
                d_sign = np.sign(d)
                q_para = self.parabolic(i, d_sign)
                # q_linear = self.linear(i, d_sign)
                if (self.q[i] < q_para) and (q_para < self.q[i+1]):
                    self.q[i] = q_para
                else:
                    self.q[i] = self.linear(i, d_sign)
                self.n[i] += d_sign
        return self
    
    def parabolic(self, i, d):
        # copy from paper
        i = int(i)
        d = int(d)
        return self.q[i] + ((d/(self.n[i+1] - self.n[i-1])) * (
            (self.n[i] - self.n[i-1] + d) * ((self.q[i+1] - self.q[i])/(self.n[i+1] - self.n[i])) + (self.n[i+1] - self.n[i] + d) * ((self.q[i] - self.q[i-1])/(self.n[i] - self.n[i-1]))
        ))
    def linear(self, i, d):
        i = int(i)
        d = int(d)
        return self.q[i] + d*((self.q[i+d]-self.q[i])/(self.n[i+d] - self.n[i]))

    def score(self):
        # returns the percentile
        if self.p == 0:
            return self.q[0]
        if self.p == 100:
            return self.q[4]
        if len(self.q) < 5:
            target_index = int(int(len(self.q) * self._p))
            return self.q[target_index]        
        return self.q[2]

class FiveNumberSummary(object):
    def __init__(self):
        self.summary = [ApproxQuantile1D(0), ApproxQuantile1D(25), ApproxQuantile1D(50), ApproxQuantile1D(75), ApproxQuantile1D(100)]

    def fit(self, X):
        for s in self.summary:
            s.fit(X)
        return self

    def partial_fit(self, X):
        for s in self.summary:
            s.partial_fit(X)
        return self

    def score(self):
        return [
            s.score() for s in self.summary
        ]

# examples - its not perfect, and I _probably_ have implemented something wrong
# but its a pretty good start.
X = np.random.normal(size=100)
X1 = np.random.uniform(size=100)
fns = FiveNumberSummary()
fns.fit(X)
print("approx summary for X", fns.score())
print("summary for X", [np.percentile(X, p) for p in [0, 25, 50, 75, 100]])

fns.partial_fit(X1)
fns.score()
X2 = np.hstack([X, X1])
print("approx summary for X2", fns.score())
print("summary for X2", [np.percentile(X2, p) for p in [0, 25, 50, 75, 100]])
```

The original paper "The $P^2$ Algorithm for Dynamic Statistical Computing Calculation of Quantiles and Histograms Without Storing Observations" can be found here: https://www.cse.wustl.edu/~jain/papers/ftp/psqr.pdf

