# Well-Architected Framework Reference

> **Cross-cutting reference** -- The Well-Architected Framework (WAF) is central to the PCA exam. Questions across ALL six exam sections may reference WAF pillars. Understanding the pillars helps you evaluate architecture trade-offs, which is the core skill the PCA exam tests.

---

## Overview

### What Is the Google Cloud Well-Architected Framework?

The Google Cloud Well-Architected Framework is a set of **prescriptive guidance, best practices, and design principles** that help architects build secure, high-performing, resilient, cost-effective, and sustainable workloads on Google Cloud. It provides a consistent approach for evaluating architectures and implementing designs that scale.

The framework is structured around **six pillars**, each representing a dimension of architecture quality:

| # | Pillar | Core Question It Answers |
|---|--------|--------------------------|
| 1 | Operational Excellence | "How do we run and monitor systems to deliver business value and continuously improve?" |
| 2 | Security, Privacy, and Compliance | "How do we protect information, systems, and assets while delivering business value?" |
| 3 | Reliability | "How do we ensure a workload performs its intended function correctly and consistently?" |
| 4 | Cost Optimization | "How do we achieve business outcomes at the lowest price point?" |
| 5 | Performance Optimization | "How do we allocate resources efficiently to meet system requirements as demand changes?" |
| 6 | Sustainability | "How do we minimize the environmental impact of running cloud workloads?" |

### How WAF Differs from the AWS Well-Architected Framework

| Aspect | GCP WAF | AWS WAF |
|--------|---------|---------|
| Number of pillars | **6** (includes Sustainability as a full pillar) | 6 (Sustainability added later as 6th) |
| Security naming | **Security, Privacy, and Compliance** | Security |
| Sustainability | First-class pillar with Carbon Footprint tooling | Added in late 2021 as a separate pillar |
| Review tool | Architecture Framework review in console | AWS Well-Architected Tool |
| Lens concept | Architecture Center blueprints + industry guides | Well-Architected Lenses |
| SRE influence | Heavy -- SLOs, error budgets, toil reduction baked in | Less explicit SRE integration |
| AI/ML guidance | Integrated into pillars (Vertex AI, Gemini, Model Armor) | Separate ML Lens |

### WAF's Role in the PCA Exam

The PCA exam tests your ability to **evaluate architecture trade-offs**. Almost every question implicitly asks: "Which option best balances the relevant WAF pillars for this scenario?" You will rarely see a question that names a pillar directly, but the underlying reasoning maps to WAF thinking:

- **Section 1 (Designing/Planning):** Reliability + Cost Optimization + Performance
- **Section 2 (Managing/Provisioning):** Operational Excellence + Security
- **Section 3 (Security/Compliance):** Security, Privacy, and Compliance (directly)
- **Section 4 (Optimizing):** Cost Optimization + Performance + Operational Excellence
- **Section 5 (Managing Implementation):** Operational Excellence + Reliability
- **Section 6 (Operations):** Operational Excellence + Reliability (directly)

