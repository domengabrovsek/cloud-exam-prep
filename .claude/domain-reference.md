# Domain Topic Reference

Quick lookup: which file and section covers a given topic.

## GCP Associate Cloud Engineer (`gcp/ace/`)

### Domain 1 -- `gcp/ace/docs/01-cloud-environment-setup.md`

| Topic | Section |
|-------|---------|
| Resource hierarchy (org/folders/projects) | 1.1 |
| Organization policies / constraints | 1.1 |
| IAM role assignment | 1.1 |
| Cloud Identity, GCDS, SAML SSO | 1.1 |
| Enabling APIs | 1.1 |
| Operations suite setup (Monitoring, Logging) | 1.1 |
| Quotas (rate, allocation, requesting increases) | 1.1 |
| Billing accounts (self-serve vs invoiced) | 1.2 |
| Linking projects to billing | 1.2 |
| Budgets and alerts (don't stop spending!) | 1.2 |
| Billing exports to BigQuery | 1.2 |
| Labels for cost attribution | 1.2 |

### Domain 2 -- `gcp/ace/docs/02-planning-and-configuring.md`

| Topic | Section |
|-------|---------|
| Compute service selection (CE vs GKE vs Run vs Functions) | 2.1 |
| Compute decision tree | 2.1 |
| Spot VMs (60-91% discount, preemption) | 2.1 |
| Custom machine types | 2.1 |
| Database selection (SQL vs Spanner vs Firestore vs Bigtable vs BQ) | 2.2 |
| Cloud Storage classes (Standard/Nearline/Coldline/Archive) | 2.2 |
| Persistent Disk types (zonal vs regional) | 2.2 |
| Load balancer types and selection | 2.3 |
| Network Service Tiers (Premium vs Standard) | 2.3 |
| Resource location/scope (global/regional/zonal) | 2.3 |

### Domain 3 -- `gcp/ace/docs/03-deploying-and-implementing.md`

| Topic | Section |
|-------|---------|
| Compute Engine VMs (create, disks, SSH keys, OS Login) | 3.1 |
| Managed instance groups (MIGs) and autoscaling | 3.1 |
| VM Manager (OS patch, config, inventory) | 3.1 |
| GKE clusters (Standard, Autopilot, private, regional) | 3.2 |
| kubectl and cluster credentials | 3.2 |
| GKE Enterprise | 3.2 |
| Cloud Run deployment and traffic splitting | 3.3 |
| Eventarc triggers | 3.3 |
| Cloud Functions (2nd gen) | 3.3 |
| Cloud Run vs Functions vs Cloud Run for Anthos | 3.3 |
| Cloud SQL, AlloyDB, Firestore, BigQuery, Spanner | 3.4 |
| Pub/Sub, Dataflow, Cloud Storage | 3.4 |
| Data loading methods (CLI, GCS import, Storage Transfer) | 3.4 |
| VPC and subnets (custom mode) | 3.5 |
| Shared VPC | 3.5 |
| Firewall rules (priority, tags, service accounts) | 3.5 |
| HA VPN with BGP | 3.5 |
| VPC Peering (non-transitive) | 3.5 |
| Terraform (init/plan/apply/destroy) | 3.6 |
| Cloud Foundation Toolkit | 3.6 |
| Config Connector | 3.6 |
| Helm charts | 3.6 |

### Domain 4 -- `gcp/ace/docs/04-operations.md`

| Topic | Section |
|-------|---------|
| SSH, RDP, serial console, IAP tunneling | 4.1 |
| VM inventory and management | 4.1 |
| Snapshots (create, schedule, incremental) | 4.1 |
| Custom images and image families | 4.1 |
| GKE cluster inventory | 4.2 |
| Artifact Registry | 4.2 |
| Node pools (add, resize, upgrade) | 4.2 |
| Pods, Deployments, Services, StatefulSets | 4.2 |
| HPA, VPA, Cluster Autoscaler | 4.2 |
| Cloud Run revisions and traffic splitting | 4.3 |
| Cloud Run scaling (min/max instances) | 4.3 |
| Cloud Storage lifecycle management | 4.4 |
| Signed URLs, uniform bucket-level access | 4.4 |
| Query commands (Cloud SQL, BQ, Spanner, Firestore) | 4.4 |
| Cost estimation (BQ dry run) | 4.4 |
| Backup/restore (Cloud SQL PITR, Firestore) | 4.4 |
| Job status (BigQuery, Dataflow) | 4.4 |
| Subnet management (expand CIDR, never shrink) | 4.5 |
| Static IP addresses (external, internal) | 4.5 |
| Cloud DNS (zones, records, transactions) | 4.5 |
| Cloud NAT (outbound only, with Cloud Router) | 4.5 |
| Cloud Monitoring alerts and custom metrics | 4.6 |
| Log-based metrics | 4.6 |
| Log sinks and exports (BQ, GCS, Pub/Sub) | 4.6 |
| Log buckets (_Required vs _Default) | 4.6 |
| Log Analytics | 4.6 |
| Ops Agent | 4.6 |
| Managed Prometheus | 4.6 |
| Audit logs (4 types, always-on vs configurable) | 4.6 |

### Domain 5 -- `gcp/ace/docs/05-access-and-security.md`

| Topic | Section |
|-------|---------|
| IAM policies (view, create, manage) | 5.1 |
| Role types (basic, predefined, custom) | 5.1 |
| IAM conditions (CEL expressions) | 5.1 |
| IAM deny policies (evaluated before allow) | 5.1 |
| Policy Troubleshooter | 5.1 |
| Service account creation and management | 5.2 |
| Minimum permissions / IAM Recommender | 5.2 |
| Assigning SAs to resources (CE, Run, Functions, GKE) | 5.2 |
| serviceAccountUser vs serviceAccountTokenCreator | 5.2 |
| Service account impersonation | 5.2 |
| Short-lived credentials (access tokens vs ID tokens) | 5.2 |
| Default service accounts (Editor role risk) | 5.2 |
| Service account keys (why to avoid) | 5.2 |
| Workload Identity Federation (GitHub, AWS, Azure) | 5.2 |
| Org policy constraints for IAM | 5.2 |

### CLI Cheat Sheet -- `gcp/ace/docs/06-key-gcloud-commands.md`

Covers commands for: config, compute, container, run, functions, iam, projects, organizations, resource-manager, services, storage/gsutil, bq, kubectl, logging, monitoring, pubsub, sql, dns, billing, terraform.

### Decision Trees -- `gcp/ace/docs/09-decision-trees.md`

Service selection decision trees for: compute, database/storage, load balancers, networking, IAM, monitoring/logging, IaC, data movement. Includes master scenario quick-reference table.

### Practice Questions -- `gcp/ace/questions/`

140 questions organized by exam section:
- `section-1-cloud-environment-setup.md` (25 questions)
- `section-2-planning-and-implementing.md` (58 questions)
- `section-3-operations.md` (34 questions)
- `section-4-access-and-security.md` (23 questions)
- `google-official-sample.md` (20 questions)

## GCP Professional Cloud Architect (`gcp/pca/`)

### Section 1 -- `gcp/pca/docs/01-designing-planning-architecture.md`

| Topic | Section |
|-------|---------|
| Business requirements → architecture mapping | 1.1 |
| Functional / non-functional requirements (SLAs, latency, durability) | 1.1 |
| Business continuity plan, BCP vs DR | 1.1 |
| Cost optimization (CUDs, SUDs, Spot VMs, right-sizing, Active Assist) | 1.1 |
| Microservices, 12-factor app, event-driven architecture | 1.1 |
| Integration patterns (Pub/Sub, Cloud Tasks, Workflows, Apigee) | 1.1 |
| Data movement (Storage Transfer, Transfer Appliance, DMS, Datastream) | 1.1 |
| Design decision trade-offs | 1.1 |
| Workload disposition (6 R's: rehost, replatform, refactor...) | 1.1 |
| Success measurements (KPIs, DORA metrics, SRE) | 1.1 |
| Observability design (four golden signals) | 1.1 |
| WAF alignment in architecture design | 1.2 |
| High availability / failover (regional, multi-regional) | 1.2 |
| Scalability (HPA, VPA, MIG autoscaler, Spanner scaling, Cloud Run) | 1.2 |
| Performance / latency (CDN, Premium Tier, Memorystore) | 1.2 |
| Gemini Cloud Assist | 1.2 |
| Backup and recovery (RPO/RTO, DR patterns: cold/warm/hot/active-active) | 1.2 |
| Hybrid / multicloud networking (VPN, Interconnect, PSC, GKE Enterprise) | 1.3 |
| AI/ML solutions (Vertex AI, Gemini, Agent Builder, Model Garden, AI Hypercomputer) | 1.3 |
| VPC design (Shared VPC, peering, hub-and-spoke) | 1.3 |
| Load balancer selection | 1.3 |
| Data processing (Dataflow, Dataproc, BigQuery, Pub/Sub) | 1.3 |
| Database selection (Cloud SQL, AlloyDB, Spanner, Firestore, Bigtable, BigQuery) | 1.3 |
| Storage selection (GCS classes, Filestore, Persistent Disk) | 1.3 |
| Compute selection (CE, GKE, Cloud Run, Batch, GPUs/TPUs) | 1.3 |
| Migration planning (6 R's, Migration Center, DMS, strangler fig) | 1.4 |
| Software licensing (BYOL, sole-tenant nodes, TCO) | 1.4 |
| Cloud-first design, future improvements | 1.5 |

### Section 2 -- `gcp/pca/docs/02-managing-provisioning-infrastructure.md`

| Topic | Section |
|-------|---------|
| Hybrid networking (HA VPN, Dedicated/Partner Interconnect, Cross-Cloud) | 2.1 |
| Cloud Armor, Cloud IDS, hierarchical firewalls | 2.1 |
| VPC design, load balancing, DNS architecture | 2.1 |
| Cloud Storage (classes, lifecycle, Autoclass, dual-region, Turbo Replication) | 2.2 |
| Data retention, Bucket Lock, WORM compliance | 2.2 |
| Database scaling (Spanner PUs, BigQuery slots, Bigtable nodes) | 2.2 |
| Data protection per service (backups, PITR, replicas) | 2.2 |
| Machine families, MIGs, sole-tenant nodes | 2.3 |
| Spot VMs, compute volatility | 2.3 |
| GKE Standard vs Autopilot, GKE Enterprise (fleet, Config Sync) | 2.3 |
| Cloud Run, Cloud Run functions, Eventarc, serverless patterns | 2.3 |
| VM Manager, patch management, MIG update policies | 2.3 |
| Vertex AI Pipelines, Feature Store, training workflows | 2.4 |
| AI Hypercomputer, GPUs vs TPUs | 2.4 |
| Model serving (online, batch, serverless endpoints) | 2.4 |
| Pre-built AI APIs (Vision, Video, NLP, Speech, Document AI) | 2.5 |
| Gemini Enterprise, AI Agents, Agent Builder | 2.5 |
| Model Garden (open-source models, fine-tuning) | 2.5 |

### Section 3 -- `gcp/pca/docs/03-security-and-compliance.md`

| Topic | Section |
|-------|---------|
| IAM at scale (groups, conditions, deny policies, Recommender, custom roles) | 3.1 |
| Resource hierarchy for security (folders, environment isolation) | 3.1 |
| Encryption (CMEK, CSEK, Cloud EKM, Cloud HSM) | 3.1 |
| Secret Manager | 3.1 |
| Sensitive Data Protection / DLP | 3.1 |
| Separation of duties | 3.1 |
| VPC Service Controls (perimeters, access levels, ingress/egress, dry-run) | 3.1 |
| Organization policies as security guardrails | 3.1 |
| Hierarchical firewall policies | 3.1 |
| Security Command Center (Standard vs Premium) | 3.1 |
| CMEK integration with services (Cloud SQL, GCS, BigQuery, GKE, etc.) | 3.1 |
| IAP, service account impersonation, Workload Identity Federation | 3.1 |
| Chrome Enterprise Premium / BeyondCorp | 3.1 |
| OS Login | 3.1 |
| Binary Authorization, Artifact Registry, SLSA, Software Delivery Shield | 3.1 |
| Model Armor, AI security | 3.1 |
| HIPAA compliance (BAA, Assured Workloads) | 3.2 |
| GDPR (data residency, crypto-shredding, DPA) | 3.2 |
| COPPA, data sovereignty | 3.2 |
| PCI DSS | 3.2 |
| PII handling, data classification | 3.2 |
| SOC, ISO, FedRAMP certifications | 3.2 |
| Cloud Audit Logs (4 types), Access Transparency, Access Approval | 3.2 |
| Assured Workloads (compliance-bound folders) | 3.2 |

### Section 4 -- `gcp/pca/docs/04-optimizing-processes.md`

| Topic | Section |
|-------|---------|
| CI/CD (Cloud Build, Artifact Registry, Cloud Deploy) | 4.1 |
| Deployment strategies (rolling, blue-green, canary) | 4.1 |
| Testing (load, unit, integration, chaos engineering) | 4.1 |
| Disaster recovery patterns and testing | 4.1 |
| Troubleshooting / root cause analysis | 4.1 |
| FinOps, cost optimization, CUDs/SUDs, right-sizing | 4.2 |
| Stakeholder management, change management | 4.2 |
| Business continuity planning | 4.2 |

### Section 5 -- `gcp/pca/docs/05-managing-implementations.md`

| Topic | Section |
|-------|---------|
| API management (Apigee, API Gateway, Cloud Endpoints) | 5.1 |
| Migration tooling (DMS, Migrate to VMs/Containers, Migration Center) | 5.1 |
| Gemini Cloud Assist | 5.1 |
| Terraform (provider, state, modules, CFT, best practices) | 5.2 |
| Cloud emulators (Pub/Sub, Spanner, Bigtable, Firestore) | 5.2 |
| Google Cloud SDKs (gcloud, gsutil, bq, kubectl) | 5.2 |
| Config Connector, Deployment Manager | 5.2 |

### Section 6 -- `gcp/pca/docs/06-solution-operations-excellence.md`

| Topic | Section |
|-------|---------|
| WAF operational excellence pillar | 6.1 |
| Cloud Monitoring (metrics, dashboards, alerting, Prometheus) | 6.2 |
| Cloud Logging (sinks, buckets, Log Analytics, log-based metrics) | 6.2 |
| Cloud Trace, Cloud Profiler, Error Reporting | 6.2 |
| SLO-based burn rate alerting | 6.2 |
| Cloud Deploy, canary, traffic splitting, rollbacks | 6.3 |
| Incident response, runbooks, post-mortems | 6.4 |
| SLOs/SLIs/SLAs, error budgets, capacity planning | 6.5 |
| Chaos engineering, load testing, penetration testing | 6.6 |
| GKE upgrades, Cloud SQL maintenance, rolling updates | 6.6 |

### Cross-cutting References

| Topic | File |
|-------|------|
| Well-Architected Framework (6 pillars) | `gcp/pca/docs/07-well-architected-framework.md` |
| Case studies (EHR Healthcare, Cymbal Retail, Altostrat Media, KnightMotives) | `gcp/pca/docs/08-case-studies.md` |
| Service selection decision trees | `gcp/pca/docs/09-decision-trees.md` |
| gcloud commands, Terraform, kubectl, emulators | `gcp/pca/docs/10-key-commands-and-terraform.md` |
