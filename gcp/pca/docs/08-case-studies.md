# PCA Case Studies Reference

> **Exam relevance:** The PCA exam includes 4 published case studies. Two of these will appear on your exam. You can review the case study during the exam, but reading it during the test wastes time. Memorize the key requirements and recommended architectures.

> **Strategy:** For each case study, focus on: (1) What are the business/technical requirements? (2) What GCP services map to each requirement? (3) What are the common exam traps?

> **Time tip:** Case study questions are not grouped together. They appear scattered throughout the exam. A button lets you open the case study text in a side panel. If you have memorized the case studies, you save 10-15 minutes of reading time during the exam.

**Docs:** [Official PCA Case Studies](https://cloud.google.com/learn/certification/cloud-architect)

---

## Table of Contents

1. [Case Study 1: EHR Healthcare](#case-study-1-ehr-healthcare)
2. [Case Study 2: Cymbal Retail](#case-study-2-cymbal-retail)
3. [Case Study 3: Altostrat Media](#case-study-3-altostrat-media)
4. [Case Study 4: KnightMotives Automotive](#case-study-4-knightmotives-automotive)
5. [Cross-Case Study Exam Strategy](#cross-case-study-exam-strategy)
6. [Universal Analysis Framework](#universal-analysis-framework)
7. [Quick Recall Cheat Sheet](#quick-recall-cheat-sheet)

---

## Case Study 1: EHR Healthcare

**Official doc:** [EHR Healthcare Case Study](https://services.google.com/fh/files/blogs/master_case_study_ehr_healthcare.pdf)

### Company Overview

EHR Healthcare is a leading provider of electronic health record (EHR) software to the medical industry. They serve a rapidly growing customer base across multiple hospital systems in the United States. Their current infrastructure is hosted in a colocation facility, and they are experiencing growth pains that prevent them from onboarding new customers quickly.

**Key facts to memorize:**
- Colocation facility (NOT their own data center -- they rent space)
- Rapidly growing customer base -- scalability is critical
- Healthcare industry -- HIPAA compliance is non-negotiable
- Multi-tenant SaaS model serving multiple hospital systems

### Solution Concept

EHR Healthcare wants to migrate their existing applications and infrastructure to Google Cloud to improve scalability, reliability, and security while maintaining HIPAA compliance. The migration must be phased -- they cannot afford downtime during the transition, and some workloads will remain on-premises temporarily during the migration window.

### Existing Technical Environment

```
Current State (Colocation):
- Mix of containerized (Docker) and VM-based workloads
- Windows Server workloads (legacy .NET applications)
- Linux servers for newer services
- On-premises Microsoft Active Directory for identity management
- MS SQL Server databases (primary EHR data store)
- MySQL databases (supporting services, analytics)
- Current SLA commitment: 99.9% availability to hospital customers
- HL7v2 and FHIR APIs for hospital system integrations
- Custom-built APIs for data exchange
- On-premises backup with tape rotation
- Manual scaling -- capacity planning done quarterly
```

**Environment red flags for exam:**
- Active Directory = identity integration challenge
- Windows + Linux = heterogeneous compute requirement
- SQL Server + MySQL = multiple database migration paths
- HL7/FHIR = healthcare-specific interoperability standards
- 99.9% SLA = must maintain or exceed during and after migration

### Business Requirements

| # | Requirement | Priority | Key Implication |
|---|------------|----------|-----------------|
| B1 | Maintain HIPAA compliance throughout migration and ongoing operations | Critical | BAA, encryption, audit logs, access controls |
| B2 | Onboard new hospital customers faster (currently takes months) | High | Automation, IaC, multi-tenant architecture |
| B3 | Achieve 99.9% availability SLA for core EHR application | Critical | HA architecture, multi-zone, health checks |
| B4 | Reduce operational overhead and infrastructure management burden | High | Managed services, GKE Autopilot, Cloud SQL |
| B5 | Provide data analytics capabilities for population health insights | Medium | BigQuery, de-identification, analytics tools |
| B6 | Support mobile access for healthcare providers | Medium | API gateway, Identity-Aware Proxy, mobile backend |
| B7 | Reduce infrastructure costs through better resource utilization | Medium | Autoscaling, committed use discounts, right-sizing |

### Technical Requirements

| # | Requirement | Implementation Detail |
|---|------------|----------------------|
| T1 | Migrate containerized workloads to managed Kubernetes | GKE Autopilot with private clusters |
| T2 | Hybrid connectivity between colocation and GCP during migration | Dedicated Interconnect (HA pair) |
| T3 | Active Directory integration with Google Cloud IAM | GCDS + SAML SSO via Cloud Identity |
| T4 | Encrypt all data at rest and in transit (HIPAA) | CMEK with Cloud KMS, TLS 1.2+ everywhere |
| T5 | Audit logging for all access to patient data | Cloud Audit Logs + Access Transparency |
| T6 | DR plan with RPO < 1 hour, RTO < 4 hours | Cross-region replication, automated failover |
| T7 | Support for Windows Server workloads | Compute Engine with Windows images |
| T8 | Database migration with minimal downtime | Database Migration Service (DMS) |
| T9 | FHIR/HL7 interoperability | Cloud Healthcare API |
| T10 | Automated infrastructure provisioning | Terraform + Cloud Build CI/CD |

### Executive Statement

> "Our current infrastructure is limiting our growth. We need to modernize to handle our expanding customer base while ensuring we never compromise on patient data security."

**What this tells you:**
- Growth is the primary driver (not cost reduction)
- Security/compliance is a hard constraint (never compromise)
- Modernization is desired, not just lift-and-shift
- Executive sponsor supports cloud migration

### Architecture Diagram

```
                            ┌─────────────────────────────────────────────────────────────┐
                            │                    GOOGLE CLOUD                              │
                            │                                                             │
                            │  ┌─────────────────────────────────────────────────────┐    │
                            │  │              Shared VPC (Hub)                        │    │
                            │  │                                                     │    │
                            │  │   ┌───────────────┐    ┌───────────────────────┐    │    │
                            │  │   │  Cloud DNS     │    │  Cloud NAT            │    │    │
                            │  │   │  (Private)     │    │  (Egress)             │    │    │
                            │  │   └───────────────┘    └───────────────────────┘    │    │
                            │  └─────────────────────────────────────────────────────┘    │
                            │           │                        │                        │
                            │  ┌────────┴────────┐    ┌─────────┴───────────┐            │
                            │  │ Service Project 1│    │ Service Project 2   │            │
                            │  │ (Production)     │    │ (Analytics)         │            │
  ┌──────────────┐          │  │                  │    │                     │            │
  │  COLOCATION  │          │  │ ┌──────────────┐ │    │ ┌────────────────┐  │            │
  │              │          │  │ │ GKE Autopilot│ │    │ │ BigQuery       │  │            │
  │ ┌──────────┐ │  Cloud   │  │ │ (Private)    │ │    │ │ (PHI de-ident) │  │            │
  │ │ Docker   │ │  Inter-  │  │ │              │ │    │ │                │  │            │
  │ │ Hosts    │ │  connect │  │ │ - EHR App    │ │    │ │ ┌────────────┐│  │            │
  │ ├──────────┤ │◄────────►│  │ │ - API GW     │ │    │ │ │ Looker     ││  │            │
  │ │ Windows  │ │  (HA)    │  │ │ - FHIR Proxy │ │    │ │ │ Dashboards ││  │            │
  │ │ Servers  │ │          │  │ └──────────────┘ │    │ │ └────────────┘│  │            │
  │ ├──────────┤ │          │  │                  │    │ └────────────────┘  │            │
  │ │ Active   │ │          │  │ ┌──────────────┐ │    └────────────────────┘            │
  │ │ Directory│ │          │  │ │ Compute Eng. │ │                                      │
  │ ├──────────┤ │          │  │ │ (Windows VMs)│ │    ┌──────────────────────┐           │
  │ │ MS SQL   │ │          │  │ └──────────────┘ │    │ Security Layer       │           │
  │ │ Server   │ │          │  │                  │    │                      │           │
  │ ├──────────┤ │          │  │ ┌──────────────┐ │    │ - VPC Service Ctrls  │           │
  │ │ MySQL    │ │          │  │ │ Cloud SQL    │ │    │ - Cloud KMS (CMEK)   │           │
  │ └──────────┘ │          │  │ │ (MySQL + SQL │ │    │ - Cloud Armor        │           │
  └──────────────┘          │  │ │  Server)     │ │    │ - IAP                │           │
                            │  │ └──────────────┘ │    │ - Audit Logs         │           │
                            │  │                  │    │ - Access Transparency │           │
                            │  │ ┌──────────────┐ │    │ - Assured Workloads  │           │
                            │  │ │ Cloud        │ │    │ - DLP API            │           │
                            │  │ │ Healthcare   │ │    └──────────────────────┘           │
                            │  │ │ API (FHIR)   │ │                                      │
                            │  │ └──────────────┘ │    ┌──────────────────────┐           │
                            │  └──────────────────┘    │ Identity             │           │
                            │                          │                      │           │
                            │                          │ - Cloud Identity     │           │
                            │                          │ - GCDS (AD sync)     │           │
                            │                          │ - SAML SSO           │           │
                            │                          │ - Workforce IdP      │           │
                            │                          └──────────────────────┘           │
                            │                                                             │
                            │  ┌─────────────────────────────────────────────────────┐    │
                            │  │ DR Region (Secondary)                                │    │
                            │  │  - Cloud SQL read replicas (cross-region)            │    │
                            │  │  - GKE standby cluster                               │    │
                            │  │  - Cloud Storage regional backup                     │    │
                            │  └─────────────────────────────────────────────────────┘    │
                            └─────────────────────────────────────────────────────────────┘
```

### Detailed Service Mapping

| Requirement | GCP Service(s) | Why This Service | Alternative (and why NOT) |
|------------|----------------|------------------|---------------------------|
| Container orchestration | **GKE Autopilot** (private cluster) | Managed K8s reduces ops burden (B4), private cluster for HIPAA (B1) | GKE Standard -- more ops overhead, contradicts B4 |
| Windows workloads | **Compute Engine** (Windows Server images) | Only option for Windows VMs on GCP | Cloud Run/GKE -- no native Windows container support on GKE |
| MySQL migration | **Cloud SQL for MySQL** + **Database Migration Service** | Managed MySQL, DMS for minimal-downtime migration (T8) | AlloyDB -- overkill for supporting services; Spanner -- cost/complexity |
| SQL Server migration | **Cloud SQL for SQL Server** | Managed, HIPAA-eligible, handles licensing | SQL Server on CE -- more ops overhead; bare metal -- not cloud-native |
| HIPAA compliance | **Assured Workloads** + **CMEK** + **VPC-SC** + **Audit Logs** | Assured Workloads = compliance guardrails; CMEK = key control; VPC-SC = data exfiltration prevention | Default encryption alone is NOT sufficient for HIPAA key management requirements |
| Identity integration | **GCDS** + **SAML SSO** + **Cloud Identity** | GCDS syncs AD to Cloud Identity; SAML for federated auth (T3) | Migrating users manually -- doesn't scale; AD on CE -- defeats purpose |
| Hybrid connectivity | **Dedicated Interconnect** (HA pair, 2x10Gbps) | Enterprise-grade, SLA-backed, private connectivity (T2) | VPN -- lower bandwidth, less reliable for production healthcare traffic |
| Data analytics | **BigQuery** + **Looker** + **DLP API** | BigQuery for population health analytics (B5); DLP for PHI de-identification | Dataproc -- not needed unless Hadoop workloads exist |
| Healthcare APIs | **Cloud Healthcare API** (FHIR R4, HL7v2) | Native FHIR/HL7 support, HIPAA-compliant, pre-built integrations (T9) | Custom APIs -- reinventing the wheel, no standards compliance |
| DR (RPO<1h, RTO<4h) | **Cross-region Cloud SQL replicas** + **GKE standby** + **Cloud Storage** | Cross-region replication meets RPO; automated failover meets RTO (T6) | Single-region HA only -- doesn't meet DR requirements |
| Audit logging | **Cloud Audit Logs** + **Access Transparency** + **Data Access Logs** | Required for HIPAA audit trail (T5); Access Transparency shows Google admin access | Only Admin Activity logs -- insufficient for HIPAA; must enable Data Access logs |
| Encryption | **CMEK** with **Cloud KMS** + TLS 1.2+ in transit | Customer-managed keys for HIPAA (T4); org policy to enforce | Google-managed keys alone -- less control for compliance auditors |
| Mobile access | **Identity-Aware Proxy (IAP)** + **Apigee** | IAP for BeyondCorp-style access (B6); Apigee for API management | VPN-only mobile access -- poor UX for clinicians |
| Faster onboarding | **Terraform** + **Cloud Build** + **Config Connector** | IaC for repeatable tenant provisioning (B2) | Manual provisioning -- takes months (current pain point) |
| Cost optimization | **Committed Use Discounts** + **Autopilot autoscaling** + **Active Assist** | CUDs for predictable workloads (B7); autoscaling for variable load | On-demand only -- overpaying for steady-state workloads |

### Migration Strategy

```
Phase 1: Foundation (Months 1-2)
├── Set up Organization, folders, projects (resource hierarchy)
├── Configure Shared VPC with private subnets
├── Establish Dedicated Interconnect (HA pair)
├── Deploy Cloud Identity + GCDS sync from AD
├── Set up Assured Workloads for HIPAA
├── Configure VPC Service Controls perimeter
└── Deploy Terraform pipelines (Cloud Build)

Phase 2: Database Migration (Months 2-4)
├── Deploy Cloud SQL for MySQL (DMS continuous replication)
├── Deploy Cloud SQL for SQL Server (DMS)
├── Validate data integrity and run parallel queries
├── Cut over MySQL (supporting services first)
└── Cut over SQL Server (EHR data -- final step)

Phase 3: Application Migration (Months 3-5)
├── Deploy GKE Autopilot (private cluster)
├── Migrate containerized workloads to GKE
├── Deploy Windows VMs on Compute Engine
├── Set up Cloud Healthcare API for FHIR/HL7
├── Configure Cloud Armor + IAP
└── Internal testing and validation

Phase 4: Analytics & Optimization (Months 5-7)
├── Deploy BigQuery dataset with PHI de-identification
├── Set up Looker dashboards for population health
├── Configure DR (cross-region replicas, standby GKE)
├── Performance testing and SLA validation
├── Decommission colocation (or reduce footprint)
└── Enable Active Assist recommendations
```

### Common Question Patterns

**Pattern 1: HIPAA Compliance Architecture**
> "EHR Healthcare needs to ensure patient data is protected. Which combination of services should they use?"
- **Answer direction:** VPC Service Controls + CMEK + Cloud Audit Logs + Assured Workloads
- Questions test whether you know that HIPAA requires layered controls, not a single service

**Pattern 2: Identity Integration**
> "How should EHR Healthcare integrate their existing Active Directory with Google Cloud?"
- **Answer direction:** GCDS to sync identities to Cloud Identity, then SAML SSO for federated authentication
- NOT: Migrate all users to Google Cloud Identity manually
- NOT: Run AD Domain Controllers on Compute Engine as the primary approach

**Pattern 3: Database Migration Path**
> "Which approach should EHR Healthcare use to migrate their SQL Server database with minimal downtime?"
- **Answer direction:** Database Migration Service for continuous replication, then cutover
- NOT: Export/import (causes downtime)
- NOT: Bare Metal Solution (overkill, more ops)

**Pattern 4: Hybrid Connectivity During Migration**
> "What connectivity solution should EHR Healthcare use between their colocation and Google Cloud?"
- **Answer direction:** Dedicated Interconnect with HA (redundant connections in 2 metro areas)
- NOT: Partner Interconnect (they need direct, high-bandwidth)
- NOT: Cloud VPN (insufficient for production healthcare traffic)

**Pattern 5: DR Design**
> "Design a DR solution that meets RPO < 1 hour and RTO < 4 hours for EHR Healthcare."
- **Answer direction:** Cross-region Cloud SQL replicas (async replication for RPO) + standby GKE cluster (pre-deployed for RTO) + automated failover with Cloud DNS
- NOT: Cold standby (RTO too slow)
- NOT: Backup/restore only (RPO too wide)

**Pattern 6: Analytics with PHI**
> "How should EHR Healthcare provide population health analytics without exposing PHI?"
- **Answer direction:** Cloud DLP API for de-identification before loading into BigQuery; authorized views for access control
- NOT: Copy raw PHI to BigQuery and rely on IAM only
- NOT: Give analysts direct access to production databases

**Pattern 7: Container Migration**
> "What is the best approach for migrating EHR Healthcare's containerized workloads?"
- **Answer direction:** GKE Autopilot (reduces ops overhead per B4) with private cluster (HIPAA), use Migrate to Containers for any VM-to-container conversions
- NOT: GKE Standard (more operational burden unless specific justification)
- NOT: Cloud Run (doesn't support the complexity of a full EHR application)

**Pattern 8: Faster Customer Onboarding**
> "How can EHR Healthcare reduce new hospital onboarding time from months to days?"
- **Answer direction:** Terraform modules for per-tenant infrastructure, Cloud Build for CI/CD, Config Controller for Kubernetes-native resource management
- NOT: Manual provisioning with console clicks

### Exam Traps

| Trap | Why It's Wrong | Correct Approach |
|------|---------------|------------------|
| "Use Google-managed encryption keys for HIPAA" | HIPAA requires demonstrable key management control; CMEK gives auditors evidence of key ownership | **CMEK with Cloud KMS**; optionally Cloud EKM for external key management |
| "Use Cloud VPN for hybrid connectivity" | Cloud VPN is insufficient bandwidth/reliability for production healthcare SLA (99.9%) | **Dedicated Interconnect** with HA configuration (2 connections, 2 metros) |
| "Migrate AD to Cloud Identity completely" | EHR still has on-premises workloads during migration; need hybrid identity | **GCDS + SAML** federation; keep AD as source of truth |
| "Use Dataflow for healthcare API integration" | Dataflow is for data processing, not healthcare interoperability standards | **Cloud Healthcare API** for FHIR/HL7v2 natively |
| "GKE Standard for more control" | Business requirement B4 explicitly says reduce operational overhead | **GKE Autopilot** unless a specific technical requirement demands Standard |

---

## Case Study 2: Cymbal Retail

**Official doc:** [Cymbal Retail Case Study](https://services.google.com/fh/files/blogs/master_case_study_cymbal_retail.pdf)

### Company Overview

Cymbal Retail is a large omnichannel retail company with both physical stores and an e-commerce presence. They are looking to use generative AI to transform their customer experience across the entire shopping journey -- from product catalog enrichment to conversational commerce to personalized recommendations.

**Key facts to memorize:**
- Large retailer with both online and physical stores
- Currently behind competitors on AI adoption
- Monolithic legacy Java application
- Oracle database (migration away from Oracle is a theme)
- Separate web and mobile backends (needs unification)
- Peak traffic events (Black Friday, holiday sales)

### Solution Concept

Modernize their technology stack and implement AI-powered solutions for catalog management, customer service, and product discovery. The core theme is becoming an "AI-first retailer" by leveraging Google Cloud's generative AI capabilities while simultaneously modernizing the underlying platform from monolith to microservices.

### Existing Technical Environment

```
Current State (On-Premises):
- Monolithic Java application (single deployable WAR file)
- Oracle Database 19c for product catalog, orders, inventory
- Traditional IVR (Interactive Voice Response) for customer support
- Basic keyword-based search on website (Solr/Elasticsearch)
- Separate mobile app with entirely different backend (Node.js)
- Manual product catalog enrichment (copywriters + photographers)
- Batch processing for inventory updates (overnight)
- On-premises CDN / caching layer
- Seasonal traffic patterns (10x spikes during Black Friday)
- PCI DSS compliance for payment processing
```

**Environment red flags for exam:**
- Monolith = needs decomposition strategy (strangler fig pattern)
- Oracle = expensive licensing, need migration to cloud-native DB
- Separate backends = API unification opportunity
- Manual catalog = AI automation opportunity
- IVR = replace with conversational AI
- Batch inventory = move to real-time event-driven
- Peak traffic = autoscaling is critical

### Business Requirements

| # | Requirement | Priority | Key Implication |
|---|------------|----------|-----------------|
| B1 | Use generative AI to automatically enrich product descriptions and metadata | Critical | Vertex AI with Gemini models |
| B2 | Replace IVR with AI-powered conversational agent | High | Vertex AI Agent Builder / Dialogflow CX |
| B3 | Improve product discovery and recommendations | High | Vertex AI Search, Recommendations AI |
| B4 | Unify web and mobile backends | High | API-first architecture, Apigee |
| B5 | Reduce time-to-market for new features | High | Microservices, CI/CD, Cloud Build |
| B6 | Improve customer satisfaction scores (CSAT) | High | AI-powered personalization |
| B7 | Maintain PCI DSS compliance | Critical | Tokenization, VPC-SC, audit logs |
| B8 | Handle 10x traffic spikes (Black Friday) | Critical | Autoscaling, CDN, Memorystore caching |

### Technical Requirements

| # | Requirement | Implementation Detail |
|---|------------|----------------------|
| T1 | Migrate from Oracle to a cloud-native database | AlloyDB (PostgreSQL-compatible) or Cloud Spanner |
| T2 | Containerize the monolithic application (microservices) | GKE Autopilot, strangler fig pattern |
| T3 | Implement AI-powered search and recommendations | Vertex AI Search + Recommendations AI |
| T4 | Build conversational AI agent for customer support | Vertex AI Agent Builder (Dialogflow CX) |
| T5 | Real-time inventory and pricing updates | Pub/Sub + Eventarc + Dataflow |
| T6 | API-first architecture for unified backends | Apigee API Management |
| T7 | Scalable during peak retail events | GKE HPA/VPA + Cloud Run + Cloud CDN |
| T8 | Payment processing (PCI DSS) | Tokenization, isolated PCI project |
| T9 | Personalization engine | Vertex AI, BigQuery ML |
| T10 | Image generation/enhancement for catalog | Vertex AI (Imagen) |

### Executive Statement

> "We need to become an AI-first retailer. Our competitors are already using AI to personalize shopping experiences, and we're falling behind."

**What this tells you:**
- AI is the top strategic priority
- Competitive pressure is the driver
- Speed of adoption matters (falling behind)
- Personalization is the key AI use case
- Executive buy-in for AI investment

### Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                              GOOGLE CLOUD                                        │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────────┐  │
│  │                         Frontend / Edge Layer                              │  │
│  │                                                                            │  │
│  │   ┌──────────────┐   ┌──────────────┐   ┌──────────────────────────┐      │  │
│  │   │ Cloud CDN    │   │ Cloud Armor  │   │ Global HTTPS LB          │      │  │
│  │   │ (Static +    │   │ (WAF, DDoS)  │   │ (Web + Mobile + API)     │      │  │
│  │   │  Dynamic)    │   └──────────────┘   └──────────────────────────┘      │  │
│  │   └──────────────┘                                                         │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                         API Layer                                          │  │
│  │                                                                            │  │
│  │   ┌──────────────────────────────────────────────────────────────────┐     │  │
│  │   │                    Apigee API Management                         │     │  │
│  │   │  (Unified API gateway for web, mobile, partner, internal APIs)   │     │  │
│  │   └──────────────────────────────────────────────────────────────────┘     │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                      Microservices Layer                                   │  │
│  │                                                                            │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │  │
│  │  │ Catalog     │ │ Orders      │ │ Inventory   │ │ Customer Service    │ │  │
│  │  │ Service     │ │ Service     │ │ Service     │ │ (Dialogflow CX)     │ │  │
│  │  │ (GKE)      │ │ (GKE)      │ │ (GKE)      │ │                     │ │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────────┘ │  │
│  │                                                                            │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │  │
│  │  │ Search      │ │ Recommend   │ │ Payment     │ │ User/Auth           │ │  │
│  │  │ Service     │ │ Service     │ │ Service     │ │ Service             │ │  │
│  │  │ (Cloud Run) │ │ (Cloud Run) │ │ (GKE-PCI)  │ │ (Cloud Run)         │ │  │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────────┘ │  │
│  │                                                                            │  │
│  │  All on GKE Autopilot (core) + Cloud Run (event-driven / lightweight)     │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                      AI / ML Layer                                         │  │
│  │                                                                            │  │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐ │  │
│  │  │ Vertex AI        │  │ Vertex AI Search │  │ Vertex AI Agent Builder  │ │  │
│  │  │ (Gemini)         │  │ (Product         │  │ (Dialogflow CX)          │ │  │
│  │  │                  │  │  Discovery)      │  │                          │ │  │
│  │  │ - Catalog enrich │  │ - Semantic search│  │ - Chat agent             │ │  │
│  │  │ - Image gen      │  │ - Faceted search │  │ - Voice agent            │ │  │
│  │  │ - Descriptions   │  │ - Autocomplete   │  │ - Order status           │ │  │
│  │  │ - Translations   │  │ - Recommendations│  │ - Returns/exchanges      │ │  │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────────────┘ │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                      Data Layer                                            │  │
│  │                                                                            │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌────────────┐ │  │
│  │  │ AlloyDB       │  │ Memorystore   │  │ Cloud Storage │  │ Firestore  │ │  │
│  │  │ (Catalog,     │  │ for Redis     │  │ (Product      │  │ (User      │ │  │
│  │  │  Orders,      │  │ (Session,     │  │  Images,      │  │  Sessions, │ │  │
│  │  │  Inventory)   │  │  Cache,       │  │  Static       │  │  Carts)    │ │  │
│  │  │               │  │  Cart)        │  │  Assets)      │  │            │ │  │
│  │  └───────────────┘  └───────────────┘  └───────────────┘  └────────────┘ │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                      Event / Streaming Layer                               │  │
│  │                                                                            │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────┐  │  │
│  │  │ Pub/Sub       │  │ Eventarc      │  │ Dataflow                      │  │  │
│  │  │ (Order events,│  │ (Event-driven │  │ (Real-time inventory updates, │  │  │
│  │  │  Inventory    │  │  triggers)    │  │  price sync, ETL to BigQuery) │  │  │
│  │  │  updates)     │  │               │  │                               │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                      │
│  ┌────────┴───────────────────────────────────────────────────────────────────┐  │
│  │                      Analytics Layer                                       │  │
│  │                                                                            │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌──────────────────────────────┐   │  │
│  │  │ BigQuery      │  │ Looker        │  │ Vertex AI Workbench          │   │  │
│  │  │ (Sales, user  │  │ (Dashboards,  │  │ (ML experimentation,         │   │  │
│  │  │  behavior,    │  │  Reports,     │  │  model training)             │   │  │
│  │  │  inventory)   │  │  Embedded)    │  │                              │   │  │
│  │  └───────────────┘  └───────────────┘  └──────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Service Mapping

| Requirement | GCP Service(s) | Why This Service | Alternative (and why NOT) |
|------------|----------------|------------------|---------------------------|
| Product catalog AI enrichment | **Vertex AI (Gemini)** for text; **Imagen** for images | Gemini generates product descriptions, tags, translations at scale (B1) | Fine-tuning custom models -- unnecessary when Gemini handles it via prompting |
| Conversational AI agent | **Vertex AI Agent Builder** (Dialogflow CX) | Enterprise-grade conversational AI with multi-turn, handoff to human (B2) | Dialogflow ES -- lacks advanced features needed for retail (CX is the successor) |
| Product search & discovery | **Vertex AI Search** (formerly Retail Search) | Semantic search, faceted navigation, autocomplete, recommendations (B3) | Custom Elasticsearch on GKE -- undifferentiated heavy lifting |
| Database modernization | **AlloyDB** (from Oracle) | PostgreSQL-compatible, 4x faster than standard PG, Oracle migration tools (T1) | Cloud Spanner -- overkill unless global scale needed; Cloud SQL -- lower performance |
| Microservices platform | **GKE Autopilot** (core services) + **Cloud Run** (event-driven) | GKE for stateful services; Cloud Run for stateless event handlers (T2) | GKE Standard -- more ops; Cloud Run alone -- can't handle all service types |
| API management | **Apigee** | Enterprise API management, rate limiting, analytics, monetization (T6) | Cloud Endpoints -- lacks enterprise features needed for partner/public APIs |
| Real-time events | **Pub/Sub** + **Eventarc** + **Dataflow** | Pub/Sub for messaging; Eventarc for event routing; Dataflow for stream processing (T5) | Kafka on GKE -- more operational overhead; Cloud Functions alone -- limited |
| Caching | **Memorystore for Redis** | Session management, product catalog caching, cart data (T7) | Self-managed Redis on GKE -- unnecessary ops burden |
| Peak traffic scaling | **GKE autoscaling** (HPA/VPA/CA) + **Cloud CDN** + **Cloud Run** (auto) | CDN offloads static content; autoscaling handles dynamic (B8) | Over-provisioning -- wasteful and expensive |
| Analytics | **BigQuery** + **Looker** | BigQuery for data warehouse; Looker for business intelligence (T9) | Dataproc -- only if existing Spark workloads |
| Payment PCI DSS | Isolated GKE namespace or separate project + **VPC-SC** | PCI scope minimization via network isolation (B7) | Processing payments in shared namespace -- PCI scope explosion |
| Image processing | **Vertex AI (Imagen)** + **Cloud Vision API** | Imagen for generation; Vision API for tagging/classification (T10) | Custom image models -- unnecessary when pre-built APIs exist |
| User sessions & carts | **Firestore** | Serverless, real-time, scales automatically for session data | Cloud SQL -- overkill for session/cart; Memorystore -- not durable enough for carts |
| Content delivery | **Cloud CDN** + **Cloud Storage** | CDN for product images/static assets; Storage as origin (T7) | Third-party CDN -- less integration with GCP; higher latency to origin |

### Monolith Decomposition Strategy

```
Strangler Fig Pattern:

1. Identify bounded contexts in the monolith:
   ├── Catalog management
   ├── Order processing
   ├── Inventory management
   ├── User authentication
   ├── Payment processing
   ├── Search
   └── Customer service

2. Extract services one at a time (lowest risk first):
   ├── Step 1: Search service (independent, read-heavy) -> Vertex AI Search
   ├── Step 2: Customer service (standalone) -> Dialogflow CX
   ├── Step 3: Catalog service (AI enrichment benefit) -> GKE + Vertex AI
   ├── Step 4: Inventory service (real-time needed) -> GKE + Pub/Sub
   ├── Step 5: User/auth service -> Cloud Run + Firebase Auth
   ├── Step 6: Order service (complex transactions) -> GKE + AlloyDB
   └── Step 7: Payment service (PCI isolation) -> GKE (isolated)

3. Apigee sits in front as unified API gateway throughout migration
4. Monolith shrinks as services are extracted
5. Eventually decommission monolith
```

### Common Question Patterns

**Pattern 1: AI Service Selection**
> "Cymbal Retail wants to automatically generate product descriptions. Which service should they use?"
- **Answer direction:** Vertex AI with Gemini models for text generation
- NOT: Custom-trained model on Vertex AI (unnecessary complexity for text generation)
- NOT: Cloud Natural Language API (analysis, not generation)

**Pattern 2: Conversational Commerce**
> "How should Cymbal Retail replace their IVR system?"
- **Answer direction:** Vertex AI Agent Builder (Dialogflow CX) with phone gateway integration, human handoff for complex issues
- NOT: Dialogflow ES (legacy, lacks advanced routing)
- NOT: Custom chatbot on Cloud Run (reinventing the wheel)

**Pattern 3: Database Migration from Oracle**
> "Which database should replace Oracle for the product catalog?"
- **Answer direction:** AlloyDB (PostgreSQL-compatible, Oracle migration tools, high performance)
- NOT: Cloud SQL for PostgreSQL (lower performance for this workload)
- If question emphasizes "global scale": Spanner (but more expensive)

**Pattern 4: Handling Peak Traffic**
> "How should the architecture handle Black Friday traffic spikes?"
- **Answer direction:** Cloud CDN (static), GKE autoscaling (dynamic), Memorystore (caching), Pub/Sub (decouple writes), Cloud Run (overflow)
- NOT: Pre-provisioning static capacity (wasteful)
- NOT: Cloud Functions for everything (cold starts, limits)

**Pattern 5: Search Modernization**
> "How should Cymbal Retail improve product search?"
- **Answer direction:** Vertex AI Search for semantic search, autocomplete, and personalized results
- NOT: Deploying Elasticsearch on GKE (operational burden)
- NOT: BigQuery for search queries (not designed for interactive search)

**Pattern 6: API Unification**
> "How should Cymbal Retail unify their web and mobile backends?"
- **Answer direction:** Apigee as unified API gateway + Backend for Frontend (BFF) pattern on Cloud Run
- NOT: Merging codebases into one monolith
- NOT: Cloud Endpoints (lacks enterprise features)

**Pattern 7: Monolith to Microservices**
> "What migration approach should Cymbal Retail use for their monolithic Java application?"
- **Answer direction:** Strangler fig pattern -- incrementally extract services behind Apigee
- NOT: Big-bang rewrite (too risky for a large retailer)
- NOT: Lift-and-shift of the monolith (doesn't address modernization goals)

**Pattern 8: Real-time Inventory**
> "How should Cymbal Retail move from batch to real-time inventory updates?"
- **Answer direction:** Pub/Sub for event ingestion + Dataflow for stream processing + AlloyDB for inventory state
- NOT: Scheduled Cloud Functions (still batch-like)
- NOT: Direct database writes from POS systems (no decoupling)

### Exam Traps

| Trap | Why It's Wrong | Correct Approach |
|------|---------------|------------------|
| "Use Cloud Natural Language API for product descriptions" | NL API **analyzes** text (sentiment, entities); it does NOT **generate** text | **Vertex AI (Gemini)** for text generation |
| "Use Dialogflow ES for the conversational agent" | ES is the legacy version; CX has advanced routing, multi-turn, and enterprise features | **Dialogflow CX** (part of Vertex AI Agent Builder) |
| "Migrate Oracle to Cloud SQL for PostgreSQL" | Cloud SQL for PG works but lacks the performance for a large retail catalog under peak load | **AlloyDB** (4x faster than standard PostgreSQL, Oracle-compatible migration tools) |
| "Use Cloud Functions for all microservices" | Cloud Functions has cold starts, execution time limits, and isn't suitable for complex stateful services | **GKE Autopilot** for core services; **Cloud Run** for event-driven lightweight services |
| "Build custom search with Elasticsearch on GKE" | Undifferentiated operational work; Vertex AI Search provides retail-optimized search out of the box | **Vertex AI Search** with retail features (browse, search, recommendations) |

---

## Case Study 3: Altostrat Media

**Official doc:** [Altostrat Media Case Study](https://services.google.com/fh/files/blogs/master_case_study_altostrat_media.pdf)

### Company Overview

Altostrat Media is a media and entertainment company that manages a large library of video, audio, and image content distributed globally. They need to modernize their media management platform to leverage AI for content processing, reduce storage costs, and improve global content delivery. Their content library is their primary revenue-generating asset.

**Key facts to memorize:**
- Media and entertainment industry
- Petabytes of media content (video, audio, images)
- NAS/SAN on-premises storage (expensive, limited)
- Manual content tagging (slow, error-prone)
- Global audience (CDN requirements)
- Custom bare-metal transcoding (outdated)
- Revenue depends on content discoverability

### Solution Concept

Build a cloud-native media management platform with AI-powered content processing (automatic tagging, summarization, metadata extraction, content moderation). Migrate petabytes of media content to cloud storage with intelligent tiering to optimize costs. Deploy a global CDN for low-latency content delivery.

### Existing Technical Environment

```
Current State (On-Premises Data Centers):
- Petabytes of media assets on NAS/SAN storage
- Custom media processing pipelines on bare-metal servers
- Manual content tagging and metadata creation (team of 50+ people)
- Third-party CDN contract (expensive, limited flexibility)
- In-house video transcoding farm (FFmpeg on bare metal)
- Audio transcription done manually
- Content moderation: manual review process
- No centralized content search (metadata in spreadsheets)
- Aging hardware with capacity constraints
- 2 data centers: US West Coast, Europe
```

**Environment red flags for exam:**
- Petabytes = massive data transfer challenge (Transfer Appliance)
- NAS/SAN = need cloud storage with tiering
- Manual tagging = AI automation opportunity
- Bare-metal transcoding = replace with Transcoder API
- Third-party CDN = migrate to Cloud CDN
- 2 data centers = multi-region strategy
- Manual moderation = replace with AI content moderation

### Business Requirements

| # | Requirement | Priority | Key Implication |
|---|------------|----------|-----------------|
| B1 | Automate content tagging and metadata extraction | Critical | Video Intelligence API, Vision AI, Speech-to-Text |
| B2 | Reduce media processing time from days to hours | Critical | Parallel processing on GKE, managed transcoding |
| B3 | Content moderation at scale | High | Video Intelligence API (SafeSearch), custom models |
| B4 | Global content delivery with low latency | Critical | Cloud CDN + multi-region Cloud Storage |
| B5 | Reduce storage costs for archival content | High | Cloud Storage lifecycle policies (Nearline/Coldline/Archive) |
| B6 | Monetize content library through better discoverability | High | Search, AI-powered metadata, BigQuery analytics |
| B7 | Reduce reliance on manual processes | High | Event-driven automation, AI pipelines |
| B8 | Maintain content rights and DRM compliance | Medium | Cloud KMS, IAM, watermarking |

### Technical Requirements

| # | Requirement | Implementation Detail |
|---|------------|----------------------|
| T1 | Migrate petabytes of media to cloud storage | Transfer Appliance + Cloud Interconnect |
| T2 | AI-powered video/audio/image analysis | Video Intelligence API, Vision AI, Speech-to-Text |
| T3 | Automated transcoding pipeline | Transcoder API |
| T4 | Hybrid connectivity during migration (large data volumes) | Dedicated Interconnect (HA pair) |
| T5 | Storage tiering for cost optimization | Cloud Storage lifecycle policies |
| T6 | GKE-based media processing platform | GKE Autopilot with GPU node pools |
| T7 | Global CDN for content delivery | Cloud CDN + Media CDN |
| T8 | Content search and discovery | Vertex AI Search or custom search on BigQuery |
| T9 | Event-driven processing pipeline | Pub/Sub + Eventarc + Cloud Functions/Cloud Run |
| T10 | Working file storage for active processing | Filestore (NFS) for media processing workflows |

### Executive Statement

> "Our media library is our most valuable asset, but we can't find or process content fast enough. AI and cloud will unlock the value."

**What this tells you:**
- Content is the core business asset (protect it)
- Discoverability is a revenue problem (not just operational)
- Speed of processing is a competitive differentiator
- AI is seen as the value unlock mechanism
- Cloud is the enabler, not the end goal

### Architecture Diagram

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                              GOOGLE CLOUD                                         │
│                                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────────┐  │
│  │                        Content Delivery Layer                               │  │
│  │                                                                             │  │
│  │   ┌──────────────┐   ┌──────────────┐   ┌────────────────────────────┐     │  │
│  │   │ Media CDN    │   │ Cloud CDN    │   │ Global HTTPS LB            │     │  │
│  │   │ (Video       │   │ (Images,     │   │ (API + Web Console)        │     │  │
│  │   │  Streaming)  │   │  Thumbnails) │   │                            │     │  │
│  │   └──────────────┘   └──────────────┘   └────────────────────────────┘     │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     Data Ingestion / Migration Layer                         │  │
│  │                                                                             │  │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────────────┐     │  │
│  │  │ Transfer         │  │ Cloud Interconnect│  │ Storage Transfer     │     │  │
│  │  │ Appliance        │  │ (HA pair)         │  │ Service              │     │  │
│  │  │ (Initial bulk    │  │ (Ongoing sync)    │  │ (Cloud-to-cloud)     │     │  │
│  │  │  migration)      │  │                   │  │                      │     │  │
│  │  └──────────────────┘  └──────────────────┘  └───────────────────────┘     │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     Storage Layer (Multi-Tier)                               │  │
│  │                                                                             │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌────────────┐  │  │
│  │  │ Cloud Storage │  │ Cloud Storage │  │ Cloud Storage │  │ Cloud      │  │  │
│  │  │ STANDARD      │  │ NEARLINE     │  │ COLDLINE      │  │ Storage    │  │  │
│  │  │ (Active       │  │ (Accessed    │  │ (Accessed     │  │ ARCHIVE    │  │  │
│  │  │  content,     │  │  monthly,    │  │  quarterly,   │  │ (Rarely    │  │  │
│  │  │  multi-region)│  │  older shows)│  │  older films) │  │  accessed) │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────┘  └────────────┘  │  │
│  │                                                                             │  │
│  │  ┌───────────────────┐  ┌───────────────────────────────────────────────┐  │  │
│  │  │ Filestore (NFS)   │  │ Lifecycle Policies:                           │  │  │
│  │  │ (Working files,   │  │  Standard -> Nearline (90 days)              │  │  │
│  │  │  temp processing) │  │  Nearline -> Coldline (180 days)             │  │  │
│  │  └───────────────────┘  │  Coldline -> Archive (365 days)              │  │  │
│  │                          └───────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     Media Processing Layer                                  │  │
│  │                                                                             │  │
│  │  ┌──────────────────────────────────────────────────────────────────────┐   │  │
│  │  │                   GKE Autopilot (with GPU pools)                     │   │  │
│  │  │                                                                      │   │  │
│  │  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │   │  │
│  │  │  │ Ingest     │  │ Transcode  │  │ Quality    │  │ Thumbnail    │  │   │  │
│  │  │  │ Pipeline   │  │ Pipeline   │  │ Check      │  │ Generator    │  │   │  │
│  │  │  └────────────┘  └────────────┘  └────────────┘  └──────────────┘  │   │  │
│  │  │                                                                      │   │  │
│  │  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │   │  │
│  │  │  │ DRM        │  │ Watermark  │  │ Format     │  │ Packaging    │  │   │  │
│  │  │  │ Encryption │  │ Service    │  │ Conversion │  │ (HLS/DASH)   │  │   │  │
│  │  │  └────────────┘  └────────────┘  └────────────┘  └──────────────┘  │   │  │
│  │  └──────────────────────────────────────────────────────────────────────┘   │  │
│  │                                                                             │  │
│  │  ┌──────────────────┐                                                      │  │
│  │  │ Transcoder API   │  (Managed transcoding for standard profiles)         │  │
│  │  └──────────────────┘                                                      │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     AI / Content Intelligence Layer                          │  │
│  │                                                                             │  │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐  │  │
│  │  │ Video            │  │ Vision AI        │  │ Speech-to-Text           │  │  │
│  │  │ Intelligence API │  │                  │  │                          │  │  │
│  │  │                  │  │ - Image labels   │  │ - Transcription          │  │  │
│  │  │ - Shot detection │  │ - Object detect  │  │ - Speaker diarization    │  │  │
│  │  │ - Label detect   │  │ - Face detection │  │ - Language detection     │  │  │
│  │  │ - Object track   │  │ - OCR            │  │ - Timestamps             │  │  │
│  │  │ - Text detection │  │ - SafeSearch     │  │                          │  │  │
│  │  │ - SafeSearch     │  │ - Crop hints     │  │ ┌──────────────────────┐ │  │  │
│  │  │ - Person detect  │  │                  │  │ │ Translation API      │ │  │  │
│  │  │ - Celebrity recog│  └──────────────────┘  │ │ (Subtitles, captions)│ │  │  │
│  │  └──────────────────┘                        │ └──────────────────────┘ │  │  │
│  │                                               └──────────────────────────┘  │  │
│  │  ┌──────────────────┐  ┌──────────────────┐                                │  │
│  │  │ Vertex AI        │  │ Vertex AI        │                                │  │
│  │  │ (Gemini)         │  │ Search           │                                │  │
│  │  │                  │  │                  │                                │  │
│  │  │ - Summarization  │  │ - Content search │                                │  │
│  │  │ - Content desc   │  │ - Similarity     │                                │  │
│  │  │ - Scene analysis │  │ - Filtering      │                                │  │
│  │  └──────────────────┘  └──────────────────┘                                │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     Event-Driven Orchestration Layer                         │  │
│  │                                                                             │  │
│  │  ┌──────────────────────────────────────────────────────────────────────┐   │  │
│  │  │              Pub/Sub + Eventarc + Workflows                          │   │  │
│  │  │                                                                      │   │  │
│  │  │  Upload Event -> Ingest -> Transcode -> AI Analysis -> Publish       │   │  │
│  │  │       │              │          │            │             │          │   │  │
│  │  │       v              v          v            v             v          │   │  │
│  │  │  [Validate]    [Store in    [Multiple   [Tag, Label,  [Update      │   │  │
│  │  │  [Format ]    [GCS Raw ]   [Profiles]  [Moderate,    [Metadata    │   │  │
│  │  │  [Rights ]    [Bucket  ]   [HLS/DASH]  [Transcribe]  [in BQ     ]│   │  │
│  │  │                                                       [Notify   ]│   │  │
│  │  └──────────────────────────────────────────────────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│           │                                                                       │
│  ┌────────┴────────────────────────────────────────────────────────────────────┐  │
│  │                     Analytics & Metadata Layer                              │  │
│  │                                                                             │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌──────────────────────────────┐    │  │
│  │  │ BigQuery      │  │ Looker        │  │ Dataplex                     │    │  │
│  │  │ (Content      │  │ (Content      │  │ (Data governance,            │    │  │
│  │  │  metadata,    │  │  performance, │  │  catalog, quality)           │    │  │
│  │  │  analytics,   │  │  usage        │  │                              │    │  │
│  │  │  viewership)  │  │  dashboards)  │  │                              │    │  │
│  │  └───────────────┘  └───────────────┘  └──────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Service Mapping

| Requirement | GCP Service(s) | Why This Service | Alternative (and why NOT) |
|------------|----------------|------------------|---------------------------|
| Bulk data migration (PBs) | **Transfer Appliance** (initial) + **Dedicated Interconnect** (ongoing) | Transfer Appliance for offline bulk transfer of PBs; Interconnect for ongoing sync (T1) | gsutil/gcloud storage -- too slow for petabytes (would take months over the wire) |
| Video analysis & tagging | **Video Intelligence API** | Shot detection, label detection, object tracking, content moderation, person detection (T2) | Custom video ML models -- unnecessary when pre-built API covers most use cases |
| Image analysis & tagging | **Vision AI** (Cloud Vision API) | Label detection, object detection, OCR, SafeSearch (T2) | Custom vision models -- only needed for domain-specific labels |
| Audio transcription | **Speech-to-Text API** | Automatic transcription with speaker diarization, timestamps (T2) | Manual transcription -- current process, too slow and expensive |
| Video transcoding | **Transcoder API** (standard profiles) + **GKE with FFmpeg** (custom) | Transcoder API for standard outputs; GKE for custom formats (T3) | All transcoding on GKE -- more expensive for standard profiles |
| Storage tiering | **Cloud Storage lifecycle policies** (Standard -> Nearline -> Coldline -> Archive) | Automated cost optimization based on access patterns (T5) | Single storage class -- wastes money on rarely accessed content |
| Working file storage | **Filestore** (NFS) | Media processing pipelines need shared NFS access (T10) | Local SSD -- not shared across nodes; GCS FUSE -- latency issues for active processing |
| Global content delivery | **Media CDN** (streaming) + **Cloud CDN** (images/thumbnails) | Media CDN optimized for large media streaming; Cloud CDN for web assets (T7) | Third-party CDN -- current state, less integrated, more expensive |
| Content summarization | **Vertex AI (Gemini)** | Multimodal model can summarize video content, generate descriptions (B1) | Custom models -- more expensive to train and maintain |
| Content search | **Vertex AI Search** or **BigQuery** full-text search | Vertex AI Search for semantic search; BigQuery for structured metadata queries (T8) | Elasticsearch on GKE -- operational overhead |
| Event-driven pipeline | **Pub/Sub** + **Eventarc** + **Workflows** | Pub/Sub for events; Eventarc for triggers; Workflows for orchestrating multi-step processing (T9) | Cron-based batch -- defeats the purpose of real-time processing |
| Content moderation | **Video Intelligence API** (SafeSearch) + **Vision AI** (SafeSearch) | Pre-built moderation classification for video and images (B3) | Manual moderation -- current process, too slow |
| Subtitles & captions | **Speech-to-Text** + **Translation API** | STT for transcription; Translation for multi-language subtitles (B1) | Manual transcription and translation -- slow, expensive |
| Content analytics | **BigQuery** + **Looker** + **Dataplex** | BigQuery for data warehouse; Looker for dashboards; Dataplex for governance (B6) | Custom analytics -- unnecessary |
| DRM/rights management | **Cloud KMS** + custom DRM service on GKE + **BeyondCorp Enterprise** | KMS for key management; custom service for rights logic (B8) | Third-party DRM -- may be needed alongside, but GCP handles key management |
| Data governance | **Dataplex** | Content catalog, data quality rules, lineage tracking (B6) | Manual spreadsheets -- current state, unscalable |

### Data Transfer Strategy

```
Migration Approach for Petabytes of Media:

Phase 1: Transfer Appliance (Weeks 1-6)
├── Order Transfer Appliance(s) from Google
├── Load data from NAS/SAN to appliance(s) (parallel loading)
├── Ship appliance(s) to Google (logistics ~1-2 weeks)
├── Google ingests data into Cloud Storage
├── Estimated: 100TB per appliance, multiple appliances in parallel
└── Total transfer time: ~4-6 weeks for full library

Phase 2: Dedicated Interconnect (Concurrent with Phase 1)
├── Establish HA Interconnect (2 connections, 2 metros)
├── Sync new/modified content in real-time
├── Handle incremental updates during appliance transit
└── Ongoing connectivity for hybrid operations

Phase 3: Validation & Cutover (Weeks 6-8)
├── Checksum validation of all transferred content
├── Update application pointers to Cloud Storage
├── Switch CDN origin to Cloud Storage
├── Decommission NAS/SAN (or keep for DR)
└── Apply lifecycle policies for cost optimization

Key considerations:
- Transfer Appliance is encrypted (AES-256, customer-managed key)
- Multiple appliances can be used in parallel
- Data is ingested directly into Cloud Storage buckets
- No network bandwidth consumed for bulk transfer
```

### Common Question Patterns

**Pattern 1: Data Migration Strategy**
> "How should Altostrat Media migrate their petabytes of content to Google Cloud?"
- **Answer direction:** Transfer Appliance for bulk migration + Dedicated Interconnect for ongoing sync
- NOT: gsutil transfer over internet (too slow for petabytes)
- NOT: Storage Transfer Service (designed for cloud-to-cloud, not on-prem)

**Pattern 2: Storage Cost Optimization**
> "How should Altostrat Media reduce storage costs for their media library?"
- **Answer direction:** Cloud Storage lifecycle policies: Standard (active) -> Nearline (90 days) -> Coldline (180 days) -> Archive (365 days)
- NOT: Delete old content (it's their primary asset)
- NOT: Use a single storage class (wastes money)
- TRAP: Archive storage retrieval time is NOT slower than other classes -- cost is the differentiator (retrieval is still fast, but minimum storage duration and per-operation costs are higher)

**Pattern 3: AI Content Processing**
> "Which services should Altostrat use for automated content tagging?"
- **Answer direction:** Video Intelligence API (video), Vision AI (images), Speech-to-Text (audio), Vertex AI Gemini (summarization)
- NOT: Training custom models from scratch (pre-built APIs cover most needs)
- Custom models only for domain-specific classification (e.g., genre-specific tagging)

**Pattern 4: Transcoding Architecture**
> "How should Altostrat modernize their video transcoding pipeline?"
- **Answer direction:** Transcoder API for standard profiles (HLS, DASH) + GKE with FFmpeg for custom formats
- NOT: Transcoder API alone (may not support all custom formats)
- NOT: FFmpeg on Compute Engine (no autoscaling, manual management)

**Pattern 5: Global Content Delivery**
> "How should Altostrat deliver content globally with low latency?"
- **Answer direction:** Media CDN for video streaming + Cloud CDN for web assets + multi-region Cloud Storage
- NOT: Cloud CDN alone (Media CDN is optimized for large media files and streaming)
- NOT: Third-party CDN (current state, less integrated)

**Pattern 6: Processing Pipeline Design**
> "Design the content processing pipeline for new media uploads."
- **Answer direction:** GCS upload -> Pub/Sub event -> Workflows orchestration -> parallel AI analysis (Video Intelligence, Vision AI, STT) -> metadata to BigQuery -> CDN-ready output to GCS
- NOT: Synchronous processing (too slow for large media files)
- NOT: Cron-based batch (delays processing, doesn't scale)

**Pattern 7: Working File Storage**
> "What storage should Altostrat use for media files being actively processed?"
- **Answer direction:** Filestore (managed NFS) for shared access during processing; Cloud Storage for final output
- NOT: Cloud Storage alone with FUSE mount (latency issues for active processing)
- NOT: Local SSD (not shared across processing nodes)

**Pattern 8: Content Moderation**
> "How should Altostrat implement content moderation at scale?"
- **Answer direction:** Video Intelligence SafeSearch + Vision AI SafeSearch for automated detection, human review queue for edge cases
- NOT: Only manual review (current state, doesn't scale)
- NOT: Only automated (need human review for edge cases and appeals)

### Exam Traps

| Trap | Why It's Wrong | Correct Approach |
|------|---------------|------------------|
| "Use Storage Transfer Service for on-prem migration" | Storage Transfer Service is designed for cloud-to-cloud or URL-based transfers, not on-prem NAS/SAN | **Transfer Appliance** for bulk on-prem to GCS migration |
| "Archive class has slow retrieval" | Archive storage class retrieval speed is the SAME as other classes; cost structure differs (higher retrieval cost, 365-day minimum) | Archive class has **higher cost per retrieval** and **minimum storage duration**, NOT slower speed |
| "Use Persistent Disk for shared media processing" | Standard PD cannot be mounted read-write on multiple VMs simultaneously (except in multi-writer mode for specific use cases) | **Filestore** (NFS) for shared read-write access across processing nodes |
| "Use Cloud Functions for video transcoding" | Cloud Functions has execution time limits (9 min gen1, 60 min gen2) and memory limits; video transcoding can take hours | **Transcoder API** or **GKE** for long-running transcoding jobs |
| "Use Cloud CDN for all media streaming" | Cloud CDN is optimized for web content; Media CDN is specifically designed for large-scale media streaming | **Media CDN** for video/audio streaming; **Cloud CDN** for images and web assets |

---

## Case Study 4: KnightMotives Automotive

**Official doc:** [KnightMotives Automotive Case Study](https://services.google.com/fh/files/blogs/master_case_study_knightmotives_automotive.pdf)

### Company Overview

KnightMotives Automotive is a global automotive manufacturer developing connected and autonomous vehicles. They are building AI-powered in-vehicle experiences (voice assistants, predictive maintenance, autonomous driving features) and need a cloud platform to process massive amounts of vehicle sensor data. They have manufacturing facilities in the US and Germany, and must comply with GDPR for EU customer data.

**Key facts to memorize:**
- Global automotive manufacturer (US + Germany)
- Connected and autonomous vehicles
- IoT data at massive scale (millions of vehicles x thousands of sensors)
- Legacy mainframe for manufacturing/ERP
- SAP on on-premises servers
- GDPR compliance required (EU data residency)
- AI/ML for autonomous driving
- Edge computing for in-vehicle AI

### Solution Concept

Build a cloud platform for autonomous vehicle data processing, connected car services, dealer tools, and modernize legacy mainframe/ERP systems. The platform must handle the "data tsunami" from connected vehicles while maintaining GDPR compliance for EU operations. Edge computing is needed for latency-sensitive in-vehicle AI.

### Existing Technical Environment

```
Current State (US + Germany Data Centers):
- Legacy mainframe (IBM Z-series) for manufacturing/ERP
- SAP ERP on on-premises servers
- Scattered IoT data from vehicle sensors (not centralized)
- Basic dealer management system (web-based, hosted in US)
- Vehicle telematics data stored in silos
- Custom ML models for autonomous driving (on-prem GPU clusters)
- Data centers:
  - US (Detroit) - primary manufacturing
  - Germany (Stuttgart) - European operations
- Compliance: GDPR (EU), CCPA (California), ISO 27001
- No centralized data lake
- Manual data exchange between departments
```

**Environment red flags for exam:**
- Mainframe = modernization challenge (not lift-and-shift)
- SAP = SAP on Google Cloud (not re-platforming)
- IoT at scale = Pub/Sub + Dataflow + BigQuery
- Scattered data = need a centralized data lake
- GDPR = EU data residency, data sovereignty
- GPU clusters = Vertex AI, GKE with GPUs
- Edge computing = Google Distributed Cloud Edge
- Dealer system in US only = needs regionalization

### Business Requirements

| # | Requirement | Priority | Key Implication |
|---|------------|----------|-----------------|
| B1 | Process autonomous vehicle sensor data at scale | Critical | IoT ingestion, stream processing, ML training |
| B2 | AI-powered in-vehicle experience (voice, predictive maintenance) | Critical | Edge AI, low-latency inference, Google Distributed Cloud |
| B3 | Modernize dealer tools (real-time inventory, customer management) | High | Cloud-native apps, Apigee, regional deployment |
| B4 | Replace mainframe/ERP with cloud-native solution | High | Mainframe modernization, SAP on Google Cloud |
| B5 | Strict GDPR compliance with EU data residency | Critical | Assured Workloads, regional storage, Cloud KMS + EKM |
| B6 | Data monetization (anonymized driving patterns, traffic data) | Medium | BigQuery, Analytics Hub, de-identification |
| B7 | Reduce manufacturing downtime with predictive analytics | High | Vertex AI, IoT data analysis, anomaly detection |
| B8 | Over-the-air (OTA) updates for vehicle software | High | Cloud Storage, CDN, staged rollout |

### Technical Requirements

| # | Requirement | Implementation Detail |
|---|------------|----------------------|
| T1 | IoT data ingestion at massive scale (millions of vehicles) | Cloud IoT Core (deprecated) -> Pub/Sub directly + MQTT broker on GKE |
| T2 | Edge computing for in-vehicle AI | Google Distributed Cloud (Edge) |
| T3 | EU data residency for customer PII | Dual-region EU storage, Assured Workloads, org policies |
| T4 | Mainframe modernization strategy | Dual-write pattern, phased migration to microservices |
| T5 | Real-time analytics for vehicle telemetry | Pub/Sub -> Dataflow -> BigQuery (streaming) |
| T6 | Machine learning for autonomous driving improvements | Vertex AI (training), GKE with GPUs (inference) |
| T7 | Low-latency global API for connected car services | Cloud Run + Global LB + Cloud CDN |
| T8 | SAP migration | SAP on Google Cloud (Bare Metal Solution or CE) |
| T9 | Data lake for centralized analytics | Cloud Storage (data lake) + BigQuery (data warehouse) |
| T10 | Data de-identification for monetization | Cloud DLP API + BigQuery |

### Executive Statement

> "The future of automotive is software-defined vehicles. We need a cloud platform that can handle the data tsunami from connected cars while keeping us compliant in every market we operate."

**What this tells you:**
- Software is the future of the business (not just hardware/manufacturing)
- Data volume is the primary technical challenge
- Compliance is a hard constraint across all markets
- Global platform needed (not just US or EU)
- Executive sees cloud as the platform, not just infrastructure

### Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                              GOOGLE CLOUD (MULTI-REGION)                              │
│                                                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────────┐  │
│  │                    Vehicle / IoT Ingestion Layer                                │  │
│  │                                                                                │  │
│  │  ┌───────────────────┐    ┌───────────────────┐    ┌────────────────────────┐  │  │
│  │  │ MQTT Broker       │    │ Pub/Sub            │    │ Dataflow               │  │  │
│  │  │ (GKE, EMQX)      │───>│ (Vehicle events,   │───>│ (Stream processing,    │  │  │
│  │  │                   │    │  sensor data,       │    │  enrichment,           │  │  │
│  │  │ Millions of       │    │  telemetry)         │    │  anomaly detection)    │  │  │
│  │  │ vehicle           │    │                     │    │                        │  │  │
│  │  │ connections       │    │  Billions of        │    │                        │  │  │
│  │  └───────────────────┘    │  events/day         │    └────────────────────────┘  │  │
│  │                           └───────────────────┘              │                   │  │
│  └──────────────────────────────────────────────────────────────┼───────────────────┘  │
│                                                                  │                     │
│  ┌──────────────────────────────────────────────────────────────┼───────────────────┐  │
│  │                    Data Lake / Analytics Layer                │                   │  │
│  │                                                              v                   │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────────┐   │  │
│  │  │ Cloud Storage │  │ BigQuery      │  │ Vertex AI                         │   │  │
│  │  │ (Data Lake)   │  │ (Telemetry,   │  │                                   │   │  │
│  │  │               │  │  Analytics,   │  │ - AutoML (anomaly detection)      │   │  │
│  │  │ - Raw sensor  │  │  Ad-hoc       │  │ - Custom training (autonomous)    │   │  │
│  │  │   data        │  │  queries)     │  │ - Vertex AI Workbench             │   │  │
│  │  │ - Training    │  │               │  │ - Model Registry                  │   │  │
│  │  │   datasets    │  │ ┌───────────┐ │  │ - Predictions (batch + online)    │   │  │
│  │  │ - Model       │  │ │Analytics  │ │  │                                   │   │  │
│  │  │   artifacts   │  │ │Hub (data  │ │  │ GKE + GPU pools (training)        │   │  │
│  │  │               │  │ │sharing)   │ │  │                                   │   │  │
│  │  └───────────────┘  │ └───────────┘ │  └───────────────────────────────────┘   │  │
│  │                      └───────────────┘                                          │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                    Connected Car Services Layer                                  │  │
│  │                                                                                 │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌─────────────────┐  │  │
│  │  │ Cloud Run     │  │ GKE Autopilot │  │ Apigee        │  │ Firebase        │  │  │
│  │  │ (Vehicle      │  │ (Core         │  │ (Dealer APIs, │  │ (Mobile app     │  │  │
│  │  │  APIs, OTA)   │  │  services)    │  │  Partner APIs) │  │  for drivers)   │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────┘  └─────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                    Enterprise / ERP Layer                                        │  │
│  │                                                                                 │  │
│  │  ┌────────────────────────────┐  ┌──────────────────────────────────────────┐   │  │
│  │  │ SAP on Google Cloud        │  │ Mainframe Modernization                  │   │  │
│  │  │ (Bare Metal Solution       │  │                                          │   │  │
│  │  │  or Compute Engine)        │  │ Phase 1: Dual-write (mainframe + cloud)  │   │  │
│  │  │                            │  │ Phase 2: Read from cloud, write to both  │   │  │
│  │  │ - S/4HANA migration        │  │ Phase 3: Cloud-primary, mainframe read   │   │  │
│  │  │ - BigQuery connector       │  │ Phase 4: Decommission mainframe          │   │  │
│  │  │ - Integration with         │  │                                          │   │  │
│  │  │   cloud services           │  │ Target: Cloud Spanner + microservices    │   │  │
│  │  └────────────────────────────┘  └──────────────────────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                    Security & Compliance Layer                                   │  │
│  │                                                                                 │  │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │  │
│  │  │ Assured        │  │ Cloud KMS +    │  │ VPC Service    │  │ Org Policies │  │  │
│  │  │ Workloads (EU) │  │ Cloud EKM      │  │ Controls       │  │              │  │  │
│  │  │                │  │ (EU key mgmt)  │  │                │  │ - Location   │  │  │
│  │  │ - EU data      │  │                │  │ - Perimeter    │  │   restrict.  │  │  │
│  │  │   residency    │  │ - CMEK for all │  │   for PII data │  │ - EU storage │  │  │
│  │  │ - Compliance   │  │   services     │  │ - Ingress/     │  │   only       │  │  │
│  │  │   guardrails   │  │ - External key │  │   egress rules │  │ - Service    │  │  │
│  │  │                │  │   manager for  │  │                │  │   restrict.  │  │  │
│  │  │                │  │   EU control   │  │                │  │              │  │  │
│  │  └────────────────┘  └────────────────┘  └────────────────┘  └──────────────┘  │  │
│  │                                                                                 │  │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                    │  │
│  │  │ Cloud DLP      │  │ Cloud Audit    │  │ Security       │                    │  │
│  │  │ (De-identify   │  │ Logs + Access  │  │ Command Center │                    │  │
│  │  │  PII for data  │  │ Transparency   │  │ (Threat detect,│                    │  │
│  │  │  monetization) │  │                │  │  compliance)   │                    │  │
│  │  └────────────────┘  └────────────────┘  └────────────────┘                    │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                    Regional Architecture                                        │  │
│  │                                                                                 │  │
│  │  ┌──────────────────────┐         ┌──────────────────────┐                     │  │
│  │  │ US Region (us-central1)│         │ EU Region (europe-west3) │                │  │
│  │  │                      │         │ (Frankfurt)            │                     │  │
│  │  │ - Manufacturing data │         │ - EU customer PII      │                     │  │
│  │  │ - US customer data   │         │ - EU vehicle telemetry │                     │  │
│  │  │ - ML training (GPUs) │         │ - GDPR-compliant       │                     │  │
│  │  │ - Primary SAP        │         │ - Assured Workloads    │                     │  │
│  │  └──────────────────────┘         └──────────────────────────┘                   │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────────────┐
│                              EDGE LAYER (IN-VEHICLE)                                  │
│                                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │  Google Distributed Cloud (Edge)                                                │  │
│  │                                                                                 │  │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                    │  │
│  │  │ Edge AI        │  │ Local Data     │  │ OTA Update     │                    │  │
│  │  │ Inference      │  │ Processing     │  │ Agent           │                    │  │
│  │  │                │  │                │  │                │                    │  │
│  │  │ - Voice asst   │  │ - Sensor       │  │ - Download     │                    │  │
│  │  │ - Predictive   │  │   aggregation  │  │   updates      │                    │  │
│  │  │   maintenance  │  │ - Local        │  │ - Stage rollout│                    │  │
│  │  │ - ADAS assist  │  │   buffering    │  │ - Rollback     │                    │  │
│  │  └────────────────┘  └────────────────┘  └────────────────┘                    │  │
│  │                                                                                 │  │
│  │  Syncs to cloud when connected (cellular/WiFi)                                  │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Service Mapping

| Requirement | GCP Service(s) | Why This Service | Alternative (and why NOT) |
|------------|----------------|------------------|---------------------------|
| IoT data ingestion | **Pub/Sub** + **MQTT broker on GKE** (EMQX/HiveMQ) | Pub/Sub handles billions of events/day; MQTT broker for vehicle protocol (T1) | Cloud IoT Core -- deprecated January 2024; Pub/Sub directly with HTTP -- MQTT is vehicle standard |
| Edge computing | **Google Distributed Cloud (Edge)** | Run GKE and AI inference at the edge (in-vehicle or near-vehicle) (T2) | Custom edge solution -- less integrated with GCP |
| Stream processing | **Dataflow** (Apache Beam) | Real-time stream processing with exactly-once semantics (T5) | Dataproc Spark Streaming -- more operational overhead |
| Data lake | **Cloud Storage** (multi-region) | Scalable, durable, cost-effective for raw sensor data (T9) | HDFS on Dataproc -- unnecessary complexity |
| Data warehouse | **BigQuery** | Serverless analytics, streaming inserts, ML integration (T5) | Redshift equivalent -- BigQuery is the GCP answer |
| ML training | **Vertex AI** + **GKE with GPU pools** | Vertex AI for managed training; GKE GPUs for custom distributed training (T6) | On-prem GPU clusters -- current state, limited scale |
| Autonomous driving ML | **Vertex AI Custom Training** + **Vertex AI Pipelines** | Custom models for computer vision, sensor fusion, decision making (T6) | AutoML alone -- insufficient for autonomous driving complexity |
| In-vehicle AI | **Google Distributed Cloud (Edge)** + optimized models | Low-latency inference at the edge, works offline (B2) | Cloud-only inference -- latency too high for safety-critical systems |
| Connected car APIs | **Cloud Run** (global) + **Global HTTPS LB** | Serverless, scales to millions of vehicles, low latency (T7) | GKE alone -- Cloud Run better for API-style workloads with variable traffic |
| Dealer tools | **GKE Autopilot** + **Apigee** + **Cloud SQL** | Microservices for dealer apps; Apigee for API management (B3) | Monolithic app -- doesn't scale for global dealer network |
| SAP migration | **SAP on Google Cloud** (Bare Metal Solution or Compute Engine) | Certified SAP infrastructure, BigQuery connector (T8) | Re-platforming SAP -- too risky, not supported |
| Mainframe modernization | **Dual-write pattern** -> **Cloud Spanner** + microservices | Phased migration: dual-write, then gradually shift reads, then decommission (T4) | Lift-and-shift mainframe -- not possible; Big-bang rewrite -- too risky |
| GDPR compliance | **Assured Workloads (EU)** + **org policies** + **Cloud KMS + EKM** | Assured Workloads for EU guardrails; EKM for EU-controlled encryption keys (B5, T3) | US-only deployment -- violates GDPR data residency |
| EU data residency | **Dual-region EU storage** + **region-restricted org policies** | Enforced at org level; data never leaves EU (T3) | Multi-region (global) -- data could leave EU |
| Data monetization | **BigQuery** + **Analytics Hub** + **Cloud DLP** | DLP de-identifies PII; Analytics Hub for data sharing/monetization (B6) | Raw data sharing -- GDPR violation |
| Predictive maintenance | **Vertex AI** (AutoML or custom) + **Dataflow** (feature engineering) | ML models trained on telemetry data predict component failures (B7) | Rule-based alerts -- less accurate than ML |
| OTA updates | **Cloud Storage** + **Cloud CDN** + staged rollout on GKE | CDN for global distribution; staged rollout for safety (B8) | Direct push from cloud -- needs staged rollout for vehicle safety |
| Mobile app (drivers) | **Firebase** + **Cloud Run** (backend) | Firebase for mobile SDKs, auth, messaging; Cloud Run for APIs | Custom mobile backend -- unnecessary when Firebase exists |

### Mainframe Modernization Strategy

```
Phased Approach (Dual-Write Pattern):

Phase 1: Assess & Plan (Months 1-3)
├── Inventory all mainframe workloads
├── Categorize: retain, retire, rehost, refactor, replace
├── Map data dependencies and integration points
├── Choose target architecture (Spanner + microservices)
└── Design dual-write mechanism

Phase 2: Build Cloud Foundation (Months 3-6)
├── Deploy Cloud Spanner (multi-region for HA)
├── Build microservices on GKE for business logic
├── Implement dual-write: mainframe writes -> cloud writes (sync)
├── Validate data consistency between mainframe and cloud
└── Cloud reads still go to mainframe

Phase 3: Shift Reads (Months 6-9)
├── Redirect read traffic to cloud services
├── Mainframe becomes write-primary only
├── Monitor for data consistency
├── Dealers and analytics read from cloud
└── Performance testing at scale

Phase 4: Cloud Primary (Months 9-12)
├── Cloud becomes primary for writes
├── Mainframe receives sync copies (backup)
├── Validate all business processes on cloud
└── UAT with all stakeholders

Phase 5: Decommission (Months 12-15)
├── Stop mainframe sync
├── Archive mainframe data
├── Decommission mainframe infrastructure
└── Full cloud operations

Key considerations:
- SAP integration must be maintained throughout
- Some batch processing may need temporary refactoring
- COBOL business logic must be re-implemented in modern languages
- Consider using tools like Google Cloud Mainframe Assessment Tool
```

### Common Question Patterns

**Pattern 1: IoT Data Ingestion**
> "How should KnightMotives handle data from millions of connected vehicles?"
- **Answer direction:** MQTT broker on GKE -> Pub/Sub -> Dataflow -> BigQuery (streaming pipeline)
- NOT: Cloud IoT Core (deprecated January 2024)
- NOT: Direct writes to BigQuery (need stream processing for enrichment/filtering)

**Pattern 2: GDPR Compliance**
> "How should KnightMotives ensure EU customer data stays in the EU?"
- **Answer direction:** Assured Workloads (EU), region-restricted org policies, dual-region EU storage, Cloud KMS with EKM (external key manager in EU)
- NOT: Google-managed keys only (EU regulators may require customer key control)
- NOT: Multi-region (global) storage (data could leave EU)

**Pattern 3: Edge Computing**
> "What solution should KnightMotives use for in-vehicle AI inference?"
- **Answer direction:** Google Distributed Cloud (Edge) for running containers and ML inference at the edge
- NOT: Cloud-only inference (latency too high for safety-critical driving)
- NOT: Custom edge hardware (less integrated, harder to manage)

**Pattern 4: Mainframe Modernization**
> "What approach should KnightMotives use to replace their mainframe?"
- **Answer direction:** Phased dual-write pattern (mainframe + cloud in parallel, gradually shift traffic)
- NOT: Big-bang rewrite (too risky for manufacturing ERP)
- NOT: Lift-and-shift (mainframe workloads can't be lifted to cloud)

**Pattern 5: SAP on Google Cloud**
> "How should KnightMotives migrate their SAP environment?"
- **Answer direction:** SAP on Google Cloud using Bare Metal Solution (for HANA) or Compute Engine (for S/4HANA), with BigQuery connector for analytics
- NOT: Re-platform SAP to cloud-native (not practical)
- NOT: Replace SAP entirely (too complex for manufacturing)

**Pattern 6: Data Monetization**
> "How should KnightMotives monetize anonymized driving data?"
- **Answer direction:** Cloud DLP for de-identification, BigQuery for analytics, Analytics Hub for data sharing with partners
- NOT: Share raw data (GDPR violation)
- NOT: Custom data marketplace (Analytics Hub exists for this)

**Pattern 7: Vehicle OTA Updates**
> "Design the OTA update architecture for KnightMotives' vehicles."
- **Answer direction:** Cloud Storage for update packages, Cloud CDN for global distribution, staged rollout (canary -> incremental), rollback capability
- NOT: Direct push without staged rollout (vehicle safety risk)
- NOT: P2P update distribution (security and reliability concerns)

**Pattern 8: Predictive Maintenance**
> "How should KnightMotives implement predictive maintenance for vehicles?"
- **Answer direction:** Stream telemetry via Pub/Sub -> Dataflow for feature engineering -> Vertex AI for ML training -> edge model deployment for inference
- NOT: Rule-based thresholds only (misses complex failure patterns)
- NOT: Cloud-only inference (too slow for real-time alerts)

### Exam Traps

| Trap | Why It's Wrong | Correct Approach |
|------|---------------|------------------|
| "Use Cloud IoT Core for vehicle connectivity" | Cloud IoT Core was **deprecated January 2024** and is no longer available | **MQTT broker on GKE** (EMQX, HiveMQ) + **Pub/Sub** for event streaming |
| "Use multi-region storage for EU data" | Multi-region includes locations outside the EU; violates GDPR data residency | **Dual-region EU** (e.g., `eur4`: Netherlands + Finland) or single EU region |
| "Run ML training at the edge" | Edge devices lack GPU capacity for training large models | Train in cloud (**Vertex AI**), deploy optimized models to edge for **inference only** |
| "Lift-and-shift mainframe to Compute Engine" | Mainframe workloads (COBOL, JCL, VSAM) cannot run on x86 Compute Engine | **Phased modernization** with dual-write pattern to cloud-native services |
| "Use Cloud Functions for stream processing" | Cloud Functions has execution limits and no native stream processing capabilities | **Dataflow** (Apache Beam) for proper stream processing with windowing, late data handling |

---

## Cross-Case Study Exam Strategy

### Quick Reference: Which Case Tests What

| Theme | EHR Healthcare | Cymbal Retail | Altostrat Media | KnightMotives |
|-------|---------------|---------------|-----------------|---------------|
| **Migration type** | Colocation to cloud | Monolith to microservices | Petabyte data migration | Mainframe modernization |
| **Compliance** | HIPAA | PCI DSS | Copyright/DRM | GDPR |
| **AI/ML focus** | Healthcare API (FHIR) | Gen AI / Agents / Search | Content Intelligence | Autonomous driving / Edge AI |
| **Hybrid/Edge** | Dedicated Interconnect | -- | Transfer Appliance | Google Distributed Cloud (Edge) |
| **Primary database** | Cloud SQL (MySQL + SQL Server) | AlloyDB (from Oracle) | BigQuery + Cloud Storage | Spanner + BigQuery |
| **Containers** | GKE Autopilot | GKE Autopilot + Cloud Run | GKE (with GPUs) | GKE + Cloud Run |
| **Identity** | AD integration (GCDS + SAML) | Standard Cloud Identity | Standard Cloud Identity | Workforce Identity Federation |
| **Key services** | Cloud Healthcare API, Assured Workloads | Vertex AI (Gemini), Apigee | Transcoder API, Media CDN | Pub/Sub, Dataflow, Distributed Cloud |
| **Data volume** | Medium (transactional) | Medium (catalog + transactions) | Very high (petabytes media) | Very high (IoT streaming) |
| **DR requirement** | RPO<1h, RTO<4h | HA for peak traffic | Multi-region for availability | Multi-region for compliance |

### Service Frequency Across Case Studies

| Service | EHR | Cymbal | Altostrat | KnightMotives | Total |
|---------|-----|--------|-----------|---------------|-------|
| GKE Autopilot | X | X | X | X | 4/4 |
| Cloud Storage | X | X | X | X | 4/4 |
| BigQuery | X | X | X | X | 4/4 |
| Pub/Sub | - | X | X | X | 3/4 |
| Cloud KMS (CMEK) | X | X | - | X | 3/4 |
| VPC Service Controls | X | X | - | X | 3/4 |
| Cloud Run | - | X | - | X | 2/4 |
| Vertex AI | - | X | X | X | 3/4 |
| Apigee | X | X | - | X | 3/4 |
| Dataflow | - | - | - | X | 1/4 |
| Cloud SQL | X | - | - | - | 1/4 |
| AlloyDB | - | X | - | - | 1/4 |
| Cloud Spanner | - | - | - | X | 1/4 |
| Dedicated Interconnect | X | - | X | - | 2/4 |
| Assured Workloads | X | - | - | X | 2/4 |

**Insight:** GKE, Cloud Storage, and BigQuery appear in ALL four case studies. If you're unsure, these are safe choices. Pub/Sub and CMEK appear in 3 of 4.

### Compliance Comparison

| Aspect | HIPAA (EHR) | PCI DSS (Cymbal) | DRM (Altostrat) | GDPR (KnightMotives) |
|--------|-------------|-------------------|-----------------|----------------------|
| **Scope** | Patient health data (PHI) | Cardholder data (CHD) | Content rights | Personal data (EU residents) |
| **GCP tool** | Assured Workloads | VPC-SC + PCI project isolation | Cloud KMS + DRM service | Assured Workloads (EU) |
| **Encryption** | CMEK required | CMEK for CHD | CMEK + watermarking | CMEK + EKM (EU keys) |
| **Data residency** | US (typically) | Depends on region | Global (CDN) | EU (strict) |
| **Audit** | Data Access Logs + Access Transparency | PCI audit trail | Content access logs | Data Access Logs + DPIA |
| **Key service** | Cloud Healthcare API | Tokenization | Media CDN | Cloud DLP |
| **BAA needed?** | Yes (Google signs BAA) | SAQ/ROC from QSA | No | DPA (Data Processing Agreement) |

### Database Selection Summary

```
Decision tree for case study database questions:

1. Is it a healthcare transactional database?
   ├── Yes + MySQL? -> Cloud SQL for MySQL (EHR Healthcare)
   └── Yes + SQL Server? -> Cloud SQL for SQL Server (EHR Healthcare)

2. Is it migrating from Oracle?
   └── Yes -> AlloyDB (PostgreSQL-compatible, Oracle migration tools) (Cymbal Retail)

3. Does it need global scale + strong consistency?
   └── Yes -> Cloud Spanner (KnightMotives mainframe replacement)

4. Is it a data warehouse / analytics workload?
   └── Yes -> BigQuery (all case studies)

5. Is it media metadata + content analytics?
   └── Yes -> BigQuery (Altostrat Media)

6. Is it session/cart data (temporary, high-throughput)?
   └── Yes -> Firestore or Memorystore (Cymbal Retail)
```

### Migration Strategy Comparison

| Aspect | EHR Healthcare | Cymbal Retail | Altostrat Media | KnightMotives |
|--------|---------------|---------------|-----------------|---------------|
| **Pattern** | Lift-and-modernize | Strangler fig (monolith decomposition) | Data-first migration | Phased dual-write |
| **Data transfer** | Dedicated Interconnect (continuous) | Database Migration Service | Transfer Appliance (bulk) | Interconnect + streaming |
| **Duration** | 6-9 months | 12-18 months | 4-6 months (data), 12 months (apps) | 12-18 months |
| **Risk** | Medium (downtime sensitivity) | High (monolith complexity) | Low (data is independent) | Very high (mainframe complexity) |
| **Rollback** | Keep colocation as fallback | Keep monolith running | Keep NAS/SAN copy | Keep mainframe running |

---

## Universal Analysis Framework

Use this step-by-step approach for ANY case study question on the exam.

### Step 1: Identify the Case Study

Read the question carefully. It will reference one of the four case studies by company name. Recall the key themes:

```
EHR Healthcare    -> Healthcare, HIPAA, colocation, containers, AD integration
Cymbal Retail     -> Retail, Gen AI, Oracle migration, monolith, peak traffic
Altostrat Media   -> Media, petabytes, content AI, transcoding, CDN
KnightMotives     -> Automotive, IoT, mainframe, GDPR, edge computing
```

### Step 2: Map the Question to Requirements

Every question maps to one or more requirements from the case study. Identify which requirement is being tested:

```
"Which service should [company] use for..." -> Technical requirement (service selection)
"How should [company] approach..."          -> Architecture pattern / migration strategy
"What should [company] do to ensure..."     -> Compliance / security requirement
"Which is the BEST approach for..."         -> Trade-off evaluation (cost vs. performance vs. ops)
```

### Step 3: Apply Elimination Rules

For each answer choice, ask:

| Question | If Yes | Action |
|----------|--------|--------|
| Does this service exist? (Is it deprecated?) | Cloud IoT Core = deprecated | Eliminate |
| Does this violate a compliance requirement? | GDPR + global storage = violation | Eliminate |
| Does this contradict a stated requirement? | "Reduce ops" + self-managed DB = contradiction | Eliminate |
| Is this the right service category? | NL API for generation = wrong category | Eliminate |
| Is this overkill or underkill? | Spanner for a simple MySQL workload = overkill | Likely eliminate |

### Step 4: Choose the Best Answer

After eliminating wrong answers, choose based on these priorities:

```
1. Compliance first (never violate stated compliance requirements)
2. Business requirements (match the stated business goals)
3. Managed over self-managed (unless a specific requirement demands control)
4. Cost-effective (don't choose the most expensive option without justification)
5. Simplest solution that meets all requirements (Occam's razor)
```

### Step 5: Verify Against Known Traps

Before finalizing, check if your answer falls into a known trap:

```
Common trap patterns:
- Deprecated service (Cloud IoT Core, Dialogflow ES for new projects)
- Wrong compliance scope (global storage for GDPR, default keys for HIPAA)
- Overengineered (Spanner when Cloud SQL suffices)
- Underengineered (Cloud SQL when Spanner's global consistency is needed)
- Wrong AI service (NL API for generation, Vision API for video)
- Ignoring stated requirements ("reduce ops" but choosing self-managed)
```

### Step 6: Quick Decision Matrix

When stuck between two answers, use this matrix:

| Factor | Weight | Option A Score | Option B Score |
|--------|--------|---------------|---------------|
| Meets all requirements? | 5x | 0-5 | 0-5 |
| Compliance-safe? | 5x | 0-5 | 0-5 |
| Reduces operational burden? | 3x | 0-5 | 0-5 |
| Cost-appropriate? | 2x | 0-5 | 0-5 |
| GCP-native / managed? | 2x | 0-5 | 0-5 |

The option with the higher weighted score is likely correct.

---

## Quick Recall Cheat Sheet

Use this for last-minute review before the exam.

### One-Line Summaries

```
EHR Healthcare:
  "HIPAA-compliant cloud migration of containerized + Windows healthcare workloads
   from colocation, with AD integration and Cloud Healthcare API."

Cymbal Retail:
  "AI-first retail modernization: Gemini for catalog, Dialogflow CX for customer
   service, AlloyDB from Oracle, strangler fig for the monolith."

Altostrat Media:
  "Petabyte media migration with Transfer Appliance, AI content processing
   (Video Intelligence, Vision, STT), storage tiering, and Media CDN."

KnightMotives Automotive:
  "IoT-scale connected car platform with edge AI, mainframe-to-Spanner
   modernization, EU data residency (GDPR), and Vertex AI for autonomous driving."
```

### Key Service per Case Study (Differentiators)

```
EHR Healthcare     -> Cloud Healthcare API (FHIR/HL7)
Cymbal Retail      -> Vertex AI Agent Builder (Dialogflow CX)
Altostrat Media    -> Transcoder API + Media CDN
KnightMotives      -> Google Distributed Cloud (Edge)
```

### Compliance Keywords

```
HIPAA    -> BAA, PHI, Assured Workloads, CMEK, VPC-SC, Data Access Logs
PCI DSS  -> CHD, tokenization, scope minimization, isolated project, VPC-SC
DRM      -> Content rights, Cloud KMS, watermarking, access controls
GDPR     -> PII, data residency, Assured Workloads (EU), EKM, DLP, right to delete
```

### Migration Pattern Keywords

```
Colocation migration  -> Dedicated Interconnect, phased migration, hybrid
Monolith to micro     -> Strangler fig, API gateway, bounded contexts
Petabyte migration    -> Transfer Appliance, bulk transfer, validation
Mainframe modern.     -> Dual-write, phased cutover, Spanner replacement
```

### The "Golden Rule" for Case Study Questions

> When in doubt, choose the answer that:
> 1. Uses **managed services** (reduces operational burden)
> 2. Meets **compliance requirements** (never compromise)
> 3. Aligns with the **executive statement** (understand the business driver)
> 4. Uses **GCP-native services** over third-party equivalents
> 5. Matches the **scale** of the problem (don't over or under-engineer)

---

## Appendix: Official Documentation Links

### General
- [PCA Certification Overview](https://cloud.google.com/learn/certification/cloud-architect)
- [PCA Exam Guide](https://cloud.google.com/learn/certification/guides/professional-cloud-architect)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)

### Services Referenced in Case Studies
- [GKE Autopilot](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [Cloud SQL](https://cloud.google.com/sql/docs)
- [AlloyDB](https://cloud.google.com/alloydb/docs)
- [Cloud Spanner](https://cloud.google.com/spanner/docs)
- [BigQuery](https://cloud.google.com/bigquery/docs)
- [Vertex AI](https://cloud.google.com/vertex-ai/docs)
- [Vertex AI Agent Builder](https://cloud.google.com/dialogflow/cx/docs)
- [Vertex AI Search](https://cloud.google.com/generative-ai-app-builder/docs)
- [Cloud Healthcare API](https://cloud.google.com/healthcare-api/docs)
- [Pub/Sub](https://cloud.google.com/pubsub/docs)
- [Dataflow](https://cloud.google.com/dataflow/docs)
- [Cloud Storage](https://cloud.google.com/storage/docs)
- [Transfer Appliance](https://cloud.google.com/transfer-appliance/docs)
- [Dedicated Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect)
- [Cloud CDN](https://cloud.google.com/cdn/docs)
- [Media CDN](https://cloud.google.com/media-cdn/docs)
- [Transcoder API](https://cloud.google.com/transcoder/docs)
- [Cloud Run](https://cloud.google.com/run/docs)
- [Apigee](https://cloud.google.com/apigee/docs)
- [Cloud KMS](https://cloud.google.com/kms/docs)
- [Cloud EKM](https://cloud.google.com/kms/docs/ekm)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
- [Assured Workloads](https://cloud.google.com/assured-workloads/docs)
- [Cloud DLP](https://cloud.google.com/sensitive-data-protection/docs)
- [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)
- [Google Distributed Cloud (Edge)](https://cloud.google.com/distributed-cloud/edge/latest/docs)
- [SAP on Google Cloud](https://cloud.google.com/solutions/sap)
- [Database Migration Service](https://cloud.google.com/database-migration/docs)
- [Firestore](https://cloud.google.com/firestore/docs)
- [Memorystore](https://cloud.google.com/memorystore/docs)
- [Video Intelligence API](https://cloud.google.com/video-intelligence/docs)
- [Vision AI](https://cloud.google.com/vision/docs)
- [Speech-to-Text](https://cloud.google.com/speech-to-text/docs)
- [Cloud Armor](https://cloud.google.com/armor/docs)
- [Identity-Aware Proxy](https://cloud.google.com/iap/docs)
- [Cloud Identity / GCDS](https://cloud.google.com/identity/docs)
- [Workflows](https://cloud.google.com/workflows/docs)
- [Analytics Hub](https://cloud.google.com/analytics-hub/docs)
- [Dataplex](https://cloud.google.com/dataplex/docs)
- [Security Command Center](https://cloud.google.com/security-command-center/docs)
- [Firebase](https://firebase.google.com/docs)
