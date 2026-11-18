# Data Engineering Interview Handbook & Architectural Defense Guide

A comprehensive, production-tested reference manual and interview defense playbook for Data Engineers, Lakehouse Architects, and Big Data Leads. 

This repository provides original technical explanations, production code implementations (SQL, PySpark, Python), performance triage runbooks, and enterprise system designs across **495 problems and interview questions**.

---

## 📚 Repository Structure

```text
data-engineering-interview-handbook/
├── playbooks/                        # Core Interview Defense Playbooks (125 OCR Slides + 100 Spark Issues)
│   ├── 100_spark_data_issues_problems_and_answers.md # 100 Real-World Spark/PySpark/SQL Data Issues
│   ├── 01_sql_and_pyspark_125.md     # SQL & PySpark internals, Catalyst, Tungsten, GC triage
│   ├── 02_kafka_and_streaming_125.md # Apache Kafka, EOS, and Spark Structured Streaming
│   ├── 03_azure_databricks_125.md    # Delta Lake 3.x, Liquid Clustering, Unity Catalog
│   ├── 04_aws_cloud_125.md           # AWS Glue, EMR, S3 Lakehouse, Step Functions
│   ├── 05_project_defense_125.md     # STAR project narratives, scale-up strategies, triage
│   ├── 06_system_design_125.md       # High-throughput streaming, fraud detection, recovery
│   ├── 07_managerial_leadership_125.md # Technical disputes, FinOps, team leadership
│   └── 125_master_defense_playbook.md # Full consolidated 125-question playbook
├── pillars/                          # 20 Knowledge Base Technical Pillars (270 Core Questions)
│   ├── 01_spark_and_databricks.md    # Spark 3.5, Catalyst custom rules, Tungsten spill, Photon
│   ├── 02_databricks_lakehouse.md    # Unity Catalog, DABs, Serverless, Lakeflow, System Tables
│   ├── 03_modern_lakehouse.md        # Delta Lake Liquid Clustering, UniForm, Deletion Vectors
│   ├── 04_datalake_architecture.md   # S3/ADLS prefix throttling, bin-packing, DR RPO/RTO
│   ├── 05_apache_airflow.md          # Airflow 2/3, Dynamic Task Mapping, Deferrable Operators
│   ├── 06_azure_data_factory.md      # SHIR HA clusters, watermark pipelines, Mapping Data Flows
│   ├── 07_dbt_data_build_tool.md     # Incremental merge/overwrite, dbt Mesh, Semantic Layer
│   ├── 08_microsoft_fabric.md        # OneLake Shortcuts, Direct Lake mode, Capacity Smoothing
│   ├── 09_database_architecture.md   # B-Tree vs LSM, ARIES crash recovery, MVCC, Raft consensus
│   ├── 10_sql_server_tsql.md         # Clustered Columnstore, parameter sniffing, partition switching
│   ├── 11_python_for_data_engineering.md # CPython GC, PyArrow zero-copy IPC, Polars LazyFrames, PyO3
│   ├── 12_data_modeling_and_cdc.md   # SCD 1-6, Data Vault 2.0, Debezium Outbox, Bi-Temporal
│   ├── 13_power_bi_and_serving.md    # VertiPaq compression, DAX context transition, Aggregations
│   ├── 14_system_design_at_scale.md  # Multi-tenant lakehouse, real-time feature store, FinOps
│   ├── 15_apache_iceberg_deep_dive.md # Metadata tree, hidden partitioning, positional deletes, REST/Nessie
│   ├── 16_apache_flink_streaming.md  # Chandy-Lamport barriers, RocksDB state, unaligned checkpoints
│   ├── 17_trino_presto_duckdb.md     # Trino MPP, dynamic filtering, DuckDB SIMD embedded analytics
│   ├── 18_genai_vector_lakehouse.md  # HNSW vs IVF-PQ, Databricks Vector Search, RAG hybrid search
│   ├── 19_finops_compute_optimization.md # DBU tuning, Serverless cost curves, Photon ROI, spot fallback
│   ├── 20_data_quality_contracts_observability.md # Data Contracts, streaming circuit breakers, IQR anomalies
│   └── all_20_pillars_master_guide.md # Consolidated 20-pillar reference manual (270 questions)
└── data/                             # Structured JSON datasets
    ├── questions_125.json            # 125 OCR slide questions
    ├── 100_spark_data_issues.json    # 100 Spark/PySpark data issue problems
    ├── questions_20_pillars.json     # 270 pillar questions across 20 domains
    └── all_interview_questions.json  # Complete 495-question master dataset
```

---

## 🎯 The 20 Technical Curriculum Pillars

