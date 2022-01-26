---
layout: post
category: web micro log
tags:
---

String fuzzy matching to me has always been a rather curious part of text mining. There are the canonical and intuitive
Hamming and LevenShtein distance, which consider the difference between two sequences of characters, but there are also
less commonly heard of approaches, the n-gram approach.

Within text mining, n-grams can be addressed at a word or character level, in this post we shall only consider the
character representation for purposes of approximate matching.

# What is n-grams?

n-grams is the consideration of breaking up a character string into multiple character strings each with length `n`.
In an example where `n` is 3, we have a trigram. For example, applying n-grams on the text `abcde` would yield,
the following components:

- `abc`
- `bcd`
- `cde`

An implementation of this within python could look like this:

```py
def ngram(text, n=3, pad=True):
    text = text.strip()
    if pad:
        text = " %s " % text
    return set([text[i:i+n] for i in range(len(text)-n+1)])

def create_ngram(text1, text2, n=3, pad=True):
    return ngram(text1, n=n, pad=pad), ngram(text2, n=n, pad=pad)
```

# Calculation of Fuzzy Matchin

There are many ways to match using n-grams, and here I will introduce some of them. The basic idea is if we have two strings,
we will convert both of them into a their components and apply a function to combine them to arrive at some measure of similarity.

## qgrams

`qgrams` simply counts the number of components which match. This can be easily calculated as follows:

```py
def qgrams(text1, text2, q=3, pad=True):
    text1, text2 = create_ngram(text1, text2, n=q, pad=pad)
    return len(text1.intersection(text2))

qgrams("abcde", "abdcde", q=2, pad=False) # 3
```

## Cosine distance

Cosine distance is also known as cosine similarity. The idea is we convert the components to vectors which we can then
use to calculate the "angle" between the two vectors.

```py
def cos_dist(text1, text2, q=3, pad=True):
    from scipy import spatial
    text1, text2 = create_ngram(text1, text2, n=q, pad=pad)

    full_text = list(text1.union(text2))
    v1 = [(lambda x: 1 if x in text1 else 0)(x) for x in full_text]
    v2 = [(lambda x: 1 if x in text2 else 0)(x) for x in full_text]
    return spatial.distance.cosine(v1, v2)

cos_dist("abcde", "abdcde", q=2, pad=False) # 0.32918
```

## Jaccard Distance

Jaccard Distance can be thought of as the proportion of components which are not in agreement.

```py
def jaccard_dist(text1, text2, q=3, pad=True):
    text1, text2 = create_ngram(text1, text2, n=q, pad=pad)

    full_text = list(text1.union(text2))
    agree_tot = len(text1.intersection(text2))
    return 1 - agree_tot/float(len(full_text))

jaccard_dist("abcde", "abdcde", q=2, pad=False) #0.5
```

## Tversky Index

I like to think of Tversky index as a weighted version of the Jaccard Distance (though strictly speaking this is not quite true).
This measure is useful when the strings in question vary greatly in length, for example searching a partial name against a full name

As a practical example, consider "Sarah Smith" vs "Sarah Jessica Smith". We do not want to "normalise" against the union of all components,
as "Sarah Smith" is wholly contained in "Sarah Jessica Smith".

```py
def tversky_index(text1, text2, a=None, b=None, q=3, pad=True):
    text1, text2 = create_ngram(text1, text2, n=q, pad=pad)
    agree_tot = len(text1.intersection(text2))
    v1 = len(text1) - agree_tot
    v2 = len(text2) - agree_tot

    if a != None and b != None:
        a = a/float(a+b)
        b = b/float(a+b)
    elif a <= 1.0 and a >= 0.0:
        b = 1-a
    elif b <= 1.0 and b >= 0.0:
        a = 1-b
    else:
        a = 0.5
        b = 0.5
    return float(agree_tot)/(agree_tot+a*v1+b*v2)

tversky_index("abcde", "abdcde", a=0.5, q=2, pad=False) # 0.6666666
```

Of course we can come up with all kinds of interesting measures that would suit our purposes, each with their pros and cons. Hopefully this is
an interesting start on using n-grams, since the resources on it are relatively sparse.
