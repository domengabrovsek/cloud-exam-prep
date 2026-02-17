# Section 2: Managing and Provisioning a Cloud Solution Infrastructure

> **Exam weight:** ~17.5% of the PCA exam.
>
> This section covers configuring network topologies, storage systems, compute systems, and leveraging Vertex AI for ML workflows. Questions test hands-on infrastructure knowledge at architect level.

**Total questions: 45**

---

### Q1. Your company is expanding into a new region and needs a dedicated, high-bandwidth connection between its on-premises data center and Google Cloud. The connection must provide at least 10 Gbps of throughput and meet strict latency requirements for real-time financial trading applications. Which connectivity option should you recommend?

A) Cloud VPN with HA configuration
B) Dedicated Interconnect with a 10 Gbps connection
C) Partner Interconnect with a 10 Gbps VLAN attachment
D) Cloud CDN with custom origins pointing to the data center

<details>
<summary>Answer</summary>

**Correct: B)**

Dedicated Interconnect provides a direct physical connection between your on-premises network and Google's network, offering 10 Gbps or 100 Gbps links with the lowest latency and highest bandwidth. For real-time trading applications that need strict latency guarantees and high throughput, Dedicated Interconnect is the right choice. Option A (Cloud VPN) is limited to ~3 Gbps per tunnel and traverses the public internet, adding latency variability. Option C (Partner Interconnect) goes through a service provider, adding an extra hop and slight latency; it is better when you cannot meet Google's colocation requirements. Option D (Cloud CDN) is a content delivery network for caching static content, not a private connectivity solution.
</details>

---

### Q2. A multinational retail company wants to connect 15 branch offices across Asia-Pacific to their Google Cloud VPC. Each branch needs between 50 Mbps and 500 Mbps of bandwidth. The company does not have colocation presence in any Google peering facility. Which connectivity solution is most appropriate?

A) Dedicated Interconnect with 10 Gbps circuits at each branch
B) Partner Interconnect through a supported service provider
C) Cloud VPN tunnels from each branch over the public internet
D) Cloud Router with BGP peering directly to Google

<details>
<summary>Answer</summary>

**Correct: B)**

Partner Interconnect is ideal when you need private connectivity to Google Cloud but cannot meet the colocation requirements for Dedicated Interconnect. It supports VLAN attachments from 50 Mbps to 50 Gbps through supported service providers. For 15 branch offices across Asia-Pacific, Partner Interconnect provides the right balance of bandwidth, privacy, and manageability. Option A (Dedicated Interconnect) requires colocation in a Google peering facility, which the company does not have, and 10 Gbps per branch is massive overkill. Option C (Cloud VPN) could work for some branches but adds latency variability and is less reliable for production workloads across 15 sites. Option D (Cloud Router) is a component used alongside Interconnect or VPN for dynamic routing, not a standalone connectivity solution.
</details>

---

### Q3. You are designing a network for a SaaS platform that serves customers globally. The platform has backend services in us-central1 and europe-west1. You need to route users to the closest healthy backend with a single anycast IP address. Which load balancer should you use?

A) Regional external Application Load Balancer
B) Global external Application Load Balancer (Classic)
C) Global external proxy Network Load Balancer
D) Regional internal passthrough Network Load Balancer

<details>
<summary>Answer</summary>

**Correct: B)**

The Global external Application Load Balancer provides a single anycast IP address and routes HTTP(S) traffic to the closest healthy backend across multiple regions. It supports URL maps, SSL termination, and Cloud CDN integration. For a globally distributed SaaS platform, this is the standard choice. Option A (Regional external ALB) only serves a single region and does not provide global anycast. Option C (Global external proxy NLB) handles TCP/SSL traffic but does not offer HTTP-level routing features like URL maps and host-based routing that a SaaS platform typically needs. Option D (Regional internal passthrough NLB) is for internal traffic within a region and is not internet-facing.
</details>

---

### Q4. Your organization runs workloads in both Google Cloud and AWS. You need to establish private connectivity between a GCP VPC in us-east1 and an AWS VPC in us-east-1 without routing traffic over the public internet. What is the recommended approach?

A) Set up Cloud VPN tunnels between GCP and AWS using each provider's VPN gateway
B) Use Dedicated Interconnect to connect on-premises, then route traffic from on-premises to AWS via AWS Direct Connect
C) Use Cross-Cloud Interconnect to establish a direct connection between Google Cloud and AWS
D) Peer both VPCs with a shared on-premises router using BGP

<details>
<summary>Answer</summary>

**Correct: C)**

Cross-Cloud Interconnect is Google Cloud's solution for establishing dedicated, private connectivity directly between Google Cloud and another cloud provider (such as AWS or Azure) without routing through on-premises infrastructure. It provides high bandwidth (10 Gbps or 100 Gbps) and low latency. Option A (Cloud VPN) traverses the public internet, which violates the requirement for private connectivity. Option B routes traffic through on-premises, adding latency, complexity, and cost. Option D also depends on on-premises infrastructure, which is unnecessary with Cross-Cloud Interconnect available.
</details>

---

### Q5. A healthcare company needs to protect its public-facing FHIR API running on GKE from DDoS attacks, SQL injection, and cross-site scripting (XSS). They also need to enforce geographic access restrictions to comply with data sovereignty regulations. Which combination of services should you recommend?

A) Cloud Armor security policies with preconfigured WAF rules, combined with VPC firewall rules
B) Cloud NAT with IP allow-listing and Identity-Aware Proxy
C) Cloud Armor security policies with preconfigured WAF rules and geo-based access control rules
D) VPC Service Controls with Cloud IDS

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud Armor provides DDoS protection, preconfigured WAF rules for OWASP Top 10 threats (including SQLi and XSS), and geo-based access control rules that can restrict traffic by source country. This single service addresses all three requirements: DDoS protection, application-layer attack filtering, and geographic restrictions. Option A includes VPC firewall rules which operate at L3/L4 and cannot enforce geographic restrictions based on source country. Option B (Cloud NAT) is for outbound traffic, and IAP is for identity-based access to internal apps, not public API protection. Option D (VPC Service Controls) protects Google Cloud API boundaries, not application-level HTTP traffic, and Cloud IDS detects threats but does not block them inline.
</details>

---

### Q6. Your team is designing a hub-and-spoke VPC network topology for a large enterprise. You have 25 spoke VPCs that all need to communicate with shared services in a hub VPC. Some spoke VPCs also need to communicate with each other. What is the most scalable approach?

A) Create VPC peering between the hub and each spoke, and between every pair of spokes that need communication
B) Use a hub VPC with VPC peering to each spoke and deploy a network virtual appliance (NVA) in the hub for spoke-to-spoke routing
C) Merge all spoke VPCs into a single Shared VPC
D) Use Cloud VPN tunnels between all VPCs

