# Master-Snowflake
# Snowflake Knowledge Bible

This repository is a **comprehensive guide to Snowflake**, covering architecture, storage, compute, ingestion, security, performance, scenario-based use cases, and best practices for advanced developers and architects. Each category contains **50+ deep-dive Q&A** in a GitHub-friendly format.

---

## Table of Contents

1. [Architecture & Core Concepts](#1-architecture--core-concepts)  
2. [Storage & Micro-partitions](#2-storage--micro-partitions)  
3. [Compute & Virtual Warehouses](#3-compute--virtual-warehouses)  
4. [Data Ingestion & Bulk Loading](#4-data-ingestion--bulk-loading)  
5. [External Tables & Data Lake Integration](#5-external-tables--data-lake-integration)  
6. [Query Optimization & Performance](#6-query-optimization--performance)  
7. [Security & Governance](#7-security--governance)  
8. [Data Sharing & Collaboration](#8-data-sharing--collaboration)  
9. [Disaster Recovery & High Availability](#9-disaster-recovery--high-availability)  
10. [Advanced Features & Miscellaneous](#10-advanced-features--miscellaneous)  
11. [Scenario-Based & Advanced Use Cases](#11-scenario-based--advanced-use-cases)

---

## 1. Architecture & Core Concepts
- Snowflake architecture overview  
- Layers: Storage, Compute, Cloud Services  
- Differences from traditional data warehouses  
- Concurrency model and multi-cluster architecture  

## 2. Storage & Micro-partitions
- Micro-partitions and metadata  
- Automatic pruning and clustering keys  
- Time Travel and Fail-safe  
- Zero-copy cloning  
- Storage optimization and compression  

## 3. Compute & Virtual Warehouses
- Virtual warehouse types and sizing  
- Scaling: vertical and horizontal (multi-cluster)  
- Auto-suspend/resume and workload isolation  
- Multi-cluster warehouses and concurrency  
- Billing, cost optimization, and performance tuning  

## 4. Data Ingestion & Bulk Loading
- COPY command overview (conceptual)  
- Snowpipe and continuous ingestion  
- Bulk load strategies and best practices  
- File formats: CSV, JSON, Parquet, Avro, ORC  
- External stages (S3, Azure Blob, GCS)  
- Error handling, monitoring, and incremental load strategies  

## 5. External Tables & Data Lake Integration
- Querying external tables  
- Integrating with cloud data lakes (S3, ADLS, GCS)  
- Semi-structured data querying  
- Cost and performance considerations  
- Federated query patterns  

## 6. Query Optimization & Performance
- Metadata-based pruning  
- Result caching and query plan reuse  
- Clustering impact on performance  
- Optimizing large joins, selective queries  
- Performance tuning best practices  

## 7. Security & Governance
- RBAC: roles and privileges  
- Row-level and column-level security  
- Data masking and dynamic masking policies  
- Network security and encryption  
- Secure data sharing and access monitoring  

## 8. Data Sharing & Collaboration
- Secure data sharing concepts  
- Reader accounts and share permissions  
- Zero-copy data sharing  
- Cross-region and cross-cloud sharing  

## 9. Disaster Recovery & High Availability
- Failover and replication strategies  
- Time Travel vs Fail-safe  
- Cross-region replication for business continuity  
- Recovery planning and testing  

## 10. Advanced Features & Miscellaneous
- Streams and Tasks (CDC)  
- Materialized views  
- Stored procedures and user-defined functions  
- Semi-structured data support  
- Snowflake marketplace & data exchange  

## 11. Scenario-Based & Advanced Use Cases
- ETL vs ELT strategies  
- Incremental load strategies and CDC  
- On-premises to Snowflake migrations  
- Multi-table transformations and pipeline design  
- Data integration patterns and streaming/batch pipelines  
- Cost management in real workloads  
- Disaster recovery and compliance scenarios  
- Best practices for performance, scalability, and maintainability

Examples of Scenario-Based Interview Questions

How would you migrate a 10TB on-premises data warehouse to Snowflake?

Explain a strategy for performing incremental ETL using Streams and Tasks.

How would you design a multi-team warehouse setup to prevent workload interference?

Describe a method to handle semi-structured data during migration.

How do you monitor and optimize compute costs during large-scale batch loads?

How would you recover a table accidentally deleted in production using Time Travel?

What are best practices for moving historical partitions from legacy systems to Snowflake?

How would you handle schema changes in the source system during incremental load?

Discuss a scenario where ELT is better than ETL for analytics workloads.

How do you validate data quality and consistency after a migration?
