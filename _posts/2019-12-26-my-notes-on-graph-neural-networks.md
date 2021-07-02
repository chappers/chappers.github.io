---
layout: post
category : 
tags : 
tagline: 
---

As the year draws to an end I thought its time to reflect on what I've been reading and researching. One topic which I've realised I've been very "light" with respect to notes is Graph Neural Networks.

This post is to place a high level my thoughts on Graph Neural Networks and how they relate to other models in Deep Learning sense.

But before we dive in let's talk about the purpose of graph neural networks.

What are Graph Neural Networks used for?
----------------------------------------

Broadly speaking there are two "forms" for graph neural networks (GNN): 

*  You want to do something at a node level
*  You want to do soemthing at a graph level

For example, if you have a graph of members within a community, perhaps you wish to use graph neural networks to predict what are the interest of a new incoming member with certain attributes. This would be a _node level_ GNN. 

On the other hand, you may want to predict the output of a graph-level structure. For example, if the graph was a collection of players on a sports team, the graph-level prediction may be used to estimate the effectiveness of this collection of players. 

With these concepts in mind, let's talk about how GNN structures are to be considered and built!


Convolutions
------------

In the "normal" neural networks we have convolution neural networks (CNN) - of course there are also the analogous graph neural networks. In general, for a CNN, you would have a filter which goes over an image. The output of this would be an image with the filter applied (this sounds obvious, but is worth repeating); e.g. if the filter was for edge detection, like Sobel filter, you would get an image with just the edges!

More formally, a convolution layer consists of:

* input which is a $W\times H \times D$ feature matrix, where $W, H$ are the width and height of the image, and $D$ is the number of channels in the corresponding image (typically 3 channels for RGB image or 1 for a grayscale image)
*  Hyperparameters in a convolution layer are the number of filters $K$, and their corresponding spatial extent $F$ (assuming that the special information is square), the size of the stride $S$ and the amount of zero padding $P$

As parameters are typically shared; then this produces trainable paramters of $F \times F \times D$ per filter with a total of $F \times F \times D \times K$ weights and $K$ biases

This produces an output of the size $W^\prime \times H^\prime \times D^\prime$ where
*  $W^\prime = (W - F + 2P)/S + 1$
*  $H^\prime = (H - F + 2P)/S + 1$
*  $D^\prime = K$

It is through this formulation whereby successive convolution layers can be applied to an image as part of ConvNet. 

**Keras/TF**


```py
from tensorflow.keras.layers import Dense, Conv2D, Input, MaxPooling2D, Lambda
from tensorflow.keras.models import Sequential, Model
import numpy as np 

model = Sequential()
model.add(Conv2D(5, 3, padding='same', input_shape = (50, 50, 3)))

model.predict(np.ones((1, 50, 50, 3))).shape
model.summary()
```

**PyTorch**


```py
import torch as th
import torch.nn as nn
import torch.nn.functional as F
import math

class ConvNet(nn.Module):
    def __init__(self):
        super(ConvNet, self).__init__()
        self.conv1 = nn.Conv2d(3, 5, 4, padding=1)

    def forward(self, x):
        x = self.conv1(x)
        return x

model = ConvNet()
model(th.ones(1, 3, 50, 50)).shape
print(model)
```

### Convolution Layer in GCN

In comparison, in the common scenario where a single filter is used; a GCN layers consist of:

*  two inputs, a feature matrix $X$ of size $N \times M$, where $N$ is the number of nodes and $M$ is the number of features, and a representative description of the graph structure typically in the form of adjacency matrix $A$ of size $N \times N$ or some function thereof - such as laplacian representation of the adjacency matrix. 
*  Hyperparameters in GCN represent the embedding size for the feature matrix $F$, in order to construct the weight matrix $W$. 

Then this produces trainable parameters $M \cdot F$. Note that this is invariant to the size of the underlying graph. 

The formulation of the output in a convolution layer within GCN is:

$$X^\prime =  A \cdot X \cdot W$$

This produces an output of size $M \times F$. It is through this formulation whereby successive convolution layers can be applied to a graph part of GCN.

In the scenario that a node-wise graph convolution network is built (see GCN) then

*  two inputs, a feature matrix $X$ of size $M$, where $M$ is the number of features, and a representative description of the graph structure typically in the form of adjacency matrix $A$ of size $N \times N$ or some function thereof - such as laplacian representation of the adjacency matrix. 
*  Hyperparameters in GCN represent the embedding size for the feature matrix $F$, in order to construct the weight matrix $W$. 

