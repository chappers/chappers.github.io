---
layout: post
category : web micro log
tags : 
---

Trying to figure out how to install Spark on windows is a bit of a pain so here are some basic instructions:

1.  Download any of the prebuilt [hadoop distribution for spark](http://spark.apache.org/downloads.html). For me downloading it for Hadoop version 2.6 worked perfectly. 
2.  Download the winutils for Hadoop. Google should bring up several results. At the time of writing [this one worked fine](https://github.com/srccodes/hadoop-common-2.2.0-bin).  

Now we have to make sure all our environmental variables arre set up correctly. We need to have the following things installed with the appropriate variables set:

*  `HADOOP_HOME` : for winutils  
*  `PYTHON_PATH` : for Python
*  `SPARK_HOME`  : for Spark
*  `R_HOME` : for R

Once all of these are added to our path, then we can simply run PySpark/SparkR in the shell in the normal manner. 

For example, your environmental variables may look something like this:

```bash
set HADOOP_HOME=C:\Users\chapm\HADOOP
set PYTHON_PATH="C:\Users\chapm\Python"
set SPARK_HOME=C:\Users\chapm\Spark
set R_HOME="C:\Users\chapm\R-3.X.X\bin"

set PATH=%PATH%;%JAVA_HOME%\BIN;%PYTHON_PATH%;%HADOOP_HOME%;%SPARK_HOME%\bin;%R_HOME%;%PYTHON_PATH%\Scripts
```

If you run `pyspark` or `sparkr` in the command line the `pyspark` shell should now work. 

# Getting RStudio working with Spark

To get RStudio working with Spark, we only need to link the library within Spark with the R Script. 

If we simply run the following code, SparkR should work with RStudio:

```r
Sys.setenv(SPARK_HOME="C:/Users/chapm/Spark")
.libPaths(c(file.path(Sys.getenv("SPARK_HOME"),"R","lib"), .libPaths()))

# load sparkR
library(SparkR)

# Initialize SparkContext and SQLContext
sc <- sparkR.init()
sqlContext <- sparkRSQL.init(sc)
```

# Getting IPython working with Spark

The easiest way is to create a new profile with the correct start up options. 

```bash
ipython create profile pyspark
```

Within the startup folder, we can create a script (`00-XX.py`) with the following:

```py
import sys

spark_home = "C:/Users/chapm/Spark"

sys.path.insert(0, spark_home + "/python")

# Add the py4j to the path.
# You may need to change the version number to match your install
sys.path.insert(0, spark_home + '/python/lib/py4j-0.8.2.1-src.zip')

# Initialize PySpark to predefine the SparkContext variable 'sc'
execfile(spark_home+ '/python/pyspark/shell.py')
```

Now running `ipython notebook --profile=pyspark` should launch your notebook with Spark enabled.

