---
layout: post
category:
tags:
tagline:
---

When designing interfaces for software, a commonly cited one is the "Principle of Least Astonishment"; interfaces should work the way that you want them to!

However sometimes there are weird oddities which pop up where people need to be aware of. One of these is the current (in development) interface for gensim to scikit learn.

Imagine you are doing a TFIDF model in Scikit learn right now. The code to do this looks like:

```py
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.pipeline import make_pipeline

pipeline = make_pipeline(
    CountVectorizer(),
    TfidfTransformer())

# data : [String]
pipeline.fit(data)
```

It is clear what the input is: a list of documents, which get transformed by the count vectorizer and then to the tfidf transformer. When gensim comes out with a scikit api interface for gensim, one might hope that it would work if I simply replace `TfidfTransformer` with gensim's `gensim.sklearn_api.tfidf.TfidfTransformer`.

Unfortunately this doesn't work.

Instead to get this working, we would have to do something akin to:

```py
from gensim.sklearn_api import tfidf
from gensim.utils import simple_preprocess
from gensim import corpora

def gensim2np(trans_out):
    def convert_to_numpy(id_val, idx):
        data = [x[1] for x in id_val]
        col = [x[0] for x in id_val]
        row = [idx for x in id_val]
        return data, col, row
    coo_data = [convert_to_numpy(x, idx) for idx, x in enumerate(trans_out)]
    data = np.hstack([x[0] for x in coo_data]).flatten()
    col = np.hstack([x[1] for x in coo_data]).flatten()
    row = np.hstack([x[2] for x in coo_data]).flatten()
    test_np = coo_matrix((data, (row, col)))
    # test_np.toarray() for dense output
    return test_np

test = tfidf.TfIdfTransformer()
doc_clean = [simple_preprocess(x) for x in newsgroups_train.data]
dictionary = corpora.Dictionary(doc_clean)
doc_term_matrix = [dictionary.doc2bow(doc) for doc in doc_clean]

test.fit(doc_term_matrix)
```

One of the issues we would need to tackle in the standard usage of gensim is the amount of boilerplate code required to get into a ML scenario [(see urllib2 vs requests)](https://gist.github.com/kennethreitz/973705).
