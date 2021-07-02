---
layout: post
category : web micro log
tags :
---

<font color="purple">Programming</font> and <font color="purple">coding</font> <font color="purple">is</font> usually <font color="purple">done</font> with some <font color="blue">kind</font> of <font color="blue">syntax</font> <font color="purple">highlighting</font>, to <font color="purple">make</font> <font color="green">it</font> easier to <font color="purple">read</font> and <font color="blue">reason</font> with a <font color="blue">program</font>. <font color="green">It</font> <font color="purple">helps</font> <font color="blue">determine</font> where <font color="green">we</font> might <font color="purple">have</font> a <font color="blue">number</font> or <font color="purple">string</font> in <font color="green">our</font> <font color="blue">SQL</font> <font color="blue">query</font> , or <font color="blue">determine</font> where <font color="purple">is</font> the <font color="blue">start</font> and <font color="blue">end</font> of a <font color="blue">function</font> <font color="blue">code</font> <font color="blue">block</font>.

Then why <font color="purple">does</font>n't one of these <font color="blue">exist</font> for <font color="blue">say</font> <font color="blue">essay</font> <font color="purple">writing</font>? <font color="blue">Is</font> <font color="green">it</font> actually difficult to <font color="purple">build</font> a <font color="blue">syntax</font> <font color="blue">highlighter</font> for the <font color="blue">English</font> <font color="blue">language</font>? <font color="green">It</font> <font color="purple">turns</font> out <font color="green">it</font> <font color="purple">is</font> extremely simple to <font color="purple">build</font> one, but to <font color="purple">build</font> a good one <font color="purple">is</font> <font color="blue">something</font> entirely different.

Here <font color="green">I</font> <font color="purple">turn</font> a simple <font color="blue">version</font> of a <font color="blue">syntax</font> <font color="blue">highlighter</font> <font color="purple">written</font> in <font color="blue">Python</font> (and this <font color="blue">post</font>) to <font color="purple">see</font> whether <font color="green">it</font> <font color="purple">is</font> actually useful.


```py
import nltk
from nltk.tokenize import word_tokenize

tag_colour = {}

# verbs
for vb in ['VB', 'VBD', 'VBG', 'VBN', 'VBP', 'VBZ']:
    tag_colour[vb] = 'purple'
    
# pronouns
for pn in ['PRP', 'PRP$']:
    tag_colour[pn] = 'green'

# nouns
for nn in ['NN', 'NNP', 'NNPS', 'NNS']:
    tag_colour[nn] = 'blue'
    
# enter your text
print ' '.join([tag_single_word(x) for x in nltk.pos_tag(nltk.word_tokenize(text))])
```


There <font color="purple">were</font> various <font color="blue">shortcommings</font> with this <font color="blue">code</font>; for one <font color="green">it</font> <font color="purple">does</font>n't <font color="purple">understand</font> <font color="blue">punctuation</font> and new <font color="blue">lines</font> so that <font color="blue">everything</font> would <font color="purple">be</font> incorrectly <font color="purple">spaced</font> <font color="blue">apart</font>! <font color="blue">Nevertheless</font> <font color="green">it</font> <font color="purple">was</font> an interesting <font color="blue">experiment</font> to also <font color="purple">see</font> the <font color="blue">power</font> within <font color="blue">Python</font> and <font color="blue">NLTK</font> to <font color="purple">achieve</font> <font color="blue">something</font> in a <font color="blue">couple</font> <font color="blue">lines</font> of <font color="blue">code</font>.