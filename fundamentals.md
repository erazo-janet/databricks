# What is a Data Lakehouse?

A Data Lakehouse is a modern data architecture that combines the scalability and flexibility of data lakes with the performance and reliability of data warehouses.

# Data Warehouse

A structured repository designed for analytics

- Ideal for structured data (like rows and columns in Excel)

- Acts as a single source of truth for business intelligence

- Expensive to scale and doesn't support unstructured or semi-structured data (like images, audio files, etc)

# Data Lake

- Stores unstructured and semi-structured data in raw formats (e.g., JSON, images, logs)

- Highly scalable and cost-effective

- Great for big data and ML pipelines, but lacks traditional warehousing features (e.g., ACID transactions, governance) 

# Lakehouse = Warehouse + Lake

- A unified architecture that can handle all types of data (structured, semi-structured, unstructured)

- data lives on cloud platforms (AWS, Azure, GCP)

- Combines the governance and performance of a warehouse with the flexibility and cost-efficiency of a data lake

# What is Data Intelligence?

Data Intelligence refers to using AI models and  gen AI to Learn and understand organizational data

# Data Lifecycle in Databricks
Data Sources → Ingest → Transform → Query → Visualize → Serve → Govern

- Data Sources
  - Data can come from databases, APIs, files, IoT sensors, logs, third-party apps, etc

- Ingest
    - Import, process, and store data from various sources
    - Tools: Apache Kafka, Auto Loader, Delta Live Tables

- Transform
    - Clean, filter, and prepare raw data into usable formats

- Query
    - Analyze the transformed data to generate insights

- Visualize
    - Create dashboards and reports to make data human-readable

- Serve
    - Make data accessible to downstream systems or ML models

- Governance & Security
    - Ensure data quality, consistency, and access control
    - Includes lineage, cataloging, encryption, and auditing.

# Where Does Your Data Live?

In Databricks, data typically resides in cloud object storage like AWS S3, Azure Data Lake Storage, Google Cloud Storage

# Principles of a Well-Architected Lakehouse
- Operational Excellence	Keep systems running smoothly in production
- Security	Protect data from unauthorized access
- Reliability	Recover from failures and ensure high availability
- Performance Efficiency	Adapt to changes in load and system requirements
- Cost Optimization	Manage cloud spend and maximize ROI
- Data Governance	Ensure compliance, quality, and oversight
- Interoperability	Seamless integration with tools, systems, and users

# Key Terms for Data Engineering & Architecture

- Delta Lake:	Open-source storage layer bringing ACID transactions to data lakes
- Partitioning:	Dividing data for faster query performance
- Data Catalog:	Metadata storage for managing and discovering datasets
- MLflow:	Tool for managing the ML lifecycle
- Apache Spark:	Distributed data processing engine used by Databricks
- Streaming:	Real-time data processing (vs. batch)
- Lineage:	Tracing data transformations from source to destination
- Medallion Architecture:	Bronze (raw) → Silver (cleaned) → Gold (aggregated) data layers
- Z-Ordering:	Optimizing queries by co-locating related data in storage
- Auto Loader:	Incremental file ingestion service in Databricks
- Unity Catalog: Centralized governance for data and AI assets in Databricks



