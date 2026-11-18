## Section 6: Project Experience & Interview Defense (Questions 96 – 110)

### Q96: Give a brief about your last project and what was your individual responsibilities?

**STAR Project Brief:**
- **Project Overview:** Built an enterprise real-time Lakehouse on Azure Databricks for an omnichannel retail platform processing 12M orders and 80M telemetry events daily ($4.5	ext{TB/day}$).
- **Individual Responsibilities:**
  - Designed the end-to-end Medallion data flow (Kafka $ightarrow$ ADLS Gen2 $ightarrow$ Delta Lake).
  - Authored core PySpark Structured Streaming jobs with off-heap RocksDB state stores.
  - Implemented automated deduplication, SCD Type 2 dimensional tracking, and data quality expectations.
  - Built GitOps CI/CD deployment pipelines using Databricks Asset Bundles (DABs) and GitHub Actions.

---

### Q97: Explain End-to-End architecture of your previous project?

**End-to-End Architecture:**
1. **Ingestion Tier:** Real-time microservices published order CDC and clickstream events to Azure Event Hubs (Kafka protocol). Auto Loader (`cloudFiles`) continuously ingested files into ADLS Gen2.
2. **Bronze Layer:** Raw JSON landing into append-only Delta tables with `_rescued_data` column capturing malformed payloads.
3. **Silver Layer:** Stream-stream joins and analytical cleaning jobs validating data with DLT expectations and deduplicating on `(order_id, timestamp)`.
4. **Gold Layer:** Star schema dimensional marts with SCD Type 2 customer histories clustered via Delta Liquid Clustering.
5. **Consumption Tier:** Databricks SQL Serverless warehouses powering real-time executive dashboards in Power BI with $< 3	ext{s}$ query latencies.

---

### Q98: What data volume were you handling? What major challenges did you face and how did you optimize?

**Production Metrics & Challenge Defense:**
- **Volume:** **4.5 TB per day** of raw compressed telemetry and transaction data; **1.2 PB cumulative** Delta Lake storage.
- **Challenge - Severe Data Skew & Disk Spill:** During promotional flash sales (e.g. Black Friday), a handful of mega-merchants generated 60% of all traffic. A Sort-Merge Join on `merchant_id` caused single tasks to process 45 GB of data, causing repeated Executor OOM crashes (`Exit code 137`) and 350 GB of Tungsten shuffle spill to disk.
- **Optimization:**
  1. Enabled Spark 3.x AQE Skew Join (`spark.sql.adaptive.skewJoin.enabled = true`).
  2. Implemented dynamic salting with 16 random salt keys for unresolvable non-equi joins.
  3. Switched target Delta tables to **Liquid Clustering** on `(merchant_id, event_date)`.
  4. Outcome: Job runtime plummeted from 2 hours 45 minutes to 18 minutes with zero disk spills.

---

### Q99: Which part of the pipeline were you handling solely?

**Defense Strategy:**
"I had sole end-to-end engineering ownership of the **Streaming Ingestion & Silver Standardization Engine**:
- Designing the Kafka consumer consumer group offsets and Spark Structured Streaming micro-batch pipelines.
- Authoring the automated deduplication logic, SCD Type 2 dimension merge jobs, and dead-letter queue routing for malformed payloads.
- Configuring CI/CD deployment pipelines using **Databricks Asset Bundles (DABs)** with automated unit tests (`pytest`) and data validation suites."

---

### Q100: Can you redesign your project using the latest Databricks tools?

