# Capability Map

## 1. Course Portfolio Management
- **Course Catalog Manage**
    - Course Define (title, description, type, vendor/internal, cost, language)
    - Course Classify (tags, categories, security level)
    - Course Version Control (draft, active, retired)
    - Course Prioritize (set and update priority levels)
- **Course Content Manage**
    - Workshop Recording Store (video/audio/files)
    - Course Material Attach (slides, documents, external links)
    - Course Content Update (edit metadata and materials)
- **Course Rule Configure**
    - Booking Rule Define (eligibility, prerequisites, capacity)
    - Validation Rule Define (priority-based acceptance rules)
    - Revalidation Rule Configure (scope and triggers for re-checks)

## 2. Booking & Enrollment Management
- **Booking Request Capture**
    - Course Booking Create (end-user self-service request)
    - Booking Status Track (Proposed, Accepted, Rejected, Approved, etc.)
- **Booking Validate & Decide**
    - Booking Auto-Validate (priority & rule checks)
    - Booking Auto-Accept / Auto-Reject (based on validation outcome)
    - Booking Manual Review (administrator review & override)
- **Enrollment Confirm**
    - Enrollment Confirm (change status to Approved / Enrolled)
    - Enrollment Cancel (user/admin initiated)

## 3. User Access & Security Management
- **User Authenticate**
    - SSO Authenticate (corporate identity provider)
    - Session Manage (timeout, re-login)
- **User Authorize**
    - Role & Permission Manage (learner, admin, vendor, auditor)
    - Access Control Enforce (per-course, per-module, per-report)
- **Security Control Enforce**
    - Sensitive Data Protect (PII, security-course data)
    - Audit Trail Capture (who changed what and when)

## 4. Notification & Communication Management
- **Notification Policy Manage**
    - Notification Rule Configure (which event triggers which channel)
    - Template Manage (email/SMS templates per event)
- **Event-Based Notify**
    - Admin Change Notify (course edit, vendor change, exceptions)
    - Learner Status Notify (booking proposed/approved/rejected)
    - Vendor Change Notify (opportunity added/updated/removed)
- **Channel Integrate**
    - Email Gateway Integrate
    - SMS Gateway Integrate

## 5. Third-Party Supplier Management
- **Vendor Onboard & Profile Manage**
    - Vendor Profile Maintain (contacts, branding, SLA)
    - Vendor Access Govern (roles, permissions, SSO/federation if any)
- **Vendor Opportunity Manage**
    - Vendor Course Opportunity Create
    - Vendor Course Opportunity Update
    - Vendor Course Opportunity View & Track
- **Vendor Collaboration Govern**
    - Vendor Notification Manage (changes, approvals)
    - Vendor Reporting Access (usage, attendance, revenue share)

## 6. Search & Discovery Management
- **Course Search Execute**
    - Search By Criteria (topic, vendor, level, location, date, priority)
    - Filter & Sort Results (date, popularity, relevance)
- **Course Discover Support**
    - Recommended Course Present (based on role, history – optional)
    - Saved Search / Favorite Manage (optional capability)

## 7. Reporting & Analytics Management
- **Operational Reporting Provide**
    - Booking Report Generate (per course, per user, per period)
    - Usage Report Generate (attendance, completion, cancellations)
    - Vendor Performance Report Generate (per vendor, per course)
- **Regulatory & Compliance Reporting Provide**
    - Security Training Compliance Report Generate
    - Audit Log Report Generate (changes, approvals, access)
- **Report Delivery Manage**
    - Report Export (CSV, XLSX, PDF)
    - Scheduled Report Deliver (email distribution – optional)

## 8. Platform & Operations Management
- **System Configuration Manage**
    - Master Data Manage (categories, locations, business units)
    - Localization & Time Zone Configure
- **Operations & Support**
    - Health Monitor & Alert (availability, errors)
    - Backup & Restore Execute
    - Environment Manage (test, pre-prod, prod)

---

# Utility Tree

> Legend per leaf node: **(Importance, Difficulty)** with values H/M/L.

## Utility (Overall System Quality)

