# aws-data-engineer-exam-notes

These notes are intended for quick study/cramming for the [AWS Data Engineer Associated Exam](https://aws.amazon.com/certification/certified-data-engineer-associate/). 

## Course Recommendations

* https://www.udemy.com/course/aws-data-engineer/learn/lecture/40332324

## Sections

### RedShift

* A fully managed, petabyte scale data warehouse.
* Can also query against exabytes of data stored in S3.
* OLAP Database.
* Currently one AZ only.
* Features:
  * Columnular storage.
  * Data Compression.
  * Zone Maps (kind of like an index, e.g. min/max values in a block).
  * Parallel processing - use many hosts to run a query.
  * Result caching.
  * Backup to S3.
* Nodes
  * Leader Node - receives and coordinates queries.
  * Compute Node - execute queries.
* Redshift Spectrum
  * Query against S3.
* RedShift Streaming Ingestion
  * Amazon Kinesis Data Streams.
  * Amazon Managed Streaming for Apache Kafka (MSK).
* Redshift ML
  * CREATE MODEL sql statement.
  * Amazon SageMaker Autopilot used behind the scenes.
* STL_ALERT_EVENT_LOG
  * Records an alert when the query optimizer identifies conditions that might indicate performance issues.
  * Use this view to identify opportunities to improve query performance.
  * Visible to all users. 
    * Superusers can see all rows.
    * Regular users can see only their own data.
* Amazon Redshift supports streaming ingestion from Amazon Kinesis Data Streams
  * Can be pushed into a Materialized View.
  * Setup (assumes Kinesis DS already exists)...
    * Create a schema in RedShift that can consume a Kinesis DS...

```sql 
CREATE EXTERNAL SCHEMA kds
FROM KINESIS
REGION 'us-west-2'
IAM_ROLE { default | 'iam-role-arn' };
```

    * Create a Materialized View (AUTO REFRESH YES can be added, default is no)

```sql
CREATE MATERIALIZED VIEW my_view AUTO REFRESH YES AS
SELECT approximate_arrival_timestamp,
JSON_PARSE(kinesis_data) as Data
FROM kds.my_stream_name
WHERE CAN_JSON_PARSE(kinesis_data);
```

    * Refresh view...

```sql
REFRESH MATERIALIZED VIEW my_view;
```

* Amazon Redshift Data API
  * Simplifies access to Amazon Redshift data.
  * No need to manage db connections, drivers etc.

### DynamoDB

* Amazon CloudWatch Contributor Insights.
  * Summarized view of your DynamoDB table’s traffic trends.
  * Identify the most frequently accessed partition keys.
  * Useful for diagnosing throttling issues.

### [AWS Glue](https://aws.amazon.com/glue/features/)

* AWS Glue consists of:
  * Central metadata repository.
  * ETL Engine.
  * Flexible Scheduler

* AWS Glue Data Catalog
  * Persistent Metadata store.
  * The data that are used as sources and targets of your ETL jobs are stored in the data catalog.
  * 1 per region.
  * Can be used as a Hive metastore.
  * Can contain database and table resource links.
  * Database
    * Collection of table definitions, organized into a logical group.
    * Can be from multiple data sources.
  * Table
    * Can define tables using JSON, CSV, Parquet, Avro, and XML.
    * Can use the table as the source or target in a job definition.
  * Connection
    * Properties needed to connect to your datasource.
    * Options:
      * JDBC
      * Amazon RDS
      * Amazon Redshift
      * Amazon DocumentDB
      * MongoDB
      * Kafka
      * Network
  * Crawler
    * Populate the AWS Glue Data Catalog with tables.
    * Can crawl multiple data sources.
    * Determine the schema and format of data.
    * Writes metadata to the Glue Catalog.
  * Classifier 
    * Determine the schema of your data.
  * AWS Glue Studio
    * Visually author, run, view, and edit your ETL jobs.
  * Job
    * Central to the ETL process.
    * When creating a job, you need to provide data sources, targets, and other information. The result will be generated in a PySpark script and stored the job definition in the AWS Glue Data Catalog.
    * Job types: Spark, Streaming ETL, and Python shell.
  * Script
    * A script allows you to extract the data from sources, transform it, and load the data into the targets.
    * Scala / PySpark.
  * Trigger
    * It allows you to manually or automatically start one or more crawlers or ETL jobs.
    * You can define triggers based on schedule, job events, and on-demand.
  * Workflows
    * It helps you orchestrate ETL jobs, triggers, and crawlers.
    * Workflows can be created using the AWS Management Console or AWS Glue API.
    * You can visualize the components and the flow of work with a graph using the AWS Management Console.
  * Transform
    * To process your data, you can use AWS Glue built-in transforms.
    * These transforms can be called from your ETL script.
    * Enables you to manipulate your data into different formats.
  * Dynamic Frame
    * A distributed table that supports nested data.
    * A record for self-describing is designed for schema flexibility with semi-structured data.
    * Each record consists of data and schema.
    * You can use dynamic frames to provide a set of advanced transformations for data cleaning and ETL.
  * AWS Glue Schema Registry
    * Registry is a logical container of schemas. It allows you to organize your schemas, as well as manage access control for your applications. It also has an Amazon Resource Name (ARN) to allow you to organize and set different access permissions to schema operations within the registry.
    * It can manage and enforce schemas on data streaming applications using convenient integrations with Apache Kafka, Amazon Managed Streaming for Apache Kafka, Amazon Kinesis Data Streams, Amazon Managed Service for Apache Flink for Apache Flink, and AWS Lambda.
* AWS Glue Interactive Sessions
  * Allows developers to rapidly build, test, and run data preparation and analytics applications.
  * Provides a programmatic and visual interface for building and testing Extract, Transform, and Load (ETL) scripts for data preparation.
* AWS Glue DataBrew
  * Visual data preparation tool that enables users to clean and normalize data without writing any code.
  * 250 pre-built transformations.
  * Auto-generate 40+ data quality statistics like column-level cardinality, numerical correlations, unique values, standard deviation, and other statistics.
* AWS Glue Streaming ETL service
  * Real-time data preparation and loading.
* AWS Glue Flex
  * Allows you to optimize costs on non-urgent or non-time sensitive data integration workloads.
  * AWS Glue Flex jobs run on spare compute capacity instead of dedicated hardware.
  * The start and runtimes of jobs using Flex can vary because spare compute resources aren’t readily available and can be reclaimed during the run of a job.
* Glue Data Quality
  * Measure and monitor the quality of your data.
  * Built on top of the open-source DeeQu framework and provides a managed, serverless experience.
  * Works with Data Quality Definition Language (DQDL), a domain-specific language that you use to define data quality rules.
* AWS Glue Monitoring
  * Record the actions taken by the user, role, and AWS service using AWS CloudTrail.
* AWS Glue’s Sensitive Data Detection
  * Uses pattern matching and machine learning to automatically recognize personally identifiable information (PII) and other sensitive data at the column or cell level.
* AWS Glue Pricing
  * You are charged at an hourly rate based on the number of DPUs used to run your ETL job.
  * You are charged at an hourly rate based on the number of DPUs used to run your crawler.
  * Data Catalog storage and requests:
    * You will be charged per month if you store more than a million objects.
    * You will be charged per month if you exceed a million requests in a month.


### Athena

* `MSCK REPAIR TABLE` - This command works by scanning the specified directory in Amazon S3, which is linked to the Athena table, to discover and add any new Hive-compatible partitions.
* Amazon Athena Federated Query
  * allows SQL queries to be executed across multiple data sources without the need for data movement.
  * S3, Amazon Aurora MySQL, HBase on Amazon EMR, and Amazon DynamoDB.
  * Query data in-place, no need for ETL.
* Athena Workgroup
  * Allows saving/grouping of queries.
  * 2 Engines
    * Athena SQL
    * Apache Spark (process large datasets across multiple clusters)

### Quicksight
### Lambda

* Serverless compute service.
* Scales automatically.
* Stateless.
* Components
  * Function - a script or program that does something.
  * Execution Environment - an isolated environment where the function executes.
  * Runtimes - Python, Go, Powershell, Java, Node.JS, Custom.
  * Env Vars - kv pairs often used to configure functions.
  * Layers - slices of the execujtion enviroment. Often used to distribute libraries.
* Functions can be configured ot mount an EFS share.
* Functions can be invoked synchronously or asynchronously.
* Functions execute in their own VPC but can be configured to run in user VPCs.
* Lambda functions run in their own env/VPC and must be connected to user VPCs if interaction with user resources is needed.
  * An appropriate Security Group also needs to be attached to the Lambda function.


### AWS Batch
### AWS Step Functions
### AWS EMR
### AWS Lakeformation

* Fully managed service that helps you build, secure, and manage data lakes.
* Row-Level Permissions.
* Centralized auditing and compliance reporting.
  * Who accesses what, when and through what service?
* Persistent data stored in S3
  * Structured/unstructured data.
  * Transformed/untransformed data.
  * Aggregates data from other stores s3, databases etc.
* Data Catalog
  * Persistent metadata store.
  * Metadata about data lakes, data sources, transforms, and targets.
  * Databases/Tables - Permissions.
  * Each AWS account has one Data Catalog per AWS Region.
* Governed Tables
  * Unique to Lake Formation.
  * ACID transactions.
  * Automatic data compaction.
  * Time-travel queries.
* Resource links
  * Links to shared databases/tables in other accounts.
* Blueprints
  * A data management template to ingest data into a data lake.
* Workflow
  * Definitions for ETL Jobs

# AWS Data Exchange

* Share data with anyone that has an AWS account.
* GetDataSet API provided to facilitate consumption data.

# AWS S3

* AWS Event Notification
  * Create notifications when specific events occur in your S3 Buckets.
  * Can filter on...
    * Prefix, e.g. images/
    * Suffix, e.g. .csv
  * Can send event to Lambda functions.
    * Create a thumbnail when a profile picture if uploaded.
    * Process a csv file when one is uploaded.
* Amazon S3 VPC gateway endpoint.
  * Routes must be added for S3 traffic to the endpoint.
  * CIDR ranges for Amazon S3 as a destination -> VPC S3 Endpoint.
  * S3 traffic would then be routed through the endpoint, not the public Internet.
* Amazon S3 Object Lambda.
  * Add your own code to Amazon S3 GET, LIST, and HEAD requests.
  * Modify data before it is returned.
    * Filter rows.
    * Redact data.
    * Watermark images.
* S3 Standard-Infrequent Access
  * Designed for data accessed less frequently but requires rapid access when needed.
  * Lower storage price point than S3 Standard.

