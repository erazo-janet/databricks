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

### Clusters summary

Now for clusters: Imagine you have 1 million patient records and want to capitalize first names:
Without a cluster: One computer processes 1 million records one by one → slow.
With a cluster of 5 worker nodes:
Each node processes ~200,000 records in parallel → much faster.
Driver coordinates the work and merges results.

The type of cluster you use depends on how you want to run your spark job
All purpose cluster is used when youre developing your notebook, want to run transformations interactively, test logic, preview data, debug, etc.  a job cluster is used to run scheduled jobs and terminates when done. so you would use it on ETL transformations lets say that run daily, weekly, etc and you dont need to interact with the data while running it 

# Apache Spark Dataframes
Dataframes can be created from various multiple sources, like JSON, CSV, Parquet, text, more. They also support delta lake or other table storage format directories and can be created from tables or views in Unity Catalog, external databases, etc. Ex. a csv file can be read using spark.read.csv()
Behind the scenes: when a dataframe is evaluated, the driver creates an optimized execution plan through  a series of transformations, converting hte logical plan into a physical execution plan that minimizes resoruce usage and execution time

Catalyst optimizer and photon:
Catalyst Optimizer: this is a query optimization engine that converts dataframe operations into an optimized execution plan. it applies rule based and cost based optimizations to improve query performance
Photon engine: a vectgorized query engine that accelerates query execution by processing data in batches rather than row by row. its enabled by defauly in sql warehouses and serverless compute

Columnar Storage
<img width="223" height="246" alt="image" src="https://github.com/user-attachments/assets/50f8aeb3-67cf-43ff-b228-959a306ee3d5" />

- data is organized by columns rather than rows, being efficient for analytical workloads. it is implemnted in dataframe internal storage and in file formats like parquet 

## Dataframes
The DataFrameReader (spark.read) supports multiple file formats and schema, and DataFrameWriter(dataframe.write) allows flexible output formats and paritoning

df = spark.read.format("format").opton(..).load()
df = spark.read.csv("c:/...")
df = spark.read.parquet("s3://....")

df.write.csv("c/...")
df.write.parquet("s3/...")

Every dataframe has a defined schema which is the structure and data types of all columns
It can be inferred from data or explicitly specififed:
from pyspark.sql.types import *
schema = StructType([
  structField("name", stringtype()),
  structfield("Age",intergertype()) ])
  use printschema() to print out the dataframe schema

  instead of using StructType for defining the datagrame schemas you can use DDL strings which provide a more readable format

  ddl_schema - "name STRING NOT NULL, age INT, city STRING"
  df = spark.read.csv("c:/.."schema=ddl_schema)
  df.printschema()
<img width="1168" height="635" alt="image" src="https://github.com/user-attachments/assets/3fe8ab5b-45cd-4307-97b2-5dd30c3d1938" />
<img width="493" height="126" alt="image" src="https://github.com/user-attachments/assets/3c2c47a8-72be-46c1-be0a-b33aa9c864a8" />

  Once dataframes are created, their data cant be mofidied. Instead, you create new dataframes from existing ones for transofrmations. Then actions like showing or saving the output triger actual comutation and produce final results. Multiple transofrmations can be called, and the job is only created when an action is requested, called lazy evaluation

  <img width="495" height="274" alt="image" src="https://github.com/user-attachments/assets/d80e1e47-8583-4f16-aa5a-3b4b278c28d4" />

  <img width="508" height="276" alt="image" src="https://github.com/user-attachments/assets/4432ed85-75d6-4e15-a370-787ec0fb8617" />

