# GCP Professional Cloud Architect -- Decision Trees & Quick Reference

Architect-level decision trees for the PCA exam. Each tree focuses on trade-offs,
constraints, and the "why" behind service selection -- not just basic feature matching.

---

## 1. Compute Service Selection

```
What is the workload?
│
├── Containers / microservices?
│   │
│   ├── Need full K8s control (node pools, GPUs, DaemonSets, custom schedulers)?
│   │   └── ✅ GKE Standard
│   │       ├── Stateful workloads (StatefulSets, PVCs)? → GKE Standard
│   │       ├── Multi-cluster mesh / Anthos? → GKE Enterprise (fleet)
│   │       └── Windows containers? → GKE Standard (Windows node pools)
│   │
│   ├── K8s but no node ops (pay-per-pod, auto-secured)?
│   │   └── ✅ GKE Autopilot
│   │       ⚠️  No SSH to nodes, no privileged pods, no DaemonSets
│   │       ⚠️  Best for teams that want K8s API without infra burden
│   │
│   ├── Single container, request-driven, no cluster overhead?
│   │   └── ✅ Cloud Run
│   │       ├── Scale to zero needed? → Cloud Run (default)
│   │       ├── Min instances for latency? → Cloud Run (min-instances > 0)
│   │       ├── Long-running async work? → Cloud Run jobs
│   │       └── GPU inference serving? → Cloud Run (GPU, L4)
│   │
│   └── Event-driven, lightweight, single-purpose function?
│       └── ✅ Cloud Run functions (2nd gen)
│           ⚠️  Built on Cloud Run under the hood
│           ⚠️  Max 60 min timeout, 32 GB RAM
│
├── Need full VM control (custom kernel, license, GPU, HPC)?
│   │
│   ├── Single machine or bespoke config?
│   │   └── ✅ Compute Engine
│   │       ├── GPU / TPU attached? → Compute Engine (A2/A3/G2 machine types)
│   │       ├── SAP / Oracle / bare-metal OS? → Compute Engine (sole-tenant nodes)
│   │       └── Confidential computing? → Compute Engine (Confidential VMs)
│   │
│   ├── Auto-healing, auto-scaling group of identical VMs?
│   │   └── ✅ Managed Instance Group (MIG)
│   │       ├── Stateless (web tier)? → Stateless MIG + autoscaler
│   │       ├── Stateful (each VM has unique disk)? → Stateful MIG
│   │       └── Can tolerate preemption? → MIG with Spot VMs
│   │
│   └── HPC / tightly coupled parallel compute?
│       └── ✅ Compute Engine + Compact placement policy
│           (or Batch for job scheduling)
│
├── Batch / job-oriented processing?
│   │
│   ├── Arbitrary containers or scripts, need scheduling + retries?
│   │   └── ✅ Cloud Batch
│   │       ⚠️  Provisions VMs automatically, supports GPU, Spot
│   │
│   ├── Apache Spark / Hadoop / Hive / Presto ecosystem?
│   │   └── ✅ Dataproc
│   │       ├── Short-lived cluster per job? → Dataproc on Compute Engine
│   │       ├── Shared multi-tenant? → Dataproc on GKE
│   │       └── Serverless, no cluster management? → Dataproc Serverless
│   │
│   └── Apache Beam (unified batch + stream)?
│       └── ✅ Dataflow
│           ├── Streaming with exactly-once? → Dataflow streaming
│           └── Batch ETL? → Dataflow batch
│
└── Web application, managed platform, version/traffic control?
    └── ✅ App Engine
        ├── Supported runtime, auto-scaled? → App Engine Standard
        ├── Custom runtime, Docker, longer requests? → App Engine Flexible
        ⚠️  App Engine is legacy-leaning; prefer Cloud Run for new apps
        ⚠️  One App Engine app per project (cannot delete, only disable)
```

### Compute Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "Microservices, team owns K8s, needs node tuning" | GKE Standard | Full control over node pools, taints, GPUs |
| "K8s workloads, platform team wants guardrails" | GKE Autopilot | Google manages nodes, enforces best practices |
| "Stateless HTTP service, scale to zero" | Cloud Run | No cluster, pay-per-request, fastest time-to-deploy |
| "GPU inference serving, serverless" | Cloud Run (GPU) | L4 GPU, scale to zero, no K8s overhead |
| "Event-driven glue code (< 60 min)" | Cloud Run functions | Simplest unit of deployment for triggers |
| "SAP HANA, Oracle DB, BYOL" | Sole-tenant nodes | Dedicated physical servers, license compliance |
| "HPC, MPI, tightly coupled" | CE + compact placement | Low-latency node-to-node networking |
| "1000 rendering jobs, tolerate preemption" | Cloud Batch + Spot | Automatic VM provisioning, retry on preempt |
| "Migrate Spark pipelines from on-prem" | Dataproc | Drop-in Hadoop/Spark, ephemeral clusters |
| "Real-time stream + batch in one pipeline" | Dataflow | Apache Beam, auto-scaling, exactly-once |
| "Legacy App Engine app, minimal changes" | App Engine Flexible | Docker-based, compatible with legacy config |

**Exam Tips:**
- GKE Autopilot vs Standard is a frequent question -- Autopilot = no node management, no SSH, no DaemonSets, Pod Security enforced.
- Cloud Run functions (2nd gen) is built on Cloud Run. If the question says "serverless function," either answer may apply -- look for "no infrastructure" = Cloud Run functions, "container" = Cloud Run.
- Dataproc is the answer whenever you see Spark, Hadoop, Hive, or Presto. Dataflow is for Beam.
- Cloud Batch is the answer for "schedule thousands of containerized jobs with retries" (not Dataflow, not Dataproc).
- App Engine: one per project, cannot delete. If the question involves multiple apps in one project, that is a trap.

---

## 2. Database Selection

