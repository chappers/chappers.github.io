---
layout: post
category : web micro log
tags :
---

Trying to decipher LDA topics is hard. In this post I propose an extremely naïve way of labelling topics which was inspired by the (unsurprisingly) named paper [Automatic Labelling of Topic Models](http://www.aclweb.org/anthology/P11-1154).

The gist of the approach is that we can use web search in an information retrieval sense to improve the topic labelling of our LDA model.

Algorithm (Extracting Relevant Information):

```
Input: top keywords for an LDA topic

for each search result in Wikipedia for each keyword:
    if any other keyword appears in article result summary:
        yield article header
    end if
end for

human topic = keywords(list of article headers)
return human topic
```

Of course this can be extended in many ways. The paper suggests not only using article headers, but also article summaries and Google search results in an "ensemble" style of model.

In my naïve approach the libraries I used within Python are:

*  `gensim`: for determining the keywords using `gensim.summarization.keywords`
*  `wikiapi`: for crawling through Wikipedia

Both which can be installed via `pypi`

```py
from wikiapi import WikiApi
import gensim

def get_relevant_articles(keywords, search_depth=5, keyword_summary=5):
    """
    Searches through a list of keywords and returns keywords based on article headers
    in Wikipedia.    

    args:
    *  keywords: A list of keywords
    *  search_depth: how many wikipedia search results are checked, assumes to be between 1-10
    *  keyword_summary: gensim word argument to how many words should be used in summarization
    """
    if len(keywords) == 0:
        return []
    wiki = WikiApi()

    keywords = [x.lower() for x in keywords]
    info = []
    for keyword in keywords:
        results = wiki.find(keyword)
        other_words = [x for x in keywords if x != keyword]
        
        if search_depth is not None:
            results = results[:search_depth]

        for result in results:
            article = wiki.get_article(result)
            summary_words = article.summary.lower().split(' ')
            has_words = any(word in summary_words for word in other_words)

            if has_words:
                info.append(article.heading)

    try:
        info_keyword = gensim.summarization.keywords(' '.join(info),
                    words=keyword_summary).split('\n')
    except:
        print("keyword extraction failed, defaulting to article heading output")
        info_keyword = info[:]
    return info_keyword

def lemmatize_all(docs):
    """lemmatize a list of strings"""
    from gensim import utils
    import itertools
    def lemmatize_single(doc):
        result = utils.lemmatize(doc)
        return [x[:-3] for x in result]    
    return list(set(itertools.chain.from_iterable([lemmatize_single(x) for x in docs])))

all_results = get_relevant_articles("stock market investor fund trading investment firm exchange companies share".split())

            has_words = any(word in summary_words for word in other_words)

            if has_words:
                info.append(article.heading)

    try:
        info_keyword = gensim.summarization.keywords(' '.join(info),
                    words=keyword_summary).split('\n')
    except:
        print("keyword extraction failed, defaulting to article heading output")
        info_keyword = info[:]
    return info_keyword

def lemmatize_all(docs):
    """lemmatize a list of strings"""
    from gensim import utils
    import itertools
    def lemmatize_single(doc):
        result = utils.lemmatize(doc)
        return [x[:-3] for x in result]    
    return list(set(itertools.chain.from_iterable([lemmatize_single(x) for x in docs])))

all_results = get_relevant_articles("stock market investor fund trading investment firm exchange companies share".split())

print(all_results) # [u'investor', u'investors', u'reform', u'investment', u'exchange', u'trade', u'trading']
print(lemmatize_all(all_results)) # ['exchange', 'reform', 'trade', 'trading', 'investor', 'investment']
```
