---
layout: post
category : 
tags : 
tagline: 
---

Data Engineering.

>  In my view, Data Engineers can be thought of as the "Type B" Data Scientists - the builders. The Data Engineering role is a blend between Data Scientists and Software Engineers.
>  Recently, thats a term that has been appearing more and more (at least to me). I thought I'll look into two things:

*  What is it?
*  How does one become proficient at it?

To answer both of these questions, the easiest way is to look at the training which the major cloud providers have. Unlike Data Science, Data Engineering is definitely showing its youth - as the only player which seems to have a "free" MOOC available is Google for Data Engineering.

*  Google: Data Engineering on Google Cloud Platform Specialization on Coursera. This is "on-demand" training for Google's cloud certification
*  Microsoft: Offers certification with two exams. The suggested resource and training do no have an on-demand online training which can be audited. The costs are also much higher than Coursera (>$1k AUD per course)
*  AWS: Does not offer Data Engineering certification, but has a closely related "Big Data" one, which requires going through their pathway. There also isn't a MOOC where the material is presented for this either

There are lots of other courses from udemy and other places which purport to provide Data Engineering skills. Based on the above there are several components which appear to be core to a Data Engineering skill set (in order of my own perceived importance):

## 1 Understanding of a "Big Data" platform

This can be specific to the cloud provider or just general Hadoop knowledge. Data Engineers must understand the nuances in how platforms such as Google's Dataproc or AWS EMR works, how to configure, how to scale and how to tune Machine Learning (ML) assets which are created by a Data Science team.

## 2 Be able to construct and support Machine Learning Pipelines

As Google pushes their Cloud Dataflow/Apache Beam product, it becomes clear that there is a need to surface data in all forms in a structure sensible for ML. Examples of this could be mixing both batch and streaming sources of data for ML consumption.

If you weren't in the Google world, this could also be handled within technologies such as Spark, which demonstrates that perhaps on the key competencies of many Data Engineering roles - that is the knowledge of Spark/Scala programming.

In my own experience, one of the strongest differentiators between excellent Data Engineers is discernment to be able to avoid anti-patterns when working with Data Scientists. It is important to understand when decisions to rewrite code from one language to another is sensible or whether pipelines and transforms can be scaled in a more naive fashion.

## 3 Model Deployment

As ML becomes more and more commoditized, many Cloud providers are providing easier ways to build ML models and maintain them. Examples of these include the new AWS Sagemaker, and Google's Cloud AI. I believe these are useful tools for analytics teams but alone cannot solve all the deployment pains an analytics team might have. Afterall ML models often require several components:

*  Feature Preprocessing: where one might alter data based on their own domain expertise (including, text models etc)
*  Model scoring: this is where one might apply Deep Learning, or a GBM
*  Decision layer: this is where the actual score is converted to an insight or action, such as reason codes or trigger another pipeline in a system

This means that the role of the Data Engineer in this context is the facilitate and harden ML pipelines to ensure that data which flows in and out is appropriate for data scientists.

In many contexts, this might mean re-appropriating ML assets which the Data Science team might have created to something that can be easier to maintain and support from a platforms perspective. This can be particularly tricky if models which were built in batch were now required to be used off real-time data in a near-time (or real-time) sense.

This is where having Data Engineers which understand Data Scientist workflows make a massive difference, as collaboration in this scenario is important to ensure that real-time models can hit relevant SLAs if there is a hard constraint on aspects such as latency requirements.

Training and Moving Towards Data Engineering

I believe that training towards a Data Engineering role is not straight-forward. There clearly aren't as many resources out there to assist you with it. However there are a few things which are clear. Data Engineers need to:

*  Have knowledge around the data! Coming from a DBA background would help immensely. This is especially important to understand the trade-offs and optimization when data assets are to be served to Data Scientists.
*  Understanding Big Data tools. Similar to having a DBA background, being able to tune and understand trade-offs and optimizing Big Data Platforms is immensely important skill to keep analysts happy.
*  Be comfortable building, modifying and using ML models. Data Engineers absolutely need to understand ML models. Without that understanding it would be difficult to provide recommendations or be able to optimize machine learning pipelines which are constructed with Data Scientists. 

Overall, what are Data Engineers? In my view, Data Engineers can be thought of as the "Type B" Data Scientists - the builders. The Data Engineering role is a blend between Data Scientists and Software Engineers. The world has simply "rehashed" the term, driven by the large demand of Data Scientists and the belief that Data Scientists with the skills to have it all were simply too rare.

Then perhaps the alternative frame is that Data Engineers can be analytics professionals:

*  Who gain formal software engineering training in things related to data management
*  Focus on understanding and optimizing platforms suitable for analytics pipelines
*  Continually build ML assets to be deployed - rather than in "adhoc" setting

Should we train analytics users to become better versed in software? Or software engineers to be better versed in analytics?

I'll leave that choice up to you.



