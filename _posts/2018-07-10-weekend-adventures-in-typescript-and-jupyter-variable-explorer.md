---
layout: post
category : 
tags : 
tagline: 
---

For data scientists moving their workflows from their desktop computer to the cloud, one of the hardest parts of "letting go" is the lack of an IDE. More specifically, one of the most requested features is a variable explorer like in [Spyder](https://pythonhosted.org/spyder/variableexplorer.html). 

Over the weekend I decided to take a stab into Jupyterlab extensions working off some of the initial work here: https://github.com/lckr/jupyterlab-variableInspector

Lessons learnt and additions made:

*  Dealing with changing kernels and languages
*  Improvements for Numpy and DataFrame
*  Ability to interact with Tensorflow and Spark items

Changing kernels:

![changing kernel](/img/jupyterlab/changing_kernel.gif)

Launching and Peaking into a Pandas dataframe:

![pandas](/img/jupyterlab/pandas_launch.gif)

Peeking into a Spark DataFrame - this runs a `<spark_dataframe>.limit(<max_row limit>).toPandas()` to display into `phosphorus datagrid`. 

![pandas](/img/jupyterlab/sp_df.gif)

Peeking into a Tensorflow Variable (from Keras).

![tf variable](/img/jupyterlab/tf.gif)

I believe all of these will speed up analytics workflows and make it easier for analysts to use these various frameworks. This addresses the following:

*  Data scientists no longer have to keep typing `.show()` or `toPandas()` to get results that are sensible
*  When dealing with Keras models over Tensorflow, it becomes easier to inspect the nature of the intermediate layers

