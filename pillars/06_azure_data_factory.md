> 🧭 **Modern Data Engineering Ecosystem Navigation**
> - 📘 **Architecture & Concept:** [Data System Design — Lakehouse Architecture](https://github.com/santoshjammi29/data-system-design-handbook/tree/main/modules/01_lakehouse_and_medallion_architecture)
> - 💻 **Hands-On Recipe:** [Databricks Handbook — 105 Production Recipes](https://github.com/santoshjammi29/databricks-handbook)
> - 🔬 **Empirical Lab:** [Spark Experiments — Empirical Laboratory](https://github.com/santoshjammi29/spark-experiments)
> - 🚨 **Incident Runbook:** [PySpark Performance & Triage Runbooks](https://github.com/santoshjammi29/pyspark-handbook/tree/main/runbooks)
> - 🎯 **Interview Defense:** [Interview Handbook Master Guide](https://github.com/santoshjammi29/data-engineering-interview-handbook/blob/main/pillars/all_20_pillars_master_guide.md) *(You are here)*
> - 🌐 **Interactive Web Practice:** [Practice on moderndataengineer.uk](https://moderndataengineer.uk/practice)

# Pillar 06: Azure Data Factory (ADF) & Integration Runtimes
**Category:** `ADF` | **Domain:** `Cloud ETL & Orchestration`

---

### Q1: How do you architect a high-availability Self-Hosted Integration Runtime (SHIR) cluster with zero open inbound ports?

**Difficulty:** `HARD` | **Niche:** `SHIR Architecture & Security`

- **Zero Inbound Ports:** The SHIR agent installed on an on-premises VM or Azure IaaS VM initiates only **outbound connections** over HTTPS port 443 to the Azure Relay service and ADF control plane.
- **High Availability (HA) Clustering:** Deploy up to **4 SHIR nodes** in a cluster with automatic failover and load balancing. The primary node manages dispatcher tasks, while worker nodes process concurrent pipeline jobs.
- **Private Link Integration:** Route all outbound communication through **Azure Private Endpoints** (`portal` and `gateway` sub-resources), completely avoiding the public internet.

---

### Q2: When should you use ADF Mapping Data Flows versus Azure Databricks for transformation workloads?

**Difficulty:** `MEDIUM` | **Niche:** `ADF vs Databricks Selection`

**Decision Framework:**
- **ADF Mapping Data Flows:** Best for citizen developers and teams requiring no-code/low-code visual transformations without managing Python/Scala codebases. Ideal for standard SCD merges, lookups, and aggregations.
- **Azure Databricks:** Mandatory for heavy enterprise transformation ($> 1\text{TB}$), complex machine learning inference, sub-second streaming, and advanced custom algorithms. Offers significantly faster runtimes via Photon and lower DBU execution costs.

---

### Q3: Design an incremental watermark-based ingestion pipeline in ADF for an on-premises SQL Server database.

**Difficulty:** `MEDIUM` | **Niche:** `Incremental Ingestion Patterns`

**Pipeline Design:**
1. **Lookup Activity (Old Watermark):** Read the last processed timestamp from a `watermark_control_table`.
2. **Lookup Activity (New High-Watermark):** Query `SELECT MAX(LastModifiedDate) FROM source_table`.
3. **Copy Activity (Delta Extraction):** Query source with parameterized dynamic SQL: `SELECT * FROM source_table WHERE LastModifiedDate > '@{activity('OldWatermark').output.firstRow.val}' AND LastModifiedDate <= '@{activity('NewWatermark').output.firstRow.val}'`.
4. **Stored Procedure Activity (Update Watermark):** Update `watermark_control_table` with the new timestamp upon successful copy completion.

---

### Q4: How do you handle REST API pagination and dynamic bearer token refresh inside ADF pipelines?

**Difficulty:** `HARD` | **Niche:** `API Ingestion & Web Activities`

**Production Architecture:**
1. **Web Activity (OAuth Authentication):** POST client credentials to auth endpoint to retrieve a short-lived bearer token.
2. **Until Activity (Pagination Loop):**
   - Copy Activity queries the paginated REST API endpoint using the bearer token in headers.
   - Extract the `next_page_url` or cursor token using ADF expression language: `@activity('CopyPage').output.pagination.nextCursor`.
   - Loop terminates when `next_page_url` evaluates to empty or null.

---

### Q5: How do you achieve zero-secret pipeline authentication between ADF, Azure Key Vault, and ADLS Gen2?

**Difficulty:** `MEDIUM` | **Niche:** `Managed Identity Security`

1. Enable **System-Assigned Managed Identity (SMI)** on the Azure Data Factory instance.
2. In Azure Key Vault, grant ADF's Managed Identity the **Key Vault Secrets User** role.
3. In ADLS Gen2, grant ADF's Managed Identity the **Storage Blob Data Contributor** role.
4. Linked Services authenticate directly via Managed Identity without storing connection strings, client secrets, or passwords in ADF JSON definitions.

---

### Q6: Compare Event-Based Triggers and Tumbling Window Triggers in ADF.

**Difficulty:** `MEDIUM` | **Niche:** `Triggers & Scheduling`

**Comparison:**
- **Event-Based Triggers:** Fires in real-time in response to Azure Event Grid events (e.g. `Microsoft.Storage.BlobCreated`). Non-deterministic, reactive, and cannot execute historical backfills.
- **Tumbling Window Triggers:** Generates periodic, non-overlapping contiguous time intervals (e.g. every 1 hour). Guarantees ordered execution, supports automated dependency tracking across pipelines, and natively supports historical **Backfilling**.

---

### Q7: How do you configure centralized monitoring and alerting for ADF pipeline failures using Azure Monitor?

**Difficulty:** `MEDIUM` | **Niche:** `Observability & Alerting`

1. Configure **Diagnostic Settings** in ADF to stream `PipelineRuns`, `TriggerRuns`, and `ActivityRuns` logs to a central **Log Analytics Workspace**.
2. Author Kusto (KQL) alert queries detecting failed pipeline states:
```kusto
ADFPipelineRun | where Status == 'Failed' | project PipelineName, RunId, Start, End, FailureType
```
3. Bind the KQL query to an **Azure Monitor Action Group** triggering PagerDuty incidents and automated Slack webhooks.

---

### Q8: Explain the production CI/CD deployment lifecycle in ADF using Git integration and ARM/Bicep templates.

**Difficulty:** `HARD` | **Niche:** `CI/CD & Release Automation`

**Production GitOps Lifecycle:**
1. Developers author pipelines on personal feature branches in Azure DevOps / GitHub.
2. Merging to the `collaboration` branch triggers an automated build that validates JSON definitions.
3. ADF generates ARM/Bicep deployment templates into the `adf_publish` branch.
4. Release pipelines use the **Azure Data Factory Deployment Task** to deploy the ARM templates to Staging and Production, dynamically overriding linked service endpoints and Key Vault URLs per environment.

---

### Q9: How do you tune Data Integration Units (DIUs) in ADF Copy Activities to minimize cost while meeting SLAs?

**Difficulty:** `MEDIUM` | **Niche:** `ADF FinOps & DIU Tuning`

DIU (Data Integration Unit) represents compute capacity (combination of CPU, memory, and network resources) used by copy activities.
- **Over-Provisioning Trap:** Setting DIU to 64 or 128 for small datasets ($< 1\text{GB}$) costs significantly more without speeding up ingestion, as network setup overhead dominates.
- **Rule of Thumb:** Use Auto (4 DIUs) for small/medium files. For multi-gigabyte files, scale DIUs to 16 or 32 paired with **Degree of Copy Parallelism** (`parallelCopies = 16-32`) to achieve saturated cloud bandwidth.

---

### Q10: How do you implement exponential backoff retries and idempotent writes in ADF Copy Activities?

**Difficulty:** `MEDIUM` | **Niche:** `Fault Tolerance & Retries`

1. **Retry Policy:** On the Copy Activity, configure `Retry = 3` and `Retry interval = 300 seconds`.
2. **Idempotent Destination:** Configure the sink write behavior to execute an **Upsert** (e.g. Azure SQL merge stored procedure) or pre-copy script that cleans staging tables (`TRUNCATE TABLE staging_orders`), ensuring retried runs never generate duplicate records.

---

### Q11: How do you architect a High-Availability Self-Hosted Integration Runtime (SHIR) cluster in Azure Data Factory?

**Difficulty:** `HARD` | **Niche:** `Integration Runtime Architecture`

A standalone SHIR node represents a single point of failure (SPOF) for on-premises/VNet database extraction.
**HA Clustering Architecture:**
1. Install SHIR on up to 4 nodes in a single high-availability cluster.
2. Register all nodes to the same ADF authentication key.
3. **Load Balancing:** ADF automatically balances pipeline copy tasks across online nodes based on CPU/RAM utilization.
4. **Failover:** If the active dispatcher node crashes, another node in the cluster automatically assumes leadership in < 60 seconds with zero pipeline interruption.
5. Enable concurrent job limits (up to 56 concurrent copy activities per node) based on physical memory.

---

### Q12: How do you build a high-resilience incremental watermark extraction pipeline in ADF?

**Difficulty:** `MEDIUM` | **Niche:** `Incremental Ingestion Patterns`

1. **Watermark Control Table:** Store `TableName`, `LastProcessedTimestamp`, and `Status` in Azure SQL.
2. **ADF Pipeline Flow:**
   - Activity 1 (Lookup): Query high-watermark from source table (`SELECT MAX(UpdatedAt) FROM SourceTable`).
   - Activity 2 (Lookup): Fetch last watermark from control table.
   - Activity 3 (Copy Activity): Extract records using predicate: `WHERE UpdatedAt > LastProcessed AND UpdatedAt <= CurrentMax`.
   - Activity 4 (Stored Procedure): Update control table with `CurrentMax` inside a database transaction only after Copy Activity succeeds.

---

### Q13: What happens under the hood when executing an Azure Data Factory Mapping Data Flow?

**Difficulty:** `HARD` | **Niche:** `ADF Security & Enterprise Architecture`

ADF Mapping Data Flows translate visual GUI graph transformations into Scala/Spark code executed on managed Azure Databricks clusters behind the scenes.
- **Provisioning:** Uses an Azure Integration Runtime with compute type (General Purpose vs Memory Optimized) and core count (e.g. 16+16 cores).
- **Cold Start Remediation:** Enable **Quick Re-use / Time to Live (TTL)**: Keeps the underlying Spark cluster warm for 10-30 minutes after execution, reducing startup latency from 5 minutes to seconds for subsequent pipeline activities.

---

### Q14: How do you handle OAuth2 bearer token expiration and pagination in ADF REST Copy Activity?

**Difficulty:** `HARD` | **Niche:** `Incremental Ingestion Patterns`

1. **Token Refresh:** Front the pipeline with a Web Activity calling the OAuth token endpoint (`/oauth/token`) with Client ID and Secret stored in Azure Key Vault.
2. **Pagination:** Configure ADF Copy Activity RFC 5988 pagination rules (`AbsoluteUrl` or `QueryParameters['nextPage']`).
3. **Looping:** For APIs requiring dynamic session tokens, wrap Copy Activity in an `Until` loop: check `hasMore` response property, update the token dynamically, and append results to ADLS Gen2 Bronze landing zone.

---

### Q15: How do you automate ADF CI/CD deployments using ARM templates and Azure DevOps without exposing plain-text secrets?

**Difficulty:** `HARD` | **Niche:** `ADF Security & Enterprise Architecture`

1. **Git Integration:** Develop in a collaboration branch (`main`). Publishing exports generated ARM templates to `adf_publish` branch.
2. **Parameterization Template (`arm-template-parameters-definition.json`):** Parameterize Key Vault URLs, storage account endpoints, and linked service references.
3. **Azure DevOps Release Pipeline:** Deploy via ARM Template Deployment task; link secrets dynamically to Azure Key Vault secrets, ensuring zero hardcoded credentials exist in source control.

---
