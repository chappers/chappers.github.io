---
layout: post
category:
tags:
tagline:
---

If you use the `foreach` method, you can run an arbitrary Python function and use arbitrary Python objects. This means you could attach a `scikit-learn` model at the end of a Kafka stream!

```py

"""
(mldata.csv)
31,4
10,8
32,9
103,11
34,23
1022,2
"""

from pyspark.sql.types import FloatType, TimestampType, StructType, StructField
from pyspark.sql.functions import to_timestamp
from sklearn.linear_model import SGDRegressor
from joblib import dump, load

eventSchema = StructType().add("var", "float").add("label", "float")
df = spark.readStream.schema(eventSchema).format("csv").load("mldata*.csv")
model_joblib = "model.joblib"

# to verify this works
# df.writeStream.outputMode("append").format("console").trigger(once=True).start()


def get_model():
    try:
        model = load(model_joblib)
    except:
        model = SGDRegressor()
    return model


def train_one(row):
    import numpy as np
    X = np.array([row["var"]]).reshape(1, -1)
    y = np.array([row["label"]])
    model = get_model()
    model.partial_fit(X, y)
    dump(model, model_joblib)
    pass

df.writeStream.foreach(train_one).outputMode("append").trigger(once=True).start()

m = get_model()
import numpy as np
m.predict(np.array([1]).reshape(1, -1))
```

Here is an example of how one could integrate `scikit-learn` with Spark structured streaming. It leverages the `foreach` pattern, which may not be the most effective way and could lead to race conditions. The better approach is probably to use a Python object explicitly with `open` and `close` methods.
