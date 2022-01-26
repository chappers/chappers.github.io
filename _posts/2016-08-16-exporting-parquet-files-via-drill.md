---
layout: post
category: web micro log
tags:
---

A problem I've been struggling with is this:

> How can I export/import a parquet file without Spark?

There are various efforts in the form of Apache Arrow, and a Python Parquet package which allows you to read, but how might one export the data?

## Enter Apache Drill

This isn't exactly a new approach, but rather I thought I'll document what I did to get it working _locally_ as there aren't any really clear instructions online!

Firstly start [Apache drill as shown in the tutorial](https://drill.apache.org/docs/starting-drill-on-windows/).

Next the connection settings to the Apache drill JDBC interface is show below. Make sure that the `jdbc` url is what is shown below; **not** `jdbc:drill:zk=local` as it will not work.

```r
library(RJDBC)

drv <-  JDBC("org.apache.drill.jdbc.Driver",
              "path/to/apache-drill-1.7.0/jars/jdbc-driver/drill-jdbc-all-1.7.0.jar", "'")

conn <- dbConnect(drv, "jdbc:drill:drillbit=localhost")
```

My `dfs` configuration is shown below. The key highlights are:

- connection field should be `file:///`
- make use of your workspaces and the location to navigate effectively.

This can be accessed under `http://localhost:8047/storage` after drill is running (assuming defaults are all used!)

```json
{
  "type": "file",
  "enabled": true,
  "connection": "file:///",
  "config": null,
  "workspaces": {
    "root": {
      "location": "C:/Users/chapm/Documents/apache-drill-1.7.0",
      "writable": false,
      "defaultInputFormat": null
    },
    "tmp": {
      "location": "C:/tmp",
      "writable": true,
      "defaultInputFormat": null
    },
    "output": {
      "location": "C:/Users/chapm/Documents/apache-drill-1.7.0/output",
      "writable": true,
      "defaultInputFormat": null
    }
  },
  "formats": {
    "psv": {
      "type": "text",
      "extensions": ["tbl"],
      "delimiter": "|"
    },
    "csv": {
      "type": "text",
      "extensions": ["csv"],
      "delimiter": ","
    },
    "tsv": {
      "type": "text",
      "extensions": ["tsv"],
      "delimiter": "\t"
    },
    "parquet": {
      "type": "parquet"
    },
    "json": {
      "type": "json",
      "extensions": ["json"]
    },
    "avro": {
      "type": "avro"
    },
    "sequencefile": {
      "type": "sequencefile",
      "extensions": ["seq"]
    },
    "csvh": {
      "type": "text",
      "extensions": ["csvh"],
      "extractHeader": true,
      "delimiter": ","
    }
  }
}
```

## Writing to Parquet

To write to a parquet file, drill will need to take in another file and output the new table:

```python
dbGetQuery(conn, paste0("create table dfs.output.iris as ",
    "select * from dfs.root.`iris.csv`", collapse=" "))
```

_n.b. if the command above doesn't work and you are following this, make sure that the references to "output" in your configuration above actually exists, i.e. you may need to create a folder._

_Also notice that by default, the csv parser does not skip the header row in drill! You may need to alter the configuration in order to get the behaviour you are probably expecting!_

Notice that I had to create a `csv` file first! Drill can not even be used to insert, update, or delete data, but only able to `create table as`.

This naturally leads to how can I potentially import an R object? Approaches could be:

- Write to an intermediary file format, like `csv`
- Write to a sqlite database - this should be better as it will better preserve data types of the R data frame object.

### Setting up SQLite

To set up SQLite, we will need to [download the appropriate jar](https://bitbucket.org/xerial/sqlite-jdbc/downloads), and make sure you place it in `jars/3rdparty` folder of the drill installation.

Then we can connect in the normal way:

```r
sqlitedrv <- JDBC("org.sqlite.JDBC",
                  "path/to/apache-drill-1.7.0/jars/3rdparty/sqlite-jdbc-3.8.11.2.jar", "'")

sqliteconn <- dbConnect(sqlitedrv, "jdbc:sqlite:C:/Users/chapm/Documents/apache-drill-1.7.0/sqlite/test.sqlite")

```

We can write to this table using the normal `RJDBC` commands:

```r
dbWriteTable(sqliteconn, "iris", iris)
```

#### Adding SQLite config to drill

Adding the configuration is quite simple (I called this configuration `sqlite`):

```json
{
  "type": "jdbc",
  "driver": "org.sqlite.JDBC",
  "url": "jdbc:sqlite:path/to/apache-drill-1.7.0/sqlite/test.sqlite",
  "username": null,
  "password": null,
  "enabled": true
}
```

We can then verify that it is added correctly by querying the sqlite database:

```
dbGetQuery(conn, "select * from sqlite.iris")
```

If this works correctly, we can then write to parquet this way:

```
dbGetQuery(conn, paste0("create table dfs.output.iris2 as ",
    "select * from sqlite.iris", collapse=" "))
```

And that is it! You should be able to export (and import) parquet files using R and drill.

# Python

Given that everything is set up as above, python is much easier:

```sh
pip install pydrill
```

Then just use the library:

```py
from pydrill.client import PyDrill
import pandas as pd

drill = PyDrill(host='localhost', port=8047)
df = drill.query("select * from sqlite.iris").to_dataframe()

import sqlite3
import numpy as np
sqliteconn = sqlite3.connect('path/to/apache-drill-1.7.0/sqlite/test.sqlite')

# some dummy data
d = {'one' : pd.Series([1., 2., 3.], index=['a', 'b', 'c']),
     'two' : pd.Series([1., 2., 3., 4.], index=['a', 'b', 'c', 'd'])}
df1 = pd.DataFrame(d)
df1.to_sql('dummy', sqliteconn)
# drill can see the table
drill.query("select * from sqlite.dummy").to_dataframe()

# write to parquet:
drill.query("""create table dfs.output.dummy as
select * from sqlite.dummy""")
```

This could be a simple approach to convert data to Parquet format!