```
What is the data model and scale?
│
├── Relational (SQL, ACID, joins)?
│   │
│   ├── Global scale, 99.999% SLA, unlimited horizontal scaling?
│   │   └── ✅ Cloud Spanner
│   │       ├── Multi-region for HA? → Multi-region Spanner config
│   │       ├── Need PostgreSQL interface? → Spanner PostgreSQL dialect
│   │       └── Financial / inventory (strong global consistency)? → Spanner
│   │       ⚠️  Starts at ~$0.90/node/hr (~$650/mo min). Cost-justified at scale.
│   │
│   ├── PostgreSQL-compatible, need 4x Cloud SQL performance?
│   │   └── ✅ AlloyDB
│   │       ├── HTAP (transactions + analytics on same data)? → AlloyDB
│   │       ├── AI/ML embeddings, vector search? → AlloyDB AI
│   │       └── Cross-region read replicas? → AlloyDB
│   │       ⚠️  PostgreSQL only. No MySQL, no SQL Server.
│   │
│   ├── Standard managed RDBMS (MySQL, PostgreSQL, SQL Server)?
│   │   └── ✅ Cloud SQL
│   │       ├── High availability? → Regional Cloud SQL (auto-failover)
│   │       ├── Read scaling? → Cloud SQL read replicas
│   │       ├── Cross-region DR? → Cross-region read replica, promote on failover
│   │       └── > 64 TB or need horizontal writes? → Consider Spanner or AlloyDB
│   │       ⚠️  Max 64 TB storage. Vertical scaling only for writes.
│   │
│   └── Oracle / SQL Server with full feature parity, bare-metal?
│       └── ✅ Bare Metal Solution
│           ⚠️  For Oracle RAC, SAP HANA, workloads that cannot be refactored
│
├── NoSQL -- Document / key-value?
│   │
│   ├── Mobile/web, real-time sync, offline support, < 10 TB?
│   │   └── ✅ Firestore (Native mode)
│   │       ├── Strong consistency, real-time listeners? → Native mode
│   │       └── Need Datastore API backward compatibility? → Datastore mode
│   │
│   └── Large-scale server-side only, no real-time sync needed?
│       └── ✅ Firestore (Datastore mode)
│           ⚠️  Cannot switch modes after creation
│
├── NoSQL -- Wide-column (time-series, IoT, high throughput)?
│   │
│   └── ✅ Cloud Bigtable
│       ├── Millions of rows/sec, single-digit ms latency? → Bigtable
│       ├── Time-series, IoT sensor data, financial ticks? → Bigtable
│       ├── Serving layer for ML feature store? → Bigtable
│       └── HBase migration? → Bigtable (HBase-compatible API)
│       ⚠️  No multi-row transactions. No SQL (use Bigtable client or cbt).
│       ⚠️  Min 1 node (~$0.65/hr). Not cost-effective for small datasets.
│
├── Analytics / data warehouse?
│   │
│   └── ✅ BigQuery
│       ├── Petabyte-scale SQL analytics? → BigQuery
│       ├── Real-time analytics? → BigQuery streaming inserts + materialized views
│       ├── ML on warehouse data? → BigQuery ML (BQML)
│       ├── Cross-cloud analytics? → BigQuery Omni (on AWS/Azure data)
│       ├── Data sharing without copying? → Analytics Hub
│       └── BI/reporting layer? → BigQuery + Looker / Looker Studio
│       ⚠️  Not for OLTP. Not a transactional database.
│
└── In-memory cache / session store?
    │
    └── ✅ Memorystore
        ├── Key-value cache, pub/sub, Lua scripting? → Memorystore for Redis
        ├── Simple object caching, multi-threaded? → Memorystore for Memcached
        └── Redis Cluster mode (> 300 GB, sharding)? → Memorystore for Redis Cluster
```

### Database Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "Global transactions, financial ledger" | Cloud Spanner | Strongly consistent, multi-region, unlimited scale |
| "PostgreSQL, need analytics + OLTP together" | AlloyDB | Columnar engine for analytics on OLTP data |
| "Standard MySQL, < 64 TB, lift-and-shift" | Cloud SQL | Managed, familiar, lowest barrier |
| "Oracle RAC, cannot refactor" | Bare Metal Solution | Bare metal in GCP colos, full Oracle support |
| "Mobile app, offline sync, real-time" | Firestore (Native) | Client SDKs, real-time listeners, offline cache |
| "IoT, 1M writes/sec, time-series" | Bigtable | Wide-column, single-digit ms, HBase API |
| "Petabyte analytics, ad hoc SQL" | BigQuery | Serverless warehouse, pay-per-query option |
| "ML training on warehouse data" | BigQuery ML | Train models with SQL, no data movement |
| "Session cache, sub-ms reads" | Memorystore (Redis) | In-memory, managed, HA available |
| "Cross-cloud query (S3 data)" | BigQuery Omni | Run BQ on AWS/Azure storage in-place |

**Exam Tips:**
- Spanner vs Cloud SQL: if the question mentions "global," "horizontal scaling for writes," or "five 9s," it is Spanner. If "single region" and "MySQL/PostgreSQL," it is Cloud SQL.
- AlloyDB appears when the question says "PostgreSQL" + "analytics on operational data" or "4x performance."
- Bigtable is never the answer for "complex queries" or "joins." It is for single-key or range-scan workloads at massive scale.
- BigQuery is not OLTP. If the scenario requires low-latency transactional reads/writes, BigQuery is the wrong answer.
- Bare Metal Solution: if you see "Oracle RAC" or "cannot modify application," this is likely the answer.
- Firestore: cannot change mode (Native vs Datastore) after database creation.

---

## 3. Storage Selection

```
What type of storage?
│
├── Object storage (files, images, backups, data lake)?
│   │
│   └── ✅ Cloud Storage
│       │
│       ├── Know the access pattern?
│       │   ├── Frequent access ────────────────── Standard ($0.020/GB)
│       │   ├── < 1x / month ──────────────────── Nearline (30-day min, $0.010/GB)
│       │   ├── < 1x / quarter ────────────────── Coldline (90-day min, $0.004/GB)
│       │   └── < 1x / year (compliance/audit) ── Archive (365-day min, $0.001/GB)
│       │
│       ├── Access pattern unpredictable or mixed?
│       │   └── ✅ Autoclass
│       │       ⚠️  Automatically transitions objects between classes
│       │       ⚠️  No early deletion fees when Autoclass moves objects
│       │       ⚠️  Best for data lakes with mixed hot/cold data
│       │
│       ├── Compliance: immutable objects, legal hold?
│       │   └── ✅ Bucket Lock (retention policy) + Object holds
│       │
│       ├── Cross-region redundancy?
│       │   ├── Dual-region bucket → 2 specific regions, turbo replication (< 15 min RPO)
│       │   └── Multi-region bucket → automatic geo-redundancy (US, EU, ASIA)
│       │
│       └── Signed URLs for temporary access?
│           └── ✅ Signed URLs / Signed Policy Documents
│
├── File storage (NFS, shared across VMs)?
│   │
│   ├── General-purpose NFS (GKE, Compute Engine)?
│   │   └── ✅ Filestore
│   │       ├── Basic (HDD) → cost-effective, 1-63 TB
│   │       ├── Zonal (SSD) → performance, 1-100 TB
│   │       ├── Regional (SSD) → HA, multi-zone, 1-100 TB
│   │       └── Enterprise → mission-critical, regional, 1-10 TB
│   │
│   └── HPC, AI/ML training, ultra-high throughput?
│       └── ✅ Parallelstore
│           ⚠️  Lustre-based, > 100 GB/s throughput
│           ⚠️  For A3 GPU VMs, tightly coupled HPC
│           ⚠️  Integrates with Cloud Storage for data staging
│
└── Block storage (VM disks)?
    │
    └── ✅ Persistent Disk / Hyperdisk
        │
        ├── Standard workloads (boot disks, dev/test)?
        │   └── pd-balanced (SSD, default) or pd-standard (HDD, cheap)
        │
        ├── High IOPS databases, latency-sensitive?
        │   └── pd-ssd (provisioned IOPS) or Hyperdisk Extreme
        │
        ├── High throughput, sequential reads (analytics)?
        │   └── Hyperdisk Throughput
        │
        ├── Need disk shared across multiple VMs (read-only)?
        │   └── pd-ssd or pd-balanced in multi-writer / read-only mode
        │
        ├── Ephemeral, fastest possible (scratch space)?
        │   └── Local SSD (375 GB per disk, data lost on stop/preempt)
        │
        └── Snapshots for backup / DR?
            ├── Same-region backup → Standard snapshot
            ├── Cross-region DR → Cross-region snapshot schedule
            └── Application-consistent? → VSS (Windows) or pre/post scripts
```

