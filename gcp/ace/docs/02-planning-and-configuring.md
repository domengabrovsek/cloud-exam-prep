# Domain 2: Planning and Configuring a Cloud Solution (~17.5% of Exam)

> **Quick-reference study guide for the GCP Associate Cloud Engineer renewal exam.**
> Last updated: February 2026

---

## 2.1 Planning and Configuring Compute Resources

### Compute Service Decision Matrix

| Criteria | Compute Engine | GKE | Cloud Run | Cloud Functions |
|---|---|---|---|---|
| **Abstraction level** | IaaS (full VM) | Container orchestration | Managed containers | FaaS (single function) |
| **Unit of deployment** | VM instance | Container (Pod) | Container image | Function |
| **Scaling** | MIG autoscaling | Cluster/pod autoscaler | Automatic (0 to N) | Automatic (0 to N) |
| **Scale to zero** | No | No (min 1 node) | Yes | Yes |
| **Stateful workloads** | Yes | Yes | No (stateless only) | No (stateless only) |
| **Startup time** | Minutes | Seconds (pod) | Seconds | Milliseconds-seconds |
| **Max request timeout** | Unlimited | Unlimited | 60 min (default) | 9 min (1st gen) / 60 min (2nd gen) |
| **Billing model** | Per-second (min 1 min) | Per-second (nodes) | Per-request + CPU/mem/sec | Per-invocation + compute time |
| **Sustained use discounts** | Yes | Yes (on nodes) | No | No |
| **Committed use discounts** | Yes | Yes (on nodes) | No | No |
| **GPU support** | Yes | Yes | No | No |
| **Custom OS/kernel** | Yes | Limited (node image) | No | No |

### Decision Tree: Which Compute Service?

```
START
 |
 +--> Need full OS control, custom kernel, or specific licensing?
 |     YES --> Compute Engine
 |
 +--> Running containers?
 |     YES --> Need Kubernetes features (namespaces, service mesh, stateful sets)?
 |     |        YES --> GKE
 |     |        NO  --> Want to manage cluster infrastructure?
 |     |                 YES --> GKE
 |     |                 NO  --> Cloud Run
 |     |
 |     NO --> Is it an event-driven, single-purpose function?
 |              YES --> Cloud Functions
 |              NO  --> Is it a stateless HTTP service or container?
 |                       YES --> Cloud Run
 |                       NO  --> Compute Engine
 |
 +--> Batch processing, HPC, or ML training with GPUs?
       --> Compute Engine (or GKE with GPU node pools)
```

