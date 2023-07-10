---
layout: post
category: web micro log
tags: [sas, python]
---

After playing around with [pycco](http://fitzgen.github.io/pycco/) I decided to give a go of creating a custom one for SAS.

When applying `pycco` onto SAS, you will run into strange formatting since when you have:

```js
some sas code /*a comment*/
```

you would expect the comment to reference that line of code. `pycco` handles this correctly.

In comparison since SAS doesn't have a concept of "functions" (yes there are macros, but lets ignore that for a moment).

Often when the following happens:

```js
proc statement

/*
Multi line comment
*/

data step
```

The multi line comment will be "attached" to the proc statement, whilst in my experience the SAS programmer generally would prefer the comment to be attached to the data step. So I decided it would handle that.

So here is my 100 or so line solution:

```python
# quick and very dirty literate programming style
# written in python for SAS

import re
from markdown import markdown
from pygments import highlight
from pygments.lexers import SqlLexer
from pygments.formatters import HtmlFormatter

# Functions? SAS Users have never heard of that, who needs it

# ignore single line comments, because markdown probably wouldn't be applied
# What kind of SAS user uses single comments for **ACTUALLY** commenting?
multiline = ["/*", "*/"]

comments = re.compile(r'((?:\n\s*)?/\*(?:.|[\r\n])*?)\*/', re.DOTALL)
# regex to get all comments cause why not.
# `/\*(.|[\r\n])*?\*/` from http://ostermiller.org/findcomment.html

FILENAME = ''

code = open(FILENAME, 'r').read()

parse_code = [x for x in comments.split(code) if x.strip() != '']

# combine multiple comments together
new_list = []
for i, el in enumerate(parse_code):
    if i == 0:
        new_list.append(el)
    elif parse_code[i-1].strip()[:2] == "/*" and el.strip()[:2] == "/*":
        new_list[-1] = "%s\n\n%s" % (new_list[-1].replace("*/", "").rstrip(), el.replace("/*", "", 1).lstrip())
    else:
        new_list.append(el)

# reorder inline_comments to be in front of code (but only for that line!)
inline_comments = [x for x in new_list if x.strip()[:2] == r"/*" and x[0] != '\n']

for comment in inline_comments:
    index = new_list.index(comment)
    code = new_list.pop(index-1)
    code = code.rsplit('\n', 1)
    if len(code) == 1:
        new_list.insert(index, code[0]) # place this after the comment
    elif len(code) == 2:
        new_list.insert(index-1, code[0]) # place this before the comment
        new_list.insert(index+1, code[1]) # place this after the comment (remember the index has shifted by 1 now)

# combine multiple code blocks together
final_list = []
for i, el in enumerate(new_list):
    if i == 0:
        if el.strip()[:2] != "/*": #ensure the first thing is a "comment"
            final_list.append("/*<p></p>*/")
        final_list.append(el)
    elif new_list[i-1].strip()[:2] != "/*" and el.strip()[:2] != "/*":
        final_list[-1] = "%s\n%s" % (final_list[-1], el)
    else:
        final_list.append(el)
if final_list[-1].strip()[:2] == "/*": # ensure that there is an even number of blocks
    final_list.append(r'')

# zip it up. So then the 0 element will be the comment on the left and 1 element will be the code
to_html = zipped = zip(final_list[0::2], final_list[1::2])


grid_items = '\n\n'.join(["""<div class="grid">
    <div class="col-1-2">
       <div class="content">
           %s
       </div>
    </div>
    <div class="col-6-12" >
       <div class="content">
           %s
       </div>
    </div>
</div>""" % (markdown(x.strip()[2:-2]), highlight(y, SqlLexer(stripall=True), HtmlFormatter(noclasses=True))) for x,y in to_html])
# `noclasses=True` cause what are style sheets?


html_body = r"""
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
<style>
*,:after,:before{-webkit-box-sizing:border-box;-moz-box-sizing:border-box;box-sizing:border-box}body{margin:0}[class*=col-]{float:left}.grid{width:100%;max-width:1140px;min-width:755px;margin:0;overflow:hidden}.grid:after{content:"";display:table;clear:both}.push-right{float:right}.col-1-2,.col-6-12{width:50%}.col-1-2{padding-left:20px}@media handheld,only screen and (max-width:767px){.grid{width:100%;min-width:0;margin-left:0;margin-right:0;padding-left:0;padding-right:0}[class*=col-]{width:auto;float:none;margin-left:0;margin-right:0;margin-top:10px;margin-bottom:10px;padding-left:20px;padding-right:20px}}
</style>
</head>
<body>
  <div class="grid"> """ + grid_items + """</div>
</body>
</html>
"""

f = open(FILENAME[:-4]+'.html', 'w')
f.write(html_body)
f.close()
```
