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

CUDs for baseline capacity provide predictable, discounted pricing for the steady-state workload (up to 57% discount for 1-year, 70% for 3-year on compute). On-demand instances handle the short burst periods without long-term commitment. Option A is wrong because sizing CUDs for peak means paying for unused capacity ~350 days/year -- massively wasteful. Option C is wrong because Spot VMs can be preempted at any time, which is unacceptable for customer-facing e-commerce traffic. Option D is wrong because SUDs only provide up to ~30% discount and do not give the predictable budgeting the CFO requires (SUDs are automatic but less cost-effective than CUDs for known baseline workloads).
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

Pub/Sub provides durable, at-least-once message ingestion at any scale. Dataflow (Apache Beam) in streaming mode processes events in real-time with exactly-once semantics. BigQuery supports streaming inserts for near-real-time analytics, and Looker provides visualization. This is fully managed with minimal ops overhead. Option B is wrong because Cloud SQL is a relational OLTP database that cannot handle 500K events/sec analytics queries and is not designed for this scale of writes. Option C is wrong because running Kafka on GKE and Dataproc requires significant operational management (cluster sizing, upgrades, monitoring), violating the minimal overhead requirement. Option D is wrong because batch processing every 5 seconds is not true streaming -- micro-batching adds latency and complexity, and Cloud Storage is not designed as a streaming ingestion layer.
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

Cloud Spanner in a multi-region EU configuration (e.g., eur5) provides 99.999% SLA, strong consistency, and keeps all data within EU regions -- exceeding the 99.99% requirement while meeting data residency constraints. Cloud Audit Logs exported to Cloud Storage with a 7-year retention policy and bucket lock ensures immutable, cost-effective long-term retention that satisfies regulatory audit requirements. Option B is wrong because Cloud SQL regional HA provides only 99.95% SLA, falling short of the 99.99% requirement. Option C is wrong because BigQuery is an analytics warehouse, not a transactional database for core banking with complex transactions. Option E is wrong because BigQuery table expiration deletes data after the period -- you need retention, not expiration. Also, storing 7 years of audit logs in BigQuery is significantly more expensive than Cloud Storage.
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

Premium Tier networking uses Google's global backbone for lowest latency routing. Cloud CDN caches the frequently accessed content (the 20% that drives 80% of views) at edge locations globally, dramatically reducing latency. Regional Storage buckets are cheaper than multi-regional, and since CDN handles caching, multi-regional redundancy for storage is unnecessary. Option A is wrong because deploying full stacks in every region is extremely expensive and operationally complex -- especially replicating 500 TB everywhere when only 20% is frequently accessed. Option B is wrong because multi-regional buckets add cost without benefit when CDN is already caching popular content at the edge; it also does not specify the network tier. Option D is wrong because managing custom Nginx caching proxies in each region creates significant operational burden compared to the fully managed Cloud CDN, and lacks the global edge network coverage.
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

Dedicated Interconnect provides 10 Gbps or 100 Gbps connections between on-premises and GCP with low latency, high bandwidth, and private connectivity. Private Google Access ensures traffic stays on Google's network. This meets all requirements: secure (private, not over public internet), low-latency, and supports 10 Gbps. Option A is wrong because Cloud VPN maxes out at 3 Gbps per tunnel (can aggregate up to 8 tunnels for HA VPN), adding complexity to reach 10 Gbps, and traffic traverses the public internet with higher latency. Option C is wrong because Partner Interconnect maxes at 50 Gbps total but individual attachments are typically 50 Mbps to 10 Gbps depending on the partner -- and since the company needs 10 Gbps consistently, Dedicated Interconnect is the direct, simpler solution without a third-party dependency. Option D is wrong because Transfer Appliance is for one-time bulk data migration, not real-time integration, and nightly replication does not meet the real-time requirement.
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

Choosing multi-region Spanner for lowest global read latency is a performance optimization, not a compliance requirement. MedTrack is a startup -- there is no stated need for global distribution. HIPAA compliance focuses on data protection, access controls, and audit trails, not global latency optimization. Option A is relevant because CMEK gives the organization control over encryption keys, which is often required for HIPAA compliance. Option B is relevant because VPC Service Controls prevent data exfiltration and restrict which networks can access sensitive services. Option D is relevant because Data Access audit logs track who accessed what data, which is essential for HIPAA audit trail requirements.
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

A complete TCO analysis should include the primary ongoing costs (BigQuery pricing model -- either on-demand or editions), and the savings from eliminating on-premises infrastructure costs (hardware, data center, cooling, staff). Both BigQuery pricing models (A and E) should be evaluated to determine the most cost-effective approach for TechRetail's workload pattern. Option B is wrong because Interconnect is a one-time migration cost, not a recurring TCO factor over 3 years -- it would appear in a migration cost analysis but is not a primary driver of the 3-year TCO comparison. Option D is wrong because BigQuery uses standard SQL, so there is no new programming language to learn -- Teradata SQL migration to BigQuery SQL requires query adjustments, not full language retraining.
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

This architecture uses the right tool for each job. Bigtable excels at high-throughput, low-latency reads by key (vehicle ID) -- perfect for a real-time dashboard. BigQuery is ideal for historical analytics and route optimization queries over large datasets. Dataflow handles the fan-out with exactly-once processing. At 25,000 events/second (50K vehicles x 0.5 Hz), this pipeline scales well. Option A is wrong because BigQuery streaming inserts work but BigQuery is not optimized for point-lookup real-time serving (it is an analytics engine with higher query latency). Option C is wrong because Firestore has a limit of 10,000 writes/sec per database (can be increased but is not designed for this sustained write throughput), and Cloud Functions would struggle at this scale. Option D is wrong because Cloud SQL cannot handle 25,000 writes/sec and is not suitable for large-scale analytics.
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

Organizing folders by environment allows applying stricter org policies (e.g., disable external IPs, enforce CMEK) to the entire prod folder, which automatically inherits to all production projects. Projects per product line within each environment folder enable clear resource isolation. Labels on projects enable cost tracking per product line via billing exports. Option A is wrong because a single project per environment provides no isolation between product lines and makes cost attribution difficult. Option B is wrong because putting environment folders under product line folders means you cannot apply a single org policy to all production environments -- you would need to apply it separately to the prod sub-folder under each product line. Option C is wrong because shared projects across product lines break the principle of least privilege and make cost attribution imprecise.
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

Dataproc with Spot VMs provides 60-91% discount on worker nodes, and since the workload is highly parallelizable and can handle worker preemption (Spark/Hadoop handle task redistribution), this is the most cost-effective option. Cloud Composer orchestrates the daily pipeline, and results in Cloud Storage are directly accessible for ML training. Option A is wrong because a persistent GKE GPU cluster running 24/7 is extremely expensive when processing only needs to happen once daily for a few hours. Option B is wrong because Dataflow is excellent for streaming and well-suited for batch, but its autoscaling workers use standard pricing -- no Spot VM option for Dataflow workers, making it more expensive than Dataproc with Spot VMs. Option D is wrong because permanently running VMs waste money during the ~20 hours/day when no processing occurs.
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

