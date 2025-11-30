# 1. Capability Map

### 1.1 Capability Map Table

| Capability Level | Capability                           | Description                                                                                               |
|------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------|
| L1               | Manage Courses                       | End-to-end management of internal, vendor and security courses throughout their lifecycle.                |
| L2               | Record Internal Courses              | Capture and store recordings of internal workshops and courses.                                           |
| L2               | Maintain Course Catalog              | Create, update, deactivate and version course definitions and metadata.                                   |
| L2               | Assign Course Priority               | Set and modify course priority levels to influence validation and booking rules.                          |
| L2               | Revalidate Courses                   | Re-run validation rules on all non-paid courses after catalog changes.                                   |
| L2               | Notify Course Stakeholders           | Send notifications (email/SMS) to admins and learners on course changes.                                  |
| L1               | Manage Course Bookings               | Support end-user booking requests, validation and approval workflows.                                     |
| L2               | Capture Booking Requests             | Let users request/book courses and assign initial “Proposed” status.                                      |
| L2               | Validate Booking Requests            | Automatically validate requests against booking rules and course priority.                                |
| L2               | Decide Booking Outcome               | Automatically accept or reject bookings based on validation results.                                      |
| L2               | Approve Bookings Manually            | Allow administrators to review details and change status to “Approved”.                                   |
| L2               | Notify Booking Stakeholders          | Inform users and admins about booking status changes via email/SMS.                                       |
| L1               | Authenticate and Authorize Users     | Control access to LMS features for learners, admins, and vendors.                                         |
| L2               | Authenticate Users via SSO           | Integrate with corporate SSO for user login; avoid local password storage.                                |
| L2               | Authorize Roles and Permissions      | Enforce role-based access control for admin, vendor, and audit functions.                                 |
| L1               | Integrate Third-Party Suppliers      | Enable external training vendors to manage their course opportunities.                                    |
| L2               | Manage Supplier Opportunities        | Allow suppliers to create, edit, and retire their course offerings.                                       |
| L2               | Notify Supplier Changes              | Inform suppliers and administrators of changes in opportunities.                                          |
| L1               | Search Courses                       | Provide flexible search capabilities over the course and booking catalog.                                 |
| L2               | Search Course Catalog                | Filter and search courses by attributes (type, priority, vendor, dates, etc.).                            |
| L2               | Search Bookings                      | Filter and search bookings for users and admins by defined criteria.                                      |
| L1               | Report on Learning Activities        | Provide stakeholders with analytical and operational reports.                                             |
| L2               | Report on Bookings                   | Generate and download reports about booking volumes and statuses.                                         |
| L2               | Report on Vendors                    | Generate and download reports on vendor performance and course usage.                                     |
| L2               | Report on System Usage               | Provide usage statistics across users, courses and suppliers.                                             |
| L1               | Operate and Monitor LMS Platform     | Run, monitor and support the LMS in production (implicit but required).                                   |
| L2               | Monitor System Health                | Track availability, errors, and performance metrics for LMS components.                                   |
| L2               | Manage Configuration and Rules       | Maintain business rules for validation, priority, notifications and integrations.                         |

---

# 2. Utility Tree

### 2.1 Utility Tree (Quality Attributes and Scenarios)

Legend: **(Importance, Difficulty/Risk)** — H/M/L.

## Security
- Enforce SSO-only authentication for all internal users; no local password storage. **(H, M)**
- Ensure all external communication uses TLS with strong ciphers. **(H, L)**
- Provide fine-grained, role-based authorization for admin, vendor, and audit functions. **(H, M)**
- Log and audit all security-relevant actions (auth, role changes, booking approval decisions). **(H, M)**

## Performance
- Course search returns results ≤ 2s for 95% of requests for up to N courses. **(H, M)**
- Booking validation and decision complete ≤ 3s for 95% of requests. **(H, M)**
- Reports generated/down­loaded ≤ 10s for 90% of requests. **(M, M)**

## Availability & Reliability
- 99.5% uptime for core functions during business hours. **(H, H)**
- No booking or course changes lost during transient failures of external systems. **(H, M)**
- Recover from service failure with no data loss; max manual failover ≤ 30 min. **(M, M)**

## Usability
- Admin can create/update a course in ≤ 5 minutes. **(H, L)**
- End-user booking submission in ≤ 3 steps. **(M, M)**
- Clear, localized messages for validation/approval outcomes. **(M, L)**

