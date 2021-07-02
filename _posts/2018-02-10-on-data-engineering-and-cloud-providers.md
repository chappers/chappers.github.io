---
layout: post
category : 
tags : 
tagline: 
---

Recently, I've been thinking "what are the skill sets required for a data engineer" and "how would one demonstrate data engineering knowledge"? 

Quick searches reveal several things:

*  There aren't a lot of (free) MOOCs on data engineering specifically
*  All cloud providers have exams specific to their cloud offering for "data engineering" (being Google, AWS, Microsoft/Azure) - none which are more "generalist"

But its clear that there are some commonalities in what "data engineering" is. To break it down, I thought I would take Google's specialization certificate from Coursera for a spin (n.b. the specialization is meant to assist in sitting the actual certification exam). The components of the course are:

1.  Google Cloud Platform Big Data and Machine Learning Fundamentals
2.  Leveraging Unstructured Data with Cloud Dataproc on Google Cloud Platform
3.  Serverless Data Analysis with Google BigQuery and Cloud Dataflow
4.  Serverless Machine Learning with Tensorflow on Google Cloud Platform
5.  Building Resilient Streaming Systems on Google Cloud Platform

Now although I don't personally use Google in my day job, we can easily "translate" what these might look like on AWS and by extension what are the roles and responsibilities that is being suggested:

1.  Understand the offerings in the cloud provider (e.g. the difference between Redshift, EC2, EMR, Kinesis, S3, which one is for compute or storage and in what situation would you use it)
2.  Cloud Dataproc is the Hadoop flavour from Google, which is analogous to AWS EMR. 
3.  Cloud Dataflow is similar to data pipelines and has been baked into a product called Apache Beam.
4.  Google Machine Learning includes both ML API (Tensorflow) and DataLab, which is analogous to AWS ML SaaS and Sagemaker. 
5.  Google's streaming model is most similar to AWS Kinesis streams. It is important to understand how to perform analytics on streams of data, and how to prepare streams for Data Scientists to use

Overall the topics focus on several things:

*  Providing and understanding the appropriate technologies which data scientist need to use. 
*  Construction of suitable machine learning pipelines to leverage data sources in sensible ways (in both batch and streamed)
*  Usage of commoditized ML solutions to streamline deployment

**On Dataproc/EMR**

This topic is around the adminstration, setup and fundamentals of how distributed computing work in a "no ops" scenario. One should strive to understand differences between master vs slave nodes and high level understanding of how you might scale these out and the consequences of doing so.

**On ML SaaS solutions**

Google seems to push Tensorflow to be able to perform all ML workflows. I believe Google has definitely commoditized machine learning - however Tensorflow itself is probably not the best answer to most commercial use-cases. The area is growing, but many models which analysts build are not simply "deployed ML models", but rather consist of several stages, which I would describe as:

*  Feature Preparation: this is used in the feature engineering sense to construct features based on domain expertise or automated manner
*  Model Scoring: this component is what is generally solved in ML SaaS solutions
*  Decision engine: models generally only provide a single number or category. Often times we would still need to perform an action based on this single number or based on a collection of models. This might include model _reasons_ to describe why a machine is making a recommendation. 

Based on this it isn't sufficient to see ML to merely be "solved" on the back of ML SaaS solutions. To be fair none of the providers explicitly say that it is solved; only the promise that deployment would be easier in a no ops scenario


**On Dataflow/Apache Beam and Data Pipelines**

Google touts Dataflow/Beam as a way of doing both batch and streaming workflows in one single sweep. My personal experience of using Apache Beam and working in these scenarios tells me that the integration with Data Science workflows can not merely be solved through using a tool - furthermore Apache Beam's lack of upgrade into the Python 3 landscape is problematic. I feel that this area is one that is still very young and has many opportunities to grow out for all cloud providers. In Google's scenario, the data types are relatively weak, and they do suggest using Dataproc (AWS EMR) to perform some of these transformations in Spark. Although this "works" and can lead to high performing pipelines, it has a tendency to lead to anti-patterns particularly due to the limitations around Spark library (Spark "feels" like it has prioritised ability to scale over providing a rich and complete ML framework)





