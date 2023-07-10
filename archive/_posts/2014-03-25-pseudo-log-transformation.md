---
layout: post
category: web micro log
---

\\(log\\) transforms are commonly used for data visualisation. But there is a downside. The domain
of \\(log(x)\\) is \\(x > 0 \\). Which then leads to the next question. What if my data can extend
across the whole real line? For example, profit and loss numbers. How could you transform it?

Maybe the immediate thought is to do

$$sign(x) log(|x|+1)$$

Which is an acceptable response answer. Another possible solution is using \\(sinh(x)\\).
Using the fact that

$$ lim\_{x \to +\infty} sinh(x) = e^x $$

Rearranging we can easily see that a suitable alternative is

$$ arcsinh(x/2) $$

which is strangely surprising. The graphs are actually quite similar as well:

![pseudo-log](/img/pseudo-log/pseudo-log.png)

```r
library(ggplot2)
library(reshape2)

x  <- seq(-10, 10, 0.05)
arcsinh <- asinh(x/2)
sign.log <- sign(x)*log(abs(x) + 1)
df <- data.frame(x, arcsinh, sign.log)
df2 <- melt(data = df, id.vars = "x")
ggplot(data = df2, aes(x = x, y = value, colour = variable)) + geom_line()
```

This was an interesting exercise into a possible application of the hyperbolic functions in statistics; something
that I would have easily ignored and passed over.
