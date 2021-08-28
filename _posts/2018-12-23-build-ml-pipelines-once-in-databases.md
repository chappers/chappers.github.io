---
layout: post
category : 
tags : 
tagline: 
---

In a previous post we thought through how we can build a Python pipeline and deploy to Javascript. This indeed does allow us to deploy practically on anything with a computer, but let's take a different spin on it - can we deploy onto anything with a database?

There are many reasons why we want to do this. On an enterprise level, there would be a preference on deploying onto the same shared intermediary architecture. As an example, you probably do not want to do point to point integration with existing assets or systems, but rather share transformations, features and models. 

Broad approach would be:

*  Write some code in Python
*  Transcompile to SQL
*  Run models in SQL

This repository: [https://github.com/chappers/sklearn-predict](https://github.com/chappers/sklearn-predict) shows broadly how this approach may work on sqlite databases. Some of the obvious challenges are:

*  Supporting mutiple databases (sqlalchemy?)
*  Supporting databases with limited mathematical transformations

The solution then may be to only support a small subset of feature engineering and machine learning models. As databases can perform rudimentary transformations related to group by aggregations or joins, what remains are the preprocessing which are:

*  One hot encoding (for categorical data)
*  Count vectorizer (for text data)
*  Linear SVM (for a simple prediction)

These were all implemented in several ways. 

**One hot encoding**

Implementation of OHE requires generating `"CASE WHEN ... THEN ..."` statements, this is relatively straight forward and can be easily gleaned from the attributes as part of OHE in scikit learn.

**Count Vectorizer**

This implementation is a bit trickier and requires some care. In our approach, we've aimed to keep the tokenizer the same, by standardising the transforms first, (using `'REPLACE(text, '<my punc>', '.'`)), and then followed with `"CASE WHEN text like ... then ..."` to generate a binarised version of count vectorise. Another approach would be simply to use in-built functions in a database such as `INST` or even `REGEXP_COUNT` if they exist. In the scenario with `INST`, it will still be a binarised count vectorised, but `REGEXP_COUNT` may resolve that issue. 

TFIDF was not considered in this simple pipeline, on the basis that many of the databses do not support "advance" mathematical operations out of the box (including sqlite). This make things more challenging down the road, as TFIDF is commonly used as a measure of entropy of the words. 

**Linear SVM**

As linear svm simply uses coefficients and a model to yield classifications, this is an easy approach to determine what prediction it lies in. This model could be extended to logistc regression if the logarithm function exists within the database. 

## Future Work

From my perspective, this is a starting point and largely "done". It supports out of the box simple machine learning engineering tasks and building rudimentary machine learning algorithms. There are probably vastly better improvements that can be made depending on what databases you may use - but then it wouldn't really be portable!







