---
layout: post
category:
tags:
tagline:
title: Onnx API and Retreiving Intermediary Output
---

It's quite frustrating trying to navigate through comments and posts on how to do "simple" things, so I'll just document how this is currently done, as well as _why_ it is the way it is. Of course down the track maybe these issues get resolved - however for now, let's run through this!

**Problme Statement**: How do we get intermediary layer output as part of an ONNX model?

This approach has many issues raised, with no completely working code. Some of the issues listed are here:

- https://github.com/microsoft/onnxruntime/issues/2119
- https://github.com/microsoft/onnxruntime/issues/1455

and many more. To do this in a naive way is:

```py
import onnxruntime.backend as backend
from onnx import load
import numpy as np

model = load("squeezenet1.1.onnx")
rep = backend.prepare(model, 'CPU')

# the input for this is a picture of size 224 x 224 with all values moved in 0, 1.
# try with random input first
X = np.random.normal(size=(1, 3, 224, 224)).astype(np.float32)
X += np.abs(np.min(X))
X /= np.max(X)

rep.run(X)[0].shape
```

If you try running the model, as the parameters in the graph nodes are inferred and not known before the graph is built - there is no way to directly output the intermediary layer. Another item to consider we completely rebuild the ONNX models which stops at the target intermediary layer - however we won't be going through this approach in this post.

## Solution

The actual approach is to use intermediate inference through the `InferenceSession`. The "secret sauce" is encoded in this portion:

```py
# take "-2" to be the 2nd last layer
intermediate_tensor_name = model.graph.node[-2].name
intermediate_layer_value_info = helper.ValueInfoProto()
intermediate_layer_value_info.name = intermediate_tensor_name

# the output will be shown in list [1]
model.graph.output.extend([intermediate_layer_value_info])
save(model, model_path)
```

From here, we can load the model, and then use `rt.InferenceSession`.

Putting it altogether would look like this:

```py
import onnxruntime as rt
import onnxruntime.backend as backend
from onnx import load, save, helper
import numpy as np

import cv2

model = load("squeezenet1.1.onnx")
model_path = "squeezenet-intermediary.onnx"

intermediate_tensor_name = model.graph.node[-2].name
intermediate_layer_value_info = helper.ValueInfoProto()
intermediate_layer_value_info.name = intermediate_tensor_name
model.graph.output.extend([intermediate_layer_value_info])
save(model, model_path)

model = load(model_path)
sess = rt.InferenceSession(model_path)

# now try doing a cat jpeg extraction and make it channels first...
# I know this might not be precisely what Squeezenet does for normalizing
img = cv2.resize(cv2.imread('cat.jpeg'), (224, 224)).transpose(2, 0, 1) / 256
batch_img = np.stack([img]).astype(np.float32)

# our feature vector!
output = sess.run(None, {'data': batch_img})
feature_vector = output[1].reshape(batch_img.shape[0], -1)  # index 1 is intermediate layer
```
