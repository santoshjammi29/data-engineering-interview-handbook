# Pillar 04: Data Lake Architecture & Object Storage
**Category:** `DATALAKE ARCHITECTURE` | **Domain:** `Cloud Storage & Data Lake Infrastructure`

---

### Q1: How do modern cloud object storage consistency models (S3, ADLS Gen2, GCS) affect data lake engineering?

**Difficulty:** `HARD` | **Niche:** `Consistency Models`

Both AWS S3 (since Dec 2020) and Azure ADLS Gen2 provide **strong read-after-write consistency** for all `PUT` and `DELETE` requests of objects.
- **Impact:** Eliminates the need for external consistency tools (like S3Guard/DynamoDB). Once a Spark executor writes a Parquet file to S3, a subsequent `LIST` or `GET` from any other node is guaranteed to observe the latest data.
- **Gotcha:** Hierarchical directory operations in S3 are still simulated object-key prefixes. In contrast, ADLS Gen2 implements a true POSIX-compliant **hierarchical namespace (HNS)**, allowing atomic directory renames and deletes ($O(1)$ operations instead of copying millions of objects).

---

### Q2: Explain Amazon S3 prefix request rate limits and how you architect high-concurrency data lake ingestion to avoid HTTP 503 Slow Down errors.

**Difficulty:** `HARD` | **Niche:** `Object Storage Request Limits`

Amazon S3 throttles requests per partition prefix at **3,500 PUT/POST/DELETE** and **5,500 GET/HEAD requests per second**.
- **The Bottleneck:** Writing 5,000 files concurrently to a single date folder (`s3://lake/orders/2026-11-18/`) will exhaust prefix limits and trigger HTTP 503 Slow Down.
- **Architectural Solution:** Distribute writes across multiple unique prefixes using hash pre-pending (e.g. `s3://lake/orders/<hash_prefix>/2026-11-18/`) or leverage Delta Lake / Iceberg, where metadata files isolate writes while underlying Parquet files can be distributed across multi-prefix namespaces automatically.

---

### Q3: Compare the Azure Blob File System (ABFS) driver with the Hadoop S3A connector for Spark data access.

**Difficulty:** `HARD` | **Niche:** `Cloud Storage Connectors`

**Comparison:**
- **ABFS (`abfss://`):** Azure's purpose-built cloud storage driver optimized for ADLS Gen2 hierarchical namespace. Implements atomic file operations, server-side directory renames, multi-threaded read-ahead caching, and native OAuth 2.0 Azure Managed Identity integration.
- **S3A (`s3a://`):** Open-source Apache Hadoop connector for AWS S3. Uses multipart upload commits (Magic Committer / S3A Committer) to simulate atomic job directory commitments on top of S3's non-hierarchical object model.

---

### Q4: How do you architect an Active-Passive Disaster Recovery (DR) lakehouse with RPO < 15 minutes and RTO < 1 hour?

**Difficulty:** `ARCHITECT` | **Niche:** `Disaster Recovery & Business Continuity`

**Production DR Architecture:**
1. **Underlying Storage Replication:** Configure asynchronous cloud object replication (e.g. Azure GRS / AWS S3 Cross-Region Replication - CRR) replicating raw and Delta tables from Primary to Secondary region ($< 15\text{ min}$ RPO).
2. **Metastore Sync:** Replicate Unity Catalog / AWS Glue Data Catalog metadata using automated infrastructure-as-code or catalog sync jobs.
3. **Compute Failover:** Maintain a warm standby compute cluster definition in the DR region (or Databricks Serverless pool) deployed via Terraform / DABs.
4. **DNS Cutover:** Route BI and pipeline triggers to the secondary region using Traffic Manager / Route 53 health-check failover.

---

### Q5: Design a zero-trust security perimeter for an enterprise data lake on cloud object storage.

**Difficulty:** `ARCHITECT` | **Niche:** `Zero-Trust Data Lake Security`

**Security Perimeter Blueprint:**
1. **Network Isolation:** Disable public internet endpoints. All access routed through **Private Endpoints (VNet / VPC)** and private DNS zones.
2. **Identity & Access Management:** Eliminate long-lived access keys and connection strings; authenticate exclusively via **Managed Identities** or IAM Roles with short-lived STS tokens.
3. **Encryption:** Enforce Customer-Managed Encryption Keys (CMEK) via Key Vault / AWS KMS with automated key rotation. Enforce TLS 1.3 for data in transit.
4. **Data Governance & Masking:** Use Unity Catalog or AWS Lake Formation for centralized column masking, row-level security, and access audits.

