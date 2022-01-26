---
layout: post
category:
tags:
tagline:
---

When considering how we would build airflow pipelines what considerations should we make?

From an airflow perspective, the task runners are not a spark cluster, so the immediate item is we need to do a remote execution. There are Spark templates to do this, though of course a simple bash operator would work too.

Then in order to ensure idempotency (or no multiple jobs running) - we would probably a bash sensor operator in conjunction - although a more native Spark one would work as well.

An overly simple example:

```py
from pyspark.sql import SparkSession
import pandas as pd
from datetime import datetime
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--date", type=str, default=datetime.now().strftime("%Y%m%d"))

args = parser.parse_args()

spark = SparkSession.builder.appName("etl").getOrCreate()
start_date = datetime.strptime(args.date, "%Y%m%d")


df = spark.createDataFrame(pd.DataFrame({
    'id': range(5),
    'start_date': [start_date for _ in range(5)]
}))

df.write.mode('append').parquet('etl.parquet')
```

Then the sensor operator would be as simple as

```py
import pandas as pd
from datetime import datetime
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--date", type=str, default=datetime.now().strftime("%Y%m%d"))

args = parser.parse_args()
start_date = datetime.strptime(args.date, "%Y%m%d")

df = pd.read_parquet("etl.parquet")
assert np.sum(df['start_date'].dt.date == start_date.date()) == 0
```

Putting it altogether in airflow it would look like:

```py
from textwrap import dedent
from airflow import DAG
from datetime import timedelta
from datetime import datetime

from airflow.operators.bash import BashOperator
from airflow.sensors.bash import BashSensor

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

with DAG(
    'etl_example',
    default_args=default_args,
    start_date=datetime(2020, 12, 1),
    description='A simple tutorial DAG',
    schedule_interval='@daily',
    catchup=True
) as dag:

    etl_template = dedent(
        """
    spark-submit etl.py --date {{ ds_nodash }}
        """
    )

    etl_task = BashOperator(
        task_id="etl",
        bash_command = etl_template
    )

    etl_sensor = dedent(
        """
    python sensor.py --date {{ ds_nodash }}
        """
    )

    sensor_task = BashSensor(
        task_id = 'sensor',
        bash_command = etl_sensor
    )

    sensor_task >> etl_task



```

These probably should be executed as non-bash operators - exercise for the reader; perhaps a better pattern than a `sensor` is the `ShortCircuitOperator` for example.

_edit: this is slightly wrong - you probably need to guarentee idempotence in the Spark code, and the sensor - although it works, probably doesn't operate as expected_
