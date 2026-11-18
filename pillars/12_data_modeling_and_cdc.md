# Pillar 12: Data Modeling, CDC & Streaming Ingestion
**Category:** `GENERAL DE` | **Domain:** `Data Architecture & Integration`

---

### Q1: Compare Kimball Dimensional Modeling, Inmon Corporate Information Factory (CIF), and Data Vault 2.0.

**Difficulty:** `ARCHITECT` | **Niche:** `Data Modeling Methodologies`

**Comparison Matrix:**
- **Kimball (Dimensional / Bottom-Up):** Star schemas with conformed dimensions and fact tables. Highly optimized for analytical BI queries, intuitive for business users, fast delivery.
- **Inmon (Enterprise / Top-Down):** Normalized 3NF (Third Normal Form) enterprise data warehouse acting as the central source of truth, with downstream departmental data marts.
- **Data Vault 2.0 (Agile / Scalable):** Models business core concepts into **Hubs** (business keys), **Links** (relationships/transactions), and **Satellites** (context/attributes with full history). Extremely agile, parallelizable, and auditable for multi-source enterprise lakehouses.

---

### Q2: Explain Slowly Changing Dimension (SCD) Types 1, 2, 3, 4, and 6 with concrete enterprise use cases.

**Difficulty:** `HARD` | **Niche:** `Slowly Changing Dimensions`

**SCD Types Breakdown:**
- **Type 1 (Overwrite):** Overwrites old data with new data. No history tracked. (Use case: correcting typo in user name).
- **Type 2 (Add New Row):** Inserts a new row with `valid_from`, `valid_to`, and `is_current`. Full history preserved. (Use case: address change affecting sales tax).
- **Type 3 (Add New Column):** Adds an `original_attribute` and `current_attribute` column in the same row. Tracks 1 historical state.
- **Type 4 (Historical Table):** Retains only current values in main dimension; historical changes written to a separate history table.
- **Type 6 (Hybrid 1 + 2 + 3):** Combines Types 1, 2, and 3: row tracks history (Type 2) while also maintaining a current attribute column updated across all rows (Type 1).

---

### Q3: How does Log-Based Change Data Capture (CDC) via Debezium outperform trigger-based or query-based CDC?

**Difficulty:** `HARD` | **Niche:** `CDC Architecture & Debezium`

- **Query-Based CDC (`WHERE updated_at > watermark`):** Misses physical `DELETE` operations, imposes heavy query overhead on production databases, and misses intermediate state changes that occur between polling cycles.
- **Log-Based CDC (Debezium):** Reads directly from database internal transaction logs (e.g. PostgreSQL WAL, MySQL binlog). Captures all `INSERT`, `UPDATE`, and `DELETE` events with zero application overhead, guarantees exact ordering, and captures true transaction boundaries without modifying source schemas.

---

### Q4: How do you calculate the optimal partition count for an Apache Kafka topic with a 500 MB/sec throughput SLA?

**Difficulty:** `HARD` | **Niche:** `Kafka Partition Topology`

**Sizing Formula:**
$$\text{Partitions} = \max\left(\frac{\text{Target Throughput}}{\text{Max Single-Producer Throughput}}, \frac{\text{Target Throughput}}{\text{Max Single-Consumer Throughput}}\right)$$
- If target throughput is **500 MB/sec**, a single producer thread achieves ~50 MB/sec, and a single streaming consumer task achieves ~25 MB/sec:
$$\text{Partitions} = \max\left(\frac{500}{50}, \frac{500}{25}\right) = \max(10, 20) = 20\text{ partitions}$$
**Production Rule:** Size to **24 to 32 partitions** (powers of 2/4) to provide head-room for future consumer scaling.

---

### Q5: Explain the Transactional Outbox Pattern and how it prevents dual-write distributed system inconsistencies.

**Difficulty:** `ARCHITECT` | **Niche:** `Outbox Pattern & Dual Writes`

