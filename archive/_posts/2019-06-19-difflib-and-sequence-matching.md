---
layout: post
category:
tags:
tagline:
---

How do we detect plagiarism?

There's probably many state of the art ways on how one could approach this problem; in this post we'll explore how we can use standard Python library to do some simple `diff`s and comparison across large corpuses to fine out when things overlap or don't overlap.

To use the `difflib` library to match arbritary strings is quite easy:

```py
import difflib

text1 = 'text1 says hello world because there is only one!'
text2 = 'text2 says hello worlds because there are multiple'

seq = difflib.SequenceMatcher(None, text1, text2)
match_blocks = [x for x in seq.get_matching_blocks() if x.size > 2]
```

When using `get_matching_blocks()` method make sure that `size` is used as a filter otherwise, you'll get some "no matches" within the results which is pretty pointless. From here one can retrieve the substrings which matched!

```py
for m in match_blocks:
    print(m)
    print("{}:{}:{}".format(text1[:m.a], text1[m.a:m.a+m.size], text1[m.a+m.size:]))
    print("{}:{}:{}".format(text2[:m.b], text2[m.b:m.b+m.size], text2[m.b+m.size:]))
    print("--------------------\n")
```

This leads to the output:

```
Match(a=0, b=0, size=4)
:text:1 says hello world because there is only one!
:text:2 says hello worlds because there are multiple
--------------------

Match(a=5, b=5, size=17)
text1: says hello world: because there is only one!
text2: says hello world:s because there are multiple
--------------------

Match(a=22, b=23, size=15)
text1 says hello world: because there :is only one!
text2 says hello worlds: because there :are multiple
--------------------
```

From here we can begin to extract similar phrases and text simply using the standard library in Python.