Then this produces trainable parameters $M \cdot F$. This is a reflection of the nature of invariance to the size of the underling graph. The construction of this node-wise level model requires that the mini-batch update produced is precisely the same size as the number of nodes in the graph. 


<!-- 

In the scenario where multiple filters are used, they can be used to combine and concatenate the underlying representation _(insert citation here including DGCNN algorithm)_. _We note that this is not in the canonical GCN paper by Kipf et al._ 

In this scenario, the formulation for stacked GCN layers consist of:

*  two inputs, a feature matrix $X$ of size $N \times M$, where $N$ is the number of nodes and $M$ is the number of features, and a representative description of the graph structure typically in the form of adjacency matrix $A$ of size $N \times N$ or some function thereof - such as laplacian representation of the adjacency matrix. 
*  Hyperparameters in GCN represent the embedding size for the feature matrix $F$, and number of filters used $K$, in order to construct the weight matrix $W$. 

Then this produces trainable parameters $M \cdot F \cdot K$. Note that this is invariant to the size of the underlying graph. 

The formulation of the output in a convolution layer within GCN is (with abuse of notation):

$$X^\prime =  A \cdot X \cdot W$$

where the last dot product is calculated as batch dot product. This produces an output of size $N \times F \times K$. In this setting, subsequent applications of GraphGCN can not be easily identified without the use of a pooling operator to "collapse" the underlying node-level feature representation

-->

**Keras/TF**


```py
from tensorflow.keras.layers import Layer

class GraphWiseConvolution(Layer):
    """
    Graph Convolution on a graph-by-graph,
    Performs convolution and outputs a graph
    """
    def __init__(self, units, use_bias=True, **kwargs):
        super(GraphWiseConvolution, self).__init__(**kwargs)
        self.units = units
        self.use_bias = use_bias
        
    def compute_output_shape(self, input_shape):
        graph_index = int(input_shape[0][0])
        node_feats = input(input_shape[0][1])
        
        units = self.units        
        return (
            graph_index,
            node_feats,            
            units
        )
    def build(self, input_shape):
        node_feats = int(input_shape[0][-1])
        
        self.kernel = self.add_weight(
            name="kernel",
            shape=(node_feats, self.units),
            initializer="glorot_uniform",
            trainable=True,
        )
        if self.use_bias:
            self.bias = self.add_weight(
                name="bias",
                shape=(1, self.units)
            )
        else:
            self.use_bias = None
    
    def call(self, inputs):
        features = inputs[0]
        basis = inputs[1] # adjancency information/basis information
        
        # perform one stack...
        supports = K.batch_dot(basis, features, 0)
        output = K.dot(supports, self.kernel)
        if self.use_bias:
            output += self.bias
        return output

class NodeWiseConvolution(Layer):
    """
    Graph Convolution on a node-level,
    Performs convolution on a node by node.
    
    You should use: batch_input_shape
    """
    def __init__(self, units, use_bias=True, **kwargs):
        super(NodeWiseConvolution, self).__init__(**kwargs)
        self.units = units
        self.use_bias = use_bias
        
    def compute_output_shape(self, input_shape):
        node_feats = input(input_shape[0][1])
        
        units = self.units        
        return (
            node_feats,            
            units
        )
    def build(self, input_shape):
        node_num = int(input_shape[0][0])
        node_feats = int(input_shape[0][-1])
        
        self.kernel = self.add_weight(
            name="kernel",
            shape=(node_feats, self.units),
            initializer="glorot_uniform",
            trainable=True,
        )
        if self.use_bias:
            self.bias = self.add_weight(
                name="bias",
                shape=(1, self.units)
            )
        else:
            self.use_bias = None
    
    def call(self, inputs):
        features = inputs[0]
        basis = inputs[1] # adjancency information/basis information
        
        # perform one stack...
        supports = K.dot(basis, features)
        output = K.dot(supports, self.kernel)
        if self.use_bias:
            output += self.bias
        return output


# Usage

nodes = 5
feat = 3
input_x = Input(shape=(nodes, feat))
input_a = Input(shape=(None, None))

output = GraphWiseConvolution(4)([input_x, input_a])

model = Model(inputs=[input_x, input_a], outputs=output)
model.predict([np.ones((1, 5, 3)), np.ones((1, 5, 5))]).shape
model.summary()

# node wise

nodes = 5
feat = 3
input_x = Input(batch_shape=(nodes, feat))
input_a = Input(batch_shape=(None, None))

output = NodeWiseConvolution(4)([input_x, input_a])

model = Model(inputs=[input_x, input_a], outputs=output)
model.predict([np.ones((5, 3)), np.ones((5, 5))]).shape
model.summary()
```

