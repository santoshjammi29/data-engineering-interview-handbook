> 🧭 **Modern Data Engineering Ecosystem Navigation**
> - 📘 **Architecture & Concept:** [Data System Design — Lakehouse Architecture](https://github.com/santoshjammi29/data-system-design-handbook/tree/main/modules/01_lakehouse_and_medallion_architecture)
> - 💻 **Hands-On Recipe:** [Databricks Handbook — 105 Production Recipes](https://github.com/santoshjammi29/databricks-handbook)
> - 🔬 **Empirical Lab:** [Spark Experiments — Empirical Laboratory](https://github.com/santoshjammi29/spark-experiments)
> - 🚨 **Incident Runbook:** [PySpark Performance & Triage Runbooks](https://github.com/santoshjammi29/pyspark-handbook/tree/main/runbooks)
> - 🎯 **Interview Defense:** [Interview Handbook Master Guide](https://github.com/santoshjammi29/data-engineering-interview-handbook/blob/main/pillars/all_20_pillars_master_guide.md) *(You are here)*
> - 🌐 **Interactive Web Practice:** [Practice on moderndataengineer.uk](https://moderndataengineer.uk/practice)

# Pillar 08: Microsoft Fabric & Unified Analytics
**Category:** `FABRIC` | **Domain:** `Unified SaaS Analytics & Microsoft Fabric`

---

### Q1: Explain OneLake architecture in Microsoft Fabric and the concept of 'OneDrive for Data'.

**Difficulty:** `HARD` | **Niche:** `OneLake Architecture`

OneLake is a single, unified, logical data lake for the entire organization built on top of Azure Data Lake Storage Gen2 (ADLS Gen2):
- **Multi-Tenant Logical Store:** Every Microsoft Fabric tenant has exactly one OneLake, eliminating disparate storage silos across departments.
- **Open Format (Delta Lake):** All Fabric engines (Synapse Data Engineering, Synapse Data Warehouse, Real-Time Analytics, and Power BI) read and write to OneLake in open Delta-Parquet format.
- **Workspaces as Folders:** Fabric workspaces manifest as hierarchical folders inside OneLake, governed by unified Microsoft Entra ID permissions.

---

### Q2: What is Direct Lake mode in Power BI, and how does it outperform both Import mode and DirectQuery?

**Difficulty:** `HARD` | **Niche:** `Direct Lake Mode`

- **Import Mode:** Loads data into Power BI's in-memory VertiPaq engine. Ultra-fast, but requires duplicate data storage and hours of scheduled batch refreshes.
- **DirectQuery:** Translates user clicks into live SQL queries sent to the database. Zero data duplication, but slow query latency on complex dashboards.
- **Direct Lake Mode (Fabric Standard):** The VertiPaq engine loads Delta-Parquet files directly from OneLake storage into memory on-demand **without executing SQL translation or importing duplicates**. Delivers the speed of Import mode with the real-time freshness of DirectQuery.

---

### Q3: How do OneLake Shortcuts eliminate data movement across ADLS Gen2, AWS S3, and Google Cloud Storage?

**Difficulty:** `HARD` | **Niche:** `OneLake Shortcuts`

Shortcuts are embedded symbolic metadata links that point directly to data stored in external storage accounts (ADLS Gen2, Amazon S3, Google Cloud Storage, or Dataverse):
- **Zero ETL:** Data remains in the source cloud object store without copying or egress.
- **Unified Serving:** Fabric compute engines see the external bucket as a native local OneLake folder, querying Parquet/Delta files in place.

---

### Q4: Compare Fabric Lakehouse vs Fabric Warehouse: engines, transaction models, and developer personas.

**Difficulty:** `MEDIUM` | **Niche:** `Lakehouse vs Warehouse`

**Comparison:**
- **Fabric Lakehouse:** Spark-centric. Supports Python, Scala, R, and SQL. Code-first developer persona. Read/Write Delta Lake format with folder/table navigation.
- **Fabric Warehouse:** T-SQL enterprise engine. Supports ANSI SQL, full distributed cross-table transactions, primary/foreign key constraints, and dynamic data masking. Low-code / BI developer persona. Stores data natively in Delta-Parquet format.

---

### Q5: Explain Microsoft Fabric Capacity Smoothing and the 24-Hour Bursting algorithm for F-SKUs.

**Difficulty:** `HARD` | **Niche:** `Fabric FinOps & Capacity`

Fabric capacities (e.g. F64 = 64 Capacity Units - CUs) support **Capacity Smoothing**:
- **Bursting:** A short, compute-intensive Spark job can temporarily burst up to $4\times$ the purchased capacity limit.
- **Smoothing (24-Hour Window):** The spike in CU consumption is automatically smoothed and averaged over a rolling 24-hour evaluation window, preventing throttling during peak morning ETL batches as long as off-peak consumption compensates.

---

### Q6: How does Fabric Real-Time Intelligence process streaming data using Eventstreams and KQL Databases?

**Difficulty:** `HARD` | **Niche:** `Real-Time Streaming & KQL`

1. **Eventstreams:** Ingests live streaming events from Kafka, Event Hubs, or IoT Hubs with no-code data transformations and filtering.
2. **KQL Database (Kusto Engine):** Indexes streaming data automatically in memory using inverted indexes, enabling sub-second analytical queries across billions of log and telemetry records using Kusto Query Language (KQL).

---

### Q7: Compare Dataflows Gen2 in Fabric with traditional Azure Data Factory pipelines.

**Difficulty:** `MEDIUM` | **Niche:** `Dataflows Gen2`

**Comparison:**
- **Dataflows Gen2:** Powered by the Power Query M engine. Best for visual low-code ETL, rapid data profiling, and direct loading into OneLake Lakehouse/Warehouse staging.
- **ADF Pipelines:** Enterprise control flow orchestrator designed for scheduling, complex conditional logic, hybrid on-premises connectivity via SHIR, and calling external APIs and Databricks clusters.

---

### Q8: How is unified governance and sensitivity labeling enforced across OneLake using Microsoft Purview?

**Difficulty:** `HARD` | **Niche:** `Governance & Purview`

Microsoft Purview integrates natively with Fabric OneLake:
- **Sensitivity Labels:** Applied at the OneLake item level (e.g. 'Highly Confidential'); automatically flow downstream to Power BI reports, encrypting exports and enforcing DLP (Data Loss Prevention) policies.
- **Automated Lineage:** Tracks data provenance end-to-end from source shortcuts to Lakehouse Delta tables, Warehouse models, and Power BI semantic models.

---

### Q9: Explain Git integration and Deployment Pipelines in Microsoft Fabric for enterprise ALM.

**Difficulty:** `MEDIUM` | **Niche:** `Application Lifecycle Management`

**Production ALM Workflow:**
1. Connect Fabric workspace to an Azure DevOps / GitHub Git repository.
2. Workspaces serialize assets (notebooks, reports, lakehouses) into source-controlled JSON/YAML files.
3. **Fabric Deployment Pipelines:** Facilitate moving content across **Development $
ightarrow$ Test $
ightarrow$ Production** workspaces with automated stage comparison and selective deployment.

---

### Q10: What is the recommended migration strategy from Azure Synapse Analytics to Microsoft Fabric?

**Difficulty:** `ARCHITECT` | **Niche:** `Cloud Migration Strategy`

**Migration Strategy:**
1. **Storage Link:** Create OneLake Shortcuts pointing to existing Synapse ADLS Gen2 storage accounts (immediate access with zero data migration).
2. **SQL Scripts:** Migrate Synapse Dedicated SQL Pool T-SQL scripts to Fabric Warehouse, updating non-supported syntax.
3. **Spark Notebooks:** Transfer PySpark notebooks to Fabric Spark workspaces; Fabric runs modern Spark 3.5 runtimes with out-of-the-box Delta Lake 3.x support.

---

### Q11: How do OneLake Shortcuts unify data across AWS S3, ADLS Gen2, and Google Cloud Storage without physical movement?

**Difficulty:** `HARD` | **Niche:** `OneLake Storage & Shortcuts`

OneLake Shortcuts are virtual metadata pointers embedded inside the Fabric OneLake filesystem (`https://onelake.dfs.fabric.microsoft.com`).
- **Mechanism:** A shortcut references external cloud object storage (e.g. `s3://corp-lake/delta_tables/`) using cloud-native IAM credentials.
- **Zero-ETL:** No data is physically copied or synced into Fabric. The Fabric Spark engine, SQL endpoint, and Power BI query external Parquet files in place.
- **Unified Catalog:** External Delta/Parquet tables appear as native tables inside Fabric workspaces, unifying multi-cloud assets under a single governance boundary.

---

### Q12: What is the internal execution path of Power BI Direct Lake mode, and how does it achieve Import mode speeds without data refreshes?

**Difficulty:** `HARD` | **Niche:** `Power BI Direct Lake Mode`

**Import Mode vs DirectQuery:** Import mode loads data into in-memory VertiPaq columns (fast, but requires hours of scheduled refreshes). DirectQuery sends live SQL queries to the database (slow, high compute cost).
**Direct Lake Mode:**
- Bypasses the SQL translation and network serialization layer completely.
- Power BI's Analysis Services VertiPaq engine reads the Parquet column chunks directly from OneLake into in-memory memory-mapped files.
- Because Delta Lake Parquet is already columnar, VertiPaq directly maps column pages into memory without transcoding.
- Yields instant in-memory reporting performance over multi-billion row tables with 0 minutes of data refresh latency.

---

### Q13: What is the architectural distinction between a Fabric Lakehouse and a Fabric Data Warehouse?

**Difficulty:** `MEDIUM` | **Niche:** `Engine Comparison & Workloads`

**1. Fabric Lakehouse:**
- Primary engine is **Apache Spark** (Python/Scala/R/SQL).
- Supports unmanaged and managed Delta tables, unstructured files (images, audio), and sub-directories.
- Optimized for data engineers and data scientists doing ETL, ML, and ad-hoc analytics.
**2. Fabric Data Warehouse:**
- Primary engine is the distributed **T-SQL Engine** (derived from SQL Server/Synapse MPP).
- Supports full ACID transactions, cross-database queries, primary/foreign key constraints, and dynamic data masking.
- Both write Delta Parquet to OneLake, enabling Spark to read Warehouse tables and T-SQL to query Lakehouse tables seamlessly.

---

### Q14: How does Fabric Capacity Smoothing handle heavy computational spikes, and what happens during throttling?

**Difficulty:** `HARD` | **Niche:** `Capacity Units & Governance`

Fabric compute is measured in Capacity Units (CUs) tied to F-SKUs (e.g. F64 = 64 CUs).
**Capacity Smoothing:**
- Evaluates compute consumption in a 24-hour rolling window for background jobs (Spark, Data Factory) and a 10-minute window for interactive queries (Power BI, SQL).
- Spikes exceeding capacity are 'smoothed' over subsequent hours, allowing large Spark jobs to consume $400\%$ of nominal capacity without failing.
**Throttling Stages:**
1. *Interactive Delay:* Queries experience artificial latency.
2. *Interactive Rejection:* Power BI reports throw capacity overload errors.
3. *Background Rejection:* Scheduled pipelines fail to trigger until rolling consumption drops below 100%.

---

### Q15: When should you use a Fabric KQL Database (Eventhouse) instead of a Spark Structured Streaming Lakehouse?

**Difficulty:** `MEDIUM` | **Niche:** `Engine Comparison & Workloads`

**Use Fabric KQL (Azure Data Explorer engine):**
- For high-frequency log, telemetry, IoT, and cybersecurity event streams requiring sub-second query latency.
- Ingestion to query availability is < 1 second; indexes all text columns automatically with dynamic inverted index.
- Queries use Kusto Query Language (KQL), which is 10x faster and more concise than SQL for timeseries filtering and regex.
**Use Spark Lakehouse:** For complex multi-table dimensional joins, relational rollups, and machine learning model training.

---