Vertex AI Endpoints provide managed model serving with autoscaling, low-latency prediction, and built-in traffic splitting for A/B testing between model versions. This is purpose-built for production ML serving at scale. Option B is wrong because Cloud Functions have cold start latency that can exceed 100ms, and they are not optimized for ML model serving (model loading on each cold start is slow). Option C is wrong because a single instance is a single point of failure and cannot scale to handle 10M daily users during peak traffic. Option D is wrong because BigQuery ML is designed for batch predictions and analytics, not real-time sub-100ms serving -- query overhead alone typically exceeds 100ms.
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

A comprehensive success framework should align with the Google Cloud Architecture Framework (WAF) pillars and business objectives. For a government agency, this means tracking cost optimization (TCO reduction vs. on-premises), operational excellence (deployment frequency, incident response time), security/compliance (audit findings, compliance certification maintenance), and citizen experience (page load times, availability, user satisfaction). Option A is wrong because focusing solely on cost ignores critical dimensions like security compliance and user experience, which are paramount for a government agency. Option C is wrong because Google's SLAs are minimum guarantees, not success metrics -- the agency needs custom KPIs aligned with their mission. Option D is wrong because migration velocity is an output metric, not an outcome metric -- migrating fast means nothing if the services are unreliable or insecure.
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

Cloud Spanner multi-region instances (e.g., nam6, eur5) replicate data synchronously across regions with RPO = 0 by design. A global external Application Load Balancer with health checks automatically routes traffic to healthy backends, achieving RTO well under 30 seconds. This is the only option that achieves both RPO = 0 and RTO < 30s. Option B is wrong because Cloud Spanner does not have "read replicas" in the traditional sense -- it uses multi-region configurations for this purpose. A single-region instance with manual failover cannot achieve RPO = 0 or RTO < 30s. Option C is wrong because application-level replication between Cloud SQL instances introduces replication lag (RPO > 0) and DNS failover typically takes minutes (RTO >> 30s). Option D is wrong because deploying the application in a single region means a regional outage takes down the application even though Spanner data is available -- Cloud DNS failover also takes minutes due to TTL propagation.
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

For 100x traffic spikes with little warning, you need aggressive scaling. HPA on custom metrics (like queue depth) reacts faster than CPU-based scaling since it detects demand before CPU saturates. Cluster Autoscaler with NAP automatically provisions the right node types and sizes. Provisioning profiles allow pre-configured surge capacity. This combination provides the fastest scaling response. Option A is wrong because CPU-based HPA is reactive (CPU must rise before scaling triggers) and can be too slow for sudden 100x spikes -- the application may become unresponsive before scaling catches up. Option B is wrong because VPA adjusts resource requests per pod but does not scale the number of pods or nodes -- it is for right-sizing, not handling traffic spikes. Option D is wrong because sizing a fixed node pool for 500K users wastes significant resources during off-peak (5K users), directly violating the "without over-provisioning" requirement.
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

Cloud SQL HA with automated backups and point-in-time recovery meets RPO of 1 hour (backups + transaction logs allow recovery to any point) and RTO of 4 hours (restore from backup takes minutes to hours depending on size, well within 4 hours). This is the most cost-effective option for these relaxed RPO/RTO requirements. Option B is wrong because cross-region read replicas provide faster failover but cost significantly more (running a full replica 24/7), which is unnecessary given the relaxed 4-hour RTO. Option C is wrong because Cloud Spanner multi-region is designed for RPO = 0 and near-zero RTO -- massively over-engineered and expensive for RPO of 1 hour / RTO of 4 hours. Option D is wrong because daily exports give you RPO of up to 24 hours, which exceeds the 1-hour RPO requirement.
</details>

---

### Q16. A company is deploying a microservices architecture on GKE with 30 services. They need to implement mutual TLS between services, circuit breaking, traffic management (canary deployments), and distributed tracing -- all without modifying application code. What should they implement?

A) Anthos Service Mesh (managed Istio) with sidecar proxies
B) Custom Envoy proxy configurations deployed as DaemonSets
C) Network Policies with GKE Dataplane V2 and manual mTLS certificate management
D) Cloud Load Balancing with NEGs for each service

<details>
<summary>Answer</summary>

**Correct: A)**

Anthos Service Mesh (ASM), based on Istio, provides all required features (mTLS, circuit breaking, traffic management, distributed tracing) via sidecar proxy injection -- without any application code changes. It is fully managed on GCP and integrates with Cloud Trace and Cloud Monitoring. Option B is wrong because manually configuring Envoy proxies as DaemonSets requires significant operational effort and custom configuration for each feature -- ASM automates all of this. Option C is wrong because Network Policies only handle L3/L4 network segmentation, not L7 features like circuit breaking, canary deployments, or distributed tracing. Manual mTLS certificate management is error-prone and does not meet the "without modifying application code" requirement. Option D is wrong because Cloud Load Balancing handles north-south traffic (external to cluster), not east-west service-to-service communication within the cluster. It also does not provide mTLS, circuit breaking, or distributed tracing.
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

The WAF Reliability pillar emphasizes defining SLOs (Service Level Objectives) based on business needs before choosing an architecture. Multi-region active-active may be unnecessary if the business SLO can be met with regional HA. The architecture should be driven by measurable targets, not assumptions. Option A is wrong because WAF explicitly warns against over-engineering -- multi-region active-active adds complexity and cost that may not be justified by business requirements. Option C is wrong because it is reactive rather than proactive -- WAF advocates designing for reliability upfront based on defined targets, not waiting for failures. Option D is wrong because this question is about reliability architecture, not performance optimization. While latency may influence region selection, the reliability decision should be driven by SLOs, not latency alone.
</details>

---

### Q18. A video conferencing platform must handle 10,000 concurrent WebSocket connections per instance with low latency. The connections are long-lived (average 45 minutes). The system must scale horizontally and maintain session affinity. Which compute and load balancing architecture should you use?

A) GKE pods with a global external Application Load Balancer using session affinity (cookie-based)
B) Compute Engine instances with a regional external TCP/UDP Network Load Balancer with connection tracking
C) Cloud Run services with a global external Application Load Balancer
D) GKE pods with a global external proxy Network Load Balancer with session affinity

<details>
<summary>Answer</summary>

**Correct: D)**

