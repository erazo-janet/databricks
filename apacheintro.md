# Introudction to Apache Spark

## Apache Spark Runtime Architecture
<img width="902" height="538" alt="image" src="https://github.com/user-attachments/assets/13dca747-0afe-4aba-a579-16a8392d097e" />

- Driver: this is the brain of a spark application, it plans and coordinates execution
- Cluster manager/master - manages/allocates cluster resources and allocates them to the driver
- workers: nodes in the clustor that hosts executors
- Executors: processes on worker nodes that execute tasks assigned by the driver 

### The Spark Driver
- this manages the application lifecycle by creating the SparkSession, which is the entry point for all spark applications for communication with the cluster
- it analyses the spark app and constructs a DAG, Directed Acyclic Graph
- it schedules and distributes tasks to execotrs for execution, and monitors progress and handles failures to ensure fault tolerance and returns the result to the client

SparkSession is created for you automatically as spark

from pyspark.sql import SparkSession
spark = SparkSession.builder / 
  .appName("MySparkApplication"\
  .config("spark.some.config.option","some-value")\
  .getOrCreate()

  ### Executors
  - these run on worker nodes in a Spark cluster and hosts Tasks and comminicates the results back to the driver
  -since executors run on worker nodes, each node is capable of hosting multiple executors, depending on avalaible CPU cores and memory resources
  - you can store intermediate and final results in memory or on disk
  - they interact with the driver by processing assigned tasks and reporting status 

  ### The Spark DAG
  <img width="469" height="369" alt="image" src="https://github.com/user-attachments/assets/33426c93-25df-49f2-b601-d066b1c748ed" />

  - spark jobs are broken down into stages, which are groups of tasks that can be executed in parallel
  -operations flow in a directed manner from one stage to the next, with no backward loops so all in oen direction (the directed bit)
  -since the stages never loop back, the acyclic structure ensures computations complete successfully witout infinite loops (the acyclic bit)
  - the stages are organized into a dependency graph, a dependency graph is used to plan the execution flow for paralel processing (the graph bit)

<img width="839" height="416" alt="image" src="https://github.com/user-attachments/assets/2da685de-3051-4dcb-95aa-4e7886326221" />

Here, jobs are divided into independent stages so it can run in parallel. the tasks are the individual units of work executed by the executors. in parallel exectuion, tasks within a stage run independently using a 'shared nothing' architecture
Databricks has a spark UI to monitor the spark application, like the dag, resource usage, etc and the master UI which shows clusterwide view of node health and resource allocation

### Spark Clusters in Databricks
theres different ways to deploy spark clusters, including:
- All purpose clusters - these are clusters that support notebooks, jobs and dashboards with configurable auto termination
- job clusters - ephemeral clusters that start when a job runs and terminates automatically upon completion, optimzed for non interactive workloads
- SQL Warehouses - optimized clusters for SQL query performance with instant startup and auto scaling to balance cost and perfroeance 


## Summary check so far:
Spark is a data processing engine that lets you process large datasets quickly. It can handle data in memory, making it faster
A spark application is a program you write that uses spark to do something with data. 

The driver splits the job into smaller tasks. The cluster manaer assigns tasks to differnt executors. executors process the data in parallel, store results in memory so its faster and return the results to the driver

Non technical example:
Imagine you run a library and need to categorize 1 million books by genre:
- Driver → You, deciding the plan: “Sort books by genre.”
- Cluster Manager → Your assistant assigning different shelves to different librarians.
- Executors → Librarians who are actually moving books and la

so when you start a spark session, SparkSession automatically creates a driver for your application. Cluster manager is already setup, and executors are automatically started by the cluster manager, and tasks are created when you perform operations like df.show(), df.filter(), df.groupby()
So you dont have to set these things up when creating a spark application. you only need to change things if you want more control or are running large workloads so youd change the number of executors, memory per executor, number of cores per executors


Now for clusters: Imagine you have 1 million patient records and want to capitalize first names:
Without a cluster: One computer processes 1 million records one by one → slow.
With a cluster of 5 worker nodes:
Each node processes ~200,000 records in parallel → much faster.
Driver coordinates the work and merges results.
