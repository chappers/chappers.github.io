---
layout: post
category : proofs
tags : [proofs]
tagline: 
---

If \\(X~LN(\mu,\sigma^2)\\) then \\(ln(X)=Y\\) is \\(Y\\) is distributed \\(N(\mu,\sigma^2)\\).  

**Derivation of log-normal distribution**

\\(\begin{align}   
Pr(X < k) &= Pr(e^{Y} < k) \\\\
&= Pr(Y < ln(k)) \\\\
&= \int_{\infty}^{ln(k)} \frac{1}{\sqrt{2\pi \sigma}} e^{- \frac{(Y-\mu)^2}{2\sigma^2}} dy \\\\
&= \int_{\infty}^{ln(k)} \frac{1}{\sqrt{2\pi \sigma}} e^{- \frac{(ln(x)-\mu)^2}{2\sigma^2}} \frac{1}{x} \frac{dx}{dy}dy  \\\\
&= \int_{\infty}^{ln(k)} \frac{1}{x\sqrt{2\pi \sigma}} e^{- \frac{(ln(x)-\mu)^2}{2\sigma^2}} dx
\end{align}\\)