---
layout: post
category : web micro log
tags : 
---

One way which you can use randomized optimization is to determine the best subset of features for your particular model. The idea is that we want to find the best vector of 0's and 1's which represent the selection of features. 

The setup of the problem is quite easy. Simply define the function which you want to maximize and wrap it around the optimization problem.

For example:

```r
fitFun <- function(ind, x, y) {
  ind <- which(ind == 1)
  if (length(ind) < 2) return(0)
  out <- caret::train(x[, ind], 
               y,
               method="rpart")
  caret:::getTrainPerf(out)[, "TrainAccuracy"]
}
```

In the function above, we have used `caret::train` to model the data of interest. We the parameter `ind` which is the vector of 0's and 1's as stated above. We finally output "TrainAccuracy" which is the value which we wish to maximise.

Then all that remains is that we wrap it within genetic algorithm optimization function as follows:

```r
ga_resampled <- ga(type = "binary", 
                   fitness = fitFun, 
                   min = 0, 
                   max = 1, 
                   maxiter = 2, 
                   nBits = ncol(PimaIndiansDiabetes) - 1, 
                   names = names(PimaIndiansDiabetes)[-ncol(PimaIndiansDiabetes)], 
                   x = PimaIndiansDiabetes[, -ncol(PimaIndiansDiabetes)], 
                   y = PimaIndiansDiabetes$diabetes, 
                   keepBest=TRUE)
```

With the full code being this:

```r
library(caret)
library(mlbench)
library(GA)
data(PimaIndiansDiabetes)

fitFun <- function(ind, x, y) {
  ind <- which(ind == 1)
  if (length(ind) < 2) return(0)
  out <- caret::train(x[, ind], 
               y,
               method="rpart")
  caret:::getTrainPerf(out)[, "TrainAccuracy"]
}

ga_resampled <- ga(type = "binary", 
                   fitness = fitFun, 
                   min = 0, 
                   max = 1, 
                   maxiter = 2, 
                   nBits = ncol(PimaIndiansDiabetes) - 1, 
                   names = names(PimaIndiansDiabetes)[-ncol(PimaIndiansDiabetes)], 
                   x = PimaIndiansDiabetes[, -ncol(PimaIndiansDiabetes)], 
                   y = PimaIndiansDiabetes$diabetes, 
                   keepBest=TRUE)
```

Of course in the normal sense you probably should seperate your data into training and validation data, and also incorporate cross validation and other techniques to ensure that you don't overfit.

