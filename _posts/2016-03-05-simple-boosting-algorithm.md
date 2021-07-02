---
layout: post
category : web micro log
tags :
---

We can think of boosting as some kind of weighted sample. Essentially we build models and have higher weights assigned to the
observations which we score incorrectly. Our simple boosting algorithm is as follows:

1.  Build a model, on a weighted sample of points
2.  See which ones we score incorrectly/correctly and assign a penalty of 110%/90% 
3.  Repeat 1.

Then we can average out the predictions to get the final score. This should improve the model over a simple one.

As a quick example, below is some R code which demonstrates this:

```r

#' simple boosting algorithm
#' also uses caret library

library(mlbench)
library(rpart)
library(dplyr)

#' let's work off pimaindiandiabetes
data(PimaIndiansDiabetes)

simple_model <- rpart(diabetes ~., data=PimaIndiansDiabetes)

#' Boosting is essentially weighted repeated sampling
#' of a model. In this case we will
#' use 10 models with rpart together to create
#' a naive boosted tree...


#' normalises a vector, so that the sum is 1
normalise <- function(vec) {
  vec/sum(vec)
}

#' from stackoverflow
getMode <- function(InVec) {
  names(which.max(table(InVec)))
}

predict.boost <- function(modelList, data) {
  model_pred_all <- as.data.frame(lapply(modelList, function(mod) {
    predict(mod, data, type="class")
  }))
  return(apply(model_pred_all, 1, getMode))
}

n <- 10
index_weight <- normalise(rep(1, nrow(PimaIndiansDiabetes)))
mod_list <- list()

for(i in 1:n) {
  samp_idx <- sample(seq_len(nrow(PimaIndiansDiabetes)), nrow(PimaIndiansDiabetes)*2, prob=index_weight, replace=TRUE)
  new_model <- rpart(diabetes ~., data=PimaIndiansDiabetes[samp_idx,])
  mod_list[[i]] <- new_model
  reweight_idx <- unique(samp_idx)
  new_pred <- predict(new_model, PimaIndiansDiabetes[reweight_idx, ], type="class")

  for(idx in unique(samp_idx)){
    if (predict(new_model, PimaIndiansDiabetes[idx, ], type="class") == PimaIndiansDiabetes[idx, "diabetes"]){
      index_weight[idx] <- index_weight[idx] * 0.9
    } else {
      index_weight[idx] <- index_weight[idx] * 1.1
    }
  }
  index_weight <- normalise(index_weight)
}

boost_mod <- predict.boost(mod_list, PimaIndiansDiabetes)

# compare performance...
caret::confusionMatrix(PimaIndiansDiabetes$diabetes, predict(simple_model, PimaIndiansDiabetes, type="class"))
caret::confusionMatrix(PimaIndiansDiabetes$diabetes, predict.boost(mod_list, PimaIndiansDiabetes))


```

With the output:

```
> caret::confusionMatrix(PimaIndiansDiabetes$diabetes, predict(simple_model, PimaIndiansDiabetes, type="class"))
Confusion Matrix and Statistics

          Reference
Prediction neg pos
       neg 449  51
       pos  72 196

               Accuracy : 0.8398         
                 95% CI : (0.812, 0.8651)
    No Information Rate : 0.6784         
    P-Value [Acc > NIR] : < 2e-16        

                  Kappa : 0.641          
 Mcnemar's Test P-Value : 0.07133        

            Sensitivity : 0.8618         
            Specificity : 0.7935         
         Pos Pred Value : 0.8980         
         Neg Pred Value : 0.7313         
             Prevalence : 0.6784         
         Detection Rate : 0.5846         
   Detection Prevalence : 0.6510         
      Balanced Accuracy : 0.8277         

       'Positive' Class : neg            

> caret::confusionMatrix(PimaIndiansDiabetes$diabetes, predict.boost(mod_list, PimaIndiansDiabetes))
Confusion Matrix and Statistics

          Reference
Prediction neg pos
       neg 473  27
       pos  65 203

               Accuracy : 0.8802          
                 95% CI : (0.8551, 0.9023)
    No Information Rate : 0.7005          
    P-Value [Acc > NIR] : < 2.2e-16       

                  Kappa : 0.7274          
 Mcnemar's Test P-Value : 0.0001145       

            Sensitivity : 0.8792          
            Specificity : 0.8826          
         Pos Pred Value : 0.9460          
         Neg Pred Value : 0.7575          
             Prevalence : 0.7005          
         Detection Rate : 0.6159          
   Detection Prevalence : 0.6510          
      Balanced Accuracy : 0.8809          

       'Positive' Class : neg  

```