**Modern Architecture Redesign (Databricks 2026 Standards):**
1. **Delta Live Tables $\rightarrow$ Lakeflow Pipelines:** Replace legacy cron notebooks with Lakeflow Declarative Pipelines for automated DAG lineage, incremental materialized views, and declarative SQL/Python.
2. **Hive Metastore $\rightarrow$ Unity Catalog 3-Tier Hierarchy:** Migrate tables into `catalog.schema.table`, enabling cross-workspace data sharing (Delta Sharing), dynamic row filtering, and column masking without duplication.
3. **Partitioning & Z-Order $\rightarrow$ Liquid Clustering:** Replace rigid date-based directory partitioning with `CLUSTER BY (merchant_id, order_date)`, enabling multi-dimensional data skipping without file rewrite overhead.
4. **All-Purpose Compute $\rightarrow$ Serverless Compute & DBSQL:** Run scheduled workflows on Serverless compute, eliminating cluster startup latency ($< 5\text{s}$ start vs $5\text{min}$) and cutting idle cost to zero.

---

### Q101: Have you worked on an Azure cloud project in the past? Explain your role.

**Key Technical Highlights to State:**
- Configured **Azure Virtual Networks (VNet Injection)** and Private Endpoints for secure, non-public data transit between ADLS Gen2, Azure Key Vault, and Databricks clusters.
- Configured **Azure Managed Identities** for passwordless token authentication into Azure resources.
- Integrated **Azure Data Factory (ADF)** as the enterprise orchestrator triggering Databricks job runs and sending incident alerts to Azure Monitor and Microsoft Teams.

---

### Q102: What was the design layout of your tables?

**Kimball Star Schema on Delta Lake:**
- **Fact Tables:**
  - `fact_orders` (Grain: 1 row per order line item; clustered by `order_date, merchant_id`).
  - `fact_daily_inventory_snapshot` (Grain: 1 row per product per store per day).
- **Dimension Tables:**
  - `dim_customer` (SCD Type 2: historical tracking with `valid_from`, `valid_to`, `is_current`).
  - `dim_product` (SCD Type 1: latest master data; clustered by `product_category`).
  - `dim_date` (Standard enterprise calendar dimension).

---

### Q103: How many tasks, stages, and partitions were created in your last production project?

**Concrete Production Answer:**
- **Baseline Stage Profile:** An incremental daily batch run of 250 GB compressed Parquet generated approximately:
  - **Input Stage:** ~2,000 tasks (with `maxPartitionBytes` set to 128MB: $\frac{250\text{GB}}{128\text{MB}} \approx 2000\text{ tasks}$).
  - **Shuffle Stage (Joins & Aggregations):** AQE dynamically coalesced partitions from an initial 1,000 down to **384 shuffle partitions** to perfectly match our 96 cluster cores ($4\times \text{ ratio}$).
  - **Total Stages:** 4 to 6 stages per job depending on wide dependencies.

---

### Q104: Did you face any sudden job delay in your previous project? How did you debug it?

**Debugging War Story:**
- **Symptom:** A nightly 30-minute ETL pipeline suddenly took 3 hours to finish without any code changes.
- **Investigation:**
  1. *Cluster Telemetry:* CPU utilization dropped to 5% across 95% of workers while 1 executor CPU pinned at 100%.
  2. *Spark UI:* Discovered a single task in Stage 3 processing 18 GB while all other 383 tasks processed 50 MB.
  3. *Root Cause:* Upstream source system began sending records with `NULL` or `'UNKNOWN'` as the customer key. All null values hashed to the identical partition during the join shuffle.
- **Remediation:** Isolated null keys prior to the join using a conditional branch, applied the join only to non-null keys, and re-unioned null records with default dimension values. Runtime restored to 28 minutes.

---

### Q105: Which all Integrations have you implemented till now?

- **Message Queues & Streams:** Apache Kafka, Azure Event Hubs, AWS Kinesis.
- **Databases & CDC:** PostgreSQL, SQL Server (via Debezium and Azure Data Factory CDC), MongoDB, Snowflake.
- **File Systems & Formats:** Parquet, Delta Lake, Apache Iceberg, Avro, ORC, CSV, JSON, XML.
- **REST APIs & SaaS:** Salesforce CRM API, Stripe Billing Webhooks, Google Analytics export via cloud storage buckets.

---

### Q106: Which optimization techniques did you use in your last project?

