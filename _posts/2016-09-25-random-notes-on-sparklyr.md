---
layout: post
category : web micro log
tags :
---

I recently started playing with `sparklyr` and have found it an amazing package. Here are some notes from reading the source code and the documentation:


## `sparklyr::compile_package_jars`

This method compiles the jars, but for whatever reason basically disobeys every single default installation path! How can we get this working?

Reading the [documentation](http://spark.rstudio.com/extensions.html) didn't help me either. It claimed that the compilers had to be in the following paths:

*  /opt/scala  
*  /opt/local/scala  
*  /usr/local/scala  
*  ~/scala (Windows-only)  

Finally realised it means that the location of `scalac` should be:

```sh
$location/scala-$version/bin/scalac
```

What I found to be useful is simply to discard the default settings in `spark_compilation_spec` but instead we need to use `spark_compilation_spec`. For example, to run the `sparkhello` example:

```sh
sparklyr::compile_package_jars(
  sparklyr::spark_compilation_spec(spark_version="1.6.1",
                                   scalac_path=find_scalac("2.10"),
                                   jar_name="sparkhello-1.6-2.10.jar")
)```

Since `sparklyr` is still in development, we have to be careful of the folder structure within our packages. At the time of writing the location of the `*.scala` file has moved from:

*  `inst\\scala`

To

* `java`
