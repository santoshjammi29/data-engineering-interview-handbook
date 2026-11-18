# Pillar 13: Power BI, Semantic Layer & Serving
**Category:** `POWER BI` | **Domain:** `Business Intelligence & Serving Layer`

---

### Q1: Compare Direct Lake mode vs Import mode vs DirectQuery in Power BI enterprise deployments.

**Difficulty:** `HARD` | **Niche:** `Power BI Storage Modes`

**Comparison Matrix:**
- **Import Mode:** Data imported into VertiPaq memory. Fastest sub-second report interactivity. Drawback: memory size limits (typically 10-100GB), data staleness between scheduled refreshes, and duplicate data storage.
- **DirectQuery:** Translates DAX queries into live SQL on underlying warehouse/lakehouse. Data is always 100% fresh. Drawback: slow visual render times, high query concurrency burdens on database engines.
- **Direct Lake Mode (Fabric):** VertiPaq engine loads Delta-Parquet files directly from OneLake storage into memory on-demand without data copying or SQL translation. Combines the speed of Import with the real-time freshness of DirectQuery.

---

### Q2: Why does a Kimball Star Schema significantly outperform a single flat wide table in the Power BI VertiPaq engine?

**Difficulty:** `HARD` | **Niche:** `VertiPaq Engine & Star Schema`

VertiPaq compresses columns independently using **Dictionary Encoding**:
- **Flat Wide Table (Anti-pattern):** Storing customer attributes (name, address, city) repeated across 100 million order rows explodes dictionary sizes and destroys compression efficiency.
- **Star Schema (Best Practice):** Normalizing attributes into dimension tables leaves small integer surrogate keys in the fact table. Fact table integer columns compress via Bit-Packing into negligible memory, maximizing CPU cache utilization and speeding up aggregations.

---

### Q3: Explain the architectural difference between Row Context and Filter Context in DAX, and what is Context Transition?

**Difficulty:** `HARD` | **Niche:** `DAX Internals & Context`

- **Row Context:** The concept of the 'current row' during iterative calculations (e.g. calculated columns, `SUMX`, `FILTER`). Does NOT filter the table automatically.
- **Filter Context:** The set of active filters applied to the data model by report slicers, visuals, page filters, and `CALCULATE()` statements.
- **Context Transition:** Occurs when `CALCULATE()` is invoked inside a Row Context. It automatically converts the current row's column values into an equivalent Filter Context, filtering the entire data model.

---

### Q4: Explain the 3 primary compression algorithms utilized by the Power BI VertiPaq engine.

**Difficulty:** `HARD` | **Niche:** `VertiPaq Compression Internals`

**3 Core Compression Encodings:**
1. **Value Encoding:** Subtracts the minimum value of an integer column from all values, shrinking the bit-width required to store numbers.
2. **Dictionary Encoding:** Replaces strings with integer IDs and builds an in-memory dictionary. Memory footprint is determined by column **cardinality**, not row count.
3. **Run-Length Encoding (RLE):** Compresses consecutive duplicate values into `(value, count)` pairs. Extremely efficient when tables are sorted by the encoded column.

---

### Q5: How do User-Defined Aggregation Tables optimize query performance over billion-row lakehouses?

**Difficulty:** `HARD` | **Niche:** `Aggregation Tables & Composite Models`

In Composite Models, create a pre-aggregated table (e.g. `agg_sales_monthly`) cached in **Import Mode**, while keeping the underlying billion-row `fact_sales` in **DirectQuery Mode**:
- **Automatic Query Routing:** When an executive visual asks for monthly sales, Power BI intercepts the query and answers it from the fast in-memory aggregation table ($< 100\text{ms}$).
- If a user drills down to individual invoice line items, Power BI automatically falls back to DirectQuery against the lakehouse.

---

### Q6: Compare Row-Level Security (RLS) vs Object-Level Security (OLS) in Power BI semantic models.

**Difficulty:** `MEDIUM` | **Niche:** `Security & RLS/OLS`

**Comparison:**
- **Row-Level Security (RLS):** Restricts data access at the row level based on user identity (e.g. `[Region] = USERPRINCIPALNAME()`). Users can still see the table and visual structure, but only their authorized rows appear.
- **Object-Level Security (OLS):** Completely secures and conceals entire tables or sensitive columns (e.g. `SSN`, `Executive Salary`) from unauthorized users. To restricted users, the column appears non-existent.

---

### Q7: How do you configure Incremental Refresh and Hybrid Tables in Power BI Premium?

**Difficulty:** `MEDIUM` | **Niche:** `Incremental Refresh`

**Configuration Steps:**
1. Define reserved `RangeStart` and `RangeEnd` datetime parameters in Power Query.
2. Configure incremental refresh policy: e.g., **Store 5 years of history**, but **Refresh only last 7 days**.
3. **Hybrid Tables (Real-Time):** In Power BI Premium, configure the latest day/partition in **DirectQuery** mode while keeping historical partitions in **Import Mode**, providing real-time data with sub-second historical reporting.

---

### Q8: How do you size and cluster On-Premises Data Gateways for high-throughput enterprise Power BI refreshes?

**Difficulty:** `MEDIUM` | **Niche:** `Enterprise Gateway Infrastructure`

**Gateway Clustering Best Practice:**
1. Deploy at least **2 high-spec Gateway VMs** in an active-active cluster to eliminate single points of failure.
2. Enable **Load Balancing:** Automatically distributes concurrent scheduled refresh jobs across cluster nodes.
3. Allocate at least 8 CPU cores, 32GB RAM, and 1 Gbps dedicated network link per node to prevent spooling bottlenecks during massive multi-million row extracts.

