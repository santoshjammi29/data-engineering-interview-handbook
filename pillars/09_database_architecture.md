# Pillar 09: Database Architecture & Distributed Systems
**Category:** `DATABASE ARCHITECTURE` | **Domain:** `Database Internals & Distributed Storage`

---

### Q1: Compare OLTP (Row-Oriented) vs OLAP (Columnar) storage engines at the disk page, compression, and CPU level.

**Difficulty:** `HARD` | **Niche:** `Storage Engine Internals`

**Comparison Matrix:**
- **Disk Page Layout:** OLTP packs all columns of a single record contiguously on an 8KB data page (optimal for point reads/writes). OLAP packs values of a single column across thousands of records on contiguous pages.
- **Compression:** OLTP achieves modest compression (20-30%) due to heterogeneous column data types. OLAP achieves **$5\times \text{ to } 10\times$ compression** using dictionary encoding, run-length encoding (RLE), and bit-packing across homogeneous column values.
- **CPU Execution:** OLAP leverages vectorized SIMD CPU instructions to filter millions of values in CPU registers; OLTP is constrained by scalar pointer chasing.

---

### Q2: Compare B-Tree and Log-Structured Merge-Tree (LSM-Tree) storage engines in terms of Read and Write Amplification.

**Difficulty:** `HARD` | **Niche:** `B-Tree vs LSM-Tree`

**Comparison:**
- **B-Trees (PostgreSQL, MySQL, SQL Server):**
  - In-place updates on disk pages. High write amplification (writing 100 bytes modifies an entire 8KB page and WAL log).
  - Low read amplification: point lookups take $O(\log N)$ disk seeks.
- **LSM-Trees (Cassandra, RocksDB, ScyllaDB):**
  - Append-only writes. Writes hit in-memory MemTable and write-ahead log, flushing sequentially to immutable SSTables on disk (minimal write amplification, high write throughput).
  - Higher read amplification: reads must query MemTable and multiple SSTables via Bloom filters and merge-sort records.

---

### Q3: Explain the CAP and PACELC Theorems and classify PostgreSQL, Cassandra, and Spanner.

**Difficulty:** `HARD` | **Niche:** `Distributed Consistency Theorems`

**PACELC Theorem Definition:**
If there is a Partition (**P**), how does the system trade off Availability (**A**) vs Consistency (**C**); **E**lse, how does the system trade off Latency (**L**) vs Consistency (**C**)?
- **PostgreSQL (Traditional RDBMS):** PC/EC. Prioritizes consistency at all times.
- **Apache Cassandra:** PA/EL. Prioritizes availability during network partitions, and prioritizes low latency over immediate consistency in normal operation (eventual consistency).
- **Google Cloud Spanner:** PC/EC. True distributed CP system utilizing atomic clocks (TrueTime API) to guarantee external consistency with high availability across global regions.

---

### Q4: Explain Database Sharding Topologies (Range, Hash, Directory) and how Vitess implements VSchema sharding.

**Difficulty:** `ARCHITECT` | **Niche:** `Database Sharding & Vitess`

**Sharding Topologies:**
1. **Range Sharding:** Splits data by key ranges (e.g. A-D, E-H). Susceptible to write hotspotting on sequential keys (e.g. auto-increment IDs).
2. **Hash Sharding:** Computes `hash(key) % N`. Distributes writes uniformly; requires consistent hashing to avoid massive re-shuffling when nodes are added.
3. **Vitess VSchema Sharding:** Used to horizontally scale MySQL clusters. VSchema defines sharding keys and routes queries via VTGate proxies, enabling live dynamic re-sharding without application downtime.

---

### Q5: How does the Raft Consensus Algorithm establish leader election and log replication in distributed databases?

**Difficulty:** `HARD` | **Niche:** `Distributed Consensus`

Raft achieves distributed consensus through two core phases:
1. **Leader Election:** Nodes operate in Follower, Candidate, or Leader states. If followers miss leader heartbeats (election timeout 150-300ms), they transition to Candidate, increment term, and request votes. A candidate wins with a majority vote ($> N/2$).
2. **Log Replication:** All client writes go to the Leader. The Leader appends entries to its log and sends `AppendEntries` RPCs to followers. Once a majority of followers acknowledge writing to their local logs, the Leader commits the write and notifies followers.

