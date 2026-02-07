# GCP Service Decision Trees & Quick Reference Diagrams

Quick visual guides for choosing the right GCP service based on scenario requirements.

---

## 1. Compute Service Selection

```
What are you running?
│
├── Containers?
│   │
│   ├── Need full orchestration (scheduling, service mesh, stateful)?
│   │   └── ✅ GKE Standard
│   │
│   ├── Want managed K8s, no node management?
│   │   └── ✅ GKE Autopilot
│   │
│   ├── Simple stateless container, no cluster?
│   │   └── ✅ Cloud Run
│   │
│   └── Short-lived, event-triggered container?
│       └── ✅ Cloud Run (with Eventarc/Pub/Sub trigger)
│
├── Just code (no container)?
│   │
│   ├── Short-lived, event-triggered (< 60 min)?
│   │   └── ✅ Cloud Functions (2nd gen)
│   │
│   ├── Web app with versions and traffic splitting?
│   │   └── ✅ App Engine
│   │
│   └── Long-running background processing?
│       └── ✅ Cloud Run jobs
│
├── Need full OS control / GPU / custom kernel?
│   │
│   ├── Single VM?
│   │   └── ✅ Compute Engine
│   │
│   ├── Group of identical VMs that autoscale?
│   │   └── ✅ Managed Instance Group (MIG)
│   │
│   └── Fault-tolerant batch job, cost matters most?
│       └── ✅ Compute Engine Spot VMs
│
└── Batch data processing?
    │
    ├── Apache Spark / Hadoop?
    │   └── ✅ Dataproc
    │
    └── Apache Beam (streaming or batch)?
        └── ✅ Dataflow
```

### Compute Quick Reference

| Scenario | Service |
|----------|---------|
| "Minimal admin, Docker container" | Cloud Run |
| "Complex microservices, service mesh" | GKE |
| "Don't want to manage nodes" | GKE Autopilot |
| "Event-driven, small function" | Cloud Functions |
| "Need GPUs or specific OS" | Compute Engine |
| "Identical VMs that autoscale" | MIG |
| "Batch, can tolerate interruption" | Spot VMs |
| "Spark/Hadoop workload" | Dataproc |
| "Stream processing pipeline" | Dataflow |

---

## 2. Database & Storage Selection

```
What kind of data?
│
├── Unstructured (files, images, backups, logs)?
│   │
│   └── ✅ Cloud Storage
│       │
│       ├── Accessed frequently ──────────── Standard
│       ├── Accessed < 1x / month ─────────── Nearline (30-day min)
│       ├── Accessed < 1x / quarter ────────── Coldline (90-day min)
│       └── Accessed < 1x / year ──────────── Archive (365-day min)
│
├── Structured / relational?
│   │
│   ├── Need global scale + strong consistency?
│   │   └── ✅ Cloud Spanner ($$$)
│   │
│   ├── PostgreSQL-compatible, high performance?
│   │   └── ✅ AlloyDB
│   │
│   └── Standard MySQL / PostgreSQL / SQL Server?
│       └── ✅ Cloud SQL
│
├── NoSQL / document?
│   │
│   ├── Mobile/web app, real-time sync, < 10 TB?
│   │   └── ✅ Firestore (Native mode)
│   │
│   └── Need Datastore API compatibility?
│       └── ✅ Firestore (Datastore mode)
│
├── NoSQL / wide-column (IoT, time-series, high throughput)?
│   │
│   └── ✅ Cloud Bigtable
│
├── Analytics / data warehouse (ad hoc SQL on huge datasets)?
│   │
│   └── ✅ BigQuery
│
└── In-memory cache?
    │
    └── ✅ Memorystore (Redis / Memcached)
```

### Database Quick Reference

| Scenario | Service |
|----------|---------|
| "Relational, single region, managed" | Cloud SQL |
| "Relational, global, five 9s" | Cloud Spanner |
| "PostgreSQL but need more performance" | AlloyDB |
| "Document DB, mobile/web sync" | Firestore |
| "IoT sensor data, millions of writes/sec" | Bigtable |
| "Ad hoc SQL analytics, petabyte scale" | BigQuery |
| "Store files, images, backups" | Cloud Storage |
| "Session cache, low latency" | Memorystore |

