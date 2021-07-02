---
layout: post
category : web micro log
tags : 
---

The easiest way to get some performance gain when using genetic algorithms in R (using the library GA and on Windows) is to set parallel to `snow`. This is a documented method, but not exactly clear. 

```r
library(GA)

Rastrigin <- function(x1, x2)
{
  Sys.sleep(10)
  20 + x1^2 + x2^2 - 10*(cos(2*pi*x1) + cos(2*pi*x2))
}

system.time(GA4 <- ga(type = "real-valued",
                      fitness = function(x) -Rastrigin(x[1], x[2]),
                      min = c(-5.12, -5.12), max = c(5.12, 5.12),
                      popSize = 10, maxiter = 5, monitor = TRUE,
                      seed = 12345, parallel = "snow"))
system.time(GA4 <- ga(type = "real-valued",
                      fitness = function(x) -Rastrigin(x[1], x[2]),
                      min = c(-5.12, -5.12), max = c(5.12, 5.12),
                      popSize = 10, maxiter = 5, monitor = TRUE,
                      seed = 12345, parallel = FALSE))

```