WebSocket connections are long-lived TCP connections that need a proxy-based load balancer supporting WebSocket protocol upgrade and session affinity. The global external proxy Network Load Balancer (Envoy-based) supports TCP connections with session affinity and distributes traffic globally. GKE provides horizontal scaling of pods. Option A is wrong because the external Application Load Balancer works for WebSocket but is optimized for HTTP(S) traffic. While it can handle WebSocket via upgrade, cookie-based affinity requires the initial HTTP request to set a cookie, which adds complexity for WebSocket-first connections. The proxy Network Load Balancer is more appropriate for TCP-level affinity. Option B is wrong because the regional external TCP/UDP Network Load Balancer is a passthrough (non-proxy) load balancer -- it supports connection tracking but is regional only, limiting global distribution. Option C is wrong because Cloud Run has a maximum request timeout of 60 minutes (adjustable) but is designed for request-response patterns, not long-lived WebSocket connections at scale. Cloud Run also does not guarantee session affinity.
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

Cloud Composer (managed Airflow) provides reliable orchestration with retry logic, scheduling, and alerting. Dataproc with standard primary workers ensures cluster stability, while preemptible (Spot) secondary workers reduce cost for the parallelizable, failure-tolerant tasks. Spark/Hadoop automatically retries tasks on preempted workers. Option B is wrong because a single VM is a single point of failure (does not meet 99.5% reliability), cannot leverage parallelism, and vertical scaling has limits. Option C is wrong because Dataflow batch jobs use standard workers at full price -- Dataproc with preemptible secondary workers is more cost-effective for known batch workloads that can tolerate task-level retries. Option D is wrong because Cloud Run jobs have a maximum execution time of 24 hours and are designed for containerized tasks, not complex multi-stage data processing pipelines that benefit from distributed computing frameworks like Spark.
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

For RPO < 5 minutes, Cloud SQL cross-region replication (async, typically seconds of lag) meets the requirement. Automated promotion via Cloud Functions (triggered by health check failures or monitoring alerts) achieves RTO < 10 minutes. The global Application Load Balancer with health checks automatically redirects traffic to the healthy region's application instances. Option A is wrong because manual promotion requires human intervention, which typically takes longer than 10 minutes (pager, assessment, execution), likely exceeding the RTO. Option B is wrong because a single-region deployment cannot survive a regional outage -- HA only protects against zonal failures. Option D is wrong because Cloud Spanner multi-region is over-engineered for RPO < 5 minutes. It provides RPO = 0 but at significantly higher cost. The question asks for the minimum architecture.
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

Vertex AI Endpoints with GPUs provide the managed infrastructure for model serving. A 15 GB model requires GPU memory for fast inference. Scheduled scaling (minimum replicas during business hours, scaled down outside) matches the predictable traffic pattern. Minimum 1 replica avoids cold-start latency during business hours. Option A is wrong because scale-to-zero means the first request after idle would face cold-start latency (loading a 15 GB model onto a GPU takes minutes), violating the sub-200ms latency requirement. Option C is wrong because managing GKE GPU nodes manually with cron jobs adds significant operational overhead compared to managed Vertex AI Endpoints. Option D is wrong because Cloud Functions cannot load a 15 GB model into memory (memory limit is 32 GB, but model loading time far exceeds function timeout limits), and each cold start would re-load the model.
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

Cloud Spanner provides strongly consistent reads globally with multi-region deployment, ensuring session data is always up-to-date regardless of which region serves the request. Deploying the auth service in each region ensures < 50ms response times by processing requests locally. Option A is wrong because Firestore multi-region provides strong consistency within a region but eventual consistency for cross-region reads -- a session created in one region might not be immediately visible in another. Option C is wrong because Memorystore for Redis is regional only -- there is no cross-region replication, so a user who authenticates in North America and is then routed to Europe would not have their session. Option D is wrong because caching auth tokens at CDN is a security risk (tokens could be served from cache after revocation), and a single-region auth service means users in distant regions exceed the 50ms requirement.
</details>

---

### Q23. An e-commerce platform expects a 20x traffic increase during a flash sale event in 2 weeks. Their current GKE-based architecture handles normal traffic well. The architect needs to ensure the system can handle the surge without service degradation. Which combination of actions should they take? (Choose TWO)

A) Perform load testing with Cloud Load Test to identify bottlenecks and determine required capacity
B) Pre-warm the Cluster Autoscaler by setting minimum node count to expected peak capacity for the sale duration
C) Switch all workloads to Spot VMs to reduce cost during the sale
D) Configure Horizontal Pod Autoscaler with aggressive scaling policies (rapid scale-up, slow scale-down)
E) Migrate from GKE to Cloud Run for automatic scaling

<details>
<summary>Answer</summary>

**Correct: A) and D)**

Load testing identifies bottlenecks, validates scaling limits, and determines required capacity before the event. Configuring HPA with aggressive scale-up policies ensures pods scale rapidly to meet sudden demand, while slow scale-down prevents premature scaling during traffic fluctuations. Option B is wrong because pre-setting minimum nodes to peak capacity wastes resources during non-peak portions of the sale and is expensive. You should rely on autoscaling with appropriate headroom instead. Option C is wrong because Spot VMs can be preempted at any time, which is unacceptable during a critical flash sale event when availability is paramount. Option E is wrong because migrating the entire platform 2 weeks before a major event introduces enormous risk. Architecture changes of this scale require thorough testing, not rushed migrations.
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

Warm standby provides a scaled-down (minimal) environment in the DR region with continuous database replication (RPO close to 0, well within 15 minutes). On failover, autoscaling brings the environment to full capacity within the 1-hour RTO. This balances cost (scaled-down resources) with recovery time. Option A is wrong because hot standby meets the requirements but at maximum cost -- the question explicitly asks to minimize standby costs, and running a full duplicate environment is the most expensive option. Option C is wrong because cold standby requires provisioning infrastructure from scratch, which for a multi-tier application typically takes 2-4+ hours, exceeding the 1-hour RTO. Option D is wrong because daily backups give RPO of up to 24 hours (far exceeding 15 minutes), and manual restore procedures are slow and error-prone, likely exceeding the 1-hour RTO.
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

Each data center needs 20 Gbps, which requires Dedicated Interconnect (2x10 Gbps LACP bundle or more). Cloud Routers in each GCP region handle BGP route exchange. Cloud VPN backup provides redundancy if an Interconnect circuit fails. Once connected to GCP, data centers can communicate via Google's backbone network through VPC peering or Shared VPC. Option A is wrong because HA VPN provides a maximum of 3 Gbps per tunnel, and even with 8 tunnels per gateway (max), achieving reliable 20 Gbps is impractical and complex. Option B is wrong because it lacks redundancy -- if the Interconnect connection to one data center fails, there is no backup connectivity. The question asks for an architecture design, and production architectures should include redundancy. Option C is wrong because a single Cloud Router is a single point of failure, and Partner Interconnect bandwidth depends on the partner's capabilities -- it may or may not support 20 Gbps per location.
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

