---
layout: post
title: What is SpaCy's default Text Categorization Model?
category : 
tags : 
tagline: 
---

SpaCy makes text classification easy. How easy? Let's compare!

Start with scikit-learn version

```py
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import SGDClassifier
from sklearn.pipeline import make_pipeline
categories = ['comp.graphics', 'sci.med']

twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)
pipeline = make_pipeline(TfidfVectorizer(), SGDClassifier())
pipeline.fit(twenty_train.data, twenty_train.target)

pipeline.predict(['Polygon Splitting algo', 'doctor patient relationship problem'])
```

For SpaCy, we can use this:

<!-- 

```py
from sklearn.datasets import fetch_20newsgroups
import spacy
from spacy.util import minibatch
from spacy.training import Example
from spacy.pipeline.textcat import DEFAULT_SINGLE_TEXTCAT_MODEL

categories = ['comp.graphics', 'sci.med']
twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)
nlp = spacy.load("en_core_web_sm")
nlp.add_pipe('textcat', last=True)
textcat = nlp.get_pipe("textcat")

textcat.add_label("label")

train_data = []
for doc, label in zip(twenty_train.data, twenty_train.target.tolist()):
    train_data.append(
        (doc, {'cats': {'label': label}})
    )

# optimizer = nlp.begin_training()  # use this if we called spacy.blank.load("en")
optimizer = nlp.create_optimizer()
losses = {}
for batch in minibatch(train_data, size=8):
    for text, annotations in batch:
        doc = nlp.make_doc(text)
        example = Example.from_dict(doc, annotations)
        nlp.update([example], drop=0.35, sgd=optimizer, losses=losses)

doc = nlp(u'It is good.')
print(doc.cats)

```


```py
from sklearn.datasets import fetch_20newsgroups
import spacy
from spacy.util import minibatch
from spacy.training import Example
from spacy.pipeline.textcat import DEFAULT_SINGLE_TEXTCAT_MODEL

categories = ['comp.graphics', 'sci.med']
twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)
nlp = spacy.load("en_core_web_sm")
if "textcat" not in nlp.pipe_names:
    textcat = nlp.create_pipe(
        "textcat", config={"architecture": "simple_cnn"}
    )
    nlp.add_pipe(textcat, last=True)
else:
    textcat = nlp.get_pipe("textcat")

textcat.add_label("pos")
textcat.add_label("neg")

# Train only textcat

training_excluded_pipes = [
    pipe for pipe in nlp.pipe_names if pipe != "textcat"
]

with nlp.disable_pipes(training_excluded_pipes):
    optimizer = nlp.begin_training()
    # Training loop
    print("Beginning training")
    for i in range(10):
        loss = {}
        random.shuffle(training_data)
        batches = minibatch(training_data, size=batch_sizes)
        for batch in batches:
            text, labels = zip(*batch)
            nlp.update(
                text,
                labels,
                drop=0.2,
                sgd=optimizer,
                losses=loss
            )



train_data = []
for doc, label in zip(twenty_train.data, twenty_train.target.tolist()):
    train_data.append(
        (doc, {'label': label})
    )

# optimizer = nlp.begin_training()  # use this if we called spacy.blank.load("en")
optimizer = nlp.create_optimizer()
losses = {}
for batch in minibatch(train_data, size=8):
    for text, annotations in batch:
        doc = nlp.make_doc(text)
        example = Example.from_dict(doc, annotations)
        nlp.update([example], drop=0.35, sgd=optimizer, losses=losses)

doc = nlp(u'It is good.')
print(doc.cats)

```

