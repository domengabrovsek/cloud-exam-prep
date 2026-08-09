# Section 1: Designing and Planning a Cloud Solution Architecture

> **Exam weight:** ~25% of the PCA exam.
>
> This section covers designing solution architecture for business and technical requirements, network/storage/compute design, migration planning, and future improvements. Questions are scenario-based and architect-level.

**Total questions: 65**

---

## 1.1 Business Requirements (Q1-Q12)

---

### Q1. A global e-commerce company, ShopNova, expects 10x traffic spikes during annual sales events lasting 3-5 days. They currently run on-premises and want to move to GCP. Their CFO requires predictable monthly cloud costs while keeping the flexibility to handle traffic bursts. Which pricing strategy best meets their needs?

A) Purchase 3-year Committed Use Discounts (CUDs) sized for peak traffic
B) Purchase 1-year CUDs for baseline capacity and use on-demand instances for burst traffic
C) Use only Spot VMs to minimize cost across all workloads
D) Use Sustained Use Discounts (SUDs) exclusively by running on-demand instances year-round

<details>
<summary>Answer</summary>

**Correct: B)**

Committed use discounts on the steady-state baseline give the CFO the predictable monthly figure, and on-demand capacity absorbs the 3-5 day spike without extending the commitment. Compute Engine CUDs reach up to 55% for most machine series and up to 70% for memory-optimized, with a 3-year term discounting deeper than a 1-year term.

- **A) is wrong** -- sizing a commitment for peak means paying for peak capacity for the roughly 360 days a year the peak is not happening. A commitment is a floor you must pay for, not a ceiling you may use.
- **C) is wrong** -- Spot VMs can be reclaimed at any time. Customer-facing checkout traffic during a revenue event is the worst possible place to accept preemption.
- **D) is wrong** -- SUDs cap at 30%, apply automatically to resources used for more than 25% of a billing month, and cannot be combined with CUDs. They are a passive rebate, not a budgeting instrument.

**Exam tip:** when a stem pairs "predictable cost" with "bursty traffic", the answer is almost always commit to the baseline and burst on-demand. Never commit to peak. A CUD is a spend obligation, so anything described as short-lived, seasonal or uncertain stays on-demand.

Docs: https://cloud.google.com/docs/cuds and https://cloud.google.com/compute/docs/sustained-use-discounts
</details>

---

### Q2. DataMesh Corp is building a real-time analytics platform. The business team requires that insights be available within 5 seconds of an event occurring. The engineering team wants a managed solution with minimal operational overhead. The data arrives as a continuous stream from IoT devices at ~500,000 events per second. Which architecture best meets both business and technical requirements?

A) IoT devices -> Pub/Sub -> Dataflow (streaming) -> BigQuery -> Looker
B) IoT devices -> Pub/Sub -> Cloud Functions -> Cloud SQL -> Looker
C) IoT devices -> Kafka on GKE -> Dataproc (Spark Streaming) -> BigQuery -> Looker
D) IoT devices -> Cloud Storage -> Dataflow (batch every 5 seconds) -> BigQuery -> Looker

<details>
<summary>Answer</summary>

**Correct: A)**

Pub/Sub absorbs the 500K events/sec ingest durably, Dataflow in streaming mode does the per-event processing with exactly-once semantics, BigQuery holds the results, and Looker visualises. Every component is fully managed, which is the stated engineering constraint.

- **B) is wrong** -- Cloud SQL is an OLTP relational database. It cannot sustain 500K writes/sec and is not an analytics engine.
- **C) is wrong** -- self-managed Kafka on GKE plus Dataproc means owning cluster sizing, upgrades and failure handling, which directly contradicts "minimal operational overhead".
- **D) is wrong** -- landing events in Cloud Storage and micro-batching every 5 seconds adds latency at exactly the point the requirement is tightest, and Cloud Storage is not a streaming ingestion layer.

**Exam tip:** the streaming reference pipeline on Google Cloud is Pub/Sub then Dataflow then BigQuery or Bigtable. Learn it as one unit. Any option that swaps Pub/Sub for Cloud Storage, or Dataflow for Cloud Run functions, is trading a managed streaming primitive for something that only looks like one.

Docs: https://cloud.google.com/pubsub/docs/overview and https://cloud.google.com/dataflow/docs
</details>

---

### Q3. FinServe Bank is migrating its core banking application to GCP. Regulatory requirements mandate that all customer data must remain within the EU, audit logs must be retained for 7 years, and the system must be available 99.99% of the time. The application uses a relational database with complex transactions. Which combination of services best meets these requirements? (Choose TWO)

A) Cloud Spanner with a multi-region configuration across EU regions
B) Cloud SQL with regional HA in europe-west1
C) BigQuery with dataset-level location set to EU multi-region
D) Cloud Audit Logs exported to Cloud Storage with a 7-year retention policy and bucket lock
E) Cloud Audit Logs exported to BigQuery with a 7-year table expiration

<details>
<summary>Answer</summary>

**Correct: A) and D)**

A Spanner multi-region configuration inside the EU keeps data resident, gives strong consistency for complex transactions, and carries a 99.999% availability SLA, which clears the 99.99% requirement. Audit logs exported to Cloud Storage under a locked retention policy give immutable 7-year retention at archival cost.

- **B) is wrong** -- Cloud SQL regional HA is a 99.95% SLA, below the stated 99.99%.
- **C) is wrong** -- BigQuery is an analytics warehouse, not a transactional store for core banking.
- **E) is wrong** -- table expiration deletes data at the end of the window. The requirement is retention, which is the opposite, and BigQuery storage for 7 years of audit logs costs far more than Cloud Storage.

**Exam tip:** read availability numbers as elimination criteria. Cloud SQL HA is 99.95%, Spanner regional is 99.99%, Spanner multi-region is 99.999%. Also learn the retention-versus-expiration trap: expiration deletes, a locked retention policy prevents deletion. Compliance stems always want the second one.

Docs: https://cloud.google.com/spanner/docs/instance-configurations and https://cloud.google.com/storage/docs/bucket-lock
</details>

---

### Q4. A media streaming company, StreamVault, wants to expand from North America to serve users in Asia-Pacific and Europe. They need to minimize latency for video content delivery while controlling costs. Their content catalog is 500 TB of video files, with 20% of content accounting for 80% of views. What should you recommend?

A) Deploy full application stacks in every target region with complete content replicas
B) Use Cloud CDN with Cloud Storage multi-regional buckets, keeping origin servers in North America
C) Use Premium Tier networking with Cloud CDN, origin servers in North America, and Cloud Storage regional buckets with lifecycle rules
D) Deploy Compute Engine instances in each region running Nginx as a caching reverse proxy

<details>
<summary>Answer</summary>

**Correct: C)**

Premium Tier routes over Google's backbone for the lowest latency, and Cloud CDN caches the 20% of the catalogue that drives 80% of views at the edge, so the origin only serves misses. Regional buckets are cheaper than multi-region and the redundancy multi-region buys is not needed once CDN is doing the fan-out.

- **A) is wrong** -- full stacks plus a 500 TB content replica in every region is the most expensive way to solve a caching problem.
- **B) is wrong** -- multi-region storage adds cost that CDN already makes unnecessary, and the option says nothing about the network tier, which is the other half of the latency answer.
- **D) is wrong** -- self-managed Nginx caches in each region mean running and patching cache fleets, with far less edge coverage than Google's CDN footprint.

**Exam tip:** a stated 80/20 access skew is the exam signalling "cache it, do not replicate it". Pair that with the network tier question: Premium Tier is Google's backbone and is required for global load balancing, Standard Tier hands traffic to the public internet near the source region and is regional.

Docs: https://cloud.google.com/cdn/docs/overview and https://cloud.google.com/network-tiers/docs/overview
</details>

---

### Q5. GreenEnergy Inc. has a legacy on-premises ERP system that must remain on-premises for 2 more years due to a vendor contract. They want to build new cloud-native microservices on GCP that need to access ERP data in real time. The connection must be secure, low-latency, and support up to 10 Gbps throughput. What integration pattern should you recommend?

A) Site-to-site VPN with Cloud Router for BGP routing
B) Dedicated Interconnect with Private Google Access
C) Partner Interconnect with a Cloud VPN backup
D) Transfer Appliance to replicate ERP data to Cloud Storage nightly

<details>
<summary>Answer</summary>

**Correct: B)**

Dedicated Interconnect is a physical connection into Google's network at 10, 100 or 400 Gbps link speeds, with private, low-latency, predictable performance. Private Google Access keeps the API traffic off the public internet.

- **A) is wrong** -- HA VPN runs over the public internet with variable latency, and its documented throughput limit is expressed in packets per second, so reaching a reliable 10 Gbps means aggregating tunnels and accepting jitter.
- **C) is wrong** -- Partner Interconnect is the answer when you cannot meet Google's colocation requirement. Here the requirement is a clean 10 Gbps, so the direct product with no third party in the path is simpler.
- **D) is wrong** -- Transfer Appliance is offline bulk migration. Nightly replication cannot serve a real-time integration.

**Exam tip:** the connectivity ladder is Cloud VPN, then Partner Interconnect, then Dedicated Interconnect. Pick by two facts in the stem: required bandwidth, and whether the customer has colocation presence. "No colocation" forces Partner. "Real-time" or "consistent multi-gigabit" rules out VPN.

Docs: https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview and https://cloud.google.com/vpc/docs/private-google-access
</details>

---

### Q6. A healthcare startup, MedTrack, is designing a patient monitoring system. They must comply with HIPAA, ensure data is encrypted at rest and in transit, and maintain a complete audit trail. The system processes sensitive patient vitals in real time. Which design consideration is LEAST relevant to meeting their business requirements?

A) Enabling Customer-Managed Encryption Keys (CMEK) for all data stores
B) Configuring VPC Service Controls to create a security perimeter around sensitive services
C) Choosing a multi-region Cloud Spanner instance for the lowest possible read latency globally
D) Enabling Data Access audit logs for all services handling patient data

<details>
<summary>Answer</summary>

**Correct: C)**

Choosing multi-region Spanner for the lowest global read latency is a performance optimisation. Nothing in the stem says the startup has global users, so this is the option least connected to the stated compliance requirements.

- **A) is relevant** -- CMEK puts key control in the customer's hands, which is routinely part of a healthcare control set.
- **B) is relevant** -- VPC Service Controls draw a perimeter around the managed services holding patient data and block exfiltration paths.
- **D) is relevant** -- Data Access audit logs are the record of who read which record, which is the audit trail the stem asks for.

**Exam tip:** "LEAST relevant" questions are read backwards. Find the three options that map to a stated requirement and the leftover one is the answer. Watch for a technically excellent choice that answers a requirement the stem never made, which is usually global latency or global scale.

Docs: https://cloud.google.com/vpc-service-controls/docs/overview and https://cloud.google.com/logging/docs/audit
</details>

---

### Q7. TechRetail is evaluating moving their data warehouse from an on-premises Teradata system to GCP. Their annual Teradata license costs $2M, and they process 50 TB of data. The CFO wants to understand the total cost of ownership (TCO) on GCP over 3 years. Which factors should the architect include in the TCO analysis? (Choose THREE)

A) BigQuery on-demand query pricing based on estimated data scanned
B) Cost of Dedicated Interconnect for the migration period
C) Savings from eliminating on-premises hardware refresh cycles and data center costs
D) Cost of retraining all developers on a new programming language
E) BigQuery flat-rate (editions) pricing for predictable workloads

<details>
<summary>Answer</summary>

**Correct: A), C), and E)**

A 3-year TCO comparison needs the recurring platform cost under both BigQuery pricing models, on-demand and editions, plus the avoided on-premises costs (hardware refresh, floor space, power, cooling, operations staff) that make up most of the saving against a $2M annual Teradata licence.

- **B) is wrong** -- Interconnect for the migration window is a one-off project cost. It belongs in the migration business case, not in the 3-year run-rate comparison.
- **D) is wrong** -- BigQuery uses GoogleSQL. Moving Teradata SQL means dialect work, not retraining developers in a new programming language.

**Exam tip:** TCO questions separate recurring run cost and avoided cost from one-time migration cost. If an option describes something you pay once, it is usually the distractor. Watch also for invented retraining or rewrite costs attached to a service that speaks the same language the team already uses.

Docs: https://cloud.google.com/bigquery/pricing and https://cloud.google.com/bigquery/docs/editions-intro
</details>

---

### Q8. A logistics company, FastFreight, needs to track 50,000 delivery vehicles in real time. Each vehicle sends GPS coordinates every 2 seconds. The business requires a live dashboard showing vehicle locations and a historical analytics system for route optimization. What architecture balances real-time requirements with cost efficiency?

A) All data to BigQuery via streaming inserts; use BigQuery for both real-time dashboard and analytics
B) Pub/Sub -> Dataflow with two outputs: one to Bigtable for real-time serving, one to BigQuery for historical analytics
C) Pub/Sub -> Cloud Functions writing to Firestore for real-time views and Cloud Storage for analytics
D) Pub/Sub -> Dataflow -> Cloud SQL for both real-time and analytics workloads

<details>
<summary>Answer</summary>

**Correct: B)**

One Dataflow pipeline fanning out to two sinks uses each store for what it is good at. Bigtable serves the live dashboard with single-digit millisecond lookups by row key (vehicle ID plus timestamp); BigQuery holds history for route-optimisation analytics. At 50,000 vehicles every 2 seconds this is 25,000 events/sec, comfortably inside both services.

