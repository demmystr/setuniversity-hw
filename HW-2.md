# Homework 2 — Capability Map, Utility Tree, ASR, Constraints  
Learning Management System (LMS)

This document defines the capability map, utility tree, architecturally significant requirements (ASR), quality attribute requirements, and constraints for the Learning Management System.

---

# 1. Capability Map

Capabilities are expressed from a **product perspective**, not as implementation features.  
L1 — top-level business capabilities; L2 — supporting sub-capabilities.

### 1.1 Capability Map Table

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Manage Courses** | End-to-end lifecycle management of internal, vendor and security courses. |
| L2 | Record Internal Courses | Capture, store and access recordings of internal workshops and courses. |
| L2 | Maintain Course Catalog | Create, update, deactivate and version course definitions and metadata. |
| L2 | Assign Course Priority | Set and modify priority levels to influence validation and booking logic. |
| L2 | Revalidate Courses | Re-run validation rules on non-paid courses after catalog changes. |
| L2 | Notify Course Stakeholders | Send notifications (email/SMS) to learners and administrators. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Manage Course Bookings** | Manage booking requests, automated decisions, and approval workflows. |
| L2 | Capture Booking Requests | Users submit booking requests; system assigns "Proposed" status. |
| L2 | Validate Booking Requests | Automatic validation based on rules, course priority, and constraints. |
| L2 | Decide Booking Outcomes | Automated acceptance or rejection of booking requests. |
| L2 | Approve Bookings Manually | Administrators review details and approve valid bookings. |
| L2 | Notify Booking Stakeholders | Notify learners and admins about booking status changes. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Authenticate and Authorize Users** | Provide secure access and enforce permission controls. |
| L2 | Authenticate Users via SSO | Authenticate via corporate identity provider using standardized protocols. |
| L2 | Authorize Roles and Permissions | Enforce role-based permissions for learner, admin, vendor, and auditor roles. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Integrate Third-Party Suppliers** | Enable vendors to manage and offer their course opportunities. |
| L2 | Manage Supplier Opportunities | Vendors create, edit, update and retire course opportunities. |
| L2 | Notify Supplier Changes | Notify suppliers and admins of opportunity updates. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Search Courses** | Provide search and filtering across courses and bookings. |
| L2 | Search Course Catalog | Search internal and vendor course catalog using flexible filters. |
| L2 | Search Bookings | Search and filter bookings for users and administrators. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Reporting and Analytics** | Provide operational, vendor, usage, and booking reports. |
| L2 | Report on Bookings | Export and view booking performance and statuses. |
| L2 | Report on Vendors | Export and view vendor usage and course performance. |
| L2 | Report on System Usage | Provide metrics on system usage by role, user and time period. |

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **L1** | **Operate and Monitor LMS Platform** | Ensure reliable and observable LMS operations. |
| L2 | Monitor System Health | Track availability, performance and failures. |
| L2 | Manage Configuration and Rules | Maintain rules for validation, notifications, priorities and integrations. |

---

# 2. Utility Tree

Legend: (Importance, Difficulty/Risk) — H/M/L  
Each scenario represents a **quality attribute** that is relevant to architectural decisions.

## 2.1 Utility Tree

### **Security**
- LMS must rely on corporate IdP using SAML/OIDC for authentication; no local password storage. **(H, M)**
- All external communication must use TLS with strong and updated cipher suites. **(H, L)**
- Role-based access control must protect admin, vendor and audit functions. **(H, M)**
- Security-relevant events (auth failures, permission denials) must be logged centrally. **(H, M)**

### **Performance**
- Course search must return results within ≤2 seconds for 95% of requests under normal load. **(H, M)**
- Booking validation and decision must complete within ≤3 seconds for 95% of requests. **(H, M)**
- Reports must generate and download within ≤10 seconds for 90% of typical use cases. **(M, M)**

### **Availability & Reliability**
- Core LMS functions must maintain ≥99.5% uptime during business hours. **(H, H)**
- No booking or course changes may be lost even when external services are temporarily unavailable. **(H, M)**
- LMS must recover from node/service failure without data loss; manual failover ≤30 minutes. **(M, M)**

### **Usability**
- Admin must be able to create or update a course in ≤5 minutes after basic training. **(H, L)**
- Booking submission must require ≤3 steps from search to confirmation. **(M, M)**
- System must provide clear, localized error, validation, and status feedback. **(M, L)**

