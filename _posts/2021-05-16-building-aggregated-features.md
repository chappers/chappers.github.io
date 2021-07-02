---
layout: post
category : 
tags : 
tagline: 
---


Writing aggregated features is not new or novel. The question is how can we make it scriptable? 

[Tecton offers some insights to how this could be done](https://github.com/tecton-ai-ext/tecton-examples/blob/main/fraud_model_end_to_end/1%20-%20tecton_features/features/transaction_aggregates.py):

```py
transaction_aggregates = TemporalAggregateFeaturePackage(
    name="transaction_aggregates",
    description="Average transaction amount over a series of time windows ",
    entities=[e.user],
    transformation=transaction_aggregate_transformers,
    aggregation_slide_period="1h",
    aggregations=[
        FeatureAggregation(column="transaction", function="sum", time_windows=["1h", "12h", "24h","72h","168h", "960h"]),
        FeatureAggregation(column="amount", function="mean", time_windows=["1h", "12h", "24h","72h","168h", "960h"])],
    materialization=MaterializationConfig(
        online_enabled=True,
        offline_enabled=True,
        feature_start_time=datetime(2020, 12, 1),
    )
)
```

But how would we do this without Tecton? PySpark `window` functions offer some insight to how we could generalise it. 

```py
from pyspark.sql import SparkSession
import featuretools as ft

spark = SparkSession.builder.appName("example").getOrCreate()

data = ft.demo.load_mock_customer()
trans_sdf = spark.createDataFrame(data["transactions"])
sessions_sdf = spark.createDataFrame(data["sessions"])
trans_sdf.createOrReplaceTempView("transactions")
sessions_sdf.createOrReplaceTempView("sessions")

customer_trans_raw = spark.sql("""
    select distinct
        s.customer_id,
        t.transaction_time,
        1 as transaction,
        t.amount as amount
    from 
        transactions as t
        inner join sessions as s
        on t.session_id = s.session_id
""")

customer_trans_raw.createOrReplaceTempView("cust_trans")

# generate feats

def feature_aggregation(table="cust_trans", entity="customer_id", event_date="transaction_time", time_window="1 day"):
    return spark.sql(f"""
    select
        {entity},
        window({event_date}, "{time_window}").start as {event_date},
        mean(amount) as mean_amount,
        sum(transaction) as sum_transaction,
        sum(amount) as sum_amount
    from {table}
    group by {entity}, window({event_date}, "{time_window}")
    """)

feats_2day = feature_aggregation("cust_trans", "customer_id", "transaction_time", "2 day")
feats_1week = feature_aggregation("cust_trans", "customer_id", "transaction_time", "1 week")
feats_2day.toPandas().head()
feats_1week.toPandas().head()
```

Output would look something like this:

```
In [ ]: feats_2day.toPandas().head()
Out[ ]:                                                                        
   customer_id    transaction_time  mean_amount  sum_transaction  sum_amount
0            2 2013-12-31 11:00:00    77.422366               93     7200.28
1            4 2013-12-31 11:00:00    80.070459              109     8727.68
2            1 2013-12-31 11:00:00    71.631905              126     9025.62
3            5 2013-12-31 11:00:00    80.375443               79     6349.66
4            3 2013-12-31 11:00:00    67.060430               93     6236.62

In [ ]: feats_1week.toPandas().head()
Out[ ]:                                                                        
   customer_id    transaction_time  mean_amount  sum_transaction  sum_amount
0            5 2013-12-26 11:00:00    80.375443               79     6349.66
1            2 2013-12-26 11:00:00    77.422366               93     7200.28
2            3 2013-12-26 11:00:00    67.060430               93     6236.62
3            4 2013-12-26 11:00:00    80.070459              109     8727.68
4            1 2013-12-26 11:00:00    71.631905              126     9025.62
```

This could be referenced by Feast which can generate the appropriate training or serving dataset as needed!