- **A) is wrong** -- BigQuery can ingest the stream, but it is an analytical engine. Point lookups for a live map are the wrong access pattern for it.
- **C) is wrong** -- Firestore is a document database tuned for application data, not for sustained high-rate time-series writes at this volume.
- **D) is wrong** -- Cloud SQL cannot take 25,000 writes/sec and is not an analytics warehouse.

**Exam tip:** when a stem asks for both a live view and historical analysis, expect a fan-out answer rather than one database. The pairing to memorise is Bigtable for key-and-time-range serving, BigQuery for ad-hoc SQL over history. A single store claiming to do both is usually the trap.

Docs: https://cloud.google.com/bigtable/docs/overview and https://cloud.google.com/bigquery/docs/introduction
</details>

---

### Q9. A SaaS company, CloudHR, is planning their GCP project structure. They have 3 product lines, each with dev, staging, and production environments. The security team requires that production environments have stricter policies than non-production. The finance team needs to track costs per product line. What organizational structure should you recommend?

A) One project per environment with labels for product lines
B) Folders per product line, sub-folders per environment, separate projects per service within each environment
C) One shared project for all dev environments, one for staging, one for production per product line
D) Folders per environment (dev, staging, prod), with org policies on the prod folder, and projects per product line within each environment folder, using labels for cost tracking

<details>
<summary>Answer</summary>

**Correct: D)**

Folders by environment let one org policy on the prod folder inherit to every production project at once. Projects per product line inside each environment folder give resource isolation, and project labels feed per-product-line cost attribution through the billing export.

- **A) is wrong** -- one project per environment means all three product lines share a blast radius and an IAM surface.
- **B) is wrong** -- putting environment folders under product-line folders inverts the inheritance. You would have to apply the production policy separately under each product line and keep the copies in sync.
- **C) is wrong** -- shared projects across product lines break least privilege and blur cost attribution.

**Exam tip:** decide the folder axis by asking what you need to apply policy to as a single unit. Org policy inherits down the hierarchy, so the thing you govern (usually environment) goes high and the thing you only need to report on (usually team or product) goes low, because labels handle reporting.

Docs: https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy and https://cloud.google.com/resource-manager/docs/organization-policy/overview
</details>

---

### Q10. An autonomous vehicle company, DriveAI, needs to process 10 TB of sensor data daily from test vehicles. The data must be processed within 4 hours of collection to meet their development cycle requirements. Results are used by ML engineers for model training. Processing is compute-intensive but highly parallelizable. What is the most cost-effective approach?

A) A persistent GKE cluster with GPU node pools running 24/7
B) Dataflow batch pipeline triggered when data arrives, using autoscaling workers
C) Dataproc cluster with Spot VM workers, triggered by Cloud Composer, writing results to Cloud Storage
D) Compute Engine with custom machine types running permanently

<details>
<summary>Answer</summary>

**Correct: C)**

Dataproc secondary workers can be Spot VMs at up to 91% off on-demand, and Spark redistributes tasks from reclaimed workers automatically, so a highly parallel, restartable job absorbs preemption cheaply. Cloud Composer schedules the daily run, and results land in Cloud Storage where the ML training jobs can read them directly.

- **A) is wrong** -- a GPU node pool running around the clock is billed around the clock for a job that occupies a few hours a day.
- **B) is wrong**, but not for the reason usually given. Dataflow does have a discounted batch mode: FlexRS runs the batch pool at roughly 90% preemptible VMs. The catch is what you pay for it -- FlexRS queues the job and submits it for execution within six hours of creation, which cannot honour a 4-hour deadline. Regular Dataflow batch starts immediately but pays full worker rates.
- **D) is wrong** -- permanently running custom VMs waste the roughly 20 hours a day with no work to do.

**Exam tip:** "highly parallelizable" plus "tolerates retries" is the exam telling you preemptible capacity is allowed. But check the deadline before reaching for the cheapest option: FlexRS trades a start-time guarantee for its discount, so any stem with a hard processing window rules it out.

Docs: https://cloud.google.com/dataproc/docs/concepts/compute/secondary-vms and https://cloud.google.com/dataflow/docs/guides/flexrs
</details>

---

### Q11. RetailMax is building a recommendation engine that must serve personalized product suggestions to 10 million daily active users with sub-100ms latency. The ML team has trained a TensorFlow model. The business team requires A/B testing capability between model versions. Which serving architecture should you recommend?

A) Deploy the model on Vertex AI Endpoints with traffic splitting for A/B testing
B) Deploy the model in Cloud Functions triggered by API calls
C) Host the model on a single large Compute Engine instance behind a load balancer
D) Serve predictions from BigQuery ML using SQL queries at request time

<details>
<summary>Answer</summary>

**Correct: A)**

Vertex AI Endpoints (the platform is now marketed as Gemini Enterprise Agent Platform, but the exam guide still says Vertex AI) provide managed online serving with autoscaling and native traffic splitting across deployed model versions, which is exactly the A/B testing mechanism the business team asked for.

- **B) is wrong** -- function cold starts alone can breach a 100ms budget, and reloading a model per cold start makes it worse.
- **C) is wrong** -- a single instance is a single point of failure and cannot scale to 10 million daily users.
- **D) is wrong** -- BigQuery ML is built for batch scoring and analytics. Its per-query overhead sits above the 100ms target before the model even runs.

**Exam tip:** "A/B test between model versions" maps to traffic splitting on a Vertex AI endpoint, not to a load balancer or a custom router. And treat "sub-100ms" as an elimination rule: anything that scales from zero or reloads a model per request is out.

Docs: https://cloud.google.com/vertex-ai/docs/general/deployment and https://cloud.google.com/vertex-ai/docs/predictions/configure-compute
</details>

---

### Q12. A government agency is evaluating GCP for their citizen services portal. They require FedRAMP High compliance, data sovereignty within the country, and the ability to measure success of the migration against defined KPIs. Which approach should the architect take to define and measure success?

A) Define KPIs focused solely on infrastructure cost reduction
B) Define KPIs across four dimensions: cost optimization, operational excellence, security/compliance posture, and citizen experience (latency, availability)
C) Use only Google's default SLA metrics as success criteria
D) Measure success based on the number of services migrated per quarter

<details>
<summary>Answer</summary>

**Correct: B)**

Success criteria should span the pillars the Architecture Framework names rather than a single dimension: cost, operational excellence, security and compliance posture, and the citizen-facing experience. For a public agency, compliance and user experience carry as much weight as spend.

- **A) is wrong** -- cost-only KPIs leave the compliance and citizen-experience objectives unmeasured, which for a government portal are the ones that decide whether the migration succeeded.
- **C) is wrong** -- a Google SLA is a floor Google commits to, not a target the agency chose. Success metrics have to come from the agency's own objectives.
- **D) is wrong** -- services migrated per quarter is an output measure. It says nothing about whether the migrated services are reliable, secure or usable.

**Exam tip:** distinguish output metrics (things done) from outcome metrics (effects achieved). The exam consistently rewards outcome measures tied to business objectives, and consistently punishes velocity counters and vendor SLAs presented as success criteria.

Docs: https://cloud.google.com/architecture/framework and https://cloud.google.com/architecture/framework/reliability
</details>

---

## 1.2 Technical Requirements (Q13-Q24)

---

### Q13. A fintech company requires their payment processing service to maintain 99.99% availability across two regions. The service uses Cloud Spanner as the database. If the primary region experiences a complete outage, the system must fail over with zero data loss (RPO = 0) and less than 30 seconds downtime (RTO < 30s). Which architecture achieves this?

A) Cloud Spanner multi-region instance with application deployed to two regions behind a global external Application Load Balancer with health checks
B) Cloud Spanner single-region instance with cross-region read replicas and manual failover
C) Two independent Cloud SQL instances with application-level replication and DNS failover
D) Cloud Spanner multi-region instance with application in a single region and Cloud DNS failover

<details>
<summary>Answer</summary>

**Correct: A)**

A Spanner multi-region configuration replicates synchronously across regions, so RPO is 0 by design. Running the application in both regions behind a global external Application Load Balancer with health checks means a regional failure is absorbed by the load balancer redirecting to healthy backends, well inside 30 seconds.

- **B) is wrong** -- Spanner does not have traditional promotable read replicas, and a single-region instance with manual failover meets neither RPO 0 nor RTO under 30 seconds.
- **C) is wrong** -- application-level replication between two Cloud SQL instances is asynchronous, so RPO is above 0, and DNS failover takes minutes because of TTL propagation.
- **D) is wrong** -- the data survives the region loss but the application does not, and DNS failover is again too slow.

**Exam tip:** an RTO measured in seconds rules out anything that fails over through DNS, because clients cache records for the TTL. Seconds-level failover comes from an anycast global load balancer with health checks. Read RPO and RTO as two separate filters: the database answers RPO, the traffic layer answers RTO.

Docs: https://cloud.google.com/spanner/docs/instance-configurations and https://cloud.google.com/load-balancing/docs/application-load-balancer
</details>

---

### Q14. An online gaming company experiences highly variable traffic: 5,000 concurrent users during off-peak and 500,000 during peak events that can start with little warning. Their backend runs on GKE. Which scaling configuration ensures they can handle sudden traffic spikes without over-provisioning during off-peak?

A) Horizontal Pod Autoscaler (HPA) based on CPU utilization with Cluster Autoscaler enabled
B) Vertical Pod Autoscaler (VPA) to right-size pods dynamically
C) HPA based on custom Pub/Sub queue metrics, Cluster Autoscaler with provisioning profiles, and node auto-provisioning (NAP)
D) Fixed node pool sized for peak traffic with HPA for pod scaling

<details>
<summary>Answer</summary>

**Correct: C)**

A 100x spike arriving with little warning needs a leading signal, not a lagging one. HPA on a queue-depth custom metric reacts as demand appears rather than after CPU saturates, cluster autoscaler adds nodes, and node auto-provisioning creates suitably shaped node pools without someone predefining them.

- **A) is wrong** -- CPU-based HPA is inherently reactive. CPU must climb before the signal exists, by which time a 100x spike has already degraded the service.
- **B) is wrong** -- VPA right-sizes the resource requests of individual pods. It does not add pods or nodes, so it does not answer a traffic-spike question at all.
- **D) is wrong** -- a fixed node pool sized for 500,000 users is paid for at 5,000 users too, which is the over-provisioning the stem explicitly forbids.

**Exam tip:** learn the two autoscaler axes. HPA and VPA scale pods (count versus size); cluster autoscaler and node auto-provisioning scale nodes (count versus shape). A traffic-spike stem needs HPA plus node capacity. If a stem stresses sudden onset, prefer a demand-side custom metric such as queue depth over CPU.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler and https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-provisioning
</details>

---

### Q15. A healthcare platform must achieve RPO of 1 hour and RTO of 4 hours for its patient records system. The system uses Cloud SQL for PostgreSQL and serves a single country. Which DR strategy is most cost-effective while meeting these requirements?

A) Cloud SQL with HA configuration (regional) and automated backups with point-in-time recovery
B) Cloud SQL cross-region read replica with automatic promotion
C) Multi-region Cloud Spanner instance
D) Cloud SQL with daily export to Cloud Storage in another region

<details>
<summary>Answer</summary>

**Correct: A)**

Automated backups plus point-in-time recovery let you restore to any moment inside the log retention window, which satisfies a 1-hour RPO, and a restore of a single-country database completes inside a 4-hour RTO. It is the cheapest configuration that clears both numbers.

- **B) is wrong** -- a cross-region read replica runs, and is billed, continuously. It buys a faster failover than a 4-hour RTO requires.
- **C) is wrong** -- Spanner multi-region is engineered for RPO 0 and near-zero RTO. Against RPO 1 hour and RTO 4 hours it is large over-engineering at a much higher price.
- **D) is wrong** -- daily exports leave up to 24 hours of data at risk, which breaches the 1-hour RPO outright.

**Exam tip:** the phrase "most cost-effective while meeting the requirements" means find the cheapest option that clears the bar, not the most robust one. Work it as arithmetic: daily backup means RPO 24h, PITR means RPO minutes, synchronous replication means RPO 0. Pick the first one that fits and stop.

Docs: https://cloud.google.com/sql/docs/postgres/backup-recovery/pitr and https://cloud.google.com/architecture/dr-scenarios-planning-guide
</details>

---

### Q16. A company is deploying a microservices architecture on GKE with 30 services. They need to implement mutual TLS between services, circuit breaking, traffic management (canary deployments), and distributed tracing -- all without modifying application code. What should they implement?

A) Cloud Service Mesh (managed Istio, formerly Anthos Service Mesh) with sidecar proxies
B) Custom Envoy proxy configurations deployed as DaemonSets
C) Network Policies with GKE Dataplane V2 and manual mTLS certificate management
D) Cloud Load Balancing with NEGs for each service

<details>
<summary>Answer</summary>

**Correct: A)**

Cloud Service Mesh (this is the current name for what shipped as Anthos Service Mesh, Google's managed Istio) delivers mTLS between services, circuit breaking, canary traffic splitting and distributed tracing through injected sidecar proxies, with no application code changes.

- **B) is wrong** -- hand-configuring Envoy as DaemonSets means building and maintaining each of those features yourself; the managed mesh exists precisely to avoid that.
- **C) is wrong** -- network policy is L3/L4 segmentation. It cannot express circuit breaking, canary routing or tracing, and hand-rolled mTLS certificate management is both error-prone and outside the "no code changes" constraint.
- **D) is wrong** -- Cloud Load Balancing handles north-south traffic into the cluster. The requirements are all east-west, service to service.