The global external Application Load Balancer provides HTTPS termination, URL-based routing via URL maps (directing api/* to one backend service and static/* to another), automatic failover via health checks, and global anycast IP for low-latency routing. Network Endpoint Groups (NEGs) connect to GKE pods in both regions. Option A is wrong because regional Application Load Balancers do not provide automatic cross-region failover. Cloud DNS geographic routing can direct users to regions but does not handle failover as seamlessly (DNS TTL delays). Option C is wrong because the proxy Network Load Balancer operates at L4 (TCP/SSL) and does not support URL-based routing (L7 feature). Option D is wrong because passthrough Network Load Balancers do not perform HTTPS termination or URL-based routing -- they pass traffic through to backends without L7 inspection.
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

TPU v3 pod slices are optimized for TensorFlow training workloads and provide exceptional throughput for image classification (data-parallel training). They are more cost-effective than GPU clusters for large-scale TensorFlow training. Vertex AI Training manages the infrastructure lifecycle, so you pay only for training time. Option A is wrong because a single A100 GPU, while powerful, may not complete training on 2 million images within 8 hours depending on model complexity -- and it does not leverage parallelism. Option B is wrong because T4 GPUs are inference-optimized (lower cost, lower performance per unit). A cluster of T4s costs more and trains slower than TPU v3 for TensorFlow workloads. Option D is wrong because Dataproc is designed for Spark/Hadoop big data processing, not deep learning training. While Dataproc supports GPUs, it lacks the ML training optimizations of Vertex AI.
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

Bigtable handles the write throughput (100K messages/sec = 100 MB/sec) and provides single-digit millisecond reads by row key (device ID + timestamp). Its column family-based storage is ideal for time-series data. A scheduled Dataflow job can export data older than 90 days to BigQuery for cost-effective analytics. Option B is wrong because Cloud SQL cannot handle 100,000 writes per second -- it maxes out at thousands of writes/sec even with the largest instance, and single-row read latency exceeds 10ms under this load. Option C is wrong because Firestore has a limit of 10,000 writes/sec per database and is designed for document data, not high-throughput time-series. Option D is wrong because Memorystore for Redis is an in-memory store -- storing 90 days of data from 1M devices (1KB x 8,640 messages/day x 1M devices x 90 days = ~777 TB) in memory is not feasible or cost-effective.
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

Separate VPCs per application team with VPC peering to a shared services VPC provides strong network isolation between teams. VPC peering is non-transitive, so Team A cannot reach Team B through the shared services VPC. Each team gets their own VPC for autonomy, while the shared services VPC is accessible from all teams. Option A is wrong because a single VPC means all teams share the same network, and relying solely on firewall rules for isolation is error-prone. A misconfigured rule could expose one team's resources to another. Option B is wrong because Shared VPC connects service projects to a host project's network, meaning all service projects share the same VPC network. While firewall rules can restrict traffic, this does not provide the same level of isolation as separate VPCs. Option D is wrong because full mesh peering allows all teams to communicate with each other, violating the requirement that application teams cannot communicate directly.
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

BigQuery is a serverless, petabyte-scale analytics warehouse that supports standard SQL, complex joins, and requires zero infrastructure management. Using external tables for raw data on Cloud Storage avoids data duplication, while native tables for frequently queried data provide optimal performance. This gives data scientists self-service ad-hoc query capability. Option A is wrong because Dataproc requires cluster management (sizing, scaling, upgrades), violating the "without managing infrastructure" requirement. Hive is also slower than BigQuery for ad-hoc SQL queries. Option C is wrong because Cloud Spanner is an OLTP database optimized for transactional workloads, not analytical queries with complex joins across petabyte-scale datasets. Option D is wrong because AlloyDB is a regional PostgreSQL-compatible database with a maximum storage of 128 TB per instance -- it cannot handle petabyte-scale data. While its columnar engine is good for analytics, it requires instance management.
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

Private Google Access for on-premises hosts allows traffic from on-premises to reach Google APIs via Dedicated Interconnect without traversing the public internet. The restricted.googleapis.com VIP (199.36.153.4/30) routes API traffic through the Interconnect. Custom DNS on-premises resolves *.googleapis.com to this VIP. Cloud DNS inbound forwarding handles the DNS resolution chain. Option A is wrong because Private Google Access (standard) applies to VM instances within a VPC subnet that lack external IPs -- it does not enable on-premises hosts to access Google APIs. Option B is wrong because Private Service Connect creates endpoints within a VPC for accessing Google APIs, which works for VMs in the VPC but does not directly solve the on-premises access pattern without additional routing. Option D is wrong because Cloud NAT provides outbound internet access for VMs without external IPs -- it does not route on-premises traffic to Google APIs, and it specifically does not work with Interconnect traffic.
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

Sole-tenant nodes provide physical server isolation required for BYOL of Windows Server licenses (Microsoft licensing requires dedicated hardware or specific license mobility agreements). Local SSDs provide the highest IOPS and lowest latency storage. Using existing licenses avoids paying Google's Windows Server per-core premium, significantly reducing cost. Option B is wrong because it includes Google's Windows Server license fee (which is per-core and adds substantial cost), even though the company already owns licenses. Persistent Disk SSD also has lower performance than local SSDs. Option C is wrong because GKE Windows containers do not support running traditional Windows Server applications that require full OS features, and local SSDs on GKE nodes are ephemeral node storage, not directly available as persistent application storage. Option D is wrong because Bare Metal Solution is designed for specialized workloads like SAP HANA or Oracle databases, not general Windows application hosting. It is significantly more expensive than sole-tenant nodes.
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

Google Cloud Batch is designed for batch processing jobs. It automatically provisions GPU-enabled VMs when jobs are submitted and decommissions them when complete, so you pay only for processing time. Eventarc triggers batch jobs when files land in Cloud Storage. 100 files x 30 min = 50 GPU-hours/day, which Batch handles efficiently by parallelizing. Option A is wrong because GKE GPU node pools have a minimum scale of 0 but scaling GPU nodes takes several minutes, and the GKE overhead (control plane, monitoring) adds cost for what is essentially a batch workload. Option C is wrong because running GPU instances 24/7 wastes money -- at 50 GPU-hours/day, you are paying for 24 hours but using only ~50/24 = ~2 GPUs worth of compute. Option D is wrong because Vertex AI Custom Training is designed for ML training, not video transcoding. It adds unnecessary ML infrastructure overhead and is not cost-optimized for non-ML batch workloads.
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

(1) Cloud SQL is appropriate for a transactional e-commerce system -- it provides ACID compliance, relational data model for orders/inventory, and managed PostgreSQL/MySQL. (2) Memorystore (Redis) is ideal for session stores -- microsecond latency, key-value access pattern, and TTL support. (3) Firestore supports hierarchical data natively with collections and subcollections, perfect for product categories. (4) BigQuery is the standard for clickstream analytics -- serverless, petabyte-scale, SQL-based analytics. Option B is wrong because Cloud Spanner is over-engineered for most e-commerce systems (designed for global scale), and Bigtable is not ideal for session data (higher latency than Redis). Option C is wrong because Bigtable is not appropriate for session stores (no TTL natively, higher latency than Redis for small key-value lookups). Option D is wrong because Memorystore is an in-memory cache, not suitable for storing a product catalog (volatile, limited storage) -- Firestore is the right fit for hierarchical data.
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

Private Google Access allows pods (which lack external IPs) to reach Google APIs like Artifact Registry and BigQuery over Google's internal network. Cloud NAT is not needed because there is no stated requirement for external (non-Google) internet access. Connect Gateway or IAP TCP tunneling provides secure cluster management access without exposing a public endpoint. Option A is wrong because Cloud NAT is unnecessary if the only external access needed is to Google APIs (handled by Private Google Access). Adding Cloud NAT adds cost and complexity without benefit for this scenario. Option C is wrong because a proxy server adds operational overhead and a single point of failure. It is also unnecessary when Private Google Access provides direct access to Google services. Option D is wrong because you cannot create VPC peering to Google's API network -- Private Google Access and Private Service Connect are the supported mechanisms for accessing Google APIs privately.
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

Pub/Sub handles 1M events/sec ingestion reliably. Dataflow streaming mode supports windowed aggregations (5-minute sliding/tumbling windows) with exactly-once semantics using Apache Beam's windowing API. Bigtable provides single-digit millisecond reads by key, perfect for serving computed aggregations to trading applications. Option B is wrong because Cloud Functions have concurrency and scaling limits that make handling 1M events/sec unreliable, and Cloud SQL cannot handle the write throughput or serve reads with sub-second latency at this scale. Option C is wrong because managing Kafka on GKE and Spark on Dataproc requires significant operational overhead. While technically capable, it introduces unnecessary complexity compared to the fully managed Pub/Sub + Dataflow pipeline. Option D is wrong because BigQuery is not designed for sub-second point-lookup queries from trading applications. BigQuery's strength is analytical queries, not low-latency key-value serving. Minimum query latency in BigQuery is typically 1-2 seconds.
</details>

---

### Q37. An architect is designing a multi-cloud strategy where the company runs workloads on both GCP and AWS. They need a consistent container orchestration platform across both clouds with centralized policy management and service mesh. What should they use?

A) GKE on GCP and EKS on AWS, managed independently with separate CI/CD pipelines
B) Anthos with GKE on GCP and Anthos attached clusters on AWS (EKS), with Anthos Config Management and Anthos Service Mesh
C) Cloud Run on GCP and AWS Lambda, with Terraform managing both
D) GKE Autopilot on GCP and self-managed Kubernetes on AWS EC2 instances

<details>
<summary>Answer</summary>

**Correct: B)**

Anthos provides a consistent Kubernetes platform across clouds. Anthos attached clusters can manage EKS clusters on AWS. Anthos Config Management (ACM) provides centralized GitOps-based policy management, and Anthos Service Mesh extends service mesh capabilities across both environments. This is Google's purpose-built multi-cloud solution. Option A is wrong because managing GKE and EKS independently does not provide centralized policy management or a consistent service mesh, which are explicit requirements. Option C is wrong because Cloud Run and Lambda are different serverless platforms with different APIs, capabilities, and constraints. Terraform manages infrastructure but does not provide runtime consistency or service mesh. Option D is wrong because self-managed Kubernetes on EC2 requires significant operational effort and does not provide centralized policy management. GKE Autopilot and self-managed K8s have very different management models.
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

Vertex AI Model Garden provides access to foundation models (Google's Gemini, PaLM, and third-party models) that can be deployed with minimal ML expertise. Vertex AI Agent Builder enables grounding the model with proprietary data through managed RAG (Retrieval-Augmented Generation) pipelines, search capabilities, and conversational AI -- all without requiring deep ML knowledge. Option B is wrong because training a custom transformer from scratch requires extensive ML expertise, massive datasets, and significant compute resources -- the opposite of "minimal ML expertise." Option D is wrong because deploying on GKE with custom serving infrastructure requires Kubernetes expertise and ML operations knowledge, violating the minimal expertise requirement. Option E is wrong because calling OpenAI directly bypasses GCP's managed AI services, loses integration with proprietary data grounding, and introduces a third-party dependency without the benefits of Vertex AI's enterprise features (governance, monitoring, security).
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

Firestore has built-in offline support -- it caches data locally and automatically syncs when connectivity is restored. Each store can continue reading and writing locally during a 4-hour outage. Firestore's real-time listeners provide instant updates when online. Eventual consistency for cross-store queries is inherent in this architecture. Option A is wrong because Cloud Spanner requires continuous connectivity to the cloud and has no offline mode. If a store loses internet, it cannot read or write to Spanner. Option C is wrong because Cloud SQL has no built-in offline/sync capability. Custom replication logic between 200 stores and a central database is extremely complex to build and maintain (conflict resolution, sync ordering, etc.). Option D is wrong because Bigtable has no offline mode and edge caching is not a standard Bigtable feature. Custom edge caching would not support writes during connectivity loss.
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

This uses each service optimally. BigQuery native tables for structured data provide the best query performance and support partitioning/clustering for cost optimization. Semi-structured JSON on Cloud Storage with BigQuery external tables allows SQL analysis without data duplication. Cloud Storage handles unstructured data (images/videos) cost-effectively. This is the canonical data lake pattern on GCP. Option A is wrong because loading 200 TB of images and videos into BigQuery native tables is not possible (BigQuery is not designed for unstructured binary data) and storing 50 TB of JSON as native tables increases BigQuery storage costs unnecessarily. Option B is wrong because storing structured relational data as CSV in Cloud Storage and querying via external tables sacrifices query performance. BigQuery native tables are significantly faster for frequently queried structured data. Option D is wrong because Cloud SQL is not designed for 10 TB analytical workloads (it is OLTP), and Dataproc requires cluster management, adding operational overhead compared to serverless BigQuery.
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

Compute Engine MIG with scheduled scaling provides the best cost optimization: full capacity during business hours (10 AM - 6 PM) and scaled down to minimum (0 or 1) overnight. A100 GPUs are expensive, so scaling down 14 hours/day yields significant savings. Autoscaling handles intra-day traffic variations. Option A is wrong because Spot VMs can be preempted at any time, which is unacceptable for serving user-facing predictions. GPU Spot VMs are particularly likely to be preempted due to high demand. Option B is wrong because Vertex AI Endpoints do not currently support scaling to zero with GPU instances (minimum 1 replica), so you pay for at least 4 A100 GPUs 24/7. While convenient, it is more expensive than scheduled scaling. Option D is wrong because running GPU nodes 24/7 means paying for expensive A100 GPUs during the 14 hours of minimal overnight traffic -- a significant waste.
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

Shared VPC with a centralized host project is the simplest and most scalable approach. All service projects share the host project's VPC network, so custom routes in the host project direct all egress traffic through the firewall appliance. The security team manages the host project and firewall centrally. This avoids the complexity of hub-and-spoke for only 10 projects. Option B is wrong because VPC peering does not support transitive routing. If all egress must go through a central firewall, peered VPCs cannot use custom routes pointing to the hub's firewall because peering does not exchange custom routes pointing to other peering connections. Option C is wrong because VPN tunnels add latency, bandwidth limitations, and operational complexity. While hub-and-spoke with VPN works, it is over-engineered for 10 projects when Shared VPC provides a simpler solution. Option D is wrong because independent Cloud NAT gateways allow direct internet egress from each project, bypassing the centralized firewall inspection requirement entirely.
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

At 10 cameras x 30 fps = 300 frames/sec per factory, streaming raw video to the cloud would require far more than 50 Mbps bandwidth. Google Distributed Cloud Edge deploys Kubernetes and AI models at the edge (factory), processing frames locally with < 100ms latency. Only alerts and summaries (minimal bandwidth) are sent to GCP for aggregation and dashboarding. Option A is wrong because streaming raw video from 20 factories exceeds the 50 Mbps bandwidth constraint per factory. Additionally, round-trip latency to the cloud would likely exceed the 100ms requirement. Option C is wrong because Cloud Vision API requires sending each image to the cloud (bandwidth issue), introduces network latency, and is a general-purpose vision API, not optimized for specific manufacturing defect detection. Option D is wrong because batch prediction is not real-time -- it processes stored data, not live video frames, and cannot meet the 100ms latency requirement.
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

Cloud Spanner multi-region provides strong external consistency with automatic synchronous replication across regions. With proper schema design (stale reads with bounded staleness for sub-10ms local reads, or leader-aware routing), it meets the latency requirements. It handles the stated throughput easily. It is the only GCP database that supports multi-region strong consistency with writes from any region. Option B is wrong because Firestore multi-region stores data in one primary region with replicas. Writes must go through the primary, and strong consistency reads are only guaranteed from the primary region, not globally. Cross-region reads may see stale data. Option C is wrong because Cloud SQL read replicas provide eventual consistency only -- the replicas may lag behind the primary. There is no multi-primary write capability, so writes must go to a single primary region. Option D is wrong because Bigtable multi-cluster replication provides eventual consistency, not strong consistency. Writes in one cluster may take seconds to replicate to other clusters.
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

Migration Center (formerly StratoZone and mFit) provides automated discovery, assessment, and planning for large-scale migrations. It identifies workload dependencies, performance profiles, and recommends right-sized GCP targets. For 500 workloads, this assessment phase is essential to categorize workloads, identify dependencies, and plan migration waves. Option A is wrong because migrating without assessment risks breaking dependent systems and missing critical dependencies. Quick wins are part of the strategy but should come after assessment. Option C is wrong because refactoring all applications before migrating is the "big bang" anti-pattern -- it delays migration indefinitely and provides no business value until everything is refactored. The 6 R's framework recommends different strategies per workload. Option D is wrong because lifting and shifting everything simultaneously without assessment is reckless for 500 workloads. Dependencies, compliance requirements, and application characteristics must be understood first.
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

A phased approach is best for mission-critical Oracle databases. First, rehost Oracle on Bare Metal Solution to get off on-premises hardware quickly while maintaining full Oracle compatibility. Then, gradually replatform to AlloyDB (PostgreSQL-compatible, high performance) which can handle most Oracle features with moderate code changes. This minimizes risk and business disruption. Option B is wrong because Database Migration Service supports MySQL, PostgreSQL, SQL Server, and Oracle-to-PostgreSQL migrations, but a direct migration of 30 TB with complex PL/SQL packages requires significant code refactoring that cannot happen in a single step. It is high-risk for a mission-critical workload. Option C is wrong because rewriting all database code for Cloud Spanner is the most expensive and time-consuming option. Spanner's data model (interleaved tables, no stored procedures) requires a fundamentally different application architecture. Option D is wrong because Firestore is a document database completely unsuitable for relational workloads with complex SQL, joins, and PL/SQL packages. Retiring and rebuilding would be a complete system rewrite.
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

The strangler fig pattern incrementally replaces a monolith by extracting one module at a time. Starting with a low-risk, loosely coupled module reduces risk. An API gateway or reverse proxy routes requests to either the old monolith or the new microservice. Over time, more modules are extracted until the monolith can be decommissioned. Option A is wrong because migrating all 12 modules simultaneously is a big-bang migration, not the strangler fig pattern. It carries maximum risk and disruption. Option C is wrong because replicating the entire monolith on Compute Engine is a rehost (lift-and-shift), not the strangler fig pattern. Refactoring after rehosting is valid but is a different strategy (rehost-then-refactor). Option D is wrong because building all microservices before switching is essentially a parallel build with a big-bang cutover. The strangler fig specifically avoids this by incrementally routing traffic module by module.
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

Migrate to Virtual Machines (M2VM, formerly Migrate for Compute Engine/Velostrata) provides continuous block-level replication from VMware to GCP. It enables testing migrated VMs in GCP before cutover, and final cutover requires only minutes of downtime. No application modification is needed. Option B is wrong because manually exporting VMDKs and creating images is slow, error-prone, and requires extended downtime for each VM. There is no continuous replication, so data changes during migration are lost. Option C is wrong because the question explicitly states applications cannot be significantly modified. Containerizing 200 applications is a major refactoring effort. Option D is wrong because Transfer Appliance is for bulk data transfer (petabytes of data), not VM migration. It does not preserve VM configurations, networking, or OS state.
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

**Correct: B) and C)**

Software Assurance includes License Mobility, but Microsoft SQL Server requires dedicated hardware for BYOL in the cloud. Sole-tenant nodes provide the physical isolation needed for BYOL compliance, allowing the company to use existing licenses without additional cost. Alternatively, migrating to AlloyDB (PostgreSQL-compatible) eliminates SQL Server licensing entirely, providing the most cost savings long-term if the application can be adapted. Option A is wrong because Google-provided SQL Server Enterprise licenses are very expensive (per-vCPU pricing). When the company already owns licenses through Software Assurance, using BYOL is significantly cheaper. Option D is wrong because Cloud SQL includes licensing in its pricing -- you cannot bring your own license to Cloud SQL. You would be double-paying. Option E is wrong because Microsoft licensing requires dedicated hosts (sole-tenant) for BYOL in the cloud; running SQL Server BYOL on standard multi-tenant Compute Engine instances violates Microsoft licensing terms.
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

Mainframe modernization for a bank requires a conservative, phased approach. The strangler fig pattern incrementally migrates functionality from the mainframe to cloud services. Starting with read-only services (e.g., balance inquiries, statement viewing) is lowest risk. Dual Writes (writing to both mainframe and cloud systems during transition) maintain data consistency. A 2-3 year timeline is realistic for core banking. Option A is wrong because big-bang migration of a mainframe core banking system carries enormous risk -- mainframe applications often have undocumented dependencies, and a weekend cutover does not allow adequate testing. Any failure could impact banking operations. Option C is wrong because completely refactoring COBOL to Java before migrating would take years and provides no value until complete. This is the riskiest approach for a regulated bank. Option D is wrong because Database Migration Service does not support mainframe databases. Mainframe data structures (VSAM, IMS, DB2 on z/OS) require specialized migration tools and significant data model transformation.
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

Dataproc is Google's managed Hadoop/Spark service that is API-compatible with Apache Spark. Migrating HDFS data to Cloud Storage allows Spark jobs to run with minimal code changes (change hdfs:// to gs:// paths). Cloud Storage as the persistent data layer enables ephemeral Dataproc clusters (pay only when processing), decoupling compute from storage. Option B is wrong because rewriting Spark jobs as SQL queries is a significant refactoring effort, violating the "minimal changes" requirement. Many Spark jobs use complex UDFs, ML pipelines, and graph processing that cannot be expressed as SQL. Option C is wrong because running Spark on GKE requires Spark operator setup, custom container images, and manual cluster management -- more operational overhead than managed Dataproc, and not "minimal changes." Option D is wrong because replicating a self-managed Hadoop cluster on Compute Engine provides no cloud benefits -- you still manage HDFS, cluster scaling, patches, and failures. It is a lift-and-shift that misses the primary value of cloud migration.
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

This is a clean rehost (lift-and-shift) that achieves the 2-week timeline. M2VM handles continuous VM replication with minimal downtime cutover. DMS provides continuous MySQL replication to Cloud SQL, allowing validation before cutover. No application changes are required. Option B is wrong because refactoring to Cloud Run, GKE, and Cloud Spanner requires significant application changes (containerization, database schema redesign) and cannot be done in 2 weeks. Option C is wrong because containerizing a traditional 3-tier application requires creating Dockerfiles, Kubernetes manifests, and testing containerized behavior -- this is not "minimal changes" and is unlikely to be completed in 2 weeks. Running MySQL as a StatefulSet also adds operational complexity. Option D is wrong because manual VM export/import and MySQL dump/restore require extended downtime (the dump/restore of a large database can take hours or days) and do not provide the continuous replication that enables minimal-downtime cutover.
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

Retain (also called "revisit") means keeping workloads on-premises, typically connected via hybrid networking (Interconnect/VPN). This is appropriate for workloads with hard dependencies on specialized hardware that has no cloud equivalent. These workloads can be revisited in the future as cloud capabilities evolve. Option A is wrong because custom machine types on Compute Engine offer custom CPU/memory ratios but do not support FPGA accelerators or specialized NICs. The question explicitly states these cannot run on standard cloud infrastructure. Option C is wrong because Retire means decommissioning workloads that are no longer needed. These workloads are still running and serving a purpose -- they just cannot be migrated. Option D is wrong because Replace means substituting with a SaaS product. Workloads requiring custom FPGA accelerators are highly specialized and unlikely to have SaaS equivalents.
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

Database Migration Service supports migrations from AWS RDS PostgreSQL to Cloud SQL for PostgreSQL. DMS sets up continuous replication, keeping the Cloud SQL instance in sync with the source. When ready, the cutover takes only seconds to minutes, well within the 5-minute downtime tolerance. Option A is wrong because pg_dump/restore of a 500 GB database would take several hours (dump time + transfer time + restore time), far exceeding the 5-minute downtime window. Option C is wrong because while logical replication is technically possible, it requires manual configuration of publication/subscription, careful schema management, and sequence/index synchronization. DMS automates all of this and is the recommended tool. Option D is wrong because Transfer Appliance is for petabyte-scale offline data transfer, not 500 GB database migration. It also takes days/weeks for the physical shipping process.
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

(1) The legacy CRM can be rehosted (lift-and-shift) initially to quickly move to the cloud, with refactoring planned later. (2) On-premises email servers should be replaced with a SaaS solution (Google Workspace). (3) The analytics platform with hardware dependencies should be retained on-premises with hybrid connectivity. (4) The unused HR system should be retired (decommissioned). Option B is wrong because rehosting email servers makes no sense when Google Workspace (SaaS) is available, and refactoring the CRM as the first step is slower than rehosting. Option C is wrong because retiring email servers means eliminating email, not migrating it, and rehosting hardware-dependent workloads is explicitly stated as impossible. Option D is wrong because the CRM is custom-built, so there may not be a direct SaaS replacement. Rehosting is a safer first step.
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

Automated benchmarking provides consistent, repeatable, and efficient validation for 200 workloads. Capturing baselines before migration and running identical tests after allows objective comparison. Automated acceptance criteria (latency, throughput, error rates) flag workloads that need attention without manual review of each one. Option A is wrong because traffic mirroring to both environments requires running production workloads on both systems simultaneously, which doubles cost and is complex to set up for 200 workloads. It is also risky for workloads with side effects (writes, transactions). Option B is wrong because SLAs guarantee infrastructure availability, not application performance. A VM with the same specs may perform differently due to database tuning, network configuration, or application-level factors. Option C is wrong because manual testing of 200 workloads is too slow, inconsistent (different testers may use different methods), and does not scale. Automation is essential at this scale.
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

Containerization is a natural next step after lift-and-shift. GKE Autopilot manages infrastructure while providing the flexibility to incrementally extract microservices from the monolith (strangler fig pattern). This is lower risk than a complete refactor and delivers incremental value. Option A is wrong because a complete serverless refactor is a massive undertaking that provides no value until complete. It is high-risk for a production e-commerce platform. Option C is wrong because while CUDs reduce cost, this approach misses the opportunity to improve scalability, deployment velocity, and resilience through modernization. It leaves technical debt unaddressed. Option D is wrong because App Engine Standard has significant limitations (supported runtimes, request timeouts, limited VPC connectivity) that may not accommodate an existing monolithic application. It is also not an incremental modernization path.
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

Dataflow provides fully serverless, autoscaling data processing that eliminates cluster management entirely. For workloads that must remain Spark-based (due to code, libraries, or team expertise), Dataproc Serverless runs Spark workloads without managing clusters. Together, these modernize the pipeline while accommodating different workload needs. Option C is wrong because over-provisioning a fixed cluster is the opposite of modernization -- it increases cost and does not address operational overhead. Option D is wrong because replacing ETL with manual SQL scripts in Cloud SQL is a step backward in automation and does not support the data volumes typically processed by Dataproc. Option E is wrong because Cloud Functions have execution time limits (up to 60 minutes for 2nd gen), memory limits, and are not designed for large-scale data processing that ETL typically requires.
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

Vertex AI provides a unified, managed platform for the entire ML lifecycle. Feature Store eliminates feature engineering duplication, Training provides managed compute with hyperparameter tuning, Pipelines automate and version the ML workflow, and Endpoints handle model serving with autoscaling. This dramatically reduces time-to-production. Option A is wrong because adding GPUs to Compute Engine does not address the broader problems: pipeline orchestration, feature management, model versioning, A/B testing, and monitoring. It only speeds up one step. Option C is wrong because Dataproc MLlib is limited compared to TensorFlow/PyTorch capabilities and does not provide the full ML lifecycle management (serving, monitoring, feature store) that Vertex AI offers. Option D is wrong because eliminating custom training removes the ability to build models specific to the company's data and use cases. Pre-trained models are a complement, not a replacement, for custom training.
</details>

---

### Q60. An organization is running a monolithic application on GKE. They want to adopt a cloud-native architecture that improves deployment velocity, fault isolation, and independent scaling of components. They also want to implement GitOps practices. Which combination of improvements should they prioritize?

A) Keep the monolith on GKE and add more replicas for scaling
B) Decompose into microservices on GKE, implement Anthos Config Management for GitOps, and use Anthos Service Mesh for inter-service communication
C) Move the monolith from GKE to Cloud Run for simpler deployments
D) Decompose into microservices and deploy each on separate Compute Engine instances

<details>
<summary>Answer</summary>

**Correct: B)**

Microservices decomposition provides fault isolation and independent scaling. Anthos Config Management implements GitOps by syncing cluster configuration from Git repos. Anthos Service Mesh provides observability, traffic management, and security for inter-service communication. This is the cloud-native architecture pattern for GKE. Option A is wrong because adding replicas of a monolith does not improve deployment velocity (all components deploy together), fault isolation (a bug in one module crashes the entire app), or independent scaling (all modules scale together). Option C is wrong because moving a monolith to Cloud Run does not address the fundamental issues of monolithic architecture. Cloud Run has request timeout limits and is designed for microservices, not monoliths. Option D is wrong because deploying microservices on separate Compute Engine instances requires managing individual VMs, losing the benefits of container orchestration (scheduling, self-healing, rolling updates) that GKE provides.
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

A comprehensive FinOps approach addresses cost optimization holistically. Active Assist provides AI-driven recommendations for idle resources, right-sizing, and CUD purchases. Billing exports to BigQuery enable custom cost analysis and anomaly detection. Cost accountability per team drives responsible spending through chargebacks/showbacks. Option A is wrong because purchasing CUDs for all current resources locks in commitment for potentially oversized or unnecessary resources. You should right-size first, then commit. Option C is wrong because deleting non-production environments eliminates development and testing capability, which impacts velocity and quality. Instead, use scheduling (stop non-prod outside business hours) and right-sizing. Option D is wrong because moving to the cheapest region may increase latency for users, violate data residency requirements, or reduce availability. Region selection should consider multiple factors, not just cost.
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

This is the cloud-native pattern for resilience. Pub/Sub decouples services for non-real-time operations, eliminating cascading failures for those flows. Circuit breakers (e.g., via service mesh) prevent cascading failures by failing fast when a downstream service is unhealthy. Exponential backoff prevents retry storms. Option A is wrong because increasing timeouts makes the problem worse -- upstream services hold connections longer, exhausting thread pools and memory, accelerating the cascade. Option C is wrong because more replicas handle increased load but do not prevent cascading failures. If a downstream service is slow, more upstream replicas just create more blocked connections to the slow service. Option D is wrong because reverting to a monolith eliminates the benefits of microservices (independent deployment, scaling, fault isolation) and does not solve the fundamental issue -- a slow component in a monolith still blocks other components.
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

Vertex AI Agent Builder (formerly Gen AI App Builder) provides a managed platform for building AI agents grounded in enterprise data. It combines Gemini models with RAG (Retrieval-Augmented Generation) to generate contextual responses from the company's knowledge articles and support history. No custom model training is needed. Option A is wrong because training a custom LLM from scratch requires enormous compute resources, ML expertise, and data. Gemini models already have strong language understanding -- grounding them with proprietary data via RAG is far more efficient. Option C is wrong because BERT is a classification/embedding model, not a generative model. It cannot generate helpful responses -- it can only classify tickets into categories. Option D is wrong because BigQuery ML supports simple text classification but cannot build a conversational AI agent that understands context, retrieves knowledge, and generates natural language responses.
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

Going from 99.9% to 99.95% requires cross-region redundancy but not the complexity of active-active. A warm standby in a second region with a cross-region read replica (promotable to primary on failure) and global load balancing provides the reliability increase. Automated failover (via Cloud Functions or monitoring alerts) ensures rapid recovery. Option A is wrong because active-active across three regions with Cloud Spanner targets 99.999% availability, which is massive over-engineering for a 99.95% SLO. The cost and complexity are disproportionate to the 0.05% improvement needed. Option C is wrong because CDN and larger instances improve performance and handle regional load spikes but do not protect against regional outages, which is the primary risk at 99.9%+. Option D is wrong because Bigtable is a NoSQL wide-column store, not a relational database replacement for Cloud SQL. This would require a complete data model rewrite.
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

WAF recommends iterative improvement across all pillars, prioritized by risk. Security Command Center provides centralized security posture management (vulnerability scanning, compliance monitoring). Cloud Monitoring with SLOs provides measurable reliability targets. Billing alerts and Active Assist provide cost visibility and optimization recommendations. This approach improves all three areas systematically. Option A is wrong because addressing all gaps simultaneously is resource-intensive and risks overwhelming teams. WAF recommends iterative, prioritized improvements that build on each other. Option C is wrong because WAF has multiple pillars (operational excellence, security, reliability, cost optimization, performance). Focusing exclusively on one pillar leaves the organization vulnerable in others. Security gaps, for example, could result in breaches far more costly than any billing optimization. Option D is wrong because WAF emphasizes building internal capabilities and organizational knowledge. External consultants can accelerate progress, but relying entirely on them without involving internal teams creates a dependency and does not build sustainable practices.
</details>

---