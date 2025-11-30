# Capability Map – Learning Management System (LMS)

| Capability Level                         | Capability (Noun + Verb)           | Description                                                                                                      | Primary Stakeholders           |
|------------------------------------------|------------------------------------|------------------------------------------------------------------------------------------------------------------|--------------------------------|
| Manage Courses                           | Capture Course Information         | Capture core course data (title, description, type, security flag, etc.) for internal and vendor courses.       | Admin, Suppliers               |
| Manage Courses                           | Maintain Course Catalogue          | Create, update, and retire course records via a web form interface.                                             | Admin                          |
| Manage Courses                           | Prioritize Courses                 | Assign and maintain course priority levels used in validation and booking rules.                                | Admin, Business Owners         |
| Manage Courses                           | Revalidate Courses                 | Automatically revalidate non-paid course bookings against updated course rules and priorities.                  | System, Admin                  |
| Manage Courses                           | Search Courses                     | Search and filter courses based on user-defined criteria (topic, supplier, priority, etc.).                     | Learners, Admin, Suppliers     |
| Authenticate Users                       | Enforce SSO Authentication         | Enforce Single Sign-On for learners, administrators, and suppliers to access the system.                        | IT Security, All Users         |
| Authenticate Users                       | Segment User Roles                 | Distinguish access and capabilities for learners, admins, and third-party suppliers.                            | Admin, IT Security             |
| Manage Bookings                          | Capture Booking Requests           | Allow users to create course booking requests and assign them an initial “Proposed” status.                     | Learners                       |
| Manage Bookings                          | Validate Booking Requests          | Validate booking requests against priority and booking rules; auto-accept or reject when possible.              | System, Admin                  |
| Manage Bookings                          | Approve Booking Requests           | Enable administrators to review booking details and set final decision (Approved/Rejected).                     | Admin                          |
| Manage Bookings                          | Track Booking Status               | Maintain and expose status history for each booking (Proposed, Accepted, Approved, Rejected, Paid, etc.).       | Learners, Admin                |
| Manage Notifications                     | Notify Course Changes              | Send email/SMS notifications to administrators when a course is edited or rules change.                         | Admin                          |
| Manage Notifications                     | Notify Booking Events              | Send email/SMS notifications to users on booking creation, validation, and status changes.                      | Learners, Admin                |
| Manage Notifications                     | Notify Supplier Updates            | Notify suppliers and administrators when supplier opportunities are created or updated.                          | Suppliers, Admin               |
| Manage Supplier Opportunities            | Maintain Supplier Catalogue        | Allow suppliers to view, add, and edit their course opportunities.                                              | Suppliers                      |
| Manage Supplier Opportunities            | Govern Supplier Changes            | Apply business and approval rules on supplier-provided course changes where required.                           | Admin, Procurement             |
| Report and Analyze Usage                 | Generate Operational Reports       | Generate detailed reports (bookings, vendors, usage) based on defined search criteria.                          | Admin, HR, Management          |
| Report and Analyze Usage                 | Export Report Data                 | Allow stakeholders to view and download reports for further analysis and archiving.                             | Admin, HR, Management          |

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