**Exam tip:** the combination "mTLS plus traffic management plus tracing, without touching application code" is the service mesh fingerprint. Note the naming: Anthos Service Mesh and Traffic Director are both now Cloud Service Mesh, and Anthos itself is GKE Enterprise, so older material and the current console disagree.

Docs: https://cloud.google.com/service-mesh/docs/overview
</details>

---

### Q17. An architect is designing a system that aligns with the Google Cloud Architecture Framework (WAF). The application handles financial transactions and must prioritize reliability and security. During the design review, the team debates whether to implement a multi-region active-active setup. According to WAF principles, what should guide this decision?

A) Always implement multi-region active-active for maximum reliability regardless of cost
B) Define reliability targets (SLOs) based on business requirements, then choose an architecture that meets those targets cost-effectively
C) Use the simplest single-region architecture and add regions only after experiencing outages
D) Follow the WAF performance pillar to determine region placement based on user latency

<details>
<summary>Answer</summary>

**Correct: B)**

The Architecture Framework's reliability pillar asks you to derive reliability targets from business need first and then choose the cheapest architecture that meets them. Multi-region active-active is a means, not a goal, and may be unnecessary if regional HA satisfies the SLO.

- **A) is wrong** -- the framework explicitly warns against building past the requirement. Unjustified multi-region adds cost and operational complexity.
- **C) is wrong** -- waiting for outages to reveal your reliability needs is the opposite of designing to a target.
- **D) is wrong** -- latency belongs to the performance pillar. It can inform region choice, but it is not what decides an active-active reliability question.

**Exam tip:** any Architecture Framework question that offers "always do the most redundant thing" is offering the wrong answer. The framework's stance is target first, architecture second. Watch for options that quietly answer a different pillar than the one the stem is about.

Docs: https://cloud.google.com/architecture/framework/reliability and https://cloud.google.com/architecture/framework
</details>

---

### Q18. A video conferencing platform must handle 10,000 concurrent WebSocket connections per instance with low latency. The connections are long-lived (average 45 minutes). The system must scale horizontally and maintain session affinity. Which compute and load balancing architecture should you use?

A) GKE pods with a global external Application Load Balancer using session affinity (cookie-based)
B) Compute Engine instances with a regional external TCP/UDP Network Load Balancer with connection tracking
C) Cloud Run services with a global external Application Load Balancer
D) GKE pods with a global external proxy Network Load Balancer with session affinity

<details>
<summary>Answer</summary>

**Correct: A)**

The external Application Load Balancer has native WebSocket support: the protocol upgrade is handled automatically and no special configuration is required. Because a WebSocket connection begins as an HTTP request, cookie-based session affinity is set during that handshake and then holds for the life of the connection. GKE supplies the horizontal pod scaling.

- **B) is wrong** -- the regional external TCP/UDP Network Load Balancer is a passthrough (non-proxy) load balancer and is regional only, so it cannot distribute globally.
- **C) is wrong** -- Cloud Run targets request-response workloads and does not guarantee affinity across instances for 45-minute connections.
- **D) is wrong** -- WebSocket support is documented as an Application Load Balancer capability. Choosing a TCP proxy here gives up the HTTP-layer affinity the scenario asks for.

**Exam tip:** WebSocket plus session affinity means Application Load Balancer, not a Network Load Balancer. The trap is reasoning "WebSocket is TCP, so pick the TCP load balancer" -- Google documents WebSocket as an Application Load Balancer capability that needs zero configuration.

Docs: https://cloud.google.com/load-balancing/docs/https and https://cloud.google.com/load-balancing/docs/application-load-balancer
</details>

---

### Q19. A retail analytics system must process overnight batch jobs that aggregate sales data from 200 stores. The jobs are CPU-intensive, take approximately 3 hours, and can tolerate individual task failures if retried. The results must be available by 6 AM daily. Reliability target is 99.5% for the batch pipeline. Which architecture best meets these requirements?

A) Cloud Composer orchestrating a Dataproc cluster with standard (on-demand) workers and preemptible secondary workers
B) A single Compute Engine VM with the largest available machine type
C) Cloud Composer orchestrating Dataflow batch jobs on n1-standard workers
D) Cloud Run jobs triggered by Cloud Scheduler

<details>
<summary>Answer</summary>

**Correct: A)**

Cloud Composer gives managed Airflow orchestration with retries, scheduling and alerting. Dataproc with on-demand primary workers keeps the cluster stable while preemptible secondary workers carry the parallel, retry-tolerant work cheaply, and Spark redistributes tasks off reclaimed workers automatically.

- **B) is wrong** -- one large VM is a single point of failure against a 99.5% target, cannot use parallelism, and runs into a hard vertical ceiling.
- **C) is wrong** -- Dataflow batch runs its workers at standard rates unless you accept the FlexRS scheduling delay, so for a known nightly Spark-shaped job Dataproc with preemptible secondaries is cheaper.
- **D) is wrong** -- Cloud Run jobs run containerised tasks; a multi-stage distributed aggregation is what Spark on Dataproc is for.

**Exam tip:** on Dataproc, primary workers must be standard and only secondary workers can be preemptible. That split is itself an exam answer: keep enough on-demand capacity to hold HDFS and the driver, and buy the elastic tail cheap.

Docs: https://cloud.google.com/dataproc/docs/concepts/compute/secondary-vms and https://cloud.google.com/composer/docs/concepts/overview
</details>

---

### Q20. An architect needs to design a system that can failover between GCP regions automatically. The application is stateless, but uses Cloud SQL for persistent storage. The target is RPO < 5 minutes and RTO < 10 minutes. What is the minimum architecture required?

A) Cloud SQL cross-region read replica with manual promotion, application on MIGs in two regions behind global Application Load Balancer
B) Cloud SQL HA configuration in a single region, application on a MIG with autoscaling
C) Cloud SQL cross-region read replica with automated promotion using Cloud Functions, application on MIGs in two regions behind global Application Load Balancer
D) Cloud Spanner multi-region with application on MIGs in two regions

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud SQL cross-region replication is asynchronous but typically lags by seconds, which fits RPO under 5 minutes. Automating the promotion (triggered from a health check or monitoring alert) is what brings RTO under 10 minutes, and the global Application Load Balancer moves traffic to the surviving region's MIG.

- **A) is wrong** -- the only difference from C is that a human has to notice, decide and act. Pager plus assessment plus execution rarely fits inside 10 minutes.
- **B) is wrong** -- HA is a zonal protection. A single-region deployment does not survive a regional outage at all.
- **D) is wrong** -- Spanner multi-region gives RPO 0, which is better than asked, at a much higher price. The stem says minimum architecture.

**Exam tip:** when two options differ only by "manual" versus "automated" promotion, the RTO number decides. Anything under about 15 minutes needs automation. And "minimum architecture required" is an instruction to reject the stronger, pricier option even though it also works.

Docs: https://cloud.google.com/sql/docs/postgres/replication/cross-region-replicas and https://cloud.google.com/architecture/dr-scenarios-planning-guide
</details>

---

### Q21. A machine learning team has trained a large language model with custom weights (15 GB model size). They need to serve it for internal applications with consistent sub-200ms inference latency. Traffic is predictable at ~1,000 requests per minute during business hours and near-zero outside business hours. What infrastructure should you recommend?

A) Vertex AI Endpoints with GPU-accelerated instances and autoscaling set to scale to zero
B) Vertex AI Endpoints with GPU-accelerated instances, minimum 1 replica during business hours via scheduled scaling
C) GKE cluster with T4 GPUs and a cron job to scale the node pool on a schedule
D) Cloud Functions with the model loaded from Cloud Storage on each invocation

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI Endpoints with GPU-backed machines give managed serving for a 15 GB model, and keeping at least one replica warm during business hours removes cold-start latency from the sub-200ms path. Traffic is predictable, so a schedule matches capacity to the pattern.

- **A) is wrong** -- if the endpoint is idle at zero replicas, the first request after idle pays the cost of bringing up a GPU replica and loading 15 GB of weights, which is orders of magnitude beyond 200ms.
- **C) is wrong** -- self-managing GKE GPU node pools with cron-driven resizes is significant operational work for something the managed endpoint does natively.
- **D) is wrong** -- a function is the wrong runtime for a 15 GB model: loading it per instance start dominates the request, and the whole design fights the latency target.

**Exam tip:** separate two different latency claims. Steady-state inference latency is about the accelerator; first-request latency is about cold start and model load size. A stem that gives a large model size plus a tight latency target is testing whether you keep a warm replica.

Docs: https://cloud.google.com/vertex-ai/docs/predictions/configure-compute and https://cloud.google.com/vertex-ai/docs/general/deployment
</details>

---

### Q22. A company is designing a globally distributed application that must handle user authentication. Users are located in North America, Europe, and Asia-Pacific. The authentication service must respond within 50ms and maintain strong consistency for user session data. Which design meets these requirements?

A) Firebase Authentication with session data in Firestore (multi-region)
B) Custom auth service deployed in each region with Cloud Spanner for session data
C) Identity Platform with session data in Memorystore for Redis (per-region)
D) Custom auth service in a single region with Cloud CDN caching auth tokens

<details>
<summary>Answer</summary>

**Correct: B)**

Spanner gives strongly consistent reads with a leader placement you choose per configuration, so an auth service deployed in each region can serve session reads locally inside the 50ms budget while writes stay globally consistent.

- **A) is wrong**, but not for the reason people usually give. Firestore reads are strongly consistent by default, including in multi-region. The real problem is write latency: leader replicas sit in the primary region, so a write originating far from it pays a cross-region round trip that eats the 50ms budget.
- **C) is wrong** -- Memorystore for Redis is regional with no cross-region replication, so a user who authenticates in North America and is then routed to Europe has no session there.
- **D) is wrong** -- caching auth tokens at the CDN means a revoked token can still be served from cache, and a single-region auth service cannot hold 50ms for distant users.

**Exam tip:** "Firestore is eventually consistent" is a widespread and wrong shortcut. Firestore reads are strongly consistent; what varies is write latency relative to the leader region. When a stem sets a global latency budget, ask where writes are ordered, not whether reads are consistent.

Docs: https://cloud.google.com/spanner/docs/replication and https://cloud.google.com/firestore/docs
</details>

---

### Q23. An e-commerce platform expects a 20x traffic increase during a flash sale event in 2 weeks. Their current GKE-based architecture handles normal traffic well. The architect needs to ensure the system can handle the surge without service degradation. Which combination of actions should they take? (Choose TWO)

A) Run distributed load testing on GKE with an open-source tool such as Locust or k6 to identify bottlenecks and determine required capacity
B) Pre-warm the Cluster Autoscaler by setting minimum node count to expected peak capacity for the sale duration
C) Switch all workloads to Spot VMs to reduce cost during the sale
D) Configure Horizontal Pod Autoscaler with aggressive scaling policies (rapid scale-up, slow scale-down)
E) Migrate from GKE to Cloud Run for automatic scaling

<details>
<summary>Answer</summary>

**Correct: A) and D)**

Load testing before the event is what tells you where the system breaks and how much capacity the surge actually needs; running it as a distributed test on GKE with Locust or k6 generates enough load to be meaningful. Asymmetric HPA policies (fast scale-up, slow scale-down) then let pods track the surge without flapping.

- **B) is wrong** -- pinning minimum nodes at peak for the whole sale pays for peak capacity across the troughs too. Autoscaling with headroom is the cheaper way to get the same protection.
- **C) is wrong** -- Spot VMs can be reclaimed at any moment, which is unacceptable during the single most revenue-critical window of the year.
- **E) is wrong** -- re-platforming two weeks before a major event is the riskiest possible time to change architecture.

**Exam tip:** there is no Google product called "Cloud Load Testing". Load testing on Google Cloud means running an open-source generator (Locust, k6, JMeter) distributed on GKE or Compute Engine. An option naming a Google load-testing service is fabricated, and that pattern shows up in third-party question banks.

Docs: https://cloud.google.com/architecture/distributed-load-testing-using-gke and https://cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler
</details>

---

### Q24. An architect is designing a disaster recovery plan for a multi-tier application (web, app, database). They need to achieve RTO of 1 hour and RPO of 15 minutes while minimizing standby costs. The application is deployed in us-central1. Which DR strategy is most appropriate?

A) Hot standby: fully running duplicate environment in us-east1
B) Warm standby: scaled-down infrastructure in us-east1 with database replication, automated scaling on failover
C) Cold standby: infrastructure-as-code templates to recreate the environment in us-east1 from backups
D) Backup and restore: daily backups to multi-regional Cloud Storage, manual restore procedures

<details>
<summary>Answer</summary>

**Correct: B)**

Warm standby keeps a scaled-down copy of the stack in the DR region with continuous database replication, so RPO stays well inside 15 minutes, and autoscaling brings it to full size on failover inside the 1-hour RTO. It is the pattern that balances standby cost against recovery speed.

- **A) is wrong** -- hot standby meets the targets but at maximum cost, and the stem explicitly asks to minimise standby spend.
- **C) is wrong** -- cold standby rebuilds infrastructure from templates on the day, which for a multi-tier application typically overshoots a 1-hour RTO.
- **D) is wrong** -- daily backups give an RPO of up to 24 hours, and manual restore procedures are slow and error-prone.

