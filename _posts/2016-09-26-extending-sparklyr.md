---
layout: post
category : web micro log
tags :
---

Okay, so you've looked at `sparkhello`; now what? How can I extend and make use of some scala code!

Simple example:

```scala
def addOne(df:DataFrame) : DataFrame = {
  val colname: String = "test"
  df.withColumn("test1", df("test") + 1)
}
```

How do we add this in R?

```r
#' @import sparklyr
#' @export
spark_addOne <- function(df) {
  sdf_register(
    sparklyr::invoke_static(spark_connection(df), "SparkHello.HelloWorld", "addOne",
                            spark_dataframe(df))
  )
}
```

Important to enforce the `spark_dataframe` object, what is interesting is that we don't need to specify "sc" as it is given based on `spark_connection`

Naturally we can add parameters to this:

```scala
def addOneCols(df:DataFrame, inputcolname: String,  outputcolname:String) : DataFrame = {
  df.withColumn(outputcolname, df(inputcolname) + 1)
}
```

```r

#' @import sparklyr
#' @export
spark_addOneCols <- function(df, input_col, output_col) {
  sdf_register(
    sparklyr::invoke_static(spark_connection(df), "SparkHello.HelloWorld", "addOneCols",
                            spark_dataframe(df),
                            ensure_scalar_character(input_col),
                            ensure_scalar_character(output_col))
  )
}

```

Which begs the question; how can we automatically generate R packages if we are writing scala applications?
