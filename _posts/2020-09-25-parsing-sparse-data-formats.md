---
layout: post
category:
tags:
tagline:
---

I've been thinking about transferring data in a sparse manner. Now in production systems you probably don't want to go around defining custom formats. In fact its probably preferrable to use `jsonlines` format instead of what is shown below.

In this post, we'll try to parse an arbitrary length of data in the format

```
(key:value)+
```

Where the separator between the `key:value` is a space, and we always presume `value` to be a float (or string if surrounded by double or single quotes), and `key` to be a string. This is a variation of the [Vowpal Wabbit format](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Input-format).

Examples of this format is:

```
x:1 "col1":100 desc:"Hello world"
```

Basically this is a "flattened" YAML format which doesn't respect `\n` or `\t` characters, instead they're treated as spaces, so long as its not enclosed in a string.

```py
import csv

output = list(csv.reader(["x:1"], delimiter=' ', quotechar='"'))[0]
# ['x:1']

output = list(csv.reader(['x:1 "col1":100'], delimiter=' ', quotechar='"'))[0]
# ['x:1', 'col1:100']

output = list(csv.reader(['x:1 "col 1":100'], delimiter=' ', quotechar='"'))[0]
# ['x:1', 'col 1:100']
```

However this naive approach won't work if the column element isn't quoted at the start of the field. An example of this is if we try the below:

```py
import csv

output = list(csv.reader(['x:1 col1:"hello world"'], delimiter=' ', quotechar='"'))[0]
# ['x:1', 'col1:"hello', 'world"']

output = list(csv.reader(['x:1 col1:"hello world" desc:"1 : 2"'], delimiter=' ', quotechar='"'))[0]
# ['x:1', 'col1:hello world', 'desc:"1', ':', '2"']
```

To remedy this, we can do another pass iterating through the tokens to yield a `dict` item

```py
import csv

def try_number(val):
    try:
        return float(val)
    except:
        return val

def vw2dict(vw):
    d_ls = list(csv.reader([vw], delimiter=' ', quotechar='"'))[0]
    d_d = {}

    quote_esc = False
    key = None
    value = None
    for el in d_ls:
        if quote_esc:
            value += el
            if el.endswith('"'):
                value = value[:-1]
                quote_esc = False
        else:
            key, value = el.split(":", 1)
            if value.startswith('"'):
                value = value[1:]
                quote_esc = True

        if not quote_esc:
            key_val = d_d.get(key)
            if key_val is not None and type(key_val) is not list:
                key_val = [key_val, try_number(value)]
            elif key_val is not None and type(key_val) is list:
                key_val.append(try_number(value))
            else:
                key_val = try_number(value)
            d_d[key] = key_val
    return d_d

output = vw2dict('x:1 col1:"hello world" desc:"1 : 2"')
# {'x': 1.0, 'col1': 'helloworld', 'desc': '1:2'}

output = vw2dict('x:1 x:3 col1:"hello world" desc:"1 : 2"')
# {'x': [1.0, 3.0], 'col1': 'helloworld', 'desc': '1:2'}
```

This essentially means we've implemented a multi-pass parser for our simple language.