**Exam tip:** memorise the DR ladder against its RTO band: backup and restore (hours to days), cold standby (hours), warm standby (minutes to an hour), hot or active-active (near zero). Map the stem's RTO onto the ladder and pick the cheapest rung that clears it.

Docs: https://cloud.google.com/architecture/dr-scenarios-planning-guide and https://cloud.google.com/architecture/dr-scenarios-building-blocks
</details>

---

## 1.3 Network, Storage, and Compute Design (Q25-Q44)

---

### Q25. A multinational corporation needs to connect their three on-premises data centers (US, EU, Asia) to GCP. Each data center needs at least 20 Gbps bandwidth to GCP, and the data centers need to communicate with each other through GCP's backbone network. What connectivity architecture should you design?

A) Three HA VPN connections, one per data center, with Cloud Router in each region
B) Three Dedicated Interconnect connections (2x10 Gbps each) with Cloud Routers in the respective GCP regions
C) Partner Interconnect in all three locations with a single Cloud Router
D) Three Dedicated Interconnect connections (2x10 Gbps each) with Cloud Routers and a Cloud VPN backup for each

<details>
<summary>Answer</summary>

**Correct: D)**

Each site needs 20 Gbps, which is Dedicated Interconnect territory (a bundle of 10 Gbps links, or a single higher-speed link), with a Cloud Router per region for BGP. Adding a Cloud VPN backup per site is what makes it a production design rather than a single-path one, and once all three sites terminate on Google, inter-site traffic can ride Google's backbone.

- **A) is wrong** -- HA VPN runs over the public internet and its throughput ceiling is documented in packets per second, so building a dependable 20 Gbps per site out of tunnels is impractical.
- **B) is wrong** -- it has the right bandwidth but no backup path. Losing one circuit disconnects a whole data centre.
- **C) is wrong** -- one Cloud Router for three sites is a single point of failure, and per-site Partner Interconnect bandwidth depends on what the partner offers.

**Exam tip:** on Interconnect questions, redundancy is graded separately from bandwidth. A single link carries no availability SLA; 99.9% needs two connections in different edge availability domains, and 99.99% needs four across two metros. If two options have the same bandwidth, the redundant one wins.

Docs: https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview and https://cloud.google.com/network-connectivity/docs/interconnect/tutorials/dedicated-creating-9999-availability
</details>

---

### Q26. A company wants to deploy a web application that serves users in Europe and North America. The application backend runs on GKE in us-central1 and europe-west1. They need HTTPS termination, URL-based routing (api/* vs static/*), and automatic failover between regions. Which load balancer configuration should they use?

A) Regional external Application Load Balancer in each region with Cloud DNS geographic routing
B) Global external Application Load Balancer with URL maps, backend services pointing to NEGs in both regions, and health checks
C) Global external proxy Network Load Balancer with SSL offloading
D) Two regional passthrough Network Load Balancers with Cloud Armor

<details>
<summary>Answer</summary>

**Correct: B)**

The global external Application Load Balancer terminates HTTPS, routes by path through URL maps, fails over between regions on health-check state, and presents one anycast IP so users land on the nearest healthy backend. Network endpoint groups connect it to the GKE pods in both regions.

- **A) is wrong** -- regional load balancers plus DNS geo-routing shifts failover into DNS, where TTL caching makes recovery slow and uneven.
- **C) is wrong** -- a proxy Network Load Balancer works at L4. URL-based routing is an L7 feature it does not have.
- **D) is wrong** -- passthrough Network Load Balancers do not terminate HTTPS or inspect paths at all.

**Exam tip:** three words in a stem decide the load balancer. "URL path" or "host-based" forces Application Load Balancer (L7). "Single anycast IP across regions" forces global. "Preserve client IP" or "non-HTTP protocol" forces passthrough Network Load Balancer.

Docs: https://cloud.google.com/load-balancing/docs/application-load-balancer
</details>

---

### Q27. A data science team needs to train a custom image classification model on 2 million labeled images. Training must complete within 8 hours and the team wants to minimize cost. The model architecture uses TensorFlow and benefits from parallel computation. Which compute configuration should you recommend?

A) Vertex AI Training with a single A100 GPU
B) Vertex AI Training with a cluster of T4 GPUs using data parallelism
C) Vertex AI Training with a TPU v3 pod slice
D) Dataproc cluster with GPU-accelerated workers

<details>
<summary>Answer</summary>

**Correct: C)**

A TPU pod slice on Vertex AI Training is built for large-scale data-parallel training of this shape, and the high-bandwidth inter-chip interconnect is what makes the scale-out efficient. Vertex AI manages the cluster lifecycle so you pay for training time only.

- **A) is wrong** -- a single accelerator does not use the parallelism the model benefits from, and may not finish 2 million images inside 8 hours.
- **B) is wrong** -- T4 is an inference-oriented GPU. A cluster of them costs more and trains slower than a TPU slice for this workload.
- **D) is wrong** -- Dataproc is a Spark and Hadoop service. It can attach GPUs, but it has none of the managed training lifecycle Vertex AI provides.

**Exam tip:** accelerator selection questions turn on the framework and the scale. TPUs suit large TensorFlow or JAX training that fits the XLA path; GPUs suit PyTorch, custom CUDA kernels and mixed workloads. Note that the TPU generation named in older question banks moves, so reason about the family rather than memorising a version.

Docs: https://cloud.google.com/tpu/docs/intro-to-tpu and https://cloud.google.com/vertex-ai/docs/training/overview
</details>

---

### Q28. An IoT platform receives telemetry from 1 million devices, each sending a 1 KB message every 10 seconds. The data must be stored for real-time querying by device ID and time range, with queries completing in under 10ms. Historical data older than 90 days should be moved to cheaper storage for analytics. Which storage architecture should you design?

A) Bigtable for real-time data with a lifecycle policy exporting to BigQuery after 90 days
B) Cloud SQL with partitioning by date, archiving old partitions to Cloud Storage
C) Firestore with TTL policy, Cloud Functions copying expiring documents to BigQuery
D) Memorystore for Redis with persistence, periodic snapshots to Cloud Storage

<details>
<summary>Answer</summary>

**Correct: A)**

At 1 million devices reporting every 10 seconds this is 100,000 writes/sec, which is Bigtable's shape: high sustained write throughput with single-digit millisecond reads by row key. A row key of device ID plus reversed timestamp serves the "by device and time range" query directly, and a scheduled export moves data past 90 days into BigQuery for cheap analytics.

- **B) is wrong** -- Cloud SQL cannot take 100,000 writes/sec, and single-row latency degrades well past 10ms under that load.
- **C) is wrong** -- Firestore is a document store for application data, not a high-rate time-series sink.
- **D) is wrong** -- holding 90 days of this stream in memory is neither feasible nor affordable; Memorystore is a cache, not a time-series store.

**Exam tip:** do the write-rate arithmetic before choosing. Devices multiplied by frequency gives writes per second, and that number alone eliminates most options. Bigtable is the answer whenever the stem combines very high write throughput, key-and-range lookups, and a millisecond latency target.

Docs: https://cloud.google.com/bigtable/docs/schema-design-time-series and https://cloud.google.com/bigtable/docs/overview
</details>

---

### Q29. A company is designing a VPC network architecture for a multi-team organization. They have a central security team, three application teams, and shared services (logging, monitoring, CI/CD). All teams need to access shared services, but application teams should not be able to communicate directly with each other. What VPC design should you implement?

A) A single VPC with subnets per team and firewall rules to block inter-team traffic
B) Shared VPC with the host project managed by the security team, separate service projects per application team, and firewall rules to prevent direct inter-team communication
C) Separate VPCs per team with VPC peering between each application VPC and the shared services VPC
D) Separate VPCs per team with full mesh VPC peering between all VPCs

<details>
<summary>Answer</summary>

**Correct: C)**

Separate VPCs per application team peered to a shared-services VPC gives each team its own network boundary. VPC peering is non-transitive, so Team A cannot reach Team B through the hub, which is precisely the isolation requirement, while all teams reach shared services directly.

- **A) is wrong** -- one flat VPC makes isolation depend entirely on firewall rules being right, forever. One bad rule joins the teams together.
- **B) is wrong** -- Shared VPC puts every service project on the host project's network, so isolation again reduces to firewall configuration rather than a network boundary.
- **D) is wrong** -- full mesh peering deliberately connects every team to every other team, which is the opposite of the requirement.

**Exam tip:** non-transitivity of VPC peering is tested constantly, and it cuts both ways. It is the flaw when you want spoke-to-spoke traffic through a hub, and it is the feature when you want spokes isolated from each other. Read the stem to see which side it is asking for.

Docs: https://cloud.google.com/vpc/docs/vpc-peering and https://cloud.google.com/vpc/docs/shared-vpc
</details>

---

### Q30. A healthcare company needs to store and query genomic data (petabyte-scale). Queries involve complex joins across multiple datasets and must support standard SQL. Data scientists need to run ad-hoc queries without managing infrastructure. Which storage and query solution should you recommend?

A) Cloud Storage with Dataproc running Hive for SQL queries
B) BigQuery with external tables on Cloud Storage for raw data and native tables for frequently queried datasets
C) Cloud Spanner for structured genomic data with SQL support
D) AlloyDB with columnar engine for analytical queries

<details>
<summary>Answer</summary>

**Correct: B)**

BigQuery is serverless and petabyte-scale with full GoogleSQL joins, so data scientists get self-service ad-hoc querying with no infrastructure to run. External tables keep raw genomic files in Cloud Storage without duplicating them, while native tables hold the frequently queried datasets for best performance.

- **A) is wrong** -- Dataproc means sizing, scaling and upgrading a cluster, which the stem rules out, and Hive is slower than BigQuery for ad-hoc SQL.
- **C) is wrong** -- Spanner is an OLTP database. Petabyte-scale analytical joins are not its workload.
- **D) is wrong** -- AlloyDB is a regional PostgreSQL-compatible database with a per-cluster storage ceiling far below petabyte scale, and it is an instance you size rather than a serverless service.

**Exam tip:** "ad-hoc SQL" plus "no infrastructure management" plus "petabyte" is BigQuery every time. The native-versus-external table split is the follow-on decision: native for hot, frequently joined data, external to query cold data in place without a load step.

Docs: https://cloud.google.com/bigquery/docs/external-tables and https://cloud.google.com/bigquery/docs/introduction
</details>

---

### Q31. An architect needs to design a hybrid cloud network where on-premises applications access Google APIs (BigQuery, Cloud Storage, Pub/Sub) without traffic traversing the public internet. The on-premises network connects to GCP via Dedicated Interconnect. What should you configure?

A) Private Google Access on the VPC subnets
B) Private Service Connect endpoints for each Google API, with custom DNS forwarding from on-premises
C) Private Google Access for on-premises hosts using the restricted.googleapis.com VIP range (199.36.153.4/30) with Cloud DNS and on-premises DNS forwarding
D) A Cloud NAT gateway in the VPC for on-premises traffic

<details>
<summary>Answer</summary>

**Correct: C)**

Reaching Google APIs from on-premises over Interconnect uses the restricted VIP range 199.36.153.4/30 for restricted.googleapis.com. On-premises DNS forwards googleapis.com to that VIP, routes send it over the Interconnect, and the traffic never touches the public internet.

- **A) is wrong** -- plain Private Google Access covers VM instances inside a subnet that have no external IP. It says nothing about on-premises hosts.
- **B) is wrong** -- Private Service Connect endpoints do provide private API access, but the option omits the routing and DNS work that actually makes the on-premises path resolve and route.
- **D) is wrong** -- Cloud NAT gives VMs without external IPs outbound internet access. It does not carry on-premises traffic to Google APIs.

**Exam tip:** the give-away here is the phrase "on-premises hosts". Private Google Access has two distinct forms, and the hybrid one is defined by the restricted or private VIP plus DNS forwarding. Memorise 199.36.153.4/30 as restricted.googleapis.com, which excludes services not supported by VPC Service Controls.

Docs: https://cloud.google.com/vpc/docs/private-google-access-hybrid and https://cloud.google.com/vpc/docs/private-google-access
</details>

---

### Q32. A company needs to run a stateful Windows application that requires persistent local SSD performance, Windows Server licensing, and 64 vCPUs. They already own Windows Server licenses through a Microsoft Enterprise Agreement. How should they deploy this on GCP to minimize cost?

A) Compute Engine with a sole-tenant node and bring-your-own-license (BYOL), local SSDs attached
B) Compute Engine with a standard n2-standard-64 instance, Windows Server license included, and Persistent Disk SSD
C) GKE Windows node pool with local SSDs
D) Bare metal solution with Windows Server installed

<details>
<summary>Answer</summary>

**Correct: A)**

Sole-tenant nodes give dedicated physical hardware, which is what Microsoft's Outsourcing Software Management Rights require for bringing your own Windows Server OS licence. Local SSDs supply the highest IOPS and lowest latency, and reusing owned licences avoids Google's per-core Windows premium.

- **B) is wrong** -- it pays Google's bundled Windows licence on top of licences the company already owns, and Persistent Disk SSD is slower than Local SSD.
- **C) is wrong** -- GKE Windows node pools run Windows containers, not a full stateful Windows Server application, and local SSD on a node is ephemeral node storage.
- **D) is wrong** -- Bare Metal Solution exists for specialised licensed workloads such as Oracle; it is a far more expensive way to run a Windows application.

