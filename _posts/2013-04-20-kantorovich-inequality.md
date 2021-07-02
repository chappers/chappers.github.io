---
layout: post
category : proofs
tags : [proofs]
tagline: 
---

Kantorovich Inequality is used to show linear convergence for steepest descent algorithm (for the quadratic case). This result is important in some optimization algorithms.

---

## Kantorovich Inequality

Suppose \\(x_1 < x_2 < ... < x_n \\) are given positive numbers. Let \\(\lambda_1, ... , \lambda_n \geq 0\\) and \\(\sum_{i = 1}^n \lambda_i = 1\\) then 

$$ \left( \sum_{i = 1}^n \lambda_i x_i \right) \left( \sum_{i = 1}^n \lambda_i x_i^{-1} \right) \leq \frac{1}{4} \frac{(x_1 + x_n)^2}{x_1 x_n}$$

## Proof

(Anderson)

Note that \\(\sum_{i=1}^n \lambda_i = 1 \\) then the equation can be expressed in terms of expectation.

$$ \left( \sum_{i = 1}^n \lambda_i x_i \right) \left( \sum_{i = 1}^n \lambda_i x_i^{-1} \right)  = E(X) E(1/X) $$

where \\(E\\) is the expected value of the random variable \\(X, \frac{1}{X}\\), with probabilities \\(\lambda_1, ..., \lambda_n\\), with the values of X such that \\(0 < x_1 \leq X \leq x_n \\).

For \\(0 < x_1 \leq X \leq x_n \\)

$$ 0 \leq (x_n - X)(X- x_1) = (x_n + x_1 - X) X - x_n x_1 $$

which implies 

$$ \frac{1}{X} \leq \frac{x_1 + x_n - X}{x_n x_1} $$ 

So then 

\\(\begin{align}
 E(X) E(1/X) & \leq E(X) \frac{x_1 + x_n - E(X)}{x_n x_1} \\\\\\\\
 & = \frac{1}{4} \frac{(x_1 + x_n)^2}{x_1 x_n} - \frac{1}{x_1 x_n } (E(X) - \frac{1}{2}(x_1 + x_n))^2\\\\\\\\
& \leq \frac{1}{4} \frac{(x_1 + x_n)^2}{x_1 x_n} 
\end{align}
\\)

