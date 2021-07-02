---
layout: post
category : 
tags : 
tagline: 
---

To get dual subtitles working with chinese, pinyin and english, we can try running the following:

*  [DualSub](http://bonigarcia.github.io/dualsub/index.html)
*  [xpinyin](https://pypi.org/project/xpinyin/)

Then to convert a file in python we can run:

```py
from xpinyin import Pinyin
import sys

p = Pinyin()
text = open(sys.argv[1], 'r').read()
text_out = p.get_pinyin(text, ' ', tone_marks='marks')
fout = sys.argv[1].replace("zh-Hans", "zh-pinyin")
with open(fout, 'w') as f:
    f.write(text_out)
```