**Exam tip:** Windows Server OS licences and application-server licences such as SQL Server behave differently. The OS route generally needs sole-tenant nodes; SQL Server with active Software Assurance qualifies for License Mobility and can run on ordinary multi-tenant VMs. Check which product and whether Software Assurance is mentioned.

Docs: https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes and https://cloud.google.com/compute/docs/instances/windows/ms-licensing
</details>

---

### Q33. A media company needs to process and transcode 4K video files (average 50 GB each, 100 files per day). The processing is highly parallel, with each file taking 30 minutes on a GPU-enabled machine. Files arrive throughout the day, and processed output must be available within 2 hours. What compute architecture minimizes cost?

A) A persistent GKE cluster with GPU node pools, autoscaling from 0 to 10 nodes
B) Batch API with GPU-enabled VMs, jobs triggered by Cloud Storage events via Eventarc
C) Compute Engine GPU instances running 24/7 behind an internal load balancer
D) Vertex AI Custom Training jobs triggered per file upload

<details>
<summary>Answer</summary>

**Correct: B)**

Google Cloud Batch provisions GPU-enabled VMs when a job is submitted and releases them when it finishes, so the bill tracks the roughly 50 GPU-hours a day of real work. Eventarc turns each Cloud Storage object finalisation into a job submission, and Batch runs the files in parallel inside the 2-hour window.

- **A) is wrong** -- a GKE cluster carries control-plane and platform overhead for what is a plain batch queue, and GPU node scale-up is slow.
- **C) is wrong** -- 24/7 GPU instances are billed for 24 hours to do about two GPUs' worth of work.
- **D) is wrong** -- Vertex AI Custom Training is ML training infrastructure. Video transcoding is not an ML job and gains nothing from it.

**Exam tip:** "jobs arrive, run for a while, then stop" is Batch. Reach for GKE only when the stem needs long-running services, service discovery or a mesh. An event-driven trigger from Cloud Storage is Eventarc, and pairing Eventarc with Batch is a pattern worth recognising on sight.

Docs: https://cloud.google.com/batch/docs/get-started and https://cloud.google.com/eventarc/docs/overview
</details>

---

### Q34. An organization is migrating to GCP and needs to design their database strategy. They have: (1) a transactional e-commerce system, (2) a user session store, (3) a product catalog with hierarchical categories, and (4) a clickstream analytics platform. Match each workload to the most appropriate database service. (Choose the correct combination)

A) (1) Cloud SQL, (2) Memorystore, (3) Firestore, (4) BigQuery
B) (1) Cloud Spanner, (2) Memorystore, (3) Cloud SQL, (4) Bigtable
C) (1) AlloyDB, (2) Bigtable, (3) Firestore, (4) BigQuery
D) (1) Cloud SQL, (2) Firestore, (3) Memorystore, (4) BigQuery

<details>
<summary>Answer</summary>

**Correct: A)**

Each workload lands on the service designed for its access pattern: Cloud SQL for ACID transactions over a relational order model, Memorystore for a low-latency key-value session store with TTLs, Firestore for hierarchical catalogue documents, and BigQuery for clickstream analytics.

- **B) is wrong** -- Spanner is more than an ordinary e-commerce system needs, and Bigtable is a poor session store next to Redis.
- **C) is wrong** -- Bigtable again fails the session-store role, where the requirement is microsecond key lookups with expiry.
- **D) is wrong** -- Memorystore is a cache, so it is the wrong home for a durable product catalogue, and Firestore is the wrong home for session data here.

**Exam tip:** matching questions are fastest solved by finding the one obviously wrong pairing and eliminating every option that contains it. The session store is usually that pairing: sessions mean Memorystore unless the stem demands durability, in which case Firestore.

Docs: https://cloud.google.com/memorystore/docs/redis and https://cloud.google.com/firestore/docs
</details>

---

### Q35. A company needs to deploy a private GKE cluster that has no public endpoint and runs workloads that need to pull container images from Artifact Registry and access BigQuery. Pods must not have external IP addresses. How should you configure network access?

A) Enable Private Google Access on the subnet, configure Cloud NAT for external dependencies, and use authorized networks for cluster access
B) Enable Private Google Access on the subnet, no Cloud NAT needed, use Connect Gateway or IAP for cluster access
C) Configure a proxy server in the VPC for all outbound traffic
D) Use VPC peering to connect to Google's API network

<details>
<summary>Answer</summary>

**Correct: B)**

Private Google Access on the subnet lets nodes and pods without external IPs reach Artifact Registry and BigQuery over Google's network. Cloud NAT is not needed because nothing in the stem requires non-Google internet egress, and Connect gateway or IAP gives operators a management path without a public control-plane endpoint.

- **A) is wrong** -- Cloud NAT here is cost and attack surface added for a requirement that was never stated.
- **C) is wrong** -- a self-run proxy is operational overhead and a new failure point for something Private Google Access already does.
- **D) is wrong** -- you cannot peer a VPC to Google's API network. Private Google Access and Private Service Connect are the supported mechanisms.

**Exam tip:** distinguish "reach Google APIs" from "reach the internet". The first is Private Google Access or Private Service Connect; only the second needs Cloud NAT. Adding Cloud NAT when the stem lists only Google services is the classic over-answer.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/private-cluster-concept and https://cloud.google.com/vpc/docs/private-google-access
</details>

---

### Q36. A financial services company needs a data processing pipeline that ingests stock market data at 1 million events per second, computes real-time aggregations (moving averages over 5-minute windows), and serves results to trading applications with sub-second latency. Which architecture should you design?

A) Pub/Sub -> Dataflow (streaming with windowing) -> Bigtable for serving
B) Pub/Sub -> Cloud Functions -> Cloud SQL -> application queries
C) Kafka on GKE -> Spark Streaming on Dataproc -> Redis on Memorystore
D) Pub/Sub -> Dataflow -> BigQuery streaming inserts -> application queries BigQuery

<details>
<summary>Answer</summary>

**Correct: A)**

Pub/Sub takes the 1M events/sec ingest, Dataflow computes the 5-minute windowed aggregations with Beam's windowing model and exactly-once semantics, and Bigtable serves the computed values back to trading applications with single-digit millisecond key lookups.

- **B) is wrong** -- functions plus Cloud SQL fails on both throughput and read latency at this rate.
- **C) is wrong** -- self-managed Kafka and Spark can do it, but the operational load is large next to a managed Pub/Sub and Dataflow pipeline.
- **D) is wrong** -- BigQuery is an analytical engine. Its floor for query latency sits above the sub-second point-lookup requirement.

**Exam tip:** separate the compute layer from the serving layer. Windowed aggregation is Dataflow; who reads the result decides the sink. Dashboards and analysts mean BigQuery, applications needing millisecond key lookups mean Bigtable. A stem mentioning both a window and a latency budget is testing exactly that split.

Docs: https://cloud.google.com/dataflow/docs and https://cloud.google.com/bigtable/docs/overview
</details>

---

### Q37. An architect is designing a multi-cloud strategy where the company runs workloads on both GCP and AWS. They need a consistent container orchestration platform across both clouds with centralized policy management and service mesh. What should they use?

A) GKE on GCP and EKS on AWS, managed independently with separate CI/CD pipelines
B) GKE Enterprise with GKE on Google Cloud and attached clusters on AWS (EKS), managed as a fleet with Config Sync and Cloud Service Mesh
C) Cloud Run on GCP and AWS Lambda, with Terraform managing both
D) GKE Autopilot on GCP and self-managed Kubernetes on AWS EC2 instances

<details>
<summary>Answer</summary>

**Correct: B)**

GKE Enterprise (the product formerly sold as Anthos) manages clusters across clouds as a fleet, including attached EKS clusters on AWS. Config Sync applies GitOps-based configuration and policy consistently across the fleet, and Cloud Service Mesh (formerly Anthos Service Mesh) extends the mesh across both environments.

- **A) is wrong** -- independently managed GKE and EKS gives neither centralised policy nor a shared mesh, which are the two explicit requirements.
- **C) is wrong** -- Cloud Run and Lambda are different platforms with different contracts. Terraform provisions infrastructure but does not create runtime consistency or a mesh.
- **D) is wrong** -- self-managed Kubernetes on EC2 adds operational burden and still has no fleet-wide policy layer.

**Exam tip:** know the renames, because the exam guide and the console disagree with older material. Anthos is GKE Enterprise, Anthos Config Management is Config Sync and Policy Controller, and Anthos Service Mesh plus Traffic Director are Cloud Service Mesh. The multi-cloud requirement pattern is fleet plus GitOps plus mesh.

Docs: https://cloud.google.com/kubernetes-engine/enterprise/docs and https://cloud.google.com/kubernetes-engine/fleet-management/docs
</details>

---

### Q38. A startup is building a generative AI application that needs to use a foundation model for text generation, add grounding with their proprietary data, and deploy the solution with minimal ML expertise. Which GCP services should they use? (Choose TWO)

A) Vertex AI Model Garden to select and deploy a foundation model
B) Train a custom transformer model from scratch on Vertex AI Training with TPUs
C) Vertex AI Agent Builder (Search and Conversation) for grounding with proprietary data
D) Deploy an open-source model on a GKE cluster with custom serving infrastructure
E) Use Cloud Functions to call the OpenAI API directly

<details>
<summary>Answer</summary>

**Correct: A) and C)**

Model Garden gives a curated catalogue of Google, open-weight and third-party foundation models that can be deployed without ML engineering. Agent Builder grounds the chosen model in the company's own data through managed retrieval-augmented generation, which is the grounding requirement, again without deep ML work. (Both now sit under the Gemini Enterprise Agent Platform branding; the exam guide still names them Vertex AI.)

- **B) is wrong** -- training a transformer from scratch is the maximum-ML-expertise answer to a minimum-ML-expertise requirement.
- **D) is wrong** -- self-hosting an open model on GKE means owning serving infrastructure and Kubernetes operations.
- **E) is wrong** -- calling a third-party API from a function abandons the managed grounding, governance and monitoring the stem is implicitly asking for.

**Exam tip:** in generative AI scenarios, map the verb to the product. "Pick or deploy a model" is Model Garden. "Ground answers in our documents" is Agent Builder with a data store, which is managed RAG. "Match our house style from many examples" is fine-tuning. Only the last one justifies training.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models and https://cloud.google.com/generative-ai-app-builder/docs/introduction
</details>

---

### Q39. A retail company has 200 stores, each with a local database that syncs inventory data to a central system. They need occasional connectivity tolerance (stores may lose internet for up to 4 hours), real-time inventory queries when online, and eventual consistency is acceptable for cross-store queries. Which database architecture supports this?

A) Cloud Spanner with a multi-region configuration
B) Firestore in each store (offline mode) syncing to a central Firestore database
C) Cloud SQL in each store syncing to a central Cloud SQL via custom replication
D) Bigtable as the central database with edge caching at each store

<details>
<summary>Answer</summary>

**Correct: B)**

Firestore's client SDKs cache data locally and reconcile automatically when connectivity returns, so a store keeps reading and writing through a 4-hour outage. Real-time listeners push updates when online, and eventual consistency across stores is inherent to the sync model.

- **A) is wrong** -- Spanner requires connectivity to the service. There is no offline mode, so a disconnected store cannot read or write.
- **C) is wrong** -- Cloud SQL has no offline or sync capability. Hand-building conflict resolution and ordering across 200 sites is a large custom system.
- **D) is wrong** -- Bigtable has no offline mode, and "edge caching" is not a Bigtable feature, let alone one that accepts writes while disconnected.

**Exam tip:** "works while disconnected and syncs later" is Firestore's offline persistence and almost nothing else on Google Cloud. When a stem names an intermittent-connectivity constraint, that phrase alone selects the answer, and the acceptance of eventual consistency in the stem is the confirmation.

Docs: https://cloud.google.com/firestore/docs/manage-data/enable-offline and https://cloud.google.com/firestore/docs
</details>

---

### Q40. A company is building a data lake on GCP. They have structured data (10 TB, relational), semi-structured data (50 TB, JSON logs), and unstructured data (200 TB, images and videos). They need cost-effective storage with the ability to run SQL analytics across structured and semi-structured data. What storage architecture should you design?

A) Everything in BigQuery native tables
B) Cloud Storage for all data (structured as CSV, semi-structured as JSON, unstructured as original format), with BigQuery external tables for SQL analytics
C) Cloud Storage for unstructured data, BigQuery native tables for structured data, BigQuery external tables over Cloud Storage for semi-structured data
D) Separate Cloud SQL databases for structured data, Cloud Storage for everything else, Dataproc for analytics

<details>
<summary>Answer</summary>

**Correct: C)**

Each data class goes where it belongs. Structured data in BigQuery native tables gets the best query performance plus partitioning and clustering. Semi-structured JSON stays on Cloud Storage and is queried through external tables, avoiding a duplicate copy. Images and video sit in Cloud Storage, which is the only sensible home for 200 TB of binary objects.

- **A) is wrong** -- BigQuery native tables are not a store for video files, and loading 50 TB of JSON natively adds storage cost for no query benefit here.
- **B) is wrong** -- pushing the frequently queried structured data out to CSV on Cloud Storage trades away exactly the query performance native tables provide.
- **D) is wrong** -- Cloud SQL is OLTP and not sized for 10 TB of analytics, and Dataproc reintroduces cluster management next to serverless BigQuery.

**Exam tip:** the native-versus-external decision is about query frequency, not data type. Hot and frequently joined goes native; cold, large or already-landed data is queried in place. Unstructured binary never goes into BigQuery, whatever the option says.

