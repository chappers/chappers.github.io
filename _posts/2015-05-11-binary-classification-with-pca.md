---
layout: post
category: web micro log
tags:
---

One question which puzzled me was how can PCA be used in a classification sense? Well here is an approach which is used in unsupervised setting based on my reading on PRIDIT modelling.

Basically you approach PCA from a factor analysis perspective, providing ranks on your variables. Then you can segment your scores in the normal way and group them as your classification. In general it has been found that this approach has worse accuracy than other approaches (unsurprising since this is an unsupervised technique), however it performs better in the situation where you have unbalanced data **and** the statistical power is highly important.

To understand how this approach works, we simply calculate the principal components, and use the loadings to calculate the scores of each observations. These scores are then weighted by the variance contribution as determined by the principal components, and then can be used in classification. Below is the code on how this would be completed in R (requires `dplyr`, `caret`, `ggplot2` to be installed. )

```r
# just take two classes, so we can have a binary classification
# we can probably do something fancy to handle more than two classes.
iris.sub <- dplyr::select(iris, -Species)[iris$Species != "setosa",]
prin.iris <- princomp(iris.sub)
iris.weights <- as.data.frame(as.matrix(iris.sub) %*% prin.iris$loadings[,1:2])
iris.scores <- cbind(iris.weights, Species=iris[iris$Species != "setosa",c("Species")])
iris.scores$Species <- as.factor(as.character(iris.scores$Species))

iris.scores$Predict <- sapply(iris.scores$Comp.1, function(x) {
  if (x < quantile(iris.scores$Comp.1, 0.5)) {
    return("virginica")
  } else {
    return("versicolor")
  }
})

iris.scores$Predict <- as.factor(iris.scores$Predict)
caret::confusionMatrix(iris.scores$Predict, iris.scores$Species)
```

The code isn't too difficult, and performs relatively well with the confusion matrix as below:

```r
            Reference
Prediction   versicolor virginica
  versicolor         44         6
  virginica           6        44

               Accuracy : 0.88
                 95% CI : (0.7998, 0.9364)
    No Information Rate : 0.5
    P-Value [Acc > NIR] : 9.557e-16

                  Kappa : 0.76
 Mcnemar's Test P-Value : 1

            Sensitivity : 0.88
            Specificity : 0.88
         Pos Pred Value : 0.88
         Neg Pred Value : 0.88
             Prevalence : 0.50
         Detection Rate : 0.44
   Detection Prevalence : 0.50
      Balanced Accuracy : 0.88

       'Positive' Class : versicolor
```

If we look at some plots between first and second components and the box plots of the scores, we can start to appreciate how well this works even without the class labels in the training stage.

![scatter](/img/pca-classifier/scatter.png)

![boxplot](/img/pca-classifier/boxplot.png)
