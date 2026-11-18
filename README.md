# Data Engineering Interview Handbook & Architectural Defense Guide

A comprehensive, production-tested reference manual and interview defense playbook for Data Engineers, Lakehouse Architects, and Big Data Leads. 

This repository provides original technical explanations, production code implementations (SQL, PySpark, Python), performance triage runbooks, and enterprise system designs spanning modern lakehouse platforms and distributed data systems.

---

## 📚 Repository Structure

```text
data-engineering-interview-handbook/
├── playbooks/                        # Core Interview Defense Playbooks (125 Essential Questions)
│   ├── 01_sql_and_pyspark_125.md     # SQL & PySpark internals, Catalyst, Tungsten, GC triage
│   ├── 02_kafka_and_streaming_125.md # Apache Kafka, EOS, and Spark Structured Streaming
│   ├── 03_azure_databricks_125.md    # Delta Lake 3.x, Liquid Clustering, Unity Catalog
│   ├── 04_aws_cloud_125.md           # AWS Glue, EMR, S3 Lakehouse, Step Functions
│   ├── 05_project_defense_125.md     # STAR project narratives, scale-up strategies, triage
│   ├── 06_system_design_125.md       # High-throughput streaming, fraud detection, recovery
│   ├── 07_managerial_leadership_125.md # Technical disputes, FinOps, team leadership
│   └── 125_master_defense_playbook.md # Full consolidated 125-question playbook
├── pillars/                          # 14 Knowledge Base Technical Pillars (140 Core Questions)
│   ├── 01_spark_and_databricks.md    # Spark 3.5 engine, Catalyst, Tungsten, memory tuning
│   ├── 02_databricks_lakehouse.md    # Unity Catalog, DABs, Serverless, Photon, Lakeflow
│   ├── 03_modern_lakehouse.md        # Delta Lake vs Iceberg vs Hudi, Deletion Vectors
│   ├── 04_datalake_architecture.md   # Cloud object storage, consistency, prefix scaling
│   ├── 05_apache_airflow.md          # Airflow 2/3, Dynamic Task Mapping, Deferrable Operators
│   ├── 06_azure_data_factory.md      # SHIR architecture, VNet isolation, managed identities
│   ├── 07_dbt_data_build_tool.md     # Incremental models, snapshots, semantic layer, mesh
│   ├── 08_microsoft_fabric.md        # OneLake, Direct Lake mode, Shortcuts, F-SKU smoothing
│   ├── 09_database_architecture.md   # B-Tree vs LSM, Raft/Paxos, MVCC, ARIES, sharding
│   ├── 10_sql_server_tsql.md         # Clustered/Columnstore indexes, execution plans, RCSI
│   ├── 11_python_for_data_engineering.md # Memory management, PyArrow, Polars, concurrency
│   ├── 12_data_modeling_and_cdc.md   # Kimball, Data Vault 2.0, SCD 1-6, Debezium, Outbox
│   ├── 13_power_bi_and_serving.md    # VertiPaq engine, DAX context transition, Aggregations
│   ├── 14_system_design_at_scale.md  # Multi-tenant lakehouse, real-time ML, FinOps
│   └── all_14_pillars_master_guide.md # Consolidated 14-pillar reference manual
└── data/                             # Structured JSON datasets
    ├── questions_125.json            # 125 questions with schema attributes
    ├── questions_14_pillars.json     # 140 pillar questions
    └── all_interview_questions.json  # Complete 265-question master dataset
```

---

## 🎯 Curriculum Pillars

| # | Pillar | Topics Covered |
| :---: | :--- | :--- |
| **01** | **Spark & Databricks Architecture** | Catalyst optimizer lifecycle, Tungsten off-heap memory, shuffle sizing, Dynamic Partition Pruning, AQE skew join, Vectorized Pandas UDFs. |
| **02** | **Databricks Lakehouse & Unity Catalog** | Unity Catalog 3-tier namespace, row/column masking, Databricks Asset Bundles (DABs), Serverless compute, Photon engine, Liquid Clustering. |
| **03** | **Modern Lakehouse Table Formats** | Delta Lake 3.x vs Apache Iceberg vs Apache Hudi, Copy-on-Write vs Merge-on-Read, Deletion Vectors, UniForm multi-engine interoperability. |
| **04** | **Data Lake Architecture & Storage** | S3/ADLS Gen2 consistency models, prefix request throttling (HTTP 503), small file compaction, disaster recovery, zero-trust perimeter. |
| **05** | **Apache Airflow Orchestration** | Airflow 2 vs 3, Celery vs Kubernetes executors, Dynamic Task Mapping, Deferrable Operators, Custom XCom backends, High Availability. |
| **06** | **Azure Data Factory & Integration** | High-availability SHIR clusters, private endpoints, watermark incremental patterns, API pagination, Managed Identity security. |
| **07** | **dbt (Data Build Tool)** | Incremental strategies (`merge`, `append`, `insert_overwrite`), snapshots (SCD2), generic/singular testing, dbt Semantic Layer, dbt Mesh. |
| **08** | **Microsoft Fabric** | OneLake logical architecture, Direct Lake mode in Power BI, OneLake Shortcuts, Lakehouse vs Warehouse, Capacity Smoothing. |
| **09** | **Database Architecture & Systems** | Row vs Columnar storage, B-Tree vs LSM-Tree, PACELC theorem, Vitess sharding, Raft consensus, MVCC isolation, ARIES recovery. |
| **10** | **SQL Server & T-SQL Mastery** | B-Tree leaf node architecture, Clustered Columnstore Indexes, parameter sniffing remediation, partition switching, TempDB latch contention. |
| **11** | **Python for Data Engineering** | Reference counting and cyclic GC, generators (`yield`), GIL concurrency models, PyArrow zero-copy sharing, Polars vs Pandas 2.0. |
| **12** | **Data Modeling & Streaming Ingestion** | Kimball vs Inmon vs Data Vault 2.0, SCD Types 1-6, Log-based CDC (Debezium), Kafka partition sizing, Transactional Outbox pattern. |
| **13** | **Power BI, Semantic Layer & Serving** | VertiPaq compression (Value, Dictionary, RLE), DAX context transition, User-defined aggregations, Row-Level & Object-Level Security. |
| **14** | **Enterprise System Design at Scale** | Multi-tenant lakehouse design, real-time fraud detection (< 20ms p99), 10 PB cloud migration, FinOps cost governance, incident response. |

---

## 🛠️ Usage

### Cloning and Setup
```bash
git clone https://github.com/santoshjammi29/data-engineering-interview-handbook.git
cd data-engineering-interview-handbook
```

### JSON Datasets
All questions are available in machine-readable JSON format under `data/`:
```python
import json

with open("data/all_interview_questions.json") as f:
    questions = json.load(f)

print(f"Total questions loaded: {len(questions)}")
# Inspect a question
print(questions[0]["question"])
print(questions[0]["answer"])
```

---

## 📄 License
MIT License. Free for educational and professional preparation purposes.
