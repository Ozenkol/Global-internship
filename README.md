# 🌍 Global Internship Marketplace – System Design Report

## 🔰 Project Goal
Design a globally distributed system that connects **international students** with **internship opportunities** worldwide. The system should automate application processes, document verification, and legal compliance across borders.

---

Category | Non-Functional Requirement | Reason / Justification
🎯 Accuracy | High data accuracy & integrity | Legal documents, visa forms, contracts must be 100% accurate — errors can block applications or cause legal issues.
⚖️ Compliance | Data residency & regulation compliance (e.g., GDPR, HIPAA, etc.) | Mandatory to operate internationally — student data must stay within specific regions.
⚙️ Consistency | Strong consistency for critical workflows (e.g., document submission, status tracking) | Students must see the true status of their applications at all times. Documents must not go out of sync.
🚀 Scalability | Scalable to global user base (millions of students/employers) | Especially during peak cycles like summer internships or job fairs.
💡 Availability | High availability (ideally 99.9%+) | Employers and students may come from different time zones. Downtime directly impacts trust and usage.
🔐 Security & Privacy | End-to-end encryption, role-based access, consent management | Users upload highly sensitive documents (IDs, visas, contracts). Any leak = reputation + legal disaster.
📦 Auditability | Complete audit logs for legal actions (submission, editing, consent) | Required by law for disputes, inspections, or internal compliance.
🧠 Adaptability | Support for country-specific rule changes | Laws and visa requirements change frequently. System must evolve without downtime.
🌍 Localization | Multilingual, culturally adaptive UI | Targeting a global audience — students from over 100 countries.
⏱ Latency | Low-latency access to localized services | Delays in form submission or status changes hurt user experience. Especially critical in real-time interview scheduling or uploads.
🔄 Resilience | Fault tolerance & graceful degradation | One region going down should not take down the global system. Retry mechanisms and failovers must be in place.
📣 Observability | Monitoring, alerting, and tracing | Needed for compliance, quick issue resolution, and user trust.

## 🧠 Problem Analysis

### Key Challenges
- Legal complexity across jurisdictions (visa, labor laws, document formats)
- Global availability and scalability
- Accuracy in sensitive data and compliance
- Routing applications through the correct verification/legal channels
- Secure authentication and document storage

---

## 🧱 System Components

### Software Services (Modular Monolith + Externalized Engines)
- **User Service**: Registration, authentication (OAuth2), profiles
- **Application Service**: Submitting & tracking internship applications
- **Matching Engine**: Filters and ranks internship offers based on skills, location, language
- **Document Service**: File uploads, format checks, OCR, checksum validation
- **Verification Service**: Interfaces with embassies, universities, and background checks
- **Legal Compliance Engine**: Evaluates legal fit, flags violations per jurisdiction
- **Workflow Orchestrator**: Saga pattern coordinator for multistep operations
- **Notification Service**: Sends SMS, emails, system alerts
- **Admin & Analytics Dashboard**: For staff, support, and monitoring
- **Audit Logger**: Stores immutable logs for legal reviews

---

## ⚙️ Databases & Storage Components

| Storage          | Purpose                                               |
|------------------|--------------------------------------------------------|
| **PostgreSQL**    | Users, applications, employer profiles, status logs  |
| **Redis**         | Caching, session tokens, rate limiting               |
| **Object Store**  | File storage (CVs, passport scans, offers) via S3    |
| **Elasticsearch** | Full-text search on internships, users, companies    |
| **Kafka or Loki** | Append-only legal logs and event sourcing            |
| **Data Lake**     | Aggregated logs, matching model training data        |

---

## 📈 Data Flow

1. **Student Signup**  
   → User Service → PostgreSQL → Auth Token (Redis)  
   → Notification Service (Email Verification)

2. **Upload Documents**  
   → Document Service  
   → S3 Storage + Hash/format check  
   → Verification Service → External APIs (e.g., University registry)

3. **Apply for Internship**  
   → Application Service → PostgreSQL  
   → Legal Compliance Engine validates visa/work permissions  
   → Audit Logger stores steps

4. **Recommendation Flow**  
   → Matching Engine pulls user & company data  
   → Query Elasticsearch  
   → Return scored results

5. **Workflow Engine for Multi-step Ops**  
   → Saga Manager coordinates across services  
   → Retry/fallback logic per service  
   → Log events to Kafka

---

## ⚖️ Non-Functional Requirements

| Requirement       | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| ✅ **Data Accuracy**    | Must ensure strict document and user info correctness                   |
| ✅ **Scalability**      | Handle global growth, autoscaling backend containers                    |
| ✅ **High Availability**| Failover setups, multi-region deployments                                |
| ✅ **Consistency**      | Strong consistency for core data; eventual for audit/logging            |
| ✅ **Legal Auditability**| Immutable, tamper-proof logs for document verification and workflow     |
| ✅ **Security & Privacy**| End-to-end encryption, zero-trust auth, GDPR-compliant policies         |

---

## 🧱 Architecture Pattern

### ➤ Adopted: **Modular Monolith with Distributed Services**

- Clear module boundaries within single backend unit
- Pluggable services for document validation, notification, legal compliance
- Common DB ensures consistency
- External services handle region-specific logic

---

## 📐 C4 Model Summary

### Level 1: System Context
- Users: Students, Employers, Legal Validators
- External: OAuth, University APIs, Embassies, Payment Providers

### Level 2: Container View

| Container         | Description                                                |
|------------------|------------------------------------------------------------|
| Web/Mobile Frontend | Vue/Nuxt or React PWA                                    |
| Backend API       | Modular monolith exposing REST/gRPC endpoints             |
| Legal Engine      | Rules & workflows per country                             |
| Document Service  | Handles uploads and validations                           |
| Workflow Engine   | Coordinates services for multi-step flows                 |
| Notification Engine| Emails, SMS, push notifications                          |
| PostgreSQL        | User and app data store                                   |
| S3-Compatible Store| Files, PDFs, CVs                                         |
| Redis             | Session and temporary cache                               |
| Kafka             | Event & audit logs                                        |
| Elasticsearch     | Search and ranking                                        |

### Level 3: Component View (App Module Example)

| Component           | Purpose                                        |
|---------------------|------------------------------------------------|
| `ApplicationController` | API for applying to internships               |
| `MatchingService`      | Ranks internships by profile fit             |
| `ComplianceService`    | Calls legal engine for jurisdictional checks |
| `DocumentService`      | Upload, OCR, and verification                |
| `SagaOrchestrator`     | Coordinates process steps                    |
| `AuditLogger`          | Logs all actions for compliance              |

---

## 🧭 Deployment Architecture

- Deployed via Kubernetes on multi-region cloud (e.g., AWS, GCP)
- CDN (Cloudflare, CloudFront) for frontend & document delivery
- Load balancer and service mesh (Istio/Linkerd) for secure routing
- Secrets managed via Vault
- Centralized observability stack (Grafana + Prometheus + Loki)

---

## 📁 Folder Structure Suggestion