**Docs:** [Well-Architected Framework](https://cloud.google.com/architecture/framework)

---

## Pillar 1: Operational Excellence

### Key Principles

Operational Excellence focuses on running, monitoring, and continuously improving systems. Google Cloud's approach to this pillar is heavily influenced by **Site Reliability Engineering (SRE)** practices.

**Core principles:**

1. **Define clear SLOs, SLIs, and SLAs** -- Measure what matters to users, not just infrastructure metrics
2. **Automate everything possible** -- Reduce toil (repetitive, manual, automatable work) to free engineering time for innovation
3. **Monitor proactively, not reactively** -- Use observability (metrics, logs, traces) to detect issues before users notice
4. **Practice incident management** -- Establish clear on-call rotations, runbooks, escalation paths, and blameless post-mortems
5. **Embrace change management** -- Use CI/CD pipelines, IaC, and progressive rollouts to deploy safely
6. **Manage capacity proactively** -- Use autoscaling and quota management to handle demand fluctuations
7. **Build for operability from day one** -- Design systems that are easy to deploy, debug, and maintain

### SLOs, SLIs, and SLAs Deep Dive

This is a critical PCA exam topic. Understand the hierarchy:

| Concept | Definition | Example | Who Sets It |
|---------|------------|---------|-------------|
| **SLI** (Service Level Indicator) | A quantitative measure of service behavior | Request latency, error rate, throughput | Engineering team |
| **SLO** (Service Level Objective) | A target value or range for an SLI | 99.9% of requests < 300ms over 30 days | Engineering + Product |
| **SLA** (Service Level Agreement) | A business contract with consequences | 99.95% uptime or customer gets credits | Business + Legal |
| **Error Budget** | The allowed amount of unreliability (1 - SLO) | 0.1% = ~43 min downtime/month for 99.9% | Derived from SLO |

**Key relationships:**
- SLA <= SLO (SLA should always be less strict than SLO to provide a buffer)
- Error budget = 1 - SLO target
- When error budget is exhausted, freeze feature deployments and focus on reliability

```
# Example SLI/SLO definition in monitoring
Availability SLI = count of successful requests / count of total requests
Availability SLO = 99.9% over a rolling 30-day window
Error Budget   = 0.1% = ~43.2 minutes of downtime per 30 days
```

### GCP Services That Map to This Pillar

| Service | Role in Operational Excellence | Key Features |
|---------|-------------------------------|--------------|
| **Cloud Monitoring** | Metrics collection, dashboards, alerting | Uptime checks, custom metrics, MQL, SLO monitoring |
| **Cloud Logging** | Centralized log management | Log Router, Log Analytics, log-based metrics, audit logs |
| **Cloud Trace** | Distributed tracing for latency analysis | Auto-instrumentation, trace correlation with logs |
| **Cloud Profiler** | Continuous CPU/memory profiling in production | Low overhead (~0.5%), flame graphs |
| **Error Reporting** | Automatic error detection and grouping | Stack trace analysis, notification integration |
| **Cloud Deploy** | Managed continuous delivery to GKE/Cloud Run | Delivery pipelines, approval gates, rollbacks |
| **Cloud Build** | CI/CD build automation | Build triggers, custom builders, SLSA compliance |
| **Artifact Registry** | Artifact/container image management | Vulnerability scanning, cleanup policies, remote repos |
| **Service Mesh (Istio/Anthos)** | Traffic management, observability | Canary deployments, circuit breaking, mTLS |
| **Cloud Scheduler** | Managed cron job service | HTTP/Pub/Sub/App Engine targets |
| **Cloud Tasks** | Asynchronous task execution | Rate limiting, retry policies, task deduplication |
| **Recommender** | AI-powered operational recommendations | Idle resource detection, rightsizing, security insights |

### Architect-Level Considerations

#### Observability Strategy

An architect must design a **layered observability strategy**:

```
Layer 1: Infrastructure Metrics (CPU, memory, disk, network)
   ├── Cloud Monitoring agent metrics
   └── GKE node/pod metrics via Managed Prometheus

Layer 2: Application Metrics (request rate, error rate, latency -- the RED method)
   ├── Custom metrics via OpenTelemetry
   └── Cloud Trace for distributed tracing

Layer 3: Business Metrics (orders/sec, revenue, user signups)
   ├── Custom metrics pushed to Cloud Monitoring
   └── BigQuery + Looker dashboards

Layer 4: Security/Audit (who did what, when)
   ├── Admin Activity audit logs (always on)
   ├── Data Access audit logs (must enable)
   └── VPC Flow Logs
```

**Key commands:**

```bash
# Create an alerting policy for high error rate
gcloud monitoring policies create --policy-from-file=error-rate-policy.yaml

# Create a notification channel
gcloud monitoring channels create \
  --type=email \
  --display-name="On-Call Team" \
  --channel-labels=email_address=oncall@company.com

# View SLO compliance
gcloud monitoring slos list --service=my-service

# Create a log-based metric
gcloud logging metrics create high_latency_requests \
  --description="Requests taking more than 5 seconds" \
  --log-filter='resource.type="http_load_balancer" AND httpRequest.latency>"5s"'

# Create a log sink to BigQuery for analysis
gcloud logging sinks create audit-sink \
  bigquery.googleapis.com/projects/my-project/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com"'

# Enable Data Access audit logs for a service
gcloud projects get-iam-policy my-project --format=json > policy.json
# Edit the auditConfigs section, then:
gcloud projects set-iam-policy my-project policy.json
```

#### Deployment Strategies

| Strategy | Risk Level | Rollback Speed | Use Case |
|----------|-----------|----------------|----------|
| **Rolling update** | Low-Medium | Moderate | Standard GKE deployments |
| **Blue/Green** | Low | Instant (switch traffic) | Stateless services, zero-downtime required |
| **Canary** | Very Low | Instant (route to stable) | High-risk changes, new features |
| **A/B Testing** | Very Low | Instant | Feature validation with user segmentation |
| **Recreate** | High | Slow | Stateful apps that can't run mixed versions |

```bash
# Cloud Deploy delivery pipeline example
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-pipeline \
  --region=us-central1 \
  --images=my-app=gcr.io/my-project/my-app:v1.2.3

# Promote a release to the next stage
gcloud deploy releases promote \
  --release=release-001 \
  --delivery-pipeline=my-pipeline \
  --region=us-central1

# Rollback a release
gcloud deploy targets rollback my-target \
  --delivery-pipeline=my-pipeline \
  --region=us-central1
```

#### Toil Reduction Framework

Toil is work that is manual, repetitive, automatable, tactical, without enduring value, and scales linearly with service growth. The goal is to keep toil below **50% of an SRE team's time**.

| Toil Example | Automation Solution |
|-------------|-------------------|
| Manual VM provisioning | Terraform + Cloud Build triggers |
| Certificate renewal | Certificate Manager (managed SSL) |
| Log review for errors | Error Reporting + alerting policies |
| Capacity scaling | Autoscaler (MIG, GKE HPA/VPA) |
| Database backups | Automated backups (Cloud SQL, Spanner) |
| Security patching | OS Patch Management, GKE auto-upgrade |
| DNS updates | Cloud DNS + external-dns in GKE |

#### Runbooks and Post-Mortems

- **Runbooks**: Document step-by-step procedures for common operational tasks and incident responses. Store in a wiki or version-controlled repo.
- **Post-mortems**: Blameless analysis of incidents. Focus on systemic causes, not individual errors. Should result in action items that prevent recurrence.

Post-mortem template key sections:
1. Incident summary and impact
2. Timeline of events
3. Root cause analysis (use "5 Whys")
4. Action items with owners and deadlines
5. Lessons learned

### Exam Tips

- **SLO vs SLA**: If a question asks about setting reliability targets that the engineering team uses internally, it is an **SLO**. If it involves a customer contract with financial penalties, it is an **SLA**. SLOs should always be stricter than SLAs.
- **Error budget exhaustion**: When error budget is depleted, the correct action is to **freeze feature releases and focus on reliability** -- not to lower the SLO.
- **Cloud Operations Suite**: Know that Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler, and Error Reporting collectively form the **Cloud Operations Suite** (formerly Stackdriver).
- **Log Router**: Questions about sending logs to different destinations (BigQuery for analysis, Cloud Storage for archival, Pub/Sub for streaming) test your knowledge of the **Log Router** with inclusion/exclusion filters.
- **Audit logs**: Admin Activity logs are **always enabled and free**. Data Access logs must be **explicitly enabled** and can generate significant volume/cost.
- **Managed Prometheus**: For GKE monitoring, Google recommends **Managed Service for Prometheus** -- it is the successor to legacy Stackdriver Kubernetes monitoring.
- **Deployment strategy selection**: If the question emphasizes "minimize risk" or "gradual rollout," the answer is usually **canary deployment**. If it says "instant rollback," think **blue/green**.

**Docs:** [Operational Excellence](https://cloud.google.com/architecture/framework/operational-excellence)
**Docs:** [Cloud Monitoring](https://cloud.google.com/monitoring/docs)
**Docs:** [Cloud Logging](https://cloud.google.com/logging/docs)
**Docs:** [Cloud Deploy](https://cloud.google.com/deploy/docs)
**Docs:** [SRE Workbook](https://sre.google/workbook/table-of-contents/)

---

## Pillar 2: Security, Privacy, and Compliance

### Key Principles

Google Cloud's security pillar is broader than most frameworks -- it explicitly includes **privacy and compliance** as first-class concerns. The approach follows a **defense-in-depth** model and is built on Google's BeyondCorp zero-trust architecture.

**Core principles:**

1. **Defense in depth** -- Multiple overlapping security controls at every layer (network, identity, application, data)
2. **Least privilege** -- Grant only the minimum permissions required for a task, and revoke them when no longer needed
3. **Zero trust / BeyondCorp** -- Never trust, always verify. Access decisions based on identity, device state, and context -- not network location
4. **Shift security left** -- Integrate security into the development lifecycle (DevSecOps), not as an afterthought
5. **Encrypt everything** -- Data at rest, in transit, and in use. Use customer-managed encryption keys (CMEK) for sensitive workloads
6. **Automate security controls** -- Use Organization Policies, Security Command Center, and Binary Authorization to enforce guardrails automatically
7. **Assume breach** -- Design systems to limit blast radius when (not if) a component is compromised

### GCP Services That Map to This Pillar

#### Identity and Access Management

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **Cloud IAM** | Fine-grained access control | Predefined roles, custom roles, conditions |
| **IAM Conditions** | Context-aware access | Time-based, resource-based, IP-based conditions |
| **Workload Identity Federation** | External identity to GCP without keys | Replaces service account keys for CI/CD |
| **Workforce Identity Federation** | External IdP users access GCP console/API | Azure AD, Okta, SAML/OIDC integration |
| **IAM Recommender** | Least-privilege enforcement | Suggests role reductions based on actual usage |
| **IAM Deny Policies** | Explicit permission denial | Override allows, even for Owner role |
| **Service Account** | Machine/application identity | Key rotation, short-lived credentials |
| **Organization Policy Service** | Org-wide constraints | Restrict resource locations, disable APIs, enforce labels |

#### Network Security

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **VPC Service Controls** | API-level security perimeter | Prevents data exfiltration from GCP services |
| **Cloud Armor** | WAF and DDoS protection | OWASP Top 10 rules, adaptive protection, bot management |
| **Cloud Firewall** | Network-level access control | Hierarchical policies, FQDN-based rules, threat intelligence |
| **Private Google Access** | Access Google APIs without public IPs | Internal-only access to googleapis.com |
| **Private Service Connect** | Private endpoints for managed services | Consumer-initiated or producer-initiated connections |
| **Cloud NAT** | Outbound internet for private VMs | No inbound connectivity, configurable ports |
| **Cloud IDS** | Network-based threat detection | Palo Alto Networks-powered, inspects east-west and north-south |
| **Secure Web Proxy** | Outbound web traffic inspection | URL filtering, TLS inspection |

#### Data Protection

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **Cloud KMS** | Key management | HSM-backed, key rotation, import your own keys |
| **Cloud HSM** | Hardware security modules | FIPS 140-2 Level 3 validated |
| **Cloud EKM** | External key management | Keys stay outside Google, BYOK partner integration |
| **Secret Manager** | Secret storage and versioning | Automatic rotation, IAM-controlled access |
| **DLP (Sensitive Data Protection)** | Data discovery and classification | PII detection, de-identification, 150+ infoTypes |
| **Confidential Computing** | Data-in-use encryption | Confidential VMs, Confidential GKE Nodes, Confidential Space |
| **Certificate Manager** | TLS certificate lifecycle | Auto-renewal, DNS authorization |

#### Security Posture and Detection

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **Security Command Center (SCC)** | Centralized security posture management | Vulnerability scanning, compliance, threat detection |
| **SCC Premium - Event Threat Detection** | Real-time threat detection | Suspicious logins, cryptomining, data exfiltration |
| **SCC Premium - Container Threat Detection** | Runtime container security | Reverse shells, unexpected binaries, library loading |
| **Binary Authorization** | Deploy-time container validation | Enforce signed images, attestation-based policies |
| **Model Armor** | AI/ML security guardrails | Prompt injection detection, output validation, responsible AI |
| **Web Security Scanner** | Automated web vulnerability scanning | OWASP compliance, custom scans |
| **Access Transparency** | Visibility into Google admin access | Logs when Google personnel access your data |
| **Assured Workloads** | Compliance environment provisioning | FedRAMP, HIPAA, EU data sovereignty environments |

### Architect-Level Considerations

#### Resource Hierarchy for Security

```
Organization (domain-level)
├── Org Policies (constraints applied org-wide)
├── Folder: Production
│   ├── Folder-level IAM (inherited by all projects below)
│   ├── Project: prod-app-1
│   │   └── Project-level IAM
│   └── Project: prod-data-1
│       └── VPC Service Controls perimeter
├── Folder: Non-Production
│   ├── Folder: Development
│   └── Folder: Staging
└── Folder: Shared Services
    ├── Project: shared-vpc-host
    ├── Project: security-logging
    └── Project: dns-hub
```

**Key architectural decisions:**
- Use **folders** to group projects by environment and apply IAM at the folder level
- Place **shared infrastructure** (Shared VPC host, DNS, logging) in a dedicated folder
- Apply **Org Policies** at the org or folder level, not individual projects
- Use **VPC Service Controls perimeters** around projects containing sensitive data
- Create a **centralized logging project** with aggregated log sinks

#### Common Organization Policy Constraints

```bash
# Restrict resource locations to specific regions (data sovereignty)
gcloud resource-manager org-policies set-policy \
  --organization=123456789 \
  policy.yaml
# Where policy.yaml contains:
# constraint: constraints/gcp.resourceLocations
# listPolicy:
#   allowedValues:
#     - in:europe-west1-locations
#     - in:europe-west4-locations

# Disable service account key creation (force Workload Identity)
gcloud resource-manager org-policies enable-enforce \
  constraints/iam.disableServiceAccountKeyCreation \
  --organization=123456789

# Require OS Login for all VMs
gcloud resource-manager org-policies enable-enforce \
  constraints/compute.requireOsLogin \
  --organization=123456789

# Disable external IP addresses on VMs
gcloud resource-manager org-policies enable-enforce \
  constraints/compute.vmExternalIpAccess \
  --organization=123456789

# Restrict VPC peering to approved networks
gcloud resource-manager org-policies set-policy \
  --organization=123456789 \
  vpc-peering-policy.yaml

# Require uniform bucket-level access (disable ACLs)
gcloud resource-manager org-policies enable-enforce \
  constraints/storage.uniformBucketLevelAccess \
  --organization=123456789
```

#### Encryption Decision Tree

```
Is the data extremely sensitive (PCI, HIPAA, ITAR)?
├── YES: Do you need keys outside Google?
│   ├── YES → Cloud EKM (External Key Manager)
│   └── NO: Do you need HSM-backed keys?
│       ├── YES → Cloud HSM
│       └── NO → Cloud KMS with CMEK
└── NO: Is regulatory compliance required?
    ├── YES → CMEK (Cloud KMS) -- gives you control and audit trail
    └── NO → Google-managed encryption (default, no action needed)
```

**Encryption layers (all active by default):**
1. **At rest**: AES-256, automatic, Google-managed keys (or CMEK/CSEK/EKM)
2. **In transit**: TLS 1.3 between user and Google front-end; ALTS between Google services
3. **In use**: Confidential Computing (AMD SEV / Intel TDX / NVIDIA GPU TEE)

```bash
# Create a CMEK key ring and key
gcloud kms keyrings create my-keyring \
  --location=us-central1

gcloud kms keys create my-key \
  --keyring=my-keyring \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time=$(date -u -d "+90 days" +%Y-%m-%dT%H:%M:%SZ)

# Create a Cloud SQL instance with CMEK
gcloud sql instances create my-instance \
  --database-version=POSTGRES_15 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --disk-encryption-key=projects/my-project/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key

# Create a GCS bucket with CMEK
gcloud storage buckets create gs://my-secure-bucket \
  --location=us-central1 \
  --default-encryption-key=projects/my-project/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
```

#### VPC Service Controls Architecture

VPC Service Controls create a **security perimeter** around GCP resources to prevent data exfiltration:

```
┌─────────────────────────────────────────────────────┐
│              VPC Service Controls Perimeter          │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐                │
│  │  Project A   │  │  Project B   │                │
│  │  BigQuery    │  │  Cloud       │                │
│  │  dataset     │  │  Storage     │                │
│  └──────────────┘  └──────────────┘                │
│                                                     │
│  Services inside the perimeter can communicate      │
│  freely. External access is BLOCKED unless          │
│  explicitly allowed via access levels or            │
│  ingress/egress rules.                              │
└─────────────────────────────────────────────────────┘
```

```bash
# Create an access policy
gcloud access-context-manager policies create \
  --organization=123456789 \
  --title="My Org Policy"

# Create an access level (e.g., corporate network)
gcloud access-context-manager levels create corp-network \
  --policy=POLICY_ID \
  --title="Corporate Network" \
  --basic-level-spec=corp-network-spec.yaml

# Create a service perimeter
gcloud access-context-manager perimeters create my-perimeter \
  --policy=POLICY_ID \
  --title="Sensitive Data Perimeter" \
  --resources="projects/12345,projects/67890" \
  --restricted-services="bigquery.googleapis.com,storage.googleapis.com" \
  --access-levels="corp-network"
```

#### Zero Trust / BeyondCorp Implementation

| Component | GCP Service | Purpose |
|-----------|-------------|---------|
| Device trust | **BeyondCorp Enterprise** | Device posture assessment |
| Identity-aware access | **Identity-Aware Proxy (IAP)** | Application access without VPN |
| Context-aware access | **Access Context Manager** | Conditional access based on context |
| Micro-segmentation | **VPC Firewall Rules / Policies** | East-west traffic control |
| Service identity | **Workload Identity** | Kubernetes pod identity |
| mTLS | **Traffic Director / Anthos Service Mesh** | Service-to-service encryption |

```bash
# Enable IAP for a backend service (replaces VPN)
gcloud compute backend-services update my-backend \
  --iap=enabled \
  --global

# IAP access is then controlled via IAM:
gcloud projects add-iam-policy-binding my-project \
  --member="user:developer@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --condition='expression=request.time < timestamp("2026-06-01T00:00:00Z"),title=temporary-access'
```

### Exam Tips

- **VPC Service Controls vs Firewall**: VPC-SC protects at the **API/service level** (data exfiltration prevention). Firewall protects at the **network level** (IP/port). They are complementary, not alternatives.
- **CMEK vs CSEK**: CMEK = Google hosts the key (in Cloud KMS), you manage rotation. CSEK = you supply the key with every API call (only for Compute Engine disks and Cloud Storage). For most exam scenarios, CMEK is the answer.
- **Organization Policy vs IAM**: Org policies are **constraints on resources** (what can exist). IAM is **permissions on identities** (who can do what). If the question says "prevent any VM from having an external IP across the entire organization," that is an **Org Policy** (`compute.vmExternalIpAccess`), not IAM.
- **Shared VPC vs VPC Peering**: Shared VPC provides **centralized network administration** with cross-project resource sharing. VPC Peering connects two separate VPC networks. Shared VPC is preferred for organizational control; peering is for separate administrative domains.
- **Binary Authorization trap**: It does NOT scan for vulnerabilities -- it only enforces that images have required **attestations** (signatures). Use Artifact Analysis for vulnerability scanning.
- **IAM Deny policies**: These are relatively new and can **override allows**. If a question mentions preventing even project owners from doing something, consider deny policies.
- **Workload Identity Federation**: Whenever a question mentions CI/CD from GitHub/GitLab or workloads in AWS/Azure accessing GCP, the answer is almost always **Workload Identity Federation** (eliminates service account keys).
- **Secret Manager vs KMS**: Secret Manager stores and retrieves secrets (passwords, API keys, certificates). KMS encrypts/decrypts data. They are complementary -- Secret Manager can use KMS for envelope encryption.

**Docs:** [Security Pillar](https://cloud.google.com/architecture/framework/security)
**Docs:** [Cloud IAM](https://cloud.google.com/iam/docs)
**Docs:** [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
**Docs:** [Cloud KMS](https://cloud.google.com/kms/docs)
**Docs:** [Security Command Center](https://cloud.google.com/security-command-center/docs)
**Docs:** [Organization Policy](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
**Docs:** [Binary Authorization](https://cloud.google.com/binary-authorization/docs)
**Docs:** [Identity-Aware Proxy](https://cloud.google.com/iap/docs)

---

## Pillar 3: Reliability

### Key Principles

Reliability is about ensuring your system **performs its intended function correctly and consistently** under varying conditions. Google Cloud's reliability guidance is deeply rooted in SRE principles.

**Core principles:**

1. **Design for failure** -- Every component will eventually fail. Architect systems that degrade gracefully, not catastrophically
2. **Eliminate single points of failure** -- Use redundancy at every layer (compute, storage, networking, regions)
3. **Define and measure reliability targets** -- Use SLOs with error budgets to balance reliability investment against feature velocity
4. **Automate recovery** -- Self-healing systems recover faster than human-driven processes
5. **Test reliability continuously** -- Chaos engineering, game days, DR drills, and load testing
6. **Understand failure domains** -- Know the blast radius of zonal, regional, and global failures
7. **Plan for capacity** -- Ensure sufficient headroom for traffic spikes and failover scenarios

### Failure Domain Hierarchy

Understanding GCP's failure domains is critical for the PCA exam:

```
┌──────────────────────────────────────────────────────────────┐
│                        GLOBAL                                │
│  Services: Cloud DNS, Global Load Balancer, Cloud CDN        │
│  Failure mode: Extremely rare, affects all regions            │
│                                                              │
│  ┌───────────────────────┐  ┌───────────────────────┐       │
│  │      REGION A         │  │      REGION B         │       │
│  │  (e.g., us-central1)  │  │  (e.g., us-east1)    │       │
│  │                       │  │                       │       │
│  │  ┌─────┐  ┌─────┐   │  │  ┌─────┐  ┌─────┐   │       │
│  │  │Zone │  │Zone │   │  │  │Zone │  │Zone │   │       │
│  │  │ A   │  │ B   │   │  │  │ B   │  │ C   │   │       │
│  │  └─────┘  └─────┘   │  │  └─────┘  └─────┘   │       │
│  │  ┌─────┐             │  │  ┌─────┐             │       │
│  │  │Zone │             │  │  │Zone │             │       │
│  │  │ C   │             │  │  │ D   │             │       │
│  │  └─────┘             │  │  └─────┘             │       │
│  └───────────────────────┘  └───────────────────────┘       │
└──────────────────────────────────────────────────────────────┘

Zone failure: ~0.01-0.1% of yearly hours  → Design for with regional resources
Region failure: ~0.001% of yearly hours   → Design for with multi-regional architecture
Global failure: Extremely rare             → Accept the risk or use multi-cloud
```

### GCP Services That Map to This Pillar

#### Highly Available Compute

| Service | HA Mechanism | Recovery Behavior |
|---------|-------------|-------------------|
| **Compute Engine MIG** | Multi-zone instance groups | Auto-healing, auto-replacement across zones |
| **GKE Regional Cluster** | Control plane in 3 zones, nodes across zones | Pod rescheduling, node auto-repair |
| **Cloud Run** | Automatically regional | Transparent zone failover |
| **App Engine** | Automatically regional | Automatic scaling and healing |
| **Cloud Functions** | Automatically regional | Transparent zone failover |

#### Highly Available Storage and Databases

| Service | HA Configuration | RPO | RTO |
|---------|-----------------|-----|-----|
| **Cloud Storage** | Multi-regional / dual-regional | 0 (synchronous) | Automatic |
| **Cloud SQL** | Regional HA (primary + standby in different zones) | ~0 (synchronous replication) | ~1-2 minutes (automatic failover) |
| **Cloud SQL** | Cross-region read replicas | Seconds (async) | Minutes (manual promotion) |
| **Cloud Spanner** | Multi-regional config (e.g., `nam14`) | 0 (synchronous) | Automatic |
| **Bigtable** | Multi-cluster replication | Seconds (async) | Automatic routing |
| **Firestore** | Multi-region or regional | 0 for multi-region | Automatic |
| **AlloyDB** | Regional HA with standby | ~0 (synchronous) | ~1-2 minutes |
| **Memorystore Redis** | Standard tier (primary + replica) | ~0 | ~seconds |

#### Load Balancing and Traffic Management

| Load Balancer Type | Scope | Use Case |
|-------------------|-------|----------|
| **Global External Application LB** | Global, anycast | HTTP(S) with URL-based routing, CDN integration |
| **Regional External Application LB** | Regional | Regional HTTP(S) traffic, data sovereignty |
| **Global External Proxy Network LB** | Global | TCP/SSL with global anycast |
| **Regional External Passthrough Network LB** | Regional | UDP, ESP, or TCP without proxy |
| **Internal Application LB** | Regional | Internal HTTP(S) microservices |
| **Internal Passthrough Network LB** | Regional | Internal TCP/UDP, used as next-hop |
| **Cross-region Internal Application LB** | Global | Multi-region internal HTTP(S) |

### Architect-Level Considerations

#### Disaster Recovery Patterns

| Pattern | RTO | RPO | Cost | When to Use |
|---------|-----|-----|------|-------------|
| **Backup & Restore (Cold)** | Hours to days | Hours | Lowest | Non-critical workloads, cost-sensitive |
| **Pilot Light** | Minutes to hours | Minutes | Low-Medium | Core systems kept minimal in DR region |
| **Warm Standby** | Minutes | Seconds to minutes | Medium-High | Business-critical, moderate budget |
| **Hot Standby / Active-Active** | Seconds to zero | Zero to seconds | Highest | Mission-critical, zero tolerance |

**Cold DR example:**
```bash
# Automated backup of Cloud SQL to a different region
gcloud sql instances patch my-instance \
  --backup-location=us-east1

# Cross-region Cloud Storage replication (automatic with dual-regional bucket)
gcloud storage buckets create gs://my-dr-bucket \
  --location=nam4 \
  --default-storage-class=standard

# Export Cloud SQL database to GCS (for cross-region restore)
gcloud sql export sql my-instance gs://my-dr-bucket/backup.sql \
  --database=mydb
```

**Hot DR example (Spanner multi-region):**
```bash
# Create a multi-region Spanner instance (automatic cross-region replication)
gcloud spanner instances create my-instance \
  --config=nam14 \
  --processing-units=1000 \
  --description="Multi-region North America"

# nam14 = replicas in us-central1, us-central2, us-east1, us-east4
# Automatic failover, zero RPO, near-zero RTO
```

#### RPO/RTO Calculation Guide

```
RPO (Recovery Point Objective): "How much data can we afford to lose?"
├── RPO = 0: Synchronous replication (Spanner multi-region, Cloud Storage multi-regional)
├── RPO = seconds: Asynchronous replication (Cloud SQL cross-region replicas, Bigtable replication)
├── RPO = minutes: Frequent backups (Cloud SQL automated backups)
└── RPO = hours: Daily backups (scheduled export jobs)

RTO (Recovery Time Objective): "How long can we be down?"
├── RTO = 0: Active-active (Global LB + multi-region services)
├── RTO = minutes: Automatic failover (Cloud SQL HA, GKE regional)
├── RTO = hours: Manual failover (promote read replica, restore from backup)
└── RTO = days: Full rebuild (restore from cold backup, redeploy infrastructure)
```

**Cost-reliability trade-off:**
```
Cost: $$$$  ← Active-Active (multi-region, synchronous replication)
Cost: $$$   ← Warm Standby (scaled-down replica in DR region)
Cost: $$    ← Pilot Light (minimal infrastructure, auto-scale on failover)
Cost: $     ← Backup & Restore (only storage costs in DR region)
```

#### Chaos Engineering on GCP

| Tool/Approach | What It Tests | GCP Integration |
|--------------|---------------|-----------------|
| **Chaos Monkey** (Netflix OSS) | Random instance termination | Run on GKE to kill pods |
| **Litmus Chaos** | Kubernetes-native chaos | GKE, pod/node failures, network chaos |
| **gcloud compute instances stop** | Zone failure simulation | Manually stop VMs in a zone |
| **Network fault injection** | Latency, packet loss | Istio/Envoy fault injection, Traffic Director |
| **Load testing** | Capacity limits | Cloud Load Testing (deprecated), Locust on GKE |

```bash
# Simulate zone failure: stop all instances in a zone
gcloud compute instances list \
  --filter="zone:us-central1-a AND status:RUNNING" \
  --format="value(name)" | \
  xargs -I {} gcloud compute instances stop {} --zone=us-central1-a

# GKE: drain a node to simulate node failure
kubectl drain node-name --ignore-daemonsets --delete-emptydir-data

# Verify failover by checking MIG instance redistribution
gcloud compute instance-groups managed list-instances my-mig \
  --region=us-central1
```

#### Multi-Region Architecture Patterns

**Pattern 1: Active-Passive (Warm Standby)**
```
Primary Region (us-central1)          DR Region (us-east1)
┌────────────────────────┐      ┌────────────────────────┐
│  Global External LB    │      │                        │
│       ↓                │      │                        │
│  GKE Regional Cluster  │      │  GKE (scaled down)     │
│       ↓                │      │       ↓                │
│  Cloud SQL (Primary)   │──────│  Cloud SQL (Replica)   │
│  w/ Regional HA        │ async│  Read replica           │
└────────────────────────┘      └────────────────────────┘
```

**Pattern 2: Active-Active**
```
             Global External Application Load Balancer
                    ↙                        ↘
Region A (us-central1)              Region B (us-east1)
┌──────────────────────┐     ┌──────────────────────┐
│  GKE Regional        │     │  GKE Regional        │
│  Cluster             │     │  Cluster             │
│       ↓              │     │       ↓              │
│  Spanner (nam14)     │     │  Spanner (nam14)     │
│  (shared multi-      │     │  (shared multi-      │
│   region instance)   │     │   region instance)   │
└──────────────────────┘     └──────────────────────┘
```

### Exam Tips

- **Cloud SQL HA is NOT multi-region**: Regional HA with a standby in a different zone is automatic failover within ONE region. Cross-region requires read replicas with manual promotion.
- **Spanner multi-region configs**: Know the named configs -- `nam14` (North America), `eur6` (Europe), `nam-eur-asia1` (global). These provide zero RPO and automatic failover.
- **Cloud Storage classes and availability**: Multi-regional = 99.95%, Regional = 99.9%, Nearline/Coldline/Archive = 99.0%/99.0%/99.0%. Archive is NOT slower to read -- it just has higher retrieval costs and minimum storage duration (365 days).
- **Global vs Regional Load Balancer**: If the question mentions multi-region backend or anycast IP, the answer is **global**. If it mentions data sovereignty or single-region, the answer is **regional**.
- **GKE regional cluster**: The control plane runs in 3 zones automatically. Node pools can be single-zone or multi-zone. For HA, always use **regional clusters with multi-zone node pools**.
- **RPO = 0 requires synchronous replication**: Only achievable with services like Spanner multi-region, Cloud Storage multi-regional, or Firestore multi-region. Async replication always has RPO > 0.
- **DR drill frequency**: Google recommends testing DR plans at least **quarterly**. If a question asks about DR best practices, testing regularly is always part of the answer.
- **Capacity planning for failover**: If you have 3 zones and lose 1, remaining zones must handle 100% of traffic. This means running at ~67% capacity normally (N+1 redundancy).

**Docs:** [Reliability Pillar](https://cloud.google.com/architecture/framework/reliability)
**Docs:** [Disaster Recovery Planning](https://cloud.google.com/architecture/dr-scenarios-planning-guide)
**Docs:** [Cloud Load Balancing](https://cloud.google.com/load-balancing/docs)
**Docs:** [Cloud Spanner Multi-Region](https://cloud.google.com/spanner/docs/instance-configurations)
**Docs:** [Cloud SQL HA](https://cloud.google.com/sql/docs/mysql/high-availability)
**Docs:** [GKE Regional Clusters](https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters)

---

## Pillar 4: Cost Optimization

### Key Principles

Cost optimization is about achieving business outcomes at the lowest price point while maintaining acceptable quality. On the PCA exam, this pillar often appears as a secondary concern: "which solution meets the requirements AND is most cost-effective?"

**Core principles:**

1. **Right-size resources** -- Match resource capacity to actual demand, not peak theoretical demand
2. **Use committed discounts strategically** -- CUDs for predictable workloads, SUDs for sustained usage
3. **Eliminate waste** -- Identify and remove idle resources, orphaned disks, unused IP addresses
4. **Leverage managed services** -- Reduce operational overhead (and thus cost) by using serverless and managed offerings
5. **Architect for cost visibility** -- Use labels, billing exports, and budgets for cost allocation and tracking
6. **Optimize data costs** -- Use appropriate storage classes, lifecycle policies, and data transfer strategies
7. **Adopt FinOps practices** -- Create a culture of cost accountability across engineering teams

### GCP Pricing Models

| Model | Description | Discount | Commitment |
|-------|-------------|----------|------------|
| **On-demand** | Pay per second/minute/hour | None (baseline) | None |
| **Sustained Use Discounts (SUDs)** | Automatic discounts for running VMs 25%+ of a month | Up to **20%** | None (automatic) |
| **Committed Use Discounts (CUDs)** | 1-year or 3-year resource commitments | **20-57%** | 1 or 3 years |
| **Spot VMs (Preemptible)** | Interruptible VMs at steep discounts | **60-91%** | None (can be reclaimed anytime) |
| **Flat-rate pricing** | BigQuery, Spanner capacity-based pricing | Varies | Edition commitment |
| **Free tier** | Always-free products and limits | 100% (within limits) | None |

### CUD vs SUD Decision Matrix

```
Is the workload predictable (stable baseline)?
├── YES: Will it run for 1+ years?
│   ├── YES (3+ years) → 3-year CUD (largest discount: ~57% for memory-optimized)
│   ├── YES (1-3 years) → 1-year CUD (~20-37% discount)
│   └── YES (< 1 year) → Let SUDs apply automatically
└── NO: Is it fault-tolerant / batch processing?
    ├── YES → Spot VMs (60-91% discount, but can be preempted)
    └── NO → On-demand with autoscaling
```

**CUD types:**
- **Resource-based CUDs**: Commit to vCPUs and memory (flexible across machine types/regions within a commitment)
- **Spend-based CUDs**: Commit to a dollar amount per hour for specific services (Cloud SQL, Memorystore, Cloud Run, GKE Autopilot)

```bash
# View existing committed use discounts
gcloud compute commitments list

# Create a 1-year CUD for 16 vCPUs and 64 GB memory
gcloud compute commitments create my-commitment \
  --region=us-central1 \
  --plan=12-month \
  --resources=vcpu=16,memory=64GB

# List recommendations for cost optimization
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=my-project \
  --location=us-central1-a \
  --format="table(content.operationGroups[0].operations[0].resource, \
    content.operationGroups[0].operations[0].value.machineType)"
```

### GCP Services That Map to This Pillar

| Service | Role in Cost Optimization | Key Feature |
|---------|--------------------------|-------------|
| **Billing Export to BigQuery** | Detailed cost analysis and custom reporting | Line-item billing data, custom dashboards |
| **Billing Budgets & Alerts** | Proactive cost control | Threshold alerts, Pub/Sub integration for automation |
| **Recommender** | AI-powered optimization suggestions | Idle VMs, rightsizing, unattached disks, overprovisioned projects |
| **Active Assist** | Umbrella for all recommendation services | Cost, security, performance, sustainability recommendations |
| **Spot VMs** | Steep discounts for fault-tolerant workloads | 60-91% off, 30-second termination notice |
| **Autoscaling** | Scale resources with demand | MIG autoscaler, GKE HPA/VPA, Cloud Run concurrency |
| **Cloud Storage Lifecycle** | Automatic storage class transitions | Nearline (30d) → Coldline (90d) → Archive (365d) |
| **Preemptible/Spot Node Pools** | Cost-effective GKE nodes | Ideal for batch processing, CI/CD runners |
| **BigQuery Editions** | Capacity-based pricing with autoscaling | Standard, Enterprise, Enterprise Plus editions |
| **Cloud Functions / Cloud Run** | Pay-per-invocation/request | Zero cost when idle |
| **Committed Use Discounts** | Reserved capacity discounts | Resource-based and spend-based options |

### Architect-Level Considerations

#### Cost Allocation Strategy with Labels

Labels are key-value pairs attached to GCP resources that enable cost allocation, filtering, and chargeback:

```bash
# Apply labels to resources for cost tracking
gcloud compute instances create my-vm \
  --labels=team=data-engineering,env=production,cost-center=cc-1234

# Label an existing Cloud SQL instance
gcloud sql instances patch my-instance \
  --update-labels=team=backend,env=staging

# Export billing data to BigQuery (configured in Console > Billing > Billing export)
# Then query:
# SELECT labels.value AS team, SUM(cost) AS total_cost
# FROM `project.dataset.gcp_billing_export_v1_XXXXXX`
# WHERE labels.key = 'team'
# GROUP BY team
# ORDER BY total_cost DESC
```

**Label strategy best practices:**
- Enforce required labels via **Org Policy** (`constraints/compute.requireLabels`)
- Standard labels: `team`, `env`, `cost-center`, `app`, `owner`, `managed-by`
- Use labels in billing exports for team/project chargeback dashboards

#### TCO Analysis Framework

| Cost Category | On-Premises | GCP Cloud |
|--------------|-------------|-----------|
| **Hardware** | CapEx: servers, storage, networking | OpEx: pay-as-you-go compute/storage |
| **Software licenses** | Perpetual licenses | BYOL or pay-as-you-go (e.g., Cloud SQL for SQL Server) |
| **Facility** | Data center, power, cooling | Included in service pricing |
| **Personnel** | Large ops team | Smaller team (managed services reduce toil) |
| **Network** | Fixed cost for links | Egress-based pricing (ingress is free) |
| **DR** | Full secondary data center | Pay for what you use (cold DR = storage only) |
| **Scaling** | Over-provision for peak | Autoscale to demand |

**Key TCO insight for exam**: GCP's pricing model shifts from **CapEx to OpEx**. This benefits organizations that want to:
- Reduce upfront investment
- Convert fixed costs to variable costs
- Scale elastically without over-provisioning
- Avoid end-of-life hardware refresh cycles

#### Data Transfer Cost Optimization

| Traffic Type | Cost |
|-------------|------|
| **Ingress** (data into GCP) | **Free** |
| **Egress** to internet | $0.08-$0.23/GB (tiered, decreases with volume) |
| **Egress** between regions (same continent) | $0.01/GB |
| **Egress** between regions (intercontinental) | $0.02-$0.08/GB |
| **Egress** within same zone | Free (internal IPs) |
| **Egress** within same region, different zones | $0.01/GB |
| **Private Google Access** | Free (within same region) |
| **Cloud Interconnect** egress | $0.02/GB (vs $0.08-$0.23 for internet) |

**Optimization strategies:**
- Keep compute and data in the **same region** to avoid cross-region transfer costs
- Use **Cloud CDN** to cache content at edge PoPs, reducing origin egress
- Use **Cloud Interconnect** or **Partner Interconnect** for high-volume data transfer (lower egress rates)
- Use **Transfer Appliance** for large one-time migrations (>20 TB)
- Use **Cloud Storage Transfer Service** for scheduled cross-cloud or cross-region transfers

#### Storage Cost Optimization

```bash
# Set lifecycle policy on a bucket (auto-transition to cheaper storage classes)
cat > lifecycle.json << 'EOF'
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30, "matchesStorageClass": ["STANDARD"]}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90, "matchesStorageClass": ["NEARLINE"]}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
        "condition": {"age": 365, "matchesStorageClass": ["COLDLINE"]}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 2555}
      }
    ]
  }
}
EOF

gcloud storage buckets update gs://my-bucket --lifecycle-file=lifecycle.json
```

| Storage Class | Min Duration | Retrieval Cost | Use Case |
|--------------|-------------|----------------|----------|
| **Standard** | None | None | Frequently accessed data |
| **Nearline** | 30 days | $0.01/GB | Once a month access |
| **Coldline** | 90 days | $0.02/GB | Once a quarter access |
| **Archive** | 365 days | $0.05/GB | Once a year access, compliance |

#### Budgets and Automated Cost Controls

```bash
# Create a billing budget with alerts
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Monthly Budget" \
  --budget-amount=10000USD \
  --threshold-rules=percent=0.5,basis=CURRENT_SPEND \
  --threshold-rules=percent=0.9,basis=CURRENT_SPEND \
  --threshold-rules=percent=1.0,basis=FORECASTED_SPEND \
  --notifications-rule-pubsub-topic=projects/my-project/topics/budget-alerts

# Note: Budgets DO NOT stop spending! They only alert.
# To stop spending, use Pub/Sub + Cloud Functions to disable billing:
# 1. Budget alert → Pub/Sub topic
# 2. Cloud Function triggered by Pub/Sub
# 3. Function calls billing API to disable billing on the project
```

### Exam Tips

- **Budgets do NOT stop spending**: This is a classic exam trap. Budgets only send alerts. To actually stop spending, you must automate billing disabling via Cloud Functions triggered by budget Pub/Sub notifications.
- **SUDs are automatic**: You do NOT need to configure SUDs. They apply automatically when a VM runs for more than 25% of a month. CUDs require explicit commitment.
- **CUD flexibility**: Resource-based CUDs can be shared across projects in the same billing account. They apply to the most expensive eligible VMs first (to maximize your savings).
- **Spot VMs for batch**: If a question mentions batch processing, ML training, or CI/CD and asks for the cheapest option, **Spot VMs** are almost always correct. They are not suitable for serving live traffic.
- **Serverless for variable workloads**: Cloud Functions and Cloud Run scale to zero, meaning you pay nothing when there is no traffic. For unpredictable or spiky workloads, serverless is the most cost-effective option.
- **Cloud Storage class trap**: Archive class is NOT slower to retrieve. It has the same latency as other classes. The cost difference is in storage price (lowest) vs retrieval price (highest) and minimum storage duration (365 days).
- **BigQuery pricing**: On-demand = $6.25/TB scanned. Enterprise editions = capacity-based with autoscaling slots. For predictable, heavy query workloads, editions with commitments are cheaper.
- **Data locality = cost savings**: Always co-locate compute and storage in the same region. Cross-region data transfer adds cost.
- **Preemptible vs Spot**: Spot VMs replaced Preemptible VMs. They are functionally identical (interruptible, 60-91% discount) but Spot VMs do NOT have the 24-hour maximum runtime limit that Preemptible VMs had.

**Docs:** [Cost Optimization Pillar](https://cloud.google.com/architecture/framework/cost-optimization)
**Docs:** [Billing Budgets](https://cloud.google.com/billing/docs/how-to/budgets)
**Docs:** [Committed Use Discounts](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview)
**Docs:** [Cloud Storage Pricing](https://cloud.google.com/storage/pricing)
**Docs:** [Recommender](https://cloud.google.com/recommender/docs)
**Docs:** [Spot VMs](https://cloud.google.com/compute/docs/instances/spot)
**Docs:** [BigQuery Editions](https://cloud.google.com/bigquery/docs/editions-intro)

---

## Pillar 5: Performance Optimization

### Key Principles

Performance optimization ensures that resources are used efficiently to meet system requirements as demand changes. On the PCA exam, this pillar is about selecting the right services, architectures, and configurations to meet latency, throughput, and scalability requirements.

**Core principles:**

1. **Select the right compute** -- Match the compute platform (VMs, containers, serverless) to the workload characteristics
2. **Optimize data access patterns** -- Use caching, CDN, and data locality to minimize latency
3. **Design for horizontal scaling** -- Scale out (more instances) rather than up (bigger instances) for elasticity
4. **Minimize latency** -- Place resources close to users, use Premium Network Tier, implement caching layers
5. **Benchmark and test** -- Establish performance baselines and test under realistic load
6. **Use appropriate storage** -- Match storage type (block, file, object) and tier to access patterns
7. **Leverage specialized hardware** -- Use GPUs, TPUs, and custom machine types for specific workloads

### Compute Selection Decision Matrix

| Requirement | Best Compute Option | Why |
|-------------|-------------------|-----|
| Full VM control, custom software | **Compute Engine** | Full OS access, any software stack |
| Containerized microservices, high scale | **GKE Autopilot** | Managed Kubernetes, auto-scaling, auto-provisioning |
| HTTP services, rapid scaling | **Cloud Run** | Container-based, scales to zero, per-request pricing |
| Event-driven, lightweight functions | **Cloud Functions** | Function-based, per-invocation, auto-scaling |
| Legacy apps, minimal refactoring | **App Engine (Standard/Flexible)** | PaaS, managed runtime, auto-scaling |
| ML training (large models) | **Compute Engine + GPUs/TPUs** or **Vertex AI Training** | Accelerator access, distributed training |
| ML inference (serving) | **Vertex AI Endpoints** or **GKE + GPUs** | Auto-scaling inference, model versioning |
| Batch processing | **Batch** or **Dataflow** | Managed batch, autoscaling, Spot VM support |
| HPC / tightly-coupled workloads | **Compute Engine + HPC Toolkit** | Placement policies, high-bandwidth networking |

### GCP Services That Map to This Pillar

#### Caching and Content Delivery

| Service | Purpose | Latency Impact |
|---------|---------|----------------|
| **Cloud CDN** | Global content caching at edge PoPs | Milliseconds (cache hit) vs 100s of ms (origin) |
| **Media CDN** | Optimized for video/large file delivery | Adaptive bitrate, large-scale streaming |
| **Memorystore for Redis** | In-memory key-value cache | Sub-millisecond reads |
| **Memorystore for Valkey** | Redis-compatible open-source caching | Sub-millisecond reads, no vendor lock-in |
| **Memorystore for Memcached** | Distributed in-memory cache | Sub-millisecond reads for simple key-value |
| **AlloyDB Columnar Engine** | In-memory columnar storage for analytics | 100x faster analytical queries on OLTP data |

#### Networking for Performance

| Service | Purpose | Key Feature |
|---------|---------|-------------|
| **Premium Network Tier** | Google's global backbone for traffic routing | Lowest latency, traffic enters Google network at nearest PoP |
| **Standard Network Tier** | Public internet routing | Higher latency, lower cost |
| **Cloud Interconnect** | Dedicated/partner connections to GCP | 10-200 Gbps, private connectivity, lower latency |
| **Cloud DNS** | Managed authoritative DNS | 100% SLA, anycast, DNSSEC support |
| **Traffic Director** | Global traffic management for service mesh | Load balancing across regions, traffic splitting |
| **Network Endpoint Groups (NEGs)** | Fine-grained load balancing targets | Container-native LB (bypasses kube-proxy for lower latency) |

#### Database Performance

| Service | Optimal For | Performance Profile |
|---------|-------------|-------------------|
| **Cloud Spanner** | Global, strongly consistent RDBMS | Horizontal scaling, unlimited reads/writes |
| **Bigtable** | High-throughput, low-latency NoSQL | Single-digit ms latency at any scale, 10K+ nodes |
| **Firestore** | Mobile/web app document DB | Auto-scaling, offline support, strong consistency |
| **BigQuery** | Analytical queries on petabytes | Serverless, columnar, automatic optimization |
| **AlloyDB** | PostgreSQL-compatible, high performance | 4x faster transactions, 100x faster analytics vs standard PG |
| **Cloud SQL** | Standard RDBMS (MySQL/PG/SQL Server) | Up to 128 vCPUs, 864 GB RAM |

#### Specialized Hardware

| Hardware | Service | Use Case | Performance Gain |
|----------|---------|----------|-----------------|
| **NVIDIA GPUs** (T4, L4, A100, H100) | Compute Engine, GKE, Vertex AI | ML training/inference, rendering | 10-100x for parallel workloads |
| **Cloud TPU** (v4, v5e, v5p) | TPU VMs, Vertex AI, GKE | Large language model training | Purpose-built for TensorFlow/JAX |
| **Local SSD** | Compute Engine, GKE | High-IOPS temporary storage | 680K read IOPS, 360K write IOPS |
| **Hyperdisk** | Compute Engine, GKE | High-throughput persistent storage | Up to 2.4 TB/s throughput |
| **Persistent Disk (pd-ssd)** | Compute Engine, GKE | General high-performance persistent | 100K IOPS per VM |

### Architect-Level Considerations

#### Latency Budget Analysis

An architect must break down the end-to-end latency budget:

```
User request total latency target: 200ms
├── DNS resolution:           ~5ms   (Cloud DNS, cached)
├── Network (user → LB):      ~20ms  (Premium Tier, nearest PoP)
├── Load balancer:             ~5ms   (Global External Application LB)
├── Network (LB → backend):   ~5ms   (within region)
├── Application processing:   ~100ms (compute time)
├── Database query:            ~30ms  (Spanner, within region)
├── Cache hit alternative:     ~1ms   (Memorystore)
├── Response serialization:    ~10ms
└── Network (backend → user):  ~25ms
                              -------
Total:                        ~200ms (within budget)
```

**Optimization levers:**
- **CDN**: Bypass backend entirely for cacheable content (~5ms vs ~200ms)
- **Caching layer**: Replace database queries with Memorystore lookups (~1ms vs ~30ms)
- **Connection pooling**: Reduce database connection overhead
- **Premium Network Tier**: Lower network latency (packets enter Google backbone sooner)
- **Container-native LB (NEGs)**: Skip kube-proxy for lower intra-cluster latency
- **Regional services**: Co-locate compute and data

#### Autoscaling Strategies

```bash
# Compute Engine MIG autoscaling based on CPU
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region=us-central1 \
  --min-num-replicas=3 \
  --max-num-replicas=20 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=90

# GKE Horizontal Pod Autoscaler (HPA)
kubectl autoscale deployment my-app \
  --min=3 --max=50 --cpu-percent=70

# GKE Vertical Pod Autoscaler (VPA) -- right-size pod resources
# Apply via YAML manifest:
# apiVersion: autoscaling.k8s.io/v1
# kind: VerticalPodAutoscaler
# metadata:
#   name: my-app-vpa
# spec:
#   targetRef:
#     apiVersion: apps/v1
#     kind: Deployment
#     name: my-app
#   updatePolicy:
#     updateMode: "Auto"  # or "Off" for recommendations only

# Cloud Run autoscaling
gcloud run deploy my-service \
  --min-instances=1 \
  --max-instances=100 \
  --concurrency=80

# GKE Cluster Autoscaler (scales node pool)
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=3 \
  --max-nodes=100 \
  --node-pool=default-pool
```

| Autoscaler | Scope | Scales On | Best For |
|-----------|-------|-----------|----------|
| **MIG Autoscaler** | Compute Engine instances | CPU, LB utilization, custom metrics, schedules | VM-based workloads |
| **GKE HPA** | Kubernetes pods | CPU, memory, custom metrics, Pub/Sub queue depth | Container workloads |
| **GKE VPA** | Kubernetes pod resource requests | Historical resource usage | Right-sizing pod requests/limits |
| **GKE Cluster Autoscaler** | Kubernetes nodes | Pending pods that cannot be scheduled | Node pool capacity |
| **GKE Node Auto-provisioning (NAP)** | Kubernetes node pools | Workload requirements | Automatic node pool creation |
| **Cloud Run** | Container instances | Concurrent requests | HTTP services |
| **Cloud Functions** | Function instances | Incoming events | Event-driven workloads |
| **Bigtable Autoscaler** | Bigtable nodes | CPU utilization, storage utilization | Time-series, IoT data |
| **Spanner Autoscaler** | Processing units | CPU utilization | Globally distributed DB |

#### Database Performance Optimization

```bash
# Cloud SQL: Add read replicas for read-heavy workloads
gcloud sql instances create read-replica \
  --master-instance-name=my-primary \
  --region=us-central1 \
  --tier=db-custom-8-32768

# Cloud SQL: Enable query insights for slow query analysis
gcloud sql instances patch my-instance \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=1024 \
  --insights-config-record-application-tags \
  --insights-config-record-client-address

# Bigtable: Create a cluster for high performance
gcloud bigtable clusters create my-cluster \
  --instance=my-instance \
  --zone=us-central1-a \
  --num-nodes=5 \
  --storage-type=SSD  # SSD for latency-sensitive; HDD for throughput-heavy, cost-sensitive

# Spanner: Scale processing units
gcloud spanner instances update my-instance \
  --processing-units=3000  # 1000 PU ≈ 1 node, scale in 100 PU increments

# BigQuery: Use materialized views for repeated queries
# CREATE MATERIALIZED VIEW my_dataset.my_mv AS
# SELECT date, SUM(revenue) as total_revenue
# FROM my_dataset.sales
# GROUP BY date;

# BigQuery: Partitioning for scan reduction
# CREATE TABLE my_dataset.events
# PARTITION BY DATE(event_timestamp)
# CLUSTER BY user_id
# AS SELECT * FROM my_dataset.raw_events;
```

#### Network Performance Decisions

| Scenario | Recommended Configuration | Why |
|----------|--------------------------|-----|
| Global user base, latency-sensitive | Premium Tier + Global LB + Cloud CDN | Anycast routing, edge caching |
| Single-region, cost-sensitive | Standard Tier + Regional LB | Lower cost, acceptable latency |
| Hybrid connectivity, high bandwidth | Dedicated Interconnect | 10-200 Gbps, private, SLA-backed |
| Hybrid, moderate bandwidth | Partner Interconnect | 50 Mbps - 50 Gbps, via partner |
| Burst connectivity to on-prem | Cloud VPN (HA VPN) | IPSec tunnels, 3 Gbps per tunnel |
| Microservices communication | Container-native LB (NEGs) + Service Mesh | Bypass kube-proxy, traffic policies |

### Exam Tips

- **Premium vs Standard Network Tier**: If the question mentions lowest latency for global users, the answer is **Premium Tier** (traffic routes through Google's private backbone). Standard Tier uses public internet routing and is cheaper but higher latency.
- **Cloud CDN cache key**: By default, the cache key includes the URI. You can customize it to include headers, cookies, or query parameters for more granular caching. If users in different regions get stale data, check the cache key configuration.
- **Bigtable row key design**: The exam often tests Bigtable performance. If the question mentions "hotspotting" or "uneven load," the answer is usually **fix the row key design** (avoid monotonically increasing keys like timestamps; use salted/reversed keys).
- **BigQuery partitioning vs clustering**: Partitioning = reduces data scanned (cost + performance). Clustering = sorts data within partitions (performance). Use both for large tables.
- **GKE Autopilot vs Standard**: Autopilot automatically provisions and manages nodes (including autoscaling). For the exam, Autopilot is the answer when the question emphasizes "reduce operational overhead" or "managed Kubernetes."
- **Connection pooling**: If a question describes connection exhaustion to Cloud SQL, the answer is usually **connection pooling** via Cloud SQL Proxy sidecar or application-level pooling, or **Cloud SQL Auth Proxy + AlloyDB Omni** for advanced use cases.
- **Local SSD vs Persistent Disk**: Local SSDs provide highest IOPS but are ephemeral (data lost when VM stops). pd-ssd provides high IOPS with persistence. If the question says "temporary high-IOPS storage," choose Local SSD; if "persistent high-IOPS," choose pd-ssd.
- **Hyperdisk**: Google's latest block storage, offering higher throughput and IOPS than Persistent Disk. If the question mentions "extreme IOPS requirements" with persistence, Hyperdisk is the answer.

**Docs:** [Performance Optimization Pillar](https://cloud.google.com/architecture/framework/performance-optimization)
**Docs:** [Cloud CDN](https://cloud.google.com/cdn/docs)
**Docs:** [Memorystore](https://cloud.google.com/memorystore/docs)
**Docs:** [Cloud Bigtable Performance](https://cloud.google.com/bigtable/docs/performance)
**Docs:** [GKE Autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
**Docs:** [Network Service Tiers](https://cloud.google.com/network-tiers/docs)
**Docs:** [Persistent Disk and Local SSD](https://cloud.google.com/compute/docs/disks)
**Docs:** [BigQuery Partitioning](https://cloud.google.com/bigquery/docs/partitioned-tables)

---

## Pillar 6: Sustainability

### Key Principles

Sustainability is about minimizing the environmental impact of cloud workloads. While it is the least-weighted pillar on the PCA exam, it does appear in questions -- particularly when combined with cost optimization (they often align).

**Core principles:**

1. **Choose low-carbon regions** -- Google publishes carbon intensity data per region; some regions run on nearly 100% carbon-free energy (CFE)
2. **Right-size resources** -- Over-provisioned resources waste energy and money
3. **Use serverless and autoscaling** -- Scale to zero when not needed, eliminating idle resource energy consumption
4. **Optimize storage** -- Use lifecycle policies to transition data to efficient storage classes and delete what you no longer need
5. **Leverage managed services** -- Google operates infrastructure more efficiently than most organizations can on-premises
6. **Measure and track** -- Use the Carbon Footprint dashboard to understand and reduce your impact

### GCP's Sustainability Commitments

| Commitment | Details |
|-----------|---------|
| **Carbon-free energy by 2030** | Google aims to run on 24/7 CFE on every grid where it operates by 2030 |
| **Net-zero emissions** | Across all operations and value chain by 2030 |
| **Carbon-free energy score** | Published per region (e.g., us-central1 = 93% CFE, europe-north1 = 97% CFE) |
| **Circular economy** | Google designs servers for reuse, refurbishment, and recycling |
| **PUE (Power Usage Effectiveness)** | Google's average PUE is ~1.10 (industry average is ~1.58) |

### Carbon-Free Energy by Region (Selected)

| Region | Location | Approximate CFE Score |
|--------|----------|--------------------|
| `europe-north1` | Finland | ~97% |
| `us-central1` | Iowa | ~93% |
| `europe-west1` | Belgium | ~78% |
| `us-east1` | South Carolina | ~53% |
| `asia-northeast1` | Tokyo | ~28% |
| `us-west1` | Oregon | ~89% |
| `northamerica-northeast1` | Montreal | ~99% |

**Note:** These scores change over time. Check the [Carbon Free Energy for Google Cloud regions](https://cloud.google.com/sustainability/region-carbon) page for current data.

### GCP Services That Map to This Pillar

| Service | Sustainability Role | Impact |
|---------|-------------------|--------|
| **Carbon Footprint Dashboard** | Gross and market-based carbon emissions per project, region, service | Visibility into environmental impact |
| **Active Assist Sustainability Recommendations** | Identifies workloads that could be moved to lower-carbon regions | Actionable region migration suggestions |
| **Cloud Run / Cloud Functions** | Scale to zero | Zero energy when idle |
| **GKE Autopilot** | Efficient bin-packing of pods | Less wasted node capacity |
| **Autoscaling (all services)** | Match capacity to demand | No idle over-provisioning |
| **Cloud Storage Lifecycle Policies** | Automatic data deletion and class transitions | Reduced storage footprint |
| **Spot VMs** | Use excess compute capacity | Better utilization of existing hardware |
| **Batch** | Managed batch processing | Optimized scheduling on available infrastructure |

### Architect-Level Considerations

#### Region Selection for Sustainability

When multiple regions meet the business requirements (latency, data residency, availability), prefer the region with the highest CFE score:

```
Business requirements:
  - Serve North American users
  - Data must stay in North America
  - Latency < 100ms from east coast

Candidate regions:
  ┌─────────────────────────┬──────────┬──────────────┬───────────┐
  │ Region                  │ CFE %    │ Latency (NYC)│ Selection │
  ├─────────────────────────┼──────────┼──────────────┼───────────┤
  │ northamerica-northeast1 │ ~99%     │ ~15ms        │ BEST      │
  │ us-central1             │ ~93%     │ ~40ms        │ Good      │
  │ us-east1                │ ~53%     │ ~10ms        │ Meets req │
  └─────────────────────────┴──────────┴──────────────┴───────────┘

  Recommendation: northamerica-northeast1 (Montreal)
  - Highest CFE score
  - Meets latency requirement
  - Data stays in North America
```

#### Sustainable Architecture Patterns

**Pattern 1: Serverless-First Architecture (Maximize Scale-to-Zero)**
```
Cloud Run (scale to zero) → Cloud Functions (event processing) → BigQuery (serverless analytics)
                         ↓
                    Firestore (serverless database)

Energy savings: No idle compute. Resources only consumed during active requests.
```

**Pattern 2: Efficient Batch Processing**
```
Cloud Scheduler → Batch (Spot VMs) → Cloud Storage (result output)

Energy savings: Use Spot VMs (existing excess capacity). Schedule batch jobs during
high-CFE periods (afternoon in Iowa when solar production peaks).
```

**Pattern 3: Data Lifecycle Optimization**
```
Ingest → Standard Storage → [30d] Nearline → [90d] Coldline → [365d] Archive → [7y] Delete

Energy savings: Reduced storage footprint. Archive class uses less active infrastructure.
```

```bash
# View Carbon Footprint data via API
gcloud beta carbon-footprint get \
  --billing-account=BILLING_ACCOUNT_ID

# Or access via BigQuery export:
# Console > Carbon Footprint > Export to BigQuery
# SELECT month, total_carbon_footprint_kgCO2e, region
# FROM `project.dataset.carbon_footprint`
# ORDER BY total_carbon_footprint_kgCO2e DESC
```

### Exam Tips

- **Sustainability + Cost often align**: Serverless, right-sizing, and autoscaling reduce both cost AND carbon footprint. When a question asks for the "most cost-effective AND sustainable" option, these overlap.
- **Region selection questions**: If a question mentions sustainability or carbon footprint AND gives you a choice of regions, pick the one with the highest CFE score that meets the other requirements.
- **Carbon Footprint dashboard**: Know that it exists and provides gross emissions data broken down by project, region, and service. It integrates with billing export to BigQuery.
- **Sustainability is rarely the primary driver**: On the PCA exam, sustainability considerations usually appear as a secondary factor (tiebreaker) when two options are otherwise equivalent.
- **Google's PUE**: Google's average data center PUE of ~1.10 is one of the lowest in the industry. This means moving from on-premises to GCP typically reduces the carbon footprint of the same workload -- a migration benefit.
- **Spot VMs and sustainability**: Spot VMs use excess capacity that would otherwise be idle. This is both cost-effective and sustainable -- double benefit.

**Docs:** [Sustainability Pillar](https://cloud.google.com/architecture/framework/sustainability)
**Docs:** [Carbon Footprint](https://cloud.google.com/carbon-footprint)
**Docs:** [Carbon Free Energy for Google Cloud Regions](https://cloud.google.com/sustainability/region-carbon)
**Docs:** [Active Assist](https://cloud.google.com/solutions/active-assist)

---

## How WAF Maps to PCA Exam Questions

### Decision Framework

When you encounter a PCA exam question that presents multiple architecture options, use this mental framework:

```
Step 1: Identify the PRIMARY requirement
  └── What is the question REALLY asking? (reliability? cost? security? performance?)

Step 2: Identify SECONDARY constraints
  └── Are there budget constraints? Compliance requirements? Latency targets?

Step 3: Evaluate each option against the relevant pillars
  └── Does Option A satisfy the primary pillar better than Option B?

Step 4: Apply the trade-off hierarchy
  └── When pillars conflict, which takes priority for THIS scenario?

Step 5: Eliminate obviously wrong answers
  └── Does any option violate a hard constraint? (e.g., data residency, budget cap)
```

**Example question analysis:**

> "A company needs to store customer financial data in Europe. The data must be encrypted with keys managed by the company's security team. The solution should minimize operational overhead. Which approach should you recommend?"

```
Step 1: PRIMARY = Security (encryption, key management, data residency)
Step 2: SECONDARY = Operational Excellence (minimize overhead)
Step 3: Evaluate:
  A) Default Google-managed encryption + europe-west1 → Meets residency, fails key mgmt
  B) CMEK with Cloud KMS + europe-west1 → Meets all, managed service = low overhead
  C) CSEK (Customer-supplied keys) + europe-west1 → Meets all, but HIGH operational overhead
  D) Cloud EKM + europe-west1 → Meets all, but higher overhead than CMEK
Step 4: Between B and D: question says "minimize overhead" → CMEK (B) wins
Step 5: Eliminate A (no company key management), C (high overhead)
Answer: B
```

### Common Question Patterns

| Question Pattern | Primary Pillar(s) | Key Signal Words |
|-----------------|-------------------|------------------|
| "Which solution minimizes cost?" | Cost Optimization | "cost-effective," "budget," "minimize spending," "cheapest" |
| "Which solution is most resilient?" | Reliability | "high availability," "disaster recovery," "minimal downtime," "failover" |
| "Which solution meets compliance?" | Security | "compliance," "regulation," "HIPAA," "PCI," "data residency," "encryption" |
| "Which solution provides lowest latency?" | Performance | "fastest," "lowest latency," "performance," "throughput" |
| "Which solution reduces operational overhead?" | Operational Excellence | "managed," "serverless," "reduce toil," "automate," "minimal operations" |
| "Which solution is most scalable?" | Performance + Reliability | "scale," "growing," "elastic," "peak traffic," "millions of users" |
| "Which solution is best for the environment?" | Sustainability | "carbon footprint," "green," "sustainable," "environmental impact" |
| "Which solution best balances cost and reliability?" | Trade-off question | Multiple signals -- read carefully for the PRIMARY constraint |

### Architecture Decision Signals

| If the question says... | The answer likely involves... |
|------------------------|-------------------------------|
| "globally distributed users" | Global LB + multi-region + Cloud CDN |
| "strict data residency" | Regional resources + Org Policy `gcp.resourceLocations` |
| "RPO of zero" | Synchronous replication (Spanner multi-region, multi-regional GCS) |
| "minimize blast radius" | Micro-segmentation, VPC-SC, separate projects, least privilege |
| "lift and shift" | Compute Engine, Migrate to VMs, minimal refactoring |
| "modernize" | Containers (GKE), managed services, Cloud Run |
| "real-time analytics" | BigQuery streaming, Bigtable, Pub/Sub + Dataflow |
| "batch processing, cost-sensitive" | Batch with Spot VMs, Dataflow Flex Templates |
| "ML training at scale" | Vertex AI Training + GPUs/TPUs, distributed training |
| "unpredictable traffic" | Cloud Run, Cloud Functions, App Engine auto-scaling |
| "hybrid connectivity" | Cloud Interconnect (high bandwidth) or HA VPN (lower bandwidth) |
| "legacy application" | App Engine Flexible, Compute Engine, or GKE with lift-and-shift containers |
| "multi-cloud" | Anthos, BigQuery Omni, GKE Enterprise multi-cloud |
| "CI/CD pipeline" | Cloud Build + Cloud Deploy + Artifact Registry |
| "secrets management" | Secret Manager (NOT KMS -- KMS is for encryption keys) |

### Pillar Trade-off Matrix

When WAF pillars conflict, use this matrix to determine which takes priority. In general, **Security and Reliability trump Cost and Performance** for enterprise workloads, but the exam question context always determines the final priority.

| Conflict | Resolution | Example |
|----------|-----------|---------|
| **Security vs Cost** | Security wins (almost always) | CMEK adds cost but is required for compliance. Use CMEK. |
| **Security vs Performance** | Security wins (usually) | VPC-SC adds API call latency but prevents data exfiltration. Use VPC-SC. |
| **Reliability vs Cost** | Depends on business criticality | Revenue-generating system → reliability wins (multi-region). Dev/test → cost wins (single zone). |
| **Reliability vs Performance** | Reliability wins (usually) | Synchronous replication adds write latency but ensures zero RPO. Accept the latency. |
| **Performance vs Cost** | Depends on user impact | User-facing service → performance wins (Premium Tier). Internal batch → cost wins (Standard Tier, Spot VMs). |
| **Cost vs Sustainability** | Usually aligned | Both prefer right-sizing, serverless, and autoscaling. Rare conflict. |
| **Performance vs Sustainability** | Performance wins (usually) | Use the nearest region for low latency even if it has a lower CFE score. |
| **Operational Excellence vs Cost** | OpEx usually wins | Managed services cost more but reduce operational burden, leading to fewer outages and faster delivery. |

#### Priority Hierarchy by Scenario Type

```
Mission-Critical / Regulated Workload:
  1. Security & Compliance  (non-negotiable)
  2. Reliability            (SLAs with penalties)
  3. Performance            (user experience)
  4. Operational Excellence (team efficiency)
  5. Cost Optimization      (secondary concern)
  6. Sustainability         (nice-to-have)

Internal Tool / Dev Environment:
  1. Cost Optimization      (minimize spend)
  2. Operational Excellence (developer productivity)
  3. Performance            (acceptable, not critical)
  4. Security               (standard, not elevated)
  5. Reliability            (single zone is fine)
  6. Sustainability         (if it aligns with cost)

User-Facing Web Application:
  1. Performance            (user experience = revenue)
  2. Reliability            (availability = trust)
  3. Security               (data protection)
  4. Cost Optimization      (within budget)
  5. Operational Excellence (team efficiency)
  6. Sustainability         (tiebreaker)

Data Analytics / ML Platform:
  1. Performance            (query speed, training time)
  2. Cost Optimization      (data processing costs scale fast)
  3. Security               (data governance)
  4. Reliability            (reprocessing is possible)
  5. Operational Excellence (pipeline automation)
  6. Sustainability         (batch scheduling flexibility)
```

### Cross-Reference to Exam Sections

| PCA Exam Section | Weight | Primary WAF Pillars | Secondary WAF Pillars |
|-----------------|--------|--------------------|-----------------------|
| **S1: Designing and planning architecture** | ~25% | Reliability, Performance, Cost | Security, OpEx |
| **S2: Managing and provisioning infrastructure** | ~17.5% | OpEx, Security | Performance, Cost |
| **S3: Designing for security and compliance** | ~17.5% | Security (directly) | Reliability, OpEx |
| **S4: Analyzing and optimizing processes** | ~15% | Cost, Performance, OpEx | Reliability |
| **S5: Managing implementation** | ~12.5% | OpEx, Reliability | Security |
| **S6: Ensuring operations excellence** | ~12.5% | OpEx, Reliability (directly) | All pillars |

### Master Exam Strategy: The "ARCHITECT" Mnemonic

When evaluating any PCA question, run through this checklist:

```
A - Availability: What's the required uptime? (SLO, multi-zone, multi-region?)
R - Recovery: What are the RPO/RTO targets? (DR pattern selection)
C - Compliance: Are there regulatory or data residency requirements?
H - Hierarchy: Is the resource hierarchy correct? (Org → Folders → Projects)
I - Identity: Who needs access and with what permissions? (Least privilege)
T - Traffic: Where are users? How much traffic? (Global LB, CDN, autoscaling)
E - Economics: What's the budget? (CUDs, Spot, serverless, right-sizing)
C - Continuity: How do we operate this? (Monitoring, alerting, CI/CD, IaC)
T - Technology: Is this the right service for the job? (Decision tree)
```

---

## Quick Reference: WAF Pillar Summary Table

| Pillar | One-Liner | Key GCP Services | Top Exam Trap |
|--------|-----------|-------------------|---------------|
| **Operational Excellence** | Run it, monitor it, improve it | Cloud Ops Suite, Cloud Deploy, Cloud Build | SLO != SLA; error budgets drive release decisions |
| **Security** | Protect everything, trust nothing | IAM, VPC-SC, KMS, SCC, Binary Auth | VPC-SC != Firewall; Org Policy != IAM |
| **Reliability** | Design for failure at every layer | Global LB, Spanner, GKE regional, Cloud SQL HA | Cloud SQL HA is NOT multi-region; RPO=0 needs sync replication |
| **Cost Optimization** | Pay only for what you need | Recommender, Spot VMs, CUDs, lifecycle policies | Budgets do NOT stop spending; Archive is NOT slow |
| **Performance** | Fast, scalable, efficient | Cloud CDN, Memorystore, Premium Tier, Bigtable | Premium != Standard Tier; Local SSD is ephemeral |
| **Sustainability** | Minimize environmental impact | Carbon Footprint, serverless, Spot VMs | Rarely primary driver; usually a tiebreaker |

---

## Appendix: WAF Review Checklist for Architecture Designs

Use this checklist when reviewing any architecture on the PCA exam or in real-world design reviews:

### Operational Excellence Checklist
- [ ] Are SLOs defined and monitored with error budgets?
- [ ] Is there centralized logging with appropriate retention?
- [ ] Are alerting policies configured for key SLIs?
- [ ] Is infrastructure defined as code (Terraform, Config Connector)?
- [ ] Are deployments automated with CI/CD pipelines?
- [ ] Are rollback procedures defined and tested?
- [ ] Is there a documented incident response process?

### Security Checklist
- [ ] Is the resource hierarchy properly structured (Org → Folders → Projects)?
- [ ] Are Org Policies applied at the appropriate level?
- [ ] Is IAM following least privilege (no `roles/owner` in production)?
- [ ] Are service account keys eliminated (using Workload Identity)?
- [ ] Is data encrypted with appropriate key management (CMEK where required)?
- [ ] Are VPC Service Controls in place for sensitive data?
- [ ] Is network security layered (Firewall, Cloud Armor, Private Google Access)?
- [ ] Are audit logs enabled and exported to a secure, centralized project?

### Reliability Checklist
- [ ] Are single points of failure eliminated at every layer?
- [ ] Are compute resources spread across multiple zones (or regions)?
- [ ] Is there a documented and tested DR plan?
- [ ] Do RPO/RTO targets match the DR implementation?
- [ ] Is autoscaling configured with appropriate headroom?
- [ ] Are health checks configured for all load-balanced backends?
- [ ] Is there capacity for failover (N+1 or N+2 redundancy)?

### Cost Optimization Checklist
- [ ] Are resources right-sized based on actual utilization?
- [ ] Are CUDs in place for predictable workloads?
- [ ] Are Spot VMs used for fault-tolerant workloads?
- [ ] Are storage lifecycle policies configured?
- [ ] Are labels applied for cost allocation and chargeback?
- [ ] Is billing exported to BigQuery for analysis?
- [ ] Are budgets and alerts configured?
- [ ] Is compute co-located with data (same region)?

### Performance Checklist
- [ ] Is the right compute platform selected for the workload type?
- [ ] Are caching layers in place (CDN, Memorystore)?
- [ ] Is the appropriate network tier selected (Premium for user-facing)?
- [ ] Are databases optimized (indexes, partitioning, connection pooling)?
- [ ] Is autoscaling configured for variable workloads?
- [ ] Are GPUs/TPUs considered for ML workloads?
- [ ] Is latency measured end-to-end with acceptable budgets?

### Sustainability Checklist
- [ ] Is the region with the highest CFE score selected (when requirements allow)?
- [ ] Are serverless services preferred to reduce idle resource consumption?
- [ ] Is autoscaling configured to scale to zero where possible?
- [ ] Are storage lifecycle policies removing unnecessary data?
- [ ] Is the Carbon Footprint dashboard being monitored?
