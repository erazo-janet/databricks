getting started with databricks for data engineering in databricks academy 
# Using Databricks for Data Engineering
catalog is your window for unity catalog
below catalog are schemas 
then tables, volumes, views and model
so you have a catalog called dbacademy then schemas below

to setup a git folder, you need to connect git in your settings to your repository 
organizing your work is as simple as dragging and dropping to folders 

workspace lets tyou find folders and files
below workspace is catalog 

# Lakeflow overview 
Lkeflow consists of 3 compontnets:
lakeflow connect - connect to any source without code
lakrflow declarative pipeliens - for automated batch or streaming transformations
lakeflow jobs for orchestraction, triggers and monitoring

## Lakeflow connect
simple connectors to connect data to databricks like volumes or other sources
it supports 3 main types of ingestion: manual file uploads to upload as a volume or a table, standard connectors that support ingestion from cloud sources adn more, with batch and incremental batch and streaming ingestion methods, and managed connectors with ingests data from SAAS applications and databases . So you can use this to connect to SQL Server 
once data is ingested, declarative pipelines allows for etl with sql or python to create your bronze, silver and gold layer 


## Delta lake overview
when data arrives it lands to your orgs data lake 
which is in a lakehouse 
within delta lake you utilize delta tables. under the hood the data gets stored as files within a directory and the data is stored as parquet files and delta adds delta log stored as json files to track the transactions with metadata
so if you isnert, delete, update data delta lake makes a record in transaction log so you can see whats going on with your data and travel back in time
behind the scenes, delta lake is the default format for tables created in databricks
so you dont have to specify CREATE TABEL orders USING DELTa cus it automaticallu does it
delta table features:
ACID transactions

DML operations
Time travel
schema evolution

ACID transactions - Atomicity - entire transaction comples or fails, consistency - data follows rules or itll be rolled back , isolation - one transaction completed before the start or another , durability - data is saved in a presistent sate once completed 

DML operations - data manipulation language operations such as:
INSERT
UPDATE
DELETE
MERGE - select records from a source tbale and performs DML operations on anotehr table

Time travel - delta tables keeps sa log for each version (Writes) of the table so you can query as data existed in any given moment  
Schema evolution - automatically adjust the schema of your delta table as your data changes like add new colums or data types in your table so any data written tothe delta table mattches the tables defined scheuma

Volumes contain your files like if you have a csv file

USE CATALOG dbacademy
USE SCHEM IDENTIFIER)da.schema_name) identifier clause interpets the constant string in our variable as a function name, column name, field name schema name, catalog name so you dont have to type your schema itll reference it instead
SELECT current_catalog, current_schema to view your settings that youre using in your notebook

USE CATALOG dbdemos_retail_360
USE SCHEMA IDENTIFIER(da.schema_name)

You can do this in spark too
spark.catalog.setCurrentCatalog(da.catalog_name)
spark.catalog.setCurrentDatabase(Da.schema_name)
spark.catalog.listTables(da.schema_name)

SELECT current_catalog(), current_schema()

DESCRIBE SCHEMA EXTENDED IDENTIFIER(da.schema_name) shows you info about your schema 
SHOT TABLES; to see your tables in your schema or database

SHOW VOLUMES- volumes are unity catalog objects representing the logical volume of storage in a cloud object storage location. volumes provide capabilities for accessing, storing, governing and organizing files. its essentially the files you have stored. theyre not tables thyre just files like csv text etc. so SHOW VOLUMES will show u what csv fileyou have in your database/schema
what if you want to see your files in a volume? write:
spark.sql\(f"LIST '/volumes/dbacademy/({da.schema_name}/myfiles'").display()

now that we know we have a file in a volume within our schema within our catalog, we can create a delta table




## Delta lake lake flow connect techniques
how you can create tables:
CREATE TABLE AS creates table by selecting data from an existing table or data source. defualt table format is delta
upload ui: upload csv, json, parquet, text files and you can upload files directly to a unity catalog volume
copy into: loads files from a file location into a detla table, supports many file formats and cloud storage locations, automatically handles schema changes and has idempotent operation - already loaded files in the source are skipped
autoloader: incrementally processes new data files as they arrive in cloud storage
automatically infers schemas and accomodates schema changes
includes a rescue data column for data that does not adhere to the schema