```py
import spacy
import random
import json
from spacy.training import Example

TRAINING_DATA = [
    ["My little kitty is so special", {"KAT": True}],
    ["Dude, Totally, Yeah, Video Games", {"KAT": False}],
    ["Should I pay $1,000 for the iPhone X?", {"KAT": False}],
    ["The iPhone 8 reviews are here", {"KAT": False}],
    ["Noa is a great cat name.", {"KAT": True}],
    ["We got a new kitten!", {"KAT": True}]
]

nlp = spacy.blank("en")
category = nlp.add_pipe("textcat")
category.add_label("KAT")

# Start the training
nlp.begin_training()

# Loop for 10 iterations
for itn in range(100):
    # Shuffle the training data
    random.shuffle(TRAINING_DATA)
    losses = {}
    
    # Batch the examples and iterate over them
    for batch in spacy.util.minibatch(TRAINING_DATA, size=1):
        for text, annotations in batch:
            doc = nlp.make_doc(text)
            example = Example.from_dict(doc, annotations)
            # annotations = [{"cats": entities} for text, entities in batch]
            example = Example.from_dict(texts, annotations)
            nlp.update(example, losses=losses)
        if itn % 20 == 0:
            print(losses)
```
-->


```py
import spacy
import random
from sklearn.datasets import fetch_20newsgroups

categories = ['comp.graphics', 'sci.med']
twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)


train_data = []
for doc, label in zip(twenty_train.data, twenty_train.target.tolist()):
    train_data.append(
        (doc, {'label': label})
    )


nlp = spacy.blank("en")  
# or
# nlp = spacy.load("en_core_web_sm")
category = nlp.create_pipe("textcat")
category.add_label("label")
nlp.add_pipe(category)

# Start the training
nlp.begin_training()


# Loop for 100 iterations
for itn in range(1):
    # Shuffle the training data
    random.shuffle(train_data)
    losses = {}
    
    # Batch the examples and iterate over them
    for batch in spacy.util.minibatch(train_data, size=64):
        texts = [nlp(text) for text, entities in batch]
        annotations = [{"cats": entities} for text, entities in batch]
        nlp.update(texts, annotations, losses=losses)
    if itn % 20 == 0:
        print(losses)

texts = ['Polygon Splitting algo', 'doctor patient relationship problem']
for t in texts:
    doc = nlp(t)
    print(doc.cats)
```

Now let's tidy this up a bit to make it look more scikit-like

```py
import spacy
import numpy as np
from sklearn.datasets import fetch_20newsgroups
from sklearn.base import ClassifierMixin


class SpacyTextCat(ClassifierMixin):
    def __init__(self, pack="en", n_classes = None, cats=None, batch_size=64, iters=1000):
        # TODO support multi-label and multiclass properly
        if pack == "en":
            self.nlp = spacy.blank("en")  
        else:
            self.nlp = spacy.load(pack)
        self.category = self.nlp.create_pipe("textcat")

        self.category_groups = []

        if cats is not None:
            n_classes = len(cats)
            for c in cats:
                self.category.add_label(c)
        elif n_classes is not None:
            cats = []
            for c in range(n_classes):
                self.category.add_label(f"class{c}")
                cats.append(f"class{c}")

        else:
            n_classes = 2
            self.category.add_label("label")
            cats = "label"

        self.nlp.add_pipe(self.category)

        # Start the training
        self.nlp.begin_training()

        self.iters = iters
        self.cats = cats
        self.n_classes = n_classes if n_classes is not None else 2
        self.is_binary = self.n_classes == 2
        self.batch_size = batch_size
            
    
    def preprocess_data(self, X, y):
        # reformat data...
        #X = np.array(X)
        y = np.array(y)

        train_data = []
        for idx in range(len(X)):
            x_sample = X[idx]
            if len(y.shape) == 1:
                y_sample = y[idx]
            else:
                y_sample = y[idx, :]

            if self.cats == "label":
                # binary
                y_sample = {'label': y_sample}
            elif type(y_sample) == np.ndarray:
                y_sample = {y_: 1 for y_ in y_sample}
            else:
                y_sample = {y_sample : 1}
            train_data.append((x_sample, y_sample))
        return train_data

    def fit(self, X, y, iters=None):
        train_data = self.preprocess_data(X, y)
        if iters is None:
            iters = self.iters

        losses = {}
        
        for _ in range(iters):
            for batch in spacy.util.minibatch(train_data, size=self.batch_size):
                texts = [self.nlp(text) for text, response in batch]
                annotations = [{"cats": response} for text, response in batch]
                self.nlp.update(texts, annotations, losses = losses)

    def partial_fit(self, X, y, iters=None):
        # to do reset weights if fit is called...
        return self.fit(X, y, iters)

    def predict(self, X):
        # doesn't adhere properly to interface, but we'll leave it
        # for now
        return [self.nlp(x).cats for x in X]


categories = ['comp.graphics', 'sci.med']
twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)

clf = SpacyTextCat(iters=1)
clf.fit(twenty_train.data, twenty_train.target)
pred = clf.predict(twenty_train.data)
print(pred)
```