**Docs:**
- [Where should I run my stuff?](https://cloud.google.com/blog/topics/developers-practitioners/where-should-i-run-my-stuff-choosing-google-cloud-compute-option)
- [Compute Engine docs](https://cloud.google.com/compute/docs)
- [GKE docs](https://cloud.google.com/kubernetes-engine/docs)
- [Cloud Run docs](https://cloud.google.com/run/docs)
- [Cloud Functions docs](https://cloud.google.com/functions/docs)

---

### Knative Serving on GKE

**Knative** is an open-source serverless runtime built on Kubernetes that provides automatic scaling (including scale-to-zero) and revision-based traffic management.

**Key facts:**
- **Cloud Run is built on Knative** -- Cloud Run uses Knative under the hood but is fully managed
- Knative Serving can be installed **on your own GKE cluster** to get serverless capabilities while retaining Kubernetes control
- Supports **scale to zero** (like Cloud Run, unlike standard GKE workloads)
- Provides **revision-based traffic splitting** -- route % of traffic to different versions of your service
- Integrates with **Istio** for advanced service mesh features
- Uses **Knative Services** (not to be confused with K8s Services) as the deployment unit

**When to use Knative on GKE:**
- You already have a GKE cluster and want to add serverless capabilities without migrating to Cloud Run
- You need features that Cloud Run does not offer: custom sidecars, privileged containers, GPUs (on GKE Autopilot with Knative)
- You want the portability of running the same Knative workloads on-premises or in multi-cloud environments
- You need integration with existing Kubernetes-native tooling (Helm, Operators, etc.)
- You want fine-grained control over the underlying K8s cluster (networking policies, node configuration)

**When NOT to use Knative (use Cloud Run instead):**
- You want a fully managed experience with no cluster management
- You don't need Kubernetes features or customization
- Your team is not familiar with Kubernetes
- You want the simplest path to serverless containers

**Comparison: Cloud Run vs Knative on GKE**

| Feature | Cloud Run | Knative on GKE |
|---|---|---|
| **Management** | Fully managed (no cluster to manage) | Self-managed (you manage the GKE cluster) |
| **Built on** | Knative | Knative |
| **Scale to zero** | Yes | Yes |
| **Kubernetes access** | No (abstracted away) | Full Kubernetes API access |
| **Custom networking** | Limited (VPC connector, Serverless VPC Access) | Full control (NetworkPolicies, custom CNI) |
| **Node control** | No | Yes (node pools, taints, GPU nodes, etc.) |
| **Portability** | GCP-specific | Portable (any K8s cluster with Knative) |
| **Cost** | Pay per request + compute time | Pay for GKE cluster nodes (always running unless Autopilot) |
| **Setup complexity** | None (fully managed) | Moderate (install Knative on GKE) |
| **Use case** | Simplest serverless containers | Serverless on existing K8s infrastructure |

**Updated Compute Service Decision Matrix** (with Knative)

| Criteria | Compute Engine | GKE | Cloud Run | Cloud Functions | **Knative on GKE** |
|---|---|---|---|---|---|
| **Abstraction level** | IaaS (full VM) | Container orchestration | Managed containers | FaaS (single function) | **Serverless on K8s** |
| **Unit of deployment** | VM instance | Container (Pod) | Container image | Function | **Knative Service** |
| **Scaling** | MIG autoscaling | Cluster/pod autoscaler | Automatic (0 to N) | Automatic (0 to N) | **Automatic (0 to N)** |
| **Scale to zero** | No | No (min 1 node) | Yes | Yes | **Yes** |
| **Cluster management** | N/A | Required | None | None | **Required** |
| **Kubernetes features** | N/A | Full | None | None | **Full** |
| **Managed by Google** | VMs | Cluster (optional Autopilot) | Fully | Fully | **Cluster only** |

**gcloud / kubectl example:**
```bash
# Install Knative on GKE via Cloud Run for Anthos (deprecated, use direct Knative install now)
# Modern approach: install Knative Serving via YAML or Helm

# Deploy a Knative Service (after Knative is installed on GKE)
kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
      - image: gcr.io/knative-samples/helloworld-go
        env:
        - name: TARGET
          value: "World"
EOF

# List Knative Services
kubectl get ksvc

# Route 80% traffic to v1, 20% to v2 (revision-based traffic splitting)
kubectl apply -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  traffic:
  - revisionName: hello-v1
    percent: 80
  - revisionName: hello-v2
    percent: 20
EOF
```

**Exam tip:**
> "Already have GKE but want serverless scale-to-zero for some workloads?" --> **Knative Serving on GKE**
> "Need serverless containers with zero infrastructure management?" --> **Cloud Run**
> "Want to run the same serverless workload on GCP and on-premises?" --> **Knative** (portable)

**Docs:** [Cloud Run for Anthos (deprecated, based on Knative)](https://cloud.google.com/anthos/run/docs) | [Knative.dev](https://knative.dev/) | [GKE with Knative](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-knative)

---

### Spot VMs

Spot VMs are spare Compute Engine capacity available at a **60--91% discount** off on-demand pricing. They replace the older Preemptible VMs.

| Feature | Spot VMs | Preemptible VMs (legacy) |
|---|---|---|
| Max runtime | **No limit** (unless you set one) | 24 hours hard cap |
| Preemption notice | 30-second warning | 30-second warning |
| Discount range | 60--91% | Fixed ~60--91% |
| Availability SLA | None | None |
| Live migration | No | No |
| Auto-restart | No | No |

**When to use Spot VMs:**
- Batch processing, data pipelines, CI/CD jobs
- Fault-tolerant workloads that checkpoint progress
- Large-scale distributed computing (MapReduce, Spark)
- Dev/test environments where interruption is acceptable
- MIG-managed groups that can tolerate instance loss

**Limitations -- know these for the exam:**
- Can be reclaimed **at any time** with 30-second notice
- **Cannot live migrate**; cannot be set to auto-restart on host events
- **No availability SLA** -- you might not get capacity at all
- Not suitable for serving user-facing production traffic
- Cannot be converted to a regular VM (must recreate)
- MIGs with Spot VMs may not be able to scale if capacity is constrained
- If using Spot VMs without preemptible quota, they consume your standard quota

**Exam tip:**
> "Batch processing that can be interrupted?" --> **Spot VMs**
> "Need guaranteed uptime for a web server?" --> **Regular VMs** (not Spot)

**gcloud example:**
```bash
gcloud compute instances create my-spot-vm \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP \
  --machine-type=e2-standard-4 \
  --zone=us-central1-a
```

**Docs:** [Spot VMs](https://cloud.google.com/compute/docs/instances/spot)

---

### Custom Machine Types

When predefined machine types don't match your workload, you can specify **exact vCPU and memory** combinations.

**When to use:**
- Workload needs more memory than the closest predefined type but fewer CPUs (or vice versa)
- Per-CPU software licensing -- minimize vCPUs to reduce license costs
- Right-sizing after analyzing utilization metrics to avoid overprovisioning

**Key rules:**
- Available on **N1, N2, N2D, E2** machine series (general-purpose family)
- vCPUs: must be **even number** (or 1 vCPU) -- e.g., 2, 4, 6, 8...
- Memory: between **0.5 GB and 8 GB per vCPU** (N1 can use extended memory for more)
- Memory must be a **multiple of 256 MB**
- **5% premium** over standard pricing for the equivalent resources
- Extended memory (N1 only): go beyond 8 GB/vCPU for memory-intensive workloads, charged at a higher rate

**gcloud example:**
```bash
# 4 vCPUs, 10 GB memory
gcloud compute instances create my-custom-vm \
  --custom-cpu=4 \
  --custom-memory=10GB \
  --zone=us-central1-a

# With extended memory (N1 only)
gcloud compute instances create my-custom-vm \
  --custom-cpu=4 \
  --custom-memory=30GB \
  --custom-extensions \
  --zone=us-central1-a
```

**Exam tip:**
> "Application needs 6 vCPUs and 20 GB RAM but next predefined type has 8 vCPUs and 32 GB?"
> --> **Custom machine type** to save cost.

**Docs:** [Custom machine types](https://cloud.google.com/compute/docs/instances/creating-instance-with-custom-machine-type)

---

## 2.2 Planning and Configuring Data Storage Options

### Database Product Comparison

| Feature | Cloud SQL | Cloud Spanner | Firestore | Bigtable | BigQuery |
|---|---|---|---|---|---|
| **Type** | Relational (managed) | Relational (distributed) | Document (NoSQL) | Wide-column (NoSQL) | Analytical (columnar) |
| **Data model** | Tables, rows, SQL | Tables, rows, SQL | Documents, collections | Key-value / wide-column | Tables, SQL-like |
| **Engines** | MySQL, PostgreSQL, SQL Server | Proprietary | Proprietary (was Datastore) | Proprietary | Proprietary |
| **Scale** | Vertical (read replicas) | **Horizontal** (global) | Auto-scales | **Horizontal** (nodes) | Auto-scales |
| **Max storage** | 64 TB | Unlimited | Unlimited | Unlimited | Unlimited |
| **Consistency** | Strong (single region) | **Strong (global)** | Strong | Eventual (single-row strong) | Eventual |
| **Transactions** | Full ACID | Full ACID | ACID (limited) | Single-row atomic | No (DML has limits) |
| **Availability** | 99.95% (regional HA) | Up to **99.999%** (multi-region) | 99.999% (multi-region) | 99.99% (replicated) | 99.99% |
| **Best for** | Web apps, CMS, ERP, OLTP | Global finance, gaming, supply chain | Mobile/web apps, real-time sync | IoT, time-series, analytics (>1 TB) | Data warehouse, BI, ad-hoc analytics |
| **Cost model** | Per instance hour + storage | Per node-hour + storage | Per read/write/delete ops + storage | Per node-hour + storage | Per query (on-demand) or slots (flat-rate) + storage |
| **Min recommended data size** | Any | Any (but cost-effective at scale) | Any | **> 1 TB** (not cost-effective below) | Any (optimized for large datasets) |

### Database Decision Tree

```
START
 |
 +--> Is it an analytics/reporting/BI workload?
 |     YES --> BigQuery
 |
 +--> Need relational (SQL) with ACID transactions?
 |     YES --> Need global scale or > 99.95% availability?
 |     |        YES --> Cloud Spanner
 |     |        NO  --> Cloud SQL
 |
 +--> Document-oriented / mobile-web real-time sync?
 |     YES --> Firestore
 |
 +--> High-throughput, low-latency, wide-column / time-series / IoT?
 |     YES --> Is data > 1 TB?
 |              YES --> Bigtable
 |              NO  --> Firestore (or Bigtable if throughput demands it)
```

**Exam tips:**
> - "Relational data, global consistency?" --> **Cloud Spanner**
> - "Relational data, single-region, well-understood workload?" --> **Cloud SQL**
> - "Mobile app with offline sync?" --> **Firestore**
> - "Millions of time-series writes per second?" --> **Bigtable**
> - "Run SQL analytics on petabytes of data?" --> **BigQuery**

**Docs:**
- [Cloud SQL](https://cloud.google.com/sql/docs)
- [Cloud Spanner](https://cloud.google.com/spanner/docs)
- [Firestore](https://cloud.google.com/firestore/docs)
- [Bigtable](https://cloud.google.com/bigtable/docs)
- [BigQuery](https://cloud.google.com/bigquery/docs)
- [Database options explained (blog)](https://cloud.google.com/blog/topics/developers-practitioners/your-google-cloud-database-options-explained)

---

### Storage Options: Persistent Disks

| Feature | Zonal Persistent Disk | Regional Persistent Disk |
|---|---|---|
| **Replication** | Within a single zone | Across **2 zones** in the same region |
| **Availability** | Protected against hardware failure in zone | Protected against **full zonal outage** |
| **Cost** | Standard pricing | **2x** the cost of zonal PD |
| **Attachment** | VMs in the same zone only | VMs in either of the 2 replica zones |
| **Failover** | No cross-zone failover | Can **force-attach** to a VM in the other zone |
| **Performance** | Standard/SSD/Balanced | Standard/SSD/Balanced (same types available) |
| **Use case** | General workloads, dev/test | HA workloads, databases, critical services |
| **Snapshots** | Yes (stored in multi-regional or regional) | Yes |

**Exam tip:**
> "Need disk-level HA without a full regional database?" --> **Regional Persistent Disk**
> "Cost-sensitive workload in a single zone?" --> **Zonal Persistent Disk**

**Docs:** [Persistent Disk overview](https://cloud.google.com/compute/docs/disks/persistent-disks) | [Regional PD](https://cloud.google.com/compute/docs/disks/regional-persistent-disk)

---

### Google Cloud Hyperdisk

**Hyperdisk** is a next-generation block storage service for Compute Engine that **decouples IOPS and throughput from disk size**. Traditional Persistent Disks scale performance linearly with capacity, but Hyperdisk lets you independently provision performance.

**Key difference from standard Persistent Disks:**
- **Standard PD (pd-standard, pd-balanced, pd-ssd):** IOPS and throughput scale with disk size. Want more IOPS? Provision a larger disk, even if you don't need the capacity.
- **Hyperdisk:** IOPS and throughput are independently configurable. You can provision a small disk with high IOPS or a large disk with low IOPS based on your workload needs.

**Hyperdisk types:**

| Type | Best For | Max IOPS | Max Throughput | Capacity Range |
|---|---|---|---|---|
| **Hyperdisk Balanced** | General-purpose, balanced IOPS and throughput | 80,000 read / 80,000 write | 1,200 MB/s read / 1,200 MB/s write | 4 GB - 64 TB |
| **Hyperdisk Extreme** | Ultra-high IOPS, latency-sensitive databases | **350,000 read / 350,000 write** | 2,400 MB/s read / 2,400 MB/s write | 64 GB - 64 TB |
| **Hyperdisk Throughput** | High sequential I/O, analytics, streaming | 80,000 read / 80,000 write | **Up to 4,800 MB/s read / write** | 2 TB - 64 TB |
| **Hyperdisk ML** | Machine learning, training, large sequential datasets | 160,000 read / 160,000 write | 2,400 MB/s read / 2,400 MB/s write | 4 GB - 64 TB |

**Comparison: Hyperdisk vs Standard Persistent Disks**

| Feature | pd-standard | pd-balanced | pd-ssd | Hyperdisk Extreme | Hyperdisk Balanced | Hyperdisk Throughput |
|---|---|---|---|---|---|---|
| **IOPS scaling** | Scales with size | Scales with size | Scales with size | **Provisioned independently** | **Provisioned independently** | **Provisioned independently** |
| **Throughput scaling** | Scales with size | Scales with size | Scales with size | **Provisioned independently** | **Provisioned independently** | **Provisioned independently** |
| **Max IOPS** | 7,500 | 80,000 | 100,000 | **350,000** | 80,000 | 80,000 |
| **Max Throughput** | 1,200 MB/s | 1,200 MB/s | 1,200 MB/s | 2,400 MB/s | 1,200 MB/s | **4,800 MB/s** |
| **Typical latency** | ~10 ms | ~1 ms | <1 ms | **Sub-millisecond** | ~1 ms | ~1 ms |
| **Cost model** | Per GB-month | Per GB-month | Per GB-month | **Per GB-month + per provisioned IOPS + per provisioned throughput** | **Per GB-month + per provisioned IOPS + per provisioned throughput** | **Per GB-month + per provisioned IOPS + per provisioned throughput** |
| **Use case** | Archival, low I/O | General purpose | High-performance OLTP | **Extreme IOPS databases, sub-ms latency** | Balanced workloads with flexible perf | **Streaming, analytics, ML datasets** |

**Use cases for Hyperdisk:**

| Workload | Recommended Hyperdisk Type | Why |
|---|---|---|
| High-performance transactional databases (MySQL, PostgreSQL, SAP HANA) | **Hyperdisk Extreme** | Highest IOPS, lowest latency |
| NoSQL databases (Cassandra, MongoDB) needing high IOPS | **Hyperdisk Extreme** | Independent IOPS provisioning |
| Large-scale ML model training with sequential reads | **Hyperdisk Throughput** or **Hyperdisk ML** | Maximize throughput, not IOPS |
| Analytics workloads (Hadoop, Spark) processing large files | **Hyperdisk Throughput** | High sequential throughput |
| General-purpose VMs needing predictable performance | **Hyperdisk Balanced** | Flexible IOPS/throughput |
| Small disk needing high IOPS (e.g., 100 GB with 50,000 IOPS) | **Hyperdisk Balanced or Extreme** | Standard PD would require ~500 GB to reach that IOPS |

**When to use Hyperdisk vs standard Persistent Disk:**

- **Use Hyperdisk when:**
  - You need IOPS or throughput that doesn't match your capacity requirements
  - You want to avoid overprovisioning disk size just to meet performance targets
  - You need the absolute highest IOPS (>100K) or throughput (>1,200 MB/s)
  - Your workload has unpredictable I/O patterns and you want to scale performance independently

- **Use standard Persistent Disk when:**
  - Your performance needs scale proportionally with capacity
  - You want simpler, usage-based pricing (no separate IOPS/throughput charges)
  - Your workload fits within pd-ssd or pd-balanced performance limits
  - Cost optimization via committed use discounts (Hyperdisk has fewer discount options currently)

**gcloud example:**
```bash
# Create a Hyperdisk Extreme with custom IOPS
gcloud compute disks create my-hyperdisk-extreme \
  --type=hyperdisk-extreme \
  --size=500GB \
  --provisioned-iops=100000 \
  --provisioned-throughput=1500 \
  --zone=us-central1-a

# Create a Hyperdisk Balanced disk
gcloud compute disks create my-hyperdisk-balanced \
  --type=hyperdisk-balanced \
  --size=100GB \
  --provisioned-iops=10000 \
  --zone=us-central1-a

# Create a Hyperdisk Throughput for ML workloads
gcloud compute disks create my-hyperdisk-ml \
  --type=hyperdisk-throughput \
  --size=5TB \
  --provisioned-throughput=3000 \
  --zone=us-central1-a

# Attach Hyperdisk to a VM
gcloud compute instances attach-disk my-vm \
  --disk=my-hyperdisk-extreme \
  --zone=us-central1-a
```

**Exam tips:**
> - "Database needs 100,000 IOPS but only 200 GB storage?" --> **Hyperdisk Extreme** (decouple IOPS from size)
> - "ML training pipeline processing large sequential datasets?" --> **Hyperdisk Throughput** or **Hyperdisk ML**
> - "Want highest possible IOPS for transactional database?" --> **Hyperdisk Extreme** (350K IOPS)
> - "Standard workload, want cost-effective balanced performance?" --> **pd-balanced** (standard PD is often cheaper for typical workloads)
> - Hyperdisk costs more than standard PD for equivalent capacity -- only use when you need independent performance scaling

**Docs:** [Hyperdisk overview](https://cloud.google.com/compute/docs/disks/hyperdisks) | [Hyperdisk types](https://cloud.google.com/compute/docs/disks/hyperdisks#hyperdisk_types) | [Provisioning IOPS and throughput](https://cloud.google.com/compute/docs/disks/hyperdisks#provisioned_iops)

---

### Cloud Storage Classes

| Storage Class | Min Storage Duration | Use Case | Availability SLA (multi-region / region) | Relative Storage Cost | Relative Access Cost |
|---|---|---|---|---|---|
| **Standard** | None | Frequently accessed ("hot") data | 99.95% / 99.9% | Highest | Lowest |
| **Nearline** | **30 days** | Accessed less than once/month | 99.9% / 99.0% | Lower | Higher |
| **Coldline** | **90 days** | Accessed less than once/quarter | 99.9% / 99.0% | Even lower | Even higher |
| **Archive** | **365 days** | Accessed less than once/year | 99.9% / 99.0% | **Lowest** | **Highest** |

**Key facts for the exam:**

1. **Early deletion charges** -- If you delete, replace, or move an object before the minimum storage duration, you are billed as if it was stored for the full minimum duration.
2. **All classes** have the same throughput, latency, and durability (eleven 9s).
3. **All classes** have millisecond access time (Archive is not tape -- it's still fast to read).
4. **Autoclass** automatically transitions objects between classes based on access patterns.
5. **Object Lifecycle Management** rules can transition objects between classes (can only move to a colder class, e.g., Standard --> Nearline --> Coldline --> Archive).

**Cost tradeoff pattern:**

```
Storage cost:   Standard > Nearline > Coldline > Archive
Access cost:    Standard < Nearline < Coldline < Archive
Min duration:   None       30 days    90 days    365 days
```

**Exam tips:**
> - "Store backups accessed quarterly?" --> **Coldline** (90-day min, cheaper storage)
> - "Long-term compliance archives, rarely accessed?" --> **Archive** (365-day min, cheapest storage)
> - "Data accessed multiple times a month?" --> **Standard**
> - "Monthly access for reporting?" --> **Nearline** (30-day min)
> - "Don't know the access pattern?" --> **Autoclass** or start with Standard

**Docs:** [Storage classes](https://cloud.google.com/storage/docs/storage-classes) | [Storage pricing](https://cloud.google.com/storage/pricing)

---

## 2.3 Planning and Configuring Network Resources

### Load Balancing Types

Google Cloud load balancers are categorized by **traffic type**, **scope**, and **proxy behavior**.

| Load Balancer | Layer | Scope | Traffic Type | Proxy / Passthrough | External / Internal |
|---|---|---|---|---|---|
| **External Application LB** (HTTP/S) | L7 | Global or Regional | HTTP / HTTPS | Proxy | External |
| **Internal Application LB** (HTTP/S) | L7 | Regional (or cross-region) | HTTP / HTTPS | Proxy | Internal |
| **External Proxy Network LB** (SSL Proxy) | L4 | Global | SSL/TLS (non-HTTP) | Proxy (terminates SSL) | External |
| **External Proxy Network LB** (TCP Proxy) | L4 | Global | TCP (non-SSL) | Proxy | External |
| **External Passthrough Network LB** | L4 | Regional | TCP / UDP / ESP / ICMP | Passthrough (DSR) | External |
| **Internal Passthrough Network LB** | L4 | Regional | TCP / UDP / ICMP | Passthrough (DSR) | Internal |
| **Internal Proxy Network LB** | L4 | Regional | TCP | Proxy (Envoy-based) | Internal |

### Load Balancer Decision Tree

```
START
 |
 +--> Is traffic INTERNAL (VPC-to-VPC, on-prem to VPC)?
 |     YES --> HTTP/HTTPS traffic?
 |     |        YES --> Internal Application LB (L7)
 |     |        NO  --> Need proxy features (e.g., Envoy, traffic management)?
 |     |                 YES --> Internal Proxy Network LB
 |     |                 NO  --> Internal Passthrough Network LB (TCP/UDP)
 |     |
 |     NO (EXTERNAL) -->
 |         HTTP/HTTPS traffic?
 |         YES --> External Application LB (global or regional)
 |         NO  --> SSL/TLS (non-HTTP) traffic?
 |                  YES --> External Proxy Network LB (SSL Proxy) -- global
 |                  NO  --> TCP-only (non-SSL)?
 |                           YES --> Need global? --> External Proxy Network LB (TCP Proxy)
 |                           NO  --> TCP/UDP/other --> External Passthrough Network LB (regional)
```

**Key exam scenarios:**

| Scenario | Answer |
|---|---|
| Global HTTP(S) load balancer for a web app | External Application LB (global) |
| Internal microservices communicating via HTTP | Internal Application LB |
| Non-HTTP SSL traffic (e.g., LDAPS) across regions | External Proxy Network LB (SSL Proxy) |
| UDP game server traffic, single region | External Passthrough Network LB |
| Internal TCP/UDP between services in a VPC | Internal Passthrough Network LB |
| Need to preserve client source IP (external) | External Passthrough Network LB |
| Need Cloud Armor (DDoS protection) on HTTP | External Application LB |
| WebSocket support | External Application LB (supports WebSockets) |

**Docs:**
- [Load balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)
- [Choose a load balancer](https://cloud.google.com/load-balancing/docs/choosing-load-balancer)
- [Feature comparison](https://cloud.google.com/load-balancing/docs/features)

---

### Resource Location and Availability in a Network

Understanding where resources live is critical for exam questions about network design.

| Resource | Scope | Notes |
|---|---|---|
| VPC | **Global** | Spans all regions automatically |
| Subnet | **Regional** | One subnet per region; IP range is regional |
| Firewall rules | **Global** (VPC-level) | Apply across all regions in the VPC |
| Routes | **Global** (VPC-level) | Apply to instances by tags or all instances |
| VM instances | **Zonal** | Exist in one zone |
| Managed Instance Groups | Zonal or **Regional** | Regional MIG spreads across zones in a region |
| Internal IP addresses | **Regional** | Bound to the subnet's region |
| External IP addresses | Regional or **Global** | Global for global LBs (Premium Tier); Regional for Standard Tier |
| Cloud Interconnect | **Regional** (attachment) | Connects to a specific region's VLAN attachment |
| Cloud VPN | **Regional** | Gateway in a specific region |
| Cloud Router | **Regional** | Manages BGP sessions for a region |

**Exam tip:**
> VPCs are global, subnets are regional. This means a single VPC can have subnets in every region -- you do NOT need one VPC per region. Firewall rules apply globally across the VPC.

**Docs:** [VPC overview](https://cloud.google.com/vpc/docs/vpc) | [Regions and zones](https://cloud.google.com/compute/docs/regions-zones)

---

### Network Service Tiers: Premium vs Standard

| Feature | Premium Tier | Standard Tier |
|---|---|---|
| **Routing** | Cold-potato routing: traffic enters Google's network at the nearest PoP and traverses Google's **private backbone** | Hot-potato routing: traffic enters/exits via the **nearest peering point** to the VM's region, traveling over the public internet |
| **Global IP addresses** | Yes (anycast) | No (regional only) |
| **Global load balancing** | Yes | No (regional LBs only) |
| **SLA** | **99.99%** for VM connectivity | No comparable SLA |
| **Performance** | Lower latency, higher throughput, more consistent | Higher latency, variable performance |
| **Cost** | Higher egress pricing | **Lower egress pricing** |
| **Cloud CDN** | Supported | Not supported |
| **Cloud Armor** | Supported (with global LB) | Not supported (no global LB) |
| **Default** | **Yes** -- Premium is the default | Must be explicitly selected |
| **Scope** | Set per project, resource, or per forwarding rule | Set per project, resource, or per forwarding rule |

**How routing differs in practice:**

```
Premium Tier (cold-potato):
  User --> [Nearest Google PoP] --Google backbone--> [GCP Region] --> VM

Standard Tier (hot-potato):
  User --> [Public Internet] --> [Google PoP near VM region] --> VM
```

**Exam tips:**
> - "Optimize for global performance?" --> **Premium Tier**
> - "Optimize for cost, single-region app, latency not critical?" --> **Standard Tier**
> - "Need a global HTTP(S) LB or Cloud CDN?" --> **Must use Premium Tier**
> - "Need a global anycast IP?" --> **Premium Tier only**
> - Standard Tier only supports **regional** external IP addresses and **regional** load balancers

**Docs:** [Network Service Tiers overview](https://cloud.google.com/network-tiers/docs/overview) | [Network Service Tiers pricing](https://cloud.google.com/network-tiers/pricing)

---

### Cloud Next Generation Firewall (Cloud NGFW)

**Cloud NGFW** extends traditional VPC firewall rules with advanced enterprise security features including **hierarchical policies**, **FQDN-based rules**, **geo-based filtering**, **threat intelligence**, and **intrusion detection/prevention (IDS/IPS)**.

**What Cloud NGFW adds over standard VPC firewall rules:**

| Feature | Standard VPC Firewall Rules | Cloud NGFW |
|---|---|---|
| **Scope** | VPC-level (apply to one VPC) | **Hierarchical** (org/folder/project level) |
| **Rule targeting** | IP ranges, tags, service accounts | **+ FQDN (domain names), geo-location, threat intelligence feeds** |
| **Inspection depth** | Layer 3/4 (IP, protocol, port) | **Layer 7** (application-aware, deep packet inspection) |
| **IDS/IPS** | No | **Yes** (intrusion detection and prevention) |
| **Threat intelligence** | No | **Yes** (Google's threat feeds, Palo Alto Networks integration) |
| **Centralized management** | Per-VPC | **Centralized** (org-wide policy inheritance) |
| **Policy tiers** | N/A | **Standard, Enterprise, Enterprise Plus** |
| **Logging** | Basic (Firewall Logs to Cloud Logging) | **Enhanced** (detailed threat logs, PCAP export) |

**Hierarchical Firewall Policies:**

Cloud NGFW supports **hierarchical policies** that cascade from organization --> folder --> VPC, allowing centralized security governance.

- **Organization-level policies** apply to all VPCs in the org (e.g., block all traffic from specific countries)
- **Folder-level policies** apply to all projects in a folder (e.g., allow internal traffic for a business unit)
- **VPC-level policies** are specific to a VPC (e.g., allow HTTP to a specific subnet)
- Policies are **inherited** and **evaluated in order**: org --> folder --> VPC --> individual rules
- Lower-level policies **cannot override** higher-level denies (security best practice)

**Cloud NGFW Policy Tiers:**

| Tier | Capabilities | Use Case |
|---|---|---|
| **Standard** | Hierarchical firewall policies, FQDN filtering, geo-based rules | Basic centralized policy management |
| **Enterprise** | Standard + **IDS** (intrusion detection) | Threat visibility and alerting |
| **Enterprise Plus** | Enterprise + **IPS** (intrusion prevention), advanced threat detection, malware defense | Full threat prevention (auto-block malicious traffic) |

**Key features in detail:**

1. **FQDN-based rules:** Allow or deny traffic by domain name (e.g., `allow traffic to *.googleapis.com`)
   - Traditional VPC rules require IP addresses, which can change for external services
   - FQDN rules automatically resolve and update as IPs change

2. **Geo-based filtering:** Block or allow traffic from specific countries or regions
   - Example: `deny all traffic from source country: CN, RU, NK`
   - Useful for compliance (GDPR, data residency) and security (nation-state threat actors)

3. **Threat intelligence:** Automatically block traffic from known malicious IPs
   - Google's threat feeds updated in real-time
   - Integration with third-party feeds (e.g., Palo Alto Networks)

4. **IDS/IPS (Enterprise/Enterprise Plus):**
   - **IDS (Intrusion Detection):** Alerts on suspicious traffic patterns (log only, does not block)
   - **IPS (Intrusion Prevention):** Automatically blocks detected threats (requires Enterprise Plus)
   - Uses signature-based and behavioral analysis to detect attacks (SQL injection, XSS, malware, C2 traffic)

**When to use Cloud NGFW:**
- You need **centralized, org-wide security policies** that apply across all VPCs
- You want **domain-based firewall rules** (e.g., allow traffic to `*.google.com` without hardcoding IPs)
- You need to **block traffic by country** (geo-filtering for compliance or security)
- You want **intrusion detection/prevention** without deploying third-party appliances
- You need **threat intelligence integration** to auto-block known malicious actors
- You have regulatory requirements for advanced logging and threat visibility

**When NOT to use Cloud NGFW (standard VPC firewall rules are sufficient):**
- Simple allow/deny rules based on IP ranges and ports
- Single-VPC environments with no centralized policy needs
- Cost-sensitive workloads (Cloud NGFW has additional charges)
- No advanced threat detection requirements

**Comparison: VPC Firewall Rules vs Cloud NGFW**

| Scenario | Standard VPC Firewall | Cloud NGFW |
|---|---|---|
| Allow SSH (port 22) from specific IP range | Yes (simple ingress rule) | Overkill (use standard rules) |
| Block all traffic from China and Russia | Complex (need IP lists) | **Yes (geo-filtering)** |
| Allow traffic to `*.googleapis.com` | Complex (IPs change) | **Yes (FQDN rules)** |
| Detect SQL injection attempts | No | **Yes (IDS/IPS)** |
| Apply org-wide "deny all RDP" policy | Manual per VPC | **Yes (hierarchical policy)** |
| Auto-block traffic from known malware C2 servers | No | **Yes (threat intelligence)** |

**gcloud example:**
```bash
# Create a hierarchical firewall policy at the organization level
gcloud compute firewall-policies create my-org-policy \
  --organization=123456789012 \
  --description="Organization-wide security policy"

# Add a rule to block traffic from specific countries (geo-filtering)
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=my-org-policy \
  --organization=123456789012 \
  --action=deny \
  --direction=INGRESS \
  --src-region-codes=CN,RU \
  --description="Block traffic from China and Russia"

# Add a rule allowing traffic to Google APIs using FQDN
gcloud compute firewall-policies rules create 2000 \
  --firewall-policy=my-org-policy \
  --organization=123456789012 \
  --action=allow \
  --direction=EGRESS \
  --dest-fqdns=*.googleapis.com \
  --description="Allow traffic to Google APIs"

# Associate the policy with a folder or VPC
gcloud compute firewall-policies associations create \
  --firewall-policy=my-org-policy \
  --organization=123456789012 \
  --folder=456789012345

# Enable Cloud NGFW Enterprise (IDS/IPS)
gcloud compute networks update my-vpc \
  --enable-cloud-ngfw-enterprise
```

**Exam tips:**
> - "Need to block traffic from specific countries?" --> **Cloud NGFW (geo-filtering)**
> - "Firewall rule for domain names, not IPs?" --> **Cloud NGFW (FQDN rules)**
> - "Apply firewall policy across entire organization?" --> **Cloud NGFW (hierarchical policies)**
> - "Detect and prevent SQL injection at network layer?" --> **Cloud NGFW Enterprise Plus (IPS)**
> - "Simple allow/deny rules for internal subnets?" --> **Standard VPC firewall rules** (simpler, cheaper)
> - Cloud NGFW is **additive** -- it works alongside standard VPC firewall rules, not instead of them

**Docs:** [Cloud NGFW overview](https://cloud.google.com/firewall/docs/about-firewall-policies) | [Hierarchical firewall policies](https://cloud.google.com/firewall/docs/firewall-policies) | [Cloud NGFW Enterprise](https://cloud.google.com/firewall/docs/about-intrusion-prevention) | [FQDN rules](https://cloud.google.com/firewall/docs/using-fqdn-filtering)

---

### Secure Tags vs Network Tags

**Tags** are used to target firewall rules and routes, but **network tags** and **secure tags** have critical differences in security governance.

**Network Tags (traditional):**
- **Not IAM-controlled** -- Any user with `compute.instances.update` permission can add/remove network tags from a VM
- Applied directly to VM instances
- Used by **standard VPC firewall rules** and **routes**
- **Security risk:** A developer could tag their VM with a production tag to bypass firewall restrictions
- Managed via `gcloud compute instances add-tags` / `remove-tags`

**Secure Tags (Resource Manager Tags):**
- **IAM-controlled** -- Require `resourcemanager.tagValues.use` permission to attach a tag to a resource
- Centrally managed via **Resource Manager** (org/folder/project hierarchy)
- Used by **Cloud NGFW (hierarchical firewall policies)**
- **Security benefit:** Tags are governed by IAM, preventing unauthorized users from changing tags
- Support **conditional IAM policies** based on tags (e.g., "only allow production SA to access resources tagged `env:prod`")
- Managed via `gcloud resource-manager tags` commands

**Key differences:**

| Feature | Network Tags | Secure Tags (Resource Manager Tags) |
|---|---|---|
| **IAM governance** | **No** (anyone with VM edit can change tags) | **Yes** (require IAM permission to use tag) |
| **Management scope** | Compute Engine only | **Org/folder/project hierarchy** |
| **Used by** | Standard VPC firewall rules, routes | **Cloud NGFW (hierarchical policies), IAM conditions** |
| **Centralized control** | No (per-VM) | **Yes** (org-wide tag definitions) |
| **Security risk** | High (developers can self-tag) | Low (centralized, IAM-protected) |
| **Best practice** | Use for simple, dev/test environments | **Use for production, compliance, security** |
| **Example** | `env:prod`, `tier:web` (set on VM) | `tagValues/123456789012` (key-value pair managed in Resource Manager) |

**When to use each:**

| Scenario | Recommended Tag Type |
|---|---|
| Dev/test environment with trusted users | Network tags (simpler) |
| Production environment with strict security | **Secure tags** (IAM-governed) |
| Firewall rule targeting in a single VPC | Network tags (if no IAM control needed) |
| Org-wide firewall policy enforcement | **Secure tags** (with Cloud NGFW) |
| Prevent unauthorized tag changes | **Secure tags** |
| Conditional IAM (e.g., "only allow access to resources tagged `compliance:pci`") | **Secure tags** |

**gcloud examples:**

```bash
# ---- NETWORK TAGS (traditional) ----

# Add a network tag to a VM
gcloud compute instances add-tags my-vm \
  --tags=web,production \
  --zone=us-central1-a

# Remove a network tag
gcloud compute instances remove-tags my-vm \
  --tags=production \
  --zone=us-central1-a

# Use network tags in a standard VPC firewall rule
gcloud compute firewall-rules create allow-web \
  --target-tags=web \
  --allow=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0

# ---- SECURE TAGS (Resource Manager Tags) ----

# Create a tag key (e.g., "environment")
gcloud resource-manager tags keys create environment \
  --parent=organizations/123456789012 \
  --description="Environment classification"

# Create tag values under the key (e.g., "production", "staging")
gcloud resource-manager tags values create production \
  --parent=tagKeys/456789012345 \
  --description="Production environment"

gcloud resource-manager tags values create staging \
  --parent=tagKeys/456789012345 \
  --description="Staging environment"

# Attach a secure tag to a VM
gcloud resource-manager tags bindings create \
  --location=us-central1-a \
  --tag-value=tagValues/789012345678 \
  --parent=//compute.googleapis.com/projects/my-project/zones/us-central1-a/instances/my-vm

# Use secure tags in a Cloud NGFW hierarchical firewall policy
gcloud compute firewall-policies rules create 3000 \
  --firewall-policy=my-org-policy \
  --organization=123456789012 \
  --action=allow \
  --direction=INGRESS \
  --target-secure-tags=tagValues/789012345678 \
  --src-ip-ranges=10.0.0.0/8 \
  --description="Allow internal traffic to production VMs"

# Grant IAM permission to use a secure tag
gcloud resource-manager tags values add-iam-policy-binding tagValues/789012345678 \
  --member=user:alice@example.com \
  --role=roles/resourcemanager.tagUser
```

**Exam tips:**
> - "Prevent developers from changing firewall targeting tags?" --> **Secure tags** (IAM-protected)
> - "Org-wide policy enforcement across all VPCs?" --> **Secure tags** + Cloud NGFW
> - "Simple firewall rule for a single VPC, no security risk?" --> **Network tags** (simpler)
> - "Conditional IAM based on resource classification?" --> **Secure tags**
> - **Best practice for production:** Use secure tags with Cloud NGFW for centralized, IAM-governed security

**Docs:** [Network tags](https://cloud.google.com/vpc/docs/add-remove-network-tags) | [Secure tags (Resource Manager Tags)](https://cloud.google.com/resource-manager/docs/tags/tags-overview) | [Tags with firewall policies](https://cloud.google.com/firewall/docs/using-tags-with-firewall-policies)

---

### Cloud Interconnect: Dedicated vs Partner

**Cloud Interconnect** provides **private, high-bandwidth connectivity** between your on-premises network and Google Cloud, bypassing the public internet. Two options: **Dedicated Interconnect** and **Partner Interconnect**.

**Dedicated Interconnect:**
- **Direct physical connection** between your on-premises network and Google's network at a **Google colocation facility (PoP)**
- **Bandwidth:** 10 Gbps or 100 Gbps per link (can bundle multiple links for more capacity)
- **Latency:** Lower and more consistent than internet or VPN
- **Cost:** Connection fee + egress charges (lower than internet egress)
- **Setup complexity:** Requires physical cross-connect in a colocation facility
- **Use case:** Highest bandwidth needs, latency-sensitive workloads, large-scale hybrid cloud, data migration

**Partner Interconnect:**
- **Connection via a supported service provider** (ISP or carrier) -- no need to collocate at a Google PoP
- **Bandwidth:** 50 Mbps to 50 Gbps per VLAN attachment (more flexible sizing)
- **Latency:** Higher than Dedicated, lower than internet/VPN (depends on partner network)
- **Cost:** Partner charges (varies) + Google egress charges
- **Setup complexity:** Easier (partner handles physical connectivity)
- **Use case:** Don't have presence in Google colocation facility, need flexible bandwidth, lower bandwidth requirements

**Cloud VPN (for comparison):**
- **IPsec VPN tunnel** over the public internet
- **Bandwidth:** 3 Gbps per tunnel (HA VPN supports multiple tunnels)
- **Latency:** Higher and variable (depends on internet path)
- **Cost:** Cheapest option (VPN gateway charges + egress)
- **Setup complexity:** Easiest (no physical infrastructure)
- **Use case:** Low-bandwidth, cost-sensitive, quick setup, temporary connectivity

**Comparison table:**

| Feature | Dedicated Interconnect | Partner Interconnect | Cloud VPN |
|---|---|---|---|
| **Bandwidth** | 10 Gbps / 100 Gbps per link | 50 Mbps - 50 Gbps | 3 Gbps per tunnel (multi-tunnel HA VPN) |
| **Max total capacity** | **200 Gbps** (8 x 100 Gbps) or more | **50 Gbps** per attachment | ~10-20 Gbps (with multiple tunnels) |
| **Latency** | **Lowest** (direct connection) | Low (via partner network) | Higher (over internet) |
| **SLA** | **99.99%** (dual attachments) | **99.99%** (dual attachments) | 99.9% (HA VPN) |
| **Physical connection** | Direct at Google PoP | Via partner (no colocation needed) | **None** (virtual tunnel) |
| **Setup time** | Weeks (cross-connect provisioning) | Days-weeks (partner-dependent) | **Minutes-hours** |
| **Cost** | High (connection fee + lower egress) | Medium (partner fee + egress) | **Lowest** (VPN gateway + egress) |
| **Use private IPs** | Yes | Yes | Yes (via Cloud VPN to VPC) |
| **Encryption** | Not encrypted (private fiber) | Not encrypted (private connection) | **Encrypted** (IPsec) |
| **Google SLA requirement** | Need **2 connections** (redundancy) | Need **2 VLAN attachments** | Need **2 tunnels** (HA VPN) |

**When to use each:**

| Scenario | Recommended Option |
|---|---|
| Need 50+ Gbps, low latency, have presence in Google PoP | **Dedicated Interconnect** |
| Need 1-50 Gbps, don't have presence in Google PoP | **Partner Interconnect** |
| Need <1 Gbps, cost-sensitive, quick setup | **Cloud VPN** |
| Migrating TBs/PBs of data to Google Cloud | **Dedicated Interconnect** (or Transfer Appliance) |
| Need encryption in transit | **Cloud VPN** (or encrypt at app layer) |
| Temporary or dev/test connectivity | **Cloud VPN** |
| High availability, mission-critical hybrid cloud | **Dedicated Interconnect (dual)** or **Partner Interconnect (dual)** |

**Key setup steps (Dedicated Interconnect):**

1. **Verify colocation facility:** Ensure your on-prem network or data center is in or near a [Google colocation facility](https://cloud.google.com/interconnect/docs/how-to/dedicated/colocation-facilities)
2. **Create VLAN attachments:** Define the VPC networks and regions to connect
3. **Provision physical cross-connect:** Order a cross-connect from your colocation provider to Google's equipment
4. **Configure Cloud Router:** Set up BGP peering between your on-prem router and Google Cloud Router
5. **Advertise routes:** Exchange routes via BGP (on-prem subnets <--> VPC subnets)
6. **Test connectivity:** Verify traffic flows over the Interconnect (not the internet)

**Key setup steps (Partner Interconnect):**

1. **Choose a supported partner:** Select from [Google's supported service providers](https://cloud.google.com/interconnect/docs/how-to/partner/supported-service-providers)
2. **Request connection from partner:** Partner provisions connectivity between your network and Google
3. **Create VLAN attachments:** Define VPC networks and bandwidth requirements in Google Cloud
4. **Partner completes provisioning:** Partner establishes the link to Google
5. **Configure Cloud Router and BGP:** Same as Dedicated Interconnect
6. **Activate VLAN attachment:** Once partner confirms, activate the attachment

**gcloud examples:**

```bash
# ---- DEDICATED INTERCONNECT ----

# Create an Interconnect connection (this reserves capacity at a colocation facility)
gcloud compute interconnects create my-interconnect \
  --interconnect-type=DEDICATED \
  --link-type=LINK_TYPE_ETHERNET_10G_LR \
  --location=den-zone1-1 \
  --requested-link-count=1 \
  --description="Dedicated Interconnect to Denver PoP"

# Create a VLAN attachment (connects the Interconnect to a VPC)
gcloud compute interconnects attachments dedicated create my-attachment \
  --interconnect=my-interconnect \
  --router=my-cloud-router \
  --region=us-west1 \
  --vlan=100 \
  --description="VLAN attachment for VPC-1"

# Create a Cloud Router for BGP peering
gcloud compute routers create my-cloud-router \
  --network=my-vpc \
  --region=us-west1 \
  --asn=65001

# ---- PARTNER INTERCONNECT ----

# Create a Partner Interconnect VLAN attachment (pairing key is shared with partner)
gcloud compute interconnects attachments partner create my-partner-attachment \
  --router=my-cloud-router \
  --region=us-central1 \
  --edge-availability-domain=AVAILABILITY_DOMAIN_1 \
  --bandwidth=BPS_1G \
  --description="Partner Interconnect via ExampleISP"

# The command returns a pairing key -- provide this to your service provider
# Example pairing key: 7e51371e-72a3-40b5-b844-2e3efefaee59/us-central1/1

# After the partner provisions the connection, activate the attachment
gcloud compute interconnects attachments partner update my-partner-attachment \
  --region=us-central1 \
  --admin-enabled

# ---- CONFIGURE CLOUD ROUTER (BGP) ----

# Add a BGP peer to the Cloud Router (for both Dedicated and Partner)
gcloud compute routers add-bgp-peer my-cloud-router \
  --peer-name=on-prem-peer \
  --peer-asn=65002 \
  --interface=interconnect-interface-0 \
  --region=us-west1 \
  --peer-ip-address=169.254.1.1 \
  --ip-address=169.254.1.2

# View Interconnect status
gcloud compute interconnects describe my-interconnect
gcloud compute interconnects attachments describe my-attachment --region=us-west1

# View learned routes (from on-prem)
gcloud compute routers get-status my-cloud-router --region=us-west1
```

**Exam tips:**
> - "Need 100 Gbps bandwidth to Google Cloud?" --> **Dedicated Interconnect**
> - "Don't have presence in Google colocation facility?" --> **Partner Interconnect** or **Cloud VPN**
> - "Cost-sensitive, low bandwidth (<1 Gbps)?" --> **Cloud VPN**
> - "Need encryption without application-layer encryption?" --> **Cloud VPN** (IPsec encrypted)
> - "Migrating 500 TB of data, need consistent high throughput?" --> **Dedicated Interconnect** (or Transfer Appliance)
> - "High availability requirement?" --> Use **dual connections** for any Interconnect option (meets 99.99% SLA)
> - **Cloud Router is required** for both Dedicated and Partner Interconnect to exchange routes via BGP
> - Both Interconnect options are **not encrypted** by default -- use VPN over Interconnect or app-layer encryption if needed

**Docs:** [Cloud Interconnect overview](https://cloud.google.com/interconnect/docs/concepts/overview) | [Dedicated Interconnect](https://cloud.google.com/interconnect/docs/concepts/dedicated-overview) | [Partner Interconnect](https://cloud.google.com/interconnect/docs/concepts/partner-overview) | [Cloud VPN](https://cloud.google.com/vpn/docs) | [Choosing a connectivity option](https://cloud.google.com/architecture/best-practices-vpc-design#choosing-connectivity)

---

## 2.4 Planning IaC and Organization Bootstrapping

### Fabric FAST (Cloud Foundation Fabric)

**Fabric FAST** (Fabric Automation for Scalable Topologies) is a **Terraform-based framework** from Google's **Cloud Foundation Fabric** team for **bootstrapping GCP organizations** with opinionated, production-ready infrastructure.

**What it is:**
- **Open-source Terraform modules** maintained by Google Cloud
- Provides a **reference architecture** for landing zones, org hierarchy, IAM, networking, security, logging, and more
- **Opinionated best practices** based on Google's Cloud Foundation Toolkit (CFT) and years of enterprise customer experience
- Designed for **greenfield GCP organization setup** -- setting up a new GCP org from scratch
- **Stages:** Resource hierarchy --> Shared services --> Networking --> Security --> Projects --> Workloads

**Key features:**
- **Modular design:** Each stage is a separate Terraform module that builds on the previous stage
- **Hierarchical IAM:** Defines roles and permissions at org, folder, and project levels
- **Networking best practices:** VPC design, Shared VPC, hub-and-spoke, firewall policies
- **Security controls:** Organization policies, Cloud Armor, VPC Service Controls, logging sinks
- **GitOps-friendly:** Designed to be version-controlled and managed via CI/CD pipelines
- **Customizable:** Modules can be extended or replaced based on your requirements

**Stages in Fabric FAST:**

| Stage | Purpose | What It Provisions |
|---|---|---|
| **0-bootstrap** | Initial setup | Org-level IAM, Terraform state bucket, CI/CD foundation |
| **1-organization** | Resource hierarchy | Folders (e.g., Prod, Dev, Shared), org policies, logging sinks, IAM roles |
| **2-networking** | Networking foundation | VPCs, Shared VPC, subnets, Cloud Interconnect/VPN, firewall policies |
| **2-security** | Security foundation | VPC Service Controls, Cloud Armor, Security Command Center, KMS keys |
| **3-project-factory** | Project creation | Automated project creation with predefined configurations |
| **4-workloads** (optional) | Application deployment | Example workloads (GKE, Cloud Run, Compute Engine) |

**Relationship to Cloud Foundation Toolkit (CFT):**
- **Cloud Foundation Toolkit (CFT):** Broader collection of Terraform modules for GCP (e.g., `terraform-google-modules/*`)
- **Fabric FAST:** Part of the **Cloud Foundation Fabric** project, which builds on CFT with a full end-to-end framework
- Think of CFT as **individual Lego bricks** (modules for VPCs, IAM, GKE, etc.) and Fabric FAST as a **complete Lego blueprint** (how to assemble them into a production-ready org)

**When to use Fabric FAST:**
- **Greenfield GCP organization setup** -- you're starting from scratch and want Google's best practices
- You need a **landing zone** (foundational infrastructure for hosting workloads)
- You want **opinionated, tested configurations** instead of building from scratch
- You're setting up a **multi-environment architecture** (dev, staging, prod) with centralized controls
- You want **IaC (Terraform) from day 1** for repeatability and version control
- You need **compliance-ready configurations** (logging, security, governance)

**When NOT to use Fabric FAST:**
- **Existing GCP org with established patterns** -- Fabric FAST is opinionated and may conflict with your current setup
- **Small, simple projects** -- overkill for a single project or small team
- **Non-Terraform environments** -- if you're using other IaC tools (Pulumi, Ansible, etc.)
- **Highly customized requirements** -- Fabric FAST is opinionated; heavy customization may be harder than building from scratch

**Example workflow:**

```bash
# Clone the Cloud Foundation Fabric repo
git clone https://github.com/GoogleCloudPlatform/cloud-foundation-fabric.git
cd cloud-foundation-fabric/fast

# Stage 0: Bootstrap (set up Terraform state bucket, org-level IAM)
cd 0-bootstrap
terraform init
terraform plan -var="organization_id=123456789012" -var="billing_account_id=ABCDEF-123456-GHIJKL"
terraform apply

# Stage 1: Organization (create folders, org policies, logging)
cd ../1-organization
terraform init
terraform plan
terraform apply

# Stage 2: Networking (create VPCs, Shared VPC, firewall policies)
cd ../2-networking
terraform init
terraform plan
terraform apply

# Stage 3: Project Factory (automate project creation)
cd ../3-project-factory
terraform init
terraform plan -var="project_name=my-prod-project" -var="folder_id=folders/456789012345"
terraform apply
```

**Key concepts provisioned by Fabric FAST:**

1. **Resource Hierarchy:**
   - Org --> Folders (Prod, Dev, Shared, Security, etc.) --> Projects
   - Centralized IAM roles (org admins, folder admins, security team)

2. **Networking:**
   - Hub-and-spoke VPC architecture with Shared VPC
   - Firewall policies (hierarchical Cloud NGFW)
   - Cloud Interconnect or VPN for hybrid connectivity

3. **Security:**
   - Organization policies (e.g., restrict public IPs, require OS Login, enforce VPC Service Controls)
   - Centralized logging to a dedicated logging project
   - VPC Service Controls for data perimeter protection
   - KMS keys for encryption

4. **Logging and Monitoring:**
   - Log sinks aggregating logs from all projects to a centralized project
   - Cloud Monitoring dashboards and alerting

5. **CI/CD:**
   - GitOps setup with Cloud Build triggers for Terraform automation
   - Service accounts with least-privilege IAM for Terraform execution

**Comparison: Fabric FAST vs Manual Setup vs Other IaC**

| Aspect | Manual Console Setup | Fabric FAST | Custom Terraform from Scratch |
|---|---|---|---|
| **Time to production** | Weeks-months | Days-weeks | Weeks-months |
| **Best practices** | Up to you | **Opinionated, Google-recommended** | Up to you |
| **Repeatability** | Low (manual steps) | **High** (IaC) | High (IaC) |
| **Complexity** | High (many services) | **Moderate** (modules abstract complexity) | High (learn everything) |
| **Customization** | Full | Moderate (extend modules) | Full |
| **Learning curve** | Low (Console UI) | **Moderate** (understand module structure) | High (learn all GCP Terraform resources) |
| **Use case** | POCs, small projects | **Production orgs, landing zones** | Custom requirements not covered by FAST |

**Exam tips:**
> - "Need to bootstrap a new GCP organization following Google best practices?" --> **Fabric FAST**
> - "Need reusable Terraform modules for individual GCP services (VPC, GKE, IAM)?" --> **Cloud Foundation Toolkit (CFT)**
> - "Want opinionated, end-to-end org setup from scratch?" --> **Fabric FAST**
> - "Need custom IaC for existing org with established patterns?" --> Build custom Terraform or use CFT modules
> - Fabric FAST is **not a managed service** -- it's an open-source framework you run yourself
> - Fabric FAST is **stage-based** -- you progress through stages (bootstrap --> org --> networking --> security)

**Docs:** [Cloud Foundation Fabric (GitHub)](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric) | [Fabric FAST documentation](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast) | [Cloud Foundation Toolkit (CFT)](https://cloud.google.com/foundation-toolkit) | [Landing zone best practices](https://cloud.google.com/architecture/landing-zones)

---

## Quick-Fire Exam Tips for Domain 2

### Compute Cheat Sheet
- **Lift-and-shift legacy app** --> Compute Engine
- **Microservices needing Kubernetes** --> GKE
- **Stateless container, no infra mgmt** --> Cloud Run
- **Event-triggered function** --> Cloud Functions
- **Fault-tolerant batch job, cost-sensitive** --> Spot VMs
- **Workload between predefined sizes** --> Custom machine type
- **Serverless on existing GKE cluster** --> Knative Serving

### Storage Cheat Sheet
- **Relational + single region** --> Cloud SQL
- **Relational + global** --> Cloud Spanner
- **Document DB + mobile sync** --> Firestore
- **Wide-column + massive throughput** --> Bigtable
- **Data warehouse + analytics** --> BigQuery
- **Object storage (files, images, backups)** --> Cloud Storage
- **HA block storage** --> Regional Persistent Disk
- **Ultra-high IOPS, independent of disk size** --> Hyperdisk Extreme
- **High sequential throughput (ML, analytics)** --> Hyperdisk Throughput

### Network Cheat Sheet
- **Global HTTP/S LB** --> External Application LB (Premium Tier)
- **Internal HTTP routing** --> Internal Application LB
- **Non-HTTP encrypted traffic** --> SSL Proxy (External Proxy Network LB)
- **TCP without SSL, global** --> TCP Proxy (External Proxy Network LB)
- **UDP or need to preserve source IP** --> Passthrough Network LB
- **Lower cost egress, regional only** --> Standard Tier
- **Global LB, CDN, Armor** --> Premium Tier (required)
- **Org-wide firewall policies, FQDN/geo rules** --> Cloud NGFW
- **Block traffic by country** --> Cloud NGFW (geo-filtering)
- **Prevent unauthorized tag changes** --> Secure Tags (not network tags)
- **50+ Gbps private connectivity to GCP** --> Dedicated Interconnect
- **Private connectivity without colocation** --> Partner Interconnect
- **Low bandwidth, encrypted, quick setup** --> Cloud VPN

### IaC and Organization Cheat Sheet
- **Bootstrap new GCP org with best practices** --> Fabric FAST
- **Reusable Terraform modules for individual services** --> Cloud Foundation Toolkit (CFT)
- **Need landing zone for production workloads** --> Fabric FAST

### Common Exam Traps

1. **Spot VMs have no max runtime** -- Preemptible VMs had a 24-hour limit. Spot VMs do not. Both can be reclaimed at any time.
2. **Archive storage is NOT slow** -- All Cloud Storage classes have millisecond access latency. The difference is cost structure, not speed.
3. **VPCs are global; subnets are regional** -- You do not need a VPC per region.
4. **Standard Tier cannot use global load balancing** -- If the question mentions global LB, the answer requires Premium Tier.
5. **Regional PD costs 2x zonal PD** -- It replicates to 2 zones. Know the tradeoff.
6. **Custom machine types have a 5% premium** -- They cost slightly more per resource unit than predefined types.
7. **Cloud Spanner = global strong consistency** -- This is its defining feature over Cloud SQL.
8. **Bigtable is not cost-effective under 1 TB** -- If data is small, consider Firestore or Cloud SQL instead.
9. **Cloud Run scales to zero; GKE does not** -- This matters for cost optimization questions.
10. **Cloud Functions 2nd gen is built on Cloud Run** -- It supports longer timeouts (up to 60 min) and concurrency.
11. **Hyperdisk decouples IOPS from disk size** -- You can provision high IOPS on a small disk, unlike standard PDs.
12. **Network tags are not IAM-controlled** -- Use Secure Tags for production security governance.
13. **Cloud NGFW is additive** -- It works alongside standard VPC firewall rules, not instead of them.
14. **Dedicated Interconnect requires colocation at Google PoP** -- If you can't collocate, use Partner Interconnect or Cloud VPN.
15. **Cloud Run is built on Knative** -- Knative on GKE gives you similar features with full Kubernetes control.

---

## Key Documentation Links

| Topic | URL |
|---|---|
| Compute Engine | https://cloud.google.com/compute/docs |
| GKE | https://cloud.google.com/kubernetes-engine/docs |
| Cloud Run | https://cloud.google.com/run/docs |
| Cloud Functions | https://cloud.google.com/functions/docs |
| Knative | https://knative.dev/ |
| GKE with Knative | https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-knative |
| Spot VMs | https://cloud.google.com/compute/docs/instances/spot |
| Custom Machine Types | https://cloud.google.com/compute/docs/instances/creating-instance-with-custom-machine-type |
| Machine Families Guide | https://cloud.google.com/compute/docs/machine-resource |
| Cloud SQL | https://cloud.google.com/sql/docs |
| Cloud Spanner | https://cloud.google.com/spanner/docs |
| Firestore | https://cloud.google.com/firestore/docs |
| Bigtable | https://cloud.google.com/bigtable/docs |
| BigQuery | https://cloud.google.com/bigquery/docs |
| Persistent Disk | https://cloud.google.com/compute/docs/disks/persistent-disks |
| Hyperdisk | https://cloud.google.com/compute/docs/disks/hyperdisks |
| Cloud Storage Classes | https://cloud.google.com/storage/docs/storage-classes |
| Load Balancing Overview | https://cloud.google.com/load-balancing/docs/load-balancing-overview |
| Choose a Load Balancer | https://cloud.google.com/load-balancing/docs/choosing-load-balancer |
| Network Service Tiers | https://cloud.google.com/network-tiers/docs/overview |
| VPC Overview | https://cloud.google.com/vpc/docs/vpc |
| Cloud NGFW | https://cloud.google.com/firewall/docs/about-firewall-policies |
| Hierarchical Firewall Policies | https://cloud.google.com/firewall/docs/firewall-policies |
| Secure Tags (Resource Manager Tags) | https://cloud.google.com/resource-manager/docs/tags/tags-overview |
| Network Tags | https://cloud.google.com/vpc/docs/add-remove-network-tags |
| Cloud Interconnect Overview | https://cloud.google.com/interconnect/docs/concepts/overview |
| Dedicated Interconnect | https://cloud.google.com/interconnect/docs/concepts/dedicated-overview |
| Partner Interconnect | https://cloud.google.com/interconnect/docs/concepts/partner-overview |
| Cloud VPN | https://cloud.google.com/vpn/docs |
| Fabric FAST | https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast |
| Cloud Foundation Toolkit | https://cloud.google.com/foundation-toolkit |
| ACE Exam Guide | https://cloud.google.com/learn/certification/guides/cloud-engineer |