### Storage Class Decision

```
How often will you access the data?
│
├── Regularly (multiple times/month) ─────── Standard ($0.020/GB)
├── Less than once a month ────────────────── Nearline ($0.010/GB)
├── Less than once a quarter ──────────────── Coldline ($0.004/GB)
└── Less than once a year (compliance) ────── Archive ($0.001/GB)

⚠️  All classes have millisecond access time
⚠️  Early deletion fees apply (30/90/365 days)
⚠️  Retrieval cost increases as class gets colder
```

---

## 3. Load Balancer Selection

```
External or internal traffic?
│
├── External (internet-facing)
│   │
│   ├── HTTP(S) traffic?
│   │   │
│   │   ├── Global (multi-region)?
│   │   │   └── ✅ Global External Application LB (L7, Premium Tier)
│   │   │
│   │   └── Single region?
│   │       └── ✅ Regional External Application LB (L7)
│   │
│   ├── TCP traffic (non-HTTP)?
│   │   │
│   │   ├── Need SSL offload?
│   │   │   └── ✅ External Proxy Network LB (L4, SSL Proxy)
│   │   │
│   │   ├── Need global?
│   │   │   └── ✅ External Proxy Network LB (L4, TCP Proxy)
│   │   │
│   │   └── Need client IP preservation / UDP?
│   │       └── ✅ External Passthrough Network LB (L4)
│   │
│   └── UDP traffic?
│       └── ✅ External Passthrough Network LB (L4)
│
└── Internal (VPC-only)
    │
    ├── HTTP(S) traffic?
    │   └── ✅ Internal Application LB (L7)
    │
    └── TCP/UDP traffic?
        └── ✅ Internal Passthrough Network LB (L4)
```

### Load Balancer Quick Reference

| Scenario | Load Balancer |
|----------|---------------|
| "Website, global users, HTTPS" | Global External Application LB |
| "HTTPS, single region only" | Regional External Application LB |
| "TCP non-HTTP, need global" | External Proxy Network LB |
| "Need client source IP" | External Passthrough Network LB |
| "UDP traffic" | External Passthrough Network LB |
| "Internal microservices, HTTP" | Internal Application LB |
| "Internal TCP/UDP" | Internal Passthrough Network LB |

> **Exam trap:** Global load balancing requires **Premium Network Tier**. Standard Tier only supports regional.

---

## 4. Networking & Connectivity Selection

```
Connect on-premises to GCP?
│
├── Need high bandwidth (10-200 Gbps)?
│   │
│   ├── Can colocate at Google PoP?
│   │   └── ✅ Dedicated Interconnect (10/100 Gbps)
│   │
│   └── Use a service provider?
│       └── ✅ Partner Interconnect (50 Mbps - 50 Gbps)
│
├── Need encrypted tunnel, moderate bandwidth?
│   │
│   ├── Need 99.99% SLA?
│   │   └── ✅ HA VPN (2 tunnels + BGP)
│   │
│   └── Basic connectivity, lower cost?
│       └── ✅ Classic VPN (single tunnel, static routing)
│
└── One-time large data transfer?
    │
    ├── > 20 TB?
    │   └── ✅ Transfer Appliance (ship physical device)
    │
    └── < 20 TB, online transfer from other cloud/URL?
        └── ✅ Storage Transfer Service
```

```
Connect two VPC networks?
│
├── Same organization, need shared networking?
│   └── ✅ Shared VPC (host project + service projects)
│
├── Different orgs or need isolation?
│   └── ✅ VPC Peering
│       ⚠️  Non-transitive (A↔B and B↔C does NOT mean A↔C)
│       ⚠️  No overlapping CIDR ranges
│
└── VMs need internet access but no external IP?
    └── ✅ Cloud NAT (outbound only, with Cloud Router)
```

### Connectivity Quick Reference