| Category                     | Requirement                                                                                                                             | Importance | Difficulty |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|------------|------------|
| **Availability**             | System should be available for learners and admins during business hours with minimal downtime.                                         | H          | M          |
|              | Critical workflows (SSO login, course search, booking) must remain available even under partial component failure.                      | H          | H          |
| **Performance & Scalability** | Course search results should be returned within 2 seconds for 95% of queries under normal load.                                         | H          | M          |
|  | Booking creation (including validation and decision) should complete within 3 seconds for 95% of requests.                              | H          | M          |
|  | The system should scale to handle seasonal peaks without significant degradation.                                                       | H          | H          |
| **Security**                 | Users must authenticate via corporate SSO/IdP; no local passwords for employees.                                                        | H          | M          |
|                  | All sensitive data in transit must use TLS; admin and vendor operations protected with strong access control.                           | H          | L          |
|                  | Audit logs must capture all administrative/vendor changes to courses, rules, and bookings for at least N years.                         | H          | M          |
| **Usability**                | Admins should create/update a course via a guided web form without technical support.                                                   | H          | M          |
|                 | End-users should find/request a course within 3–4 clicks or <1 minute.                                                                   | H          | M          |
|                 | Vendor users should manage opportunities with minimal LMS-specific training.                                                             | M          | M          |
| **Modifiability & Extensibility** | Booking rules and validation logic should be configurable (not hard-coded).                                                            | H          | H          |
|  | Notification channels should be pluggable with minimal impact on core flows.                                                         | M          | M          |
|  | Adding a new vendor integration should not affect existing integrations.                                                             | M          | H          |
| **Interoperability**         | Platform integrates with corporate SSO and email/SMS gateways.                                                                         | H          | M          |
|          | External training vendors should consume/expose standard APIs for opportunity synchronization.                                           | M          | H          |
| **Reliability & Consistency** | Revalidation after course rule changes must complete without inconsistent states.                                                       | H          | H          |
|  | Notification sending must be resilient to transient failures (retry, DLQ).                                                              | M          | M          |
| **Auditability & Compliance** | All changes to course definitions, rules, booking decisions must be traceable to user, timestamp, and reason.                           | H          | M          |
|  | Compliance reports must be reproducible and trustworthy (no silent data loss).                                                          | H          | H          |


---

# Architecture Significant Requirements (ASR)

> These requirements have major impact on architecture, technology choices, and structure.

1. **SSO-Centric Authentication**
    - The system must authenticate all internal users (learners, admins) using corporate SSO (e.g., SAML/OIDC) and rely on central identity attributes (roles, departments, groups).
2. **Role-Based Access Control**
    - The system must support multiple personas (learner, admin, vendor, auditor) with clear separation of permissions and least-privilege access.
3. **Configurable Booking & Validation Engine**
    - Booking validation, prioritization, and auto-accept/reject logic must be externalized into configurable rules (e.g., rule engine, configuration store) rather than scattered in application code.
4. **Course Change Revalidation Mechanism**
    - Any change to course rules or priority must trigger revalidation of all affected non-paid bookings in a controlled, scalable way (e.g., background jobs, batch processing, idempotent updates).
5. **Event-Driven Notifications**
    - Key lifecycle events (course updated, booking proposed/accepted/rejected/approved, vendor opportunity updated) must publish events to a notification subsystem that dispatches email/SMS in a decoupled manner.
6. **Vendor Self-Service Portal**
    - Vendors must have a secure self-service interface (UI or API) to create, update, and manage their course opportunities, isolated from internal administration areas.
7. **Search & Filtering Across Course Catalog**
    - The system must provide flexible search with multi-criteria filtering across internal and vendor courses while remaining performant on large catalogs.
8. **Reporting & Data Warehouse Alignment**
    - The system must provide structured reporting (booking, usage, vendor performance, compliance) that can be exported or integrated into corporate reporting/BI platforms.
9. **Audit Trail for Business-Critical Operations**
    - All create/update/delete operations on courses, rules, approvals, and vendor opportunities must be fully auditable (who, when, before/after state).
10. **Scalable Architecture for Campaign Peaks**
    - The architecture must handle load peaks (e.g., mandatory security trainings) without significant rework—supporting horizontal scaling or efficient queueing where applicable.