| # | Pillar | Questions | Key Production Topics Covered |
| :---: | :--- | :---: | :--- |
| **01** | **Spark & Databricks Architecture** | 15 | Catalyst optimizer rules, Tungsten off-heap memory, shuffle spill triage, AQE coalescing, RocksDB streaming, Photon C++ SIMD. |
| **02** | **Databricks Lakehouse & Unity Catalog** | 15 | Unity Catalog 3-tier namespace, DABs GitOps, dynamic row/column masking, Serverless compute, System Tables FinOps, Lakeflow. |
| **03** | **Modern Lakehouse Table Formats** | 15 | Delta Lake 3.x Liquid Clustering, UniForm metadata translation, Deletion Vectors Roaring Bitmaps, Shallow vs Deep Clone, OCC write conflicts. |
| **04** | **Data Lake Architecture & Storage** | 15 | S3/ADLS Gen2 prefix request throttling (HTTP 503), small file bin-packing, DR Active-Passive RPO/RTO, Glacier tiering traps, Zero-Trust security. |
| **05** | **Apache Airflow Orchestration** | 15 | Airflow 2 vs 3 decoupled scheduler, Dynamic Task Mapping (`.expand()`), Deferrable Operators & Triggerer daemon, Custom XCom S3 backends, Multi-Scheduler HA. |
| **06** | **Azure Data Factory & Integration** | 15 | Self-Hosted IR (SHIR) HA clustering, watermark incremental extraction, Mapping Data Flows cluster reuse, REST OAuth2 pagination, ARM CI/CD gates. |
| **07** | **dbt (Data Build Tool)** | 15 | Incremental strategies (`merge`, `append`, `insert_overwrite`), snapshots (SCD2), MetricFlow Semantic Layer, dbt Mesh cross-project contracts, unit tests. |
| **08** | **Microsoft Fabric & OneLake** | 15 | OneLake Shortcuts, Power BI Direct Lake mode over Delta, Lakehouse vs Warehouse, F-SKU Capacity Smoothing, KQL Real-Time Intelligence. |
| **09** | **Database Architecture & Systems** | 15 | B-Tree vs LSM-Tree amplification, MVCC snapshot isolation, ARIES WAL crash recovery (Analysis/Redo/Undo), distributed sharding (Vitess), Raft consensus. |
| **10** | **SQL Server & T-SQL Mastery** | 15 | Clustered Columnstore rowgroups, parameter sniffing remediation, partition switching zero-logging, RCSI TempDB version store, DMV execution plan analysis. |
| **11** | **Python for Data Engineering** | 15 | CPython reference counting and cyclic GC, PyArrow zero-copy IPC memory, Polars LazyFrame SIMD vectorization, asyncio vs multiprocessing, PyO3 Rust extensions. |
| **12** | **Data Modeling & CDC** | 15 | SCD Types 1-6 complete implementations, Data Vault 2.0 hash keys, Debezium Transactional Outbox CDC, Star vs Snowflake vs OBT, Bi-Temporal financial ledgers. |
| **13** | **Power BI & Serving Layer** | 15 | VertiPaq Value/Dict/RLE compression, DAX context transition (`CALCULATE()`), Composite Models & Aggregation Tables, XMLA partition refresh, Dynamic RLS. |
| **14** | **System Design at Scale** | 15 | Multi-tenant lakehouse isolation, dual-storage real-time feature store (< 5ms), Lakehouse FinOps auto-scaling, CQRS event sourcing, automated lineage. |
| **15** | **Apache Iceberg Deep Dive** | 10 | 4-tier metadata tree, hidden partitioning, positional vs equality deletes, schema evolution with Field IDs, REST/Nessie/Polaris catalogs, WAP pattern. |
| **16** | **Apache Flink & Stateful Streaming** | 10 | JobManager/TaskManager slots, Chandy-Lamport barriers, HashMap vs RocksDB backends, unaligned checkpoints, watermark alignment, two-phase commit EOS. |
| **17** | **Trino, Presto & DuckDB** | 10 | Trino coordinator MPP, dynamic filtering pushdown, memory pools and spill, Project Tardigrade fault tolerance, DuckDB morsel SIMD, Arrow zero-copy. |
| **18** | **GenAI, Vector Search & Embeddings** | 10 | HNSW vs IVF-PQ indexing, Databricks Vector Search Delta sync, recursive chunking, GPU Spark embedding batches, semantic caching, hybrid search (BM25 + Dense). |
| **19** | **FinOps & Cost Engineering** | 10 | DBU cost reduction checklist, Serverless vs Classic compute cost curves, Photon ROI profiling, S3 multipart aborts, BigQuery slots vs on-demand, Spot fallback. |
| **20** | **Data Quality, Contracts & Observability** | 10 | Data Contracts YAML specification, streaming circuit breakers, 5 pillars of observability, statistical IQR anomaly detection, CDC automated checksum reconciliation. |
| **Σ** | **TOTAL** | **495** | **Complete Full-Stack Data Engineering & Lakehouse Architecture Mastery** |

---

## ⚡ Master Playbooks

- 📖 **[100 Spark, PySpark & SQL Data Issue Problems & Solutions](playbooks/100_spark_data_issues_problems_and_answers.md)**
- 📖 **[125 Master Technical Defense Playbook](playbooks/125_master_defense_playbook.md)**
- 📖 **[All 20 Technical Pillars Master Guide](pillars/all_20_pillars_master_guide.md)**
