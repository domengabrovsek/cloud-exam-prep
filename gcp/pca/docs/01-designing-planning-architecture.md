# Section 1: Designing and Planning a Cloud Solution Architecture (~25% of Exam)

> **Quick context:** This is the highest-weighted section on the PCA exam. It tests your ability to translate business and technical requirements into cloud architectures, design for reliability and performance, plan migrations, and envision future improvements. Expect 12-15 questions.

---

## 1.1 Designing a Cloud Solution Infrastructure That Meets Business Requirements

### Business Use Cases and Product Strategy

Architects must translate business goals into technical architecture. The exam tests whether you can:
- Map revenue goals to scalability requirements
- Align product roadmaps with infrastructure choices
- Balance time-to-market against long-term maintainability

**Key patterns:**

| Business Goal | Architecture Implication |
|--------------|------------------------|
| Rapid scaling | Serverless (Cloud Run), autoscaling MIGs, GKE Autopilot |
| Global reach | Multi-region deployment, Cloud CDN, global load balancing |
| Cost reduction | Managed services, Spot VMs, autoscaling to zero |
| Compliance | Assured Workloads, regional data residency, CMEK |
| Innovation speed | Microservices, CI/CD with Cloud Build + Cloud Deploy |
| Data-driven decisions | BigQuery, Looker, Vertex AI |

**Exam tips:**
- The exam favors managed/serverless services over self-managed infrastructure
- "Minimize operational overhead" = use managed services
- "Fastest time to market" = use pre-built APIs and serverless

