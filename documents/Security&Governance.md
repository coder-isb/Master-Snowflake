# Category 7: Security & Governance  
Deep-Dive, Senior Engineering & Architecture Notes

This chapter covers **RBAC, access control, masking, network security, encryption, secure data sharing, governance, auditing, compliance**, and advanced scenario-driven design considerations.  
Answers are detailed, real-world, and interview-focused.

---

## Table of Contents  

1. [Fundamentals](#1-fundamentals)  
2. [RBAC & Access Control](#2-rbac--access-control)  
3. [Object, Row, and Column-Level Security](#3-object-row-and-column-level-security)  
4. [Data Masking & Dynamic Policies](#4-data-masking--dynamic-policies)  
5. [Network Security & Encryption](#5-network-security--encryption)  
6. [Secure Data Sharing](#6-secure-data-sharing)  
7. [Auditing, Monitoring & Compliance](#7-auditing-monitoring--compliance)  
8. [Scenario-Based Questions](#8-scenario-based-questions)  
9. [Tradeoffs & Cost Considerations](#9-tradeoffs--cost-considerations)  

---

## 1. Fundamentals

### Q1: What is Snowflake security and governance?
**Answer:**  
It is the combination of Snowflake’s built-in features—RBAC, encryption, masking, role hierarchy, access policies, auditing, secure sharing, and network protection—that ensure data protection, least-privilege access, and regulatory compliance.

### Q2: Is security default-on in Snowflake?
**Answer:**  
Yes. All data is encrypted at rest and in transit. RBAC-based access controls are mandatory, and governance features are enabled through managed cloud services.

### Q3: What are the pillars of Snowflake security?
**Answer:**  
- Identity & Access Management  
- Data Protection  
- Network Security  
- Monitoring & Auditing  
- Governance & Compliance  

---

## 2. RBAC & Access Control

### Q4: What is RBAC in Snowflake?
**Answer:**  
RBAC controls access using **roles**, **privileges**, and **ownership hierarchy**. Users receive access by being assigned roles, not directly through grants.

### Q5: Why is RBAC hierarchical?
**Answer:**  
It enables clear inheritance of privileges, separation of duties, and simplified permission maintenance.

### Q6: What is the principle of least privilege?
**Answer:**  
Users are granted **only the access needed** to perform their duties and no more.

### Q7: What is role chaining?
**Answer:**  
Switching between roles to perform different tasks. Only one role is active at a time.

### Q8: What are common Snowflake role patterns?
**Answer:**  
- **System roles**: ACCOUNTADMIN, SECURITYADMIN, SYSADMIN, PUBLIC  
- **Functional roles**: DATA_ENGINEER, ANALYST, APP_USER  
- **Database roles**: DB_READ, DB_WRITE, DB_ADMIN  

### Q9: Why separate system roles from business roles?
**Answer:**  
Avoids overprivileged users, makes audits cleaner, and reduces lateral movement risks.

---

## 3. Object, Row, and Column-Level Security

### Q10: What is object-level security?
**Answer:**  
Controls access to tables, schemas, warehouses, stages, and all Snowflake objects. Access is explicitly granted to roles.

### Q11: What is row-level security?
**Answer:**  
Policies that filter rows based on user role or attributes. Useful for multi-tenant applications, region restrictions, and sensitive operational datasets.

### Q12: What is column-level security?
**Answer:**  
Restricts or masks specific columns like SSNs, credit cards, or PII.

### Q13: When do you use row-level vs column-level security?
**Answer:**  
- Row-level: limit **which records** a user sees  
- Column-level: limit **which fields** a user sees  
Often used together in regulated industries.

---

## 4. Data Masking & Dynamic Policies

### Q14: What is Snowflake dynamic data masking?
**Answer:**  
A policy that transforms sensitive values at query time based on the requesting user’s role. It enforces real-time, conditional visibility.

### Q15: Why is dynamic masking preferable to static masking?
**Answer:**  
- No data duplication  
- No transformation overhead  
- Context-aware (time, role, device attributes)  
- Zero maintenance of masked copies

### Q16: What types of masking exist?
**Answer:**  
- Full masking  
- Partial masking  
- Tokenization-like patterns  
- Role-based visibility  
- Environment-based masking (dev/test vs prod)  

### Q17: Common pitfalls of masking?
**Answer:**  
- Incorrectly scoped masking policies  
- Forgetting to apply masking on new columns  
- Over-masking degrading analytical usability  

---

## 5. Network Security & Encryption

### Q18: How does Snowflake protect data in transit?
**Answer:**  
TLS-based encryption, strict certificate validation, secure endpoints, and managed cloud-service isolation.

### Q19: What is IP whitelisting?
**Answer:**  
Restricting Snowflake access to approved IP ranges—important for corporate networks, partner integration, and regulated workloads.

### Q20: What is network policy?
**Answer:**  
A Snowflake object controlling allowed IPs, blocked IPs, and user-specific access restrictions.

### Q21: How does Snowflake ensure data encryption at rest?
**Answer:**  
Uses strong AES encryption with hierarchical key management. Keys rotate automatically and are different for each micro-partition file.

### Q22: What additional network isolation options exist?
**Answer:**  
- PrivateLink (AWS)  
- Private Service Connect (GCP)  
- VNet Integration (Azure)  
These provide private, non-public routing.

---

## 6. Secure Data Sharing

### Q23: What is secure data sharing?
**Answer:**  
A mechanism to share live data with external consumers **without copying**, **moving**, or **exporting** data.

### Q24: How is it different from file sharing?
**Answer:**  
Secure sharing exposes live tables with metadata control; file sharing requires duplicating data outside Snowflake.

### Q25: Who pays for compute?
**Answer:**  
- Provider pays for storage  
- Consumer pays for compute when querying shared data  
This reduces cost for the data provider.

### Q26: Why is secure sharing governance-friendly?
**Answer:**  
- Access is traceable  
- No data egress  
- No file transfers  
- Revocation is instant  

---

## 7. Auditing, Monitoring & Compliance

### Q27: What is Snowflake's auditing framework?
**Answer:**  
Account usage tables, login history, query history, access history, and system events provide complete end-to-end audit trails.

### Q28: What compliance standards does Snowflake support?
**Answer:**  
SOC 2, PCI DSS, HIPAA, FedRAMP, ISO, GDPR-aligned controls, data residency features, and encryption guarantees.

### Q29: How does Snowflake help ensure compliance?
**Answer:**  
- Full audit logs  
- Role-based access  
- Dynamic masking  
- Row/column security  
- Locality controls  
- Secure data sharing  
- Encryption everywhere  

### Q30: Why is access history important?
**Answer:**  
Tracks who accessed what, when, and through which role—critical for forensic analysis and compliance attestation.

---

## 8. Scenario-Based Questions

### Q31: A team wants sensitive PII visible only to HR but partially masked for finance. How do you design it?
**Answer:**  
Use column masking policies with conditional logic based on user roles.  
HR sees full values; finance sees partial masked values; others see fully masked fields.

### Q32: A downstream analytics team needs aggregated data but not raw sensitive data. What is the best solution?
**Answer:**  
Use secure views or derived tables with masking + role restrictions.  
Avoid giving access to raw tables to prevent accidental exposure.

### Q33: A partner company needs real-time read-only access to specific datasets. How do you provide this?
**Answer:**  
Use secure data sharing.  
Partner pays for compute; provider retains control; no copying or exports.

### Q34: A company wants to restrict access to Snowflake only from corporate VPN. What do you configure?
**Answer:**  
Use network policies with IP whitelisting and optionally private connectivity (PrivateLink).

### Q35: Regulatory auditors require proof that no unauthorized user accessed customer PII. How is this demonstrated?
**Answer:**  
Use Access History and Query History records.  
Cross-reference roles and privileges with masking policies to validate visibility.

### Q36: A multi-tenant application needs tenant-based row filtering. Which feature do you use?
**Answer:**  
Row-level security policy filtering based on session attributes or user roles.

### Q37: Sensitive tables are frequently updated, and masking performance is a concern. How do you handle this?
**Answer:**  
Ensure masking rules are efficient, avoid overly complex policies, and consider flattening and normalizing sensitive fields to reduce policy overhead.

### Q38: Analysts complain they cannot see required fields due to masking. What is the fix?
**Answer:**  
Review role assignments and policy logic to ensure correct role mappings and intended visibility.

### Q39: A high-security environment requires separation between admin and operational roles. What architecture do you apply?
**Answer:**  
Enforce strict RBAC separation:  
- ACCOUNTADMIN rarely used  
- SECURITYADMIN for privileges  
- SYSADMIN for objects  
- Operational roles for daily work  

### Q40: How do you detect abnormal login attempts or suspicious user activity?
**Answer:**  
Monitor LOGIN_HISTORY, ACCESS_HISTORY, and query patterns using anomaly detection dashboards or SIEM integration.

---

## 9. Tradeoffs & Cost Considerations

### Q41: Tradeoff: Dynamic Masking vs Pre-masked tables  
**Dynamic Masking Pros:**  
- Context-aware  
- No data duplication  
- Real-time enforcement  

**Cons:**  
- Slight runtime overhead if poorly designed  
- Complexity in large masking rules  

Pre-masked tables increase storage cost and introduce maintenance overhead.

### Q42: Tradeoff: PrivateLink vs Public Internet with IP whitelisting  
PrivateLink offers private connectivity with highest security but costs more and requires cloud-networking setup.  
Public with IP whitelisting is simpler but less secure.

### Q43: Tradeoff: Separate warehouses for governance vs shared warehouses  
Separate warehouses improve security isolation and logging clarity but increase cost.  
Shared warehouses reduce cost but risk cross-team workload impact.

### Q44: Cost implications of excessive role sprawl  
Too many overlapping roles complicate governance, increase audit complexity, and raise compliance risk.  
Proper role design reduces operational burden.

### Q45: Cost impact of secure data sharing  
Provider saves cost because consumer pays for compute.  
Provider only pays for storage, which is often minimal compared to replicated data architecture.

### Q46: Tradeoff: Row-level security vs maintaining separate tables per tenant  
Row-level security reduces storage cost and simplifies pipelines but may add runtime overhead.  
Separate tables offer isolation but increase storage and maintenance cost.

### Q47: Tradeoff: Masking vs Tokenization  
Masking is fast and reversible conditionally.  
Tokenization is irreversible and more secure but harder to use for analytical needs.

### Q48: Compliance tradeoffs between minimal and strict governance  
Strict governance improves auditability but increases operational friction for developers.

### Q49: Should all data be encrypted client-side before loading?  
Provides extra security but increases complexity, prevents warehouse-level optimizations, and may degrade performance.

### Q50: When is it worth investing in full private connectivity?  
When handling highly regulated data (banking, healthcare, government) or when preventing public exposure is mandatory.

---

# End of Category 7  
GitHub-ready, scenario-driven, architecture-focused reference.

