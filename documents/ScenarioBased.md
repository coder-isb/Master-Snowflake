# Category 11: Scenario-Based & Advanced Use Cases  
Deep Architectural, Migration, Pipeline & Real-World Solutioning

This category covers end-to-end Snowflake solution architecture, CDC/SCD design,
incremental pipelines, on-prem migrations, hybrid ingestion, workload segregation,
cost optimization, and high availability strategies.  
Every question includes architecture-level reasoning and real-world tradeoffs.

---

## Table of Contents

1. ETL vs ELT Strategies  
2. Incremental Loads, CDC & Change Management  
3. Multi-Table Transformations & Workflow Design  
4. On-Prem to Snowflake Migrations  
5. Real-World Data Integration Patterns  
6. Cost Management in Production Workloads  
7. Disaster Recovery & Continuity  
8. Best Practices, Scalability, Maintainability  
9. Scenario Questions (Additional Deep Scenarios Included)  
10. Tradeoffs  
11. **Extended Scenario-Based Interview Questions (Requested Section)**  

---

# 1. ETL vs ELT Strategies

### Q1: What’s the difference between ETL and ELT in Snowflake?  
**Answer:**  
ETL transforms data *before* it reaches Snowflake, typically on external servers.  
ELT loads raw data directly into Snowflake, where transformations occur using elastic compute.  
Snowflake’s architecture—separation of compute/storage, parallel engines, micro-partitions—makes ELT faster, cheaper, and easier to scale.

### Q2: When would ETL still be useful?  
**Answer:**  
- Data must be masked before entering the cloud.  
- Legacy systems have complex procedural logic.  
- Regulatory restrictions prohibit raw data storage.  
- Heavy transformations unsuitable for SQL.

### Q3: Why does ELT dominate Snowflake workloads?  
**Answer:**  
Snowflake warehouse scaling, columnar storage, cluster pruning, and automatic optimization allow massive transformations to be computed efficiently. External ETL engines often cost more and bottleneck computation.

### Q4: How to architect hybrid ETL + ELT pipelines?  
**Answer:**  
Perform minimal cleanup outside Snowflake → land raw or semi-raw data → apply heavy logic, dimensional modeling, SCD, CDC, aggregation, and consumption-layer transformations inside Snowflake.

### Q5: Which workloads require pure ELT?  
**Answer:**  
- Fact table transformations  
- Large joins  
- Data warehousing/star schemas  
- Feature engineering for ML  
- Streaming merges  

---

# 2. Incremental Loads, CDC & Change Management

### Q6: What is incremental loading?  
**Answer:**  
Loading only new or changed records based on timestamps, version columns, or change logs. Reduces cost, compute, and processing time.

### Q7: What are common incremental load drivers?  
**Answer:**  
- Updated_at timestamps  
- Log-based CDC offsets  
- Surrogate version keys  
- Internal streams for consumed deltas

### Q8: What is CDC?  
**Answer:**  
CDC (Change Data Capture) tracks inserts, updates, and deletes from source systems.  
Snowflake supports CDC via Streams, external tools, and metadata-driven patterns.

### Q9: When to use SCD Type 1?  
**Answer:**  
When only the latest value matters and no historical preservation is required.

### Q10: When to use SCD Type 2?  
**Answer:**  
When business requires full historical traceability. Maintains multiple versions with effective date ranges.

### Q11: How to handle high-volume CDC loads?  
**Answer:**  
- Append-only staging  
- Streams for delta detection  
- Micro-batch processing  
- Separate ingestion and transformation warehouses  
- Partition by ingestion timestamps

### Q12: Why is CDC difficult in migration projects?  
**Answer:**  
Legacy systems may not retain full logs or timestamps. Often require synthetic CDC using hashing or snapshot-based delta computation.

### Q13: What is a late arriving update?  
**Answer:**  
A change that arrives after newer data is already processed. Must adjust SCD windows or correct history retrospectively.

### Q14: Micro-batch vs streaming CDC tradeoff?  
**Answer:**  
Streaming offers near real-time latency but costs more due to continuous compute.  
Micro-batching reduces cost but increases latency.

---

# 3. Multi-Table Transformations & Workflow Design

### Q15: How multi-table transformations run in Snowflake?  
**Answer:**  
Using tasks, streams, and scheduling DAGs where dependencies ensure correct ordering (e.g., dimension loads before fact loads).

### Q16: Why dependency-based tasks?  
**Answer:**  
Guarantee deterministic sequencing, prevent partial updates, reduce manual orchestration.

### Q17: How to avoid race conditions?  
**Answer:**  
- Isolated warehouses  
- Single-writer pattern  
- Stream consumption semantics  
- Serial task chains

### Q18: Multi-step transformation design?  
**Answer:**  
Structured into stages: ingest → CDC → dimension → fact → aggregates → analytics models.

### Q19: Challenges in multi-hop processing?  
**Answer:**  
Latency accumulation, schema evolution impact, dependency propagation, and cost escalation if steps are not isolated.

---

# 4. On-Prem to Snowflake Migrations

