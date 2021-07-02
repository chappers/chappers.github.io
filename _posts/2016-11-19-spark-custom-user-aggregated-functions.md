---
layout: post
category : web micro log
tags :
---

Not much to really say here. This was a question which was asked from OMSCS about creating your own user defined aggregation function, so here it is.

It is clear there are influences around the notion of a monoid.

`initialize` is essentially your identity.

`merge` is where you put the two objects together.

and so on...

Unfortunately this function is (poorly?) documented, though perusing Google suggests that this will be fixed in Spark 2.1.X

```scala

import org.apache.spark.sql.types._
import org.apache.spark.sql.{Row, Column}
import org.apache.spark.sql.expressions._

val data = Array((92,"",2.0,266.0),
(11,"",12.0,266.0),
(93,"1",1000.0,288.0),
(92,"1",1750.0,288.0),
(96,"A",250.0,209.0),
(38,"C",500.0,209.0),
(35,"E",250.0,209.0),
(92,"D",25.0,472.0),
(93,"L",12.0,455.0),
(83,"4",1200.0,129.0))

var df = spark.createDataFrame(data).toDF("ID", "name", "idsum", "indx")

import org.apache.spark.sql.types._
import org.apache.spark.sql.{Row, Column}
import org.apache.spark.sql.expressions._

object GroupedVectorizer extends UserDefinedAggregateFunction {
  def inputSchema = new StructType().add("col", DoubleType) // you're putting in a double type...
  def bufferSchema = new StructType().add("col", ArrayType(DoubleType)) // maybe you want some custom type?
  def dataType = ArrayType(DoubleType)
  def deterministic = false
  def initialize(buffer: MutableAggregationBuffer) = {
    buffer(0) = Seq[Double]()
  }
  def update(buffer: MutableAggregationBuffer, input: Row) = {
    val dbl = input.getAs[Double](0)
    var arrayDbl = buffer.getAs[Seq[Double]](0)
    arrayDbl = arrayDbl ++ Seq(dbl)
    buffer(0) = arrayDbl
  }
  def merge(buffer1: MutableAggregationBuffer, buffer2: Row) = {
    val arrayDbl1 = buffer1.getAs[Seq[Double]](0)
    val arrayDbl2 = buffer2.getAs[Seq[Double]](0)
    val combined = arrayDbl1 ++ arrayDbl2
    buffer1(0) = combined
  }
  def evaluate(buffer: Row) = {
    buffer.getAs[Seq[DoubleType]](0)
  }
}

val df_indx = df.groupBy("ID").agg(GroupedVectorizer($"indx").alias("indx"), GroupedVectorizer($"idsum").alias("idsum"))
df_indx.show()

```
