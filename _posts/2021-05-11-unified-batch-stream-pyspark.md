---
layout: post
category : 
tags : 
tagline: 
---

```py
from pyspark.sql import SparkSession
from pyspark.sql.functions import window, first


spark = SparkSession \
    .builder \
    .appName("StructuredNetworkWordCount") \
    .getOrCreate()

df = spark.createDataFrame([
    ("2016-03-11 01:00:07", None),
    ("2016-03-11 04:00:07", 1),
]).toDF("date", "val")
w = df.groupBy(window("date", "1 day")).agg(first("val").alias("sum"))
w.select(w.window.start.cast("string").alias("start"), "sum").collect()

```

Let's try similar setup but writing SQL.

```py
df.createOrReplaceTempView("tbl")
w1 = spark.sql("""
    select 
        window(date, "1 day").start as event_timestamp,
        first(val, True) as val
    from tbl
    group by window(date, "1 day")
""")
```

With this in mind, how would we best create features from a unified streaming and batch pattern?

If we assume that the data would already have restrictions such as maximum number of instances in a time window; setting `first(<col>, True)` would also allow for taking the first non-null in the event that there are null values present.  

When this is flattened in this way - we can ingest it directly into a feature store like Feast in both a stream and batch. To observe this in action, suppose there is a `csv` file in the following format:

```py

"""
(test.csv)
20210102 11:00:00,3
20210101 11:00:00,10
20210202 11:00:00,31
20210201 11:00:00,10
20210211 11:00:00,32
20210212 11:00:00,103
20210214 11:00:00,34
20210221 11:00:00,1022
"""

from pyspark.sql.types import FloatType, TimestampType, StructType, StructField
from pyspark.sql.functions import to_timestamp

eventSchema = StructType().add("string_timestamp", "string").add("val", "float")
df = spark.readStream.schema(eventSchema).format("csv").load("test*.csv")

df_event = (
    df
    .withColumn("event_timestamp", to_timestamp(df.string_timestamp, "yyyyMMdd HH:mm:ss"))
    .drop("string_timestamp")
)

df_event.writeStream.format("console").trigger(once=True).start()
```

Putting it altogether with aggregation:

```py
from pyspark.sql.types import FloatType, TimestampType, StructType, StructField
from pyspark.sql.functions import to_timestamp

eventSchema = StructType().add("string_timestamp", "string").add("val", "float")
df = spark.readStream.schema(eventSchema).format("csv").load("test*.csv")

df_event = (
    df
    .withColumn("event_timestamp", to_timestamp(df.string_timestamp, "yyyyMMdd HH:mm:ss"))
    .drop("string_timestamp")
    .withWatermark("event_timestamp", "1 day")  # to allow for append mode
)


df_event.createOrReplaceTempView("df_event")
w1 = spark.sql("""
    select 
        window(event_timestamp, "1 day").start as event_timestamp,
        first(val, True) as val
    from df_event
    group by window(event_timestamp, "1 day")
""")

w1.writeStream.outputMode("append").format("console").trigger(once=True).start()
```

If there is no output - you probably need larger size data in order to trigger Spark to push something out. This is explained on [stackoverflow](https://stackoverflow.com/questions/44403690/empty-output-for-watermarked-aggregation-query-in-append-mode).

With this pattern in mind, we can have a unified code base for both batch and streaming within Spark SQL - pretty neat!
