---
layout: post
category: web micro log
tags:
---

I've always tried to regularly string together something interesting that I've picked up
this month however, was a bit of a strange month; the many things that I've picked up
weren't necessarily _difficult_ but in some ways _obvious_ to people who might have been
a bit more observant (or should have been obvious to me earlier!)

## Neural Networks

This month two insights which I had already knew, but never managed to piece together finally clicked:

- Neural networks can be framed in terms of a composition of functions; firstly a linear component and then a non-linear component
- Neural networks have the ability to approximate any function.

### On the requirement of non-linear portion of a neural network

Last year when I was working around neural networks I was thinking around what are the consequences if we remove the condition of a neural network requiring the non-linear component. Couldn't a neural network simply be a linear component followed by a linear component?

To understand what in the world I'm talking about, it is realising that instead of framing neural networks using the idea of neurons and the brain you can instead frame it as a linear and non-linear component.

For each layer in a neural network it has:

1.  The weights which we estimate. This component is considered to be linear as it is computed via dot product.
2.  Non-linear activation function, which is used to transform the output of the weights as inputs to the next layer.

Without the non-linear component, a neural network is only able to estimate linear models, as it would simply be multiple dot products of weight matrices.

## On Feature Learning for Natural Language Processing

Perhaps this was a stupid thought which I didn't think through clearly, but one question which I had in my mind was simply, is there "one NLP algorithm to rule them all". The answer unsurprisingly "no".

As a simple example is if we create binary classifier based on the presence on a bunch of words; then anything besides a binary representation would have an inferior representation. This demonstrates that in text mining there is "no free lunch" (which can be seen of course in the no free lunch theorem).

The consequence of this is to consider encodings which take into account different representation of words. For example a "feature rich encoding" can be constructed through:

- TFIDF
- POS tagging with TFIDF
- NER tagging with TFIDF
- word2vec

It may not out perform a particular representation, but its average performance is likely to be "good enough".

The consequence of all this is that we shouldn't strive to chase the "best" representation of our text data, but rather we should always use multiple representation, and combine them in an ensemble base method.
