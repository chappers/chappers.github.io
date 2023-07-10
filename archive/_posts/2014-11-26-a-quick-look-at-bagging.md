---
layout: post
category: web micro log
tags:
---

Continuing on the previous post, I thought we'll have a quick look at bagging (bootstrap aggregating). We will have a look at using bagging for regression and classification.

Firstly what is bootstrapping?

# Bootstrap

Bootstrap is where we try to estimate something based on a sample. This is important when we don't know the distribution of the variable, and we want to get a feel for the error which we may have for our estimate.

### Methodology

The goal of bootstrap is to find the theoretical distribution of a particular statistic. The steps to achieve this is as follows:

1.  Resample our observations with replacement
2.  Apply the statistic in question
3.  Repeat this process as desired

This assumes that by resampling our observations we will be able to infer the distribution of our statistic.

### Example

_The code below is a simple example. It is not a representation of how bagging should be done._

Lets compute the example on Wikipedia in R.

> Consider a coin-flipping experiment. We flip the coin and record whether it lands heads or tails. (Assume for simplicity that there are only two outcomes) Let \\(X = x_1, x_2, ..., x_10\\) be 10 observations from the experiment. \\(x_i = 1\\) if the \\(i^{th}\\) flip lands heads, and \\(0\\) otherwise.

Firstly we can quickly get 10 flips of the coin using the `sample` function:

```r
flips <- sample(c(0,1), 10, replace=TRUE)
```

To bootstrap the mean for this, we would simple resample the vector `flips` many times, computing the mean each time.

```r
resample <- lapply(1:100, function(x) {sample(flips, replace=T)})
bootstrap.mean <- sapply(resample, mean)
```

Now the vector `bootstrap.mean` will be the empirical bootstrap distribution of mean.

# Bagging

How does this idea extend to combining models? Quite simply, the example in my previous post was basically a bagging example; the idea where we resample and reconstruct the model, with the final model simply being the average of all these models. In the classification scenario it is determined by majority vote.

So the steps are as follows:

1.  Resample our data with replacement
2.  Fit the model
3.  Repeat this process as desired
4.  Average all the results

### Regression Example

Similar to my previous post we will use the `Prestige` dataset.

```r
#' bagging example using Prestige data
library(car) # for Prestige dataset

Prestige.train <- Prestige[sample(nrow(Prestige), nrow(Prestige)/3),]
reg <- lm(prestige ~ education + log(income), data=Prestige.train)
```

Even creating the models is exactly the same because I used resampling, (except this time I will have `replace = True`)

```r
#' this portion of code is still the same as the stacked regression example.
reg_results <- c()
for(i in 1:1000) {
  Prestige.sample <- Prestige.train[sample(1:nrow(Prestige.train), sample(10:30, 1), replace=TRUE),]
  reg1 <- lm(prestige ~ education + log(income) + women, data=Prestige.sample)
  reg_results <- cbind(reg_results, as.vector(predict(reg1, Prestige)))
}
```

Now we can simply take the average of our predictions to get our final predictions.

```
#' since this is regression, we will take the "average" of the three models
#' otherwise it would be majority vote in the classification scenario
predictions <- apply(reg_results, 1, function(x) mean(x))
```

The Loss function reveals (randomly) that it is an improvement (or not)!

```r
> sum(unlist(Map(function(x) x*x, (Prestige$prestige - predictions))))
[1] 4943.626
> sum(unlist(Map(function(x) x*x, (Prestige$prestige - predict(reg, Prestige)))))
[1] 5092.777
```

### Classification Example

Bagging can be further expanded to classification problems. Here we will use the `iris` dataset using linear discriminant analysis as our method of classification. This model isn't the "best" model, but merely to demonstrate how bagging can be better than non-bagged example.

```r
library(MASS) # for lda

iris.train <- iris[c(20:40, 70:90, 115:135),]
fit <- lda(Species ~ Sepal.Length + Sepal.Width, data=iris.train)
```

We can then go ahead and applying resampling.

```r
class_results <- c()
for(i in 1:1000) {
  iris.sample <- iris.train[sample(1:nrow(iris.train), sample(35:55, 1), replace=TRUE),]
  fit1 <- lda(Species ~ Sepal.Length + Sepal.Width, data=iris.sample)
  class_results <- cbind(class_results, as.vector(predict(fit1, iris)$class))
}
```

To calculate the most popular classification, we are simply computing the mode. The mode function can be defined and applied as follows:

```r
Mode <- function(x) {
  ux <- unique(x)
  ux[which.max(tabulate(match(x, ux)))]
}

predictions <- apply(class_results, 1, function(x) Mode(x))
```

Now lets compare the performance between these two methods:

```r
> sum(iris$Species == predict(fit, iris)$class)/length(iris$Species)
[1] 0.7866667
> sum(iris$Species == predictions)/length(iris$Species)
[1] 0.7933333
```

#### Code

```r
#' bagging example using Prestige data
library(car) # for Prestige dataset

reg <- lm(prestige ~ education + log(income), data=Prestige)

#' this portion of code is still the same as the stacked regression example.
reg_results <- c()
for(i in 1:1000) {
  Prestige.sample <- Prestige[sample(1:nrow(Prestige), sample(35:65, 1), replace=TRUE),]
  reg1 <- lm(prestige ~ education + log(income) + women, data=Prestige.sample)
  reg_results <- cbind(reg_results, as.vector(predict(reg1, Prestige)))
}

#' since this is regression, we will take the "average" of the three models
#' otherwise it would be majority vote in the classification scenario
predictions <- apply(reg_results, 1, function(x) mean(x))

sum(unlist(Map(function(x) x*x, (Prestige$prestige - predictions))))
sum(unlist(Map(function(x) x*x, (Prestige$prestige - predict(reg, Prestige)))))
```

```r
library(MASS) # for lda

iris.train <- iris[c(20:40, 70:90, 115:135),]
fit <- lda(Species ~ Sepal.Length + Sepal.Width, data=iris.train)


class_results <- c()
for(i in 1:1000) {
  iris.sample <- iris.train[sample(1:nrow(iris.train), sample(35:55, 1), replace=TRUE),]
  fit1 <- lda(Species ~ Sepal.Length + Sepal.Width, data=iris.sample)
  class_results <- cbind(class_results, as.vector(predict(fit1, iris)$class))
}

Mode <- function(x) {
  ux <- unique(x)
  ux[which.max(tabulate(match(x, ux)))]
}

predictions <- apply(class_results, 1, function(x) Mode(x))

sum(iris$Species == predict(fit, iris)$class)/length(iris$Species)
sum(iris$Species == predictions)/length(iris$Species)
```
