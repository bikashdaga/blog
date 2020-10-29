---
toc: true
description: "Using Databricks & Prefect to automate your ETL tasks"
categories: [prefect, databricks, etl, spark]
comments: true
image: images/prefect\_databricks/intro.jpg
title: "Tutorial: Integrating Prefect & Databricks"
---

Prefect is a workflow management system that enables you to easily take your data applications and add retries, logging, dynamic mapping, caching, failure notifications, scheduling and more - all with functional Python API. Prefect allows you to take your existing code, and with some small edits, transform it into a DAG with dependencies already identified [1]. It simplifies the creation of ETL pipelines and dependencies and enables you to strictly focus on the application code instead of the pipeline code (looking at you Airflow). Prefect even enables you to create distributed pipelines to parallelize your data applications.

Databricks at its core is a PaaS (Platform as a Service) around spark that delivers fully managed Spark clusters, interactive & collaborative notebooks (similar to Jupyter), a production pipeline scheduler and a platform for powering your Spark based applications. They are integrated in both the Azure and AWS ecosystem to make working with big data simple. Databricks enables users to run their custom Spark applications on their managed Spark clusters. It even allows users to schedule their notebooks as Spark jobs. It has completely simplified big data development and the ETL process surrounding it.

Databricks has become such an integral big data ETL tool, one that I use everyday at work, so I made a [contribution]() to the Prefect project enabling users to integrate Databricks jobs with Prefect. In this tutorial we will go over just that - how you can incorporate running Databricks notebooks and Spark jobs in your Prefect flows. 

## Pre-requisites
There is no prior knowledge needed for this post but however to follow along with the example and do it yourself you will need a free [Prefect Cloud]()() account.

## Prefect Basics
### Tasks

A task in Prefect is the equivalent of a step in your data pipeline. It is as simple as a Python function in your application or script. There are no restrictions on how simple or complex tasks can be. That being said, it's best to follow coding best practices and develop your functions so they only do one thing. Prefect themselves also recommend this.

> In general, we encourage "small tasks" over monolithic ones: each task should perform a discrete logical step of your workflow, but not more. [3]

By keeping tasks small, you will get the most out of Prefect's engine such as efficient state checkpoints.

### Flows

A flow is what ties all your tasks and their dependencies together. It describes dependencies between tasks, their order and data flow. Flows pulls together your tasks and makes it into a pipeline rounding our your data application.

## Native Databricks Integration in Prefect
I made a contribution to this project by the implementing Prefect tasks  called TODO: insert Databricks task names here enabling seamless integration between Prefect and Databricks. Through these tasks we can externally trigger notebooks and run Spark jobs. Once a task has been executed it uses Databricks native API calls to run notebooks or Spark Jobs. When the task is running It will continue to poll the current status of the run until it’s completed. Once a task is completed it will allow for downstream tasks to run if it is successful.

## Example
TODO: Example create sample Prefect Flow

## Conclusion
You now have all the knowledge you need to run Databricks Notebooks and Spark jobs as part of your ETL flows. For more information on Prefect and Databricks jobs, I recommend reading their documentation found [here][1] and [here]()().

## Feedback
As always, I encourage any feedback about my post. You can e-mail me at sidhuashton@gmail.com or leave a comment on the post if you have any questions or need any help.

You can also reach me and follow me on Twitter at [@ashtonasidhu][5].

## References

[1]:	https://docs.prefect.io/core/ "Prefect Documentation"
[3]: https://docs.prefect.io/core/getting_started/first-steps.html	"Prefect Getting Started"



[5]:	https://twitter.com/ashtonasidhu
[6]:	https://docs.prefect.io/core/ "Prefect Documentation"