# Snowflake Architecture & Key Concepts

This repository contains a structured **Q&A guide on Snowflake Architecture**, designed for learning, reference, and interview preparation.

---

## 1. Snowflake Architecture Overview

### Q1: Explain Snowflake architecture. How does it differ from traditional data warehouses?

**Answer:**  

Snowflake is a **cloud-native data platform** with a **multi-cluster, shared-data architecture** consisting of three main layers:

1. **Database Storage Layer** – Stores data in **columnar, compressed micro-partitions** within cloud object storage.  
2. **Compute Layer (Virtual Warehouses)** – Independently scalable compute clusters that read/write from storage **without blocking each other**.  
3. **Cloud Services Layer** – Coordinates **query parsing, optimization, security, metadata**, and transaction management.  

**Difference from traditional data warehouses:**  
- Snowflake **separates compute from storage**, allowing independent scaling.  
- Features like **auto-suspend/resume** reduce costs.  
- Supports **concurrent workloads without contention**.

**Follow-up Q&A:**

- **Q:** How does this separation benefit cost and performance?  
  **A:** Compute resources can be scaled per workload, paying only for active usage. Concurrent queries run independently without resource contention.

- **Q:** What is Snowflake’s concurrency model?  
  **A:** Each virtual warehouse operates independently. Multiple warehouses can access the same storage layer, enabling **unlimited concurrency** without performance degradation.

---

### Q2: What are micro-partitions, and why are they important?

**Answer:**  

- **Micro-partitions** are **16MB compressed columnar data blocks** used for storage.  
- Each partition stores **metadata** (column min/max, null counts, distinct values).  
- This enables **automatic partition elimination** to prune irrelevant data during query execution, improving performance.

**Follow-up Q&A:**

- **Q:** How does clustering interact with micro-partitions?  
  **A:** Clustering keys reorganize micro-partitions to improve pruning for large datasets. Optional but essential for selective queries.  

- **Q:** What are best practices for micro-partitioning?  
  **A:** Avoid over-clustering small tables; choose high-cardinality columns for clustering; monitor clustering depth to reduce maintenance cost.

---

## 2. Cloud Services Layer

### Q3: Explain the role of the cloud services layer in query execution.

**Answer:**  

- Handles **query parsing, optimization, and transaction management**.  
- Coordinates compute clusters to access storage.  
- Manages **metadata, security, access control**, and ensures **ACID compliance**.

**Follow-up Q&A:**

- **Q:** How does Snowflake ensure ACID transactions in a distributed system?  
  **A:** Metadata is centralized, and transactions are coordinated via a commit protocol, guaranteeing atomicity across clusters.

- **Q:** What happens if a query fails mid-execution?  
  **A:** The cloud services layer rolls back partial writes using metadata and micro-partition logs, ensuring a consistent state.

---

## 3. Compute Layer (Virtual Warehouses)

### Q4: What are virtual warehouses, and how do they scale?

**Answer:**  

- Virtual warehouses are independent **MPP compute clusters**.  
- Can scale **vertically** (X-Small → 6X-Large) or **horizontally** (multi-cluster warehouses) for concurrency.  
- **Auto-suspend/resume** reduces compute costs.

**Follow-up Q&A:**

- **Q:** How do multi-cluster warehouses handle concurrency?  
  **A:** If one cluster is busy, additional clusters spin up automatically to serve concurrent queries.  

- **Q:** How does billing work for multi-cluster warehouses?  
  **A:** You pay only for **active clusters** based on size and duration; idle clusters do not incur cost.

---

### Q5: How does Snowflake handle workload isolation?

**Answer:**  

- Each warehouse is **isolated**; queries in one do not affect others.  
- Storage is **shared but read-only**, allowing multiple warehouses to query simultaneously without conflict.

**Follow-up Q&A:**

- **Q:** Can multiple warehouses update the same table concurrently?  
  **A:** Yes, Snowflake supports ACID transactions. Updates are serialized via the cloud services layer to ensure consistency.

---

## 4. Storage Layer & Time Travel

### Q6: Explain Time Travel and Fail-safe. How do they differ?

**Answer:**  

- **Time Travel:** Query or recover data for **1–90 days** depending on edition.  
- **Fail-safe:** **7-day disaster recovery** managed by Snowflake.  

**Difference:** Time Travel is user-accessible; Fail-safe is Snowflake-managed.

**Follow-up Q&A:**

- **Q:** How is storage impacted by Time Travel?  
  **A:** Deleted or modified data is retained during the Time Travel period; old micro-partitions are not immediately purged.  

- **Q:** Can Time Travel be used across cloned tables?  
  **A:** Yes, zero-copy clones retain historical data independently of the source.

---

### Q7: What is zero-copy cloning, and how is it implemented?

**Answer:**  

- Creates a **logical copy** without duplicating storage.  
- Uses **micro-partition pointers**; only changes consume additional storage.

**Follow-up Q&A:**

- **Q:** Can you clone a database with ongoing writes?  
  **A:** Yes, the clone captures the current snapshot; subsequent writes in the source do not affect the clone.

---

## 5. Query Optimization and Metadata

### Q8: How does Snowflake optimize queries?

**Answer:**  

- Uses **metadata on micro-partitions** (min/max, null counts, distinct values).  
- **Automatic pruning** of irrelevant partitions.  
- **Cost-based optimization**.  
- **Result caching** and query plan reuse.

**Follow-up Q&A:**

- **Q:** What is result caching?  
  **A:** Query results are cached for 24 hours; identical queries read from cache instead of re-executing.  

- **Q:** How do clustering keys affect query performance?  
  **A:** Reduce the number of micro-partitions scanned for selective queries, improving performance but requiring maintenance (RECLUSTER).

---

## 6. Security Architecture

### Q9: Explain Snowflake security model.

**Answer:**  

- **RBAC** for roles and privileges.  
- Column-level and row-level security.  
- Data masking policies.  
- Network security (**TLS 1.2+ in-transit, AES-256 at rest**).  
- Secure Data Sharing uses **tokenized access** without copying data.

**Follow-up Q&A:**

- **Q:** Can you share a table with another Snowflake account securely?  
  **A:** Yes, using **Secure Data Sharing**; data remains in your account, only metadata and pointers are shared.

---

