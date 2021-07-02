---
layout: post
category : 
tags : 
tagline: 
---

The paper ["Learning Deep ResNet Blocks Sequentially using Boosting Theory"[1] is a paper to appear in ICML 2018](https://arxiv.org/abs/1706.04964). At a high level, the paper talks through the relationship between ResNet and Boosting, and how a ResNet can be trained in a single forward pass in a manner to how boosted models are trained in lieu of back propagation. The idea here is that we can then train non-differentiable layers in a neural network. 

Instead of focusing on that side of boosting and ResNet, in this post, I thought I'll talk through some of the interesting comments around the relationship between boosting and ResNet which was hinted at. 

>  The key difference is that boing is an ensemble of estimated hypotheses whereas ResNet is an ensemble of estimated feature representations $\sum_{t=0}^T f_t(g_t(x))$. To solve this problem, we introduce an auxiliary linear classifier $w_t$ on top of each residual block construct a hypothesis module. 

Where $f_t(x)$ is a non-linear unit, and $g_t(x)$ of the $t$th module to be the output of the previous module, i.e. $$g_{t+1}(x) = f_t(g_t(x)) + g_t(x)$$

## Relation between Residual Functions and Identity Mapping

If we consider the residual function

$$F(X) = H(X) - x$$

Then this is alternative to solving $H(x) = F(x) + x$. 

## Relation to Boosting

In order to break this statement down (as there was not much further discussion due to the proposed Boosted ResNet architecture not using multiple auxiliary layers), let's go through some background to how boosting may be conceive.

The simplist formulation of a boosting algorithm (see [Boosting](https://mitpress.mit.edu/books/boosting)) can be formulated as

$$F_T(x) = \sum_{t=1}^T \alpha_t h_t(x)$$

For some choice of weight $\alpha_t$ and a hypothesis function $h_t$. The hypothesis function is also a _base classifier_ (i.e. a model on its own right[2]). A boosted classifier can be framed as weighted sum of base classifiers based on the $\alpha_t$ which measures the importance that is assigned to $h_t$[2]. 

Then if we move this back to the notation in ResNet paper, we can easily see that

$$\begin{align}
F_T(x) &= w^T g_{T+1}(x)\\
&= w^T (f_T(g_{T}(x)) + g_T(x))\\
&= F_{T-1} + w^T f_T(g_{T}(x))\\
\end{align}$$

![rollup](/img/boost-resnet/resnet.jpg)

### Simple Keras implementation

To finish off this post, here is a simple boosted logistic regression to motivate the idea

```py
import pandas as pd
import numpy as np
from sklearn.datasets import make_classification

import tensorflow  as tf

from keras.models import Sequential
from keras.layers import Dense
from keras.layers import Flatten
from keras.layers import Input
from keras.layers import Reshape
from keras.models import Model

from keras import backend as K
from keras import initializers, layers
from keras.engine.topology import Layer

from keras.utils import to_categorical

X, y_ = make_classification()
y = to_categorical(y_)

# traditional resnet, feeding the output of the previous model
# into the next step would be residual fitting - more like gbm (fitting to residuals)
# though of not strictly gbm model(how do a show/demonstrate this?), as we're not computing hypothesis h_t 
# to the negative gradient of the loss function
# these can be defined functionally (left as an exercise for reader)

# using more of a resnet boosting paper formulation

main_input = Input(shape=(20,), name='main_input')

# first model ensemble
# this is the shared layer
aux_weight = Dense(2, name='aux_resnet1', activation='softmax')

resnet1 = Dense(20, name='resnet1')(main_input)
aux_resnet1 = aux_weight(resnet1)
resnet1_g = layers.add([resnet1, main_input])
resnet1_out = layers.Activation('relu')(resnet1_g)

# second model ensemble
resnet2 = Dense(20, name='resnet2')(resnet1_out)
aux_resnet2 = aux_weight(resnet2)
resnet2_g = layers.add([resnet2, resnet1_out])
resnet2_out = layers.Activation('relu')(resnet2_g)

pred_out = Dense(2, name='main_output', activation='softmax')(resnet2_out)

model = Model(inputs=[main_input], outputs=[pred_out, aux_resnet1, aux_resnet2])
model.compile(optimizer='adam', 
              loss='categorical_crossentropy',
              metrics=['accuracy'])

hist = model.fit({'main_input': X}, {'main_output': y, 'aux_resnet1': y, 'aux_resnet2': y}, epochs=1000, verbose=0)
print(pd.DataFrame(hist.history).iloc[-1])
model.summary()

```

**References**

1. Furong Huang, Jordan Ash, John Langford, Robert Schapire, "Learning Deep ResNet Blocks Sequentially using Boosting Theory", 2018, ICML
2. Robert Schapire, Yoav Freund, "Boosting - Foundations and Algorithms", 2012, MIT (see pages 5-6)