### **Modifiability**
- Validation and booking rules must be editable without code changes (config or rule engine). **(H, H)**
- New vendor integration based on standard interface must require ≤5 person-days. **(M, H)**
- New notification channels must be pluggable with minimal impact on existing code. **(M, M)**

### **Interoperability & Integration**
- LMS must integrate with corporate SSO following standard protocols (SAML/OIDC). **(H, M)**
- LMS must support at least one email/SMS gateway for notifications. **(H, M)**
- LMS must integrate with at least one external vendor API using stable interface. **(M, H)**

### **Auditability**
- Audit logs must be exportable in a standardized, machine-readable format. **(H, M)**
- Parameterized operational reports must be available for authorized users. **(M, M)**

---

# 3. Architecturally Significant Requirements (ASRs), QA Requirements, and Constraints

ASRs are the subset of requirements that **directly influence architecture**.  
They are distinct from technical requirements (behavioral) and constraints (limitations).

---

# 3.1 Architecturally Significant Requirements (ASR)

| ID | Category | ASR | Rationale |
|----|----------|-----|-----------|
| **ASR-SEC-01** | Security | The LMS must delegate all authentication to the corporate identity provider using SAML/OIDC; no local credential storage allowed. | Drives choice of authentication mechanism, security architecture, libraries, and integration model. |
| **ASR-SEC-02** | Security | All external system communication must use TLS with strong cipher suites. | Determines API gateway configuration, certificate management, service endpoints. |
| **ASR-SEC-03** | Security | LMS must implement centralized, immutable audit logging for admin, vendor and booking operations. | Requires logging architecture, audit storage design, and compliance alignment. |
| **ASR-PERF-01** | Performance | Course search must support ≤2s response time for 95% of requests, requiring indexing and/or caching mechanisms. | Influences DB choice, indexing strategy, caching tier, or search engine use. |
| **ASR-PERF-02** | Performance | Booking validation must complete within ≤3s under normal load using efficient rule evaluation. | Drives rule engine architecture, synchronous/async design. |
| **ASR-AVAIL-01** | Availability | LMS must maintain ≥99.5% availability, requiring redundancy, failover, and monitoring strategies. | Influences deployment topology, clustering, health checks. |
| **ASR-MOD-01** | Modifiability | Business validation rules must be externally configurable without code changes (rule engine or config-driven approach). | Determines modularity, configuration architecture, extensibility. |
| **ASR-INTEG-01** | Integration | LMS must integrate with at least one external vendor API using standardized integration patterns. | Influences integration layer design, error handling, API gateway. |

---

# 3.2 Quality Attribute Requirements (Non-ASR Technical Requirements)

| ID | Quality Attribute | Requirement |
|----|-------------------|-------------|
| **QA-SEC-01** | Security | Only authenticated users may access LMS functionality. |
| **QA-SEC-02** | Security | Security events must be timestamped and logged. |
| **QA-PERF-01** | Performance | Report generation must complete within ≤10 seconds for standard periods. |
| **QA-USAB-01** | Usability | Admin must configure or update a course in ≤5 minutes. |
| **QA-USAB-02** | Usability | Booking process must consist of ≤3 steps. |
| **QA-INTEG-01** | Interoperability | LMS must support multiple notification channels (email, SMS). |
| **QA-REP-01** | Auditability | Audit logs must be exportable in CSV or similar format. |

---

# 3.3 Constraints (True Architectural Limitations)

Constraints define what **cannot** be changed and limit architectural freedom.

| ID | Type | Constraint | Source |
|----|------|------------|--------|
| **CON-ORG-01** | Organizational | LMS must use the existing corporate SSO provider; replacing or modifying IdP is not allowed. | IT/Security |
| **CON-ORG-02** | Organizational | LMS must comply with corporate audit and logging retention policies. | Compliance |
| **CON-TECH-01** | Technical | LMS must be deployed within corporate infrastructure (on-prem/cloud as defined). | Enterprise Architecture |
| **CON-TECH-02** | Technical | Only standard SSO and security protocols (SAML/OIDC, TLS) may be used. | Security Architecture |
| **CON-TECH-03** | Technical | Vendor-side APIs cannot be modified; LMS must adapt to existing vendor interfaces. | Vendor |
| **CON-OPS-01** | Operational | Monitoring and logging must integrate with existing ops tools and processes. | DevOps |

---