---

### Q6: Why is the small file problem lethal to cloud object stores, and how do you design an automated compaction service?

**Difficulty:** `MEDIUM` | **Niche:** `Storage Compaction & Maintenance`

Millions of small files ($< 10\text{MB}$) explode cloud storage metadata costs (billing per 1,000 HTTP GET/LIST operations) and cripple query planning because Spark must scan thousands of file footers.
**Automated Compaction Service:**
Run a daily daemon job that queries table metadata, identifies partitions with file counts $> 100$ and average size $< 64\text{MB}$, and executes bin-packing:
```sql
OPTIMIZE target_table WHERE partition_date = CURRENT_DATE() - INTERVAL 1 DAY;
```

---

### Q7: What is Metadata Catalog Federation, and how does it enable querying across disparate clouds and data warehouses?

**Difficulty:** `HARD` | **Niche:** `Catalog Federation`

Catalog Federation allows a central query engine to query external catalogs (e.g. Unity Catalog querying Snowflake, AWS Glue, Google BigQuery, or Salesforce) without moving physical data into the lakehouse.
- The engine acts as a federated coordinator, pushing predicates and projections down to the remote database engine.
- Ideal for hybrid architectures where operational CRM databases must be joined with lakehouse telemetry.

---

### Q8: How do you implement storage lifecycle tiering policies for a 5 PB data lake to reduce storage costs by 60%?

**Difficulty:** `HARD` | **Niche:** `Storage FinOps`

**Storage Tiering Architecture:**
1. **Hot Tier (Standard S3 / ADLS Hot):** Retain data for 0 to 30 days. High query frequency.
2. **Cool Tier (Infrequent Access - IA):** Transition data aged 31 to 90 days. 50% cheaper storage; slight retrieval cost per GB.
3. **Archive Tier (Glacier Instant Retrieval / ADLS Cold):** Transition data aged 91 to 365 days. 70% cheaper storage.
4. **Deep Archive / Expiration:** Move audit logs to Glacier Deep Archive after 1 year; delete temporary staging/scratch folders after 7 days.

---

### Q9: Describe the 3 most dangerous data lake partitioning anti-patterns and their remediation.

**Difficulty:** `MEDIUM` | **Niche:** `Partitioning Anti-patterns`

**3 Dangerous Anti-patterns:**
1. **Over-Partitioning by High-Cardinality Keys:** Partitioning by `user_id` or `timestamp` creates millions of directories containing 1-row files, causing metastore exhaustion. *Fix:* Partition only by coarse date (`year/month`) or use Liquid Clustering.
2. **Uneven Partition Skew:** Partitioning by `country` where 90% of data is in 'US'. *Fix:* Cluster or bucket within the US partition.
3. **Changing Partition Keys Mid-Lifecycle:** Requires rewriting historical data folders. *Fix:* Adopt Apache Iceberg Hidden Partitioning or Delta Liquid Clustering.

---

### Q10: What are the trade-offs of POSIX-compliant file systems vs cloud object storage for big data analytics?

**Difficulty:** `HARD` | **Niche:** `File Systems vs Object Stores`

**Comparison:**
- **POSIX File Systems (HDFS, Lustre, NFS):** Supports byte-range random writes, atomic directory renames, and low-latency file system locks. However, tightly couples compute with storage, is expensive to scale to petabytes, and requires dedicated operational management.
- **Cloud Object Storage (S3, ADLS Gen2, GCS):** Infinite elasticity, 99.999999999% durability, and virtually zero operational maintenance. However, objects are immutable (no in-place random writes), directory renames in S3 are simulated through copy-and-delete, and HTTP network roundtrips require large batch reads.

---

### Q11: Why do massive Spark jobs hit HTTP 503 Slow Down errors on Amazon S3, and how do you remediate prefix throttling?

**Difficulty:** `HARD` | **Niche:** `Cloud Object Storage & Consistency`

