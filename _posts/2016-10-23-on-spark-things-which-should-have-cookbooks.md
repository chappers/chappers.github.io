---
layout: post
category: web micro log
tags:
---

There are some things which to be are completely inexplicable in the world of Spark. One of them is finding help for things which really should be recipes.

What makes it worse is that sometimes when you find the question asked on Stackoverflow; it is marked as a duplicate, despite the question not being answered in the other thread. For me it is this one: [How to access the values of denseVector in PySpark](http://stackoverflow.com/questions/37148947/how-to-access-the-values-of-densevector-in-pyspark)

To actually get the answer to my question I needed to reference a Scala post on creating [my own UDF](http://stackoverflow.com/questions/32570799/how-to-split-the-predicted-probabilities-produced-by-ml-pileline-logistic-regres), and then decode a mysterious numpy to Spark error which I wouldn't have understood without looking at [Stackoverflow](http://stackoverflow.com/questions/37165152/error-in-converting-a-dataframe-to-another-dataframe?noredirect=1&lq=1)...which again wasn't even part of the original question!

Regardless to answer the question I was after:

> How to access the values of denseVector in PySpark

If the values of the vector are in a Dataframe, and we **do not** want to convert it to an RDD and then back to a Dataframe, the correct way would be to define a UDF in Python:

```py
vector_udf = udf(lambda vector: float(vector[1]), DoubleType())
```

Where the full code would look like this:

```py
from pyspark import SparkContext
from pyspark.sql import Row
from pyspark.sql.types import DoubleType
from pyspark.sql.functions import udf
from pyspark.ml.linalg import Vectors

FeatureRow = Row('id', 'features')
data = sc.parallelize([(0, Vectors.dense([9.7, 1.0, -3.2])),
                       (1, Vectors.dense([2.25, -11.1, 123.2])),
                       (2, Vectors.dense([-7.2, 1.0, -3.2]))])
df = data.map(lambda r: FeatureRow(*r)).toDF()

vector_udf = udf(lambda vector: float(vector[1]), DoubleType())

df.withColumn('Prob_1', vector_udf(df.features)).first()
```

I hate PySpark. I think I will stick to Scala.