This seems like a suitable approach at a brief level - though it might be more prudent to have custom code rather than trying to make it generic, but it can be wrapped up in a cleaner module rather than Spacy specific code everywhere. 

<!-- 

import spacy
import numpy as np
import pandas as pd
from sklearn.base import ClassifierMixin
from spacy.training import Example
from thinc.api import get_array_module, Model, Optimizer, set_dropout_rate, Config

from spacy.pipeline.textcat import DEFAULT_SINGLE_TEXTCAT_MODEL

single_label_default_config = """
[model]
@architectures = "spacy.TextCatEnsemble.v2"

[model.tok2vec]
@architectures = "spacy.Tok2Vec.v2"

[model.tok2vec.embed]
@architectures = "spacy.MultiHashEmbed.v2"
width = 64
rows = [2000, 2000, 1000, 1000, 1000, 1000]
attrs = ["ORTH", "LOWER", "PREFIX", "SUFFIX", "SHAPE", "ID"]
include_static_vectors = false

[model.tok2vec.encode]
@architectures = "spacy.MaxoutWindowEncoder.v2"
width = ${model.tok2vec.embed.width}
window_size = 1
maxout_pieces = 3
depth = 2

[model.linear_model]
@architectures = "spacy.TextCatBOW.v1"
exclusive_classes = true
ngram_size = 1
no_output_layer = false
"""

single_label_bow_config = """
[model]
@architectures = "spacy.TextCatBOW.v1"
exclusive_classes = true
ngram_size = 1
no_output_layer = false
"""

DEFAULT_SINGLE_TEXTCAT_MODEL = Config().from_str(single_label_default_config)["model"]
DEFAULT_SINGLE_TEXTBOW_MODEL = Config().from_str(single_label_bow_config)["model"]

config = {"threshold": 0.5}
config = {**config, **Config().from_str(single_label_bow_config)}


class SpacyTextCat(ClassifierMixin):
    def __init__(self, pack="en", n_classes = None, batch_size=64, iters=1000):
        # spacy 3 has separate interfaces for classification vs multilabel...
        if pack == "en":
            self.nlp = spacy.blank("en")  
        else:
            self.nlp = spacy.load(pack)
        self.category = self.nlp.add_pipe("textcat", config=config, last=True)
        n_classes = 2 if n_classes is None else n_classes
            
        for c in range(n_classes):
            self.category.add_label(f"label{c}")

        # Start the training
        self.nlp.begin_training()

        self.iters = iters
        self.n_classes = n_classes
        self.batch_size = batch_size
            
    
    def preprocess_data(self, X, y):
        # reformat data...
        #X = np.array(X)
        y = np.array(y)

        train_data = []
        for idx in range(len(X)):
            x_sample = X[idx]
            if len(y.shape) == 1:
                y_sample = y[idx]
            else:
                y_sample = y[idx, :]

            y_sample = {f'label{y_sample}': 1}
            train_data.append((x_sample, y_sample))
        return train_data

    def fit(self, X, y, iters=None):
        train_data = self.preprocess_data(X, y)
        if iters is None:
            iters = self.iters

        losses = {}
        
        for _ in range(iters):
            for batch in spacy.util.minibatch(train_data, size=self.batch_size):
                texts = [self.nlp(text) for text, response in batch]
                annotations = [{"cats": response} for text, response in batch]
                # print(annotations)
                examples = [Example.from_dict(text, annon) for text, annon in zip(texts, annotations)]
                self.nlp.update(examples, losses = losses)

    def partial_fit(self, X, y, iters=None):
        # to do reset weights if fit is called...
        return self.fit(X, y, iters)

    def predict(self, X):
        # doesn't adhere properly to interface, but we'll leave it
        # for now
        return [self.nlp(x).cats for x in X]



-->
