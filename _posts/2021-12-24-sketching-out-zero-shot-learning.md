---
layout: post
category : 
tags : 
tagline: 
---

How can we architect zero shot learning?

Perhaps the easiest way is to leverage existing models with ANN. Let me explain:

1.  Find an existing model and extract the embedding
2.  Use the embedding combined with kNN to learn new concepts quickly

Hopefully this approach can quickly scale out datasets to different scenarios with little effort beyond engineering. 

One of the key issues is the lack of "easy" libraries/applications which can do this successfully. 