11. **Separation of Concerns per Domain**
    - Core domains (Course Management, Booking, Vendor Management, Notification, Reporting) should be modularized (e.g., services/modules) to allow independent evolution and deployment.
12. **Resilient Integration with External Gateways**
    - Email/SMS and vendor integrations must tolerate temporary outages (retry, backoff, dead-letter handling) without data loss or inconsistent business states.

---

# Quality Attribute Requirements (Selected)

> Structured as [ID] – *Attribute* – Description (Importance, Difficulty).

| ID             | Category        | Requirement Description                                                                                                                                           | Importance | Difficulty |
|----------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|------------|
| QA-SEC-01      | Security         | Internal users must authenticate via corporate SSO; LMS must not store employee passwords locally.                                                               | H          | M          |
| QA-SEC-02      | Security         | Access to admin, vendor, and audit functions must be restricted to specific roles and permissions, configurable per organization policy.                        | H          | M          |
| QA-SEC-03      | Security         | All external communication must use TLS with modern protocols and ciphers.                                                                                       | H          | L          |
| QA-PERF-01     | Performance      | 95% of course search requests must complete within 2 seconds under normal load.                                                                                  | H          | M          |
| QA-PERF-02     | Performance      | Booking creation (incl. validation & decision) must complete within 3 seconds for 95% of requests.                                                               | H          | M          |
| QA-AVAIL-01    | Availability     | SSO login, search, and booking creation must achieve 99.x% availability (SLA TBD).                                                                               | H          | M          |
| QA-MOD-01      | Modifiability    | Changing booking/priority rules should not affect more than one module and should not require DB schema migration in typical cases.                               | H          | H          |
| QA-MOD-02      | Modifiability    | Onboarding a new vendor using an existing pattern should not require downtime and must not affect other vendor integrations.                                      | M          | H          |
| QA-REL-01      | Reliability      | Revalidation jobs must be restartable and idempotent to avoid inconsistent booking states or duplicate notifications.                                             | H          | H          |
| QA-AUD-01      | Auditability     | All changes to courses, rules, approvals, and vendor opportunities must be logged with user ID, timestamp, and old/new values.                                   | H          | M          |
| QA-REP-01      | Reporting        | Compliance reports must rely on immutable completion records and be reproducible for any date range.                                                             | H          | H          |
| QA-USAB-01     | Usability        | Admins should be able to create/update a course in a single workflow with limited steps (to be refined during UX design).                                        | H          | M          |

---

# Constraints

> Constraints are external limitations or design boundaries that the solution must respect; they are not simple copies of functional requirements.

| ID      | Constraint Title                      | Description                                                                                                                                                 |
|---------|----------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| C-001   | Corporate Identity Provider            | The LMS must integrate with the organization’s existing identity provider for SSO (e.g., SAML/OIDC) and cannot introduce a parallel identity store.        |
| C-002   | Standard Communication Channels        | The LMS must use existing corporate email and SMS gateways; introducing new messaging providers requires security and procurement approval.                |
| C-003   | Technology & Hosting Standards         | The solution must comply with the organization’s hosting model (cloud/on-prem/hybrid) and follow standard tech stacks and IT operational practices.        |
| C-004   | Security & Compliance Policies         | The LMS must comply with corporate security policies and applicable regulations, including retention and access control rules for training/user data.       |
| C-005   | Data Residency & Retention             | Training, booking, and audit data must be stored in approved regions and retained for a compliance-defined minimum period (e.g., N years).                 |
| C-006   | Integration Protocols                  | Integrations with external vendors/gateways must use approved protocols (HTTPS REST, secure file transfer). Direct DB-level integration is prohibited.    |
| C-007   | Change Management & Release Process    | Deployment of LMS changes (including configuration affecting rules and validation) must comply with the organization’s change management process (release windows, approvals, rollback strategy)|
| C-008   | Licensing & Cost Constraints           | Selected platform components (DB, messaging, rule engine, reporting tools) must fit within existing license agreements or approved budget; non-approved commercial components are not allowed.|