- **The Dual-Write Problem:** Updating a database and publishing an event to Kafka in the same application service risks inconsistency if Kafka crashes after the database commits, or vice versa.
- **The Solution:** Inside the **exact same local ACID database transaction**, write business state to the `orders` table AND append an event record into an `outbox` table.
- A dedicated log-tailing process (Debezium / Kafka Connect) reads from the `outbox` table transaction log and publishes guaranteed events to Kafka with zero data loss.

---

### Q6: How do you implement high-throughput stream deduplication using Sliding Windows and Bloom Filters?

**Difficulty:** `HARD` | **Niche:** `Stream Deduplication`

1. Maintain a **Bloom Filter** (space-efficient probabilistic bit array) in memory representing seen transaction IDs over the last 15 minutes.
2. If the Bloom filter returns `False`, the transaction is guaranteed unique $\rightarrow$ emit immediately.
3. If it returns `True` (potential false positive), perform an exact lookup in an off-heap state store (RocksDB) to confirm duplication.
4. Rotate Bloom filters periodically with sliding event-time watermarks.

---

### Q7: How do you design a Dead Letter Queue (DLQ) pipeline with automated reprocessing and alerting?

**Difficulty:** `MEDIUM` | **Niche:** `Dead Letter Queues & DLQ`

**Production DLQ Architecture:**
1. Catch deserialization or validation errors; wrap payload with error metadata (stack trace, timestamp, source topic, partition, offset).
2. Publish to `topic.dlq` without stopping the primary stream.
3. Alert engineering team when DLQ message count exceeds threshold (e.g. $> 10\text{ messages/min}$). 
4. Implement an administrative reprocessing tool that replays fixed DLQ records back to the main topic once upstream bugs are patched.

---

### Q8: Compare Schema Registry compatibility modes: BACKWARD, FORWARD, FULL, and NONE.

**Difficulty:** `HARD` | **Niche:** `Schema Registry Governance`

**Comparison:**
- **BACKWARD (Default):** Consumers using the new schema can read data produced with the previous schema. Allows upgrading consumers first.
- **FORWARD:** Consumers using the previous schema can read data produced with the new schema. Allows upgrading producers first.
- **FULL:** Both backward and forward compatible. New schema can read old data, and old schema can read new data. Ensures zero-downtime rolling upgrades across producers and consumers.
- **NONE:** No compatibility checks enforced. High risk of breaking downstream consumers.

---

### Q9: Compare Lambda Architecture, Kappa Architecture, and the Delta Lakehouse Architecture.

**Difficulty:** `ARCHITECT` | **Niche:** `Streaming System Paradigms`

**Comparison:**
- **Lambda Architecture:** Dual-pipeline. Fast speed layer (streaming) + accurate batch layer (MapReduce/Spark), merged at the serving layer. High operational complexity: developers must maintain two duplicate codebases.
- **Kappa Architecture:** Single pipeline. All data treated as streams (Kafka + Flink). Eliminates duplicate codebases, but historical re-processing is expensive.
- **Delta Lakehouse (Modern Standard):** Combines the simplicity of Kappa with the cost-efficiency of Lambda. Uses a single ACID storage format (Delta Lake) queried by both streaming and batch engines interchangeably.

---

### Q10: What is a Factless Fact Table, and give two distinct enterprise use cases.

**Difficulty:** `MEDIUM` | **Niche:** `Dimensional Modeling`

A Factless Fact Table contains no numerical measures or metrics; it consists purely of foreign keys representing the **occurrence of an event** or relationship:
1. **Event Tracking (Student Attendance):** `fact_attendance` containing `(student_key, course_key, date_key)`. Answers: *Did student attend class on this date?*
2. **Coverage / Eligibility (Insurance Eligibility):** `fact_promotion_coverage` containing `(product_key, store_key, promotion_key, date_key)`. Answers: *Which products were eligible for promotional discounts in which stores?*