customers_df = spark.read.format("csv")\
.opton("header","true")\
.option("inferschema","true")\
.load("/volumes/v01/sourcefiles)

customers_df.printschema() --shows you the columns of the headers and the type
display(customers_df) --to see your data, results are limited to 10k rows or 2mb
when you display the results, you can click on the plus sign > data profiling to see more info on your data in the dataframe 
<img width="836" height="454" alt="image" src="https://github.com/user-attachments/assets/e64c970d-aa27-4454-8042-c6847c608ec2" />

you use infer schema often when you load from csv and text files not so much parquet
so if youre reading from csv you can define hhe schema and define the structtype and data fields 

after you diaplsyed the data in a dataframe for a csv file you can write it out to a parquet file
first define a file path:
parquet_output_volume_path = f"volumes/{da.catalog_name}/default/vo1/customers_parquet"

then write the dataframe out as parquet files into a directoey
customers_ddl_df.write.format("parquet"\
.mode("overwrite")\
.save(parquet_output_volume_path)

then to see the files in your directory:
display(dbutils.fs.ls(parquet_output_volume_path) --dbutils is a built in package in databricks

we can also save our dataframe to a new table:
customers_ddl_df.write.saveAsTable("customers_ddl_df_table")

or you can use the writeTo method 
customers_ddl.df.writeTo(
f"{da.catalog_name}.defualt.customers_ddl_df_table"
).createOrReplace()

select * from customers_ddl_df_table (or use display)

# Shared Nothing Architecture
- In this architecture, each node has its onw memory, disk, and cpu, reducing contention and improving scalability
- independende:  nodes communicate when neccessary only, usually when shuffling or writing results
- scalability: scalability is achieved by adding more nodes with performance scaling linerarly if partioning is done correctly
- fault tolerance is ensured as failures are isolated to specific nodes, allowing others to continue processing, minimizing system wide impacts
- resource partiioning: divides and distributes data across nodes enabling parallel processing so it can eliminate contention

## Partioning
In Spark, data is broken into chunks called paritions, which are processed independently, not to be confused with table, disk paritoning. 
How spark distributes data across the cluster:
- Data Distribution - data is divided into mutually exclusive in memory paritions, based upon input and can be maipulated, and size and number of paritions affect parallelism. Each partition is processed by a single task within a spark job
- Processing model
- - each parition processed independently
  - multiple paritions can run in parallel
  - one partition = one task in spark
  - impacts performance
 
## The Shuffle Operation
    <img width="520" height="270" alt="image" src="https://github.com/user-attachments/assets/c0f58b01-c8d2-4cf8-846a-afc384d76ff0" />

  - Shuffling is the redistribution of data across teh cluster to complete operations such as groupBy, join and sorting
  - its necessary when data needs to be aggregated or combined across paritions
  - it happens when operations that requre data organization like group by or join and hte overhead increased with hte number of paritions involved
  - minimzing suffling is critical for performance, and optimziation techniques include co-located data

## Map Reduce in Action
Understanding how spark executes Map, Shuffle and Reduce is key to optimization, properly tuning each stage can improve performance
<img width="521" height="292" alt="image" src="https://github.com/user-attachments/assets/5d3a8528-da3f-49e5-9eb5-a4513f1966ac" />
-The Map Stage is the inital transformation where data is filtered, mapped or preprated. operations like select and filter are narrow transformations, meaning data remains in teh same node
- The shuffle stage resturctures data acorss partitions to meet aggregation or join requirements and is the most resource intensive stae of the map reduce pattern
- the reduce stage involves aggregation or final transformation of the shuffled data with results often written to storage or returned to the driver

Spark follows a map/shuffle/reduce model for most operations. even simple transofrmations like filter() can involce these stages if repartitioning occurs. common excmles inclde groupBy, which extract keys, shuffles, adn then aggregates:join, which prepares the keys, shuffles and then combines; then filter, which evaluates a condition and requires no shuffle or reduce

# Basic ETL Operations with DataFrame API
<img width="507" height="228" alt="image" src="https://github.com/user-attachments/assets/89add9d8-0afd-42b5-9f4d-57d9943c0ed8" />

- DataFrame.join with a second ataframe as the other argument and you join two dataframes by a key

<img width="506" height="257" alt="image" src="https://github.com/user-attachments/assets/9224ec85-fe6f-4348-a5b5-b1099e8aeb6e" />

<img width="495" height="264" alt="image" src="https://github.com/user-attachments/assets/e9bbc842-8a58-4cbe-b464-55aa42356f91" />

<img width="478" height="277" alt="image" src="https://github.com/user-attachments/assets/3eb5cf7e-7ee2-4a50-8916-c40a34ca0b1a" />

<img width="520" height="281" alt="image" src="https://github.com/user-attachments/assets/71aca975-29b3-447f-a781-9b0719978a93" />


# Example walkthrough:
first, read the data into a dataframe:

flights_df = spark.read.table("dbacademy_airline.vo1_flights_small")
flights_df.printschema() --inspect the schema
display(flights_df.limit(10)) -- view your data

Now we can do transformations. if there are columns you dont need, you should do it first

flights_required_col_df = flights_df.select( --this is you creating a new data frame. since this is a transofrmation it wont kick off a job
"year"
"month"
...
)

now lets get a count of records
inital_count = flights_required_cols_def.count()
print(f"soruce data has {inital_count} records")

now lets create a view:

flights_required_col_df\
.selectExpr(
"year",
"month",
"CAST(DepTime AS INT) as DepTime",
...
)\
.createOrReplaceTempView("flights_temp")

now you can use spark sql to count null values

invalid_counts_sql = spark.sql("""
SELECT
COUNT_IF(YEAR IS NULL) AS NULL_YEAR_COUNT)
...
FROM FLIGHTS_TEMP
""")

<img width="886" height="383" alt="image" src="https://github.com/user-attachments/assets/87ccc853-35c3-48f4-ab4d-ffec29b337f5" />

you can perform these same things with spark api in python:

<img width="907" height="482" alt="image" src="https://github.com/user-attachments/assets/be89cc6e-549d-4a73-9a7a-0e9e7d7feffc" />

with every dataframe you can run a explain plan, to see how does it compile to execute:

sql_plan = invalid_Counts.sql.explain()

df_plan = invalid_counts_df.explain()

now you can say is the sql plan equal to the dataframe plan?
sql_plan == df_plan

now we can clean the data
lets drop rows where columns are null, so e can use na.drop

non_null_flights_df = flights_required_cols_df.na.drop(
how = 'any',
subset=[CRSElapsedTime']) --dropping nulls for this column

now we can remove rows with invalid values for certain columns 

flights_with_valid_data_Df = non_null_flights_df.filter)

col)"ArrDelay").cast("integer").isNotNull() )

now that our column contains integer values only, we can cast them from strings to integer, replacing the existing columns:

clean_flights_df = flights_with_valid_data_fd\
.withColumn("ArrDelay",col("arrdelay").cast("integer"))


<img width="877" height="288" alt="image" src="https://github.com/user-attachments/assets/b0196f43-cce4-499e-a3c4-c76c4293f7d2" />

so every transformation being created is being saved to new dataframes 
