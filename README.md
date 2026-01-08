# snowflake


# **Snowflake Complete Syllabus (Merged & Structured)**

---

## **Topic 1 – Snowflake Architecture**

* Snowflake architecture overview
* Storage, Compute, and Cloud Services layers
* Separation of storage and compute
* Multi-cluster shared data architecture
* Scalability and concurrency handling

---

## **Topic 2 – Data Warehousing Concepts**

* Columnar storage fundamentals
* Micro-partitioning architecture
* Automatic partitioning vs traditional partitioning
* Clustering vs partitioning
* Cluster keys and their impact on query performance
* How micro-partitions improve query pruning

---

## **Topic 3 – Stages and SnowSQL**

* Internal stages vs external stages
* Table stage vs named internal stage
* External stages with AWS S3
* Storage integration and security model
* SnowSQL CLI usage
* Secure data access and credential management

---

## **Topic 4 – File Formats & Semi-Structured Data**

* File format creation and management
* Supported file types (CSV, JSON, Parquet, Avro, ORC, XML)
* Compression types and performance considerations
* Semi-structured data handling using VARIANT
* JSON data loading and querying
* Querying JSON using `LATERAL` and `FLATTEN`
* Performance optimization for semi-structured vs CSV data

---

## **Topic 5 – Data Loading & Unloading**

* COPY INTO command (inbound & outbound)
* Loading from internal stages
* Loading from external stages (S3)
* Secure unloading to S3
* Bulk loading vs continuous loading
* Error handling and data validation

---

## **Topic 6 – Snowpipe (Continuous Data Ingestion)**

* Snowpipe architecture
* Continuous data ingestion
* Auto-ingest using cloud events
* Manual vs auto-triggered Snowpipe
* Cost considerations and optimization strategies
* When to use Snowpipe vs COPY INTO

---

## **Topic 7 – Streams (Change Data Capture)**

* Streams architecture
* Change Data Capture (CDC) concepts
* Table streams vs view streams
* Insert, update, delete tracking
* Stream consumption behavior
* Best practices for CDC pipelines

---

## **Topic 8 – Tasks (Automation & Scheduling)**

* Task creation and execution
* Scheduling tasks using CRON
* Task dependencies and DAGs
* Automating ETL pipelines
* Integrating streams with tasks
* Error handling and monitoring tasks

---

## **Topic 9 – Time Travel & Fail-safe**

* Time Travel architecture
* Querying historical data
* Undrop and data recovery
* Fail-safe concepts
* Cost implications of Time Travel
* Cost optimization (batch deletes before 24 hours)

---

## **Topic 10 – Query Performance Optimization**

* Query profiling using Query History
* Understanding Query Profile
* Avoiding full table scans
* Micro-partition pruning techniques
* Effective use of clustering keys
* Warehouse sizing strategies
* Result caching behavior

---

## **Topic 11 – Clustering & Search Optimization**

* Automatic clustering vs manual clustering
* Cluster key design best practices
* Re-clustering costs and monitoring
* Search optimization service
* When clustering is required and when to avoid it

---

## **Topic 12 – Materialized Views**

* Materialized views vs standard views
* Automatic refresh behavior
* Incremental maintenance
* Performance benefits and use cases
* Cost considerations

---

## **Topic 13 – Security & Access Control**

* Role-Based Access Control (RBAC)
* User, role, and privilege management
* Network policies
* Data encryption at rest and in transit
* Secure data access patterns

---

## **Topic 14 – S3 Integration & Security**

* AWS IAM setup for Snowflake
* Storage integration objects
* Secure data loading from S3
* Secure data unloading to S3
* Cross-account access best practices

---

## **Topic 15 – Data Sharing**

* Secure data sharing concepts
* Zero-copy data sharing
* Cross-account and cross-region sharing
* Secure views for controlled sharing
* Provider and consumer accounts

---

## **Topic 16 – Zero-Copy Cloning**

* Zero-copy cloning architecture
* Cloning databases, schemas, and tables
* Use cases for development and testing
* Impact on storage and cost

---

## **Topic 17 – Dynamic Data Masking & Row Access Policies**

* Dynamic data masking
* Column-level security
* Row access policies
* Policy enforcement and performance impact

---

## **Topic 18 – Monitoring & Governance**

* Query history views
* Usage and billing views
* Account-level governance
* Monitoring credit usage
* Auditing and compliance

---

## **Topic 19 – Cost Optimization Strategies**

* Warehouse sizing and scaling
* Auto-suspend and auto-resume
* Result caching to reduce compute cost
* Avoiding unnecessary clustering
* Snowpipe vs batch load cost comparison

---

## **Topic 20 – Modern Table Types**

* Hybrid tables (OLTP workloads)
* Iceberg tables (open table format)
* External tables
* Choosing the right table type

---

## **Topic 21 – Event Tables & Observability**

* Event table architecture
* Logging and tracing
* Monitoring Snowflake activity
* Observability best practices

---

## **Topic 22 – Handling Unstructured Data**

* Managing PDFs, images, and audio
* External storage integration
* Metadata extraction

---

## **Topic 23 – Snowpark**

* Snowpark architecture
* Python, Java, and Scala APIs
* Pushing computation to Snowflake
* Performance and security considerations

---

## **Topic 24 – Catalog & Metadata Management**

* Snowflake Catalog
* Polaris / Catalyst integration
* Metadata management
* Data discovery and governance

---



Just tell me what you want next 🚀
