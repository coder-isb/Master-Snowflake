# Category 8: Data Sharing & Collaboration  
Deep Dive for Architects, Data Engineers & Interview Preparation

Snowflake’s data-sharing capabilities allow organizations to exchange **live, governed, zero-copy data** across accounts, regions, and even clouds with strong governance and minimal operational effort.

This document explains the architecture, concepts, governance, scenarios, tradeoffs, cost implications, and best practices.

---

## Table of Contents

1. [Fundamentals](#1-fundamentals)  
2. [Secure Data Sharing Concepts](#2-secure-data-sharing-concepts)  
3. [Reader Accounts](#3-reader-accounts)  
4. [Zero-Copy Sharing](#4-zero-copy-sharing)  
5. [Cross-Region & Cross-Cloud Sharing](#5-cross-region--cross-cloud-sharing)  
6. [Governance & Access Controls](#6-governance--access-controls)  
7. [Advanced Scenarios](#7-advanced-scenarios)  
8. [Architecture & Tradeoffs](#8-architecture--tradeoffs)  
9. [Cost Considerations](#9-cost-considerations)  
10. [Best Practices](#10-best-practices)  

---

## 1. Fundamentals

### Q1: What is Snowflake data sharing?  
**Answer:**  
A mechanism to give other Snowflake accounts access to **live data** without copying or moving it. Data is shared logically via metadata pointers, while storage remains with the provider.

### Q2: Why is sharing “live” data significant?  
**Answer:**  
Consumers see updates in real time. There are no pipelines, file transfers, replication jobs, or data duplication overhead.

### Q3: What is the core benefit of Snowflake sharing?  
**Answer:**  
The producer retains control and reduces operational complexity, while consumers access fresh, governed data with minimal setup.

### Q4: Who pays for compute during sharing?  
**Answer:**  
The consumer pays for compute used when querying shared data.  
The provider only pays for storage.

### Q5: What objects can be shared?  
**Answer:**  
Databases, schemas, tables, views, and secure UDFs, as long as they are allowed by governance policies and marked as secure.

---

## 2. Secure Data Sharing Concepts

### Q6: What is secure data sharing?  
**Answer:**  
A Snowflake mechanism where data is shared in a **read-only** manner through secure metadata links, with strong access controls and no data movement.

### Q7: How is access to shared data granted?  
**Answer:**  
Producers create shares and assign objects to them; consumers import shares into their own accounts.

### Q8: Is shared data physically moved?  
**Answer:**  
No. Snowflake relies on metadata pointers; physical storage remains with the producer.

### Q9: Can shared data be modified by consumers?  
**Answer:**  
No, shared data is read-only. Consumers can create their own derived tables if needed.

### Q10: What happens if a provider revokes sharing?  
**Answer:**  
Access is immediately removed, and consumers can no longer query the shared objects.

### Q11: How is security enforced during data sharing?  
**Answer:**  
Through RBAC, secure objects, metadata isolation, and auditing of all access events.

---

## 3. Reader Accounts

### Q12: What is a reader account?  
**Answer:**  
A Snowflake-managed account created by a provider for customers or partners who don’t have their own Snowflake subscription.

### Q13: Why use reader accounts?  
**Answer:**  
To provide partners access to data without requiring them to manage their own Snowflake account.

### Q14: Who pays for compute in a reader account?  
**Answer:**  
The data provider pays for compute and storage for reader accounts.

### Q15: When should you **not** use reader accounts?  
**Answer:**  
When consumers run heavy analytical workloads—because provider will pay compute costs.

---

## 4. Zero-Copy Sharing

### Q16: What is zero-copy sharing?  
**Answer:**  
Sharing data without duplicating physical storage. Multiple consumers reference the same micro-partition data.

### Q17: Why is zero-copy efficient?  
**Answer:**  
- No ETL or replication  
- No additional storage cost  
- Real-time availability  
- Simplifies governance and lineage  

### Q18: Is zero-copy sharing the same as cloning?  
**Answer:**  
No. Cloning creates a metadata copy for internal use; sharing exposes data to external accounts.

---

## 5. Cross-Region & Cross-Cloud Sharing

### Q19: Can Snowflake share data across regions?  
**Answer:**  
Yes, using cross-region replication and sharing capabilities.

### Q20: Can Snowflake share data across different cloud platforms?  
**Answer:**  
Yes. Snowflake supports cross-cloud sharing between AWS, Azure, and GCP.

### Q21: Why share across clouds?  
**Answer:**  
- Multi-cloud analytics strategies  
- Partner ecosystems in different clouds  
- Regulatory requirements  
- Disaster recovery architectures  

### Q22: What is cross-region replication?  
**Answer:**  
Copying data asynchronously to a different region to enable cross-region sharing and resilience.

### Q23: How does latency affect cross-region sharing?  
**Answer:**  
Consumers might experience slightly higher latency depending on region distance, though compute executes where the consumer account resides.

---

## 6. Governance & Access Controls

### Q24: How does RBAC apply to shared data?  
**Answer:**  
Only roles with sufficient privileges can include objects in a share. Consumer roles control which users access imported data.

### Q25: What objects require “secure” designation?  
**Answer:**  
Views and UDFs must be secure to be shared, ensuring definition protection and controlled visibility.

### Q26: Are audit logs available for shared data access?  
**Answer:**  
Yes, both provider and consumer can audit access through Snowflake usage views.

### Q27: How do you prevent unauthorized propagation of shared data?  
**Answer:**  
Consumer accounts cannot reshare data unless the provider explicitly enables this.

---

## 7. Advanced Scenarios

### Q28: Scenario — A retail company wants to share weekly sales data with suppliers. How should this be designed?  
**Answer:**  
Use secure zero-copy sharing.  
Suppliers either use their existing Snowflake accounts (ideal) or receive reader accounts.

### Q29: Scenario — A partner with no Snowflake account must see the data. What is the approach?  
**Answer:**  
Provide a reader account.  
Provider pays compute; partner pays nothing.

### Q30: Scenario — Multiple analytics teams need derived datasets but must not see raw data.  
**Answer:**  
Share only curated views or aggregated tables.  
Apply masking policies and share only secure views.

### Q31: Scenario — A company wants to migrate from region A to region B but needs continuity.  
**Answer:**  
Cross-region replication and sharing allow seamless transition, with consumers switched gradually.

### Q32: Scenario — A SaaS provider needs strict tenant isolation while sharing global metadata.  
**Answer:**  
Partition sharing using secure views + row-level security ensures isolated visibility per tenant.

### Q33: Scenario — A partner needs “write-back” capabilities.  
**Answer:**  
Secure sharing only allows read access.  
Partner must ingest data back into Snowflake or use APIs for two-way integration.

### Q34: Scenario — Consumer wants to join shared data with their internal data efficiently.  
**Answer:**  
Consumers can create local derived tables, materializations, and clustering for performance improvements.

---

## 8. Architecture & Tradeoffs

### Q35: Tradeoff — Secure Sharing vs ETL-based file sharing  
**Secure Sharing Pros:**  
- No storage duplication  
- Real-time updates  
- Governance enforcement  
- No pipelines to maintain  

**Cons:**  
- Read-only for consumers  
- Consumers need Snowflake accounts  

File-based sharing may be necessary for non-Snowflake ecosystems.

### Q36: Tradeoff — Reader Accounts vs Standard Accounts  
Reader accounts are great for low-volume partners but costly for compute-heavy consumers.  
Standard accounts shift compute cost to consumers.

### Q37: Tradeoff — Cross-region sharing vs Single-region deployment  
Cross-region sharing improves availability but introduces replication storage cost.

### Q38: Tradeoff — Full dataset sharing vs sharing secure views  
Full data sharing is simpler but may expose sensitive data.  
Secure views add governance but require careful policy management.

### Q39: Tradeoff — Multi-cloud sharing vs single cloud  
Multi-cloud increases reach but adds operational and cost considerations regarding replication and governance.

---

## 9. Cost Considerations

### Q40: Who pays for what in sharing?  
- Provider: Storage (always)  
- Consumer: Compute (when querying shared data)  
- Provider for reader accounts: Compute + storage  

### Q41: Cost of cross-region or cross-cloud replication  
Replication increases storage cost and minimal metadata overhead.  
Compute cost occurs only for failover tests or consumer workloads.

### Q42: Cost impact of reader accounts  
If consumer workloads are heavy, provider’s compute cost may spike.

### Q43: Avoiding unnecessary replication  
Not all datasets need cross-region copies; share only business-critical data.

### Q44: Cost benefits of zero-copy architecture  
No duplicated storage, no data pipelines, and no ETL processing reduce operational cost significantly.

---

## 10. Best Practices

### Q45: Use secure views for sensitive datasets  
Limits exposed fields and enforces consistent business logic.

### Q46: Separate shares for different consumer groups  
Allows granular governance and reduces accidental access exposure.

### Q47: Use cross-region replication for DR only when needed  
Avoids unnecessary storage cost.

### Q48: Keep shared schemas minimal and purpose-built  
Avoid sharing entire databases unless required.

### Q49: Monitor usage to detect abnormal consumer behavior  
Identifies inefficient queries or potential misuse.

### Q50: Document your sharing contracts  
Define retention, SLA, governance expectations, and revocation processes.

---

# End of Category 8  
Fully expanded, GitHub-ready, with interview scenarios, tradeoffs, cost implications, and best practices.

