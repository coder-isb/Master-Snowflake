# Category 9: Disaster Recovery (DR) & High Availability (HA)  
Deep Dive for Architects, Data Leaders & Interview Preparation

Snowflake provides multiple layers of reliability including **automatic HA, cross-region replication, Time Travel, Fail-safe, and account-level failover/failback**.  
This guide covers DR strategies, high availability behavior, multicloud implications, failover scenarios, and senior-level tradeoffs.

---

## Table of Contents

1. [Fundamentals](#1-fundamentals)  
2. [High Availability Architecture](#2-high-availability-architecture)  
3. [Time Travel & Fail-safe](#3-time-travel--fail-safe)  
4. [Cross-Region & Cross-Cloud DR](#4-cross-region--cross-cloud-dr)  
5. [Failover & Failback](#5-failover--failback)  
6. [Testing & Automation](#6-testing--automation)  
7. [Scenarios](#7-scenarios)  
8. [Tradeoffs](#8-tradeoffs)  
9. [Cost Considerations](#9-cost-considerations)  
10. [Best Practices](#10-best-practices)  

---

# 1. Fundamentals

### Q1: What is Disaster Recovery (DR) in Snowflake?  
**Answer:**  
A set of features and processes that ensure Snowflake workloads can continue or be restored after a regional or operational failure. DR spans storage durability, metadata protection, cross-region replication, failover capabilities, and Time Travel/Failsafe.

### Q2: What does High Availability (HA) mean in Snowflake?  
**Answer:**  
HA ensures uninterrupted service during node failures or service disruptions within the same region. Snowflake automatically replaces failed nodes and maintains availability without user intervention.

### Q3: How does Snowflake ensure durability?  
**Answer:**  
Data is stored redundantly in cloud object storage, replicated across multiple availability zones (AZs), and protected using metadata logs.

### Q4: What does Snowflake manage automatically for HA?  
**Answer:**  
Node failures, cluster restarts, retry logic, availability zone outages, and query re-execution safeguards.

---

# 2. High Availability Architecture

### Q5: How does Snowflake achieve compute availability?  
**Answer:**  
Warehouses can restart automatically on new hardware when nodes fail, ensuring minimal disruption.

### Q6: What happens when a query node fails during execution?  
**Answer:**  
Snowflake re-runs affected portions of the query using DAG-based execution, providing fault tolerance.

### Q7: How does Snowflake ensure service continuity?  
**Answer:**  
The cloud services layer is distributed and auto-healing; metadata services have multiple replicas.

### Q8: Are warehouses replicated across AZs?  
**Answer:**  
Yes. Compute nodes are allocated from multiple AZs to minimize regional failure impact.

### Q9: Can users configure HA settings?  
**Answer:**  
No. HA is fully automated. Users configure only DR settings such as replication and failover groups.

### Q10: What types of outages does Snowflake HA protect against?  
**Answer:**  
Hardware failures, single AZ outages, node restarts, transient service failures, and corrupted micro-partition reads.

---

# 3. Time Travel & Fail-safe

### Q11: What is Time Travel?  
**Answer:**  
A retention feature allowing recovery of historical table versions for a configurable period. Useful for logical errors or accidental deletions.

### Q12: What is Fail-safe?  
**Answer:**  
A 7-day non-configurable recovery period after Time Travel expires, intended only for disaster scenarios and managed by Snowflake backend teams.

### Q13: Difference between Time Travel and Fail-safe?  
**Answer:**  
- **Time Travel:** User-accessible, for correcting user errors, short retention.  
- **Fail-safe:** Snowflake-accessible only, for system-level disaster recovery.  
Fail-safe should not be part of routine recovery planning.

### Q14: Can Time Travel be used for regional disasters?  
**Answer:**  
No. Time Travel works within the same Snowflake region; cross-region availability requires replication.

### Q15: Can semi-structured data be recovered with Time Travel?  
**Answer:**  
Yes. All table formats—including VARIANT—are protected.

### Q16: Why shouldn't Fail-safe be relied on for normal recovery?  
**Answer:**  
Fail-safe is slow, manual, and not guaranteed for quick recovery. It is meant strictly for last-resort disaster events.

---

# 4. Cross-Region & Cross-Cloud DR

### Q17: What is cross-region replication?  
**Answer:**  
Replication of databases, objects, and account metadata to a secondary region for DR or multi-region analytics.

### Q18: What can be replicated?  
**Answer:**  
Databases, shares, roles, users, warehouses (metadata), and account configuration.

### Q19: Does Snowflake replicate actual compute nodes?  
**Answer:**  
No. Only metadata and data are replicated; compute is provisioned on demand in each region.

### Q20: Can replication occur across cloud platforms?  
**Answer:**  
Yes. AWS ↔ Azure ↔ GCP cross-cloud replication is supported.

### Q21: How does replication impact latency?  
**Answer:**  
Replication is asynchronous. Large datasets may introduce replication lag depending on workload.

### Q22: What is RPO in Snowflake replication?  
**Answer:**  
The maximum acceptable data loss. Since replication is asynchronous, RPO depends on replication frequency.

### Q23: What is RTO in Snowflake failover?  
**Answer:**  
How quickly the failover can be executed. Typically low, because metadata is already replicated.

### Q24: Does cross-region replication support masking and RBAC?  
**Answer:**  
Yes. RBAC, security policies, and masking policies are replicated to keep governance consistent.

---

# 5. Failover & Failback

### Q25: What is account failover?  
**Answer:**  
Mechanism to redirect operations from a primary region to a secondary region after replication.

### Q26: What is a failover group?  
**Answer:**  
A logical grouping of objects replicated together to simplify regional failover.

### Q27: Is failback supported?  
**Answer:**  
Yes. After primary region is restored, failback syncs changes back.

### Q28: What happens to in-flight queries during failover?  
**Answer:**  
In-flight queries do not transfer; clients must reconnect to the failover endpoint.

### Q29: How long does failover take?  
**Answer:**  
Typically seconds to minutes, depending on metadata synchronization and DNS propagation.

### Q30: Does Snowflake offer automated failover?  
**Answer:**  
It supports programmatic or scheduled failover using automation tools, but does not auto-failover by itself to avoid false positives.

---

# 6. Testing & Automation

### Q31: Why is failover testing necessary?  
**Answer:**  
To validate access, pipelines, permissions, client connectivity, and DR readiness.

### Q32: What should DR testing include?  
**Answer:**  
- Metadata consistency validation  
- Data validation  
- Permission checks  
- Client reconnection testing  
- Warehouse startup verification  

### Q33: How can DR be automated?  
**Answer:**  
Using orchestration tools to automate replication checks, initiate failover, and validate endpoints.

### Q34: How often should DR tests be performed?  
**Answer:**  
Quarterly or aligned with compliance requirements (e.g., SOC 2).

### Q35: What kind of monitoring supports DR readiness?  
**Answer:**  
Replication lag monitoring, storage growth tracking, failover group validation, and multi-region health checks.

---

# 7. Scenarios

### Q36: Scenario — Region-wide outage. What happens?  
**Answer:**  
Primary region becomes inaccessible. If cross-region replication is configured, failover can be initiated to secondary region, restoring data and metadata rapidly.

### Q37: Scenario — Accidental table deletion.  
**Answer:**  
Use Time Travel to restore. If TT expired, Fail-safe may restore but with longer delays.

### Q38: Scenario — Corruption in micro-partitions.  
**Answer:**  
Snowflake automatically recovers using redundant copies in cloud storage.

### Q39: Scenario — Compliance requires multi-region availability.  
**Answer:**  
Use replication + failover groups. Run periodic DR validation to maintain certification readiness.

### Q40: Scenario — Workload isolation for DR.  
**Answer:**  
Critical workloads replicate to another region; non-critical ones remain single-region to reduce cost.

### Q41: Scenario — Failover of critical warehouses across regions.  
**Answer:**  
Warehouses themselves are not replicated; after failover, consumers launch warehouses in the secondary region. Metadata ensures all permissions and context remain intact.

---

# 8. Tradeoffs

### Q42: Tradeoff — Cross-region DR vs single-region HA  
Cross-region replication increases availability but introduces storage cost and replication lag.

### Q43: Tradeoff — Frequent vs infrequent replication  
Frequent replication reduces RPO but increases storage operations overhead.

### Q44: Tradeoff — Multi-cloud vs single-cloud DR  
Multi-cloud protects from cloud vendor outages but increases operational complexity and cost.

### Q45: Tradeoff — Time Travel vs Fail-safe for recovery  
Time Travel is fast, user-controlled, inexpensive.  
Fail-safe is slow, manual, and expensive—suitable only for catastrophic scenarios.

### Q46: Tradeoff — Full account replication vs selective replication  
Full replication simplifies DR but increases cost.  
Selective replication reduces cost but limits failover coverage.

---

# 9. Cost Considerations

### Q47: Cost of cross-region replication  
Storage cost increases because each region maintains a replicated copy of all selected objects.

### Q48: Compute cost during replication  
Replication itself uses Snowflake services; compute is not billed unless queries or tasks run in the secondary region.

### Q49: Cost of failover testing  
Warehouse compute is billed for any queries or pipelines executed during testing.

### Q50: Cost of Time Travel  
Increased retention increases storage consumption.

### Q51: Cost of Fail-safe  
Fail-safe retention is included in storage pricing but may increase overall storage footprint significantly for large tables.

---

# 10. Best Practices

### Q52: Separate DR strategy from HA strategy  
HA is automatic; DR requires planning and configuration.

### Q53: Use failover groups for structured replication  
Ensures related objects stay synchronized.

### Q54: Test regional failover regularly  
Validates pipelines, warehouses, external connections, and user permissions.

### Q55: Minimize Time Travel retention for non-critical tables  
Reduces storage footprint.

### Q56: Document regional endpoints for all applications  
Ensures clients can quickly reconnect after failover.

### Q57: Monitor replication lag  
Critical for low-RPO workloads.

### Q58: Implement read replicas for global analytics  
Improves performance for distributed teams while enhancing DR posture.

### Q59: Separate critical and non-critical workloads  
Reduce DR cost by replicating only essential datasets.

### Q60: Plan failback carefully  
Ensure no divergent data is overwritten during post-failover re-synchronization.

---

# End of Category 9  
Fully expanded, senior-level, scenario-rich, tradeoff-focused, GitHub-ready documentation for Disaster Recovery & High Availability.

