---
layout: post
category : 
tags : 
tagline: 
---

# Dirac Delta Function

If we wanted to have a function that was differentiable, and behaved like a discrete function how would we do it?

As a starting point we could use the [Dirac Delta Function](https://en.wikipedia.org/wiki/Dirac_delta_function) for a point estimation.

```r
library(ggplot2)

a = 0.1
dirac_delta <- function(x, a, loc=0){
  (1/(abs(a)*(pi^(0.5))))* exp(-((x-loc)/a)^2)
}

xrange = (-100:100)/100
y = dirac_delta(xrange, a=a)

ggplot() + 
  aes(x=xrange) + 
  aes(y=y)+
  geom_line()
```

But perhaps we want to have a function where it is defined to be 1 within a range, and 0 elsewhere. This might be able to be done through abuse of gumbel-softmax function

```r
library(ggplot2)

gumbel_softmax <- function(x, tau=0.01, loc=0) {
  1/(1+exp(-(x-loc)/tau))
}

xrange = (-100:100)/100
y = gumbel_softmax(xrange, loc=-0.5) - gumbel_softmax(xrange, loc=0.5)

ggplot() + 
  aes(x=xrange) + 
  aes(y=y)+
  geom_line()
```

You could also approximate the dirac delta function in this way as well:

```r
library(ggplot2)

gumbel_softmax <- function(x, tau=0.01, loc=0) {
  1/(1+exp(-(x-loc)/tau))
}

eps <- 1e-07
xrange = (-100:100)/100
y =  gumbel_softmax(xrange, loc=0-eps) - gumbel_softmax(xrange, loc=0+eps)

ggplot() + 
  aes(x=xrange) + 
  aes(y=y)+
  geom_line()
```

# Fitness Functions

What if you don't want a flat area - but instead minimum point of the range you're interested in, and an increasing penalty otherwise if it is some kind of constraint or fitness function.

Perhaps if it is a constraint function you want it to increase over the point or area estimation.

We can use the [softplus variant](https://en.wikipedia.org/wiki/Rectifier_(neural_networks)) to get this working. 

```r
library(ggplot2)

gumbel_softplus <- function(x, tau=0.5, loc=0) {
  log(1+exp((x-loc)/tau))
}

gumbel_softplus_neg <- function(x, tau=0.5, loc=0) {
  log(1+exp(-(x-loc)/tau))
}

eps <- 1e-07
xrange = (-100:100)/100
y =  gumbel_softplus(xrange, loc=0.5+eps)+gumbel_softplus_neg(xrange, loc=-0.5-eps)

ggplot() + 
  aes(x=xrange) + 
  aes(y=y)+
  geom_line()
```

We can combine the two as well

```r
library(ggplot2)

eps = 1e-7
xrange = (-100:100)/10

softmax_df <- data.frame(
  xrange = xrange,
  y = gumbel_softmax(xrange, loc=0.5) - gumbel_softmax(xrange, loc=-0.5) + 1,
  type = 'softmax'
)

softplus_df <- data.frame(
  xrange = xrange,
  y = (gumbel_softplus(xrange, tau=1, loc=0.5+eps) + gumbel_softplus_neg(xrange, tau=1, loc=-0.5-eps)),
  type = 'softplus'
)

soft_maxplus_df <- data.frame(
  xrange = xrange,
  y = (gumbel_softmax(xrange, loc=0.5) - gumbel_softmax(xrange, loc=-0.5) + 1)*(gumbel_softplus(xrange, tau=1, loc=0.5+eps) + gumbel_softplus_neg(xrange, tau=1, loc=-0.5-eps)),
  y_other = (gumbel_softmax(xrange, loc=0.5) - gumbel_softmax(xrange, loc=-0.5) + 1) * (abs(xrange-0)),
  type = 'softmax*softplus'
)


compare_df = rbind(softmax_df, softplus_df, soft_maxplus_df)

ggplot(data=compare_df) +
  aes(x=xrange) +
  aes(y=y) +
  geom_line(alpha=0.25) +
  aes(group=type,
      color=type,
      ) +
  scale_y_sqrt()


ggplot(data=soft_maxplus_df) +
  aes(x=xrange) +
  aes(y=y_other) + # or y=y
  geom_line() +
  aes(group=type,
      color=type,
  ) +
  scale_y_sqrt()
```