## Modifiability
- Validation and booking rules must be configurable without code changes. **(H, H)**
- Onboard new vendor integration in ≤ 5 person-days. **(M, H)**
- Add new notification channels with minimal changes. **(M, M)**

## Interoperability & Integration
- Support SSO integrations via standard protocols (SAML/OIDC). **(H, M)**
- Integrate with at least one email/SMS gateway. **(H, M)**
- Integrate with at least one external vendor API. **(M, H)**

## Auditability & Reporting
- Provide audit trails exportable for compliance. **(H, M)**
- Provide parameterized operational reports. **(M, M)**

---

# 3. Architecture Significant Requirements (ASR), Quality Attributes, Constraints

## 3.1 ASR (Architecture Significant Requirements)

| ID          | Category      | ASR                                                                                                      | Rationale                                                                                   |
|-------------|---------------|-----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| ASR-SEC-01  | Security      | Enforce SSO-only authentication; no local password storage.                                              | Determines identity protocol, session handling, integration components.                     |
| ASR-SEC-02  | Security      | All communication must use TLS with strong cipher suites.                                                | Influences infrastructure, API gateways, deployment.                                        |
| ASR-SEC-03  | Security      | Must support fine-grained role-based authorization.                                                      | Impacts API design, domain model, permission system.                                        |
| ASR-PERF-01 | Performance   | Course search ≤ 2s for 95% of requests.                                                                  | Drives DB indexing, caching, possible search engine choice.                                 |
| ASR-PERF-02 | Performance   | Booking validation completed ≤ 3s for 95% of requests.                                                   | Impacts workflow design (sync/async), rule evaluation engine.                               |
| ASR-AVAIL-01| Availability  | Core LMS uptime must be ≥ 99.5% during business hours.                                                   | Drives redundancy, cluster setup, monitoring.                                               |
| ASR-MOD-01  | Modifiability | Business rules must be modifiable without code change.                                                   | Impacts rules engine/DSL, dynamic configuration subsystem.                                  |
| ASR-INTEG-01| Integration   | Must integrate with corporate SSO and notification gateway.                                              | Determines boundary services and integration patterns.                                      |
| ASR-AUD-01  | Auditability  | Must provide auditable logs for course changes, booking decisions, approvals.                            | Requires structured logging, audit storage, retention policy.                               |

---

## 3.2 Quality Attribute Requirements

| ID          | Quality Attribute | Requirement                                                                                          |
|-------------|-------------------|------------------------------------------------------------------------------------------------------|
| QA-SEC-01   | Security          | Only authenticated users may access LMS; role-based access required.                                |
| QA-SEC-02   | Security          | Security events must be timestamped and logged.                                                     |
| QA-PERF-01  | Performance       | Report generation ≤ 10s for typical periods.                                                        |
| QA-AVAIL-01 | Availability      | No loss of booking or course changes during external system failures.                              |
| QA-USAB-01  | Usability         | Admin can create/update a course within 5 minutes.                                                  |
| QA-USAB-02  | Usability         | Booking submission in ≤ 3 steps.                                                                    |
| QA-MOD-01   | Modifiability     | Adding new course attributes requires minimal DB/UI changes.                                        |
| QA-INTEG-01 | Interoperability  | System supports multiple notification channels.                                                     |
| QA-REP-01   | Auditability      | Audit logs must be exportable.                                                                      |

---

## 3.3 Constraints

| ID          | Type            | Constraint                                                                                                         | Source |
|-------------|-----------------|---------------------------------------------------------------------------------------------------------------------|--------|
| CON-BUS-01  | Business        | LMS must support internal, security and vendor courses in a single platform.                                       | Business |
| CON-BUS-02  | Business        | System must send notifications via SMS/email.                                                                      | Stakeholders |
| CON-ORG-01  | Organizational  | Must integrate with corporate SSO; new IdP cannot be introduced.                                                   | IT/Security |
| CON-ORG-02  | Organizational  | Must comply with corporate logging and audit policies.                                                             | Compliance |
| CON-TECH-01 | Technical       | Must run within existing corporate infrastructure (cloud/on-prem as defined).                                      | Enterprise Architecture |
| CON-TECH-02 | Technical       | Must use standard SSO protocols and secure transport.                                                              | Architecture Policy |
| CON-TECH-03 | Technical       | Integrations must use external vendors’ existing APIs.                                                             | Vendors |
| CON-OPS-01  | Operational     | Must support monitoring/logging with existing operational tools.                                                   | DevOps |

