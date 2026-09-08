---
status: Draft
---

# emr-reporting

## System Overview

### Purpose
EMR Reporting is a UK-sovereign, cloud-native automated medical reporting and Subject Access Request (SAR) extraction platform for NHS General Practice. It ingests longitudinal patient records via the OneAdvanced Health Lake warehouse (abstracting underlying GP systems such as EMIS Web, TPP SystmOne, and Vision), executes AI-assisted third-party redaction under UK GDPR and Data Protection Act 2018 standards, synthesizes structured medical reports (DVLA, ABI, DWP, bespoke legal letters), and enforces an automated payment escrow release mechanism.

### Architecture Style
Event-driven, modular microservices architecture utilizing UK-sovereign managed cloud containers, asynchronous message queues, and an ephemeral processing pipeline to ensure zero persistent storage of unredacted clinical data outside secure enclaves.

### Key Design Principles
- **EHR Agnosticism via OneAdvanced Health Lake**: Decouple ingestion from proprietary GP software architectures by interfacing directly with OneAdvanced Health Lake normalized schemas.
- **Caldicott Principles by Design**: Enforce purpose justification, minimum necessary data extraction, strict role-based access, and complete auditability for all patient identifiable data (PID).
- **UK GDPR & Data Protection Act 2018 Compliance**: Provide automated third-party redaction, immutable access logging, and prompt Subject Access Request fulfillment within statutory timelines.
- **Data Minimization & Ephemeral Processing**: Process clinical health records in-memory or within encrypted ephemeral volumes, caching only during active review sessions with automatic TTL expiry.
- **Escrow-Gated Revenue Protection**: Withhold commercial medical reports behind a cryptographic release token that unlocks only upon verified settlement from the integrated billing gateway.

## System Context

### External Systems
- **OneAdvanced Health Lake**: Clinical data warehouse providing normalized, vendor-agnostic access to primary care EHR data (consultations, SNOMED-CT coded entries, medications, lab investigations) via secure REST/FHIR interfaces.
- **NHS CIS2 (Care Identity Service 2)**: National authentication and identity service providing NHS Smartcard and federated OpenID Connect SSO for GP practice staff.
- **Payment Gateway (Stripe / GoCardless)**: Commercial billing engine processing credit card and direct debit/BACS payments from third-party requestors with webhook event dispatch.
- **NHS Notify & SendGrid**: Secure transaction email and notification service for dispatching report readiness notifications, GP action reminders, and secure download links.

### Users & Actors
- **GP Practice Business Manager & Medical Secretary**: Initiates reporting workflows, maps incoming requests to templates, reviews automated redactions, and manages practice billing queues.
- **General Practitioner (GP Partner / Salaried GP)**: Clinically reviews side-by-side extracted evidence, verifies redaction masks, signs off reports digitally, and authorizes escrow release.
- **Commercial / Legal Requestor**: Insurers, solicitors, and employers who submit medical report requests, track status, pay invoices, and retrieve finalized reports from the secure portal.
- **Data Subject (Patient)**: Submits Subject Access Requests (SARs) and accesses redacted clinical summaries in compliance with UK GDPR rights.

### Components
#### Health Lake Ingestion Gateway
- **Responsibility**: Ingests, normalizes, and filters structured and unstructured clinical data from OneAdvanced Health Lake based on the specific scope of the medical report request.
- **Technology**: Node.js / TypeScript, AWS ECS Fargate, Axios/FHIR client.
- **Interfaces**: REST API endpoints for ingestion triggers, AWS SQS event publisher for raw clinical extraction jobs.

#### Clinical NLP & Redaction Engine
- **Responsibility**: Scans extracted clinical narratives to identify third-party identifiable references (family members, non-clinical personnel, safeguarding markers) and applies automated redaction masks.
- **Technology**: Python, FastAPI, Hugging Face Transformers (BioBERT / UK Clinical NLP models), AWS ECS Fargate.
- **Interfaces**: Internal gRPC/REST API for redaction analysis, emitting entity bounding tokens and confidence scores.

#### Report Synthesis & Template Engine
- **Responsibility**: Compiles extracted SNOMED-CT timelines, questionnaire answers, and bespoke letter drafts into standardized PDF/A documents and structured digital forms (DVLA, ABI, DWP).
- **Technology**: Node.js, Puppeteer, React-PDF, Handlebars templating engine.
- **Interfaces**: Asynchronous job worker consuming from extraction queues; outputs encrypted report packages to staging storage.

#### GP Review & Verification Service
- **Responsibility**: Serves the side-by-side verification interface for GPs, managing interactive redaction adjustments, digital signature cryptographic stamping, and workflow state transitions.
- **Technology**: Next.js frontend, Node.js API layer, PostgreSQL with Row-Level Security.
- **Interfaces**: WebSocket/REST API for real-time draft state updates and audit event logging.

#### Commercial Billing & Escrow Gateway
- **Responsibility**: Generates statutory/private invoices, tracks payment clearance via gateway webhooks, and manages the cryptographic release keys for reports held in escrow.
- **Technology**: Node.js, Stripe SDK, PostgreSQL.
- **Interfaces**: Inbound webhook endpoints from Stripe, secure tokenized download API for external requestors.

## Data Architecture