<details>
<summary>Answer</summary>

**Correct: B)**

VPC peering is non-transitive -- traffic from Spoke A cannot reach Spoke B through the Hub via peering alone. Deploying a network virtual appliance (such as a firewall or router VM) in the hub VPC and configuring custom routes allows spoke-to-spoke traffic to flow through the hub. This is a well-established hub-and-spoke pattern that provides centralized traffic inspection and policy enforcement. Option A creates an O(n^2) mesh of peering connections that is operationally complex and hits VPC peering limits. Option C (Shared VPC) is a flat model that removes network isolation between spokes, which may not meet security requirements. Option D (Cloud VPN) adds encryption overhead and is more complex to manage at scale than peering with an NVA.
</details>

---

### Q7. You need to configure DNS resolution so that VMs in your Google Cloud VPC can resolve hostnames for resources in your on-premises Active Directory domain (corp.example.com). The on-premises DNS server is at 10.0.1.10. What should you configure?

A) Create a Cloud DNS forwarding zone for corp.example.com that forwards queries to 10.0.1.10
B) Create a Cloud DNS private zone for corp.example.com with manual A records
C) Configure each VM's /etc/resolv.conf to point to 10.0.1.10
D) Create a Cloud DNS peering zone that peers with the on-premises network

<details>
<summary>Answer</summary>

**Correct: A)**

A Cloud DNS forwarding zone allows you to forward DNS queries for a specific domain (corp.example.com) to an on-premises DNS server (10.0.1.10) over VPN or Interconnect. This is the standard approach for hybrid DNS resolution. Option B requires manually creating and maintaining records, which is error-prone and does not scale. Option C bypasses Cloud DNS entirely and requires per-VM configuration; it also does not work if VPN/Interconnect routing is not configured for the metadata server. Option D (DNS peering zone) forwards queries to another VPC's Cloud DNS, not to an on-premises server.
</details>

---

### Q8. A media company needs to stream live video globally with ultra-low latency. They use a Global external Application Load Balancer in front of backend instances in three regions. They want to cache static assets (thumbnails, CSS, JS) at edge locations while ensuring live video streams are never cached. What should you configure?

A) Enable Cloud CDN on the backend service and use Cache-Control headers to set max-age=0 for live streams
B) Create two backend services: one with Cloud CDN enabled for static assets and one without CDN for live streams, using URL maps to route traffic
C) Enable Cloud CDN on all backend services and rely on the origin server to set no-cache headers
D) Use a third-party CDN instead of Cloud CDN for more granular caching control

<details>
<summary>Answer</summary>

**Correct: B)**

