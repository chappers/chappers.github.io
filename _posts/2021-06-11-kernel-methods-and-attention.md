---
layout: post
category:
tags:
tagline:
---

In the previous post, we looked at SpaCy's default classification model. But what is it actually? The short version is it uses the "Hierarachical Attention Networks" (Yang, Zichao et al, ACL 2016).

In this post, we won't directly talk about this model, but instead talk informally about attention and their relationship to kernel methods. This post is based on slides here, but I'll try to give my own (informal) flavour "Attention: the Analogue of Kernels in Deep Learning".

# What is a Kernel?

You read that Kernels are a measure of similarity - but what does this actually mean?

If we have a distance metric $d$, then we say if $d(a, b) < d(a, c)$ then $a$ is more similar to $b$ than to $c$. However for kernels this is flipped around. If we have a kernel $k$, then $k(a, b) > k(a, c)$ suggests that $a$ is more similar to $b$ than to $c$.

One kind of "pattern" you might see when converting kernels to similarity measures are transformations like (if $D$ is a measure of distance), $e^{-D}$ or $\frac{1}{D}$.

This intuition is important, as you will see these patterns pop up in this like [radial basis functions](https://en.wikipedia.org/wiki/Radial_basis_function_kernel).

# Attention is just a Kernel (Projection)

There are lots of different post expressing this, the simpliest variation I've seen is this:

- Source vector $s$
- Target vector $t$

Then a transformer is, where $\alpha$ is attention weights and $t^\prime$ is the output:

$$a_{ij} = \frac{(W^Qt_i)^\top(W^K s_j)}{\sqrt{d}}$$
$$\alpha_{ij} = \frac{\exp(a_{ij})}{\sum_{\forall j} \exp(a_{ij})}$$
$$t^\prime_i = \sum_{\forall j} \alpha_{ij} W^V s_j$$

In NLP one possible formulation is if hypothetically a document is represented through a word count vector, where the $i$-th element is the count of the $i$-th word in our vocabulary, then the _attention weight_ $i, j$ is the amount of weight the $i$-th word has in relation to the $j$-th word. Note that in general the scores would not be symetric, that is $a_{ij} \neq a_{ji}$ - (otherwise we'll just be learning an identity matrix).

When you put everything together, this is just a series of matrix multiplications, which demonstrates how this is "learnable".

With this you have a project of itself which attends to, or provides an embedding which shows "importance".

**An aside**

How might we project a vector representing a whole document to a set of weights which show importance in an unsupervised manner?

Simple answer is TFIDF! However tfidf doesn't take into account "context", whereas in this case the presence of other words in the document would change the weighting score.

## A silly example

To demostrate this, let's try to build an "attention" model, where only $W^V$ is learned, and $W^Q, W^K$ are generated randomly.

```py
import numpy as np
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.linear_model import SGDRegressor
from sklearn.multioutput import MultiOutputRegressor
from sklearn.pipeline import make_pipeline
categories = ['comp.graphics', 'sci.med']

twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)

class SimpleAttention(object):
    def __init__(self, n_components = 10, vectorizer = CountVectorizer):
        self.model = MultiOutputRegressor(SGDRegressor(random_state=123))
        self.cv = vectorizer(max_features=n_components)

    def fit(self, X, y=None):
        X_trans = self.cv.fit_transform(X).A
        vocab_size = X_trans.shape[1]

        # don't learn these for sake of example
        self.W_k = np.random.normal(size = (vocab_size, vocab_size))
        self.W_q = np.random.normal(size = (vocab_size, vocab_size))

        a = ((X_trans.dot(self.W_k)) * (X_trans.dot(self.W_q)))
        a /= np.max(a)
        alpha = np.exp(a)/(np.exp(a).sum(axis=1)[:, np.newaxis])
        X_feat = np.multiply(alpha, X_trans)

        self.model.fit(X_feat, X_trans)
        return self

    def transform(self, X, y=None):
        X_trans = self.cv.transform(X).A

        a = ((X_trans.dot(self.W_k)) * (X_trans.dot(self.W_q)))
        a /= np.max(a)  # this is a hack just to make sure I don't blow up the computer
        alpha = np.exp(a)/(np.exp(a).sum(axis=1)[:, np.newaxis])
        X_feat = np.multiply(alpha, X_trans)
        return self.model.predict(X_feat)

    def fit_transform(self, X, y=None):
        return self.fit(X, y).transform(X, y)


attn = SimpleAttention(50)
tfidf = TfidfVectorizer(max_features=50).fit(twenty_train.data)
cv = CountVectorizer(max_features=50).fit(twenty_train.data)

embed_attn = attn.fit_transform(twenty_train.data)
tfidf_embed = tfidf.transform(twenty_train.data).A
cv_embed = cv.transform(twenty_train.data).A


# train a model, and see how they perform.
from sklearn.svm import LinearSVC
embed_svm = LinearSVC().fit(embed_attn, twenty_train.target)
tfidf_svm = LinearSVC().fit(tfidf_embed, twenty_train.target)
cv_svm = LinearSVC().fit(cv_embed, twenty_train.target)

print(embed_svm.score(embed_attn, twenty_train.target))
print(tfidf_svm.score(tfidf_embed, twenty_train.target))
print(cv_svm.score(cv_embed, twenty_train.target))

# 0.7589134125636672
# 0.7758913412563667
# 0.767402376910017
```

## Putting it altogether

Now the SpaCy variation doesn't actually operate like the above. The model is more like using a feature union. To put it into a pipeline, it would look like:

```py
import numpy as np
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.linear_model import SGDRegressor
from sklearn.multioutput import MultiOutputRegressor
from sklearn.pipeline import make_pipeline, FeatureUnion
from sklearn.base import TransformerMixin
from sklearn.svm import LinearSVC
categories = ['comp.graphics', 'sci.med']

twenty_train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True, random_state=42)

class SimpleAttention(TransformerMixin):
    def __init__(self, n_components = 10, vectorizer = CountVectorizer):
        self.model = MultiOutputRegressor(SGDRegressor(random_state=123))
        self.cv = vectorizer(max_features=n_components)

    def fit(self, X, y=None):
        X_trans = self.cv.fit_transform(X).A
        vocab_size = X_trans.shape[1]

        # don't learn these for sake of example
        self.W_k = np.random.normal(size = (vocab_size, vocab_size))
        self.W_q = np.random.normal(size = (vocab_size, vocab_size))

        a = ((X_trans.dot(self.W_k)) * (X_trans.dot(self.W_q)))
        a /= np.max(a)
        alpha = np.exp(a)/(np.exp(a).sum(axis=1)[:, np.newaxis])
        X_feat = np.multiply(alpha, X_trans)

        self.model.fit(X_feat, X_trans)
        return self

    def transform(self, X, y=None):
        X_trans = self.cv.transform(X).A

        a = ((X_trans.dot(self.W_k)) * (X_trans.dot(self.W_q)))
        a /= np.max(a)  # this is a hack just to make sure I don't blow up the computer
        alpha = np.exp(a)/(np.exp(a).sum(axis=1)[:, np.newaxis])
        X_feat = np.multiply(alpha, X_trans)
        return self.model.predict(X_feat)



pipeline = make_pipeline(FeatureUnion([
    ('sa', SimpleAttention(50)),
    ('tfidf', TfidfVectorizer(max_features=50))
]),
    LinearSVC())
pipeline.fit(twenty_train.data, twenty_train.target)
print(pipeline.score(twenty_train.data, twenty_train.target))
```

# Final Thoughts

We gave a simple example of how we can think of attention models, and how SpaCy integrates them. We showed the analogy to what we might be more familiar with in the ML space.

There are some things which are unaddressed!

- use of GRUs - most word embeddings used are a sequence of tokens; not a bag of words!

We'll leave that as an exercise for the reader. I purposely excluded it as it detracts from learning/understanding more.
