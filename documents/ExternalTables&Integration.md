# Snowflake External Tables & Data Lake Integration – Comprehensive Reference

This document covers **external tables, data lake integration, semi-structured querying, federated queries, cost/performance considerations, scenario-based questions, and tradeoffs**. Fully GitHub-ready.

---

## Table of Contents

1. [Basics / Fundamentals](#1-basics--fundamentals)  
2. [External Tables Overview](#2-external-tables-overview)  
3. [Data Lake Integration](#3-data-lake-integration)  
4. [Semi-Structured Data Querying](#4-semi-structured-data-querying)  
5. [Federated Query Patterns](#5-federated-query-patterns)  
6. [Cost & Performance Considerations](#6-cost--performance-considerations)  
7. [Scenario-Based Questions](#7-scenario-based-questions)  
8. [Best Practices / Tips](#8-best-practices--tips)  
9. [Key Interview Points](#9-key-interview-points)  

---

## 1. Basics / Fundamentals

### Q1: What is an external table in Snowflake?  
**Answer:**  
An external table allows querying data stored in **cloud storage** (S3, ADLS, GCS) **without physically loading it into Snowflake**. Metadata is managed by Snowflake, while data remains in the external source.

### Q2: Difference between external tables and internal tables?  
**Answer:**  
- **Internal tables:** Data stored in Snowflake storage; high performance, fully transactional.  
- **External tables:** Data remains in cloud storage; good for large, rarely updated datasets; query performance depends on external system.

### Q3: Use cases for external tables?  
**Answer:**  
- Data lakes with large historical datasets.  
- Sharing raw data across multiple systems.  
- ETL staging zones.  
- Federated querying without duplication.

### Q4: Supported file formats for external tables?  
**Answer:**  
CSV, JSON, Avro, Parquet, ORC. Columnar formats (Parquet/ORC) are recommended for query performance.

---

## 2. External Tables Overview

### Q5: How are external tables queried?  
**Answer:**  
Using standard SQL in Snowflake, similar to internal tables. Snowflake reads **metadata first**, then queries underlying files in cloud storage.

### Q6: How does Snowflake maintain metadata?  
**Answer:**  
Stores table definitions, partitions, and file metadata in the **cloud services layer**, enabling partition pruning and parallel reading.

### Q7: What is partition pruning in external tables?  
**Answer:**  
Only the relevant partitions/files are scanned during queries, reducing read volume and improving performance.

### Q8: Can external tables be incremental?  
**Answer:**  
Yes, by organizing storage in partitioned directories or timestamp-based folders, queries can target new files only.

### Q9: Can external tables support ACID transactions?  
**Answer:**  
No, external tables do not support full ACID; consistency depends on the underlying storage. Snowflake metadata ensures query correctness but not transactional updates.

---

## 3. Data Lake Integration

### Q10: How does Snowflake integrate with cloud data lakes?  
**Answer:**  
- **S3 (AWS)**, **ADLS (Azure)**, **GCS (Google Cloud)** supported.  
- Use external stages to reference storage buckets/containers.  
- External tables query data directly; Snowflake can combine with internal tables for analytics.

### Q11: What are the benefits of integrating with data lakes?  
**Answer:**  
- Avoids duplicating large datasets.  
- Enables analytics directly on raw files.  
- Supports hybrid storage and multi-cloud strategies.

### Q12: Tradeoffs of querying directly on data lakes?  
**Answer:**  
- Pros: No copy, reduced storage cost, access to latest files.  
- Cons: Slower queries vs internal tables, depends on external storage performance, may incur cloud egress costs.

### Q13: How are credentials handled?  
**Answer:**  
Use cloud-native authentication (IAM roles, service principals, OAuth tokens) to securely access storage.

### Q14: Can external tables join with internal tables?  
**Answer:**  
Yes, Snowflake can perform **hybrid queries** combining internal and external tables. Tradeoff: external data reads may slow overall query.

---

## 4. Semi-Structured Data Querying

### Q15: Can external tables store semi-structured data?  
**Answer:**  
Yes, JSON, Avro, Parquet, ORC are supported. Snowflake can query nested structures using **VARIANT**.

### Q16: How is semi-structured data queried?  
**Answer:**  
Snowflake uses **flattening functions and path-based access** to parse nested arrays and objects. External table metadata allows partition pruning.

### Q17: Performance tradeoffs  
**Answer:**  
- Parsing nested data in external tables is slower than internal tables.  
- Use columnar formats and partitioning to reduce scanning.  

---

## 5. Federated Query Patterns

### Q18: What is federated querying?  
**Answer:**  
Federated queries combine **internal and external tables**, or data across different Snowflake accounts or cloud storage locations.

### Q19: Use cases for federated queries  
**Answer:**  
- Combine raw historical data (external table) with curated internal data.  
- Multi-cloud analytics without copying data.  
- Join on-the-fly for ETL/ELT pipelines.

### Q20: Tradeoffs of federated queries  
**Answer:**  
- Pros: Avoids duplication, enables real-time analysis.  
- Cons: Slower query performance due to external storage, potential higher cloud egress costs.

---

## 6. Cost & Performance Considerations

### Q21: Cost factors  
**Answer:**  
- Compute warehouse time to read external files.  
- Cloud egress or storage access costs.  
- Number of files and size impact query cost due to metadata and scanning.

### Q22: Performance factors  
**Answer:**  
- File format: columnar (Parquet/ORC) vs row-based (CSV).  
- File size: smaller files increase overhead; larger files reduce parallelism.  
- Partitioning: proper folder/partition strategy improves pruning.  
- Network bandwidth and cloud storage latency.

### Q23: Tradeoffs summary  
**Answer:**  
- Lower storage cost using external tables vs slower queries.  
- Using internal tables increases cost/storage but maximizes query performance.  
- Federated queries reduce duplication but may increase latency and compute consumption.

---

## 7. Scenario-Based Questions

### Q24: Large historical dataset in S3 (10TB)  
**Answer:**  
Use external table referencing Parquet files. Benefits: avoid copying, storage savings. Tradeoff: slower query performance, ensure proper partitioning.

### Q25: Ad-hoc analytics combining Snowflake internal tables and external tables  
**Answer:**  
Federated query allows joining internal curated tables with external raw files. Tradeoff: query latency may increase, plan compute size accordingly.

### Q26: Multi-cloud storage integration  
**Answer:**  
External tables in S3 + ADLS queried simultaneously using Snowflake. Tradeoff: higher management complexity and egress costs.

### Q27: Semi-structured logs in external tables  
**Answer:**  
Use VARIANT columns and flatten functions. Tradeoff: parsing large nested data can be compute-intensive.

### Q28: Incremental query on external table  
**Answer:**  
Organize data in timestamped folders, query only new partitions. Tradeoff: need consistent folder naming and metadata tracking.

---

## 8. Best Practices / Tips

### Q29: Prefer columnar formats (Parquet/ORC)  
Reduces scan volume, improves pruning, and speeds up queries.

### Q30: Partition external data  
Use meaningful partitions (date, region, category) to improve pruning and reduce scanned data.

### Q31: Use external tables for rarely updated or append-only data  
Avoid updates on external tables to maintain performance.

### Q32: Combine external and internal tables judiciously  
Perform joins only when necessary; consider ETL to internal tables for frequently queried data.

### Q33: Monitor cloud storage cost  
Track egress, storage, and query compute to optimize total cost.

---

## 9. Key Interview Points

### Q34: Differences internal vs external tables  
Internal: high performance, transactional, higher storage cost.  
External: lower cost, no transaction support, good for large historical datasets.

### Q35: Performance optimization strategies  
Columnar formats, partitioning, proper warehouse sizing, pruning, minimizing nested parsing.

### Q36: Federated query considerations  
Use only when needed, aware of latency and compute cost.

### Q37: Scenario-based tradeoffs  
Balancing cost, query performance, and real-time access needs. E.g., internal copy for frequent queries, external for historical cold data.

### Q38: Incremental load on external tables  
Partition-based queries reduce scanned data and compute usage.

### Q39: Semi-structured querying  
Flatten nested structures carefully; prefer VARIANT for dynamic fields; use pruning to avoid full scan.