**Docs:** [Cloud Architecture Center](https://cloud.google.com/architecture)

---

### Identifying Functional and Non-functional Requirements

**Functional requirements:** What the system must DO (features, integrations, data processing)

**Non-functional requirements (NFRs):** How the system must PERFORM -- these drive architecture decisions:

| NFR | Metric | Architecture Impact |
|-----|--------|-------------------|
| Availability | 99.9% vs 99.99% vs 99.999% | Single-region vs multi-region, HA configurations |
| Latency | p50, p95, p99 response time | CDN, edge caching, regional deployment, Premium Tier |
| Throughput | Requests/sec, messages/sec | Horizontal scaling, Pub/Sub, Dataflow |
| Durability | Data loss tolerance | Multi-region storage, cross-region replication |
| Scalability | Peak vs baseline ratio | Autoscaling policies, serverless patterns |
| Compliance | HIPAA, GDPR, PCI DSS | Assured Workloads, VPC-SC, CMEK, audit logs |

**Availability math for the exam:**

| Target | Downtime/Year | Downtime/Month | Architecture |
|--------|--------------|----------------|-------------|
| 99% | 3.65 days | 7.3 hours | Single zone, basic |
| 99.9% | 8.76 hours | 43.8 minutes | Multi-zone, regional resources |
| 99.99% | 52.6 minutes | 4.38 minutes | Multi-region, active-passive |
| 99.999% | 5.26 minutes | 26.3 seconds | Multi-region active-active, global LB |

**Composite SLA calculation:**
- Services in series (both must work): multiply SLAs → 99.9% × 99.9% = 99.8%
- Services in parallel (either works): 1 - (1-SLA₁)(1-SLA₂) → 1 - (0.001)(0.001) = 99.9999%

**Exam tips:**
- "Design for 99.99% availability" → multi-region deployment with failover
- When a question mentions specific latency targets, think about data locality and caching
- Composite SLA is a common exam calculation -- practice multiplying SLAs

**Docs:** [Architecture Framework: System Design](https://cloud.google.com/architecture/framework/system-design)

---

### Business Continuity Plan

**BCP vs DR:**
- **BCP** (Business Continuity Planning): Broad -- covers all threats to business operations (natural disasters, personnel, supply chain, cyber attacks)
- **DR** (Disaster Recovery): Subset of BCP -- specifically about restoring IT systems after a disruption

**Key metrics:**
- **RPO** (Recovery Point Objective): Maximum acceptable data loss, measured in time. "How much data can we afford to lose?"
- **RTO** (Recovery Time Objective): Maximum acceptable downtime. "How fast must we recover?"

| RPO/RTO Target | Strategy | Cost | GCP Services |
|----------------|----------|------|-------------|
| RPO: hours, RTO: days | Cold standby | $ | Backups in GCS, Cloud SQL automated backups |
| RPO: minutes, RTO: hours | Warm standby | $$ | Cross-region read replicas, pre-configured infra |
| RPO: seconds, RTO: minutes | Hot standby | $$$ | Multi-region active-passive, Spanner multi-region |
| RPO: ~0, RTO: ~0 | Active-active | $$$$ | Multi-region active-active, global LB, Spanner |

**Exam tips:**
- RPO drives your backup/replication strategy; RTO drives your recovery infrastructure
- "Minimize data loss" = low RPO = continuous replication
- "Minimize downtime" = low RTO = hot standby or active-active
- Lower RPO/RTO = higher cost -- the exam may ask you to find the MOST COST-EFFECTIVE option that still meets requirements

**Docs:** [Disaster Recovery Planning Guide](https://cloud.google.com/architecture/dr-scenarios-planning-guide)

---

### Cost Optimization

**Committed Use Discounts (CUDs):**
- 1-year or 3-year commitments for predictable workloads
- **Resource-based CUDs**: Commit to specific vCPU/memory amounts (Compute Engine, GKE)
  - 1-year: ~37% discount, 3-year: ~55% discount
- **Spend-based CUDs**: Commit to $/hour spend (Cloud SQL, AlloyDB, Cloud Run, BigQuery editions)
  - 1-year: ~25% discount, 3-year: ~52% discount
- CUDs apply automatically across projects in the same billing account

```bash
# List active commitments
gcloud compute commitments list

# Create a resource-based CUD
gcloud compute commitments create my-commitment \
  --region=us-central1 \
  --resources=vcpu=100,memory=400GB \
  --plan=36-month

# Spend-based CUDs are managed in Cloud Console > Billing > Commitments
```

**Sustained Use Discounts (SUDs):**
- Automatic discounts for running resources 25%+ of a month
- Up to ~30% discount at full-month usage
- Apply to N1, N2, N2D machine types and sole-tenant nodes
- **Do NOT apply to:** E2, T2D, Tau, C2, C2D, A2, Spot VMs, or committed-use resources
- Applied automatically per billing account per region

**Other cost levers:**

| Strategy | Savings | Use Case |
|----------|---------|----------|
| Spot VMs | 60-91% | Fault-tolerant batch, CI/CD workers, data processing |
| Right-sizing (Recommender) | 10-40% | Over-provisioned VMs |
| Autoscaling to zero | Variable | Cloud Run, GKE Autopilot |
| Preemptible/Spot node pools | 60-91% | GKE batch workloads |
| BigQuery flat-rate (editions) | Predictable | High-volume analytics |
| Cloud Storage Autoclass | Variable | Unknown access patterns |
| Active Assist | Variable | AI-driven recommendations across services |

```bash
# Get right-sizing recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1-a

# Apply a recommendation
gcloud recommender recommendations mark-claimed RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1-a
```

**Exam tips:**
- "Minimize cost for predictable workloads" → CUDs
- "Reduce cost for fault-tolerant batch" → Spot VMs
- "Automatic cost optimization without commitment" → SUDs (if eligible machine type)
- SUDs do NOT apply to E2 machines -- this is an exam trap
- CUDs are NOT refundable -- they're a commitment
- Active Assist includes Recommender, which covers VM right-sizing, idle resource detection, and IAM recommendations

**Docs:** [Cost Management](https://cloud.google.com/cost-management/docs), [CUDs](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview), [SUDs](https://cloud.google.com/compute/docs/sustained-use-discounts)

---

### Supporting the Application Design

**Microservices vs Monolith:**

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | All-or-nothing | Independent per service |
| Scaling | Entire app scales together | Per-service scaling |
| Technology | Single stack | Polyglot possible |
| Complexity | Lower initially | Higher (networking, observability) |
| Team structure | Single team | Multiple autonomous teams |
| GCP mapping | Compute Engine, App Engine | GKE, Cloud Run |

**12-Factor App Principles (exam-relevant ones):**
1. **Codebase**: One repo per service → Cloud Source Repositories, GitHub
2. **Dependencies**: Explicitly declare → Artifact Registry
3. **Config**: Store in environment → Secret Manager, ConfigMaps
4. **Backing services**: Treat as attached resources → Cloud SQL, Pub/Sub, Memorystore
5. **Build/release/run**: Strict separation → Cloud Build + Cloud Deploy
6. **Processes**: Stateless → Cloud Run, GKE pods
7. **Port binding**: Self-contained → Cloud Run, GKE Services
8. **Concurrency**: Scale via processes → HPA, Cloud Run instances
9. **Disposability**: Fast startup/shutdown → Containers, Cloud Run
10. **Dev/prod parity**: Keep environments similar → Terraform, Config Sync
11. **Logs**: Treat as event streams → Cloud Logging
12. **Admin processes**: Run as one-off → Cloud Run jobs, CronJobs

**Event-driven architecture:**
- **Pub/Sub**: Async messaging, at-least-once delivery, dead letter topics
- **Eventarc**: Event routing from 100+ sources to Cloud Run, GKE, Workflows
- **Cloud Tasks**: Task queues with rate limiting, retry, scheduling
- **Workflows**: Orchestration of multi-step serverless workflows

**Exam tips:**
- "Loosely coupled" = microservices + Pub/Sub/Eventarc
- "Stateless" = Cloud Run or GKE with external state (Memorystore, Cloud SQL)
- "Independent deployment" = microservices on Cloud Run or GKE
- The exam expects you to choose event-driven patterns for async processing

**Docs:** [Microservices Architecture](https://cloud.google.com/architecture/microservices-architecture-introduction), [Event-Driven Architecture](https://cloud.google.com/eventarc/docs/overview)

---

### Integration Patterns with External Systems

| Pattern | GCP Service | Use Case |
|---------|------------|----------|
| API Gateway | Apigee | External partner APIs, rate limiting, monetization |
| Simple API proxy | API Gateway | Serverless backend exposure |
| Async messaging | Pub/Sub | Decoupled systems, event streaming |
| Task queue | Cloud Tasks | Rate-limited API calls, deferred processing |
| Workflow orchestration | Workflows | Multi-step integrations, error handling |
| Data integration | Datastream | CDC from databases to BigQuery |
| Batch data sync | Storage Transfer Service | Scheduled data transfers |
| Real-time sync | Pub/Sub + Dataflow | Stream processing pipelines |

**Exam tips:**
- "Rate limit external API calls" → Cloud Tasks
- "Decouple producers and consumers" → Pub/Sub
- "Orchestrate multiple API calls with error handling" → Workflows
- "Enterprise API management with analytics" → Apigee

**Docs:** [Integration Patterns](https://cloud.google.com/architecture/integration-patterns)

---

### Movement of Data

| Tool | Data Type | Direction | Size |
|------|-----------|-----------|------|
| Storage Transfer Service | Objects | Cloud-to-cloud, on-prem-to-cloud | Any |
| Transfer Appliance | Any | On-prem to cloud | 100 TB - 1 PB |
| BigQuery Data Transfer Service | Structured | SaaS to BigQuery | Any |
| Database Migration Service (DMS) | Databases | On-prem/cloud to Cloud SQL, AlloyDB | Any |
| Datastream | CDC events | Database to BigQuery, GCS | Continuous |
| gcloud storage cp / gsutil | Objects | Any | < 100 TB |
| Migrate to VMs | VM images | On-prem to CE | Any |
| Migrate to Containers | VMs | On-prem to GKE | Any |

**Decision guide:**
- **< 10 TB, good network**: `gcloud storage cp` or Storage Transfer Service
- **10-100 TB, good network**: Storage Transfer Service (parallel, resumable)
- **> 100 TB or slow network**: Transfer Appliance
- **Database migration**: DMS (supports MySQL, PostgreSQL, SQL Server, Oracle)
- **Continuous replication**: Datastream (CDC)
- **SaaS data to BQ**: BigQuery Data Transfer Service

```bash
# Create a transfer job
gcloud transfer jobs create \
  gs://source-bucket gs://destination-bucket \
  --name=my-transfer

# Order a Transfer Appliance (done via Console)
# Console > Transfer Appliance > Create order
```

**Exam tips:**
- Transfer Appliance is for massive offline transfers when network bandwidth is insufficient
- DMS supports continuous replication (CDC) for minimal-downtime migrations
- Datastream is for ongoing CDC, not one-time migration
- Storage Transfer Service can also transfer from AWS S3 and Azure Blob

**Docs:** [Data Transfer](https://cloud.google.com/storage-transfer/docs), [Transfer Appliance](https://cloud.google.com/transfer-appliance/docs), [DMS](https://cloud.google.com/database-migration/docs)

---

### Design Decision Trade-offs

The PCA exam frequently asks you to evaluate trade-offs. Key axes:

| Trade-off | Option A | Option B | How Exam Tests It |
|-----------|----------|----------|-------------------|
| Consistency vs Availability | Spanner (strong consistency) | Firestore (eventual in multi-region) | "Requires globally consistent reads" |
| Cost vs Performance | Standard Tier networking | Premium Tier networking | "Lowest latency for global users" |
| Managed vs Self-managed | Cloud SQL | MySQL on CE | "Minimize operational overhead" |
| Flexibility vs Simplicity | GKE Standard | GKE Autopilot | "Full control over nodes" vs "minimize management" |
| Latency vs Cost | Regional resources | Multi-region | "Users in single region" vs "global users" |
| Durability vs Cost | Multi-region GCS | Regional GCS | "11 9s durability" vs "cost-effective" |

**The exam's general preference hierarchy:**
1. Managed > Self-managed
2. Serverless > Server-based
3. Global > Regional (when global users exist)
4. Autoscaling > Fixed capacity
5. Least privilege > Convenience

**Exam tips:**
- When two options both work technically, choose the one with less operational overhead
- "Most cost-effective" questions often have a constraint you must identify first

**Docs:** [Architecture Decision Records](https://cloud.google.com/architecture/architecture-decision-records)

---

### Workload Disposition Strategies

When migrating to cloud, each workload needs a disposition strategy (the "6 R's"):

| Strategy | Description | When to Use | Effort |
|----------|------------|-------------|--------|
| **Rehost** (lift-and-shift) | Move as-is to VMs | Legacy apps, quick migration | Low |
| **Replatform** (lift-and-reshape) | Minor changes for cloud benefits | Database migrations (MySQL → Cloud SQL) | Medium |
| **Refactor** (re-architect) | Rewrite for cloud-native | Strategic apps, microservices | High |
| **Repurchase** | Replace with SaaS/managed | CRM → Salesforce, email → Workspace | Medium |
| **Retire** | Decommission | Unused or redundant systems | None |
| **Retain** | Keep on-premises | Compliance, latency, not ready | None |

**Additional architect-level strategies:**
- **Build**: Create new cloud-native solution from scratch
- **Buy**: Purchase third-party SaaS
- **Modify**: Adapt existing system with targeted changes
- **Deprecate**: Phase out gradually with replacement timeline

**Exam tips:**
- "Fastest migration with minimal changes" → Rehost
- "Take advantage of managed databases" → Replatform
- "Maximize cloud-native benefits" → Refactor
- The exam often asks you to identify the MINIMUM effort strategy that meets requirements
- Not all workloads should be refactored -- the exam values pragmatic choices

**Docs:** [Migration to Google Cloud](https://cloud.google.com/architecture/migration-to-gcp-getting-started)

---

### Success Measurements

**KPIs for cloud architecture:**

| Category | KPI | Tool |
|----------|-----|------|
| Reliability | Availability (% uptime) | Cloud Monitoring uptime checks |
| Performance | Latency (p50, p95, p99) | Cloud Trace, Cloud Monitoring |
| Cost | Cost per transaction/user | Billing exports to BigQuery |
| Efficiency | Resource utilization | Recommender, Cloud Monitoring |
| Velocity | Deployment frequency | Cloud Deploy metrics |
| Quality | Error rate, change failure rate | Error Reporting, Cloud Monitoring |

**SRE metrics (DORA):**
- **Deployment frequency**: How often you deploy
- **Lead time for changes**: Code commit to production
- **Change failure rate**: % of deployments causing incidents
- **Time to restore service**: MTTR

**Exam tips:**
- "Measure success of cloud migration" → define SLOs before migration, compare after
- "Track cost efficiency" → billing exports to BigQuery with labels

**Docs:** [SRE Principles](https://sre.google/sre-book/table-of-contents/)

---

### Security and Compliance (as Architecture Design)

Security must be built into architecture from the start, not bolted on:

**Security-by-design principles:**
- **Defense in depth**: Multiple layers (network, identity, data, application)
- **Zero trust**: Never trust, always verify. Authenticate and authorize every request.
- **Least privilege**: Grant minimum permissions needed
- **Separation of duties**: No single person has complete control

**Architecture patterns:**
- Use VPC Service Controls to create security perimeters around sensitive data
- Use Private Google Access and Private Service Connect to avoid public internet
- Use CMEK for data sovereignty and key management control
- Use Binary Authorization to ensure only trusted containers run in GKE

**Exam tips:**
- "Prevent data exfiltration" → VPC Service Controls
- "Ensure only approved containers" → Binary Authorization
- Cross-reference to Section 3 (Security and Compliance) for full details

**Docs:** [Security Best Practices](https://cloud.google.com/security/best-practices)

---

### Observability (Designing for Monitorability)

Design systems to be observable from day one:

**The Four Golden Signals (from Google SRE):**
1. **Latency**: Time to serve a request (distinguish successful vs failed)
2. **Traffic**: Demand on the system (requests/sec, sessions)
3. **Errors**: Rate of failed requests (HTTP 5xx, exceptions)
4. **Saturation**: How full the system is (CPU, memory, disk, queue depth)

**Observability stack:**

| Signal | GCP Service | Implementation |
|--------|------------|---------------|
| Metrics | Cloud Monitoring | Custom metrics, Prometheus, MQL |
| Logs | Cloud Logging | Structured logs, log sinks, Log Analytics |
| Traces | Cloud Trace | Distributed tracing across services |
| Errors | Error Reporting | Automatic error grouping and alerting |
| Profiles | Cloud Profiler | CPU, memory, heap profiling |

**Design principles:**
- Emit structured logs (JSON) for machine parsing
- Propagate trace context across service boundaries
- Use SLO-based alerting instead of threshold-based
- Define SLIs before building -- they shape instrumentation

**Exam tips:**
- "How to identify the root cause of increased latency" → Cloud Trace
- "How to monitor across microservices" → distributed tracing + structured logging
- "Alert on user experience degradation" → SLO-based burn rate alerts

**Docs:** [Cloud Operations Suite](https://cloud.google.com/products/operations), [Four Golden Signals](https://sre.google/sre-book/monitoring-distributed-systems/)

---

## 1.2 Designing a Cloud Solution Infrastructure That Meets Technical Requirements

### Well-Architected Framework Alignment

Every architecture decision on the PCA exam should be evaluated against the WAF pillars:

1. **Operational Excellence**: Can you monitor, deploy, and maintain it?
2. **Security**: Is data protected? Is access least-privilege?
3. **Reliability**: Does it survive failures? What's the RPO/RTO?
4. **Cost Optimization**: Is it right-sized? Are you using discounts?
5. **Performance**: Does it meet latency/throughput requirements?
6. **Sustainability**: Is it efficient in resource use?

**How to use WAF in exam questions:**
- When choosing between two valid architectures, evaluate which one better satisfies the WAF pillar emphasized in the question
- "Most reliable" → Reliability pillar
- "Most cost-effective" → Cost Optimization pillar
- "Most secure" → Security pillar

See [07-well-architected-framework.md](07-well-architected-framework.md) for the full WAF reference.

**Docs:** [Well-Architected Framework](https://cloud.google.com/architecture/framework)

---

### High Availability and Failover Design

**Failure domains on GCP:**

```
Zone (single data center)
  └── Region (multiple zones, ~10ms latency between zones)
       └── Multi-region (multiple regions, higher latency)
```

**HA configurations per service:**

| Service | Zonal (single zone) | Regional (multi-zone) | Multi-region |
|---------|--------------------|-----------------------|-------------|
| Compute Engine | Single VM | Regional MIG | Multi-region MIG + global LB |
| GKE | Zonal cluster | Regional cluster (control plane + nodes) | Multi-cluster with MCS/MCI |
| Cloud SQL | Single zone | HA with failover replica (same region) | Cross-region read replicas |
| Spanner | Regional (3 zones) | N/A (already multi-zone) | Multi-region (5+ replicas) |
| Cloud Run | Single region | N/A (auto multi-zone in region) | Multi-region with global LB |
| Cloud Storage | Regional | Dual-region | Multi-region |
| Memorystore Redis | Standard (single zone) | Standard (zonal failover) | N/A |

**GKE HA architecture:**

```bash
# Create a regional GKE cluster (HA control plane across 3 zones)
gcloud container clusters create my-cluster \
  --region=us-central1 \
  --num-nodes=2 \
  --enable-autorepair \
  --enable-autoscaling --min-nodes=1 --max-nodes=5

# For multi-region: use Multi Cluster Ingress (MCI)
# Requires GKE Enterprise (fleet-level feature)
```

**Cloud SQL HA:**

```bash
# Create HA Cloud SQL instance
gcloud sql instances create my-instance \
  --database-version=POSTGRES_15 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --availability-type=REGIONAL  # This enables HA

# Create cross-region read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-instance \
  --region=europe-west1
```

**Exam tips:**
- Regional GKE clusters have HA control planes across 3 zones by default
- Cloud SQL `REGIONAL` availability type = HA with automatic failover
- Spanner regional config is already multi-zone (3 zones) -- you don't need to configure HA separately
- Cloud Run is automatically distributed across zones within a region
- "99.999% availability" typically requires multi-region active-active

**Docs:** [HA and DR](https://cloud.google.com/architecture/dr-scenarios-planning-guide), [Cloud SQL HA](https://cloud.google.com/sql/docs/mysql/high-availability), [GKE Regional Clusters](https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters)

---

### Flexibility of Cloud Resources

**Loose coupling principles:**
- Use Pub/Sub between services for async communication
- Use service accounts per workload (not shared)
- Use managed services to avoid infrastructure lock-in
- Use containerization for portability (GKE, Cloud Run)

**Managed services reduce flexibility concerns:**

| Self-managed | Managed Alternative | Flexibility Gained |
|-------------|--------------------|--------------------|
| MySQL on CE | Cloud SQL for MySQL | Auto-backups, HA, patching |
| Kafka on VMs | Pub/Sub | Serverless, auto-scaling |
| Redis on VMs | Memorystore | Managed HA, patching |
| Prometheus on GKE | Managed Prometheus | No infra management |
| Nginx ingress | Cloud Load Balancing | Global, managed SSL |

**Exam tips:**
- "Reduce operational burden" → managed services
- "Portable across environments" → containers (GKE/Cloud Run)
- "Avoid vendor lock-in" → open-source tools on GKE (Kafka, Redis, Postgres)

**Docs:** [Managed Services](https://cloud.google.com/products)

---

### Scalability to Meet Growth Requirements

**Horizontal scaling options:**

| Service | Scaling Mechanism | Configuration |
|---------|------------------|---------------|
| Compute Engine (MIG) | Autoscaler | CPU, custom metric, LB utilization, schedule |
| GKE pods | HPA (Horizontal Pod Autoscaler) | CPU, memory, custom/external metrics |
| GKE nodes | Cluster Autoscaler | Automatic based on pending pods |
| GKE | Node Auto-Provisioning (NAP) | Auto-creates optimal node pools |
| Cloud Run | Concurrency-based | Min/max instances, concurrency per instance |
| Cloud SQL | Read replicas | Up to 10 read replicas |
| Spanner | Processing units/nodes | Add nodes for more throughput |
| BigQuery | Slots (editions) | Autoscaling slots or reservations |
| Bigtable | Nodes | Add nodes per cluster |

**Vertical scaling:**

| Service | Scaling Mechanism |
|---------|------------------|
| GKE pods | VPA (Vertical Pod Autoscaler) |
| Compute Engine | Machine type change (requires restart) |
| Cloud SQL | Tier change (brief downtime) |

```bash
# Configure MIG autoscaler
gcloud compute instance-groups managed set-autoscaling my-mig \
  --region=us-central1 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=90

# Configure GKE HPA
kubectl autoscale deployment my-app --cpu-percent=70 --min=2 --max=20

# Scale Spanner
gcloud spanner instances update my-instance --processing-units=3000
```

**Exam tips:**
- HPA for pods, Cluster Autoscaler for nodes -- they work together
- VPA and HPA should NOT be used together on the same metric (CPU)
- Cloud Run scales based on concurrent requests, not CPU
- Spanner scales linearly -- 1 node ≈ 10,000 reads/sec or 2,000 writes/sec
- "Predictable growth" → schedule-based autoscaling or CUDs
- "Bursty traffic" → Cloud Run (scales to zero and up quickly)

**Docs:** [Autoscaling](https://cloud.google.com/compute/docs/autoscaler), [GKE Autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler), [Cloud Run Scaling](https://cloud.google.com/run/docs/about-instance-autoscaling)

---

### Performance and Latency

**Caching strategy:**

| Layer | Service | Use Case |
|-------|---------|----------|
| Edge (CDN) | Cloud CDN | Static assets, cacheable API responses |
| Application | Memorystore (Redis/Memcached) | Session data, frequently accessed data |
| Database | BigQuery BI Engine | Accelerated SQL analytics |
| DNS | Cloud DNS | Fast DNS resolution |

**Network performance:**

| Feature | Premium Tier | Standard Tier |
|---------|-------------|---------------|
| Routing | Google's global network (cold potato) | Public internet (hot potato) |
| Global LB | Yes | No (regional only) |
| Latency | Lower (Google backbone) | Higher (public internet) |
| Cost | Higher | Lower (~24% cheaper) |
| SLA | Yes | Limited |

**Performance optimization patterns:**
- Use **Premium Tier** for global users (traffic enters Google's network at closest edge)
- Use **Cloud CDN** for cacheable content (reduces origin load and latency)
- Use **Memorystore** for sub-millisecond data access
- Use **connection pooling** for database connections (Cloud SQL Auth Proxy)
- Place compute close to data (same region as storage/database)

```bash
# Enable Cloud CDN on a backend service
gcloud compute backend-services update my-backend \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC

# Create Memorystore Redis instance
gcloud redis instances create my-cache \
  --region=us-central1 \
  --size=5 \
  --tier=STANDARD_HA
```

**Exam tips:**
- "Lowest latency for global users" → Premium Tier + Cloud CDN + global LB
- "Cost-effective for single-region" → Standard Tier
- "Reduce database load" → caching with Memorystore
- Cloud CDN can cache both static and dynamic content (with appropriate cache headers)
- Cloud SQL Auth Proxy handles connection pooling and IAM auth

**Docs:** [Cloud CDN](https://cloud.google.com/cdn/docs), [Network Service Tiers](https://cloud.google.com/network-tiers/docs), [Memorystore](https://cloud.google.com/memorystore/docs)

---

### Gemini Cloud Assist

Gemini Cloud Assist is Google's AI-powered assistant integrated across Google Cloud:

- **Gemini in Console**: Natural language queries about your infrastructure, troubleshooting suggestions, architecture recommendations
- **Gemini Code Assist**: AI coding assistance in Cloud Shell Editor, Cloud Workstations, and supported IDEs
- **Gemini in BigQuery**: SQL generation, explanation, optimization
- **Gemini in Databases**: Query optimization, schema design
- **Gemini in Security**: Threat analysis, IAM recommendations

**Exam tips:**
- Gemini Cloud Assist is the answer when questions mention "AI-assisted" troubleshooting or recommendations
- It supplements (not replaces) Active Assist and Recommender
- Available across multiple Google Cloud products

**Docs:** [Gemini for Google Cloud](https://cloud.google.com/gemini/docs)

---

### Backup and Recovery

**DR patterns with GCP services:**

| Pattern | RPO | RTO | Architecture | GCP Services |
|---------|-----|-----|-------------|-------------|
| Backup & restore (cold) | Hours | Hours-days | Periodic backups, no standby | Cloud SQL backups, GCS, Spanner backup |
| Pilot light | Minutes-hours | Hours | Core infra pre-provisioned, data replicated | Cross-region replicas, Cloud DNS failover |
| Warm standby | Minutes | Minutes-hours | Scaled-down copy running in DR region | Regional MIG (small), Cloud SQL replica promoted |
| Hot standby | Seconds-minutes | Minutes | Full-scale copy in DR region, traffic ready | Multi-region LB, Cloud SQL HA cross-region |
| Active-active | Near-zero | Near-zero | Both regions serve traffic | Spanner multi-region, global LB, Cloud Run multi-region |

**Per-service backup options:**

| Service | Backup Method | RPO Achievable |
|---------|--------------|----------------|
| Cloud SQL | Automated backups + PITR | Minutes (transaction log) |
| Spanner | Backup (retained up to 1 year) | Hours (backup interval) |
| Spanner multi-region | Built-in replication | Near-zero |
| Firestore | PITR (7-day window) | 1 minute granularity |
| BigQuery | Time travel (7 days) + snapshots | Minutes |
| GCS | Versioning + dual/multi-region | Near-zero (with versioning) |
| GKE | Backup for GKE | Per backup schedule |
| Filestore | Backups (snapshots) | Per backup schedule |

```bash
# Enable automated backups for Cloud SQL with PITR
gcloud sql instances patch my-instance \
  --backup-start-time=02:00 \
  --enable-point-in-time-recovery \
  --retained-transaction-log-days=7

# Create a Spanner backup
gcloud spanner backups create my-backup \
  --instance=my-instance \
  --database=my-db \
  --retention-period=30d \
  --async

# Restore from Cloud SQL PITR
gcloud sql instances clone my-instance my-restored \
  --point-in-time=2026-02-15T10:00:00Z

# GKE Backup
gcloud beta container backup-restore backup-plans create my-plan \
  --cluster=my-cluster \
  --location=us-central1 \
  --all-namespaces \
  --cron-schedule="0 2 * * *"
```

**Exam tips:**
- Cloud SQL PITR uses the transaction log -- RPO is minutes, not hours
- Spanner multi-region doesn't need explicit DR -- it's built in (RPO ≈ 0)
- BigQuery time travel is 7 days by default (configurable 2-7 days)
- GCS versioning is not the same as backup -- it protects against accidental deletion/overwrite
- "Minimize RPO and RTO with lowest cost" → the exam wants you to find the sweet spot, not always active-active
- Backup for GKE backs up both Kubernetes resources and PV data

**Docs:** [DR Scenarios](https://cloud.google.com/architecture/dr-scenarios-planning-guide), [Cloud SQL Backups](https://cloud.google.com/sql/docs/mysql/backup-recovery/backups), [Backup for GKE](https://cloud.google.com/kubernetes-engine/docs/add-on/backup-for-gke/concepts/backup-for-gke)

---

## 1.3 Designing Network, Storage, and Compute Resources

### Integration with On-premises/Multicloud Environments

**Hybrid connectivity options:**

| Option | Bandwidth | Latency | SLA | Cost | Use Case |
|--------|-----------|---------|-----|------|----------|
| HA VPN | Up to 3 Gbps/tunnel | Variable | 99.99% (with proper config) | $ | Dev/test, small workloads, encrypted |
| Dedicated Interconnect | 10/100 Gbps per link | Low, predictable | 99.9% (1 link) or 99.99% (4 links) | $$$ | Production, high bandwidth |
| Partner Interconnect | 50 Mbps - 50 Gbps | Low | 99.9% or 99.99% | $$ | When Dedicated not available |
| Cross-Cloud Interconnect | 10/100 Gbps | Low | 99.99% | $$$$ | Direct GCP-to-AWS/Azure |

**Decision guide:**
- **< 1 Gbps, encrypted needed**: HA VPN
- **> 10 Gbps, colocation facility**: Dedicated Interconnect
- **No colocation, need low latency**: Partner Interconnect
- **Multi-cloud (AWS/Azure direct)**: Cross-Cloud Interconnect

**Private connectivity to Google APIs:**
- **Private Google Access**: VMs without external IPs access Google APIs via internal routes
- **Private Service Connect (PSC)**: Private endpoint in your VPC for Google APIs or published services
- PSC is preferred for new architectures -- provides a dedicated IP in your VPC

**Anthos / GKE Enterprise for multi-cloud:**
- Run GKE on AWS, Azure, bare metal, VMware
- Fleet management for consistent policy across clusters
- Config Sync for GitOps across environments
- Service Mesh for cross-cluster networking

```bash
# Enable Private Google Access on a subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 --enable-private-ip-google-access

# Create Private Service Connect endpoint
gcloud compute addresses create psc-address --region=us-central1 --subnet=my-subnet
gcloud compute forwarding-rules create psc-endpoint \
  --region=us-central1 --network=my-vpc --subnet=my-subnet \
  --address=psc-address \
  --target-google-apis-bundle=all-apis
```

**Exam tips:**
- "Lowest latency, highest bandwidth to on-prem" → Dedicated Interconnect
- "Encrypted connection to on-prem" → HA VPN (Interconnect traffic is NOT encrypted by default)
- "Private access to Google APIs without external IP" → Private Google Access (simple) or PSC (advanced)
- HA VPN requires 2 tunnels for 99.99% SLA -- single tunnel is NOT HA
- Cross-Cloud Interconnect is for direct cloud-to-cloud, not on-prem

**Docs:** [Hybrid Connectivity](https://cloud.google.com/network-connectivity/docs/how-to/choose-product), [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect), [GKE Enterprise](https://cloud.google.com/kubernetes-engine/enterprise/docs)

---

### Google Cloud AI and ML Solutions

**Vertex AI Platform -- the unified ML platform:**

| Component | Purpose |
|-----------|---------|
| Vertex AI Training | Custom model training (pre-built containers, custom containers) |
| Vertex AI Prediction | Model serving (online, batch, private endpoints) |
| Vertex AI Pipelines | ML workflow orchestration (Kubeflow Pipelines) |
| Vertex AI Feature Store | Centralized feature management (online + offline serving) |
| Vertex AI Experiments | Experiment tracking and comparison |
| Vertex AI Model Registry | Model versioning and lifecycle management |
| AutoML | No-code model training for vision, text, tabular, video |

**Gemini models:**
- Gemini Pro, Gemini Ultra -- Google's frontier LLMs
- Available through Vertex AI API
- Supports text, code, multi-modal (image + text)
- Fine-tuning supported (supervised, RLHF)

**Vertex AI Agent Builder:**
- Build AI agents with grounding (connect to your data)
- Vertex AI Search: enterprise search over your documents
- Vertex AI Conversation: conversational AI agents (replaces Dialogflow CX for new projects)
- RAG (Retrieval-Augmented Generation) pattern support

**Model Garden:**
- Catalog of 100+ models (Google, open-source, partner)
- One-click deployment for supported models
- Fine-tuning capabilities
- Models: Gemini, PaLM, Llama, Mistral, Stable Diffusion, etc.

**AI Hypercomputer:**
- Optimized infrastructure for AI/ML workloads
- Cloud TPUs (v5e for inference, v5p for training)
- GPU clusters (A3 with H100, A2 with A100)
- Multislice training for models across multiple TPU pods

**When to use what:**

| Need | Service |
|------|---------|
| Custom ML model training | Vertex AI Training |
| Pre-trained vision/NLP/speech | Pre-built APIs (Vision AI, NLP, Speech) |
| LLM-based applications | Gemini via Vertex AI |
| Enterprise search | Vertex AI Search (Agent Builder) |
| Conversational AI | Vertex AI Conversation (Agent Builder) |
| Open-source model deployment | Model Garden |
| No-code ML | AutoML |
| Large-scale training | AI Hypercomputer (TPUs/GPUs) |

**Exam tips:**
- "Build a custom ML model" → Vertex AI Training
- "Add search to internal documents" → Vertex AI Search (Agent Builder)
- "Use an LLM for text generation" → Gemini via Vertex AI
- "Deploy an open-source model" → Model Garden
- "Train a massive model" → AI Hypercomputer with TPUs
- Pre-built APIs are the fastest path -- use them when they fit the use case
- AutoML is for teams without ML expertise

**Docs:** [Vertex AI](https://cloud.google.com/vertex-ai/docs), [Model Garden](https://cloud.google.com/vertex-ai/docs/start/explore-models), [Agent Builder](https://cloud.google.com/products/agent-builder)

---

### Cloud-native Networking

**VPC design patterns:**

| Pattern | Architecture | Use Case |
|---------|-------------|----------|
| Shared VPC | Host project owns VPC, service projects share subnets | Centralized network management, enterprise standard |
| VPC Peering | Two VPCs directly connected | Cross-project/cross-org connectivity |
| Hub-and-spoke | Central VPC (hub) peered with spoke VPCs | Multi-team isolation with central egress |
| Private Service Connect | Private endpoints for services | Consume APIs/services privately |

**Shared VPC vs VPC Peering:**

| Feature | Shared VPC | VPC Peering |
|---------|-----------|-------------|
| Transitivity | N/A (single VPC) | Non-transitive (A↔B, B↔C ≠ A↔C) |
| Management | Centralized (network admin) | Decentralized (each project) |
| Firewall rules | Unified | Per-VPC |
| IP address management | Centralized | Per-project |
| Subnet sharing | Yes | No (separate address spaces) |
| Scale | Recommended for enterprise | Simpler, fewer projects |

**Load balancer selection:**

| Need | Load Balancer |
|------|--------------|
| Global HTTP(S) with CDN | Global external Application LB |
| Regional HTTP(S) | Regional external Application LB |
| Internal HTTP(S) | Internal Application LB |
| TCP/UDP external (preserve client IP) | External passthrough Network LB |
| TCP/UDP external (proxy) | External proxy Network LB |
| TCP/UDP internal | Internal passthrough Network LB |
| Cross-region internal | Cross-region internal Application LB |

**Container networking (GKE):**
- VPC-native clusters (alias IPs for pods -- recommended)
- Services: ClusterIP, NodePort, LoadBalancer
- Ingress: L7 load balancing for HTTP(S) traffic
- Gateway API: Next-gen ingress, multi-cluster support
- Network Policies: Pod-level firewall rules (Calico)

```bash
# Create a Shared VPC host project
gcloud compute shared-vpc enable HOST_PROJECT

# Attach a service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT \
  --host-project=HOST_PROJECT

# Create VPC peering
gcloud compute networks peerings create my-peering \
  --network=my-vpc --peer-network=projects/other-project/global/networks/other-vpc

# Create global external Application LB
gcloud compute backend-services create my-backend --global --protocol=HTTP
gcloud compute url-maps create my-urlmap --default-service=my-backend
gcloud compute target-https-proxies create my-proxy --url-map=my-urlmap --ssl-certificates=my-cert
gcloud compute forwarding-rules create my-lb --global --target-https-proxy=my-proxy --ports=443
```

**Exam tips:**
- "Centralized network management for enterprise" → Shared VPC
- "VPC peering is non-transitive" -- this is a classic exam trap
- "Global users, HTTP(S) traffic" → Global external Application LB
- "Internal microservice traffic" → Internal Application LB
- "Preserve client source IP" → Passthrough Network LB (not proxy)
- Shared VPC is preferred over VPC Peering for enterprise architectures

**Docs:** [VPC](https://cloud.google.com/vpc/docs), [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc), [Load Balancing](https://cloud.google.com/load-balancing/docs/choosing-load-balancer)

---

### Choosing Data Processing Solutions

| Service | Type | Use Case | Scaling |
|---------|------|----------|---------|
| Dataflow | Streaming + Batch | Apache Beam pipelines, ETL, real-time analytics | Autoscaling workers |
| Dataproc | Batch (Spark/Hadoop) | Existing Spark/Hadoop workloads, data science | Autoscaling nodes, Spot VMs |
| BigQuery | Analytics | SQL analytics, data warehouse, ML (BQML) | Serverless (slots) |
| Pub/Sub | Messaging | Event streaming, decoupling, at-least-once delivery | Serverless |
| Datastream | CDC | Real-time replication from databases to BigQuery/GCS | Serverless |
| Cloud Composer | Orchestration | Apache Airflow DAGs for pipeline orchestration | Managed Airflow |

**Decision guide:**
- "Real-time stream processing" → Dataflow
- "Existing Spark jobs" → Dataproc
- "SQL analytics on large datasets" → BigQuery
- "Decouple event producers/consumers" → Pub/Sub
- "Continuous database replication" → Datastream
- "Orchestrate multiple pipelines" → Cloud Composer

**Exam tips:**
- Dataflow = Apache Beam (unified batch + stream). Dataproc = Apache Spark/Hadoop.
- BigQuery is serverless -- no cluster management
- Pub/Sub guarantees at-least-once delivery; use Dataflow for exactly-once processing
- Dataproc Serverless eliminates cluster management for Spark

**Docs:** [Dataflow](https://cloud.google.com/dataflow/docs), [Dataproc](https://cloud.google.com/dataproc/docs), [BigQuery](https://cloud.google.com/bigquery/docs), [Pub/Sub](https://cloud.google.com/pubsub/docs)

---

### Choosing Appropriate Storage Types

**Object storage (Cloud Storage):**

| Class | Min Storage | Access | Use Case |
|-------|------------|--------|----------|
| Standard | None | Frequent | Hot data, websites, streaming |
| Nearline | 30 days | Monthly | Backups, infrequently accessed |
| Coldline | 90 days | Quarterly | Disaster recovery, archival |
| Archive | 365 days | Annual | Long-term regulatory archives |

- **Autoclass**: Automatically moves objects between classes based on access patterns
- **Dual-region**: Two specific regions, sub-second failover
- **Turbo Replication**: RPO < 15 minutes for dual-region buckets
- **Archive storage is NOT slow** -- same latency as other classes (exam trap!)

**File storage:**

| Service | Performance | Use Case |
|---------|------------|----------|
| Filestore Basic | Standard NFS | General file shares |
| Filestore Enterprise | High performance, regional | SAP, enterprise apps |
| Parallelstore | Ultra-high throughput | HPC, AI/ML training data |

**Database selection:**

| Database | Type | Scaling | Use Case |
|----------|------|---------|----------|
| Cloud SQL | Relational (MySQL, PostgreSQL, SQL Server) | Vertical + read replicas | Web apps, traditional RDBMS |
| AlloyDB | PostgreSQL-compatible | Vertical + read pool | High-performance PostgreSQL, analytics |
| Spanner | Relational, globally distributed | Horizontal (nodes/PUs) | Global transactions, 99.999% SLA |
| Firestore | Document (NoSQL) | Serverless, automatic | Mobile/web apps, real-time sync |
| Bigtable | Wide-column (NoSQL) | Horizontal (nodes) | IoT, time-series, 10ms latency |
| BigQuery | Analytics (columnar) | Serverless (slots) | Data warehouse, BQML |
| Memorystore | In-memory (Redis/Memcached) | Vertical | Caching, session storage |

**Exam tips:**
- "Global relational database with strong consistency" → Spanner
- "PostgreSQL compatibility with high performance" → AlloyDB
- "Sub-10ms key-value lookups at massive scale" → Bigtable
- "Serverless NoSQL for mobile/web" → Firestore
- "Analytics on petabyte-scale data" → BigQuery
- "Cache for sub-millisecond reads" → Memorystore
- Archive storage has same access latency as Standard -- only cost differs

**Docs:** [Cloud Storage](https://cloud.google.com/storage/docs), [Database Selection](https://cloud.google.com/products/databases), [Spanner](https://cloud.google.com/spanner/docs), [AlloyDB](https://cloud.google.com/alloydb/docs)

---

### Mapping Compute Needs to Platform Products

| Platform | Abstraction | Scaling | Use Case |
|----------|------------|---------|----------|
| Compute Engine | VMs | MIGs, autoscaler | Full OS control, GPU, custom kernel |
| GKE Standard | Containers (K8s) | HPA, Cluster Autoscaler | Complex microservices, full K8s control |
| GKE Autopilot | Containers (managed K8s) | Automatic (pod-level) | K8s without node management |
| Cloud Run | Containers (serverless) | Concurrency-based | Stateless HTTP services, scale to zero |
| Cloud Run functions | Functions (serverless) | Event-driven | Event handlers, lightweight processing |
| App Engine | PaaS | Automatic | Legacy PaaS apps (consider Cloud Run for new) |
| Batch | Batch jobs | Job-based | HPC, rendering, large-scale batch |

**Exam tips:**
- "Minimize operational overhead for containers" → Cloud Run (simplest) or GKE Autopilot (if you need K8s features)
- "Full Kubernetes control with custom node configs" → GKE Standard
- "Scale to zero" → Cloud Run or Cloud Run functions
- "HPC batch processing" → Batch service
- App Engine is legacy for new projects -- Cloud Run is preferred

**Docs:** [Compute Options](https://cloud.google.com/hosting-options)

---

### Choosing Compute Resources

**Machine families:**

| Family | Series | Optimized For | Use Case |
|--------|--------|--------------|----------|
| General-purpose | E2, N2, N2D, T2D, C4 | Balanced | Web, app servers, dev |
| Compute-optimized | C2, C2D, H3 | High single-thread perf | Gaming, HPC, media |
| Memory-optimized | M2, M3 | High memory | SAP HANA, in-memory databases |
| Accelerator-optimized | A2 (A100), A3 (H100), G2 (L4) | GPUs | ML training/inference, rendering |
| Storage-optimized | Z3 | High disk throughput | Databases, analytics |

**Special configurations:**
- **Spot VMs**: 60-91% discount, can be preempted anytime, no SLA
- **Custom machine types**: Specify exact vCPU/memory (N1, N2, E2 families)
- **Sole-tenant nodes**: Dedicated physical server, for BYOL licensing or compliance
- **GPUs**: Attach to VMs for ML, rendering, HPC (A100, H100, L4, T4)
- **TPUs**: Google-designed ML accelerators, optimal for TensorFlow/JAX training

**Exam tips:**
- "Cost-effective for fault-tolerant batch" → Spot VMs
- "SAP HANA" → M2/M3 memory-optimized
- "ML training at scale" → A3 (H100 GPUs) or TPUs
- "BYOL Windows/Oracle licensing" → Sole-tenant nodes
- "Balanced cost/performance for web apps" → E2 (cheapest) or N2 (better performance)
- Custom machine types save money when your workload doesn't fit standard sizes

**Docs:** [Machine Families](https://cloud.google.com/compute/docs/machine-resource), [GPUs](https://cloud.google.com/compute/docs/gpus), [TPUs](https://cloud.google.com/tpu/docs)

---

## 1.4 Creating a Migration Plan

### Integrating Solutions with Existing Systems

**Strangler Fig Pattern:**
Gradually replace components of a legacy system by routing traffic through a facade that delegates to either the old or new system:

```
                    ┌─────────────┐
                    │  API Facade  │ (Apigee / Cloud Load Balancing)
                    │  (Router)    │
                    └──────┬──────┘
                     ┌─────┴─────┐
                     ↓           ↓
              ┌──────────┐ ┌──────────┐
              │  Legacy   │ │  New GCP │
              │  System   │ │  Service │
              └──────────┘ └──────────┘
```

- Start by routing 100% to legacy
- Migrate one service at a time to GCP
- Gradually shift traffic to new services
- Eventually decommission legacy

**Data synchronization during migration:**
- **Dual-write**: Application writes to both old and new (complex, risk of inconsistency)
- **CDC (Change Data Capture)**: Datastream captures changes from source → target
- **Batch sync**: Periodic bulk data transfer (Storage Transfer Service)
- **Event-driven sync**: Pub/Sub for real-time event forwarding

**Exam tips:**
- "Gradually migrate a monolith" → Strangler Fig pattern
- "Keep data in sync during migration" → Datastream (CDC) for databases, Pub/Sub for events
- API facade (Apigee or LB) enables routing decisions without client changes

**Docs:** [Migration Path](https://cloud.google.com/architecture/migration-to-gcp-getting-started)

---

### Assessing and Migrating Systems and Data

**Migration Center (formerly Migrate for Compute Engine):**
- Discovery and assessment of on-premises workloads
- Dependency mapping between applications
- TCO (Total Cost of Ownership) analysis
- Fit assessment (which workloads are ready for cloud)
- Generates migration plans

**Tools by migration type:**

| Source | Target | Tool |
|--------|--------|------|
| VMs (VMware, Hyper-V, AWS) | Compute Engine | Migrate to VMs |
| VMs | GKE containers | Migrate to Containers |
| MySQL, PostgreSQL, SQL Server, Oracle | Cloud SQL, AlloyDB | Database Migration Service (DMS) |
| Any database | BigQuery | Datastream (CDC) or batch export/import |
| File shares | Filestore / GCS | Storage Transfer Service |
| Large data (>100 TB) | GCS | Transfer Appliance |

**Exam tips:**
- "Assess on-prem workloads before migration" → Migration Center
- "Migrate VMs with minimal changes" → Migrate to VMs (rehost)
- "Containerize existing VMs" → Migrate to Containers
- "Continuous database replication during migration" → DMS with CDC
- Migration Center provides the assessment; other tools execute the migration

**Docs:** [Migration Center](https://cloud.google.com/migration-center/docs), [Migrate to VMs](https://cloud.google.com/migrate/virtual-machines/docs), [DMS](https://cloud.google.com/database-migration/docs)

---

### Migration Methodologies

**The 6 R's -- decision criteria:**

| Strategy | Effort | Risk | When to Choose |
|----------|--------|------|---------------|
| **Rehost** | Low | Low | Quick wins, legacy apps, compliance deadlines |
| **Replatform** | Medium | Low-Medium | Database migrations, container adoption |
| **Refactor** | High | Medium-High | Strategic apps, long-term investment |
| **Repurchase** | Medium | Low | SaaS alternatives exist (CRM, email) |
| **Retire** | None | None | Unused or redundant systems |
| **Retain** | None | None | Not ready, compliance constraints, latency |

**Migration phases (Google's recommended approach):**
1. **Assess**: Inventory, dependency mapping, TCO analysis (Migration Center)
2. **Plan**: Choose strategy per workload, prioritize, build migration backlog
3. **Deploy**: Set up foundation (VPC, IAM, Shared VPC), landing zone
4. **Migrate**: Execute migration waves, validate each workload
5. **Optimize**: Right-size, enable autoscaling, adopt managed services

**Workload testing during migration:**
- Functional testing: Does it work the same?
- Performance testing: Does it meet latency/throughput requirements?
- Security testing: Are all controls in place?
- DR testing: Does failover work?
- Rollback plan: Can you revert if migration fails?

**Exam tips:**
- "Migrate with minimum effort and risk" → Rehost
- "Migrate database to managed service" → Replatform (e.g., MySQL on VM → Cloud SQL)
- "Modernize for cloud-native" → Refactor
- Always have a rollback plan -- the exam expects this
- Migration should be done in waves, not all at once

**Docs:** [Migration to Google Cloud](https://cloud.google.com/architecture/migration-to-gcp-getting-started)

---

### Software License Implications and Financial Impact

**Licensing models on GCP:**

| Model | Description | GCP Support |
|-------|------------|-------------|
| BYOL (Bring Your Own License) | Use existing on-prem licenses | Sole-tenant nodes (Windows, Oracle, SQL Server) |
| Pay-as-you-go | Included in VM cost | Windows Server, SQL Server on CE/Cloud SQL |
| License mobility | Move SA licenses to cloud | Microsoft SA licenses via Azure Hybrid Benefit equivalent |

**TCO calculation components:**
- Compute costs (VMs, GKE, serverless)
- Storage costs (disk, object, database)
- Network costs (egress, interconnect)
- Licensing costs (OS, database, middleware)
- Operations costs (staffing reduction from managed services)
- Migration costs (tools, consulting, downtime)

**Exam tips:**
- "Oracle licensing on GCP" → Sole-tenant nodes (required for per-core licensing)
- "Windows Server licensing" → Included in CE pricing OR BYOL on sole-tenant nodes
- "Reduce total cost of ownership" → managed services reduce ops costs
- TCO analysis should include operational savings, not just infrastructure costs

**Docs:** [Sole-Tenant Nodes](https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes), [Pricing Calculator](https://cloud.google.com/products/calculator)

---

## 1.5 Envisioning Future Solution Improvements

### Cloud and Technology Improvements

**Design for evolution:**
- Use managed services that Google continuously improves (Cloud SQL, GKE, Cloud Run)
- Adopt serverless patterns that auto-scale with new hardware generations
- Use Terraform modules that can be updated independently
- Monitor Google Cloud release notes for new features

**Key areas of rapid evolution:**
- **AI/ML**: Gemini models, Agent Builder, Model Garden -- new capabilities monthly
- **Serverless**: Cloud Run GPU support, Direct VPC egress, multi-region
- **Kubernetes**: GKE Autopilot improvements, Gateway API, fleet management
- **Data**: BigQuery continuous queries, Spanner PostgreSQL interface, AlloyDB AI

**Exam tips:**
- "Design for future scalability" → serverless + autoscaling + managed services
- "Stay current with GCP capabilities" → adopt managed services that evolve automatically
- The exam expects cloud-first design, not just lift-and-shift

---

### Evolution of Business Needs

**Design for extensibility:**
- API-first architecture: Expose services through well-defined APIs (Apigee)
- Event-driven patterns: Use Pub/Sub/Eventarc for loose coupling
- Microservices: Enable independent scaling and deployment of components
- Data platform: BigQuery as central analytics hub that grows with business needs

**Exam tips:**
- "Support future integrations" → API-first with Apigee
- "Handle unpredictable growth" → serverless (Cloud Run, BigQuery, Firestore)
- Loose coupling enables teams to evolve services independently

---

### Cloud-first Design Approach

**Cloud-native principles:**
- Design for horizontal scaling, not vertical
- Assume failures will happen -- design for resilience
- Use managed services by default -- self-manage only when necessary
- Automate everything (IaC, CI/CD, monitoring)
- Treat infrastructure as cattle, not pets

**Anti-patterns to avoid:**
- Lift-and-shift without optimization (leads to higher costs)
- Using VMs when serverless would work
- Manual deployments instead of CI/CD
- Monolithic databases when purpose-built databases fit better
- Over-engineering for hypothetical future requirements

**Exam tips:**
- The PCA exam strongly favors cloud-native approaches over lift-and-shift
- "Minimize operational overhead" → managed services, serverless
- "Maximize reliability" → multi-region, auto-healing, auto-scaling
- Cloud-first doesn't mean cloud-only -- hybrid/multi-cloud is valid when justified

**Docs:** [Cloud Architecture Center](https://cloud.google.com/architecture), [Application Modernization](https://cloud.google.com/solutions/application-modernization)
