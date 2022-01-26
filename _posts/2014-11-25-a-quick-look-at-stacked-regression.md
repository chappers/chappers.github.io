---
layout: post
category: web micro log
tags:
---

The [Netflix Prize competition](http://www.the-ensemble.com/) has always been interesting to me, however I had never really taken the time to really think about the implications of blending models together. In this post I will take a look at the easiest case of blending models; looking at a linear model example.

I will preface this by saying that my R code is not the most perfect, but I hope easy to follow. I have not found any blog posts looking at blending/ensemble modelling in the context of the linear models. There are plenty on CART and how they could be built, but not too many on the humble linear regression.

There are other methods for combining models such as bagging and boosting, but they will not be discussed here.

# Model Combination

There are various ways to combine models, which _can_ improve the performance compared with having only one model. Typically we would use MSE to determine whether this model is an improvement or not (remember we generally prefer parsimonious models).

## Combining _m_ estimators

Lets begin by creating a general formula for a linear combination of \\(m\\) estimators. Lets assume we have
\\(\{\hat{\theta}\_j\}, j = 1, ... , m\\) and

$$ E(\hat{\theta}\_j) = \mu_j, Var(\hat{\theta}\_j) = v_j, Bias(\hat{\theta}\_j) = b_j $$

This means we are interested in creating our combined estimator

$$ \hat{\theta}\_{combined} = \sum\_{j=1}^m w_j \hat{\theta}\_j $$

where \\(w\\) is the weights.

We can then determine MSE

$$ MSE = E((\hat{\theta}\_{combined}-\theta)^2) = w^T \Sigma w + (w^T \mu - \theta)^2 $$

We can then of course apply various constraints like the sum of weights being equal to 1, with all the weights above 0. This model of blending and its properties are well documented.

However, what happens if we know nothing about the correlation? What can we do then?

## Stacked Regression

Lets suppose we have \\(N\\) models (lets call each model \\(h_i(x)\text{, for } i = 1, ..., N \\). Perhaps these are models which use different methods, or use different training data. How can we go ahead and combine these models?

Again with a weight of \\(\beta_i\text{, for } i = 1, ..., N\\) for each model \\(h_i\\), we can create an estimate, To create the "best" weight, we will then go and solve the following equation

$$ \hat{\beta} = arg min*\beta \sum*{i=1}^N (y*i - \sum*{j=1}^m \beta_j h_j (x_i))^2 $$

Which when we really think about it, is just fitting a linear regression using the model as a variable. We can use crossvalidation techniques to avoid giving unfairly high weight to models with higher complexity (overfitting). Furthermore it was shown by Breiman, that the performance of the stacked regression improves if we add the constraint that \\(\hat{\beta}\\) is non-negative.

# R example

_The code below is a simple example. It is not a representation of how stacked regression should be done._

Below is an R example to motivate how stacked regression can be used.

```r
library(car) # we will use the data set "Prestige" which is in this library as an example
library(quadprog) # for creating constrained least squares
library(corpcor) # to make a positive definite matrix when eigenvalues are very close to 0

#' fit the model, this may/may not be the best model for this data, but mere
#' just an example to motivate this method
reg <- lm(prestige ~ education + log(income), data=Prestige)
```

Now to create our "models" I have used a loop to create a linear model based on a random subset of this data. This model is then used to predict "prestige".

```r
reg_results <- c()
for(i in 1:3) { # we'll just create 3 new "models"
  Prestige.sample <- Prestige[sample(1:nrow(Prestige), sample(35:65, 1)),]
  reg1 <- lm(prestige ~ education + log(income) + women, data=Prestige.sample)
  reg_results <- cbind(reg_results, as.vector(predict(reg1, Prestige)))
}
```

Observe that if we want to solve a constrained linear regression, this is a quadratic programming problem. Hence this problem can be solved using `solve.QP` function in `quadprog`.

```r
sol <- solve.QP(Dmat = make.positive.definite(t(reg_results) %*% reg_results),
                dvec = t(as.vector(Prestige$prestige)) %*% reg_results,
                Amat = diag(ncol(reg_results)),
                bvec = rep(0, ncol(reg_results))
)
```

The weights would be in `sol$solution`, and we can create our new prediction based on blending these three models together

```r
predictions <- as.vector(t(as.matrix(sol$solution)) %*% t(as.matrix(reg_results)))
```

Now using a quadratic loss function, \\(L(\theta, \hat{\theta}) = (\theta - \hat{\theta})^2\\), we can (hopefully) see the improvement in performance (note the R code shown above uses `sample`, your results may vary due randomness)

```r
> sum(unlist(Map(function(x) x*x, (Prestige$prestige - predictions))))
[1] 4986.051
> sum(unlist(Map(function(x) x*x, (Prestige$prestige - predict(reg, Prestige)))))
[1] 5053.639
```

### Code

```r
library(car) # we will use the data set "Prestige" which is in this library as an example
library(quadprog) # for creating constrained least squares
library(corpcor) # to make a positive definite matrix when eigenvalues are very close to 0

#' fit the model, this may/may not be the best model for this data, but mere
#' just an example to motivate this method
reg <- lm(prestige ~ education + log(income), data=Prestige)

reg_results <- c()
for(i in 1:3) { # we'll just create 3 new "models"
  Prestige.sample <- Prestige[sample(1:nrow(Prestige), sample(35:65, 1)),]
  reg1 <- lm(prestige ~ education + log(income) + women, data=Prestige.sample)
  reg_results <- cbind(reg_results, as.vector(predict(reg1, Prestige)))
}

sol <- solve.QP(Dmat = make.positive.definite(t(reg_results) %*% reg_results),
                dvec = t(as.vector(Prestige$prestige)) %*% reg_results,
                Amat = diag(ncol(reg_results)),
                bvec = rep(0, ncol(reg_results))
)

predictions <- as.vector(t(as.matrix(sol$solution)) %*% t(as.matrix(reg_results)))

sum(unlist(Map(function(x) x*x, (Prestige$prestige - predictions))))
sum(unlist(Map(function(x) x*x, (Prestige$prestige - predict(reg, Prestige)))))

```
