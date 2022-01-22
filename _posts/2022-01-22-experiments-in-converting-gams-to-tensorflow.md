---
layout: post
category : 
tags : 
tagline: 
---

I've been working on extension on some of my publications, namely in the gams space, where I've put the code here: https://github.com/charliec443/TreeGrad2

The general idea is we can learn a GAMs construct (via EBM) and convert it to a tensorflow model for stateful training, which allows for online update approach to the model. Through this general framework we separate our model learning process into two phases:

1.  learning the architecture via offline approach
2.  updating the weights in mini-batch setting

This enables very cheap, low-cost approach to get effective model (via EBM) whilst still retaining the benefits of an online updating procedure, with the side-benefit of output embeddings!