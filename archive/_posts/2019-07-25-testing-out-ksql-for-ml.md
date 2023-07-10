---
layout: post
category:
tags:
tagline:
---

<!--
./bin/zookeeper-server-start ./etc/kafka/zookeeper.properties
./bin/kafka-server-start ./etc/kafka/server.properties

./bin/ksql-server-start -daemon config/ksql-server.properties
./bin/ksql
-->

In this post we'll test out how we can deploy a machine learning model over KSQL when it has been successfully transcompiled. This leverages new features which I had a part in for adding new mathematical functions in KSQL.

## Getting Started

To get started ensure that you have Kafka and KSQL installed. You may need to install KSQL from source if it has not been updated in the monthly snapshots. For the purposes of this example, we will also leverage the inbuilt default topic generator in kafka:

```
./bin/ksql-datagen quickstart=orders topic=order_topic
```

Then when we use KSQL CLI tool, we should be able to see the topic when we use `SHOW TOPICS;`

```
ksql> SHOW TOPIC;
```

First let's create a stream so we can process things as SQL

```
CREATE STREAM orders_raw (
    itemid VARCHAR,
    orderunits DOUBLE,
    address STRUCT<
        city VARCHAR,
        state VARCHAR,
        zipcode INT>,
    ordertime VARCHAR)
 WITH (
    KAFKA_TOPIC='order_topic',
    VALUE_FORMAT='JSON');
```

We now should be able to query the stream:

```
ksql> select * from orders_raw;
+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+
|ROWTIME          |ROWKEY           |ITEMID           |ORDERUNITS       |ADDRESS          |ORDERTIME        |
+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+
|1564031036463    |3075             |Item_132         |7.287161338446933|{CITY=City_72, ST|1502749591582    |
|                 |                 |                 |                 |ATE=State_89, ZIP|                 |
```

In order to decompile to a model, imagine we have a simple logistic regression model in the form

$$y = \text{logit}(10 \times \text{ORDERUNITS} + 2)$$

```
ksql> select 1/(1+exp(-10*orderunits-2)) from orders_raw limit 2;
+---------------------+
|KSQL_COL_0           |
+---------------------+
|0.9999999999999964   |
|0.9934719923097726   |
```

And there we have it! A simple example of how we can deploy a logistic regression model on KSQL!

## Updating a Model over a Stream

Armed with this example, let's look at how we may train a model over a stream of data, and how it can then subsequently be transcompiled to a new Kafka stream.

To do this we shall use KSQL API directly.

```py
from ksql import KSQLAPI
import pandas as pd
import json
from sklearn.linear_model import SGDClassifier

def safe_load(r):
    try:
        return json.loads(r)
    except:
        # probably bad practise - but we'll get to that later...
        return None

client = KSQLAPI('http://localhost:8088', timeout=10)
query = client.query('SELECT ROWKEY, ORDERUNITS FROM ORDERS_RAW;',
        stream_properties={"ksql.streams.auto.offset.reset": "earliest"},
        idle_timeout=1)

queries = []

while True:
    try:
        queries.append(next(query))
    except:
        # probably bad practise - but we'll get to that...
        # the last output can be erroneous if it is incomplete and the
        # stream terminated early - we probably want to handle that just a little bit better...
        break


records = [safe_load(r) for r in ''.join(queries).strip().replace('\n\n\n\n', '').split('\n')]
data = [r['row']['columns'] for r in records[:-1]]
df = pd.DataFrame(data, columns=['rowkey', 'price'])
```

Next we'll trian a dummy model. The easiest way to do this is just to generate dummy labels off a stream.

```py
# prepare for ml...
df['label'] = df['rowkey'].astype(int) % 2 # this is just a dummy label

# simple model...
model = SGDClassifier(max_iter=1000, tol=None, loss='log')
model.fit(df[['price']], df['label'])
```

Finally we can just use the coefficients of the model to generate (transcompile) the model as a SQL query. There are some Python libraries which can do this for you. In this case, since we're just using a logistic regression model, this is somewhat trivial to implement.

```py
# we can now generate a query with our new model!
# we can use this to update the existing model or just straight query it...
sql_query = """select ROWKEY, 1/(1+exp((-1*{coef}*orderunits)+(-1*{intercept}))) as proba
from orders_raw limit 10;""".format(coef = model.coef_.flatten()[0],
                                    intercept=model.intercept_[0])
query = client.query(sql_query,
                     stream_properties={"ksql.streams.auto.offset.reset": "earliest"},
                     idle_timeout=10)

queries = []

while True:
    try:
        queries.append(next(query))
    except:
        # probably bad practise - but we'll get to that...
        # fix up the exceptions and expectations around using response library
        # and the KSQL pages (see also the github issues tab)
        break

out_records = [safe_load(r) for r in ''.join(queries).strip().replace('\n\n\n\n', '').split('\n')]
data = [r['row']['columns'] for r in out_records[:-1]]
print(pd.DataFrame(data, columns=['rowkey', 'probability']))
```

Or of course you can query this interactively using the KSQL CLI

```
ksql> select ROWKEY, 1/(1+exp((-1*-0.05*orderunits)+(-1*0.93))) as proba
from orders_raw limit 10;
+---------+--------------------+
|ROWKEY   |PROBA               |
+---------+--------------------+
|23208    |0.6963561533744322  |
|23209    |0.6791715821533539  |
|23210    |0.6741000363769933  |
```
