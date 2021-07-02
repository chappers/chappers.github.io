---
layout: post
category : 
tags : 
tagline: 
---


Typically when we think of transfer learning, we naturally think of Deep Learning algorithms. Afterall it makes it easy to "transfer" learnings from a _similar_ problem to a the current domain. For example, we can generate image features from one domain and use it to "kick-start" another problem. Can we do the same thing for other machine learning problems?

**What is Transfer Learning**

I like to think of transfer learning as an extension of online learning, in the sense that we can learn off another problem, and without retaining any knowledge of the underlying dataset, used that information to inform us better on how we should go ahead and tackle new information for our machine learning problem.

In this sense there are a few obvious ways how we can encode this knowledge. 

1.  We can use an algorithm to come up with a compressed representation of historical data!

One way of doing this is if we have an algorithm that can come up with a compressed representation which can be learnt in an online fashion - we can then use this to retain knowledge to then transfer them to a batch model. Based on this we will come up with a naive approach to "transfer" some learnings of historical datasets which we can no-longer see. 

## Naive Transfer Learning

On easy way to approach transfer learning is to make use of surrogate modelling. In pseudo-code it would look something like this:

To firstly prepare some information to "transfer" learnings to:

```
1.  Fit a model to the original data 
2.  Construct a supervised kernel based on supervised learning algorithm of choice
```

The kernel will allow you to transform your data into a representation which assists in how the original machine learning model understood the data in terms of what is predictive and is not. 

For new data where we wish to use this information:

```
1.  Apply the kernel transform to the new data, and append to the modelling dataset
2.  Fit the model as normal.
```

From here we can use an iterative approach to continue to transfer our learnings which would lead it to a mini-batch like updating. In this fashion, any machine learning model which can be represented as a supervised kernel can then naturally be updated in a pseudo-minibatch state just like neural networks!

The key question to this approach though is:

1.  Is it as efficient as fitting a model to all the data rather than in batches? In terms of speed and accuracy?
2.  Can this approach be used to inform/speed up models from similar domains which ultimately are modelling different data?

This is a research question which I should try to answer!
