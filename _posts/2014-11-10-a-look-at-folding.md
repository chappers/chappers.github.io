---
layout: post
category: web micro log
tags: [haskell, j]
---

Currently I'm sitting the [FP101x from edX](https://www.edx.org/course/delftx/delftx-fp101x-introduction-functional-2126). A portion of the homework was on `foldr` and `foldl` (i.e. right fold and left fold). This got me thinking how I might think about these ideas.

Lets firstly compare the two.

```hs
Prelude > foldr (-) 0 [1..4]
-2
Prelude > foldl (-) 0 [1..4]
-10
```

Why are they different? How do they work?

`foldl` is probably the intuitive one. It simply "inserts" the function between the elements.

```hs
foldl (-) 0 [1..4] -- 0-1-2-3-4 or with parenthesis ((((0-1)-2)-3)-4)
```

Whilst `foldr` starts from the _right_ side.

```hs
foldr (-) 0 [1..4] -- 1-(2-(3-(4-0)))
```

# `J`

Now what about in another language, say `J`

**left fold**

```
   -~/|.0,(1+i.4)
_10
```

**right fold**

```
   -/(1+i.4),0
_2
```

## Unpacking `J`

Lets firstly consider right fold in `J` since that one is the easier to understand. Reading from right to left we have

```
    (1+i.4),0
1 2 3 4 0
```

This creates the array `[1..4]` append `0` to the end. Next we insert (`/`) the negative sign (`-`) between each element of the array, which is actually the right fold as above. i.e.

```
   -/1 2 3 4 0
_2
   1-(2-(3-(4-0)))
_2
```

So then what is the left fold doing? We have seen this bit from above `0,(1_i.4)`, and then we have `|.` which is the _reverse_ verb.

```
   |.0,(1+i.4)
4 3 2 1 0
```

Next we have `~` which to my (extremely poor) understanding in this context is the _reflection_ adverb (this is already doing my head in, someone **please** correct me if I'm totally wrong). For example:

```
   -/1 2 3
2
```

This is interpreted to be as follows:

```
-/1 2 3

1-(2-3)
```

On the other hand:

```
   -~/1 2 3
0
```

Which would be

```
-~/1 2 3

((3-2)-1)
```

Putting this all together yields:

```
   -~/|.0,(1+i.4)
_10
   ((((0-1)-2)-3)-4)
_10
```

It is always interesting to examine how different programming languages express the same ideas.
