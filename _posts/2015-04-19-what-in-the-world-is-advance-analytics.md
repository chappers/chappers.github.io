---
layout: post
category : web micro log
tags : 
---

Advance analytics is a term that suddenly popped up in my world in the last few months. Since this is my area of expertise I thought I would investigate what exactly advance analytics is.

# What is analytics?

It is clear to me that the notion of analytics consists of two parties; those who make use of tools and programming languages, and those who insist on making use of Excel, and powerpoint slides as the basis of their analysis. 

These could be extended to ideas such as financial statement analysis, and consideration of accounting ratios. These generally consider the ideals of heuristics and experience. A common place where this is still highly important is financial crime, where it is difficult to determine based on internally held data, whether a transaction will be fraudulent. This is because generally data may need to be sourced through interviews or phone calls. 

On the other hand there are analytics which are data driven. This branch is what is commonly referred to as "advanced analytics". 

# What's in a name?

My first impression of advance analytics was of course the term advance. What exactly constitute as "advance" analytics? The more I talked to my colleagues, it became increasingly clear that advance analytics did not really exceed anything beyond statistics 101; understanding what a t-test did and being able to intepret the results well, probably constitutes to advanced analytics. Building strong linear models and again more importantly being able to explain and understand the implications of the model is advance analytics. 

Intrinsically there is nothing wrong with this type of thinking. But for me it raises another question.

# Education and the way forward

It is clear that many of the techniques discussed are neither cutting edge or even special. At the very least, tools to build hypothesis testing, such as the binomial test should indeed be on the radar for anyone performing any kind of analytics. If we want to think about the change in our accounting ratios how do we determine if the change is indeed significant? Sure we can say that our profits increased, but how does it benchmark with respect to the industry? 

These are all questions which should be asked, since it is clear that analysis requires context. My small fear is that the advance tag offers people perhaps an excuse; a reason to avoid these simple concepts and ideas and only seeks to bring a larger divide in the community. This has an effect of reducing analysts work to merely be black boxes, making it difficult to promote more useful toolsets and models to become more accessible to the average person. 

# What advance analytics is not

One qualm that I find difficult to digest, is the inappropriate application of models. It is extremely easy to fit multiple difficult models. For example in R to fit a random forest, decision tree, neural net, gradient boosted models, can all be accomplished with code that looks like this:


```r
caret::train(predictors, response, method="rf")
caret::train(predictors, response, method="rpart")
caret::train(predictors, response, method="nnet")
caret::train(predictors, response, method="gbm")
```

Which will automatically perform cross validation, and find the optimal hyperparameters. Alot of the magic is already done. However what separates analysts is the ability to discern the differences in each of these models and approaches; the advantages and disadvantages, under what circumstances they excel, and don't make sense.

# Drop the advance tag

Perhaps a bit harsh and biased. But I think the advance tag shouldn't exist. You either perform analysis or you don't, there isn't really anything inbetween. By adding such a tag, you only seek to discriminate those. Instead we should be promoting what all analytics should possess. Perhaps the only time we should use the words analytics or analysis is when certain amount of rigour has been applied. 