---

### Q9: Compare Power BI Datamarts/Semantic Models with dedicated semantic platforms (Cube, AtScale, dbt Semantic Layer).

**Difficulty:** `HARD` | **Niche:** `Semantic Layer Comparison`

**Comparison:**
- **Power BI Semantic Models:** Deeply optimized for Microsoft Power BI consumers with DAX calculations and VertiPaq caching. However, vendor-locked and difficult for non-Microsoft tools (Tableau, Python) to consume.
- **Universal Semantic Layers (Cube, AtScale, dbt):** Vendor-agnostic. Expose a single unified metric layer accessible via SQL, REST, and GraphQL, serving Power BI, Tableau, Excel, and AI agents interchangeably.

---

### Q10: How do you identify DAX bottlenecks and visual latency using the Power BI Performance Analyzer and DAX Studio?

**Difficulty:** `HARD` | **Niche:** `Performance Profiling`

**Profiling Workflow:**
1. Run **Performance Analyzer** in Power BI Desktop to capture visual duration breakdown: DAX Query, Visual Display, and Other wait time.
2. Copy the query into **DAX Studio**; enable **Server Timings**.
3. Inspect the split between **Storage Engine (SE)** and **Formula Engine (FE)**:
   - If Formula Engine $> 30\%$: DAX query contains un-vectorized row-by-row iteration (e.g. non-sargable `IF` inside `SUMX`). Rewrite to push filtering into the VertiPaq Storage Engine.

---

### Q11: How does the Power BI VertiPaq engine compress tabular data using Dictionary, Value, and Run-Length Encoding?

**Difficulty:** `HARD` | **Niche:** `VertiPaq Engine Internals`

VertiPaq achieves up to 10x-20x in-memory compression through 3 sequential encoding phases:
1. **Value Encoding:** Subtracks the minimum value from integer columns to reduce bit-width (e.g. values $1001-1005$ store integers $1-5$, requiring 3 bits instead of 32 bits).
2. **Dictionary Encoding:** Replaces strings with compact integer IDs and maintains an alphabetical dictionary.
3. **Run-Length Encoding (RLE):** Compresses repeated adjacent values into `(Value, Count)` pairs. Sorting the table by high-frequency columns prior to compression maximizes RLE efficiency.

---

### Q12: Explain DAX Context Transition and how CALCULATE() mutates Row Context into Filter Context.

**Difficulty:** `HARD` | **Niche:** `DAX Context Transition`

**Row Context:** Iterates row by row (e.g. inside `SUMX` or calculated columns); aware only of the current row's values, but does not filter the model.
**Filter Context:** The active set of filters applied by slicers, report visuals, and DAX filters that restrict data across table relationships.
**Context Transition:** When `CALCULATE()` (or a measure reference) is evaluated inside an existing Row Context, it automatically transforms all column values of the current row into an equivalent Filter Context, filtering related tables along active relationships.

---

### Q13: How do Power BI Composite Models and Aggregation Tables eliminate DirectQuery latency on 100-billion row fact tables?

**Difficulty:** `HARD` | **Niche:** `Aggregation Tables & Composite Models`

Composite models allow mixing in-memory **Import mode** tables with live **DirectQuery** tables in the same model.
**Aggregation Tables:**
- Pre-aggregate the billion-row fact table to a coarse grain (e.g. `Year`, `Month`, `Store_ID`, `Sum(Sales)`).
- Load the aggregation table in fast in-memory Import mode.
- Map the aggregation table to the underlying DirectQuery detail table in Power BI relationship properties.
- *Query Routing:* High-level dashboard queries hit the in-memory aggregation table (sub-second); detail drill-through queries automatically fall back to DirectQuery SQL.

---

### Q14: How do XMLA endpoints enable programmatic partition refresh and deployment automation in Power BI Premium?

**Difficulty:** `HARD` | **Niche:** `Incremental Refresh & Partitioning`

Power BI Premium workspaces expose read/write **XMLA endpoints** identical to SQL Server Analysis Services (SSAS).
- Data engineers use tools like Tabular Editor, PowerShell, or Azure Data Factory to connect to `powerbi://api.powerbi.com/v1.0/myorg/WorkspaceName`.
- **Micro-Partition Refresh:** Instead of refreshing the entire semantic model, pipelines execute TMSL (Tabular Model Scripting Language) to refresh only the single daily partition: `{"refresh": {"type": "full", "partition": "FactSales_20261118"}}`.

---

### Q15: How do you implement Dynamic Row-Level Security (RLS) in Power BI using USERPRINCIPALNAME()?

**Difficulty:** `MEDIUM` | **Niche:** `Row-Level Security Architecture`

1. Create an `EmployeeSecurity` table mapping Azure AD emails to authorized branch IDs (`UserEmail`, `BranchID`).
2. Establish a 1-to-many relationship from `EmployeeSecurity` to `Dim_Branch` with cross-filter direction set to 'Single' or 'Both'.
3. Define an RLS Role in Power BI modeling tab:
```dax
[UserEmail] = USERPRINCIPALNAME()
```
When a user opens the report, `USERPRINCIPALNAME()` extracts their authenticated Azure AD identity, filters `Dim_Branch`, and cascades down to restrict fact table rows dynamically.

---