---

### Q6: Explain Multi-Version Concurrency Control (MVCC) and why 'Readers never block Writers, and Writers never block Readers'.

**Difficulty:** `HARD` | **Niche:** `MVCC & Concurrency Control`

Instead of placing exclusive locks on rows during updates, MVCC creates a brand new version of the row with transaction timestamps (`xmin`, `xmax` in PostgreSQL):
- **Readers:** A reader takes a snapshot at transaction start and reads the latest version committed *before* its snapshot timestamp.
- **Writers:** Writers write new versions without locking existing versions.
- **Outcome:** Read queries never block update queries, and long-running reporting queries never block transactional writes.

---

### Q7: Why is connection pooling (e.g. PgBouncer, HikariCP) critical, and how do you implement admission control under load spikes?

**Difficulty:** `MEDIUM` | **Niche:** `Connection Pooling & Backpressure`

Every database connection consumes dedicated process memory (e.g. 5-10MB in PostgreSQL) and OS thread context switching. Thousands of concurrent client connections cause server thread thrashing and memory exhaustion.
- **Connection Pooling:** Maintains a warm pool of reusable connections (e.g. 50 connections serving 5,000 clients).
- **Admission Control:** Queues client requests when all pool connections are busy; rejects requests with HTTP 429 / backpressure when the queue reaches capacity, protecting the database engine from collapse.

---

### Q8: Explain the ARIES Crash Recovery Algorithm and Write-Ahead Logging (WAL) mechanics.

**Difficulty:** `ARCHITECT` | **Niche:** `Crash Recovery & ARIES`

**ARIES 3-Phase Crash Recovery:**
1. **Analysis Phase:** Scans the WAL forward from the last checkpoint to identify dirty pages in the buffer pool and active uncommitted transactions at the time of crash.
2. **Redo Phase:** Scans forward from the oldest uncommitted log record, replaying all logged changes (both committed and uncommitted) to restore database memory to the exact state before crash.
3. **Undo Phase:** Scans backward, undoing the operations of all transactions that were active and uncommitted at the time of crash, logging Compensating Log Records (CLRs) to ensure idempotency.

---

### Q9: Compare Two-Phase Commit (2PC) with the Saga Pattern for distributed transactions across microservices.

**Difficulty:** `ARCHITECT` | **Niche:** `Distributed Transactions`

**Comparison:**
- **Two-Phase Commit (2PC):** Coordinator orchestrates Prepare and Commit phases. Guarantees strict ACID consistency across nodes. However, locks resources across all participating databases throughout both phases, creating a single point of failure and severe latency bottlenecks.
- **Saga Pattern (Event-Driven / Choreography):** Breaks a distributed transaction into a sequence of local transactions. Each transaction updates its local database and publishes an event. If a step fails, the Saga executes **Compensating Transactions** to undo previous changes. Delivers eventual consistency without distributed database locks.

---

### Q10: How do you solve replication lag in Read-Replica architectures using Global Transaction Identifiers (GTID)?

**Difficulty:** `HARD` | **Niche:** `Replication Lag & GTID Pinning`

When users write data (e.g. update profile) and immediately refresh the page, read replicas might not have caught up with the primary, causing data to appear missing.
**GTID Pinning Solution:**
1. Primary returns the Global Transaction ID (GTID) of the committed write to the client/session.
2. When reading, the application proxy checks the replica's applied GTID.
3. If `replica_gtid < written_gtid`, the proxy automatically routes the read directly to the **Primary database** or pauses until the replica applies the GTID, guaranteeing Read-After-Write consistency.

---

### Q11: Compare B-Trees vs Log-Structured Merge (LSM) Trees across Read, Write, and Space Amplification.

**Difficulty:** `HARD` | **Niche:** `Storage Engines (B-Tree vs LSM)`

