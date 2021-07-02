---
layout: post
category : web micro log
tags : 
---

# Getting started with Spark

The easiest way to play with Spark is to check out the [Developer Resources over at databricks](https://databricks.com/spark/developer-resources), it was the only Windows solution that worked "out-of-the-box" without installing a VM.

# A quick look at Spark

Many of the normal functions in Spark are similar to what you expect. We have our typical `map` and `reduce` functions, but Spark introduces an `aggregate` function. How does it work? Consider the example below, which calculates the sum of all the value, and keeps track of how many values have been processed:

```scala
scala> val input = sc.parallelize(List(1,2,3,4))
input: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[3] at parallelize at <console>:12

scala> input.aggregate((0,0)) (
     |   (acc, value) => (acc._1 + value, acc._2 + 1),
     |   (acc1, acc2) => (acc1._1 + acc2._1, acc1._2 + acc2._2))
res0: (Int, Int) = (10,4)
```

It has three arguments:

*  `zeroValue` : the initial value of the accumulator  
*  `seqOp` : this combines the accumulator from the rdd (resilient distributed dataset), the first element is the accumulator and the second is an element of the rdd  
*  `combOp` : this merges two accumulators, the first and second argument are accumulators. 

This function differs from reduce, because you can input and output different values. As you can see from the example above, the input is single values, whilst the output is a tuple. 


