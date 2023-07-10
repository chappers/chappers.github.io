---
layout: post
category:
tags:
tagline:
---

I couldn't think of where to put this code snipper so here it is. Basically its just a coding experiment to find and delete duplicate items in a list, and also report which lines where deleted.

```py
input_text = """My
multi line input
text
text"""

input_list = input_text.split("\n")
dedup_text = "\n".join(list(dict.fromkeys([x.strip() for x in input_list])))
with open('output.txt', 'w') as f:
    f.write(dedup_text)
```

However what if you want to keep "last" instead of keep "first"? Then simply we can reverse the list and see what happens!

```py
input_text = """My
multi line input
text
text"""

input_list = input_text.split("\n")

# input_list = [x.rstrip() for x in open("text.txt", 'r').readlines()]
dedup_text = "\n".join(list(dict.fromkeys(input_list[::-1]))[::-1])
with open('output.txt', 'w') as f:
    f.write(dedup_text)
```
