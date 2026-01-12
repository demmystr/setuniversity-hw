# LMS — Architectural Views (3 Views)

> **Case:** Learning Management System (LMS) for internal courses, workshop recordings, security courses, and vendor courses.  
> **Deliverable:** 3 different views relevant to architectural design, with view briefs + legends.  
> **Notation:** Informal + Mermaid (GitHub-ready) and BPMN-like flow (Mermaid).

---

## 1) Context View — System Context

### View brief
This view defines the **system boundary** of the LMS and shows the **primary external actors** and **external systems** it interacts with.  
It helps clarify **who** uses the system and **which integrations** are required, without implementation details.

### Diagram legend
- **Actor** — external user role (human)
- **External System** — outside the LMS boundary
- **LMS** — target system boundary
- **Arrow (→)** — primary interaction direction

### Diagram (Mermaid)
```mermaid
flowchart LR
    Learner[End User / Learner]
    Admin[Administrator]
    Vendor[Training Vendor]
    IdP[Corporate IdP </br>SSO: SAML/OIDC]
    Notify[Email / SMS Gateway]

    subgraph LMS[Learning Management System]
        Courses[Course Management]
        Booking[Course Booking]
        Reports[Reporting]
        Search[Search]
    end

    Learner -->|Search, Book, View| LMS
    Admin -->|Create, Update, Approve| LMS
    Vendor -->|Manage Opportunities| LMS

    LMS -->|Authenticate| IdP
    LMS -->|Send Notifications| Notify
```
```mermaid
flowchart TB
    Auth[SSO Authentication Service]
    Course[Course Management Service create/update/priority]
    Validation[Validation Engine rules & priority]
    Booking[Booking & Approval Service]
    Notify[Notification Service email/SMS]
    Search[Search Service]
    Reports[Reporting Service]

    Auth --> Course
    Auth --> Booking

    Course -->|Course Updated| Validation
    Validation -->|Revalidate non-paid courses| Course

    Booking -->|Validate booking request| Validation
    Booking -->|Booking status changed| Notify
    Course -->|Course edited| Notify

    Course -->|Index / query| Search
    Booking -->|Booking data| Reports
    Course -->|Course data| Reports
    Search -->|Search criteria| Reports
```
