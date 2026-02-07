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

## Quick-Fire Exam Tips for Domain 2

### Compute Cheat Sheet
- **Lift-and-shift legacy app** --> Compute Engine
- **Microservices needing Kubernetes** --> GKE
- **Stateless container, no infra mgmt** --> Cloud Run
- **Event-triggered function** --> Cloud Functions
- **Fault-tolerant batch job, cost-sensitive** --> Spot VMs
- **Workload between predefined sizes** --> Custom machine type

### Storage Cheat Sheet
- **Relational + single region** --> Cloud SQL
- **Relational + global** --> Cloud Spanner
- **Document DB + mobile sync** --> Firestore
- **Wide-column + massive throughput** --> Bigtable
- **Data warehouse + analytics** --> BigQuery
- **Object storage (files, images, backups)** --> Cloud Storage
- **HA block storage** --> Regional Persistent Disk

### Network Cheat Sheet
- **Global HTTP/S LB** --> External Application LB (Premium Tier)
- **Internal HTTP routing** --> Internal Application LB
- **Non-HTTP encrypted traffic** --> SSL Proxy (External Proxy Network LB)
- **TCP without SSL, global** --> TCP Proxy (External Proxy Network LB)
- **UDP or need to preserve source IP** --> Passthrough Network LB
- **Lower cost egress, regional only** --> Standard Tier
- **Global LB, CDN, Armor** --> Premium Tier (required)

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

---

## Key Documentation Links

| Topic | URL |
|---|---|
| Compute Engine | https://cloud.google.com/compute/docs |
| GKE | https://cloud.google.com/kubernetes-engine/docs |
| Cloud Run | https://cloud.google.com/run/docs |
| Cloud Functions | https://cloud.google.com/functions/docs |
| Spot VMs | https://cloud.google.com/compute/docs/instances/spot |
| Custom Machine Types | https://cloud.google.com/compute/docs/instances/creating-instance-with-custom-machine-type |
| Machine Families Guide | https://cloud.google.com/compute/docs/machine-resource |
| Cloud SQL | https://cloud.google.com/sql/docs |
| Cloud Spanner | https://cloud.google.com/spanner/docs |
| Firestore | https://cloud.google.com/firestore/docs |
| Bigtable | https://cloud.google.com/bigtable/docs |
| BigQuery | https://cloud.google.com/bigquery/docs |
| Persistent Disk | https://cloud.google.com/compute/docs/disks/persistent-disks |
| Cloud Storage Classes | https://cloud.google.com/storage/docs/storage-classes |
| Load Balancing Overview | https://cloud.google.com/load-balancing/docs/load-balancing-overview |
| Choose a Load Balancer | https://cloud.google.com/load-balancing/docs/choosing-load-balancer |
| Network Service Tiers | https://cloud.google.com/network-tiers/docs/overview |
| VPC Overview | https://cloud.google.com/vpc/docs/vpc |
| ACE Exam Guide | https://cloud.google.com/learn/certification/guides/cloud-engineer |