---

### Q11: Provide a complete architectural comparison of Slowly Changing Dimension (SCD) Types 1, 2, 3, 4, and 6.

**Difficulty:** `HARD` | **Niche:** `SCD Types 1-6 Architecture`

**Type 1 (Overwrite):** Overwrites historical data. Zero history, updates in-place.
**Type 2 (Historical Rows):** Inserts a new row for each change with `start_date`, `end_date`, and `is_current` flag. Preserves infinite history.
**Type 3 (Previous Column):** Adds a column `previous_address` to store the immediate predecessor state.
**Type 4 (History Table):** Base dimension keeps current state (Type 1); historical changes are written to a separate `dim_customer_history` table.
**Type 6 (Hybrid 1 + 2 + 3):** Combines historical rows (Type 2) with a current attribute column (Type 1) across all historical records, enabling both point-in-time and current-state reporting from a single table.

---

### Q12: How does Data Vault 2.0 use Hash Keys to eliminate ETL loading dependencies between Hubs, Links, and Satellites?

**Difficulty:** `HARD` | **Niche:** `Data Vault 2.0 Modeling`

In traditional Data Vault 1.0, hubs generated integer surrogate keys (sequences), forcing links to wait for hubs to load before foreign keys could be resolved.
**Data Vault 2.0 (Hash Keys):**
- Replaces surrogate keys with cryptographic hashes (MD5 or SHA-256) of business keys: `HK_Customer = SHA256(UPPER(TRIM(tenant_id)) || '|' || UPPER(TRIM(customer_id)))`.
- Because hashes are deterministic and calculated in parallel, Hubs, Links, and Satellites can be loaded **simultaneously in parallel** with zero ETL dependency bottlenecks.

---

### Q13: How does the Transactional Outbox Pattern combined with Debezium CDC prevent dual-write data inconsistency?

**Difficulty:** `ARCHITECT` | **Niche:** `Change Data Capture (CDC)`

**The Dual-Write Problem:** Microservice writes to relational DB and publishes to Kafka. If Kafka fails after the DB commits, data is lost; if the DB fails after publishing, phantom events are created.
**Transactional Outbox Pattern:**
1. The business transaction writes business entity records and writes an event payload into an `outbox_table` inside the **same local ACID database transaction**.
2. **Debezium CDC:** Reads database transaction log (WAL / binlog) and extracts outbox table inserts into Kafka.
3. Eliminates dual-write inconsistencies with guaranteed at-least-once delivery.

---

### Q14: Compare Star Schema vs Snowflake Schema vs One Big Table (OBT) in modern cloud data warehouses.

**Difficulty:** `MEDIUM` | **Niche:** `Dimensional Modeling (Kimball)`

**1. Star Schema (Kimball):** De-normalized dimensions surrounding fact tables. Clean semantic model, fast query joins, balanced storage.
**2. Snowflake Schema:** Normalized dimensions (sub-dimensions branched out). Minimizes storage redundancy, but increases join overhead.
**3. One Big Table (OBT):** Completely pre-joined denormalized table with 100+ columns. Zero joins, blazing query performance on columnar engines (BigQuery/ClickHouse/Snowflake), at the cost of higher storage and complex update synchronization.

---

### Q15: What is Bi-Temporal Data Modeling, and why is it mandatory for regulated financial risk engines?

**Difficulty:** `ARCHITECT` | **Niche:** `SCD Types 1-6 Architecture`

Bi-Temporal modeling tracks two distinct timelines for every record:
1. **Valid Time (Business Time):** The time window during which the event was true in the real world (`valid_from`, `valid_to`).
2. **Transaction Time (System Time):** The time window during which the record was physically stored in the database system (`system_from`, `system_to`).
**Financial Audit Use Case:** A risk calculation executed on March 31st must be reproducible on December 31st, *even if retroactive corrections were entered in July*. System time allows querying the database exactly as it existed on March 31st.

---