### Storage Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "Data lake, mixed access patterns" | Cloud Storage + Autoclass | Auto-tiering eliminates manual lifecycle rules |
| "Compliance, immutable audit logs" | Cloud Storage + Bucket Lock | Retention policy prevents deletion |
| "Low RPO replication between 2 regions" | Dual-region bucket + turbo replication | < 15 min RPO, cheaper than multi-region |
| "Shared filesystem for GKE pods" | Filestore (ReadWriteMany PVC) | NFS, multi-pod concurrent access |
| "AI training, 100+ GB/s throughput" | Parallelstore | Lustre-based, designed for GPU clusters |
| "Database disk, high IOPS" | Hyperdisk Extreme or pd-ssd | Configurable IOPS independent of disk size |
| "Scratch space, fastest, ephemeral" | Local SSD | Attached directly to host, data not persistent |
| "Cross-region disk DR" | Scheduled PD snapshots (cross-region) | Automated, incremental, cross-region copies |
| "Archive for 7-year retention" | Archive class + retention policy | Cheapest storage, legal compliance |
| "Temporary download link for external user" | Signed URL | Time-limited, no Google account required |

**Exam Tips:**
- All Cloud Storage classes have **millisecond** first-byte latency. "Archive is slow" is always a wrong answer.
- Autoclass is the PCA-level answer when the question says "unpredictable access patterns" or "optimize cost automatically."
- Turbo replication (dual-region) gives < 15 min RPO. Regular dual/multi-region is "best effort" replication.
- Parallelstore is new and appears in questions about HPC/AI training with extreme throughput requirements.
- Local SSDs are ephemeral -- data is lost on stop, preempt, or live migration. Never for persistent data.
- Hyperdisk lets you provision IOPS and throughput independently of disk size -- this is the architect-level answer for "need more IOPS without increasing disk size."

---

## 4. Network Connectivity

```
Connect on-premises / other cloud to GCP?
│
├── Need highest bandwidth (10-200 Gbps), dedicated physical line?
│   │
│   ├── Can colocate at a Google edge PoP?
│   │   └── ✅ Dedicated Interconnect
│   │       ├── 10 Gbps or 100 Gbps circuits
│   │       ├── 99.99% SLA requires 4 connections across 2 metro areas
│   │       ├── 99.9% SLA requires 2 connections in 1 metro area
│   │       └── Not encrypted by default (add MACsec or VPN overlay)
│   │       ⚠️  8-12 week lead time for provisioning
│   │
│   └── Cannot reach Google PoP directly?
│       └── ✅ Partner Interconnect
│           ├── 50 Mbps to 50 Gbps (via service provider)
│           ├── L2 or L3 connectivity depending on provider
│           └── 99.99% SLA requires redundant connections
│           ⚠️  Lower cost than Dedicated, but shared infrastructure
│
├── Need encrypted tunnel, moderate bandwidth (up to 3 Gbps/tunnel)?
│   │
│   └── ✅ HA VPN
│       ├── 99.99% SLA (requires 2 tunnels + BGP)
│       ├── IPsec encrypted by default
│       ├── Quick to provision (hours, not weeks)
│       ├── Can run over internet or over Interconnect (for encryption)
│       └── Max 3 Gbps per tunnel (aggregate with multiple tunnels)
│       ⚠️  Classic VPN is deprecated for new deployments
│
├── Connect to another cloud provider (AWS, Azure, Oracle)?
│   │
│   ├── Google-managed, dedicated cross-cloud link?
│   │   └── ✅ Cross-Cloud Interconnect
│   │       ├── 10 Gbps or 100 Gbps
│   │       ├── Supported: AWS, Azure, Oracle Cloud, others
│   │       └── Google provisions and manages the link end-to-end
│   │       ⚠️  Not encrypted by default (add HA VPN overlay for encryption)
│   │
│   └── Using public IPs, need Google network backbone?
│       └── ✅ Direct Peering
│           ├── Free, but no SLA
│           ├── Public IP access to Google services
│           └── Not a private connection
│           ⚠️  Direct Peering is for Google public services, not VPC
│
└── Private access to Google APIs (no internet traversal)?
    │
    ├── From VPC (VMs without external IPs)?
    │   └── ✅ Private Google Access (subnet setting)
    │
    ├── From on-prem over Interconnect/VPN?
    │   └── ✅ Private Google Access for on-premises hosts
    │       (uses restricted.googleapis.com or private.googleapis.com)
    │
    └── Private endpoint inside VPC for a Google service?
        └── ✅ Private Service Connect (PSC)
            ├── Consumer-side endpoint with internal IP
            └── Works for Google APIs and published services
```

### Connectivity Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "100 Gbps, lowest latency, can colocate" | Dedicated Interconnect | Physical cross-connect at Google PoP |
| "Need 1 Gbps, no Google PoP nearby" | Partner Interconnect | Provider extends reach |
| "Encrypted tunnel, fast setup, < 10 Gbps" | HA VPN | IPsec, 99.99% SLA, quick provisioning |
| "Encrypt traffic over Interconnect" | HA VPN over Interconnect | VPN tunnels ride the Interconnect link |
| "Connect GCP to AWS VPC directly" | Cross-Cloud Interconnect | Google-managed, dedicated cross-cloud |
| "Multi-cloud, need encryption" | Cross-Cloud Interconnect + HA VPN overlay | CCI for bandwidth, VPN for encryption |
| "Private API access, no external IPs" | Private Google Access | Subnet-level setting, no extra cost |
| "On-prem accesses Google APIs privately" | Private Google Access for on-prem | restricted.googleapis.com over VPN/Interconnect |
| "Internal IP endpoint for Google service" | Private Service Connect | Consumer endpoint, no VPC peering needed |
| "99.99% Interconnect SLA" | 4 circuits, 2 metro areas | Redundancy across metros required |

**Exam Tips:**
- Dedicated Interconnect is NOT encrypted by default. If the question asks for encryption + high bandwidth, the answer is HA VPN over Interconnect (or MACsec).
- Cross-Cloud Interconnect is the PCA-level answer for "connect GCP to AWS/Azure with dedicated bandwidth." Do not confuse with Partner Interconnect (that is for on-prem).
- 99.99% SLA for Dedicated Interconnect requires connections in 2 different metro areas (not just 2 connections in 1 metro).
- Direct Peering is rarely the right answer on the PCA exam. It is for public Google service access, not private VPC connectivity.
- Private Service Connect (PSC) is the modern answer for "access Google services with an internal IP." It replaces Private Google Access in many scenarios.