**1. B-Tree (PostgreSQL, MySQL InnoDB, SQL Server):**
- *Write Amplification (High):* In-place updates modify random 8KB disk pages. Updating a 20-byte row writes a full 8KB page + WAL buffer.
- *Read Amplification (Low):* $O(\log N)$ point lookups require scanning only a single leaf page.
- *Space Amplification (Moderate):* Fragmentation and fill-factor (typically 30% empty space).
**2. LSM-Tree (RocksDB, Cassandra, Bigtable):**
- *Write Amplification (Low in write path, high in compaction):* Appends to sequential in-memory MemTable and flushes immutable SSTables.
- *Read Amplification (High):* Must check Bloom filters, MemTable, and multiple SSTable levels.
- *Space Amplification (Low):* Compact, compressed sequential pages without internal page fragmentation.

---

### Q12: How does Multi-Version Concurrency Control (MVCC) eliminate read-write locking conflicts in relational databases?

**Difficulty:** `HARD` | **Niche:** `Concurrency Control & MVCC`

MVCC allows reads to never block writes, and writes to never block reads.
**Mechanism:**
- Modifying a row does not overwrite physical data. The database inserts a new row version with metadata tags: `xmin` (creation transaction ID) and `xmax` (deletion/superseding transaction ID).
- When Transaction $T$ reads, it evaluates an active transaction snapshot.
- **Snapshot Isolation:** $T$ reads versions where `xmin` is committed before $T$'s start and `xmax` is either uncommitted or created after $T$.
- **Garbage Collection (VACUUM):** A background worker purges historical row versions once no active transaction's snapshot requires them.

---

### Q13: Explain the 3 phases of the ARIES crash recovery algorithm: Analysis, Redo, and Undo.

**Difficulty:** `ARCHITECT` | **Niche:** `Crash Recovery (ARIES Protocol)`

ARIES (Algorithms for Recovery and Isolation Exploiting Semantics) uses Write-Ahead Logging (WAL) with physiological logging to restore ACID consistency after a sudden power loss.
1. **Analysis Phase:** Scans WAL forward from the latest checkpoint to reconstruct the Dirty Page Table (pages dirty in RAM at crash) and the Transaction Table (active uncommitted transactions).
2. **Redo Phase (Repeating History):** Replays all logged changes forward from the oldest un-flushed dirty page (`RecLSN`) to bring the database to the exact state it was in at the instant of the crash, including uncommitted updates.
3. **Undo Phase:** Scans backward through active uncommitted transactions and executes Compensation Log Records (CLRs) to roll back all incomplete transactions.

---

### Q14: How does distributed database sharding (e.g. Vitess or Citus) handle cross-shard joins and distributed transactions?

**Difficulty:** `ARCHITECT` | **Niche:** `Distributed Consensus & Sharding`

**Sharding Architecture:** Rows are partitioned across $K$ independent database instances using consistent hashing on a shard key (e.g. `hash(account_id) % K`).
**Cross-Shard Joins:**
- Queries filtering by the shard key are routed directly to the single target shard (Single-Shard Query).
- Cross-shard queries require the query router (`vtgate` in Vitess) to fan out queries to all shards, pull intermediate record batches, and execute a distributed merge join in router memory.
**Distributed Transactions:** Coordinated via **Two-Phase Commit (2PC)**: Prepare phase (shards vote YES/NO and lock WAL records) followed by Commit phase. Vitess avoids 2PC whenever possible by co-locating related entity tables on identical shard keys.

---

### Q15: How does the Raft consensus algorithm guarantee state machine replication and prevent split-brain?

**Difficulty:** `ARCHITECT` | **Niche:** `Distributed Consensus & Sharding`

Raft decomposes consensus into 3 independent sub-problems:
1. **Leader Election:** Nodes operate in 3 states: Follower, Candidate, Leader. If a follower misses leader heartbeats, it transitions to candidate, increments term number, and requests votes. Election requires a strict majority quorum ($Q = \lfloor N/2 \rfloor + 1$).
2. **Log Replication:** Leader accepts client writes, appends entry to its log, and sends `AppendEntries` RPC to followers. Once a majority acknowledges, entry is committed.
3. **Split-Brain Immunity:** Because a leader must hold a strict majority of votes and logs from the highest term, network partitions cannot form two competing majorities; the minority partition stalls writes, guaranteeing linearizable consistency.

---
