---
layout: post
category : 
tags : 
tagline: 
---

In attempting to write a pure Python sampling version of _determinantal point process_ I thought I'll write through some pseudo-code and go through what worked. I have provided [my current work in progress here](https://github.com/charliec443/dpp). It is based on the [Matlab code of Alex Kulesza](http://web.eecs.umich.edu/~kulesza/), which was converted (partially) using SMOP package. 

## Changes to the Original Implementation

The changes to the Matlab implementation revolve around how to calculate the new orthogonal basis for the subspace relative to the vector which has been chosen. 

Instead we will use the equivalent formulation in the paper ["Sampling from Determinantal Point Processes for Scalable Manifold Learning"](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4524741/), 
where rather than a deletion of the column of the matrix \\( V \\), the projection is simply completed based on the chosen vector as it should guarentee that the same point is not selected twice. (The projection would mean that the probability to be chosen would be 0). 

**k-dpp Algorithm**

1.  Until we have \\( k \\) chosen eigenvector, add an eigenvector with a probability \\( \frac{\lambda}{\lambda+1}\\), where \\( \lambda \\) is the corresponding eigenvalue, and set \\( B \\) to be matrix of eigenvectors. 
2.  Until we have \\( k \\) samples, select a training instance \\( P(i) \propto  \lvert \lvert B(i) \rvert \rvert^2 \\), after each iteration altering the projection of \\( B \\).