---

## 5. Load Balancer Selection

```
What traffic and where?
│
├── External (internet-facing)?
│   │
│   ├── HTTP(S) traffic (L7)?
│   │   │
│   │   ├── Global, multi-region backends, CDN, WAF?
│   │   │   └── ✅ Global External Application LB
│   │   │       ├── Requires Premium Network Tier
│   │   │       ├── Supports Cloud CDN, Cloud Armor (WAF/DDoS)
│   │   │       ├── URL map for path-based routing
│   │   │       ├── SSL termination at edge
│   │   │       └── Backend types: MIGs, NEGs, GKE, Cloud Run, GCS buckets
│   │   │
│   │   ├── Regional, single-region only?
│   │   │   └── ✅ Regional External Application LB
│   │   │       ├── Works with Standard Network Tier (cheaper)
│   │   │       ├── Envoy-based proxy
│   │   │       └── Supports advanced traffic management (header-based routing)
│   │   │
│   │   └── Classic mode (legacy)?
│   │       └── ⚠️ Classic Application LB (being replaced, avoid for new designs)
│   │
│   ├── TCP/UDP traffic (L4)?
│   │   │
│   │   ├── Need proxy, SSL offload, or global reach?
│   │   │   │
│   │   │   ├── SSL offload?
│   │   │   │   └── ✅ External Proxy Network LB (SSL Proxy mode)
│   │   │   │
│   │   │   └── TCP proxy, global?
│   │   │       └── ✅ External Proxy Network LB (TCP Proxy mode)
│   │   │           ⚠️  Client IP is NOT preserved (seen as proxy IP)
│   │   │
│   │   └── Need client IP preservation, UDP, or highest performance?
│   │       └── ✅ External Passthrough Network LB
│   │           ├── Regional only (no global)
│   │           ├── Preserves client source IP
│   │           ├── Supports UDP, TCP, ESP, ICMP
│   │           └── Used for: NLB, gaming, VoIP, DNS
│   │
│   └── Need DDoS + WAF?
│       └── Cloud Armor attaches to:
│           ├── Global External Application LB ✅
│           ├── External Proxy Network LB ✅
│           └── External Passthrough Network LB (advanced DDoS only)
│
└── Internal (VPC only)?
    │
    ├── HTTP(S) traffic, L7 features (routing, header manipulation)?
    │   │
    │   ├── Regional?
    │   │   └── ✅ Regional Internal Application LB
    │   │       ├── Envoy-based proxy
    │   │       ├── Supports URL map, TLS, traffic splitting
    │   │       └── Ideal for service mesh / internal microservices
    │   │
    │   └── Cross-region (global)?
    │       └── ✅ Cross-Region Internal Application LB
    │           ├── Multi-region internal services
    │           └── Global anycast VIP inside VPC
    │
    └── TCP/UDP traffic, L4?
        │
        ├── Regional?
        │   └── ✅ Regional Internal Passthrough Network LB
        │       ├── Preserves client IP
        │       ├── Used for internal TCP/UDP services
        │       └── Next-hop for internal routing (e.g., NVAs)
        │
        └── Cross-region (global)?
            └── ✅ Cross-Region Internal Proxy Network LB
                ⚠️  Proxy mode -- does NOT preserve client IP
```

### Load Balancer Quick Reference

| Scenario | Load Balancer | Architect Rationale |
|----------|---------------|---------------------|
| "Website, global CDN, WAF protection" | Global External Application LB + Cloud Armor + CDN | L7, edge termination, full security stack |
| "HTTPS, single region, advanced routing" | Regional External Application LB | Envoy-based, header routing, Standard Tier OK |
| "Gaming server, preserve client IP, UDP" | External Passthrough Network LB | L4, regional, no proxy, preserves source IP |
| "SSL offload for non-HTTP TCP" | External Proxy Network LB (SSL Proxy) | Terminates SSL at Google's edge |
| "Internal microservices, gRPC routing" | Regional Internal Application LB | L7, Envoy, path/header routing |
| "Internal DB failover, TCP" | Regional Internal Passthrough Network LB | L4, preserves source, used as failover next-hop |
| "Multi-region internal service" | Cross-Region Internal Application LB | Global VIP for internal cross-region routing |
| "Cloud Run as backend" | Global External Application LB + serverless NEG | Serverless NEG points to Cloud Run service |
| "Route to NVA (firewall appliance)" | Internal Passthrough NLB as next-hop | Custom route → ILB → NVA instance |

**Exam Tips:**
- Global load balancing requires **Premium Network Tier**. If the question mentions cost optimization with Standard Tier, the answer is regional.
- Cloud Armor (WAF + DDoS) only attaches to Application LBs and Proxy Network LBs -- not to passthrough LBs (except basic DDoS).
- "Preserve client source IP" = Passthrough (not proxy). Proxy LBs replace the source IP.
- For GKE Ingress, the GKE Ingress controller creates a Global External Application LB by default. For internal, use the `kubernetes.io/ingress.class: gce-internal` annotation.
- Serverless NEG is how you put Cloud Run, Cloud Functions, or App Engine behind a global LB.

---

## 6. Security Architecture

