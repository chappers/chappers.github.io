---
layout: post
category : 
tags : 
tagline: 
---

>  This is part of a series reviewing "Machine Learning Design Patterns" from O'Reilly

"Machine Learning Design Patterns" has come recommended to me, and after having a brief skim - I remain _slightly_ unconvinced. More so because the design patterns presume:

*  Large teams
*  Deep learning
*  Big data

What about if we are a small team? What if the datasets we're working on a very small? There are still MLOps considerations which don't hold weight when our pipelines are not differentiable. 

Of course the book tackles some of this - but I want to provide more "laid-back" treatment which doesn't presume deep learning. To that end, I want to think about the composition of pipelines as a series of (possibly learnable) transformations, but not necessarily differentiable. Can we still leverage some of these ideas? Can we stick with vanilla ML libraries without depending on deep learning and infrastructure heavy approaches?


Data Representation Design Patterns
===================================

Data representation refers to the challenge in which majority of ML libraries require numeric data as input. But not all data is "naturally" numeric. For example, data can be categorical, it can be text based, it can be images, or audio as well!

Numeric data
------------


In the first instance, `scikit-learn` provides pipeline-able representations for scaling pipelines. These are important for problems which leverage regularization and gradient descent methods as part of the learning process to ensure that features have consistent magnitudes of weights for algorithms including `k-means`. 

```py
from sklearn import datasets
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline


X, y = datasets.load_diabetes(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

model = make_pipeline(LinearRegression())

model_scale.fit(X_train[:, [2]], y_train)
model.fit(X_train[:, [2]], y_train)
```

The other listed approaches including clipping, winsorizing (under `RobustScaler`), box-cox (under `PowerTransformer`) will also work out of the box in `scikit-learn`. What is important to note is the performance difference and improvements are most likely _invisible_ if we are using batch based supervised learning. 

There are a few things to note:

*  `StadardScaler` - supports adaptive learning
*  `MinMaxScaler` - supports adaptive learning
*  `RobustScaler` - batch only
*  `PowerTransformer` - batch only
*  `PolynomialFeatures` - (aka feature cross), batch only
*  `FeatureUnion` - (aka multimodal input, via embeddings), batch only

For ragged arrays (e.g. time series), generation of statistics can be done using the `kats` library from Facebook.


```py
from kats.tsfeatures.tsfeatures import TsFeatures
from datetime import datetime
import pandas as pd

model = TsFeatures()
model.transform(pd.DataFrame({
    'time': [datetime(1949, 1, 1), datetime(1949, 1, 2), datetime(1949, 1, 4), 
    datetime(1949, 1, 5), datetime(1949, 1, 6), datetime(1949, 1, 8)],
    'value': [2100, 15200, 230000, 1200, 300, 532100]
}))
```

### Hashing Trick

_Learning representations_ is an interesting topic, and can be explored in packages like `category_encoders`. 

The book provides a solid overview of hashing, as well as the downsides. It can be provided in a pipeline via

```py
from sklearn.datasets import load_boston
from category_encoders.hashing import HashingEncoder

# prepare some data
bunch = load_boston()
X = pd.DataFrame(bunch.data, columns=bunch.feature_names)
y = bunch.target

enc = HashingEncoder(cols=['CHAS', 'RAD']).fit(X)
# enc.transform(X)
```

Though in general, I don't think I would recommend usage of the Hashing trick in production - dealing with out of distribution/unseen categories probably should be done explicitly rather than implicitly?

Non-numeric data
-----------------

### Text data

This one is nearly impossible to accomplish in vanilla scikit-learn. There are many libraries which propose ways to have learnable embeddings. The "simpliest" one which I have found to be sufficiently effective is to convert the problem towards "Latent Semantic Indexing" style embeddings. 

I'll leave this as an exercise for the reader, as I'm more or less proposing for text embeddings to use BoW approach with character level ngrams. 

### Image data

This is especially difficult to justify in the present age of deep learning. Whilst BoW still have their place and out-perform deep learning on small tabular tasks, image recognition style tasks perform much strong using deep learning. 

The easiest approach is to use pre-trained networks to generate embeddings, but if we insist, we could leverage `daisy` as a feature generation technique: https://scikit-image.org/docs/stable/api/skimage.feature.html#skimage.feature.daisy

Reframing the problem
=====================

There are several scikit-learn style libraries which will accomplish these tasks out of the box. Although they are not "one to one" replacement for deep learning (deep learning is extremely flexible) they do achieve similar kinds of outcomes:

*  ClassifierChain - for chaining classification models one after the other
*  MultiOutputClassifier - transforms a ML model to support multiple outputs by cloning the estimator multiple times. 

As the models themselves are not differentiable, there will not be "learning" that is passed on from multiple "sinks". 

Better Training Loops in Scikit-Learn
=====================================

It is good practise to use train, test, validation splits - but it isn't obvious how to achieve this in `scikit-learn`. What is also problematic is the default hyperparameter tuning is fair "expensive", even when you want something simple. 

The remedy is simply...not to use the default options. 

```py
from sklearn.model_selection import train_test_split, GridSearchCV, ShuffleSplit
from sklearn import datasets
from sklearn.linear_model import Lasso
import numpy as np


X, y = datasets.load_diabetes(return_X_y = True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, random_state=7
)

model_cv = GridSearchCV(
    estimator=Lasso(),
    param_grid = {
        'alpha': np.logspace(-4, -0.5, 100)
    },
    cv = ShuffleSplit(n_splits=1, random_state = 5)  # does validation split
)
model_cv.fit(X_train, y_train)
print("Score:", model_cv.score(X_test, y_test))
```

Irreplaceable Items
===================

Some things can't be replaced. This includes topics and thoughts on:

*  Feature Extract vs Transfer learning
*  Distributed training/Federated learning (doesn't truly make sense - though can be done on simple linear models that use SGD)


Model Serving
=============

I think stateless serving is more of a software pattern than ML. If you have stateful models in production you're doing something wrong because I can't trust your model to be consistent!

The examples are also heavily tensorflow specific, which have no bearing if you're deploying an API from scratch.

Many of the challenges are infrastructure specific, and should be taken only at face value. 

Reproducibility Design Patterns
===============================

Transform
---------

This is my favourite pattern, and comes free from scikit-learn. Enough said. 

Repeatable Splitting
--------------------

This one is quite interesting. As we would probably need to roll your own to do this. It could be done with a combination of `HashingEncoder()` with `mod`. This is similar to the `BIGQUERY` examples which were presented. 

One could use `pyfarmhash` as well

```py
import farmhash
farmhash.fingerprint64('alphabet')

# to sample - using example in book for 80%, `mod(., 10) < 8 -- 80% for TRAIN`
# farmhash.fingerprint64('alphabet') % 10 < 8 
```

Bridged Schema
--------------

I believe this is a variation of the strangler pattern for deprecating old code bases. The idea is to leave the existing approach as is, and have the new one running in parallel. The nuance being to (selectively) recast old data to the new schema via remapping. This can be handled in RDMs or other approaches. 

When there is new data available the book does make good suggestions around how to manage it, particularly around "life-long" learning. But I'll leave this as a topic for another day, as I'm partial to Hoeffding racing for handling and deprecating models of this nature in production. 