Docs: https://cloud.google.com/bigquery/docs/external-data-cloud-storage and https://cloud.google.com/bigquery/docs/external-tables
</details>

---

### Q41. A company needs to serve a machine learning model that requires NVIDIA A100 GPUs. They expect steady traffic during business hours (10 AM - 6 PM) and minimal traffic overnight. The model requires 4 GPUs per replica. Which deployment minimizes cost while meeting performance needs?

A) GKE cluster with a GPU node pool (4x A100 per node), autoscaling from 1-5 nodes, with Spot VMs for the GPU node pool
B) Vertex AI Endpoints with a100 machine type, autoscaling with minimum 1, maximum 5 replicas
C) Compute Engine instances with 4x A100 GPUs, managed instance group with autoscaling, using scheduled scaling for business hours
D) GKE cluster with a dedicated non-spot GPU node pool running 24/7

<details>
<summary>Answer</summary>

**Correct: C)**

A managed instance group with scheduled scaling matches capacity to a known business-hours pattern: full capacity from 10 AM to 6 PM, scaled down for the other 14 hours, with autoscaling handling variation inside the day. Against A100 pricing, those 14 hours are where the saving lives.

- **A) is wrong** -- Spot capacity can be reclaimed at any time, and GPU Spot capacity is among the most contended. That is not a basis for serving user-facing predictions.
- **B) is wrong** -- the option pins a minimum of one replica, so four A100s are billed around the clock including the quiet overnight window. Scheduled scaling on a MIG tracks the stated pattern more closely.
- **D) is wrong** -- a non-Spot GPU node pool running 24/7 pays for the expensive accelerators through 14 hours of minimal traffic.

**Exam tip:** predictable, clock-driven demand is a scheduled-scaling signal; unpredictable demand is a metric-driven autoscaling signal. Check what the option actually configures rather than what the service could do -- an option that sets a minimum replica count has told you its own floor cost.

Docs: https://cloud.google.com/compute/docs/autoscaler and https://cloud.google.com/vertex-ai/docs/predictions/configure-compute
</details>

---

### Q42. An architect is designing a network for a regulated environment where all egress traffic must pass through a centralized firewall appliance for inspection. The environment has 10 projects across three environments (dev, staging, prod). What network topology should you implement?

A) Shared VPC with custom routes directing egress through a firewall appliance in the host project
B) Individual VPCs per project with VPC peering to a hub VPC containing the firewall appliance
C) Hub-and-spoke topology using a hub VPC with the firewall appliance, spoke VPCs connected via VPN tunnels, and custom routes for egress
D) Each project with its own VPC and independent Cloud NAT gateways

<details>
<summary>Answer</summary>

**Correct: A)**

Shared VPC puts all ten projects on one network owned by the security team's host project, so a custom route in the host project sends egress through the inspection appliance for every service project at once. There is nothing per-project to keep in sync.

- **B) is wrong** -- VPC peering is non-transitive and does not propagate custom routes between peerings, so spokes cannot forward egress to an appliance in the hub this way.
- **C) is wrong** -- hub-and-spoke over VPN adds tunnels, bandwidth limits and operational complexity that ten projects do not justify when Shared VPC solves it.
- **D) is wrong** -- per-project Cloud NAT gives each project its own direct internet exit, which bypasses the inspection requirement entirely.

**Exam tip:** centralised egress inspection has two shapes. Inside one organisation with a modest project count, Shared VPC plus custom routes is the simple answer. Hub-and-spoke with an appliance or Network Connectivity Center is for scale or for connecting networks you do not own. Both are defeated by per-project Cloud NAT, which is always the distractor.

Docs: https://cloud.google.com/vpc/docs/shared-vpc and https://cloud.google.com/vpc/docs/routes
</details>

---

### Q43. A company wants to use AI/ML to detect manufacturing defects in real time from camera feeds at 20 factories. Each factory has 10 cameras sending 30 frames per second. The model must respond within 100ms per frame. Network bandwidth to the cloud is limited (50 Mbps per factory). What architecture should you design?

A) Stream all video to GCP, process with Vertex AI Endpoints running a custom vision model
B) Deploy Google Distributed Cloud (GDC) Edge at each factory running the vision model, send only defect alerts and summary data to GCP
C) Use Cloud Vision API from each factory for real-time defect detection
D) Send video to Cloud Storage, process with Vertex AI batch prediction pipeline

<details>
<summary>Answer</summary>

**Correct: B)**

Ten cameras at 30 fps is 300 frames per second per factory, far beyond what a 50 Mbps uplink can carry as raw video, and a cloud round trip would not hold 100ms anyway. Google Distributed Cloud Edge runs the vision model inside the factory and sends only alerts and summaries to Google Cloud.

- **A) is wrong** -- streaming raw video breaks the stated bandwidth constraint before latency is even considered.
- **C) is wrong** -- Cloud Vision API means sending every frame to the cloud, which hits the same bandwidth and latency walls, and it is a general-purpose API rather than a defect model.
- **D) is wrong** -- batch prediction processes stored data after the fact and cannot meet a 100ms per-frame requirement.

**Exam tip:** two clues force an edge answer: a bandwidth ceiling that the raw data exceeds, and a latency budget shorter than a cloud round trip. When both appear, the answer processes locally and ships only results. Compute the data rate from the stem rather than trusting the option's adjectives.

Docs: https://cloud.google.com/distributed-cloud/edge/latest/docs
</details>

---

### Q44. A company is designing a multi-region active-active application. They need a global database that supports strong consistency, automatic replication, and sub-10ms reads in each region. Write operations happen in any region. The expected data volume is 5 TB with 10,000 reads per second per region and 1,000 writes per second globally. Which database should they choose?

A) Cloud Spanner with a multi-region instance configuration
B) Firestore in multi-region mode
C) Cloud SQL with cross-region read replicas
D) Bigtable with multi-cluster replication

<details>
<summary>Answer</summary>

**Correct: A)**

Spanner multi-region is the one Google Cloud database offering external consistency with synchronous replication and writes accepted from any region. With appropriate read staleness settings it serves the local sub-10ms reads, and the stated 10,000 reads/sec per region and 1,000 writes/sec globally are well inside its range.

- **B) is wrong** -- Firestore reads are strongly consistent, but writes are ordered through leader replicas in the primary region, so globally distributed writes pay a cross-region round trip.
- **C) is wrong** -- Cloud SQL read replicas are asynchronous and there is no multi-primary write path, so writes go to one region.
- **D) is wrong** -- Bigtable multi-cluster replication is eventually consistent, which the stem rules out.

**Exam tip:** "strong consistency plus writes in any region" is Spanner and only Spanner. Do not eliminate Firestore for being eventually consistent, because it is not; eliminate it for write locality. Bigtable and Cloud SQL replicas are the genuinely eventually consistent options in this family of questions.

Docs: https://cloud.google.com/spanner/docs/replication and https://cloud.google.com/spanner/docs/instance-configurations
</details>

---

## 1.4 Migration Planning (Q45-Q56)

---

### Q45. A large enterprise is planning to migrate 500 workloads to GCP over 18 months. They have a mix of legacy monoliths, modern microservices, databases, and custom hardware-dependent applications. Which is the correct first step in planning this migration?

A) Begin migrating the simplest workloads immediately to demonstrate quick wins
B) Use Migration Center to assess and discover all workloads, dependencies, and performance characteristics
C) Refactor all applications to cloud-native architectures before migrating
D) Set up Dedicated Interconnect and begin lifting and shifting all workloads simultaneously

<details>
<summary>Answer</summary>

**Correct: B)**

Migration Center is Google Cloud's discovery and assessment product. It inventories the estate, maps dependencies, captures performance profiles and produces right-sized target recommendations and a TCO view. With 500 mixed workloads, that assessment is what makes wave planning possible at all.

- **A) is wrong** -- moving before discovery risks cutting a dependency nobody documented. Quick wins are a sequencing choice made after assessment, not instead of it.
- **C) is wrong** -- refactoring everything first delays any business value until the whole portfolio is rewritten, and the 6 R's exist precisely because different workloads warrant different treatment.
- **D) is wrong** -- lifting 500 workloads simultaneously with no assessment ignores dependencies, compliance scope and sizing.

**Exam tip:** in migration questions, assess before you move, always. The order is assess, plan waves, deploy the landing zone, migrate, then optimise. Any option that starts with migrating or refactoring is wrong at step one, however sensible the rest of it sounds.

Docs: https://cloud.google.com/migration-center/docs/migration-center-overview and https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---

### Q46. A company has a legacy Oracle database (30 TB) running mission-critical workloads. The database uses Oracle-specific features (PL/SQL packages, materialized views, advanced querying). They want to migrate to GCP and reduce licensing costs. What migration strategy should you recommend?

A) Rehost: Lift and shift Oracle to Bare Metal Solution, then gradually replatform to AlloyDB
B) Replatform: Migrate directly to Cloud SQL for PostgreSQL using Database Migration Service
C) Refactor: Rewrite all database code for Cloud Spanner
D) Retire: Decommission the Oracle database and rebuild on Firestore

<details>
<summary>Answer</summary>

**Correct: A)**

A phased path suits a mission-critical Oracle estate. Rehosting onto Bare Metal Solution gets the workload off owned hardware while keeping full Oracle compatibility including PL/SQL packages, and the replatform to AlloyDB then happens incrementally with the business still running.

- **B) is wrong** -- Database Migration Service does support Oracle to PostgreSQL, but moving 30 TB with heavy PL/SQL in one step is a large refactor disguised as a migration, and high risk for a mission-critical system.
- **C) is wrong** -- rewriting for Spanner is the most expensive path. Spanner's data model and lack of stored procedures force an application rewrite, not a database migration.
- **D) is wrong** -- Firestore is a document database and cannot host a relational workload with complex SQL and PL/SQL.

**Exam tip:** two-step migrations are usually right when the stem says mission-critical and names vendor-specific database features. Bare Metal Solution is the Google answer for Oracle workloads that must stay Oracle; AlloyDB is the answer for PostgreSQL-compatible modernisation with Oracle-like performance expectations.

Docs: https://cloud.google.com/bare-metal/docs and https://cloud.google.com/alloydb/docs
</details>

---

### Q47. A company is migrating a large e-commerce application from on-premises to GCP using the strangler fig pattern. The application has a monolithic architecture with 12 tightly coupled modules. Which approach correctly implements this pattern?

A) Migrate all 12 modules simultaneously to Cloud Run microservices
B) Identify a low-risk, loosely coupled module (e.g., product reviews), extract it as a microservice on GCP, route traffic to the new service, then repeat for remaining modules
C) Create a complete replica of the monolith on Compute Engine, then refactor modules one by one
D) Build all 12 microservices on GCP first, then cut over from the monolith in a single switchover

<details>
<summary>Answer</summary>

**Correct: B)**

The strangler fig pattern extracts one module at a time behind a routing layer. Starting with a low-risk, loosely coupled module such as product reviews proves the pattern cheaply; a gateway or reverse proxy then decides per request whether it goes to the monolith or the new service, and the monolith shrinks until it can be retired.

- **A) is wrong** -- moving all twelve modules at once is a big-bang migration by definition, which is what the pattern exists to avoid.
- **C) is wrong** -- replicating the monolith onto Compute Engine is a rehost. Refactoring afterwards is a valid strategy, but it is not this pattern.
- **D) is wrong** -- building everything then switching once is a parallel build with a big-bang cutover, again the opposite of incremental traffic migration.

**Exam tip:** the defining feature of the strangler fig is that production traffic is split between old and new during the transition. If an option has a single cutover moment, it is not the strangler fig regardless of how gradual the build was.

Docs: https://cloud.google.com/architecture/microservices-architecture-refactoring-monoliths and https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---

### Q48. A company needs to migrate 200 VMs from VMware vSphere to GCP. The migration must minimize downtime and the applications cannot be significantly modified. Which migration tool and approach should they use?

A) Migrate to Virtual Machines (M2VM) with continuous replication, testing in GCP, then final cutover with minimal downtime
B) Export all VMDKs to Cloud Storage, then create Compute Engine images manually
C) Rebuild all applications as containers on GKE
D) Use Transfer Appliance to physically ship server images to Google

<details>
<summary>Answer</summary>

**Correct: A)**

Migrate to Virtual Machines performs continuous block-level replication from VMware into Google Cloud, lets you boot and test the migrated VM before committing, and keeps the final cutover down to minutes. No application changes are needed, which the stem requires.

- **B) is wrong** -- exporting VMDKs and building images by hand is slow, manual and needs a long downtime window per VM, with no way to catch up changes made during the copy.
- **C) is wrong** -- the stem says applications cannot be significantly modified, and containerising 200 of them is exactly that.
- **D) is wrong** -- Transfer Appliance moves bulk data. It does not capture VM configuration, networking or running OS state.

**Exam tip:** "minimise downtime" plus "no application changes" is the lift-and-shift signature, and the tool is Migrate to Virtual Machines. Note the lineage, since older material uses old names: Migrate for Compute Engine and Velostrata became Migrate to Virtual Machines, which is a different product from Migration Center.

Docs: https://cloud.google.com/migrate/virtual-machines/docs
</details>

---

### Q49. A company is running Microsoft SQL Server Enterprise on-premises with Software Assurance. They want to migrate to GCP and minimize licensing costs. What should they consider? (Choose TWO)