```
What security control is needed?
│
├── Identity & access control?
│   │
│   ├── Who can do what on which resource?
│   │   └── ✅ IAM
│   │       ├── Predefined roles (preferred) → least privilege
│   │       ├── Custom roles → when no predefined role fits exactly
│   │       ├── IAM Conditions → time-based, resource-based restrictions
│   │       ├── IAM Deny policies → explicit deny, overrides allow
│   │       └── Organization Policy → org-wide guardrails (constraints)
│   │
│   ├── Cross-organization or external identity?
│   │   └── ✅ Workload Identity Federation
│   │       ├── AWS, Azure, GitHub, OIDC, SAML providers
│   │       └── No service account keys -- ever
│   │
│   ├── GKE workloads accessing Google APIs?
│   │   └── ✅ Workload Identity (K8s SA → GCP SA)
│   │
│   └── Privileged Access Management?
│       └── ✅ PAM (time-bound elevated access, approval workflows)
│
├── Network security perimeter?
│   │
│   ├── Prevent data exfiltration from Google services?
│   │   └── ✅ VPC Service Controls (VPC-SC)
│   │       ├── Service perimeter around projects
│   │       ├── Controls: BigQuery, GCS, Spanner, etc.
│   │       ├── Access levels (IP, identity, device) for exceptions
│   │       ├── Ingress/egress rules for controlled cross-perimeter access
│   │       └── Dry-run mode for testing before enforcement
│   │       ⚠️  Does NOT replace IAM. Works alongside IAM.
│   │
│   ├── DDoS protection + WAF (L7)?
│   │   └── ✅ Cloud Armor
│   │       ├── Preconfigured WAF rules (OWASP Top 10)
│   │       ├── Custom rules (IP, geo, header-based)
│   │       ├── Rate limiting
│   │       ├── Adaptive Protection (ML-based anomaly detection)
│   │       └── Bot management
│   │
│   ├── Zero-trust access to web apps (no VPN)?
│   │   └── ✅ Identity-Aware Proxy (IAP)
│   │       ├── Authenticates users via Google identity
│   │       ├── Works with App Engine, GKE, Compute Engine
│   │       └── No VPN needed for remote access
│   │
│   └── Firewall rules?
│       ├── VPC Firewall Rules → basic allow/deny by IP, protocol, port
│       ├── Firewall Policies → hierarchical (org → folder → project)
│       └── Network tags vs service accounts → tags for firewall targeting
│
├── Encryption & key management?
│   │
│   ├── Google-managed keys (default)?
│   │   └── ✅ Google Default Encryption (AES-256, automatic)
│   │
│   ├── Customer controls key rotation, but Google holds key?
│   │   └── ✅ Cloud KMS (CMEK -- Customer-Managed Encryption Keys)
│   │       ├── Software keys, HSM-backed keys, or external keys
│   │       ├── Automatic rotation schedules
│   │       └── IAM controls who can use/manage keys
│   │
│   ├── Customer holds key outside Google entirely?
│   │   └── ✅ Cloud EKM (External Key Manager) -- CSEK or EKM
│   │       ├── Key never leaves customer's external KMS
│   │       └── Key Access Justifications (see why Google needs key)
│   │
│   └── Confidential computing (encrypt data in use)?
│       └── ✅ Confidential VMs / Confidential GKE Nodes
│           ├── AMD SEV / Intel TDX hardware encryption
│           └── Memory encrypted at hardware level
│
└── Supply chain & workload integrity?
    │
    ├── Only deploy trusted container images?
    │   └── ✅ Binary Authorization
    │       ├── Policy: require attestations before deploy
    │       ├── Integrates with Artifact Registry vulnerability scanning
    │       └── Enforced at GKE admission controller
    │
    ├── Vulnerability scanning for containers?
    │   └── ✅ Artifact Registry + Artifact Analysis (Container Scanning)
    │
    └── Software supply chain security?
        └── ✅ Software Delivery Shield (end-to-end framework)
            ├── Cloud Build → provenance attestation
            ├── Artifact Registry → vulnerability scanning
            └── Binary Authorization → deploy-time enforcement
```

### Security Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "Prevent BigQuery data exfiltration" | VPC Service Controls | Perimeter blocks data copy to external projects |
| "External contractor needs time-limited admin" | PAM or IAM Conditions (time-based) | Temporary elevated access with auto-revoke |
| "OWASP Top 10, L7 WAF" | Cloud Armor | Preconfigured rules, attaches to ALB |
| "Remote employees access internal app, no VPN" | Identity-Aware Proxy (IAP) | BeyondCorp-style zero-trust access |
| "Regulatory: customer must control encryption keys" | Cloud KMS (CMEK) | Customer manages rotation, Google stores key |
| "Key must never leave customer premises" | Cloud EKM (external) | Key in external KMS, GCP calls out to use it |
| "Only signed containers in production GKE" | Binary Authorization | Attestation-based admission control |
| "GitHub Actions deploys to GKE, no keys" | Workload Identity Federation | OIDC token exchange, no JSON keys |
| "Encrypt data in memory (regulatory)" | Confidential VMs | Hardware-level memory encryption |
| "Org-wide: no external IPs on VMs" | Organization Policy constraint | `constraints/compute.vmExternalIpAccess` |
| "Hierarchical firewall across all projects" | Firewall Policies (org/folder level) | Centralized security team controls |

**Exam Tips:**
- VPC-SC is the #1 PCA security topic. It prevents data exfiltration, NOT unauthorized access (that is IAM). They complement each other.
- VPC-SC dry-run mode: always recommend testing with dry-run before enforcing. The exam tests this.
- CMEK vs CSEK vs EKM: CMEK = Google stores key in Cloud KMS, you manage it. CSEK = you supply raw key with each API call. EKM = key stays in your external KMS.
- Binary Authorization: if the question mentions "only allow vetted images," "container signing," or "attestation," this is the answer.
- IAM Deny policies: these are evaluated before allow policies. Use them for hard guardrails that cannot be overridden by lower-level allows.
- IAP replaces VPN for BeyondCorp / zero-trust access. If the question says "no VPN," think IAP.

---

## 7. AI/ML Service Selection

```
What is the ML need?
│
├── Custom model training (own data, own architecture)?
│   │
│   ├── Full control (custom TensorFlow, PyTorch, JAX)?
│   │   └── ✅ Vertex AI Custom Training
│   │       ├── Custom containers or pre-built training containers
│   │       ├── GPU/TPU training (A100, H100, TPU v5)
│   │       ├── Hyperparameter tuning (Vertex AI Vizier)
│   │       ├── Distributed training (multi-node, multi-GPU)
│   │       └── Vertex AI Pipelines for orchestration (Kubeflow / TFX)
│   │
│   ├── Tabular/image/text/video, limited ML expertise?
│   │   └── ✅ Vertex AI AutoML
│   │       ├── No-code / low-code model training
│   │       ├── Tabular, image classification, object detection, NLP, video
│   │       └── Automatic feature engineering, architecture search
│   │       ⚠️  Less control, but faster time-to-value
│   │
│   └── Foundation model fine-tuning (LLMs)?
│       └── ✅ Vertex AI Model Garden + Tuning
│           ├── Adapter tuning (LoRA), distillation
│           ├── Gemini, PaLM 2, Llama, Mistral, etc.
│           └── RLHF for alignment
│
├── Use pre-trained models / APIs (no training)?
│   │
│   ├── Common vision / language / speech tasks?
│   │   └── ✅ Pre-built AI APIs
│   │       ├── Vision AI → image labeling, OCR, face detection
│   │       ├── Natural Language AI → sentiment, entity, syntax
│   │       ├── Speech-to-Text / Text-to-Speech → transcription, synthesis
│   │       ├── Translation AI → 100+ languages
│   │       ├── Document AI → structured extraction from documents
│   │       └── Video AI → label detection, shot detection, object tracking
│   │
│   ├── Generative AI (text, code, multimodal)?
│   │   └── ✅ Vertex AI Gemini API
│   │       ├── Gemini models (text, vision, code)
│   │       ├── Grounding with Google Search or custom data
│   │       └── Function calling for tool use
│   │
│   └── Open-source or third-party models?
│       └── ✅ Vertex AI Model Garden
│           ├── Deploy OSS models (Llama, Mistral, Stable Diffusion)
│           ├── One-click deploy to Vertex AI endpoints
│           └── Compare model cards and benchmarks
│
├── Build AI-powered applications / agents?
│   │
│   ├── Search over enterprise data (RAG)?
│   │   └── ✅ Vertex AI Agent Builder (Search)
│   │       ├── Ingest: websites, GCS, BigQuery, unstructured docs
│   │       ├── Auto-chunking, embeddings, vector retrieval
│   │       └── Grounded answers with citations
│   │
│   ├── Conversational agent / chatbot?
│   │   └── ✅ Vertex AI Agent Builder (Agent)
│   │       ├── Dialogflow CX for structured flows
│   │       ├── Agent Builder for LLM-native agents
│   │       └── Tool use, grounding, function calling
│   │
│   └── Recommendations?
│       └── ✅ Recommendations AI / Vertex AI Search (retail)
│
└── ML ops / model management?
    │
    ├── Model registry, versioning, deployment?
    │   └── ✅ Vertex AI Model Registry + Endpoints
    │
    ├── Feature engineering and serving?
    │   └── ✅ Vertex AI Feature Store
    │       ├── Online serving (low latency) + offline (batch)
    │       └── Bigtable-backed for online serving
    │
    ├── Pipeline orchestration?
    │   └── ✅ Vertex AI Pipelines (Kubeflow-based)
    │
    └── Monitoring model drift?
        └── ✅ Vertex AI Model Monitoring
            ├── Training-serving skew detection
            └── Feature drift alerts
```