## Ingesting data into delta lake
CREATE TABLE AS:
CREATE TABLE CURRENT_EMPLOYEES AS 
SELECT
  ID,
  FIRSTNAME,
  COUNTRY
FROM read_files(
  'volumes/dbacademy/' || da.schema_name || '/myfiles/'
  format => 'csv'
  header => true
  inferschema => true
)

COPY INTO: load files from your volume. but if it already has the data, then itll skip and just grab the new loads

COPY INTO current_employees_copyinto
FROM 'volumes/dbacademy/schemaname/myfiles/'
FILEFORMAT = CSV
FORMAT_OPTOINS(
'header' = 'true' -- to tell the notebook the file has headers
'infer schema' = 'true' -- figure out the schema. if this is already defined itll know what to use
) 
''').display()

if you run this then run it again, itll say 'i loaded that file already so i wont do anything' so itll show that nothing was done or inserted, cus it keeps track what it has already done


## data transformation overview
Bronze - you can remove PII 
silver - defines the structure of the data, and becomes the single source of truth 
gold - clean data ready for consumption, where business level aggregates happen 
if you do deletes, inserts, merges, etc it can do it throughout the data transaformation process. so if you get new data, all layers get the data. if you delete or merge data between bronze and silver, gold will ee the effects

BRONZE:
DROP TABLE IF EXISTS customer_employees
CREATE TABLE IF NOT EXISTS customer_employees ...
COPY INTO current_employees_bronze ....

SILVER:
CREATE OR REPLACE TABLE current_employees_silver AS
select
id,
firstname,
current_Timestamp() as currenttimestamp -- adding the date field 
upper(Role_ as Role --this upercases the role column,
from ....

GOLD
CREATE OR REPLACE TEMP VIEW view_total_roles
SELECT ROLE, COUNT(*) AS TOTAL_eMPLOYEES
FROM CURRENT_EMPLOYEES_sILVER

CREATE TABLE IF NOT EXISTS TOTAL_ROLES_GOLD
( Role STRING,
TotalEmployees INT);

INSERT OVERWRITE TABLE total_roles_gold
SELECT * FROM temp_total_roles

so now you can see a summary of your roles and total employees per roles

now that you have bronze silvedr gold, look at data governance and security. you can open catalog explorer to see details on your table, including lineage, permissions to users or groups in the table, schema or catalog level, history to see when it was used and lineage grpah will show you where the data is coming from

## performing etl with lakeflow delatative pipelines
delarative ipelines -
simplified pipeline authoring - use sql or python to do your transformations
interlligent optimization at scale - pipelines scale and recover upon failure as your data grows
unified batch and streaming - for near real time and batch workloads
Bring data into databricks using lakeflow connect like from cloud storage, databases, saas like salesforce or workday
then lakeflow declarative piplines can be used to follow the medallion architecture 

#Unified orchestration using Lakeflow Jobs
theyre fully managed cloud based general purpose task orchestration for databricks
pipelines can be a task in a lakeflow job
build jobs on notebooks, job for sql, ml models, python code, and delcarative pipelines

with a job you etl, refresh dashboards, and more

in a notebook, you can generate the lakeflow job confiration
%python
da.print_lakeflow_job_info
first create a workflow to create a job or pipeline, so create a job here
in your tasks, you tell it what u want it to perform. give it a name like setup bronze, under type select what do you want to run? a notebook, pythons script? etc.
condition would be like if this row is >0 then do xyz
then select your notebook
paramters is variables u can use in your task from your notebook

create a second task and call it 'silver to gold'
you can say if this job fails or succeeds if it should keep going or not
schedueles and triggers - run it when i want it to, when a new file arrives, continuous if you want it to continuously run, or scheule for the day and time you want
job notifications for start, success and failures

## quiz questions i got wrong
what are the primary high level configuration options avaliable when setting up a warehouse?
answer:compute cluster size, auto stop timer and scaling parameters. i chose data compresion, clsuter name and query optimization



--question, so if i upload a file and do bronze silver gold i need to create a job daily so it can capture new data?