A) Use Cloud SQL for SQL Server with Google-provided licenses (included in pricing)
B) Use Compute Engine sole-tenant nodes with BYOL to leverage their existing Software Assurance licenses
C) Migrate to AlloyDB to eliminate SQL Server licensing entirely
D) Use Cloud SQL for SQL Server and additionally pay for their own SQL Server licenses
E) Run SQL Server on standard (multi-tenant) Compute Engine instances with BYOL

<details>
<summary>Answer</summary>

**Correct: C) and E)**

The stem specifies Software Assurance, which is what makes this answerable. SQL Server is an application server product eligible for **License Mobility through Software Assurance**, so the licences can move to standard multi-tenant Compute Engine with no dedicated hardware requirement. That is the cheapest way to reuse licences they already own. Migrating to AlloyDB is the other lever, eliminating SQL Server licensing entirely if the application can be adapted.

- **A) is wrong** -- Google-provided SQL Server Enterprise licensing is charged per vCPU and is expensive when you already own licences.
- **B) is wrong** -- sole-tenant nodes are not required here. Sole-tenancy is what you need under **Outsourcing Software Management Rights**, the route for customers without License Mobility, and using it anyway adds a sole-tenancy premium for no licensing benefit.
- **D) is wrong** -- Cloud SQL bundles licensing into its price, so paying separately means paying twice.

**Exam tip:** on Microsoft licensing questions, find out whether the stem mentions Software Assurance. With SA you get License Mobility and multi-tenant is fine; without it you are into Outsourcing Software Management Rights and sole-tenant nodes. Windows Server OS licences behave differently from application server licences like SQL Server.

Docs: https://cloud.google.com/compute/docs/instances/windows/ms-licensing and https://cloud.google.com/compute/docs/nodes/sole-tenant-nodes
</details>

---

### Q50. A retail bank is migrating its core banking platform (mainframe-based, COBOL) to GCP. The migration must be zero-downtime and maintain regulatory compliance throughout. Which migration approach and timeline expectation is most realistic?

A) Big-bang cutover over a weekend, rehosting the mainframe application on Compute Engine
B) Phased migration using the strangler fig pattern over 2-3 years, starting with read-only services, using Dual Writes for data synchronization during transition
C) Complete refactoring of all COBOL code to Java on GKE, then migrate
D) Use Database Migration Service to migrate the mainframe database directly to Cloud Spanner

<details>
<summary>Answer</summary>

**Correct: B)**

Core banking on a mainframe calls for incremental migration. The strangler fig pattern moves functionality piece by piece, starting with read-only services such as balance enquiry where a mistake is recoverable, while dual writes keep mainframe and cloud data consistent through the transition. A 2-3 year horizon is realistic for a regulated bank.

- **A) is wrong** -- a weekend big-bang cutover of a core banking mainframe gives no room to discover undocumented dependencies, and failure is a banking outage.
- **C) is wrong** -- rewriting all the COBOL before migrating delivers nothing until it is finished and carries years of risk.
- **D) is wrong** -- Database Migration Service does not cover mainframe data stores. VSAM, IMS and DB2 for z/OS need specialised tooling and data-model transformation.

**Exam tip:** regulated plus mission-critical plus legacy means the exam wants the slowest, most reversible option. Read-only first, then dual writes, then writes. Any answer promising a single cutover weekend for a core system is there to be eliminated.

Docs: https://cloud.google.com/architecture/migration-to-gcp-getting-started and https://cloud.google.com/architecture/microservices-architecture-refactoring-monoliths
</details>

---

### Q51. A company has 50 TB of data in an on-premises Hadoop cluster (HDFS) with Spark jobs running daily. They want to migrate to GCP with minimal changes to their Spark code. What is the recommended migration approach?

A) Migrate HDFS data to Cloud Storage, run Spark jobs on Dataproc with Cloud Storage as the data layer
B) Migrate HDFS data to BigQuery, rewrite Spark jobs as SQL queries
C) Set up a permanent GKE cluster with Apache Spark operator for all Spark workloads
D) Replicate the entire Hadoop cluster on Compute Engine instances with HDFS

<details>
<summary>Answer</summary>

**Correct: A)**

Dataproc runs the same Spark APIs the team already uses, so moving HDFS data to Cloud Storage and changing hdfs:// paths to gs:// is close to the whole code change. Keeping data in Cloud Storage also separates storage from compute, which is what lets clusters be ephemeral and billed only while jobs run.

- **B) is wrong** -- rewriting Spark jobs as SQL is a major refactor, and UDFs, ML pipelines and graph work often have no SQL equivalent.
- **C) is wrong** -- Spark on GKE means operator setup, custom images and cluster operations, which is more work than managed Dataproc and not "minimal changes".
- **D) is wrong** -- rebuilding the Hadoop cluster on VMs keeps every operational burden and gains almost nothing.

**Exam tip:** the Hadoop migration answer is nearly always "data to Cloud Storage, compute on ephemeral Dataproc". The Cloud Storage connector is what makes it a path change rather than a rewrite. Watch for options that quietly demand a rewrite while claiming to be a migration.

Docs: https://cloud.google.com/dataproc/docs/concepts/connectors/cloud-storage and https://cloud.google.com/dataproc/docs/concepts/overview
</details>

---

### Q52. A company is migrating a 3-tier web application (web server, application server, MySQL database) from on-premises to GCP. They want to migrate quickly (within 2 weeks) with minimal application changes, then optimize later. Which migration strategy and target services should they use?

A) Rehost: Web and app servers to Compute Engine VMs using M2VM, MySQL to Cloud SQL using Database Migration Service (DMS) with continuous replication
B) Refactor: Web tier to Cloud Run, app tier to GKE, MySQL to Cloud Spanner
C) Replatform: Everything to GKE containers with MySQL running as a StatefulSet
D) Rehost: Export VMs as images and import to Compute Engine, manual MySQL dump and restore

<details>
<summary>Answer</summary>

**Correct: A)**

This is a straight rehost that fits two weeks. Migrate to Virtual Machines replicates the web and application VMs continuously and cuts over in minutes; Database Migration Service replicates MySQL into Cloud SQL continuously so you can validate before switching. No application changes are required.

- **B) is wrong** -- Cloud Run, GKE and Spanner mean containerisation and a schema redesign, which is not a two-week piece of work.
- **C) is wrong** -- containerising a traditional three-tier application means new Dockerfiles, manifests and testing, and MySQL as a StatefulSet adds database operations you were trying to hand to a managed service.
- **D) is wrong** -- manual export and import plus a dump and restore needs a long downtime window and has no continuous replication.

**Exam tip:** a tight deadline plus "optimise later" is the exam saying rehost. The tool pair to remember is Migrate to Virtual Machines for the VMs and Database Migration Service for the database, both with continuous replication so the cutover is short.

Docs: https://cloud.google.com/database-migration/docs and https://cloud.google.com/migrate/virtual-machines/docs
</details>

---

### Q53. During migration assessment, Migration Center identifies that 30 of 500 workloads run on custom hardware (FPGA accelerators, specialized NICs). These workloads cannot run on standard cloud infrastructure. Which of the 6 R's should be applied to these workloads?

A) Rehost them on Compute Engine with custom machine types
B) Retain them on-premises and establish hybrid connectivity
C) Retire them since they cannot be migrated
D) Replace them with SaaS alternatives

<details>
<summary>Answer</summary>

**Correct: B)**

Retain (sometimes called revisit) means leaving the workload on-premises and connecting it over Interconnect or VPN. That is the right call for workloads with hard dependencies on hardware that has no cloud equivalent, and they can be reconsidered later.

- **A) is wrong** -- custom machine types vary CPU and memory ratios. They do not provide FPGAs or specialised NICs, which the stem says are required.
- **C) is wrong** -- Retire means decommissioning something no longer needed. These workloads are still in use; they simply cannot move.
- **D) is wrong** -- Replace means substituting a SaaS product, which is unlikely to exist for a bespoke FPGA-accelerated workload.

**Exam tip:** learn the 6 R's by their trigger words. Rehost is lift and shift; replatform is minor changes; refactor is rewrite; repurchase or replace is move to SaaS; retire is switch off; retain is leave it where it is. "Cannot run on cloud infrastructure" always maps to retain, never to retire.

Docs: https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---

### Q54. A company is migrating a PostgreSQL database from AWS RDS to Google Cloud. The database is 500 GB and serves a production application that cannot tolerate more than 5 minutes of downtime. Which migration approach should they use?

A) pg_dump the database, transfer to Cloud Storage, restore to Cloud SQL for PostgreSQL
B) Use Database Migration Service (DMS) with continuous replication from AWS RDS to Cloud SQL, then perform a cutover
C) Set up logical replication directly from AWS RDS to Cloud SQL, then cutover
D) Use Transfer Appliance to move the database files physically

<details>
<summary>Answer</summary>

**Correct: B)**

Database Migration Service migrates PostgreSQL from AWS RDS to Cloud SQL with continuous replication, so the target stays in sync while the source keeps serving. The cutover is then seconds to minutes, inside the 5-minute tolerance.

- **A) is wrong** -- dump, transfer and restore of 500 GB runs for hours, and the whole of it lands inside the downtime window.
- **C) is wrong** -- hand-built logical replication works in principle but means managing publications, subscriptions, schema drift and sequences yourself. DMS automates it and is the recommended path.
- **D) is wrong** -- Transfer Appliance is offline bulk data movement measured in weeks, wildly disproportionate to 500 GB.

**Exam tip:** downtime tolerance decides the database migration tool. Minutes or less means continuous replication, which means DMS. Hours of tolerance allows dump and restore. Anything involving physical shipping is for hundreds of terabytes and up, never for a live database cutover.

Docs: https://cloud.google.com/database-migration/docs/postgres and https://cloud.google.com/database-migration/docs
</details>

---

### Q55. An enterprise is evaluating the 6 R's for their application portfolio. They have: (1) a legacy CRM built in-house, (2) on-premises email servers, (3) a custom analytics platform with tight hardware dependencies, and (4) an unused legacy HR system. Match each to the most appropriate migration strategy.

A) (1) Rehost, (2) Replace, (3) Retain, (4) Retire
B) (1) Refactor, (2) Rehost, (3) Replace, (4) Retain
C) (1) Replatform, (2) Retire, (3) Rehost, (4) Replace
D) (1) Replace, (2) Replace, (3) Retain, (4) Retire

<details>
<summary>Answer</summary>

**Correct: A)**

The in-house CRM is rehosted first to get out of the data centre quickly, with refactoring deferred. Email servers are replaced by SaaS, which is what Google Workspace is for. The hardware-dependent analytics platform is retained on-premises behind hybrid connectivity. The unused HR system is retired.

- **B) is wrong** -- rehosting email servers ignores the SaaS option, and refactoring the CRM first is the slowest possible starting move.
- **C) is wrong** -- retiring email means switching off email, and rehosting a hardware-dependent workload is exactly what the stem says cannot be done.
- **D) is wrong** -- a bespoke in-house CRM has no obvious SaaS equivalent, so replace is a poor fit where rehost is safe.

**Exam tip:** two mappings resolve most 6 R's matching questions. Commodity function with a mature SaaS market (email, HR, CRM suites) means replace. Unused means retire. Once those two are placed, the remaining options usually collapse to one.

Docs: https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---

### Q56. A company is performing a large-scale migration and needs to validate that migrated workloads perform identically on GCP compared to on-premises. They have 200 workloads to validate over 3 months. What testing strategy should they implement?

A) Run production traffic to both on-premises and GCP simultaneously (traffic mirroring) and compare responses
B) Rely on Google's SLA guarantees and skip performance testing
C) Test each workload manually with a QA team over the 3 months
D) Implement automated performance benchmarking: capture baseline metrics on-premises, run identical benchmarks on GCP after migration, and use automated comparison with acceptance criteria

<details>
<summary>Answer</summary>

**Correct: D)**

Automated benchmarking scales to 200 workloads: capture baselines on-premises, run the identical suite after migration, and compare against acceptance criteria for latency, throughput and error rate so only the failures need human attention.

- **A) is wrong** -- mirroring production traffic to both environments doubles the running cost, is complex to set up 200 times, and is unsafe for anything with side effects such as writes or payments.
- **B) is wrong** -- an SLA covers infrastructure availability, not application performance. Identical VM specs can still perform differently once tuning and configuration change.
- **C) is wrong** -- manual QA across 200 workloads is slow and inconsistent between testers.

**Exam tip:** scale in the stem is a filter. Anything manual or per-workload becomes wrong once the count reaches the hundreds. Also watch for SLAs being offered as a substitute for testing, which the exam treats as a category error.

Docs: https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---

## 1.5 Future Improvements (Q57-Q65)

---

### Q57. A company has successfully migrated their e-commerce platform to GCP using lift-and-shift. The application runs on Compute Engine VMs with a Cloud SQL database. As a next step, the architect wants to modernize the platform for better scalability and cost efficiency. Which modernization path should they follow?

A) Immediately refactor everything to serverless (Cloud Run, Cloud Functions, Firestore)
B) Containerize the application and deploy on GKE Autopilot, then incrementally extract microservices from the monolith
C) Keep the current architecture and focus only on reserved capacity (CUDs) for cost optimization
D) Migrate the entire application to App Engine Standard for automatic scaling

<details>
<summary>Answer</summary>

**Correct: B)**

Containerising and moving to GKE Autopilot is the natural step after a lift and shift: Google runs the nodes, and the platform then supports pulling microservices out of the monolith one at a time. Value arrives incrementally instead of at the end of a rewrite.

