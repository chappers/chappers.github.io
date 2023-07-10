---
layout: post
category:
tags:
tagline:
---

How can we architect zero shot learning?

Perhaps the easiest way is to leverage existing models with ANN. Let me explain:

1.  Find an existing model and extract the embedding
2.  Use the embedding combined with kNN to learn new concepts quickly

Hopefully this approach can quickly scale out datasets to different scenarios with little effort beyond engineering.

One of the key issues is the lack of "easy" libraries/applications which can do this successfully.

Some code:

```py
from tensorflow.keras.applications.mobilenet import MobileNet
from tensorflow.keras.applications.mobilenet import preprocess_input
from tensorflow.keras.preprocessing import image

from tensorflow.keras.preprocessing import image_dataset_from_directory
import numpy as np
from annoy import AnnoyIndex

model = MobileNet(weights='imagenet', include_top=False)
```

We use `MobileNet` for building a one-shot learner using a MetaLearning library `annoy`.

```py
# build a dataset
data = image_dataset_from_directory(directory='path/to/images', image_size=(224, 224), batch_size=batch_size, shuffle=False, labels=None)

# stuff for ann
t = None
counter = 0
n_trees = 50

for x in data:
    x = preprocess(x)
    features = model.predict(x)
    batch_size = features.shape[0]
    vectors = features.reshape(batch_size, -1)

    if t is None:
        f = vectors.shape[1]
        t = AnnoyIndex(f, 'angular')
    for i in range(batch_size):
        t.add_item(counter, vectors[i, :])
        counter += 1

t.build(n_trees)
t.save('myann.ann')
```

Then to evaluate we would do something like

```py
img = image.load_img(img_path, target_size=(224, 224))
x = image.imag_to_array(img)
x = np.expand_dims(x, axis=0)
x = preprocess(x)
x = model.predict(x)

k = 50
output = t.get_nns_by_vector(x[0, :], k)
# these are the k nearest neighbours
# infer label based on output labels
```
