---
layout: post
category: proofs
tags: [proofs]
tagline:
---

> Rigour is lost in the real world. But that does not mean I shouldn't continue to pursue and practise my mathematical skills. Just like a programmer might learn to code by writing programs, to improve my mathematical thinking I must read other people's proofs and **write my own**.
>
> (emphasis mine) [Adapted from "Why there is no Hitchhiker's Guide to Mathematics for Programmers"](http://jeremykun.com/2013/02/08/why-there-is-no-hitchhikers-guide-to-mathematics-for-programmers/).

So perhaps here will be a collection of (mostly other people's) proofs, until I feel confident writing my own. After all, pure mathematics was by far my worse course in university studies...

There will be plenty of mistakes, but regardless. Let us begin.

email [me](mailto:chapm0n.siu@gmail.com?Subjext=Incorrect%20Proof) if you spot something wrong!

---

#Measure Theory

Where else to begin than with my worse subject in university? Real Analysis.

In a subject that essentially everything was rote learnt, and with foundations (sort of, not really in a practical learning sense) in probability theory, this will be my (not necessarily ideal) starting point.

Goal of today: Prove Monotonicity, Subaddivity, Infinite Unions, Infinite Intersections under measure theory. Hopefully later move onto Ham Sandwich Theorem.

## \\( \sigma\\)-algebra (Definition)

Let \\(F\\) be a collection of subsets of a sample space \\(\Omega\\).

Then \\(F\\) is a \\(\sigma\\)-algebra if and only if :

1.           The empty set is in \\(F\\).
2.           If \\(A \in F\\) then the complement \\(A^c \in F\\).
3.  If \\(A_i \in F\\), \\(i=1,2,...\\), then their union \\(\cup A_i \in F\\)

## Measure (Definition)

Let \\((\Omega, F)\\) be a measurable space. A set function \\(\nu\\) defined on \\(F\\) is called a **measure** if and only if :

1.           \\(0 \le \nu(A) \le \infty\\) for any \\(A\in F\\)
2.           \\(\nu(\emptyset)=0\\)
3.  If \\(A_i \in F\\), \\(i=1,2,...\\), and \\(A_i\\)'s are disjoint for any \\(i \neq j\\), then $$ \nu (\cup\_{i=1}^\infty A_i) = \sum\_{i=1}^{\infty} \nu (A_i)$$

---

## Monotonicity, Subaddivity, Infinite Unions, Infinite Intersections (Proof)

Let \\((\Omega, F, \nu)\\) be a measurable space.

### 1. Monotonicity

If \\(A \subset B\\), then \\(\nu(A)\le \nu(B)\\).

**Proof**

Since,  
$$A \subset B$$
$$B = A \cup (A^c \cap B)$$
$$ A \text{ and } A^c \cap B \text{ are disjoint}$$

By the definition of a **measure (2)**, (since they are disjoint)
$$\nu(B) = \nu(A) + \nu(A^c \cap B)$$

But \\(0\ge\nu(A^c \cap B)\\) from **measure (1)** so then it follows that
$$\nu(A)\le \nu(B)$$
as required.

### 2. Subaddivity

For any sequence \\(A_1, A_2,...\\), $$ \nu(\cup\_{i=1}^\infty A_i) \le \sum\_{i=1}^\infty \nu(A_i)$$

**Proof**
$$ \nu(A\cup B) = \nu(A) + \nu(B) - \nu(A \cap B)$$

By **measure (1)** a measure is nowhere negative, so \\( \nu(A \cap B) \ge 0\\), rearranging above $$ \nu(A\cup B) \le \nu(A) + \nu(B) $$

### 3. Infinite Unions

If \\(A_1 \subset A_2 \subset A_3 \subset ... \\) is an increasing sequence of measurable sets then
$$ \nu(\cup\_{i=1}^\infty A_i) = \text{lim}\_{n\rightarrow\infty}\nu(A_n)$$

**Proof**

Set \\(A_0 := \emptyset \\) and \\(B_i := A_i \backslash A\_{i-1} , i \ge 1\\).

Note that \\(B_i\\) are pairwise disjoint and that $$ A_i = B_1 \cup ... \cup B_i \text{ for all } i \ge 1$$ Consequently, $$ \cup\_{i=1}^\infty A_i = \cup\_{i=1}^\infty B_i $$

Thus  
\\(
\begin{align}
\nu( \cup\_{i=1}^\infty A_i) &= \sum\_{i=1}^\infty \nu(B_i) \\\\
&= \text{lim}\_{i \rightarrow \infty}\sum\_{j=1}^\infty \nu(B_j) \\\\
&= \text{lim}\_{n\rightarrow\infty}\nu(A_n)
\end{align}
\\)

### 4. Infinite Intersections

If \\(A_1 \supset A_2 \supset A_3 \supset ... \\) is an decreasing sequence of measurable sets and if \\( \nu(A_i \lt \infty )\\) then
$$ \nu(\cup\_{i=1}^\infty A_i) = \text{lim}\_{n\rightarrow\infty}\nu(A_n)$$

**Proof**

Set \\(C_i := A_1 \backslash A_i\\).

Note that \\(\nu (A_i) + \nu(C_i)=\nu(A_1) \\) and that \\(\nu(C_i) \le \nu (A_1) \lt \infty \\). So then
$$ \nu (A_i) = \nu(C_i) - \nu(A_1) $$

Since $$ \cup\_{i=1}^\infty C_i = \cup\_{i=1}^\infty (A_1 \backslash A_i) = A_1 \backslash (\cap\_{i=1}^\infty A_i ) $$

and using the fact that \\(\nu(A_1) \lt \infty\\) (from definition above)

$$ \nu(\cap\_{i=1}^\infty A_i) = \nu(A_1) - \nu(\cup\_{i=1}^\infty C_i) $$

Using result from **3. Infinite Unions**

\\(\begin{align}
\nu(\cap\_{i=1}^\infty A_i) &= \nu(A_1) - \nu(\text{lim}\_{n\rightarrow\infty}C_n) \\\\
&= \text{lim}\_{n\rightarrow\infty}\nu(A_1-C_n) \\\\
&= \text{lim}\_{n\rightarrow\infty}\nu(A_n)
\end{align}\\)