### Q20: Major stages of migration?  
**Answer:**  
Assessment → Schema conversion → Bulk load → CDC setup → Validation → Optimization → Cutover.

### Q21: Why migrate from Oracle/Netezza/Teradata?  
**Answer:**  
Lower admin overhead, scalable compute, reduced cost, automatic micro-partitioning, better concurrency, better elasticity.

### Q22: Biggest migration challenge?  
**Answer:**  
Procedural code (PL/SQL, NZ SQL, Teradata BTEQ) must be rewritten, rearchitected, or decomposed into Snowflake-native patterns.

### Q23: How to migrate large historical datasets?  
**Answer:**  
Parallel extract and cloud staging with multi-cluster loads into Snowflake.

### Q24: How to migrate CDC?  
**Answer:**  
Through log-based CDC tools or timestamp-based synthetic CDC if logs unavailable.

---

# 5. Real-World Data Integration Patterns

### Q25: Hybrid batch + streaming ingestion pattern?  
**Answer:**  
Streaming for recent events with batch snapshots for completeness. Merged together using business keys or event timestamps.

### Q26: IoT ingestion architecture?  
**Answer:**  
Events push into streaming queues → micro-batch landing → Snowpipe auto-ingest → flattening logic.

### Q27: Preventing duplicates?  
**Answer:**  
Business keys, metadata fields, dedup logic, stream watermarking.

### Q28: Data mesh patterns?  
**Answer:**  
Domain-based datasets, independent warehouses, secure data sharing.

### Q29: ML integration?  
**Answer:**  
Feature stores, ELT transformations, materialized views for feature reuse.

---

# 6. Cost Management in Production Workloads

### Q30: What causes cost spikes?  
**Answer:**  
Long-running warehouses, excessive clustering, multi-cluster misconfig, repeated full scans, expensive semi-structured workloads.

### Q31: Optimize ingestion cost?  
**Answer:**  
Tune file sizes, auto-suspend, ingest-dedicated warehouses, Streams to limit reprocessing.

### Q32: Optimize transformation cost?  
**Answer:**  
Right-size warehouses, prune operations, break monolithic logic.

### Q33: Optimize query cost?  
**Answer:**  
Result caching, materialized views, warehouse scaling strategy.

---

# 7. Disaster Recovery & Continuity

### Q34: Snowflake DR mechanisms?  
**Answer:**  
Cross-region replication → failover/failback → Time Travel → Fail-safe.

### Q35: When use region replication?  
**Answer:**  
Strict RTO/RPO requirements and global services.

### Q36: Time Travel vs Fail-safe?  
**Answer:**  
Time Travel is fast self-service recovery.  
Fail-safe is Snowflake-operated catastrophic recovery.

### Q37: DR testing strategy?  
**Answer:**  
Regular failover validations, data checks, dependency checks, re-run pipelines on secondary regions.

### Q38: Compliance considerations?  
**Answer:**  
Encryption, audit logs, masking policies, RLS/CLS, data retention.

---

# 8. Best Practices, Scalability & Maintainability

### Q39: Architect scalable pipelines?  
**Answer:**  
Multiple warehouses, modular ELT, Streams, and domain-based data zones.

### Q40: Maintainability?  
**Answer:**  
Version control, proper naming, documentation, monitoring dashboards.

### Q41: Reduce transformation technical debt?  
**Answer:**  
Break monolithic logic, materialize intermediate steps, use tasks/streams, document business semantics.

---

# 9. Scenario Questions

### Q42: Incremental loads for 500+ tables?  
**Answer:**  
Metadata-driven design using configuration tables → Streams for delta → dynamic Tasks → abstracted merge logic.

### Q43: Analyst performance complaints post-migration?  
**Answer:**  
Cluster/partition analysis → re-clustering → materialized views → isolation warehouses.

### Q44: Multi-region architecture for 24/7 business?  
**Answer:**  
Cross-region replication → domain-level shares → decoupled ingestion vs consumption → regional failover tests.

### Q45: Streaming + batch ingestion with deduplication?  
**Answer:**  
Unified model using business keys → micro-batch merges → event timestamps.

### Q46: Oracle SCD2 migration scenario?  
**Answer:**  
Migrate historical tables with validity → create SCD2 pipeline → use streams for incremental SCD2 adjustments.

### Q47: No CDC in source system?  
**Answer:**  
Synthetic CDC: hash diff → timestamp deltas → full snapshot with change detection.

### Q48: GDPR right-to-forget?  
**Answer:**  
PII catalog → masking → soft-delete patterns → downstream propagation → lineage tracking.

---

# 10. Tradeoffs

### Q49: Batch vs Streaming  
Batch is cheaper, simpler. Streaming is fresher, higher cost, more complexity.

### Q50: SCD1 vs SCD2  
SCD1 smaller and simpler; SCD2 more expensive but provides history.

### Q51: Multi-cluster vs Multiple Warehouses  
Multi-cluster auto-scales but may lead to spikes; separate warehouses give better isolation.

