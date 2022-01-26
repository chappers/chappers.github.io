---
layout: post
category: web micro log
---

[k-means clustering](http://en.wikipedia.org/wiki/K-means_clustering) is a popular cluster analysis method. When using it the question might arise like, does the algorithm always converge? Does it converge to the _global_ minima (it doesn't, but it does converge to the _local_ minima). Here, I will take a brief look only at the convergence of k-means.

Suppose a single cluster \\(C\\) is assigned representative \\(z\\). The cost is then

$$ cost(C; z) = \sum\_{x \in C} ||x-z||^2$$

and

$$ cost(C*1, ... , C_k; z_1, ... , z_k ) = \sum*{j=1}^k \sum\_{x \in C_j} || x - z_j ||^2 $$

for the k-means algorithm

# Bias-Variance Decomposition (for random variables...)

Let \\(X \in R^d\\) be any random variable. For any \\(z \in R^d\\),

$$ E(||X-z||^2) = E(||X- E(X)||^2) + ||z - E(X)||^2 $$

**Proof**

$$
\begin{aligned}
E(||X- E(X)||^2) + ||z - E(X)||^2 & = E(||X||^2 + ||E(X)||^2 - 2X E(X)) + (||z||^2 + ||E(X)||^2 - 2 z E(X)) \\\\\\\\
& = E(||X||^2) + ||E(X)||^2 - 2 E(X) \times E(X) + ||z||^2 + ||E(X)||^2 - 2z \times E(X) \\\\\\\\
& = E(||X||^2) + ||z||^2 - 2z E(X) \\\\\\\\
& = E(||X-z||^2)
\end{aligned}
$$

---

We can rearrange the above and using the definition of \\(cost(C; z)\\) which we defined above:

$$ E(||X-z||^2) = \sum\_{x \in C} \frac{1}{|C|} ||x-z||^2 = \frac{1}{|C|} cost(C; z)$$

and

$$ E(||X- E(X)||^2) + ||z - E(X)||^2 = \frac{1}{|C|} cost(C; mean( C )) $$

Hence we can conclude that

$$ cost(C; z) = cost(C; mean( C )) + |C| \times || z - mean( C ) ||^2 $$

# Convergence

To demonstrate convergence for the k-means algorithm, we must show that the cost monotonically decreases.

Let \\(z*{1}^{(t)}, ... , z*{k}^{(t)}, C_1^{(t)}, ... , C_k^{(t)} \\) denote the centers and clusters at the start of the \\(t\\) iteration of k-means. Since the first part of the iteration assigns each data point to its closest centre, therefore

$$ cost(C_1^{(t+1)}, ... , C_k^{(t+1)}; z_1^{(t)}, ..., z_k^{(t)}) \le cost(C_1^{(t)}, ... , C_k^{(t)}; z_1^{(t)}, ..., z_k^{(t)})$$

Then using the result above, on the next step each cluster is re-centred around its own mean. So

$$ cost(C_1^{(t+1)}, ... , C_k^{(t+1)}; z_1^{(t+1)}, ..., z_k^{(t+1)}) \le cost(C_1^{(t+1)}, ... , C_k^{(t+1)}; z_1^{(t)}, ..., z_k^{(t)})$$

<!-- http://cseweb.ucsd.edu/~dasgupta/291/lec2.pdf -->