- **A) is wrong** -- a full serverless refactor delivers nothing until it is complete and is high risk for a live e-commerce platform.
- **C) is wrong** -- commitments reduce the bill but leave the scalability, deployment velocity and resilience problems untouched.
- **D) is wrong** -- App Engine Standard constrains runtimes, request duration and connectivity, which an existing monolith is unlikely to fit, and it is not an incremental path.

**Exam tip:** modernisation questions reward the smallest next step that unlocks the following one. Containerise before decomposing, decompose before going serverless. An option that jumps straight to the end state is usually the distractor, and one that only optimises cost is usually incomplete.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview and https://cloud.google.com/architecture/microservices-architecture-refactoring-monoliths
</details>

---

### Q58. A data engineering team is running batch ETL jobs on Dataproc clusters that they manually size and manage. They want to modernize their data pipeline for better efficiency and reduced operational overhead. What improvements should they implement? (Choose TWO)

A) Migrate batch ETL to Dataflow for serverless processing with autoscaling
B) Implement Dataproc serverless for Spark workloads that still require Spark
C) Increase the fixed Dataproc cluster size to handle peak loads without autoscaling
D) Replace all ETL with manual SQL scripts run in Cloud SQL
E) Move all data processing to Cloud Functions

<details>
<summary>Answer</summary>

**Correct: A) and B)**

Dataflow removes cluster management entirely for pipelines that can be expressed in Beam, and Dataproc Serverless runs Spark workloads without a cluster for the jobs that must stay Spark. Together they cover the estate while eliminating the sizing and patching work.

- **C) is wrong** -- growing a fixed cluster to cover peak is the opposite of modernisation: more cost, same operational burden.
- **D) is wrong** -- hand-run SQL scripts in Cloud SQL removes automation and cannot handle Dataproc-scale volumes.
- **E) is wrong** -- Cloud Run functions have execution time and memory limits and are not a large-scale data processing engine.

**Exam tip:** when the stem says "must remain Spark", the answer is Dataproc Serverless, not Dataflow. Dataflow is the answer when the pipeline can be rewritten in Beam or is new. Note also the naming: Cloud Functions is now Cloud Run functions, so both names appear in current material.

Docs: https://cloud.google.com/dataproc-serverless/docs and https://cloud.google.com/dataflow/docs
</details>

---

### Q59. A company currently uses custom-built ML pipelines on Compute Engine for model training and serving. They want to modernize their ML platform to increase productivity and reduce time-to-production for new models. What cloud-native approach should they adopt?

A) Continue using Compute Engine but add GPUs for faster training
B) Adopt Vertex AI for end-to-end ML lifecycle: Feature Store for feature management, Vertex AI Training for model training, Vertex AI Pipelines for orchestration, and Vertex AI Endpoints for serving
C) Migrate all ML workloads to Dataproc with MLlib
D) Use only pre-trained models from Vertex AI Model Garden and eliminate custom training

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI covers the whole lifecycle in one managed platform: Feature Store for shared feature definitions, Training for managed compute and tuning, Pipelines for orchestration and lineage, Endpoints for serving. That is what shortens time to production, not any single faster step. (The platform is now marketed as Gemini Enterprise Agent Platform; the exam guide still says Vertex AI.)

- **A) is wrong** -- faster training addresses one stage. Orchestration, feature management, versioning and monitoring are untouched.
- **C) is wrong** -- Dataproc MLlib is narrower than modern frameworks and offers none of the serving, monitoring or feature-store layers.
- **D) is wrong** -- dropping custom training removes the ability to model the company's own data. Pre-trained models complement custom ones rather than replacing them.

**Exam tip:** MLOps questions are usually won by the option naming the most complete lifecycle, because the pain in the stem is process pain rather than compute pain. Learn the Vertex AI component names and what stage each one owns.

Docs: https://cloud.google.com/vertex-ai/docs/start/introduction-unified-platform and https://cloud.google.com/vertex-ai/docs/pipelines/introduction
</details>

---

### Q60. An organization is running a monolithic application on GKE. They want to adopt a cloud-native architecture that improves deployment velocity, fault isolation, and independent scaling of components. They also want to implement GitOps practices. Which combination of improvements should they prioritize?

A) Keep the monolith on GKE and add more replicas for scaling
B) Decompose into microservices on GKE, implement Config Sync for GitOps, and use Cloud Service Mesh for inter-service communication
C) Move the monolith from GKE to Cloud Run for simpler deployments
D) Decompose into microservices and deploy each on separate Compute Engine instances

<details>
<summary>Answer</summary>

**Correct: B)**

Decomposition gives fault isolation and independent scaling. Config Sync (the GitOps engine that shipped as Anthos Config Management) reconciles cluster configuration from Git, and Cloud Service Mesh (formerly Anthos Service Mesh) handles inter-service traffic, security and observability. That covers all three stated goals plus the GitOps requirement.

- **A) is wrong** -- more replicas of a monolith improves neither deployment velocity, nor fault isolation, nor per-component scaling, since everything still ships and scales as one unit.
- **C) is wrong** -- moving the monolith to Cloud Run changes the runtime, not the architecture, and brings request-duration limits with it.
- **D) is wrong** -- microservices on individual VMs gives up scheduling, self-healing and rolling updates, which is most of why the cluster is there.

**Exam tip:** "GitOps" in a stem points at Config Sync, and "inter-service mTLS, traffic management or tracing" points at Cloud Service Mesh. Both were sold under Anthos names until recently, so recognise the old and new names as the same products.

Docs: https://cloud.google.com/kubernetes-engine/enterprise/config-sync/docs/overview and https://cloud.google.com/service-mesh/docs/overview
</details>

---

### Q61. A company has been using GCP for 2 years and wants to optimize their cloud spending. Their monthly bill is $500K with significant waste identified. Which approach provides the most comprehensive cost optimization?

A) Purchase 3-year CUDs for all current resources
B) Implement a FinOps practice: use Active Assist recommendations, right-size VMs using recommender, review CUD/SUD coverage, set up billing exports to BigQuery for custom analysis, and establish cost accountability per team
C) Delete all non-production environments
D) Move all workloads to the cheapest region

<details>
<summary>Answer</summary>

**Correct: B)**

A FinOps practice attacks the whole problem: Active Assist recommenders surface idle and oversized resources, commitment coverage is reviewed against actual usage, billing export to BigQuery supports custom analysis and anomaly detection, and per-team accountability changes the behaviour that produced the waste.

- **A) is wrong** -- committing to the current shape locks in whatever oversizing already exists. Right-size first, then commit.
- **C) is wrong** -- deleting non-production removes development and test capability. Scheduling it off outside working hours gets most of the saving without the damage.
- **D) is wrong** -- the cheapest region may break latency targets, data residency rules or availability requirements. Region choice is not a cost-only decision.

**Exam tip:** the sequence the exam rewards is measure, then right-size, then commit, then govern. Committing before right-sizing is the classic wrong answer, and so is any option that saves money by removing visibility or capability rather than waste.

Docs: https://cloud.google.com/recommender/docs and https://cloud.google.com/billing/docs/how-to/export-data-bigquery
</details>

---

### Q62. A company's application currently uses synchronous REST APIs for all inter-service communication. During peak loads, cascading failures occur when downstream services become slow or unavailable. What architectural improvement should they implement to increase resilience?

A) Increase the timeout values on all API calls to wait longer for responses
B) Implement asynchronous event-driven communication using Pub/Sub for non-real-time operations, and add circuit breakers and retries with exponential backoff for synchronous calls that must remain
C) Add more replicas of all services to handle the load
D) Move all services to a single monolithic application to eliminate network calls

<details>
<summary>Answer</summary>

**Correct: B)**

Moving non-real-time work onto Pub/Sub decouples the caller from the callee, so a slow downstream service no longer blocks upstream request threads. For the calls that must stay synchronous, circuit breakers fail fast instead of queueing, and exponential backoff with jitter prevents a retry storm from finishing off a recovering service.

- **A) is wrong** -- longer timeouts make the cascade worse. Upstream services hold connections and threads for longer, so they exhaust their own resources sooner.
- **C) is wrong** -- more replicas add more clients hammering the same slow dependency. Capacity is not the failure mode here.
- **D) is wrong** -- collapsing to a monolith discards independent deployment and scaling, and a slow component inside a process still blocks callers.

**Exam tip:** cascading failure questions are about isolation, not capacity. The winning answer combines asynchrony where possible with circuit breaking, timeouts, bulkheads and backoff where it is not. Any option that raises timeouts or adds instances is the trap.

Docs: https://cloud.google.com/architecture/framework/reliability and https://cloud.google.com/pubsub/docs/overview
</details>

---

### Q63. A company wants to leverage Google's latest AI capabilities to improve their customer support. They have 5 years of support ticket data and want to build an AI-powered support agent that can understand context, retrieve relevant knowledge articles, and generate helpful responses. What is the recommended approach?

A) Fine-tune a custom LLM from scratch using Vertex AI Training on their support data
B) Use Vertex AI Agent Builder to create an AI agent grounded in their knowledge base, powered by Gemini models
C) Deploy a pre-trained BERT model on Vertex AI Endpoints for ticket classification only
D) Use BigQuery ML to train a text classification model on support tickets

<details>
<summary>Answer</summary>

**Correct: B)**

Agent Builder creates an agent grounded in the company's own knowledge base using managed retrieval-augmented generation over Gemini models, so answers cite the source documents and no model training is needed. Five years of tickets become a data store, not a training set.

- **A) is wrong** -- training a large language model from scratch needs compute, data and expertise on a completely different scale, and the base model already has the language ability.
- **C) is wrong** -- BERT is a classification and embedding model. It can label a ticket but cannot generate a helpful answer.
- **D) is wrong** -- BigQuery ML can classify text but cannot hold a grounded conversation with retrieval and citations.

**Exam tip:** grounding in company data means RAG, and RAG on Google Cloud means Agent Builder with a data store. Fine-tuning is for style and format; RAG is for facts. If the stem asks for citations or up-to-date knowledge, the answer is retrieval, never training.

Docs: https://cloud.google.com/generative-ai-app-builder/docs/introduction and https://cloud.google.com/vertex-ai/generative-ai/docs/agent-builder/overview
</details>

---

### Q64. A company has deployed their application in a single GCP region. They want to evolve their architecture to handle regional outages without over-engineering. Their application has a web tier, an API tier, and a Cloud SQL database. Current SLO is 99.9%. They want to target 99.95%. What is the most appropriate next step?

A) Deploy a full active-active setup across three regions with Cloud Spanner
B) Add a Cloud SQL cross-region read replica, deploy the web and API tiers in a second region with a global Application Load Balancer, and implement automated failover
C) Add Cloud CDN in front of the web tier and increase instance sizes
D) Switch from Cloud SQL to Bigtable for multi-region support

<details>
<summary>Answer</summary>

**Correct: B)**

Moving from 99.9% to 99.95% requires surviving a regional event, but not active-active. A cross-region read replica that can be promoted, the web and API tiers running in a second region, a global Application Load Balancer in front, and automated failover gets there at warm-standby cost.

- **A) is wrong** -- three-region active-active on Spanner is aimed at five nines. It is a large cost and complexity increase for a 0.05 percentage point target.
- **C) is wrong** -- CDN and larger instances improve performance and absorb load, but neither survives a regional outage, which is the risk at this availability level.
- **D) is wrong** -- Bigtable is a wide-column NoSQL store, not a drop-in replacement for a relational database; this would be a data-model rewrite.

**Exam tip:** treat the availability target as a budget. Each extra nine costs an order of magnitude more, so pick the smallest architectural change that clears the number in the stem. Explicit target numbers exist to rule out the most redundant option, not to justify it.

Docs: https://cloud.google.com/architecture/dr-scenarios-planning-guide and https://cloud.google.com/sql/docs/postgres/replication/cross-region-replicas
</details>

---

### Q65. An organization is reviewing their GCP architecture against the Well-Architected Framework (WAF) and identifies gaps in observability, security posture, and cost management. They want to implement improvements systematically. Which approach aligns with WAF best practices?

A) Address all gaps simultaneously with a large cross-team project
B) Prioritize by risk: implement Security Command Center for security posture, Cloud Monitoring with SLOs for observability, and billing alerts with Active Assist for cost management, iterating on each area
C) Focus exclusively on cost management since it has the most direct business impact
D) Hire a third-party consultant to fix everything without involving internal teams

<details>
<summary>Answer</summary>

**Correct: B)**

The framework asks for iterative improvement prioritised by risk. Security Command Center covers posture and vulnerability findings, Cloud Monitoring with SLOs makes reliability measurable, and budget alerts with Active Assist recommendations give cost visibility and specific actions. Each area then improves on its own cadence.

- **A) is wrong** -- one large simultaneous programme across all gaps overloads teams and delays every outcome.
- **C) is wrong** -- concentrating on one pillar leaves the others exposed, and a security gap can cost far more than any billing optimisation saves.
- **D) is wrong** -- outsourcing the whole thing builds no internal capability, which the framework treats as part of the outcome rather than an optional extra.

**Exam tip:** Architecture Framework questions reward iterative, risk-prioritised improvement and a named tool per pillar. Memorise the mapping: Security Command Center for security posture, SLOs in Cloud Monitoring for reliability, Active Assist and budgets for cost.

Docs: https://cloud.google.com/architecture/framework and https://cloud.google.com/security-command-center/docs
</details>

---