### AI/ML Quick Reference

| Scenario | Service | Architect Rationale |
|----------|---------|---------------------|
| "Train custom PyTorch on GPUs" | Vertex AI Custom Training | Full control, custom containers |
| "Classify images, no ML team" | Vertex AI AutoML (Image) | No-code, automatic architecture search |
| "Fine-tune Gemini on company data" | Model Garden + Vertex AI Tuning | Adapter tuning, keeps base model frozen |
| "Extract text from invoices" | Document AI | Pre-trained document parsers + custom |
| "Enterprise search over internal docs" | Vertex AI Agent Builder (Search) | RAG, grounded answers, citations |
| "Customer support chatbot" | Vertex AI Agent Builder (Agent) | LLM-native agents with tool use |
| "Feature store for real-time serving" | Vertex AI Feature Store | Bigtable-backed, online + offline |
| "Serve OSS Llama model" | Model Garden → Vertex AI Endpoint | One-click deploy, managed infrastructure |
| "Detect model accuracy degradation" | Vertex AI Model Monitoring | Training-serving skew, drift alerts |
| "ML pipeline, retrain weekly" | Vertex AI Pipelines | Kubeflow-based, scheduled runs |

**Exam Tips:**
- AutoML vs Custom Training: if the question says "limited ML expertise" or "fastest path," it is AutoML. If "custom architecture" or "PyTorch/TF," it is Custom Training.
- Pre-built APIs are the answer when the question mentions standard tasks (OCR, sentiment, speech) and "no training."
- Agent Builder (formerly Gen App Builder) is the PCA answer for enterprise search / RAG. Do not confuse with Dialogflow CX (structured conversation flows).
- Model Garden is for deploying open-source or Google models. Vertex AI Endpoints is where they run.
- Feature Store: if the question mentions "consistent features between training and serving" or "low-latency feature lookup," this is the answer.

---

## 8. DR Strategy Selection

```
What are the RPO / RTO requirements?
│
├── RPO: hours-days, RTO: hours-days, lowest cost?
│   └── ✅ Cold DR
│       ├── Backup data to GCS (cross-region)
│       ├── Terraform / IaC stored and tested
│       ├── No running infrastructure in DR region
│       ├── Recovery: deploy infra from IaC, restore from backups
│       │
│       │   RPO: 24 hours (depends on backup frequency)
│       │   RTO: hours to days (full rebuild required)
│       │   Cost: $ (storage only)
│       │
│       ⚠️  Must regularly test recovery runbooks
│
├── RPO: minutes-hours, RTO: minutes-hours, moderate cost?
│   └── ✅ Warm DR
│       ├── Scaled-down infrastructure running in DR region
│       ├── Data replication: async DB replicas, GCS replication
│       ├── Recovery: scale up instances, promote DB replicas
│       │
│       │   RPO: minutes to hours (async replication lag)
│       │   RTO: minutes to hours (scale up + DNS switch)
│       │   Cost: $$ (minimal compute + replication)
│       │
│       ├── Cloud SQL: cross-region read replica, promote on failover
│       ├── Spanner: multi-region config (built-in)
│       ├── GCS: dual-region or multi-region
│       └── GKE: standby cluster, scaled to min
│
├── RPO: near-zero, RTO: minutes, higher cost?
│   └── ✅ Hot DR
│       ├── Full infrastructure running in DR region
│       ├── Data replication: sync or near-sync
│       ├── Recovery: DNS failover (manual or automated)
│       │
│       │   RPO: seconds to minutes (sync/near-sync replication)
│       │   RTO: minutes (DNS switch, health check failover)
│       │   Cost: $$$ (full duplicate infrastructure)
│       │
│       ├── Cloud SQL: HA config (auto-failover within region)
│       ├── Spanner: multi-region (automatic)
│       ├── GCS: turbo replication (< 15 min RPO)
│       └── GCLB: health checks auto-route away from failed region
│
└── RPO: zero, RTO: zero, highest cost?
    └── ✅ Active-Active (Multi-Region)
        ├── Both regions serve traffic simultaneously
        ├── Data: multi-region database (Spanner, Firestore)
        ├── Traffic: Global LB distributes across regions
        ├── No failover -- both are primary
        │
        │   RPO: 0 (synchronous replication)
        │   RTO: 0 (no failover needed)
        │   Cost: $$$$ (full capacity in all regions)
        │
        ├── Spanner: multi-region config (nam-eur, nam3, etc.)
        ├── Firestore: multi-region (nam5, eur3)
        ├── Cloud Run: deploy to multiple regions, global LB
        └── GKE: multi-cluster, fleet-based
```

### DR Strategy Matrix

| Strategy | RPO | RTO | Relative Cost | When to Use |
|----------|-----|-----|---------------|-------------|
| Cold | 24h+ | hours-days | $ | Dev/test, non-critical internal apps |
| Warm | min-hours | min-hours | $$ | Business apps with moderate SLAs |
| Hot | seconds-min | minutes | $$$ | Customer-facing, e-commerce |
| Active-Active | 0 | 0 | $$$$ | Mission-critical, financial, global apps |

### DR Quick Reference

| Scenario | Strategy | Key Services |
|----------|----------|-------------|
| "Internal HR app, 24h RPO acceptable" | Cold | GCS backups + Terraform |
| "E-commerce, 1-hour RTO" | Warm | Cross-region SQL replica + scaled-down GKE |
| "Banking portal, minutes RTO" | Hot | Multi-region Spanner + global LB + hot standby |
| "Global SaaS, zero downtime" | Active-Active | Spanner multi-region + Cloud Run multi-region + GCLB |
| "Minimize DR cost, test quarterly" | Cold | Backups + IaC + documented runbooks |
| "Meet 99.99% SLA" | Hot or Active-Active | Full redundancy, automated failover |

