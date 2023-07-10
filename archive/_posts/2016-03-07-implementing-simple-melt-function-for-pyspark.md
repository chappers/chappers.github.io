---
layout: post
category: web micro log
tags:
---

With the introduction of the `pivot` function within Spark 1.6.0, I thought I'll give implementing a simple version of `melt` a go.
Currently it isn't as flexible as the `reshape2` library within R but it already does a pretty good job following the same approach
to which the reshape library does it in.

The essential idea behind the code is using `flatMap` functionality on DataFrame objects to emit multiple rows (observations) per each row
in the data frame and remapping the resulting values.

As a simple example, consider the following:

```py
from pyspark.sql import Row

def simple_melt(row):
    # usage: `sqlContext.createDataFrame(df.flatMap(simple_melt))`
    return [Row(A=row[0], B="1", C=row[1]),
            Row(A=row[0], B="2", C=row[2]),
            Row(A=row[0], B="3", C=row[3])]
```

This will take in each row, and melt it by the first variable for the next 3.

We can generalize this by determining what the field names are for each row object, and selecting the `id` variables which we
want to melt by. This will result in the `melt` function:

```py
def melt(row, ids, var_name='variable', value_name='value'):
    """takes in a row object and melts it keeping ids the same;

    based on this: http://sinhrks.hatenablog.com/entry/2015/04/29/085353
    """
    for id in ids:
        if id not in row.__fields__:
            raise ValueError(id + ": is not found in the list of fields")

    row_names = row.__fields__[:]
    variable_fields = set(row_names) - set(ids)
    melted_rows = []

    for var in variable_fields:
        curr_var = {}
        for id in ids:
            curr_var[id] = row[row_names.index(id)] # creates the by ids part
        curr_var[var_name] = var
        curr_var[value_name] = row[row_names.index(var)]
        melted_rows.extend(Row(curr_var))
    return melted_rows
```

Usage would be:

```py

df_melted = sqlContext.createDataFrame(df.flatMap(lambda row: melt(row, ids=ids)))

```
