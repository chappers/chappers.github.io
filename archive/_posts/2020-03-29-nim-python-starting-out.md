---
layout: post
category:
tags:
tagline:
---

In an effort to learn something new and speed up some existing experiments, I've started dipping my toes into the Nim programming language.

In theory it should offer a substatial speed up to parts of my Python code but we'll see what actually happens. Here are some patterns which I'm sure I'll need to refer multiple times in the future.

**Sorting a Table**

To sort a table by value, you need to create something to help with the sorting.

```
import tables

var a = initOrderedTable[int, array[4, float]]()
for idx, el in [1,2,3]:
  var newval = if idx==1: 10.0 else: -1.0
  a[el] = [float(idx), float(-idx), float(newval), float(idx)]

echo a

echo "\tSort by idx 1"
a.sort(proc(x, y: (int, array[4, float])): int =
  result = cmp(x[1][1], y[1][1]))
echo a

echo "\tSort by idx 2"
a.sort(proc(x, y: (int, array[4, float])): int =
  result = cmp(x[1][2], y[1][2]))
echo a
```

Output:

```
{1: [0.0, 0.0, -1.0, 0.0], 2: [1.0, -1.0, 10.0, 1.0], 3: [2.0, -2.0, -1.0, 2.0]}
        Sort by idx 1
{3: [2.0, -2.0, -1.0, 2.0], 2: [1.0, -1.0, 10.0, 1.0], 1: [0.0, 0.0, -1.0, 0.0]}
        Sort by idx 2
{3: [2.0, -2.0, -1.0, 2.0], 1: [0.0, 0.0, -1.0, 0.0], 2: [1.0, -1.0, 10.0, 1.0]}
```