The cleanest architecture uses URL map path rules to route static asset requests (e.g., /static/*, /assets/*) to a backend service with Cloud CDN enabled, and live stream requests (e.g., /live/*) to a separate backend service without CDN. This provides clear separation of caching behavior. Option A risks caching live stream content if headers are misconfigured and adds unnecessary CDN overhead for live traffic. Option C depends entirely on origin headers being correct, which is fragile. Option D adds third-party complexity when Cloud CDN with proper URL map routing fully meets the requirements.
</details>

---

### Q9. Your organization has a Shared VPC host project with three service projects. The network team manages the host project. A developer in Service Project A needs to deploy a Cloud Run service that connects to a Cloud SQL instance on a private IP in the Shared VPC. What networking configuration is required?

A) Deploy Cloud Run with Direct VPC Egress or a Serverless VPC Access connector attached to the Shared VPC subnet, and grant the developer the `roles/vpcaccess.user` role on the host project
B) Configure VPC peering between the Cloud Run service project and the host project
C) Deploy the Cloud SQL instance with a public IP and use authorized networks
D) Create a separate VPC in Service Project A and use Cloud VPN to connect to the Shared VPC

<details>
<summary>Answer</summary>

**Correct: A)**

Cloud Run services connect to VPC networks through either Direct VPC Egress or a Serverless VPC Access connector. In a Shared VPC setup, the connector is created in the host project on a dedicated subnet, and users in service projects need the `roles/vpcaccess.user` role. Direct VPC Egress allows placing Cloud Run instances directly on a Shared VPC subnet. Either approach allows Cloud Run to reach the Cloud SQL private IP. Option B is incorrect because VPC peering is not how Shared VPC service projects connect to the host project (they use Shared VPC association). Option C exposes Cloud SQL to the internet, violating best practices. Option D adds unnecessary VPN complexity when Shared VPC already provides network sharing.
</details>

---

### Q10. An e-commerce company experiences a massive DDoS attack during a flash sale event. They have Cloud Armor configured but notice that some sophisticated L7 attacks are getting through the preconfigured WAF rules. They need an immediate response. What should you recommend? (Select TWO)

A) Enable Cloud Armor Adaptive Protection to automatically detect and mitigate anomalous traffic patterns
B) Create a Cloud Armor rate-limiting rule to throttle requests per IP
C) Disable the Global Load Balancer to stop all traffic
D) Move all backend instances to a different region
E) Switch from Premium Tier to Standard Tier networking

<details>
<summary>Answer</summary>

**Correct: A) and B)**

Cloud Armor Adaptive Protection uses machine learning to analyze traffic patterns and automatically generate suggested rules to block attack traffic, making it effective against sophisticated L7 attacks that bypass static WAF rules. Rate-limiting rules provide a complementary defense layer by throttling excessive requests per client IP. Together, these two measures significantly improve mitigation of advanced DDoS attacks. Option C (disabling the load balancer) takes down the entire service, which is worse than the attack itself. Option D (moving regions) does not help because the Global LB's anycast IP routes to all regions. Option E (Standard Tier networking) reduces network performance and does not improve DDoS protection -- in fact, Standard Tier does not support Cloud Armor.
</details>

---

### Q11. A data analytics company stores 500 TB of raw log data in Cloud Storage. Data less than 30 days old is queried frequently by analysts. Data between 30 and 365 days old is accessed about once per quarter for compliance audits. Data older than 365 days must be retained for 7 years but is almost never accessed. How should you configure the storage lifecycle?

A) Store all data in Standard class with a lifecycle rule to delete after 7 years
B) Store in Standard class; lifecycle rule to transition to Nearline after 30 days, Coldline after 365 days, and delete after 7 years
C) Store in Standard class; lifecycle rule to transition to Nearline after 30 days, Archive after 365 days, and delete after 7 years
D) Store all data in Coldline class from the start with a 7-year retention policy

<details>
<summary>Answer</summary>

**Correct: C)**

This lifecycle configuration matches each access pattern to the most cost-effective storage class. Standard for frequently accessed data (first 30 days), Nearline for quarterly access (30-365 days, minimum 30-day storage), and Archive for data rarely accessed but retained long-term (over 365 days, minimum 365-day storage). The 7-year delete rule handles compliance retention. Option A wastes money by keeping everything in Standard class. Option B uses Coldline instead of Archive for the long-term tier; Archive is cheaper when data is accessed less than once per year. Option D stores frequently accessed data in Coldline, which incurs high retrieval costs and has a 90-day minimum storage charge.
</details>

---

### Q12. Your team needs to implement a backup strategy for a mission-critical Cloud SQL for PostgreSQL instance. The RPO is 5 minutes and the RTO is 1 hour. The database is 2 TB. Which configuration meets these requirements?

A) Enable automated daily backups and rely on binary logging for point-in-time recovery (PITR)
B) Enable automated backups with point-in-time recovery (PITR) using write-ahead logs, and configure a cross-region read replica for disaster recovery
C) Export the database to Cloud Storage daily using a scheduled Cloud Function
D) Use Cloud SQL HA (regional) configuration only

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud SQL point-in-time recovery (PITR) for PostgreSQL uses write-ahead logs (WAL) to enable recovery to any point in time, achieving an RPO measured in seconds (well within 5 minutes). A cross-region read replica can be promoted to a standalone instance within minutes if the primary region fails, meeting the 1-hour RTO. Together, these provide both granular recovery and regional disaster recovery. Option A mentions binary logging, which is a MySQL concept, not PostgreSQL. Option C (daily exports) provides at best a 24-hour RPO, far exceeding the 5-minute requirement. Option D (HA configuration) protects against zonal failure but not regional failure, and HA alone does not define RPO/RTO for data loss scenarios.
</details>

---

### Q13. A genomics research lab generates 10 TB of sequencing data per week. The data must be immutable once written -- no one should be able to delete or modify it for 5 years to satisfy regulatory requirements. Researchers need to read the data frequently during the first 6 months, then rarely afterward. What storage configuration should you use?

A) Cloud Storage bucket with a 5-year retention policy (locked) in Standard class, with lifecycle rules to move to Coldline after 180 days
B) Cloud Storage bucket with Object Versioning enabled and Standard class only
C) Persistent Disk snapshots stored for 5 years
D) Cloud Storage bucket in Archive class with a 5-year retention policy (locked)

<details>
<summary>Answer</summary>

**Correct: A)**

A locked retention policy on a Cloud Storage bucket makes objects immutable -- they cannot be deleted or overwritten until the retention period expires. Locking the policy makes it permanent and irreversible, satisfying regulatory immutability requirements. Starting in Standard class accommodates the frequent reads in the first 6 months, and a lifecycle rule transitions to Coldline after 180 days to reduce costs for rarely accessed data. Option B (Object Versioning) keeps previous versions but does not prevent deletion of all versions; it does not enforce immutability. Option C (Persistent Disk snapshots) is not designed for long-term archival storage and is expensive at this scale. Option D (Archive class from the start) incurs high retrieval costs during the first 6 months of frequent access.
</details>

---

### Q14. Your company uses BigQuery as its data warehouse. A team stores 200 TB of sales data, but only the most recent 90 days of data is actively queried. Older data must be retained for 3 years. How should you optimize storage costs without impacting query performance on recent data?

A) Set a 90-day table expiration on all tables
B) Partition tables by ingestion time and let BigQuery automatically apply long-term storage pricing to partitions not modified for 90 days
C) Export data older than 90 days to Cloud Storage in Parquet format and delete it from BigQuery
D) Move older data to a separate BigQuery dataset in a cheaper region

<details>
<summary>Answer</summary>

**Correct: B)**

BigQuery automatically reduces storage pricing by approximately 50% for data in table partitions that have not been modified for 90 consecutive days (long-term storage pricing). By partitioning tables by date, older partitions automatically get cheaper storage without any data movement. Query performance on recent partitions is unaffected, and you can still query all data seamlessly. Option A deletes data after 90 days, violating the 3-year retention requirement. Option C requires managing an external pipeline and loses the ability to seamlessly query old data in BigQuery. Option D does not save money because BigQuery pricing does not vary significantly by region for storage, and it creates query complexity.
</details>

---

### Q15. A financial services company needs to ensure that Cloud Storage objects in a compliance bucket are protected from accidental deletion by any user, including project owners, for a minimum of 1 year. Which feature should you configure?

A) Object Versioning with lifecycle rules
B) A bucket lock with a 1-year retention policy
C) IAM Deny policies that prevent storage.objects.delete
D) A bucket-level Access Control List (ACL) restricting delete permissions

<details>
<summary>Answer</summary>

**Correct: B)**

A locked retention policy (bucket lock) enforces that no object in the bucket can be deleted or overwritten until the retention period expires. Once the retention policy is locked, even project owners and storage admins cannot remove or shorten it. This provides the strongest guarantee against accidental or malicious deletion. Option A (Object Versioning) retains previous versions but does not prevent deletion of the current version or all versions. Option C (IAM Deny policies) can restrict delete permissions but can be modified by Organization Admins, providing a weaker guarantee than bucket lock. Option D (ACLs) can be changed by bucket owners and do not provide immutability.
</details>

---

### Q16. You are designing a disaster recovery strategy for a Bigtable cluster that stores real-time ad-serving data. The cluster is in us-central1 and must fail over to us-east1 with an RPO of near zero and an RTO under 10 minutes. What should you configure?

A) A single-cluster Bigtable instance with daily exports to Cloud Storage in us-east1
B) A Bigtable instance with replication to a cluster in us-east1, with an application-level profile configured for multi-cluster routing
C) Manual Bigtable backups to Cloud Storage, with a restore script for us-east1
D) Two independent Bigtable instances with application-level dual writes

<details>
<summary>Answer</summary>

**Correct: B)**

Bigtable replication provides automatic, near-real-time replication between clusters in different regions. With a multi-cluster routing application profile, the client library automatically routes requests to the nearest available cluster and fails over transparently if one cluster becomes unavailable. This achieves near-zero RPO and sub-minute RTO. Option A (daily exports) provides an RPO of up to 24 hours, far exceeding the requirement. Option C (manual backups) introduces human delay and significant RPO/RTO gaps. Option D (dual writes) creates consistency challenges and requires complex application logic to handle write conflicts.
</details>

---

### Q17. A startup wants to migrate 200 TB of on-premises data to Cloud Storage. Their internet bandwidth is 1 Gbps. They need the data migrated within 2 weeks. Which transfer method should you recommend?

A) Use gsutil rsync over the internet
B) Use Transfer Appliance
C) Use Storage Transfer Service with a scheduled transfer
D) Use a Dedicated Interconnect provisioned for the migration

<details>
<summary>Answer</summary>

**Correct: B)**

At 1 Gbps, transferring 200 TB over the internet would take approximately 18.5 days (200 TB * 8 bits / 1 Gbps / 86400 seconds), exceeding the 2-week deadline even at 100% utilization, which is unrealistic. Transfer Appliance is a physical device shipped to your data center that you load with data and ship back to Google, supporting up to 300 TB per appliance and completing large transfers within days of Google receiving the device. Option A (gsutil) uses internet bandwidth and would exceed the deadline. Option C (Storage Transfer Service) also uses internet bandwidth with the same time constraint. Option D (Dedicated Interconnect) takes weeks to provision and is expensive for a one-time migration.
</details>

---

### Q18. Your application stores user-uploaded images in a multi-regional Cloud Storage bucket. You need to serve these images globally with low latency and reduce egress costs. The images are read-heavy (100:1 read-to-write ratio) and rarely change after upload. What should you configure?

A) Enable Autoclass on the bucket to optimize storage costs
B) Place Cloud CDN in front of a backend bucket connected to Cloud Storage
C) Create regional buckets in each region where users are located
D) Enable Requester Pays on the bucket to offset egress costs

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud CDN caches content at Google's edge locations worldwide, reducing latency for global users and significantly cutting egress costs for frequently accessed static content. With a 100:1 read-to-write ratio and rarely changing content, cache hit rates will be high. A backend bucket allows the Global external Application Load Balancer to serve Cloud Storage content through Cloud CDN. Option A (Autoclass) optimizes storage class costs but does not address serving latency or egress costs. Option C (regional buckets) requires complex replication logic and does not benefit from edge caching. Option D (Requester Pays) shifts costs to the requester but does not reduce total egress costs or improve latency.
</details>

---

### Q19. Your company needs to compute the total cost of ownership (TCO) for a workload running on Compute Engine. The workload requires 8 vCPUs, 32 GB RAM, and 500 GB SSD, running continuously for 3 years. Which pricing model minimizes cost? (Select TWO)

A) On-demand pricing with sustained use discounts
B) 3-year committed use discounts (CUDs)
C) Spot VM pricing
D) Preemptible VM pricing
E) 1-year committed use discounts (CUDs) renewed annually

<details>
<summary>Answer</summary>

**Correct: B) and E)**

For a workload running continuously for 3 years, committed use discounts provide the greatest savings. A 3-year CUD offers up to 57% discount on vCPUs and memory (the deepest discount). A 1-year CUD renewed annually offers up to 37% discount and provides more flexibility. Both are valid cost-optimization strategies for predictable, long-running workloads. Option A (sustained use discounts) provides only up to 30% discount automatically, less than either CUD tier. Option C (Spot VMs) and Option D (Preemptible VMs) can be terminated at any time by Google, making them unsuitable for workloads that must run continuously. Note: Preemptible VMs are the legacy version of Spot VMs with a 24-hour maximum runtime.
</details>

---

### Q20. A machine learning team needs to run training jobs that require high memory bandwidth and are compute-intensive. The training framework is TensorFlow. They need to minimize training time for a large language model with 70 billion parameters. Which compute option should you recommend?

A) Compute Engine with A3 Mega VMs (NVIDIA H100 GPUs)
B) Compute Engine with C3 high-cpu machine types
C) Cloud TPU v5p pods
D) GKE Autopilot with GPU node pools

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud TPU v5p pods are Google's most powerful AI accelerators, purpose-built for training large language models. TPUs provide massive interconnect bandwidth between chips (ICI network), enabling efficient parallelism for models with billions of parameters. TensorFlow has first-class TPU support through JAX and the TPU runtime. For a 70B parameter LLM, TPU pods offer the fastest training time. Option A (A3 Mega with H100 GPUs) is a strong choice for GPU-based training but TPU pods generally offer better price-performance for very large TensorFlow models. Option B (C3 high-cpu) has no accelerators and is unsuitable for LLM training. Option D (GKE Autopilot with GPUs) adds orchestration overhead without the TPU-level interconnect bandwidth needed for large-scale model parallelism.
</details>

---

### Q21. You are designing the compute architecture for an internal web application. The application has predictable traffic during business hours (8am-6pm) and minimal traffic overnight. The application runs in containers and the team wants to minimize operational overhead while paying only for actual usage. Which compute platform is most appropriate?

A) Compute Engine with a Managed Instance Group and scheduled autoscaling
B) GKE Autopilot with horizontal pod autoscaling
C) Cloud Run with minimum instances set to 0
D) Cloud Functions triggered by HTTP requests

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud Run is the best fit for a containerized application with variable traffic and a goal of minimal operational overhead. With minimum instances set to 0, you pay nothing during periods of no traffic (overnight), and Cloud Run automatically scales up during business hours. It is fully managed with no cluster infrastructure to manage. Option A (Compute Engine MIG) requires managing VM images, instance templates, and scaling policies -- more operational overhead. Option B (GKE Autopilot) reduces operational burden compared to Standard GKE but still requires managing Kubernetes resources (deployments, services, HPAs) and has a baseline cluster cost. Option D (Cloud Functions) is designed for event-driven functions, not full web applications, and has limitations on request duration and concurrency handling compared to Cloud Run.
</details>

---

### Q22. A gaming company needs to run game servers that require consistent, high single-thread performance and low latency. The servers are stateful and each instance handles up to 64 concurrent player connections. Which Compute Engine machine family should you recommend?

A) E2 general-purpose (cost-optimized)
B) C3D compute-optimized
C) N2 general-purpose with sole-tenant nodes
D) M3 memory-optimized

<details>
<summary>Answer</summary>

**Correct: B)**

C3D (and C3) compute-optimized machine types deliver the highest per-core performance in Compute Engine, with sustained all-core turbo frequencies. Game servers that depend on high single-thread performance and low latency benefit directly from compute-optimized instances. Option A (E2) uses shared-core and burstable configurations that deliver inconsistent performance. Option C (N2 with sole-tenant nodes) provides dedicated hardware for licensing or compliance but does not deliver higher per-core performance than C3D. Option D (M3 memory-optimized) is designed for in-memory databases and SAP workloads; while it provides large memory, its per-core performance focus is not as high as compute-optimized families.
</details>

---

### Q23. Your organization runs a microservices application on GKE. One microservice processes background jobs and can tolerate interruptions. Another microservice serves real-time API requests and must not be interrupted. Both run in the same GKE Standard cluster. How should you configure the node pools?

A) Run both workloads on a single node pool with standard (on-demand) VMs
B) Create two node pools: one with standard VMs for the API service and one with Spot VMs for the background job service, using node affinity and taints/tolerations
C) Use GKE Autopilot and let Google manage node provisioning
D) Create two separate GKE clusters, one for each workload

<details>
<summary>Answer</summary>

**Correct: B)**

Using separate node pools with different VM types allows you to optimize cost and reliability independently for each workload. The API service runs on standard (on-demand) VMs for guaranteed availability, while the background job service runs on Spot VMs for up to 91% cost savings. Taints on the Spot node pool prevent the API pods from being scheduled there, and tolerations on the background job pods allow them to run on Spot nodes. Option A misses the cost-saving opportunity for the fault-tolerant workload. Option C (GKE Autopilot) handles Spot VMs but the question specifies GKE Standard, and Autopilot provides less control over node pool configuration. Option D (separate clusters) wastes resources and increases operational complexity.
</details>

---

### Q24. A data engineering team needs to run Apache Spark jobs on GKE. The jobs process 10 TB of data daily and are batch-oriented. The team wants to use GKE's container orchestration but minimize cluster costs when no jobs are running. Which GKE configuration should you recommend?

A) GKE Standard with cluster autoscaler and a minimum of 3 nodes
B) GKE Autopilot with Spark on Kubernetes Operator
C) GKE Standard with node auto-provisioning and Spot VMs for Spark executors
D) GKE Standard with a static node pool of 50 n2-standard-8 nodes

<details>
<summary>Answer</summary>

**Correct: C)**

Node auto-provisioning (NAP) in GKE Standard automatically creates and deletes node pools based on workload requirements. Combined with Spot VMs for Spark executor pods (which are fault-tolerant), this minimizes costs when no jobs are running (nodes scale to zero) and optimally provisions nodes when Spark jobs launch. The Spark driver can run on a small on-demand node pool. Option A maintains a minimum of 3 nodes even when idle, incurring unnecessary cost. Option B (Autopilot) could work but provides less control over node selection and Spot VM configuration for cost optimization. Option D (static 50-node pool) wastes resources during idle periods and is vastly over-provisioned.
</details>

---

### Q25. Your company is migrating a monolithic application to GKE. The application needs to run across multiple clusters in different regions for high availability. The team needs centralized fleet management, consistent security policies, and service mesh capabilities across all clusters. Which solution should you recommend?

A) Deploy independent GKE Standard clusters in each region and manage them separately
B) Use GKE Autopilot clusters with Multi-Cluster Ingress
C) Use GKE Enterprise with fleet management, Config Sync, and Anthos Service Mesh
D) Deploy the application on Cloud Run in multiple regions instead of GKE

<details>
<summary>Answer</summary>

**Correct: C)**

GKE Enterprise (formerly Anthos) provides fleet management for centralized visibility and control across multiple GKE clusters. Config Sync ensures consistent policy and configuration across the fleet using GitOps. Anthos Service Mesh (built on Istio) provides service mesh capabilities including traffic management, observability, and mTLS between services across clusters. This combination addresses all three requirements. Option A (independent clusters) lacks centralized management and consistent policy enforcement. Option B (Multi-Cluster Ingress) handles cross-cluster traffic routing but does not provide fleet-wide policy management or service mesh. Option D (Cloud Run) is not suitable for migrating a monolithic application that needs Kubernetes-specific features.
</details>

---

### Q26. You are designing a GKE architecture for a fintech company. They need to ensure that their Kubernetes workloads meet PCI-DSS compliance requirements. The cluster must provide workload isolation, binary authorization for container images, and network policy enforcement. Which configuration should you use?

A) GKE Autopilot with Binary Authorization enabled
B) GKE Standard with GKE Sandbox (gVisor), Binary Authorization, and Calico network policies enabled
C) GKE Standard with default settings and namespace-level RBAC
D) Cloud Run with container image signing

<details>
<summary>Answer</summary>

**Correct: B)**

GKE Sandbox (gVisor) provides an additional layer of workload isolation by running containers in a sandboxed kernel, which is important for PCI-DSS compliance where multi-tenant workload isolation is required. Binary Authorization ensures only trusted, signed container images are deployed. Calico network policies (enabled via the `--enable-network-policy` flag) enforce pod-to-pod traffic restrictions. Together, these features address the three compliance requirements. Option A (Autopilot) supports Binary Authorization and has built-in security hardening but does not support GKE Sandbox for the deepest workload isolation. Option C (default GKE Standard) lacks all three required features. Option D (Cloud Run) does not support network policies or GKE Sandbox-level isolation.
</details>

---

### Q27. A logistics company runs a fleet tracking application that receives 50,000 GPS updates per second. The application runs on a MIG behind a TCP load balancer. During peak hours, the MIG needs to scale from 10 to 50 instances within 2 minutes. How should you configure autoscaling?

A) Configure autoscaling based on CPU utilization with a target of 60% and a stabilization window of 60 seconds
B) Configure autoscaling based on a custom Cloud Monitoring metric (GPS messages processed per instance) with a target value, and set a minimum of 10 instances
C) Configure scheduled autoscaling to add instances before known peak hours, combined with reactive autoscaling on CPU
D) Use predictive autoscaling only

<details>
<summary>Answer</summary>

**Correct: C)**

Combining scheduled autoscaling with reactive autoscaling provides the best outcome for workloads with known peak patterns. Scheduled autoscaling pre-provisions instances before peak hours, ensuring capacity is ready before the traffic surge arrives. Reactive autoscaling on CPU handles unexpected spikes beyond the scheduled capacity. This avoids the 2-minute scaling delay that could cause dropped connections during sudden peaks. Option A (CPU-based only) introduces a delay because instances must be created, booted, and pass health checks before serving traffic. Option B (custom metric) is more precise than CPU for application-specific scaling but still reactive and may not meet the 2-minute requirement. Option D (predictive autoscaling only) relies on historical patterns and may not account for unusual peak events.
</details>

---

### Q28. A retail company wants to deploy a containerized application that handles seasonal traffic spikes (10x baseline during holiday sales). They want zero infrastructure management, automatic scaling, and pay-per-request pricing. The application needs to connect to a Cloud SQL instance on a private IP. Which compute platform and configuration should you use?

A) Cloud Run with Direct VPC Egress and a maximum instance count of 1000
B) GKE Autopilot with cluster autoscaler
C) App Engine Flexible with VPC connector
D) Compute Engine MIG with autoscaling and a VPN to Cloud SQL

<details>
<summary>Answer</summary>

**Correct: A)**

Cloud Run provides zero infrastructure management, automatic scaling (including to zero), and per-request pricing. Direct VPC Egress allows Cloud Run services to access resources on private IPs in a VPC, including Cloud SQL. Setting a high maximum instance count (1000) accommodates the 10x traffic spike. Option B (GKE Autopilot) requires managing Kubernetes resources and does not offer pay-per-request pricing. Option C (App Engine Flexible) runs on VMs with per-hour (not per-request) pricing and does not scale as quickly as Cloud Run. Option D (Compute Engine MIG) requires significant infrastructure management and does not offer per-request pricing.
</details>

---

### Q29. You need to deploy a stateful application on GKE that requires persistent storage with ReadWriteOnce access mode. The application performs random read/write I/O and needs consistent sub-millisecond latency. Which storage option should you configure?

A) Persistent Disk (pd-standard) with a PersistentVolumeClaim
B) Persistent Disk (pd-ssd) with a PersistentVolumeClaim
C) Filestore (NFS) with ReadWriteMany access mode
D) Cloud Storage FUSE mounted as a volume

<details>
<summary>Answer</summary>

**Correct: B)**

pd-ssd (SSD Persistent Disk) provides consistent, sub-millisecond latency for random read/write I/O workloads. With ReadWriteOnce access mode, it can be attached to a single node, which is appropriate for stateful applications. PersistentVolumeClaims in GKE dynamically provision pd-ssd volumes when specified in the StorageClass. Option A (pd-standard) uses HDD-backed storage with higher latency (5-15ms), unsuitable for sub-millisecond requirements. Option C (Filestore) provides NFS with ReadWriteMany but has higher latency than SSD PD for random I/O. Option D (Cloud Storage FUSE) provides object storage access as a file system but has significantly higher latency than block storage and is best for read-heavy, sequential access patterns.
</details>

---

### Q30. Your company needs to run 100 parallel rendering jobs. Each job requires 96 vCPUs, 360 GB of RAM, and 4 NVIDIA T4 GPUs, and runs for approximately 4 hours. Jobs can be checkpointed and restarted. What is the most cost-effective compute configuration?

A) 100 n2-highmem-96 instances with T4 GPUs attached, using on-demand pricing
B) 100 a2-highgpu-4g instances using Spot VMs with checkpointing to Cloud Storage
C) GKE Autopilot with GPU node pools
D) 100 c2d-highcpu-112 instances without GPUs, using software rendering

<details>
<summary>Answer</summary>

**Correct: B)**

A2 instances are optimized for GPU workloads and come with NVIDIA A100 GPUs. However, for T4 GPUs, you would use N1 or custom machine types with T4 GPUs attached as Spot VMs. The key insight is that since jobs can be checkpointed and restarted, Spot VMs provide up to 91% cost savings. Checkpointing progress to Cloud Storage ensures that preempted jobs can resume rather than restart from scratch. Option A (on-demand) costs 60-91% more than Spot VMs for the same configuration. Option C (GKE Autopilot) adds Kubernetes overhead for simple batch jobs and provides less control over GPU VM selection. Option D (software rendering without GPUs) would take dramatically longer and likely cost more in compute hours despite not paying for GPUs.
</details>

---

### Q31. A healthcare company needs to choose between GKE Standard and GKE Autopilot for a new Kubernetes deployment. The application consists of 15 microservices, some of which require custom kernel parameters (sysctl settings) for high-performance networking. The team has limited Kubernetes expertise. What should you recommend?

A) GKE Autopilot because it requires less Kubernetes expertise
B) GKE Standard because Autopilot does not support custom kernel parameters (sysctl)
C) Cloud Run because it eliminates Kubernetes complexity entirely
D) GKE Autopilot with DaemonSets to configure kernel parameters

<details>
<summary>Answer</summary>

**Correct: B)**

GKE Standard is required when workloads need custom kernel parameters (sysctl settings), privileged containers, or other node-level customizations that Autopilot restricts for security. Autopilot manages the nodes entirely and does not allow sysctl modifications or privileged access. While the team has limited Kubernetes expertise, the custom kernel parameter requirement makes Standard the only viable GKE option. The team should invest in Kubernetes training or use managed services like Config Sync for operational simplification. Option A (Autopilot) cannot accommodate the sysctl requirement. Option C (Cloud Run) cannot run 15 interconnected microservices with custom networking requirements as effectively as GKE. Option D is incorrect because Autopilot restricts DaemonSets to system-level add-ons and does not allow custom sysctl modifications.
</details>

---

### Q32. You are designing a compute solution for a scientific simulation that requires 4 TB of RAM and 224 vCPUs. The simulation runs for 72 hours and the dataset must remain in memory throughout. Which machine type family should you use?

A) N2 general-purpose with extended memory
B) M3 memory-optimized
C) C3 compute-optimized
D) A3 accelerator-optimized

<details>
<summary>Answer</summary>

**Correct: B)**

M3 memory-optimized machine types offer up to 30 TB of memory, making them ideal for workloads that require extremely large in-memory datasets. The m3-megamem-128 provides 128 vCPUs and 1.9 TB of RAM, while the m3-ultramem-128 provides up to 3.8 TB. For 4 TB of RAM with 224 vCPUs, you would use a combination or the largest M3 configuration available. Option A (N2 with extended memory) supports custom memory configurations but maxes out well below 4 TB per instance. Option C (C3 compute-optimized) is optimized for CPU performance and does not offer the memory capacity needed. Option D (A3 accelerator-optimized) is designed for GPU workloads and does not provide 4 TB of system memory.
</details>

---

### Q33. Your company operates a global e-commerce platform. During a recent outage, the primary database in us-central1 failed, and it took 4 hours to restore service. Management mandates an RTO of less than 5 minutes for the database tier. The database is Cloud Spanner. What architecture should you implement?

A) Cloud Spanner regional instance with automated daily backups
B) Cloud Spanner multi-region instance configuration (e.g., nam-eur-asia1)
C) Cloud Spanner regional instance with a cross-region restore procedure
D) Cloud SQL with a cross-region read replica

<details>
<summary>Answer</summary>

**Correct: B)**

A Cloud Spanner multi-region configuration automatically replicates data across regions with synchronous writes. If one region fails, the other regions continue serving reads and writes without manual intervention, providing an effective RTO of near zero (no failover required). The nam-eur-asia1 configuration spans North America, Europe, and Asia. Option A (regional with daily backups) has an RTO measured in hours, not minutes, due to backup restoration time. Option C (cross-region restore) requires manual intervention and restoring from backups, exceeding the 5-minute RTO. Option D (Cloud SQL) is a different database service and does not match the existing Cloud Spanner architecture.
</details>

---

### Q34. An AI startup needs to train a custom image classification model on 10 million labeled images. The team uses TensorFlow/Keras and wants a managed service that handles distributed training, hyperparameter tuning, and model deployment. They do not want to manage infrastructure. Which service should you recommend?

A) Compute Engine with custom GPU VMs and a manual training pipeline
B) Vertex AI Training with custom training jobs, Vertex AI Vizier for hyperparameter tuning, and Vertex AI Endpoints for serving
C) GKE with Kubeflow Pipelines and TFServing
D) Cloud Functions that trigger training on each new batch of images

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI provides a fully managed ML platform. Vertex AI Training supports distributed training with automatic provisioning of GPU/TPU clusters. Vertex AI Vizier handles hyperparameter tuning with Bayesian optimization. Vertex AI Endpoints provide managed model serving with autoscaling. Together, these services form a complete managed pipeline from training to deployment without infrastructure management. Option A (Compute Engine) requires manual infrastructure management, contradicting the requirement. Option C (GKE with Kubeflow) provides flexibility but requires managing Kubernetes infrastructure. Option D (Cloud Functions) is not designed for ML training workloads and has execution time limits.
</details>

---

### Q35. A research team needs to decide between GPUs and TPUs for training a transformer-based NLP model. The model uses PyTorch with custom CUDA kernels. Training needs to scale to 256 accelerators. Which accelerator should you recommend and why?

A) Cloud TPU v5e because TPUs are always faster for transformer models
B) NVIDIA A100 GPUs (A2 VMs) because the custom CUDA kernels require NVIDIA GPU architecture
C) Cloud TPU v5p with automatic PyTorch-XLA conversion
D) NVIDIA H100 GPUs (A3 VMs) without any code changes

<details>
<summary>Answer</summary>

**Correct: B)**

Custom CUDA kernels are written specifically for NVIDIA's GPU architecture and cannot run on TPUs. While TPUs can run PyTorch through the PyTorch-XLA bridge, custom CUDA kernels bypass the standard framework and call NVIDIA-specific APIs directly. The A2 VM family provides NVIDIA A100 GPUs, which are well-suited for large-scale transformer training. Option A is incorrect because TPUs cannot execute custom CUDA code. Option C (TPU v5p with PyTorch-XLA) can run standard PyTorch operations but not custom CUDA kernels. Option D (H100 GPUs) would work with CUDA kernels but A100s also support them; the question asks about the primary decision factor (CUDA compatibility), not a specific GPU generation.
</details>

---

### Q36. Your organization is building a Vertex AI Pipeline that trains a model daily, evaluates it against a baseline, and conditionally deploys it to production. The pipeline should be version-controlled, reproducible, and run on a schedule. Which architecture should you use?

A) A Cloud Scheduler cron job that triggers a Cloud Function to run training on a Compute Engine VM
B) Vertex AI Pipelines using Kubeflow Pipelines SDK (KFP) with components for training, evaluation, and conditional deployment, triggered by Cloud Scheduler
C) A bash script on a Compute Engine VM that runs daily via cron
D) Vertex AI AutoML with scheduled retraining

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI Pipelines provides a managed, serverless environment for running ML pipelines built with the Kubeflow Pipelines SDK. Pipelines are defined as code (version-controllable), produce artifacts that are tracked for reproducibility, and support conditional logic (deploy only if evaluation metrics exceed the baseline). Cloud Scheduler triggers pipeline runs on a schedule. Option A (Cloud Function + Compute Engine) requires manual orchestration and lacks built-in ML pipeline features like artifact tracking and conditional branching. Option C (bash script) is not reproducible, version-controlled, or managed. Option D (AutoML with retraining) does not support custom conditional deployment logic or the flexibility of a full pipeline.
</details>

---

### Q37. A company wants to train a large language model (LLM) with 175 billion parameters. They need thousands of accelerators with high-bandwidth interconnect. The training job will run for several weeks. Which Google Cloud infrastructure should you recommend?

A) Multiple A2 VMs with NVIDIA A100 GPUs, connected over standard VPC networking
B) AI Hypercomputer with Cloud TPU v5p multislice pods
C) A single A3 Mega VM with 8 NVIDIA H100 GPUs
D) GKE Autopilot with GPU node pools spread across regions

<details>
<summary>Answer</summary>

**Correct: B)**

AI Hypercomputer is Google's supercomputing architecture that combines TPU v5p pods with multislice training, enabling thousands of TPU chips to work together on a single training job with ultra-high-bandwidth inter-chip interconnect (ICI). Multislice allows training to span multiple TPU pods with efficient cross-pod communication. For a 175B parameter model requiring weeks of training, this provides the scale and bandwidth needed. Option A (A2 VMs over VPC) lacks the high-bandwidth interconnect needed between thousands of accelerators; standard VPC networking creates a bottleneck. Option C (single A3 Mega) has only 8 GPUs, far insufficient for a 175B parameter model. Option D (GKE Autopilot across regions) introduces cross-region latency that cripples distributed training performance.
</details>

---

### Q38. Your data science team wants to experiment with multiple foundation models for a text summarization task before committing to one. They want to compare models from Google, open-source (Llama, Mistral), and third-party providers without deploying each model individually. Which Google Cloud service should you recommend?

A) Vertex AI Model Garden
B) Vertex AI AutoML
C) Cloud Natural Language API
D) BigQuery ML

<details>
<summary>Answer</summary>

**Correct: A)**

Vertex AI Model Garden provides a curated catalog of foundation models from Google (Gemini, PaLM), open-source (Llama, Mistral, Falcon), and third-party providers. Data scientists can discover, test, and deploy models through a unified interface without managing individual model deployments. Model Garden supports one-click deployment to Vertex AI Endpoints and includes model cards for comparison. Option B (AutoML) trains custom models on your data but does not provide a catalog of pre-trained foundation models for comparison. Option C (Cloud Natural Language API) is a prebuilt API for specific NLP tasks, not a model exploration platform. Option D (BigQuery ML) supports ML within BigQuery but does not offer a broad model catalog for foundation model comparison.
</details>

---

### Q39. A customer support team wants to build an AI-powered chatbot that can answer questions about their product documentation (500+ pages of PDFs and HTML). The chatbot should provide accurate, grounded answers and cite sources. The team has no ML expertise. Which approach should you recommend?

A) Fine-tune a Gemini model on the product documentation
B) Use Vertex AI Agent Builder with a data store grounded in the product documentation
C) Use the Cloud Natural Language API to extract entities and build a rule-based chatbot
D) Train a custom BERT model on the documentation using Vertex AI Training

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI Agent Builder (formerly Vertex AI Search and Conversation) allows you to create AI-powered chatbots grounded in your enterprise data without ML expertise. You upload your documentation to a data store, and Agent Builder uses retrieval-augmented generation (RAG) to provide accurate, grounded answers with source citations. It supports PDF, HTML, and other document formats out of the box. Option A (fine-tuning Gemini) requires ML expertise and is overkill for a document Q&A use case. Option C (Cloud Natural Language API) extracts entities and sentiment but does not support conversational Q&A. Option D (custom BERT training) requires significant ML expertise and is unnecessary when Agent Builder provides the functionality out of the box.
</details>

---

### Q40. You are developing a mobile application that needs to identify objects in user-uploaded photos and return labels with confidence scores. The team needs this feature in production within one week and has no ML team. Which approach should you use?

A) Train a custom Vision model using Vertex AI AutoML
B) Use the Cloud Vision API (pre-built) for label detection
C) Deploy a pre-trained ResNet model on Vertex AI Endpoints
D) Use Gemini's multimodal capabilities through the Gemini API

<details>
<summary>Answer</summary>

**Correct: B)**

The Cloud Vision API is a pre-built, production-ready API that provides label detection (among other features) with no ML expertise or training required. It returns labels with confidence scores for objects detected in images and can be integrated via REST API or client libraries in days, well within the one-week deadline. Option A (AutoML Vision) requires collecting and labeling training data, which exceeds the one-week timeline. Option C (custom ResNet deployment) requires ML expertise for model selection, optimization, and deployment. Option D (Gemini API) could identify objects but is a general-purpose model that is more expensive per request and returns natural language descriptions rather than structured labels with confidence scores optimized for object detection.
</details>

---

### Q41. Your company wants to build a document processing pipeline that extracts structured data from invoices, receipts, and contracts. The documents come in various formats (PDF, images, scanned documents). Which Google Cloud service should you use?

A) Cloud Vision API with OCR feature
B) Document AI (pre-built processors for invoices, receipts, and contracts)
C) Vertex AI AutoML for document classification
D) BigQuery ML with text extraction functions

<details>
<summary>Answer</summary>

**Correct: B)**

Document AI provides specialized, pre-built processors for common document types including invoices, receipts, and contracts. These processors extract structured data (line items, totals, dates, parties, clauses) with high accuracy and handle various formats including PDFs, images, and scanned documents with built-in OCR. Option A (Cloud Vision API OCR) extracts raw text but does not understand document structure or extract specific fields like invoice line items. Option C (AutoML) would require labeled training data and training time, while Document AI provides pre-built models ready to use. Option D (BigQuery ML) does not have document processing capabilities.
</details>

---

### Q42. A media company wants to generate product descriptions automatically using AI. They need the generated text to follow their brand voice and style guidelines. The team has 10,000 examples of product descriptions written by their content team. Which approach should you recommend?

A) Use the Gemini API with a detailed system prompt describing the brand voice
B) Fine-tune a Gemini model on the 10,000 product description examples using Vertex AI
C) Use Vertex AI Agent Builder with the examples as a data store
D) Use the Cloud Translation API to convert templates into product descriptions

<details>
<summary>Answer</summary>

**Correct: B)**

Fine-tuning a Gemini model on 10,000 brand-specific examples teaches the model to replicate the exact brand voice, tone, and style patterns. This produces higher-quality, more consistent outputs than prompt engineering alone because the model's weights are adjusted to match the specific style. Vertex AI provides managed fine-tuning infrastructure. Option A (system prompt) can approximate the style but is limited in how precisely it can replicate a specific brand voice compared to fine-tuning, especially with 10,000 available examples. Option C (Agent Builder) is designed for search and Q&A over documents, not text generation in a specific style. Option D (Cloud Translation API) translates between languages, not between styles.
</details>

---

### Q43. Your organization wants to deploy a Gemini model for internal use but needs to ensure that prompts and responses are monitored for safety, bias, and policy compliance. Which Vertex AI features should you use? (Select TWO)

A) Vertex AI Model Monitoring for prediction drift detection
B) Responsible AI safety filters and content classification built into the Gemini API
C) Vertex AI Evaluation for benchmark testing before deployment
D) Cloud Audit Logs for tracking API calls
E) VPC Service Controls to restrict model access

<details>
<summary>Answer</summary>

**Correct: B) and C)**

The Gemini API includes built-in responsible AI safety filters that classify and optionally block content across categories like harassment, hate speech, sexually explicit content, and dangerous content. These filters run on every prompt and response in real time. Vertex AI Evaluation allows you to benchmark model outputs against test datasets before deployment, measuring quality, safety, and bias metrics. Together, these provide pre-deployment evaluation and runtime safety monitoring. Option A (Model Monitoring) tracks feature drift and prediction drift for tabular/custom models, not generative AI content safety. Option D (Cloud Audit Logs) records who called the API and when, but does not inspect content for safety or bias. Option E (VPC Service Controls) restricts network-level access but does not monitor content.
</details>

---

### Q44. A pharmaceutical company wants to use AI to analyze medical images (X-rays, MRIs) for anomaly detection. They have a dataset of 50,000 labeled medical images. Due to regulatory requirements, the model must run entirely within their Google Cloud project and cannot send data to external APIs. Which approach should you use?

A) Use the Cloud Vision API for medical image analysis
B) Train a custom model using Vertex AI AutoML Vision on the labeled dataset, deployed to a Vertex AI Endpoint within the project
C) Use Gemini's multimodal capabilities to analyze the images
D) Use a pre-trained model from Vertex AI Model Garden without any fine-tuning

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI AutoML Vision allows you to train a custom image classification or object detection model on your own labeled data. The resulting model is deployed to a Vertex AI Endpoint within your Google Cloud project, ensuring all data stays within your project boundary. With 50,000 labeled images, AutoML Vision can train a high-quality model specific to the medical imaging domain. Option A (Cloud Vision API) is a shared, pre-built API that is not specialized for medical imaging and sends data to a shared Google service. Option C (Gemini multimodal) is a general-purpose model not optimized for specialized medical anomaly detection and may not meet regulatory requirements for data isolation. Option D (pre-trained model without fine-tuning) would not achieve the accuracy needed for medical imaging without domain-specific training.
</details>

---

### Q45. Your company has a Vertex AI Pipeline that trains and deploys a fraud detection model. The model's prediction accuracy has been declining over the past month due to changing fraud patterns. You need to implement an automated system that detects model degradation and triggers retraining. Which architecture should you implement?

A) Manually review model metrics weekly and retrain when accuracy drops
B) Configure Vertex AI Model Monitoring to detect feature drift and prediction drift, with alerts that trigger a Cloud Function to launch a retraining pipeline
C) Schedule the pipeline to retrain the model daily regardless of performance
D) Deploy multiple model versions and randomly route traffic between them

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI Model Monitoring continuously monitors deployed models for feature drift (changes in input data distribution) and prediction drift (changes in prediction output distribution). When drift exceeds configured thresholds, it generates alerts. By connecting these alerts to a Cloud Function (via Cloud Monitoring notification channels), you can automatically trigger a Vertex AI Pipeline run to retrain the model on recent data. This creates a closed-loop MLOps system. Option A (manual review) is slow, error-prone, and does not meet the "automated" requirement. Option C (daily retraining) wastes compute resources when the model is performing well and may not retrain quickly enough when drift is severe. Option D (multiple versions with random routing) does not address model degradation and could serve predictions from outdated models.
</details>

---