### Data Models
- **Report Request Entity**: Captures request metadata (Requestor ID, Patient NHS Number, Report Type, SLA Target Date, Escrow Status, Assigned GP).
- **Clinical Chronology & Extraction Snippet**: Structured SNOMED-CT code mappings, encounter dates, clinical notes, practitioner attribution, and data source timestamps.
- **Redaction Mask & Audit Entity**: Bounding coordinates/character offsets of redacted third-party text, entity classification (Third-Party Family, Safeguarding, Irrelevant Health Data), and review decision history.
- **Escrow Ledger & Invoice**: Commercial fee breakdown, VAT calculations, payment gateway transaction IDs, escrow lock state, and release authorization timestamps.

### Data Flow
1. **Request Intake**: Medical secretary logs a report request or SAR in the worklist, specifying report parameters and patient NHS number.
2. **Health Lake Ingestion**: The Ingestion Gateway queries OneAdvanced Health Lake via secure mutual TLS, retrieving relevant encounters, codes, and investigations.
3. **NLP Redaction & Parsing**: Extracted narratives are processed through the NLP Redaction Engine, identifying and flagging third-party names and sensitive data.
4. **Template Compilation**: The Synthesis Engine maps structured clinical data into target form fields (e.g., DVLA questionnaire) and presents side-by-side evidence to the secretary.
5. **Clinical Sign-Off**: The GP reviews the draft side-by-side against source data, confirms redactions, applies a digital signature, and seals the report into escrow.
6. **Payment & Release**: The external requestor settles the invoice via the billing portal; upon webhook confirmation, the escrow lock releases and delivers the report.

### Storage Strategy
- **Primary Database**: AWS Aurora Serverless v2 PostgreSQL (UK South / London region) with Row-Level Security (RLS) and transparent column-level encryption for metadata, workflow states, and audit trails.
- **Caching**: AWS ElastiCache for Redis for short-lived session storage, rate limiting, and ephemeral extraction buffers (TTL < 4 hours).
- **File Storage**: AWS S3 with Object Lock, SSE-KMS customer-managed keys (CMK) within the UK sovereign region for temporary encrypted PDF artifact staging.

## API Design

### API Style
RESTful JSON APIs over HTTPS (TLS 1.3) for administrative and clinical user interactions, complemented by asynchronous webhook handlers for Health Lake data readiness and payment events.

### Key Endpoints
- `POST /api/v1/requests`: Create a new reporting or SAR workflow request with patient and template parameters.
- `GET /api/v1/requests/{id}/healthlake-sync`: Initiate and poll asynchronous clinical data extraction from OneAdvanced Health Lake.
- `POST /api/v1/reports/{id}/redact`: Submit automated redaction verification and apply manual masking overrides.
- `POST /api/v1/reports/{id}/sign-off`: Submit GP digital signature, Caldicott approval record, and lock report into escrow.
- `POST /api/v1/billing/webhooks`: Process payment gateway settlement webhooks to trigger cryptographic release from escrow.

### Authentication & Authorization
- **NHS Staff Authentication**: Federated NHS CIS2 (OpenID Connect) with mandatory NHS Smartcard / multi-factor authentication.
- **Authorization & RBAC**: Granular role-based access control separating Medical Secretary (drafting), GP Partner (sign-off/Caldicott review), and Practice Business Manager (billing).
- **External Requestor Authentication**: OAuth 2.0 client credentials and magic-link multi-factor authentication for commercial requestors accessing the escrow download portal.

## Infrastructure

### Deployment Architecture
UK-sovereign AWS eu-west-2 (London) deployment across multiple Availability Zones. Services run as containerized microservices on AWS ECS Fargate within isolated VPC private subnets, compliant with the NHS Data Security and Protection Toolkit (DSPT) and DCB0129 standards.

### Scaling Strategy
- **Horizontal**: Auto-scaling ECS Fargate task instances based on SQS queue depth and CPU/memory utilization (>70%).
- **Vertical**: Dynamic Aurora Serverless v2 ACU scaling (0.5 to 16 ACUs) accommodating variable analytical query loads.
- **Auto-scaling**: Predictive scheduled scaling aligned with UK primary care core operating hours (08:00 to 18:30 GMT).

### Monitoring & Observability
- **Logging**: AWS CloudWatch and AWS OpenSearch structured JSON logging with strict PII filters (zero clinical/patient identifiable data logged). Dedicated immutable Caldicott audit trail log group.
- **Metrics**: Prometheus and CloudWatch metrics tracking request fulfillment latency, NLP extraction confidence, Health Lake API response times, and escrow settlement rates.
- **Alerting**: CloudWatch Alarms integrated with PagerDuty for SLA risk thresholds (SAR 30-day countdowns), Health Lake gateway disconnects, and billing webhook anomalies.

## Security Architecture

### Security Layers
- **Perimeter & Network Security**: AWS WAF with rate-limiting, geo-restriction (UK-only ingress for clinical administration), AWS Shield standard DDoS protection, and TLS 1.3 encryption across all public interfaces.
- **Caldicott Principles & UK GDPR Safeguards**: Enforced data minimization, role-specific access segregation, Caldicott Guardian audit logging for every PID access event, and automated identification of third-party references.
- **Data Protection & Cryptography**: AES-256 encryption at rest using AWS KMS Customer Managed Keys (CMK), strict ephemeral memory processing with zero persistent raw record caching, and cryptographically signed report hashes for legal chain of custody.