**Exam Tips:**
- PCA exam loves RPO/RTO questions. Map requirements to the strategy: zero RPO = active-active or synchronous replication, hours RPO = async + backups.
- Spanner multi-region gives you active-active out of the box. If the question mentions "global consistency + zero RPO," Spanner is the database answer.
- Cloud SQL HA is intra-region only (automatic failover to standby zone). Cross-region DR requires manually promoting a cross-region read replica.
- Turbo replication on dual-region GCS buckets gives < 15 min RPO. Without turbo, replication is "best effort."
- The PCA exam often asks you to balance cost vs RTO/RPO. Cold is cheapest but slowest. Active-active is fastest but most expensive. A good architect picks the cheapest option that meets the SLA.

---

## 9. Migration Strategy (6 R's)

```
What should you do with this application?
│
├── Application has no business value or is redundant?
│   └── ✅ Retire
│       └── Decommission. Save cost immediately.
│
├── Cannot migrate now (compliance, dependency, technical debt)?
│   └── ✅ Retain (Revisit)
│       └── Keep on-premises. Reassess later.
│
├── Replace with SaaS (e.g., move from on-prem Exchange to Gmail)?
│   └── ✅ Repurchase
│       └── Buy cloud-native equivalent. Fastest but may need data migration.
│
├── Migrate with minimal changes?
│   │
│   ├── Lift-and-shift (VM → VM, no code changes)?
│   │   └── ✅ Rehost
│   │       ├── Migrate for Compute Engine (M4CE) → automated VM migration
│   │       ├── Bare Metal Solution → Oracle, SAP (no changes)
│   │       ├── Fastest migration path
│   │       └── Does NOT leverage cloud-native benefits
│   │       ⚠️  Good starting point, but plan a follow-up modernization
│   │
│   └── Move + optimize (change platform, not code)?
│       └── ✅ Replatform
│           ├── MySQL on VM → Cloud SQL (managed, same engine)
│           ├── Hadoop on-prem → Dataproc (managed Hadoop)
│           ├── Redis on VM → Memorystore (managed Redis)
│           ├── Containers on VM → GKE or Cloud Run
│           └── Moderate effort, good cloud-native gains
│
└── Rewrite / redesign for cloud-native?
    └── ✅ Refactor (Re-architect)
        ├── Monolith → microservices
        ├── On-prem DB → Spanner / Firestore
        ├── Batch → event-driven (Pub/Sub + Cloud Run)
        ├── Highest effort, highest long-term benefit
        └── Use when app is strategic and long-lived
```

### Migration Decision Flow (Architect Perspective)

```
Step 1: Assess portfolio
│
├── Categorize every app: strategic, functional, or legacy
├── Identify dependencies (app-to-app, app-to-DB)
└── Build a migration wave plan (move together what is coupled)

Step 2: Choose strategy per app
│
├── Legacy / no value → Retire
├── Has SaaS replacement → Repurchase
├── Strategic, long-lived → Refactor (plan time + budget)
├── Functional, needs cloud benefits → Replatform
├── Time-critical, complex dependencies → Rehost first, modernize later
└── Cannot move yet → Retain

Step 3: Execute
│
├── Foundation: landing zone (org, folders, VPC, IAM, logging)
├── Networking: Interconnect / VPN to on-prem
├── Data: Database Migration Service, Storage Transfer Service
├── Compute: Migrate for Compute Engine, containerization
└── Validate: parallel run, cut-over, decommission source
```

### Migration Quick Reference

| Scenario | Strategy | Key Services |
|----------|----------|-------------|
| "Move 200 VMs fast, no code changes" | Rehost | Migrate for Compute Engine (M4CE) |
| "MySQL on VM → managed MySQL" | Replatform | Database Migration Service → Cloud SQL |
| "On-prem Hadoop → cloud" | Replatform | Dataproc (managed Spark/Hadoop) |
| "Monolith → microservices on K8s" | Refactor | GKE, Cloud Run, Pub/Sub |
| "Replace Exchange with Gmail" | Repurchase | Google Workspace |
| "Oracle RAC, cannot change anything" | Rehost | Bare Metal Solution |
| "App unused for 2 years" | Retire | Decommission |
| "Mainframe, not ready to migrate" | Retain | Keep on-prem, reassess |
| "Migrate SQL Server to Cloud SQL" | Replatform | Database Migration Service |
| "PostgreSQL → Spanner for global scale" | Refactor | Rewrite schema + app for Spanner |

**Exam Tips:**
- The PCA exam tests the 6 R's extensively. The key is matching the scenario constraints (time, budget, skill, business criticality) to the right strategy.
- "Minimal changes" + "fastest migration" = Rehost. "Some changes to platform" = Replatform. "Redesign for cloud" = Refactor.
- Migrate for Compute Engine (M4CE) is the go-to tool for VM rehosting. Database Migration Service (DMS) is for database replatforming.
- Rehost is often a stepping stone. The architect answer may be "rehost now, refactor later" (two-phase migration).
- Landing zone first: the exam expects you to set up org hierarchy, IAM, networking, and logging BEFORE migrating workloads.
- Wave planning: migrate tightly coupled apps together. Do not break dependencies across migration waves.

---

## 10. IaC Tool Selection

```
How to manage GCP infrastructure as code?
│
├── Multi-cloud or GCP-only, industry-standard IaC?
│   │
│   └── ✅ Terraform (with Google Provider)
│       ├── Declarative HCL, state management, plan/apply cycle
│       ├── Google best practices? → Cloud Foundation Toolkit (CFT) modules
│       ├── Remote state? → GCS backend (with state locking)
│       ├── Policy-as-code? → Sentinel (Terraform Cloud) or OPA
│       ├── Automated pipelines? → Cloud Build + Terraform
│       └── Multi-cloud (GCP + AWS + Azure)? → Terraform is the standard
│       ⚠️  Google's recommended IaC tool for most use cases
│
├── Kubernetes-native, manage GCP resources from K8s?
│   │
│   └── ✅ Config Connector (KCC)
│       ├── GCP resources as K8s custom resources (CRDs)
│       ├── Apply with kubectl, GitOps with Config Sync
│       ├── Best for: teams already deep in K8s who want one control plane
│       └── Reconciliation loop (controller continuously enforces state)
│       ⚠️  Only GCP resources. Not multi-cloud.
│
├── Google-native, YAML-based, no external tooling?
│   │
│   └── ⚠️  Deployment Manager (DEPRECATED for new projects)
│       ├── Legacy GCP-native IaC
│       ├── Jinja2 / Python templates
│       └── Migrate to Terraform or KCC
│       ⚠️  Only valid answer if question says "existing Deployment Manager"
│
├── Multi-cloud, prefer general-purpose language (Python, Go, TS)?
│   │
│   └── ✅ Pulumi
│       ├── IaC in real programming languages
│       ├── Alternative to Terraform for teams preferring code over HCL
│       └── Less common on PCA exam, but valid for multi-cloud
│
└── GitOps for K8s cluster config?
    │
    └── ✅ Config Sync (Anthos / GKE Enterprise)
        ├── Git repo → automatically synced to K8s clusters
        ├── Policy Controller (OPA Gatekeeper) for guardrails
        └── Fleet-wide config management across clusters
```

