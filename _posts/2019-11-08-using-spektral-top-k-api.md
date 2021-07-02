---
layout: post
category : 
tags : 
tagline: 
---

Theres a lot of somewhat undocumented APIs in the [Spektral APIs](https://github.com/danielegrattarola/spektral). 

As I'm interested in some of the Graph coarsening - I thought we'll look at how some of the graph networks are implemented and how they are used. 

Firstly for a node wise graph - it can be implemented where the inputs are 2 items; the node-feature input, and adjacency matrix. 

```py
nodes = 5
feats = 4

X = np.random.uniform(size=(nodes, feats)).astype(np.float32)
adj = np.random.uniform(size=(nodes, nodes)).astype(np.float32)

input_x = Input(shape=(feats))
input_adj = Input(shape=(None,))

topk = TopKPool(0.5, feats)([input_x, input_adj])
model_out = Model(inputs=[input_x, input_adj], outputs=topk)
[K.eval(x) for x in model_out([X, adj])]
```

At the same time we can use a graph-wise implementations, in which case we add an additional tensor as input which describes which segments are related to each other. For example, `[0, 0, 0, 1, 1]` would suggest that the index 0-2, are one graph with three nodes and index 3-4 are another graph with two nodes. 

If we extend this, we can have as input a graph with varying dimensions into `TopKPool`. An example of this is:

```py
nodes = [5, 4, 3]
feats = 4

input_x = Input(shape=(feats))
input_adj = Input(shape=(None,))
input_in = Input(shape=(None,), dtype="int32")
topk = TopKPool(0.5, feats)([input_x, input_adj, input_in])
model_out = Model(inputs=[input_x, input_adj, input_in], outputs=topk)

# try with multiple graphs...
X = np.random.uniform(size=(np.sum(nodes), feats)).astype(np.float32)
adj = np.random.uniform(size=(np.sum(nodes), np.sum(nodes))).astype(np.float32)
segment_i = []
for idx, nd in zip(range(len(nodes)), nodes):
    for _ in range(nd):
        segment_i.append([idx])
segment_i = np.array(segment_i).astype(np.int32)
[K.eval(x) for x in model_out([X, adj, segment_i])]
```

This demonstrates the usage, and a cool way to use additional (untrainable) input as part of the Keras API. 