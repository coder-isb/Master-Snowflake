# Category 6: Query Optimization & Performance  
Comprehensive, Senior-Level Reference

This chapter covers **metadata pruning, caching layers, query planning, clustering, join optimization, scenarios, tradeoffs, common pitfalls, and best practices**.  
Every question is fully expanded to help readers develop strong internal reasoning for real-world Snowflake performance architecture.

---

## Table of Contents

1. [Basics / Fundamentals](#1-basics--fundamentals)  
2. [Metadata Pruning & Micro-partition Optimization](#2-metadata-pruning--micro-partition-optimization)  
3. [Caching & Reuse](#3-caching--reuse)  
4. [Query Planning & Optimizer Behavior](#4-query-planning--optimizer-behavior)  
5. [Clustering & Data Layout Impact](#5-clustering--data-layout-impact)  
6. [Join Optimization & Selective Query Performance](#6-join-optimization--selective-query-performance)  
7. [Performance Tuning Best Practices](#7-performance-tuning-best-practices)  
8. [Scenario-Based Questions](#8-scenario-based-questions)  
9. [Tradeoffs & Design Considerations](#9-tradeoffs--design-considerations)  

---

## 1. Basics / Fundamentals

### Q1: What is query optimization in Snowflake?
**Answer:**  
Query optimization is Snowflake’s process of analyzing a query, inspecting metadata, evaluating statistics, assembling an execution plan, and deciding the most efficient method to retrieve data. It leverages pruning, caching, columnar storage, and micro-partition metadata, enabling Snowflake to minimize scanned data and reduce compute cost with zero manual indexing.

### Q2: What are the major components that influence query performance?
**Answer:**  
- **Micro-partition metadata**  
- **Columnar compression**  
- **Clustering depth**  
- **Caching (result, local SSD, warehouse)**  
- **Warehouse size & concurrency**  
- **Join strategy selection**  
- **Pruning efficiency**  
- **Query plan optimization**

### Q3: Why doesn’t Snowflake require indexes?
**Answer:**  
Because Snowflake relies on **metadata-rich micro-partitions**, clustering statistics, and automatic pruning instead of traditional indexes. It prioritizes **data skipping** rather than data seeking.

### Q4: What are common bottlenecks in Snowflake queries?
**Answer:**  
- Large scans due to poor filtering  
- Poor clustering  
- Highly selective queries scanning many partitions  
- Wide joins with skewed distributions  
- Excessive semi-structured parsing  
- Overly small warehouse sizes for complex workloads  
- Unnecessary repeated computation due to cache invalidation  

---

## 2. Metadata Pruning & Micro-partition Optimization

### Q5: What is micro-partition pruning?
**Answer:**  
Pruning is Snowflake’s process of **skipping irrelevant micro-partitions** based on metadata (min/max, null counts, distinct ranges). Instead of scanning full files, Snowflake inspects metadata first, which significantly reduces I/O.

### Q6: Which queries benefit most from pruning?
**Answer:**  
Queries with **selective predicates** on:  
- Timestamps  
- High-cardinality keys  
- Numeric ranges  
- Partition-aligned characteristics  
Highly selective analytical queries perform exceptionally well when pruning is effective.

### Q7: What factors hurt pruning efficiency?
**Answer:**  
- Randomly distributed values  
- Unsorted ingestion patterns  
- Lack of clustering on large tables  
- Semi-structured fields without extracted values  
- Very small files causing fragmented partitions  

### Q8: How does Snowflake use metadata to avoid scanning?
**Answer:**  
Snowflake analyzes column-level metadata from micro-partitions to:  
- Compare predicate filters with stored min/max  
- Eliminate partitions without relevant ranges  
- Prioritize partitions with high hit potential  

### Q9: What is the relationship between micro-partitions and clustering?
**Answer:**  
Clustering improves the **logical ordering** of data, making partition metadata more efficient. When values are physically grouped, pruning becomes more effective.

### Q10: Can pruning be manually optimized?
**Answer:**  
Users cannot adjust partitions directly, but pruning can be enhanced by:  
- Choosing effective clustering keys  
- Maintaining ingestion order  
- Using fewer, larger files  
- Flattening semi-structured fields into columns  

---

## 3. Caching & Reuse

### Q11: What types of caching exist in Snowflake?
**Answer:**  
- **Result Cache:** Returns identical query results instantly.  
- **Warehouse Cache (Local SSD):** Stores micro-partitions recently read.  
- **Metadata Cache:** Remains global and is shared across warehouses.  

### Q12: What triggers result cache usage?
**Answer:**  
An identical query (same filters, same tables, unchanged data) will be instantly replayed using cached results.

### Q13: Limitations of result caching?
**Answer:**  
- Invalidated by any table modification  
- Warehouse cache is not shared between warehouses  
- Not applicable for queries with non-deterministic functions  

### Q14: How does warehouse cache improve performance?
**Answer:**  
Local SSD cache reduces the need to re-fetch micro-partitions from remote storage, significantly boosting performance for repetitive scans.

### Q15: How does metadata caching help?
**Answer:**  
Metadata happens at the **cloud services layer**, enabling faster planning, faster pruning decisions, and minimized latency before execution.

---

## 4. Query Planning & Optimizer Behavior

### Q16: What is Snowflake’s cost-based optimizer?  
**Answer:**  
Snowflake evaluates query execution cost based on statistics about rows, micro-partitions, cardinalities, and data distributions. It chooses the most efficient join strategy, pruning plan, and execution order.

### Q17: How does Snowflake build a query plan?
**Answer:**  
1. Parse query  
2. Analyze metadata  
3. Evaluate predicates  
4. Determine prune sets  
5. Select join strategies  
6. Create distributed execution plan  
7. Assign tasks to compute nodes  

### Q18: What join strategies exist?
**Answer:**  
- **Broadcast join** (smaller table replicated)  
- **Partitioned join** (large tables)  
- **Adaptive join selection** based on table sizes  

### Q19: When does Snowflake broadcast smaller tables?
**Answer:**  
When one table is significantly smaller, Snowflake will duplicate the smaller dataset across nodes to avoid large shuffles.

### Q20: How does semi-structured data affect planning?
**Answer:**  
Parsing nested structures increases CPU usage, slows filtering, and reduces pruning effectiveness unless key paths are extracted into relational columns.

---

## 5. Clustering & Data Layout Impact

### Q21: How does clustering improve performance?
**Answer:**  
By improving physical ordering inside micro-partitions, clustering makes metadata more aligned with filter patterns, enabling stronger pruning and faster scans.

### Q22: When is clustering beneficial?
**Answer:**  
- Large tables (>100M rows)  
- Queries filtering on specific selective dimensions  
- Time-series workloads  
- Analytical workloads with repeated slicing patterns  

### Q23: Tradeoffs of clustering?
**Answer:**  
- Increased storage and compute cost for reclustering  
- Overuse can lead to diminishing returns  
- Frequent inserts/updates can degrade clustering over time  

### Q24: Does clustering speed up joins?
**Answer:**  
Indirectly. It reduces scanned partitions and improves filtering before join operations occur.

### Q25: What is clustering depth?
**Answer:**  
A measure showing how aligned micro-partitions are with clustering key values. Higher depth indicates worse clustering and degraded pruning effectiveness.

---

## 6. Join Optimization & Selective Query Performance

### Q26: Why are large joins expensive?
**Answer:**  
Large joins involve shuffling data across compute nodes, evaluating join conditions, and performing distributed comparisons. Poor pruning or skew intensifies costs.

### Q27: How does Snowflake optimize large joins?
**Answer:**  
- Evaluates table sizes  
- Chooses broadcast vs partitioned strategy  
- Applies pruning before join  
- Reorders join conditions for minimal compute cost  

### Q28: How do selective queries benefit from Snowflake architecture?
**Answer:**  
Selective queries hit fewer partitions, making them extremely fast due to:  
- Pruning  
- Columnar reading  
- Metadata caching  
- SSD-level caching  

### Q29: How does data skew affect joins?
**Answer:**  
Uneven distributions cause certain nodes to process more data than others, slowing down the entire distributed plan.

### Q30: Best strategies to mitigate skew?
**Answer:**  
- Using distribution-friendly keys  
- Avoiding exploding joins  
- Extracting common filters  
- Pre-aggregating data if possible  

---

## 7. Performance Tuning Best Practices

### Q31: Start with the smallest warehouse and scale up as needed  
Smaller warehouses reduce cost and are usually sufficient for well-pruned queries. Scale only when CPU-heavy operations occur.

### Q32: Improve pruning before increasing warehouse size  
Better pruning reduces I/O more effectively than brute-force scaling.

### Q33: Avoid too many small files  
Small files produce fragmented micro-partitions, hurting pruning and metadata efficiency.

### Q34: Extract selective fields from semi-structured data  
Flattening key paths enables pruning and reduces parsing overhead.

### Q35: Pre-filter before joining  
Push selective predicates early in the plan for optimal pruning.

### Q36: Use clustering on large tables  
Especially when operational queries depend on specific dimensions or time-based ranges.

### Q37: Avoid unnecessary subqueries or transformations  
Reduces complexity and allows optimizer to detect simplification opportunities.

### Q38: Separate ingestion and analytical compute  
Avoid running heavy inserts or backfills on the same warehouse used for reporting.

---

## 8. Scenario-Based Questions

### Q39: A query is slow because it scans 90% of a large table. What do you do?  
**Answer:**  
Improve clustering on the columns used in filters, extract semi-structured fields into columns, ensure clean partitioning, and validate pruning effectiveness before scaling warehouse.

### Q40: Highly selective filters run slowly. Why?  
**Answer:**  
Selectivity alone does not guarantee speed if pruning is ineffective. Causes may include:  
- Values scattered across micro-partitions  
- No clustering  
- Random ingestion order  
- Stored in semi-structured formats  

### Q41: Analysts complain about inconsistent query performance. Cause?  
**Answer:**  
Different queries may or may not leverage:  
- Result cache  
- Warehouse cache  
- Metadata pruning  
Also, warehouse may scale down or be saturated with concurrent workloads.

### Q42: Heavy semi-structured parsing slowing queries  
**Answer:**  
Flatten frequently used fields and store them as typed columns; reduces CPU parsing and accelerates pruning.

### Q43: Queries slow after table backfill  
**Answer:**  
Backfill disrupts clustering. Run background reclustering and reorganize micro-partitions.

### Q44: Join between two huge tables runs slow  
**Answer:**  
- Ensure filters applied before join  
- Identify skewed join keys  
- Consider pre-aggregating one table  
- Evaluate whether one table can be significantly reduced

### Q45: Query executed quickly yesterday but slowly today  
Possible reasons:  
- Lost result cache due to data modification  
- Warehouse was suspended and cache reset  
- Query plan changed due to updated statistics  
- More users causing higher concurrency  

---

## 9. Tradeoffs & Design Considerations

### Q46: Tradeoff: Clustering vs No Clustering  
- **Pros:** Better pruning, faster reads, improved join performance  
- **Cons:** Increased maintenance cost for reclustering  

### Q47: Tradeoff: Larger warehouse vs better pruning  
- Increasing warehouse brute-forces compute at higher cost  
- Improving pruning reduces I/O and cost significantly  
Best practice: **Optimize pruning first, scale later.**

### Q48: Tradeoff: Semi-structured vs flattened data  
- Semi-structured is flexible, easy to ingest  
- Flattened data is faster, more prunable  
Decision depends on:  
- Query patterns  
- Expected volume  
- Frequency of access  

### Q49: Tradeoff: External tables vs internal tables  
- External is cheaper but slower  
- Internal is more expensive but enables full optimization  
Use external for cold data; internal for hot or frequently accessed data.

### Q50: Tradeoff: Multi-cluster warehouses vs separate warehouses  
- Multi-cluster improves concurrency without splitting workloads  
- Separate warehouses give stronger isolation  
Decision depends on workload overlap and governance policies.

### Q51: Tradeoff: More partition files vs fewer large files  
- Many small files → poor performance  
- Fewer large files → better parallelism and reduced overhead  

---

# End of Category 6  
Fully GitHub-ready, deeply detailed, scenario-driven reference.