**Root Cause:** Amazon S3 partitions storage keys into internal partition prefixes. Each prefix supports **3,500 PUT/POST/DELETE** and **5,500 GET** requests per second. When 1,000 Spark executor cores simultaneously write to `s3://bucket/table/year=2026/month=11/part-*.parquet`, all requests target a single prefix, triggering HTTP 503.
**Remediation:**
1. Use hash-prefixed directory keys or S3 Express One Zone.
2. Adopt Delta Lake or Iceberg with modern direct committers (`magic committer` or S3A committer) to eliminate rename operations (`mv` calls that trigger thousands of copy/delete calls).
3. Increase S3A retry limits: `fs.s3a.retry.limit=20` and `fs.s3a.retry.interval=500ms`.

---

### Q12: What is the optimal Parquet file size in cloud object storage, and how do you orchestrate automated bin-packing?

**Difficulty:** `MEDIUM` | **Niche:** `Storage Optimization & Layout`

**Optimal Target:** **128 MB to 1 GB** per file. Smaller files (< 10MB) incur excessive HTTP metadata listing overhead and explode metastore memory. Massive files (> 2GB) degrade split parallelism and increase OOM risk.
**Compaction Strategies:**
1. **Auto-Compaction (Databricks):** Automatically packs micro-batches during write: `spark.databricks.delta.autoCompact.enabled = true`.
2. **Scheduled Offline Compaction:** Nightly maintenance DAG runs `OPTIMIZE table_name` targeting uniform 128MB files.
3. **Vacuum Hygiene:** `VACUUM table_name RETAIN 168 HOURS` to delete tombstoned Parquet files after 7 days, reclaiming storage costs.

---

### Q13: How do you design an enterprise Lakehouse Disaster Recovery (DR) architecture satisfying RPO < 15 mins and RTO < 1 hour?

**Difficulty:** `ARCHITECT` | **Niche:** `Disaster Recovery & High Availability`

**Architecture Blueprint: Active-Passive Warm Standby**
1. **Data Replication (RPO < 15m):** Enable cloud-native asynchronous cross-region replication (e.g. AWS S3 Cross-Region Replication or Azure GRS/CRR). For transaction logs, stream Delta Change Data Feed (CDF) or run incremental `DEEP CLONE` every 15 minutes.
2. **Metastore Sync:** Replicate Unity Catalog / AWS Glue Catalog metadata across regions using automated event-driven AWS Lambda / Azure Functions.
3. **Compute Failover (RTO < 1h):** Standby region maintains 0 active compute nodes. Databricks Asset Bundles (DABs) or Terraform scripts pre-define cluster templates and DAGs in the DR region.
4. **Failover Execution:** DNS routing (Route53/Traffic Manager) shifts traffic, Terraform scales standby cluster, and pipelines resume from the last replicated Delta log checkpoint.

---

### Q14: What are the pitfalls of transitioning active Delta Lake or Iceberg tables to S3 Glacier or ADLS Cold tiers?

**Difficulty:** `HARD` | **Niche:** `Storage Optimization & Layout`

**The Glacier / Deep Archive Pitfall:** S3 Glacier Flexible and Deep Archive objects are not immediately accessible; reading an object requires asynchronous retrieval taking minutes to hours. If a query runs against a Delta table where historical Parquet files were moved to Glacier, the query immediately crashes with `FileNotFoundException` or `403 Forbidden`.
**Safe Tiering Strategy:**
- Use **S3 Intelligent-Tiering** or **Glacier Instant Retrieval**, which offers millisecond access latency identical to S3 Standard.
- For compliance archives (> 3 years), export snapshots to dedicated unmanaged archive tables and delete them from active queryable lakehouse catalogs before applying Glacier Deep Archive rules.

---

### Q15: How do you implement a Zero-Trust network perimeter for a Cloud Lakehouse on AWS/Azure?

**Difficulty:** `ARCHITECT` | **Niche:** `Data Lake Security Architecture`

1. **Storage Perimeter:** Disable all public Internet access on storage buckets. Enforce storage bucket policies requiring access via VPC Endpoints (AWS PrivateLink) or Azure Private Endpoints.
2. **Customer-Managed Encryption Keys (CMK):** Encrypt all data at rest using cloud HSM keys (AWS KMS / Azure Key Vault) with annual key rotation.
3. **Identity Federation:** Ban shared access keys/SAS tokens. Enforce short-lived IAM assume-role policies mapped directly to Unity Catalog or IAM identity providers.
4. **Control Plane Isolation:** Front all BI and developer access through secure bastion hosts or corporate VPNs using private workspace connections.

---
