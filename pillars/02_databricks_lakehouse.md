> 🧭 **Modern Data Engineering Ecosystem Navigation**
> - 📘 **Architecture & Concept:** [Data System Design — Lakehouse Architecture](https://github.com/santoshjammi29/data-system-design-handbook/tree/main/modules/01_lakehouse_and_medallion_architecture)
> - 💻 **Hands-On Recipe:** [Databricks Handbook — Notebook 02_01: Unity Catalog Governance](https://github.com/santoshjammi29/databricks-handbook/blob/main/notebooks/02_unity_catalog_and_governance/02_01_three_level_namespace_and_securables.py)
> - 🔬 **Empirical Lab:** [Databricks Blueprints — Omnichannel Retail Lakehouse](https://github.com/santoshjammi29/azure-databricks-project-blueprints/tree/main/src/project1_omnichannel_retail)
> - 🚨 **Incident Runbook:** [Databricks Runbook 01: Driver OOM Heap Space Triage](https://github.com/santoshjammi29/databricks-handbook/blob/main/runbooks/01_driver_oom_heap_space_triage.md)
> - 🎯 **Interview Defense:** [Interview Handbook Master Guide](https://github.com/santoshjammi29/data-engineering-interview-handbook/blob/main/pillars/all_20_pillars_master_guide.md) *(You are here)*
> - 🌐 **Interactive Web Practice:** [Practice Databricks on moderndataengineer.uk](https://moderndataengineer.uk/practice/databricks)

# Pillar 02: Databricks Lakehouse & Unity Catalog
**Category:** `DATABRICKS` | **Domain:** `Lakehouse Governance & Databricks Platform`

---

### Q1: Explain Unity Catalog's 3-tier namespace and how dynamic row filters and column masks are implemented.

**Difficulty:** `HARD` | **Niche:** `Unity Catalog Governance`

Unity Catalog organizes data in a 3-tier hierarchy: `catalog.schema.table`.
- **Centralized Governance:** Metastore is decoupled from individual workspaces; a single metastore spans multiple Azure regions, cloud accounts, and workspaces.
- **Dynamic Row Filtering & Column Masking:** Defined using native SQL UDFs without creating duplicate sanitized views:
```sql
-- Column Masking for PII
CREATE FUNCTION mask_ssn(ssn STRING) RETURN 
  CASE WHEN is_account_group_member('hr_admins') THEN ssn ELSE 'XXX-XX-' || RIGHT(ssn, 4) END;
ALTER TABLE hr_catalog.payroll.employees ALTER COLUMN ssn SET MASK mask_ssn;

-- Row Filtering for Region isolation
CREATE FUNCTION filter_region(country STRING) RETURN
  is_account_group_member('global_leads') OR country = current_user();
ALTER TABLE hr_catalog.payroll.employees SET ROW FILTER filter_region ON (country);
```

---

### Q2: What are Databricks Asset Bundles (DABs) and how do they establish GitOps CI/CD across Dev, Staging, and Prod?

**Difficulty:** `ARCHITECT` | **Niche:** `GitOps & Databricks Asset Bundles`

Databricks Asset Bundles (DABs) provide an Infrastructure-as-Code (IaC) declarative framework for defining, testing, and deploying Databricks assets (jobs, pipelines, notebooks, compute settings, and permissions) via YAML:
- **Configuration:** A single `databricks.yml` file parameterizes targets (`dev`, `staging`, `prod`) with environment-specific workspace URLs, service principals, and catalog names.
- **GitOps Workflow:** Developers work on feature branches; CI runners validate bundles with `databricks bundle validate`. Merging to `main` executes `databricks bundle deploy -t prod`, ensuring 100% parity across environments without manual UI intervention.

---

### Q3: Compare Databricks Serverless Compute with Classic Compute in terms of architecture, startup latency, and cost.

**Difficulty:** `HARD` | **Niche:** `Serverless Compute & FinOps`

**Comparison Matrix:**
- **Startup Latency:** Serverless starts in **$< 10\text{ seconds}$** (instant pool warmup); Classic compute requires **3 to 7 minutes** to provision cloud VMs.
- **Cluster Plane Location:** Serverless compute runs in an optimized, isolated compute plane managed by Databricks, communicating via secure Private Link; Classic compute runs in the customer's VNet.
- **Idle Cost & Auto-Stop:** Serverless automatically scales down to zero instantly when idle; Classic compute has a minimum idle timeout (typically 10-15 mins) that continues incurring VM costs.
- **Ideal Workloads:** Serverless is best for ad-hoc SQL, sporadic batch jobs, and workflow tasks; Classic compute is used when specialized custom VNet firewalls or non-standard root init scripts are strictly required.

---

### Q4: How does the Photon engine accelerate Databricks SQL queries compared to standard Spark SQL?

**Difficulty:** `HARD` | **Niche:** `Vectorized Photon Engine`

Photon is Databricks' vectorized query engine written entirely in **C++** from the ground up:
1. **Direct Hardware Vectorization (SIMD):** Photon processes columnar data in batches using Single Instruction, Multiple Data (SIMD) CPU registers, processing dozens of values per clock cycle.
2. **Eliminates JVM Heap & JIT Limits:** Bypasses Java garbage collection and the JVM runtime completely; avoids the method size limitations of Spark Janino Whole-Stage CodeGen.
3. **Optimal for String & Decimal Computations:** Outperforms standard Spark by **$3\times \text{ to } 8\times$** on heavy string parsing, regex, aggregation hash joins, and decimal arithmetic.

---

### Q5: Explain Delta Live Tables (DLT) / Lakeflow Pipelines and how data quality expectations are enforced.

**Difficulty:** `HARD` | **Niche:** `Lakeflow Pipelines & Data Quality`

Lakeflow / DLT is a declarative framework where data engineers define tables using `@dlt.table` annotations, and Databricks manages the underlying DAG execution, batch/stream transitions, and cluster sizing.
**Enforcing Data Quality via Expectations:**
```python
import dlt
from pyspark.sql import functions as F

@dlt.table(
    name='silver_clean_orders',
    comment='Orders with validated totals'
)
# 1. Retain row, log metric
@dlt.expect('valid_order_id', 'order_id IS NOT NULL')
# 2. Drop corrupted row silently
@dlt.expect_or_drop('valid_amount', 'amount > 0')
# 3. Fail entire pipeline on critical invariant violation
@dlt.expect_or_fail('valid_currency', 'currency IN ("USD", "EUR", "GBP")')
def silver_clean_orders():
    return dlt.read_stream('bronze_raw_orders')
```

---

### Q6: What is Delta Lake Liquid Clustering (`CLUSTER BY`), and why does it replace legacy Hive partitioning and Z-Order?

**Difficulty:** `ARCHITECT` | **Niche:** `Liquid Clustering Architecture`

Traditional Hive partitioning suffers from rigid hierarchical structures (e.g. `/year/month/day/`), and Z-Ordering requires expensive, full table rewrites during compaction.
**How Liquid Clustering Works:**
1. **Multi-Dimensional Space-Filling Curve:** Clusters data along specified columns (e.g. `CLUSTER BY (merchant_id, event_date)`) without hierarchical constraints.
2. **Incremental Clustering:** Unlike Z-Order, Liquid Clustering clusters data incrementally during write and compaction operations, reducing compute overhead by up to 70%.
3. **Flexibility:** Allows redefining cluster keys via `ALTER TABLE ... CLUSTER BY` without rewriting historical data.

---

### Q7: How do you enable and query Delta Change Data Feed (CDF) for downstream incremental consumers?

**Difficulty:** `HARD` | **Niche:** `Delta Change Data Feed`

Enable CDF on the Delta table:
```sql
ALTER TABLE silver_orders SET TBLPROPERTIES (delta.enableChangeDataFeed = true);
```
**Querying Changes across Commit Versions:**
```python
# Read incremental changes between version 10 and 20
df_changes = (spark.read.format('delta')
    .option('readChangeFeed', 'true')
    .option('startingVersion', 10)
    .option('endingVersion', 20)
    .table('silver_orders'))
# Returns metadata columns: _change_type ('insert', 'update_preimage', 'update_postimage', 'delete')
```

---

### Q8: Explain Azure Databricks VNet Injection and how Private Link protects data in transit.

**Difficulty:** `ARCHITECT` | **Niche:** `Network & Cloud Security`

- **VNet Injection:** Databricks deploys compute clusters into the customer's own Virtual Network across two dedicated subnets (`public-subnet` for host control, `private-subnet` for worker communication).
- **Front-End Private Link:** Secures user browser traffic connecting to the Databricks web UI and workspace over a private IP.
- **Back-End Private Link:** Directs traffic between the compute data plane in the customer's VNet and the managed Control Plane across the Microsoft backbone network, completely eliminating public internet traversal.

---

### Q9: How do Deletion Vectors work internally in Delta Lake 3.x, and how do they eliminate write amplification?

**Difficulty:** `ARCHITECT` | **Niche:** `Deletion Vectors & Storage`

- **Without Deletion Vectors (Copy-on-Write):** Updating or deleting 1 row inside a 500 MB Parquet file forces Delta to read the entire 500 MB file, rewrite 499.9 MB of unchanged rows into a brand new Parquet file, and tombstone the old file (massive write amplification).
- **With Deletion Vectors (Soft Deletes via Bitmaps):** Delta leaves the original Parquet file intact and writes a tiny serialized bitmap file containing the row index of deleted rows. Readers read the base Parquet file and apply the deletion vector in-memory to skip deleted rows. When files accumulate many deletion vectors, `OPTIMIZE` consolidates them.

---

### Q10: Write a SQL query against Databricks System Tables to identify the top 5 most expensive jobs over the last 30 days.

**Difficulty:** `HARD` | **Niche:** `FinOps & Billing Analytics`

**Production SQL:**
```sql
SELECT
    usage.usage_metadata.job_id,
    usage.usage_metadata.job_name,
    ROUND(SUM(usage.usage_quantity), 2) AS total_dbus_consumed,
    ROUND(SUM(usage.usage_quantity * pricing.pricing.default), 2) AS estimated_usd_cost
FROM system.billing.usage usage
JOIN system.billing.list_prices pricing
  ON usage.cloud = pricing.cloud 
  AND usage.sku_name = pricing.sku_name
  AND usage.usage_start_time >= pricing.price_start_time
  AND (pricing.price_end_time IS NULL OR usage.usage_end_time < pricing.price_end_time)
WHERE usage.usage_start_time >= CURRENT_DATE() - INTERVAL 30 DAYS
  AND usage.usage_metadata.job_id IS NOT NULL
GROUP BY 1, 2
ORDER BY total_dbus_consumed DESC
LIMIT 5;
```

---

### Q11: How do Databricks Asset Bundles (DABs) implement multi-environment GitOps CI/CD across Dev, Staging, and Prod?

**Difficulty:** `HARD` | **Niche:** `Platform Architecture & Fabric`

Databricks Asset Bundles (DABs) declare jobs, pipelines (DLT), and notebooks as declarative YAML code managed in Git.
```yaml
bundle:
  name: retail-lakehouse
targets:
  dev:
    default: true
    workspace:
      host: https://adb-dev.azuredatabricks.net
  prod:
    workspace:
      host: https://adb-prod.azuredatabricks.net
    run_as:
      service_principal_name: sp-lakehouse-cicd
resources:
  jobs:
    daily_etl:
      tasks:
        - task_key: dlt_pipeline
          pipeline_task:
            pipeline_id: ${resources.pipelines.sales_dlt.id}
```
**Workflow:** CI/CD runners (GitHub Actions/Azure DevOps) authenticate as a dedicated Service Principal, run `databricks bundle validate`, and deploy via `databricks bundle deploy -t prod`, ensuring 100% infrastructure-as-code reproducibility.

---

### Q12: How do you configure dynamic row filtering and column masking in Unity Catalog?

**Difficulty:** `HARD` | **Niche:** `Unity Catalog Governance`

Unity Catalog supports fine-grained access control using SQL user-defined functions (UDFs) evaluated dynamically at runtime based on caller groups:
```sql
-- 1. Column Masking Function
CREATE OR REPLACE FUNCTION mask_ssn(ssn STRING)
RETURNS STRING
RETURN CASE 
  WHEN is_account_group_member('hr_admins') THEN ssn
  ELSE CONCAT('XXX-XX-', RIGHT(ssn, 4))
END;

ALTER TABLE gold.employees ALTER COLUMN ssn SET MASK mask_ssn;

-- 2. Row Filtering Function
CREATE OR REPLACE FUNCTION filter_by_region(region STRING)
RETURNS BOOLEAN
RETURN is_account_group_member('executives') OR 
       (is_account_group_member('emea_sales') AND region = 'EMEA');

ALTER TABLE gold.sales SET ROW FILTER filter_by_region ON (region);
```
Queries run without code changes; unauthorized users automatically receive masked values and filtered rows.

---

### Q13: What is the architectural difference between Databricks Serverless Compute and Classic Customer-Managed Compute?

**Difficulty:** `ARCHITECT` | **Niche:** `Platform Architecture & Fabric`

**Classic Compute:** Worker VMs are provisioned inside the customer's cloud VNet/VPC. Clusters take 3-7 minutes to spin up (VM bootstrapping), idle clusters burn cloud infrastructure costs, and the customer manages subnet IP exhaustion and VM quota limits.
**Serverless Compute:**
- Compute runs inside a secure, multi-tenant fleet managed by Databricks in the control plane.
- **Instant Provisioning:** Workloads start in < 2 seconds because a warm pool of pre-warmed containerized executors is dynamically attached.
- **True Zero Idle:** Scale-down to 0 is instantaneous upon query completion; zero DBU or VM cost when queries are idle.
- **Network Isolation:** Databricks connects to customer storage via Secure Storage Connectors and Private Link endpoints.

---

### Q14: How do you query Databricks System Tables to detect cost anomalies and untagged cluster spend?

**Difficulty:** `HARD` | **Niche:** `Platform Architecture & Fabric`

Databricks system tables expose audit logs, billing usage, and access telemetry as standard Delta tables under the `system` catalog:
```sql
-- Track top 5 most expensive DLT pipelines over the last 30 days
SELECT 
  usage_metadata.pipeline_id,
  SUM(usage_quantity) AS total_dbus,
  SUM(usage_quantity * 0.35) AS estimated_usd -- custom rate card
FROM system.billing.usage
WHERE sku_name LIKE '%DLT%'
  AND usage_date >= current_date() - INTERVAL 30 DAYS
GROUP BY 1
ORDER BY total_dbus DESC
LIMIT 5;

-- Audit untagged clusters
SELECT cluster_id, SUM(usage_quantity) AS unallocated_dbus
FROM system.billing.usage
WHERE custom_tags['CostCenter'] IS NULL
GROUP BY 1;
```

---

### Q15: What are Databricks Lakeflow Pipelines, and how do they evolve Delta Live Tables (DLT)?

**Difficulty:** `ARCHITECT` | **Niche:** `Platform Architecture & Fabric`

**Lakeflow** unifies data ingestion (Lakeflow Connect for Salesforce, Postgres, Oracle), transformation (Lakeflow Pipelines), and orchestration (Lakeflow Jobs) into a single declarative platform.
- **Lakeflow Connect:** Native, serverless change-data-capture (CDC) connectors that extract database log streams into Bronze Delta tables without third-party tools (Fivetran/Debezium).
- **Evolution over Classic DLT:** Adds real-time micro-batch triggering with sub-second latencies, unified SQL/Python declarative syntax, native schema contracts, and automatic dead-letter queue (DLQ) branching.
- **GitOps First:** Fully configured via DABs and deployed with automated blue-green cutover.

---