### Q52: Materialized Views vs Tasks  
MVs automated but costly with high churn; tasks provide full control but need orchestration.

### Q53: Rewrite vs Lift-and-Shift migrations  
Rewrite modernizes but slow; lift-and-shift is fast but carries legacy inefficiencies.

---

# 11. Extended Scenario-Based Interview Questions  
(Your explicitly requested additions with deep explanations)

### Q54: How would you migrate a 10TB on-premises warehouse to Snowflake?  
**Answer:**  
A 10TB migration requires a phased, highly parallelized design:  
- **Assessment:** Identify table sizes, CDC availability, data types, dependencies, incompatible SQL.  
- **Schema Conversion:** Convert to Snowflake-compatible structures. Decide on clustering and data zones.  
- **Bulk Data Movement:** Parallel extract into cloud storage using chunking and compression. Load using multi-cluster compute.  
- **CDC Enablement:** Continuous updates captured either via log-based CDC or synthetic CDC.  
- **Validation:** Count checks, profiling, reconciling aggregates, key uniqueness.  
- **Cutover:** Freeze updates, sync final CDC, deploy tasks, reroute BI tools.  
Critical challenges include maintaining referential integrity, high-speed transfers, and minimizing business downtime.

### Q55: Explain incremental ETL using Streams and Tasks.  
**Answer:**  
Streams track row-level changes since the last consumption. Tasks execute at defined intervals or dependencies.  
Pipeline:  
1. Data lands in staging.  
2. Stream captures inserts, updates, deletes.  
3. Task executes merge logic to apply deltas into target tables.  
4. Stream automatically advances its offset.  
Benefits: deterministic CDC, micro-batching, automated dependency execution, and minimal reprocessing.  
Tradeoffs: requires careful handling of high-volume updates and ensuring only one writer processes a stream.

### Q56: How would you design a multi-team warehouse setup without workload interference?  
**Answer:**  
Use **separate warehouses per team** or workload domain. Apply:  
- Resource monitors to cap runaway costs.  
- RBAC-based role isolation to prevent cross-team impact.  
- Virtual warehouses with size aligned to workload patterns.  
- Query queues disabled by right-sizing compute.  
This isolates spikes, ensures predictable SLAs, and avoids noisy-neighbor problems.

### Q57: Method to handle semi-structured data during migration?  
**Answer:**  
Semi-structured data (JSON/XML) from on-prem is often poorly optimized. Migration approach:  
- Land raw files in cloud storage.  
- Ingest as VARIANT into Snowflake.  
- Flatten and restructure using ELT transformations.  
- Pre-compute frequent paths into typed columns for performance.  
- Optionally cluster based on high-use fields.  
Ensures flexibility while maintaining query performance.

### Q58: How do you monitor and optimize compute during large batch loads?  
**Answer:**  
Monitor warehouse credit burn, load latency, queueing, and micro-partition efficiency.  
Optimization:  
- Right-size warehouses dynamically.  
- Enable auto-suspend aggressively.  
- Tune file sizes to reduce overhead.  
- Use clustering sparingly and only after stable workloads.  
- Stagger heavy loads to avoid contention.  
Balancing throughput vs cost is the core strategy.

### Q59: How to recover a table deleted in production using Time Travel?  
**Answer:**  
Time Travel allows restoring the table to a state within its retention window.  
Steps conceptually include: identify deletion timestamp → restore previous state → validate → re-enable dependencies.  
Important considerations: retention duration, downstream consistency, and restoring dependent objects.

### Q60: Best practices for moving historical partitions?  
**Answer:**  
- Move oldest partitions first to reduce risk.  
- Validate each partition independently before merging.  
- Apply clustering on high-cardinality date fields.  
- Snapshot older partitions to reduce future maintenance.  
Ensures integrity of large datasets while minimizing cost.

### Q61: How would you handle schema changes in the source during incremental load?  
**Answer:**  
Schema evolution strategy:  
- Use schema drift detection pipelines.  
- Add new columns as nullable to maintain backward compatibility.  
- For breaking changes, create versioned models or staging tables.  
- Update downstream tasks, streams, and validations.  
Ensures no pipeline failures while allowing controlled evolution.

### Q62: Scenario where ELT is better than ETL?  
**Answer:**  
Analytics workloads requiring heavy joins, aggregations, window functions, or semi-structured flattening benefit significantly from Snowflake’s compute engine. Raw staging + transformation inside Snowflake eliminates bottlenecks, centralizes logic, and uses elastic compute.

### Q63: How to validate data quality and consistency after migration?  
**Answer:**  
Validation includes:  
- Row-level counts  
- Checksums or hash diffs  
- Referential integrity checks  
- Profile comparison (cardinality, null %, range checks)  
- Business rule validation  
- BI report validation for end-user semantics  
Multiple reconciliation layers ensure enterprise-grade accuracy.

---

# End of Category 11
A complete reference for Snowflake architecture, migrations, pipelines, CDC/SCD, multi-team design, cost optimization, DR, and real-world data engineering scenarios.