### IaC Quick Reference

| Scenario | Tool | Architect Rationale |
|----------|------|---------------------|
| "Multi-cloud IaC, industry standard" | Terraform | Broadest provider support, largest community |
| "GCP landing zone, best practices" | Terraform + CFT | Google-maintained modules for org setup |
| "Team uses K8s for everything, wants one API" | Config Connector (KCC) | GCP resources as K8s CRDs, kubectl workflow |
| "GitOps for GKE fleet config" | Config Sync | Git-driven cluster configuration |
| "Enforce policies across Terraform" | OPA / Sentinel | Policy-as-code, pre-apply validation |
| "Existing Deployment Manager templates" | Migrate to Terraform | DM is deprecated, use DM Converter tool |
| "IaC in Python, no HCL" | Pulumi | General-purpose languages for IaC |
| "Remote Terraform state, team collaboration" | GCS backend + state locking | Prevent concurrent state corruption |
| "Drift detection on GCP resources" | Config Connector (reconciliation) | Controller continuously enforces desired state |

**Exam Tips:**
- Terraform is almost always the correct IaC answer on the PCA exam unless the question specifically mentions Kubernetes-native management (Config Connector) or existing Deployment Manager.
- Deployment Manager is deprecated for new projects. If the exam mentions it, the answer is usually "migrate to Terraform."
- Cloud Foundation Toolkit (CFT) is the architect-level Terraform answer for "set up a GCP organization following best practices."
- Config Sync + Policy Controller is the answer for "enforce consistent policies across a fleet of GKE clusters."
- State management: Terraform state in GCS with locking. Never store state locally in production.

---

## Master Scenario Quick-Reference Table

Comprehensive mapping of common PCA exam scenarios to the recommended service or approach.

| # | Scenario | Recommended Service / Approach |
|---|----------|-------------------------------|
| 1 | "Global relational DB, 99.999% SLA, strong consistency" | Cloud Spanner (multi-region) |
| 2 | "PostgreSQL, need analytics on transactional data" | AlloyDB |
| 3 | "Lift-and-shift MySQL to managed service" | Cloud SQL (via Database Migration Service) |
| 4 | "Oracle RAC, cannot modify application" | Bare Metal Solution |
| 5 | "IoT, millions of writes/sec, time-series" | Cloud Bigtable |
| 6 | "Petabyte analytics, ad hoc SQL, pay per query" | BigQuery (on-demand pricing) |
| 7 | "ML training on warehouse data, SQL interface" | BigQuery ML |
| 8 | "Cross-cloud analytics on S3 data" | BigQuery Omni |
| 9 | "Data lake, unpredictable access patterns" | Cloud Storage + Autoclass |
| 10 | "Compliance: immutable objects for 7 years" | Cloud Storage + Bucket Lock + Archive class |
| 11 | "Shared filesystem for GKE, ReadWriteMany" | Filestore |
| 12 | "AI training, 100+ GB/s storage throughput" | Parallelstore |
| 13 | "Stateless containers, scale to zero" | Cloud Run |
| 14 | "Complex K8s, custom scheduling, node tuning" | GKE Standard |
| 15 | "K8s without node management, enforced security" | GKE Autopilot |
| 16 | "Spark/Hadoop migration from on-prem" | Dataproc |
| 17 | "Real-time stream processing, exactly-once" | Dataflow |
| 18 | "Batch: 10,000 container jobs with retries" | Cloud Batch |
| 19 | "SAP HANA, dedicated hardware" | Sole-tenant nodes (or Bare Metal Solution) |
| 20 | "HPC, tightly coupled, MPI" | Compute Engine + compact placement policy |
| 21 | "Highest bandwidth on-prem to GCP" | Dedicated Interconnect (10/100 Gbps) |
| 22 | "On-prem connectivity, no Google PoP nearby" | Partner Interconnect |
| 23 | "Encrypted tunnel, quick setup" | HA VPN |
| 24 | "Connect GCP to AWS with dedicated link" | Cross-Cloud Interconnect |
| 25 | "Encrypt traffic over Interconnect" | HA VPN over Interconnect |
| 26 | "Private access to Google APIs, no public IP" | Private Google Access / Private Service Connect |
| 27 | "99.99% Interconnect SLA" | Dedicated Interconnect: 4 connections, 2 metros |
| 28 | "Global HTTPS, CDN, WAF" | Global External Application LB + Cloud Armor + Cloud CDN |
| 29 | "Preserve client IP, UDP traffic" | External Passthrough Network LB |
| 30 | "Internal microservices, gRPC routing" | Regional Internal Application LB |
| 31 | "Cloud Run behind global LB" | Serverless NEG + Global External Application LB |
| 32 | "Prevent data exfiltration from BigQuery" | VPC Service Controls |
| 33 | "WAF, DDoS, OWASP Top 10" | Cloud Armor |
| 34 | "Zero-trust app access, no VPN" | Identity-Aware Proxy (IAP) |
| 35 | "Customer controls encryption keys" | Cloud KMS (CMEK) |
| 36 | "Key must never leave customer's infrastructure" | Cloud EKM (External Key Manager) |
| 37 | "Only deploy signed container images" | Binary Authorization |
| 38 | "GitHub Actions → GKE, no service account keys" | Workload Identity Federation |
| 39 | "GKE pod accesses Cloud Storage" | Workload Identity (K8s SA → GCP SA) |
| 40 | "Classify images, no ML team" | Vertex AI AutoML (Image) |
| 41 | "Custom PyTorch training on GPUs" | Vertex AI Custom Training |
| 42 | "Enterprise search over internal documents" | Vertex AI Agent Builder (Search) |
| 43 | "Fine-tune Gemini for company use case" | Model Garden + Vertex AI Tuning |
| 44 | "Zero downtime, global SaaS" | Active-Active: Spanner multi-region + GCLB + multi-region Cloud Run |
| 45 | "DR for e-commerce, 1-hour RTO" | Warm DR: cross-region DB replica + scaled-down standby |
| 46 | "Cheapest DR, 24-hour RPO acceptable" | Cold DR: GCS backups + Terraform |
| 47 | "Move 200 VMs to GCP, no code changes" | Rehost: Migrate for Compute Engine (M4CE) |
| 48 | "Monolith to microservices" | Refactor: GKE/Cloud Run + Pub/Sub + managed DBs |
| 49 | "IaC, multi-cloud, industry standard" | Terraform |
| 50 | "GCP landing zone, org best practices" | Terraform + Cloud Foundation Toolkit (CFT) |
| 51 | "GitOps for fleet of GKE clusters" | Config Sync + Policy Controller |
| 52 | "Org-wide constraint: no external IPs" | Organization Policy (`vmExternalIpAccess`) |
| 53 | "Centralized network, multiple teams" | Shared VPC (host + service projects) |
| 54 | "Two VPCs, different orgs, need connectivity" | VPC Peering (non-transitive) |
| 55 | "Route traffic through NVA / firewall appliance" | Internal Passthrough NLB as next-hop + custom route |