**PyTorch**


```py
class GraphWiseConvolution(nn.Module):
    def __init__(self, in_features, out_features, bias=True):
        super(GraphWiseConvolution, self).__init__()
        self.in_features = in_features
        self.out_features = out_features
        self.weight = nn.Parameter(th.FloatTensor(in_features, out_features))
        
        if bias:
            self.bias = nn.Parameter(th.FloatTensor(out_features))
        else:
            self.register_parameter('bias', None)
        self.reset_parameters()
        
    def reset_parameters(self):
        stdv = 1. / math.sqrt(self.weight.size(1))
        self.weight.data.uniform_(-stdv, stdv)
        if self.bias is not None:
            self.bias.data.uniform_(-stdv, stdv)
        
    def forward(self, features, adj):
        x = th.matmul(adj, features)
        output = th.matmul(x, self.weight)
        if self.bias is not None:
            return output + self.bias
        else:
            return output
    
    def __repr__(self):
        class_name = self.__class__.__name__ 
        in_features = self.in_features
        out_features = self.out_features
        return f"({class_name}) -> GCN(features: {in_features}, embedding: {out_features})"


class NodeWiseConvolution(nn.Module):
    def __init__(self, in_features, out_features, bias=True):
        super(NodeWiseConvolution, self).__init__()
        self.in_features = in_features
        self.out_features = out_features
        self.weight = nn.Parameter(th.FloatTensor(in_features, out_features))
        
        if bias:
            self.bias = nn.Parameter(th.FloatTensor(out_features))
        else:
            self.register_parameter('bias', None)
        self.reset_parameters()
        
    def reset_parameters(self):
        stdv = 1. / math.sqrt(self.weight.size(1))
        self.weight.data.uniform_(-stdv, stdv)
        if self.bias is not None:
            self.bias.data.uniform_(-stdv, stdv)
        
    def forward(self, features, adj):
        x = th.mm(adj, features)
        output = th.mm(x, self.weight)
        if self.bias is not None:
            return output + self.bias
        else:
            return output
    
    def __repr__(self):
        class_name = self.__class__.__name__ 
        in_features = self.in_features
        out_features = self.out_features
        return f"({class_name}) -> GCN(features: {in_features}, embedding: {out_features})"

model = NodeWiseConvolution(3, 4, True)
model(th.ones(5, 3), th.ones(5, 5)).shape
print(model)

model = GraphWiseConvolution(3, 4, True)
model(th.ones(1, 5, 3), th.ones(1, 5, 5)).shape
print(model)
```

Pooling Layer
---------------------

_Note that in the scenario where the graph network has node level predictions, pooling may not be needed_

Pooling is an important part of ConvNet in order the reduce the number of parameters between successive parts within ConvNet. In the analogous scenario, pooling in GCN can be used to either coarsen the graph, or alternatively reduce or sort a graph before it is fed to a fully connected layer. 

<!-- https://www.cse.wustl.edu/~ychen/public/DGCNN.pdf -->
<!-- https://arxiv.org/pdf/1806.08804.pdf -->

### Pooling in ConvNet

The pooling layer consists of

*  Input of size $W \times H \times D$
*  Require hyperparameters involving their spatial extent $F$ and the stride $S$, as well as the pooling operation (typically one of MAX, MEAN)

This produces an output in the pooling layer of $W^\prime \times H^\prime \times D^\prime$ where
*  $W^\prime = (W - F)/S + 1$
*  $H^\prime = (H - F)/S + 1$
*  $D^\prime = D$

Generally the choice of $F$ is one of 2 or 3, whilst the choice of $S$ is almost universally 2, as any other larger value results in loss of too much information.

**Keras/TF**

```
model = Sequential()
model.add(Conv2D(5, 3, padding='same', input_shape = (50, 50, 3)))
model.add(MaxPooling2D(2))

model.predict(np.ones((1, 50, 50, 3))).shape
model.summary()
```

**PyTorch**

```py
class PoolNet(nn.Module):
    def __init__(self):
        super(PoolNet, self).__init__()
        #self.conv1 = nn.Conv2d(3, 5, 4, padding=1)
        self.pool1 = nn.MaxPool2d(3, stride=2)

    def forward(self, x):
        #x = self.conv1(x)
        x = self.pool1(x)
        return x

model = nn.Sequential(*[
    ConvNet(),
    PoolNet()
])
model(th.ones(1, 3, 50, 50)).shape
print(model)
```

This covers broadly speaking the operations in Graph Convolution Networks