1. **Delta Liquid Clustering & Data Skipping:** Reduced scanned data volume by 78% for ad-hoc analytical queries.
2. **Vectorized Query Execution (Photon):** Reduced CPU transformation runtime on heavy string parsing and hashing by 65%.
3. **Adaptive Query Execution (AQE):** Dynamically coalesced small partitions and auto-split skewed join keys.
4. **Broadcast Hash Joins:** Forced broadcast for lookup tables $< 200\text{MB}$.
5. **Deletion Vectors:** Enabled non-destructive atomic updates in Delta Lake, eliminating file rewrite amplification during daily deletes.

---

### Q107: How did you load huge CSV files in any of your previous projects?

**Production Strategy:**
- **Problem with CSV:** CSV files are row-oriented, uncompressed or gzip-compressed (non-splittable), requiring a single core to read the entire multi-gigabyte file.
- **Solution:**
  1. If compressed, enforce splittable compression formats like **Bzip2** or ingest raw chunks into storage.
  2. In Databricks, use **Auto Loader** (`cloudFiles`) with explicit schema definition:
     ```python
     df = (
         spark.readStream
         .format("cloudFiles")
         .option("cloudFiles.format", "csv")
         .option("cloudFiles.maxFilesPerTrigger", 50)
         .schema(defined_schema)
         .load(landing_path)
     )
     ```
  3. Immediately convert and save into **Delta Lake** in the Bronze layer; never query or perform complex transformations directly on raw CSVs.

---

### Q108: Which type of errors have you come across in your last project?

1. **`OutOfMemoryError: Java heap space`:** Caused by unbounded joins or unbroadcastable lookups.
2. **`Container killed by YARN for exceeding memory limits (Exit 137)`:** Caused by off-heap PyArrow memory allocation exceeding `memoryOverhead`.
3. **`FetchFailedException`:** Caused by network timeouts or executor crashes during shuffle block retrieval.
4. **`ConcurrentAppendException` / `ConcurrentModificationException`:** Concurrency collision in Delta Lake when two pipelines modified identical partitions. Resolved using partition-isolated write predicates.
5. **Schema Drift Exceptions:** Upstream field additions. Resolved by configuring schema evolution (`mergeSchema = true`).

---

### Q109: If you identify a 5X increase in data volume, what will be your steps to scale?

**Scalability Action Plan:**
1. **Storage Layout:** Validate partition and clustering granularity. Increase target partition size to maintain 128MB–256MB file sizes; convert legacy partition keys to **Liquid Clustering**.
2. **Compute Scaling:** Configure Databricks cluster autoscaling from 4–16 nodes to 8–64 nodes; switch to Memory-Optimized VM families (`Standard_E16ds_v5`).
3. **Shuffle Tuning:** Increase `spark.sql.shuffle.partitions` or set `spark.sql.adaptive.advisoryPartitionSizeInBytes = 134217728` (128MB).
4. **Micro-Batching:** Shift from monolithic daily batch runs to 15-minute or continuous streaming micro-batches, leveling compute spikes across the 24-hour cycle.
5. **FinOps Safeguards:** Enforce cluster auto-termination (15 mins), spot instance policies for workers, and budget threshold alerts.

---

### Q110: Did you come across duplicate records in your previous project? How do you fix this issue?

**Root Cause & Remediation:**
- **Root Cause:** Upstream network timeouts in HTTP producers triggering retries without idempotency keys; duplicate files landing in storage from overlapping SFTP cron jobs.
- **Remediation Workflow:**
  1. *Auto Loader File Deduplication:* Auto Loader inherently maintains a rocksdb-backed ledger of ingested file names to prevent reading the same file twice.
  2. *In-Batch Deduplication:* Apply `df.dropDuplicates(["business_key", "timestamp"])`.
  3. *Downstream ACID Upsert:* Use Delta `MERGE INTO` with a condition `WHEN MATCHED AND source.updated_at > target.updated_at THEN UPDATE SET *`.