| Scenario | Service |
|----------|---------|
| "Highest bandwidth, dedicated line" | Dedicated Interconnect |
| "High bandwidth via provider" | Partner Interconnect |
| "Encrypted tunnel, 99.99% SLA" | HA VPN |
| "Ship 100 TB to GCP" | Transfer Appliance |
| "Teams share one VPC, central control" | Shared VPC |
| "Two VPCs need to talk, different orgs" | VPC Peering |
| "Outbound internet, no public IPs" | Cloud NAT |
| "Private access to Google APIs" | Private Google Access |

> **Exam trap:** VPC Peering is **non-transitive**. Shared VPC is for **centralized network admin** within an org.

---

## 5. IAM & Security Decision Tree

```
Who/what needs access?
│
├── Human user?
│   │
│   ├── Single person?
│   │   └── Assign predefined role to user (use group if possible)
│   │
│   └── Team of people?
│       └── ✅ Create Google Group → assign predefined role to group
│
├── Application running on GCP?
│   │
│   ├── On Compute Engine?
│   │   └── ✅ Attach service account to VM
│   │
│   ├── On GKE?
│   │   └── ✅ Workload Identity (K8s SA → GCP SA)
│   │
│   ├── On Cloud Run / Cloud Functions?
│   │   └── ✅ Set service account at deploy time (--service-account)
│   │
│   └── Need to act as a service account temporarily?
│       └── ✅ Impersonation (--impersonate-service-account)
│
└── Application running outside GCP (AWS, GitHub, on-prem)?
    │
    └── ✅ Workload Identity Federation (no keys!)
```

```
Which role type?
│
├── "Viewer", "Editor", "Owner"?
│   └── ⚠️  Basic roles — too broad for production, avoid
│
├── Google-maintained, scoped to a service?
│   └── ✅ Predefined roles — use these (e.g., roles/storage.objectViewer)
│
└── Need custom combination of permissions?
    └── ✅ Custom roles — you maintain them, use only when no predefined fits
```

### Key IAM Roles to Know

| Role | What it does | Watch out |
|------|-------------|-----------|
| `roles/viewer` | Read all resources | Too broad for production |
| `roles/editor` | Read + write all resources | Too broad, **default SA gets this** |
| `roles/owner` | Full control + IAM | Never grant widely |
| `roles/iam.serviceAccountUser` | Attach SA to resources | Needed to deploy with a SA |
| `roles/iam.serviceAccountTokenCreator` | Generate tokens / impersonate | For impersonation |
| `roles/bigquery.dataViewer` | Read BQ data | Common exam answer for auditors |
| `roles/compute.networkAdmin` | Manage networking | No VM access |
| `roles/container.admin` | Full GKE control | Includes node management |

> **Exam trap:** Default service accounts get `roles/editor`. Always replace with minimum-privilege SA.

---

## 6. Monitoring & Logging Decision Tree

```
What do you need?
│
├── Alert when something goes wrong?
│   │
│   ├── Metric-based (CPU, memory, latency)?
│   │   └── ✅ Cloud Monitoring alerting policy
│   │
│   ├── Alert on log patterns (errors, keywords)?
│   │   └── ✅ Log-based metric → alerting policy
│   │
│   └── Track website/endpoint availability?
│       └── ✅ Uptime check
│
├── Export logs?
│   │
│   ├── Long-term storage, cheapest?
│   │   └── ✅ Log sink → Cloud Storage
│   │
│   ├── Query and analyze logs with SQL?
│   │   └── ✅ Log sink → BigQuery
│   │
│   ├── Stream to external system?
│   │   └── ✅ Log sink → Pub/Sub
│   │
│   └── Filter what gets stored?
│       └── ✅ Exclusion filters on _Default sink
│
├── Collect OS-level metrics (memory, disk)?
│   └── ✅ Install Ops Agent on VM
│
├── Prometheus metrics from GKE?
│   └── ✅ Managed Prometheus (PodMonitoring CRD)
│
└── Track who did what (audit)?
    │
    ├── Admin actions (always on, free)?
    │   └── ✅ Admin Activity audit logs
    │
    ├── Data access (who read/wrote data)?
    │   └── ✅ Data Access audit logs (must enable, costs $)
    │
    └── Policy denied actions?
        └── ✅ Policy Denied audit logs (always on)
```

