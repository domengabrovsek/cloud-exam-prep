# Domain Topic Reference

Quick lookup: which file and section covers a given topic.

## GCP Associate Cloud Engineer (`gcp/ace/`)

### Domain 1 -- `gcp/ace/01-cloud-environment-setup.md`

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

### Domain 2 -- `gcp/ace/02-planning-and-configuring.md`

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

### Domain 3 -- `gcp/ace/03-deploying-and-implementing.md`

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

### Domain 4 -- `gcp/ace/04-operations.md`

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

### Domain 5 -- `gcp/ace/05-access-and-security.md`

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

### CLI Cheat Sheet -- `gcp/ace/06-key-gcloud-commands.md`

Covers commands for: config, compute, container, run, functions, iam, projects, organizations, resource-manager, services, storage/gsutil, bq, kubectl, logging, monitoring, pubsub, sql, dns, billing, terraform.

### Practice Questions -- `gcp/ace/07-practice-questions.md`

60 questions: D1 (Q1-12), D2 (Q13-22), D3 (Q23-37), D4 (Q38-49), D5 (Q50-60).