### Logging Quick Reference

| Log type | Always on? | Cost | Retention |
|----------|-----------|------|-----------|
| Admin Activity | Yes | Free | 400 days |
| Data Access | **No, must enable** | Paid | 30 days default |
| System Event | Yes | Free | 400 days |
| Policy Denied | Yes | Free | 400 days |
| _Default bucket | Yes | Paid after 30 days | 30 days default |
| _Required bucket | Yes | Free | 400 days, cannot change |

> **Exam trap:** Data Access logs must be **explicitly enabled** and they generate volume. Admin Activity logs are **always on and free**.

---

## 7. IaC & Deployment Decision Tree

```
How to manage infrastructure?
│
├── Declarative, multi-cloud, industry standard?
│   └── ✅ Terraform
│       ├── Google best practices baked in? → Cloud Foundation Toolkit (CFT)
│       └── Store state remotely? → GCS backend
│
├── Kubernetes-native, manage GCP from K8s?
│   └── ✅ Config Connector (KCC)
│
├── Kubernetes app packaging (charts)?
│   └── ✅ Helm
│
└── Quick one-off, imperative?
    └── gcloud CLI (not recommended for repeatable infra)
```

---

## 8. Data Movement Decision Tree

```
Move data into GCP?
│
├── Initial migration, > 20 TB, limited bandwidth?
│   └── ✅ Transfer Appliance
│
├── From another cloud (AWS S3, Azure)?
│   └── ✅ Storage Transfer Service
│
├── From on-prem, recurring schedule?
│   └── ✅ Storage Transfer Service (with agent)
│
├── Small files, one-time?
│   └── ✅ gcloud storage cp / gsutil cp
│
└── Load into database?
    │
    ├── Into BigQuery?
    │   ├── From GCS → bq load
    │   ├── Streaming → BigQuery Storage Write API
    │   └── From other sources → Dataflow
    │
    ├── Into Cloud SQL?
    │   └── gcloud sql import csv/sql
    │
    └── Into Firestore / Bigtable?
        └── gcloud firestore import / cbt commands
```

---

## 9. Master Scenario Quick Reference

One table to rule them all -- common exam scenarios mapped to the right answer.

| Scenario | Answer |
|----------|--------|
| "Cheapest compute for interruptible batch" | Spot VMs |
| "Docker, auto-scale, no cluster" | Cloud Run |
| "Global relational DB, 99.999%" | Cloud Spanner |
| "IoT, millions of writes, simple schema" | Bigtable |
| "Ad hoc SQL, pay per query" | BigQuery |
| "Auditors need read-only BigQuery" | Group + `roles/bigquery.dataViewer` |
| "Don't change DNS if server dies" | Reserve static external IP |
| "Prevent overspending" | Quotas (not budgets!) |
| "Get notified about spending" | Budgets + alerts |
| "Consistent infra across dev/test/prod" | CFT + Terraform |
| "Connect on-prem, cheapest" | Cloud VPN |
| "Connect on-prem, highest bandwidth" | Dedicated Interconnect |
| "100 TB one-time migration" | Transfer Appliance |
| "Process uploaded files automatically" | Cloud Storage + Cloud Function |
| "Distribute work to many consumers" | Pub/Sub + MIG |
| "VMs need internet, no public IP" | Cloud NAT |
| "Centralized network for multiple teams" | Shared VPC |
| "Two VPCs need to communicate" | VPC Peering |
| "Labels vs tags" | Labels = billing. Tags = firewall |
| "Expand subnet, out of IPs" | `expand-ip-range` (cannot shrink) |
| "Frontend ↔ backend in GKE, survive restarts" | Kubernetes Service |
| "Stable pods communication in K8s" | ClusterIP Service |
| "Estimate costs quickly" | Pricing Calculator |
| "Microservices, variable load, minimal admin" | Cloud Run |
| "Monitor Storage + Firestore changes" | Cloud Functions event triggers |
| "Track who did what" | Audit logs (Admin Activity) |
| "External app needs GCP access, no keys" | Workload Identity Federation |
| "Default SA is dangerous because..." | It has `roles/editor` |
