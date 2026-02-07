# Section 2: Planning and Implementing a Cloud Solution

> **Exam weight:** ~30% of the standard ACE exam, ~40% of the renewal exam.
>
> This section covers planning and estimating pricing, planning compute/data storage/networking resources, creating compute/data storage/networking resources, deploying with Cloud Marketplace, and implementing infrastructure as code (Terraform, Config Connector).

**Total questions: 58**

---

### Q1. A startup wants to run a containerized web API. They want zero infrastructure management, the ability to scale to zero during off-hours to save costs, and they need to handle multiple concurrent requests per instance. Which service should they use?

A) Compute Engine with a Managed Instance Group
B) GKE Autopilot
C) Cloud Run
D) Cloud Functions (2nd gen)

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud Run is a fully managed serverless platform for containers. It scales to zero, requires no infrastructure management, and supports multiple concurrent requests per instance. Option A (Compute Engine MIG) requires infrastructure management and does not scale to zero. Option B (GKE Autopilot) manages nodes but does not scale to zero (minimum 1 node). Option D (Cloud Functions) can also handle this but is designed for single-purpose event-driven functions, not full web APIs -- and Cloud Run offers more flexibility with container images and concurrent request handling.
</details>

---

### Q2. A company needs to run a batch data processing job that processes 500 GB of data weekly. The job is fault-tolerant and can be restarted if interrupted. They want to minimize cost. What compute option should they use?

A) Compute Engine with a standard e2-medium VM
B) Compute Engine with Spot VMs
C) Cloud Run with maximum CPU allocation
D) GKE Standard with a dedicated node pool

<details>
<summary>Answer</summary>

**Correct: B)**

Spot VMs offer 60-91% discounts off on-demand pricing and are ideal for fault-tolerant batch processing workloads. Since the job can be restarted if interrupted, the preemption risk is acceptable. Unlike the older Preemptible VMs, Spot VMs have no maximum runtime limit. Option A uses standard pricing and costs significantly more. Option C is designed for request-driven workloads, not batch processing. Option D adds Kubernetes overhead unnecessarily for a simple batch job.
</details>

---

### Q3. Your application requires exactly 6 vCPUs and 20 GB of RAM. The closest predefined machine type offers 8 vCPUs and 32 GB RAM. How can you avoid overprovisioning?

A) Use an e2-standard-8 and accept the extra resources
B) Use a custom machine type with 6 vCPUs and 20 GB memory
C) Use an e2-standard-4 and vertically scale if needed
D) Use two e2-standard-4 instances with load balancing

<details>
<summary>Answer</summary>

**Correct: B)**

Custom machine types let you specify the exact number of vCPUs and memory your workload needs. This avoids paying for unused resources. vCPUs must be an even number (or 1), and memory must be between 0.5-8 GB per vCPU and a multiple of 256 MB. Custom machine types carry a 5% premium per resource unit, but the savings from right-sizing typically outweigh this. Option A wastes resources and money. Option C underprovisioning could cause performance issues. Option D adds unnecessary complexity.
</details>

---

### Q4. A financial services company needs a relational database that provides strong consistency, horizontal scalability, and 99.999% availability across multiple regions. Which database service should they choose?

A) Cloud SQL with regional HA enabled
B) Cloud Spanner with a multi-region configuration
C) AlloyDB with read replicas
D) BigQuery

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud Spanner is the only GCP database that provides strong (external) consistency with horizontal scalability and up to 99.999% availability in multi-region configurations. It is designed for global financial applications that need these properties. Option A (Cloud SQL) maxes out at 99.95% with regional HA and does not support horizontal scaling. Option C (AlloyDB) is high-performance PostgreSQL-compatible but is regional, not multi-region. Option D (BigQuery) is an analytics data warehouse, not an OLTP transactional database.
</details>

---

### Q5. You need to store 50 TB of IoT sensor data that arrives at millions of writes per second. The data will be queried by row key for real-time dashboards. Which storage option is best?

A) Cloud SQL
B) Firestore
C) Bigtable
D) BigQuery

<details>
<summary>Answer</summary>

**Correct: C)**

Bigtable is a wide-column NoSQL database designed for high-throughput, low-latency workloads at scale. It excels at IoT time-series data, handling millions of writes per second with single-digit millisecond latency. It is cost-effective for datasets over 1 TB. Option A (Cloud SQL) cannot handle millions of writes per second or 50 TB efficiently. Option B (Firestore) is designed for document-oriented mobile/web apps, not high-throughput time-series. Option D (BigQuery) is an analytics warehouse optimized for complex queries, not real-time low-latency row key lookups.
</details>

---

### Q6. A company stores compliance archives that must be retained for 7 years and are accessed less than once per year. They want to minimize storage cost. Which Cloud Storage class should they use?

A) Standard
B) Nearline
C) Coldline
D) Archive

<details>
<summary>Answer</summary>

**Correct: D)**

Archive storage has the lowest storage cost and is designed for data accessed less than once per year. It has a 365-day minimum storage duration with early deletion charges, but since the data must be retained for 7 years, this is not a concern. All storage classes have the same millisecond access latency and eleven-nines durability -- Archive is not slower to access, just more expensive per retrieval. Option A is the most expensive for rarely accessed data. Options B (30-day min, monthly access) and C (90-day min, quarterly access) cost more for storage than Archive.
</details>

---

### Q7. Your web application serves users globally and needs an HTTPS load balancer with DDoS protection via Cloud Armor. It also needs Cloud CDN for static assets. Which load balancer and network tier should you use?

A) External Application Load Balancer with Standard Tier
B) External Passthrough Network Load Balancer with Premium Tier
C) External Application Load Balancer with Premium Tier
D) Internal Application Load Balancer with Premium Tier

<details>
<summary>Answer</summary>

**Correct: C)**

Global HTTP(S) load balancing, Cloud Armor, and Cloud CDN all require Premium Tier networking. The External Application Load Balancer (formerly HTTP(S) LB) operates at Layer 7 and supports all these features. Option A is wrong because Standard Tier does not support global load balancing, Cloud CDN, or Cloud Armor. Option B is a Layer 4 passthrough load balancer that does not support Cloud Armor or Cloud CDN. Option D is for internal traffic within a VPC, not external-facing applications.
</details>

---

### Q8. Your application serves users in a single region and cost optimization is the top priority. Latency is not critical. Which Network Service Tier should you use?

A) Premium Tier -- it is the default and always recommended
B) Standard Tier -- lower egress cost, regional-only IP addresses
C) Premium Tier -- required for all external traffic
D) Standard Tier -- it provides the same SLA as Premium

<details>
<summary>Answer</summary>

**Correct: B)**

Standard Tier offers lower egress pricing by using hot-potato routing (traffic travels over the public internet rather than Google's private backbone). It is appropriate for single-region applications where global performance is not needed and cost is the priority. Standard Tier only supports regional external IPs and regional load balancers. Option A is not always recommended when cost is the top priority. Option C is incorrect -- Premium is not required for all external traffic. Option D is wrong because Standard Tier does not have a comparable SLA to Premium (which offers 99.99%).
</details>

---

### Q9. A mobile application needs a database with real-time synchronization, offline support, and automatic scaling. The data model is document-oriented. Which database should you choose?

A) Cloud SQL with PostgreSQL
B) Firestore in Native mode
C) Firestore in Datastore mode
D) Bigtable

<details>
<summary>Answer</summary>

**Correct: B)**

Firestore in Native mode provides real-time listeners for data synchronization, offline support for mobile and web clients, automatic scaling, and a document-oriented data model with collections and documents. Option A (Cloud SQL) is relational and does not provide real-time sync or offline support. Option C (Datastore mode) is for server-side applications migrating from the legacy Datastore API and does not support real-time synchronization. Option D (Bigtable) is for high-throughput analytical workloads, not mobile apps.
</details>

---

### Q10. You need to expose a UDP-based game server running on Compute Engine to external players. Which load balancer should you use?

A) External Application Load Balancer
B) External Proxy Network Load Balancer (TCP Proxy)
C) External Passthrough Network Load Balancer
D) Internal Passthrough Network Load Balancer

<details>
<summary>Answer</summary>

**Correct: C)**

The External Passthrough Network Load Balancer is the only external load balancer that supports UDP traffic. It operates at Layer 4 and passes traffic directly to backend instances (preserving client source IP). It is regional in scope. Option A only supports HTTP/HTTPS (Layer 7). Option B only supports TCP (not UDP). Option D is for internal traffic, not external players.
</details>

---

### Q11. You are creating a Compute Engine VM that will host a database. The data disk must survive if the VM is accidentally deleted. Which flag should you use when creating the additional data disk?

A) `--create-disk=name=data-disk,size=200GB,type=pd-ssd,auto-delete=yes`
B) `--create-disk=name=data-disk,size=200GB,type=pd-ssd,auto-delete=no`
C) `--boot-disk-auto-delete=no`
D) `--disk=name=data-disk,mode=ro`

<details>
<summary>Answer</summary>

**Correct: B)**

Setting `auto-delete=no` on the data disk ensures it is NOT deleted when the VM is deleted. By default, boot disks are deleted with the VM, but additional disks are NOT deleted by default. However, when creating an additional disk inline with `--create-disk`, explicitly setting `auto-delete=no` is a best practice to protect data. Option A sets `auto-delete=yes`, which would delete the disk with the VM. Option C affects the boot disk, not the data disk. Option D attaches in read-only mode, which is not appropriate for a database.
</details>

---

### Q12. You want to enable OS Login on all VMs in your project so that SSH access is managed through IAM. A developer needs SSH access with sudo privileges. What should you do?

A) Add the developer's SSH key to the project metadata and grant `roles/compute.instanceAdmin`
B) Enable OS Login at the project level and grant the developer `roles/compute.osAdminLogin`
C) Enable OS Login at the project level and grant the developer `roles/compute.osLogin`
D) Enable OS Login at the project level and grant the developer `roles/owner`

<details>
<summary>Answer</summary>

**Correct: B)**

OS Login is enabled by setting `enable-oslogin=TRUE` in project metadata. Once enabled, SSH access is controlled through IAM roles: `roles/compute.osLogin` grants basic SSH access, and `roles/compute.osAdminLogin` grants SSH access with sudo privileges. Since the developer needs sudo, `osAdminLogin` is correct. Option A uses metadata-based SSH keys, which is the older approach that OS Login replaces. Option C grants SSH without sudo. Option D (Owner) is overly permissive and violates least privilege.
</details>

---

### Q13. You create a regional GKE cluster with `--region=us-central1 --num-nodes=2`. How many total nodes will be created?

A) 2 nodes
B) 3 nodes
C) 6 nodes
D) 9 nodes

<details>
<summary>Answer</summary>

**Correct: C)**

In a regional GKE cluster, the `--num-nodes` flag specifies the number of nodes per zone, not total. A regional cluster spans 3 zones by default. So `--num-nodes=2` creates 2 nodes x 3 zones = 6 total nodes. This is a common exam trap. Option A would be correct for a zonal cluster. Options B and D use incorrect calculations.
</details>

---

### Q14. Your team uses Kubernetes for all workloads. You want Google to manage node pools, security patching, and infrastructure, and you want to pay per-pod resource requests rather than per-node. Which GKE mode should you choose?

A) GKE Standard with cluster autoscaler enabled
B) GKE Standard with auto-upgrade and auto-repair enabled
C) GKE Autopilot
D) GKE Standard with Spot VM node pools

<details>
<summary>Answer</summary>

**Correct: C)**

GKE Autopilot is a fully managed mode where Google manages node pools, security, and infrastructure. Billing is per-pod resource requests, not per-node. You do not manage or even SSH into nodes. Options A, B, and D are all GKE Standard mode variations where you manage node pools and pay per-node. Standard is preferred when you need custom node configurations, GPU support, DaemonSets, or privileged containers.
</details>

---

### Q15. You need to create a GKE cluster where nodes have no public IP addresses and the control plane is only accessible from your corporate network (203.0.113.0/24). Which flags should you use?

A) `--enable-private-nodes --enable-ip-alias`
B) `--enable-private-nodes --enable-private-endpoint --master-ipv4-cidr=172.16.0.0/28 --enable-master-authorized-networks --master-authorized-networks=203.0.113.0/24`
C) `--enable-private-endpoint --enable-master-authorized-networks --master-authorized-networks=203.0.113.0/24`
D) `--enable-private-nodes --no-enable-basic-auth --master-authorized-networks=203.0.113.0/24`

<details>
<summary>Answer</summary>

**Correct: B)**

A fully private cluster requires: `--enable-private-nodes` (nodes get internal IPs only), `--enable-private-endpoint` (control plane has no public IP), `--master-ipv4-cidr` (specifies the control plane's internal IP range), and `--enable-master-authorized-networks` with the allowed CIDR to restrict control plane access. Option A misses the private endpoint and authorized networks. Option C misses `--enable-private-nodes`. Option D misses the private endpoint and master CIDR.
</details>

---

### Q16. You deployed a new version of your Cloud Run service and want to send 10% of traffic to the new revision while keeping 90% on the previous stable revision. Which command achieves this?

A) `gcloud run deploy my-service --image=new-image --tag=canary`
B) `gcloud run services update-traffic my-service --to-revisions=my-service-v1=90,my-service-v2=10 --region=us-central1`
C) `gcloud run services update-traffic my-service --to-latest --region=us-central1`
D) `gcloud run services update my-service --concurrency=10 --region=us-central1`

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud Run supports revision-based traffic splitting. The `update-traffic` command with `--to-revisions` lets you specify the exact percentage of traffic each revision receives. Option A creates a tagged revision with a unique URL but does not split production traffic. Option C sends 100% of traffic to the latest revision. Option D sets concurrency (requests per instance), which is unrelated to traffic splitting.
</details>

---

### Q17. You want a Cloud Run service to be triggered whenever a new object is uploaded to a specific Cloud Storage bucket. What is the recommended approach?

A) Configure a Cloud Storage notification to call the Cloud Run service URL directly
B) Create an Eventarc trigger with the event type `google.cloud.storage.object.v1.finalized` targeting the Cloud Run service
C) Set up a Cloud Scheduler job to poll the bucket every minute
D) Create a Pub/Sub notification on the bucket and configure a pull subscription

<details>
<summary>Answer</summary>

**Correct: B)**

Eventarc is the recommended unified eventing layer for routing events to Cloud Run. You create a trigger that filters for `google.cloud.storage.object.v1.finalized` events on the specific bucket, and Eventarc routes the event as an HTTP request to your Cloud Run service. Option A is not how Cloud Storage notifications work -- they go to Pub/Sub topics, not directly to URLs. Option C uses polling, which is inefficient and adds latency. Option D would work but requires you to manage the subscription and a pull-based consumer -- Eventarc is simpler and recommended.
</details>

---

### Q18. You are creating a Cloud SQL PostgreSQL instance for a production application. The application requires automatic failover if the primary zone goes down. Which flag should you set?

A) `--availability-type=ZONAL`
B) `--availability-type=REGIONAL`
C) `--replica-type=FAILOVER`
D) `--enable-failover-replica`

<details>
<summary>Answer</summary>

**Correct: B)**

`--availability-type=REGIONAL` creates a Cloud SQL instance with a standby replica in a different zone within the same region. If the primary zone fails, Cloud SQL automatically fails over to the standby. This provides a 99.95% SLA. Option A (ZONAL) has no standby and no automatic failover. Options C and D are not valid Cloud SQL flags.
</details>

---

### Q19. Your organization has a centralized networking team that manages all VPC networks in a host project. Application teams deploy resources in their own service projects but need to use the shared network. What should you set up?

A) VPC Network Peering between each service project and the host project
B) Shared VPC with the networking project as the host and application projects as service projects
C) Cloud VPN tunnels between each project
D) Cloud Interconnect from each service project to the host project

<details>
<summary>Answer</summary>

**Correct: B)**

Shared VPC allows a host project to share its VPC network with service projects. The networking team manages the network centrally in the host project, while application teams deploy resources (VMs, GKE clusters, etc.) in service projects using subnets from the shared VPC. Option A (VPC Peering) connects separate VPCs but does not provide centralized network management. Options C and D are for connecting to on-premises or external networks, not for inter-project networking.
</details>

---

### Q20. You create two firewall rules on the same VPC. Rule A allows TCP:443 from 0.0.0.0/0 with priority 1000. Rule B denies TCP:443 from 0.0.0.0/0 with priority 900. A user tries to access port 443 on a VM with both rules applied. What happens?

A) Traffic is allowed because allow rules take precedence over deny rules
B) Traffic is denied because Rule B has a lower priority number (higher priority)
C) Traffic is allowed because Rule A was created first
D) Traffic is denied because the default deny-all-ingress rule blocks it

<details>
<summary>Answer</summary>

**Correct: B)**

Firewall rules with a lower priority number have higher priority. Rule B (priority 900, deny) takes precedence over Rule A (priority 1000, allow). When both rules match the same traffic, the rule with the higher priority (lower number) wins. This is a critical concept for the exam. Option A is incorrect -- there is no inherent precedence of allow over deny; it is solely based on priority number. Option C is wrong -- creation order does not matter. Option D is wrong -- the default deny-all-ingress rule has priority 65535, the lowest priority.
</details>

---

### Q21. Your VPC-A is peered with VPC-B, and VPC-B is peered with VPC-C. A VM in VPC-A tries to communicate with a VM in VPC-C. What happens?

A) Traffic flows through VPC-B because it is peered with both VPCs
B) Traffic is blocked because VPC peering is not transitive
C) Traffic flows if the firewall rules in all three VPCs allow it
D) Traffic flows if you add custom routes in VPC-A pointing to VPC-C through VPC-B

<details>
<summary>Answer</summary>

**Correct: B)**

VPC Network Peering is NOT transitive. Even though VPC-A peers with VPC-B and VPC-B peers with VPC-C, VPC-A cannot reach VPC-C through VPC-B. Each peering relationship is independent. To connect VPC-A and VPC-C, you must create a direct peering between them (or use a VPN/Cloud Interconnect). Option A incorrectly assumes transitivity. Option C adds a firewall condition but does not address the fundamental non-transitivity. Option D is also incorrect -- custom routes through a peered VPC do not enable transitive routing.
</details>

---

### Q22. You need to connect your on-premises data center to GCP with 99.99% availability. The connection must use dynamic routing with BGP. Which solution should you use?

A) Classic VPN with static routing
B) Classic VPN with BGP routing
C) HA VPN with two tunnels and Cloud Router
D) Cloud Interconnect (Partner) with a single VLAN attachment

<details>
<summary>Answer</summary>

**Correct: C)**

HA VPN provides 99.99% availability when configured with two tunnels and dynamic (BGP) routing via Cloud Router. This is the recommended VPN setup. Option A (Classic VPN with static routing) only provides 99.9% SLA. Option B (Classic VPN with BGP) also provides 99.9% SLA. Option D (Partner Interconnect with a single VLAN) does not meet the 99.99% SLA requirement -- you need redundant VLAN attachments for that.
</details>

---

### Q23. You have written Terraform configuration files for your infrastructure. You modified the configuration and want to see what changes will be made before applying them. Which command should you run?

A) `terraform validate`
B) `terraform plan`
C) `terraform apply --dry-run`
D) `terraform show`

<details>
<summary>Answer</summary>

**Correct: B)**

`terraform plan` performs a dry run that shows what resources will be created, modified, or destroyed without actually making any changes. This is the standard step before `terraform apply`. Option A (`validate`) only checks syntax and configuration validity, not what will change. Option C is wrong -- there is no `--dry-run` flag for `terraform apply`. Option D (`show`) displays the current state, not planned changes.
</details>

---

### Q24. Your team manages GCP resources using Kubernetes-native workflows. They want to create and manage GCP resources (like Cloud Storage buckets and Cloud SQL instances) using `kubectl apply` and YAML manifests. Which tool should they use?

A) Terraform with the Google Cloud provider
B) Cloud Deployment Manager
C) Config Connector
D) Helm

<details>
<summary>Answer</summary>

**Correct: C)**

Config Connector is a Kubernetes add-on that lets you manage Google Cloud resources through Kubernetes resource definitions (YAML). You use `kubectl apply` to create GCP resources, and Config Connector continuously reconciles the desired state. It runs as a controller inside a GKE cluster. Option A (Terraform) uses HCL, not Kubernetes YAML. Option B (Deployment Manager) uses its own YAML/Jinja2 format, not Kubernetes-native. Option D (Helm) is a Kubernetes package manager for deploying applications to Kubernetes, not for managing GCP infrastructure resources.
</details>

---

### Q25. You need to deploy a Cloud Function that processes files uploaded to a Cloud Storage bucket. The function should use the Python 3.12 runtime. Which command is correct?

A) `gcloud functions deploy process-files --gen2 --runtime=python312 --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" --trigger-event-filters="bucket=my-bucket" --entry-point=process_file --region=us-central1 --source=.`
B) `gcloud functions deploy process-files --runtime=python312 --trigger-bucket=my-bucket --entry-point=process_file`
C) `gcloud run deploy process-files --source=. --trigger-event=google.cloud.storage.object.v1.finalized`
D) `gcloud functions deploy process-files --gen2 --runtime=python312 --trigger-http --entry-point=process_file`

<details>
<summary>Answer</summary>

**Correct: A)**

For Cloud Functions 2nd gen (which should always be used for new deployments), you use `--gen2` with Eventarc-style event filters. The `--trigger-event-filters` flag specifies the event type and bucket. Option B uses 1st gen syntax (`--trigger-bucket`), which is legacy. Option C mixes Cloud Run deployment syntax with trigger events incorrectly. Option D creates an HTTP-triggered function, not a Cloud Storage event-triggered one.
</details>

---

### Q26. Your team needs to run a stateless web application that auto-scales. They want to use containers but have no Kubernetes experience. There are occasional traffic spikes followed by periods of zero traffic. Which compute option is best?

A) GKE Standard with Cluster Autoscaler
B) Compute Engine MIG with autoscaling
C) Cloud Run
D) GKE Autopilot

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud Run is serverless, scales to zero, handles containers, and requires no Kubernetes knowledge. GKE Standard (A) requires cluster management. MIG (B) needs VM and template management. GKE Autopilot (D) is simpler than Standard but still requires Kubernetes knowledge for deployments/services.
</details>

---

### Q27. You need persistent storage for a Compute Engine VM running a database that requires the lowest latency and highest IOPS. Which disk type should you choose?

A) Standard Persistent Disk (pd-standard)
B) Balanced Persistent Disk (pd-balanced)
C) SSD Persistent Disk (pd-ssd)
D) Google Cloud Hyperdisk Extreme

<details>
<summary>Answer</summary>

**Correct: D)**

Hyperdisk Extreme offers the highest IOPS and throughput, configurable independently of disk size. It's designed for the most demanding database workloads. SSD PD (C) is high performance but Hyperdisk Extreme exceeds it. Balanced (B) and Standard (A) are for general-purpose and sequential workloads respectively.
</details>

---

### Q28. What is the key difference between Google Cloud Hyperdisk and standard Persistent Disk?

A) Hyperdisk is only available in us-central1
B) Hyperdisk allows you to independently provision IOPS and throughput separate from disk size
C) Hyperdisk is free for the first 100 GB
D) Hyperdisk only works with GKE, not Compute Engine

<details>
<summary>Answer</summary>

**Correct: B)**

The key innovation of Hyperdisk is **decoupling performance from capacity** -- you can set IOPS and throughput independently of disk size. With standard PD, performance scales with disk size. Hyperdisk is available in multiple regions (A), is not free (C), and works with Compute Engine (D).
</details>

---

### Q29. You need a messaging system for your application that requires support for Apache Kafka APIs because existing applications already use Kafka client libraries. What GCP service should you use?

A) Pub/Sub
B) Google Cloud Managed Service for Apache Kafka
C) Dataflow
D) Memorystore for Redis

<details>
<summary>Answer</summary>

**Correct: B)**

Google Cloud Managed Service for Apache Kafka provides a fully managed Kafka cluster compatible with existing Kafka client libraries and APIs. Pub/Sub (A) is Google's native messaging service but uses different APIs -- existing Kafka apps would need rewriting. Dataflow (C) is stream processing. Memorystore (D) is caching.
</details>

---

### Q30. Your application needs a shared POSIX-compliant file system accessible by multiple Compute Engine VMs simultaneously. Which storage product fits?

A) Cloud Storage
B) Persistent Disk
C) Filestore
D) Cloud SQL

<details>
<summary>Answer</summary>

**Correct: C)**

Filestore is a managed NFS file server providing a POSIX-compliant shared filesystem. Multiple VMs can mount it simultaneously. Cloud Storage (A) is object storage, not a file system. Persistent Disk (B) can only be attached read-write to one VM (except multi-writer for specific use cases). Cloud SQL (D) is a relational database.
</details>

---

### Q31. Your enterprise needs high-performance, fully managed file storage with NFS and SMB protocol support, snapshots, and cloning. Which product should you evaluate?

A) Filestore
B) Google Cloud NetApp Volumes
C) Cloud Storage FUSE
D) Persistent Disk

<details>
<summary>Answer</summary>

**Correct: B)**

Google Cloud NetApp Volumes provides enterprise-grade managed file storage with NFS and SMB support, snapshots, cloning, and replication. Filestore (A) supports NFS but not SMB and has fewer enterprise features. Cloud Storage FUSE (C) mounts Cloud Storage buckets as a filesystem but with performance limitations. Persistent Disk (D) is block storage.
</details>

---

### Q32. You need to maintain multi-region redundancy for your Cloud Storage data. What's the simplest approach?

A) Create buckets in multiple regions and use a script to replicate objects
B) Use a multi-region bucket location (e.g., `US`, `EU`, `ASIA`)
C) Enable cross-region replication through a Cloud Function
D) Use Storage Transfer Service on a daily schedule

<details>
<summary>Answer</summary>

**Correct: B)**

Multi-region buckets automatically replicate data across at least two regions within the selected multi-region. This is built-in, requires no scripts or setup. Custom replication (A, C) adds complexity. Storage Transfer Service (D) is for moving data, not redundancy.
</details>

---

### Q33. You're creating firewall rules for your VPC. The exam guide mentions "Cloud Next Generation Firewall (Cloud NGFW)." What does Cloud NGFW add over traditional VPC firewall rules?

A) It only works with GKE, not Compute Engine
B) It adds hierarchical firewall policies, FQDN objects, geo-based filtering, and threat intelligence integration
C) It replaces IAM for network security
D) It's only available with Premium Network Tier

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud NGFW extends VPC firewall capabilities with hierarchical firewall policies (org/folder level), FQDN-based rules, geographic filtering, threat intelligence (TI) integration, and intrusion detection/prevention. It works with all compute (A), doesn't replace IAM (C), and is available regardless of network tier (D).
</details>

---

### Q34. In Cloud NGFW, what is the difference between network tags and secure Tags for firewall rule targeting?

A) They are the same thing, just renamed
B) Secure Tags are IAM-governed and centrally managed, while network tags are set by anyone who can edit a VM
C) Network tags are more secure than secure Tags
D) Secure Tags only work with Cloud Run, not Compute Engine

<details>
<summary>Answer</summary>

**Correct: B)**

Secure Tags (resource manager Tags) are IAM-controlled -- only users with the right Tag permissions can assign them, providing stronger security. Network tags can be set by anyone with `compute.instances.setTags` permission (essentially anyone who can edit the VM). Secure Tags are the recommended approach for Cloud NGFW.
</details>

---

### Q35. You need to create a Cloud NGFW ingress rule that allows HTTPS traffic from the internet to VMs tagged with `web-server`, but denies all other inbound traffic. Which attributes do you configure?

A) Direction: ingress, Action: allow, Source: 0.0.0.0/0, Target: tag `web-server`, Protocol: TCP, Port: 443
B) Direction: egress, Action: allow, Source: tag `web-server`, Destination: 0.0.0.0/0, Protocol: TCP, Port: 443
C) Direction: ingress, Action: allow, Source: tag `web-server`, Destination: 0.0.0.0/0, Protocol: TCP, Port: 443
D) Direction: ingress, Action: deny, Source: 0.0.0.0/0, Target: tag `web-server`, Protocol: TCP, Port: 443

<details>
<summary>Answer</summary>

**Correct: A)**

For ingress: source is where traffic comes FROM (0.0.0.0/0 = internet), target is where traffic goes TO (tagged VMs). Action is allow, protocol TCP port 443 for HTTPS. Option B is egress (wrong direction). Option C swaps source and target. Option D denies instead of allows.
</details>

---

### Q36. Your organization uses Fabric FAST for GCP infrastructure deployment. What is Fabric FAST?

A) A Google Cloud CLI plugin for faster deployments
B) A Terraform-based framework for quickly bootstrapping a complete GCP organization with best practices
C) A replacement for gcloud commands
D) A CI/CD pipeline tool for Cloud Run

<details>
<summary>Answer</summary>

**Correct: B)**

Fabric FAST (part of Cloud Foundation Fabric) is a Terraform-based framework that provides opinionated, production-ready modules for bootstrapping an entire GCP organization -- including resource hierarchy, networking, IAM, and security. It's not a CLI plugin (A), doesn't replace gcloud (C), and isn't CI/CD (D).
</details>

---

### Q37. You're managing Terraform state for your team. Where should you store the state file for collaboration and reliability?

A) Local filesystem on each developer's machine
B) A Google Cloud Storage bucket with versioning enabled as a remote backend
C) In the Git repository alongside the Terraform code
D) On a shared NFS mount

<details>
<summary>Answer</summary>

**Correct: B)**

GCS backend with versioning provides remote, shared, versioned state with built-in locking (via `lock` file). Local state (A) can't be shared. Git (C) causes conflicts and may expose sensitive data in state files. NFS (D) doesn't provide locking or versioning natively.
</details>

---

### Q38. You run `terraform plan` and see that a resource will be destroyed and recreated (shown as `-/+`). You only want to update it in place. What's causing this?

A) Terraform always destroys and recreates resources
B) A change was made to a "force new" attribute (e.g., changing the boot disk image or machine name) that requires resource replacement
C) The state file is corrupted
D) You need to run `terraform refresh` first

<details>
<summary>Answer</summary>

**Correct: B)**

Certain attributes in Terraform are marked as "ForceNew" -- changing them requires destroying and recreating the resource because the cloud provider doesn't support in-place updates for that attribute. Common examples: changing a VM's name, boot disk image, or network interface configuration.
</details>

---

### Q39. Your team uses Config Connector to manage GCP resources from Kubernetes. What does Config Connector do?

A) Connects multiple GKE clusters together
B) Allows you to manage GCP resources using Kubernetes YAML manifests and kubectl
C) Syncs Kubernetes secrets with Secret Manager
D) Connects GKE to on-premises networks

<details>
<summary>Answer</summary>

**Correct: B)**

Config Connector is a Kubernetes add-on that maps GCP resources to Kubernetes custom resources. You define GCP resources (e.g., Cloud SQL instances, buckets) as YAML and manage them with `kubectl apply/delete`. It doesn't connect clusters (A), sync secrets (C), or handle networking (D).
</details>

---

### Q40. You need to deploy a GKE cluster that meets these requirements: nodes cannot have external IP addresses, and the control plane is only accessible from within the VPC. What type of cluster do you need?

A) GKE Autopilot
B) Regional cluster
C) Private cluster with private endpoint enabled
D) Standard cluster with Cloud NAT

<details>
<summary>Answer</summary>

**Correct: C)**

A private cluster gives nodes internal IPs only. Enabling `--enable-private-endpoint` makes the control plane accessible only via internal IP (within the VPC). Autopilot (A) can be private but isn't by default. Regional (B) is about multi-zone, not privacy. Cloud NAT (D) gives outbound internet but doesn't make the cluster private.
</details>

---

### Q41. You need to deploy a Cloud Run function that processes Cloud Storage object uploads. The function should trigger every time a new file is uploaded to a specific bucket. What should you configure?

A) A Cloud Scheduler job that checks the bucket every minute
B) An Eventarc trigger with the `google.cloud.storage.object.v1.finalized` event type
C) A Pub/Sub subscription that polls the bucket
D) A Cloud Monitoring alert on bucket object count

<details>
<summary>Answer</summary>

**Correct: B)**

Eventarc provides event-driven triggers for Cloud Run and Cloud Run functions. The `google.cloud.storage.object.v1.finalized` event fires when an object is created or overwritten. Polling (A, C) is inefficient. Monitoring alerts (D) notify about metrics, not individual object events.
</details>

---

### Q42. You need to choose a load balancer for a TCP application running on port 8080 that needs to preserve the client's source IP address. Users are all within the same region. Which load balancer?

A) Global External Application LB
B) Regional External Passthrough Network LB
C) External Proxy Network LB
D) Internal Application LB

<details>
<summary>Answer</summary>

**Correct: B)**

Passthrough LBs preserve the original client source IP because packets are passed through without termination. Proxy-based LBs (A, C, D) terminate the connection and create a new one to the backend, replacing the source IP with the proxy's IP. For client IP preservation, passthrough is required.
</details>

---

### Q43. Your application uses Knative serving on a GKE cluster. What does Knative serving provide?

A) A managed Kafka service on GKE
B) A serverless runtime layer on Kubernetes with automatic scaling (including to zero) and revision-based traffic management
C) A replacement for kubectl
D) Managed SSL certificates for GKE Ingress

<details>
<summary>Answer</summary>

**Correct: B)**

Knative serving brings serverless capabilities to Kubernetes -- auto-scaling (including to/from zero), request-based billing model, and revision-based traffic splitting. It's the foundation that Cloud Run is built on. It's not Kafka (A), doesn't replace kubectl (C), and isn't about SSL (D).
</details>

---

### Q44. You need to maintain multi-region redundancy for Cloud SQL. What's the approach?

A) Enable HA (`--availability-type=REGIONAL`) for multi-region
B) Create cross-region read replicas
C) Cloud SQL automatically replicates across regions
D) Use Cloud Spanner instead -- Cloud SQL can't do multi-region

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud SQL's HA option is multi-**zone** within a single region, not multi-region. For multi-region redundancy, you create cross-region read replicas. Cloud SQL doesn't auto-replicate across regions (C). Spanner (D) does multi-region natively, but the question asks about Cloud SQL specifically.
</details>

---

### Q45. You want to differentiate between Premium and Standard Network Service Tiers for your project. Which statement is correct?

A) Standard Tier uses Google's global network, Premium uses the public internet
B) Premium Tier routes traffic over Google's global network, Standard Tier uses the public internet with regional load balancing only
C) Both tiers offer the same performance, only pricing differs
D) Standard Tier supports global load balancing, Premium does not

<details>
<summary>Answer</summary>

**Correct: B)**

Premium Tier routes traffic across Google's private global backbone -- lower latency, supports global LB, higher SLA. Standard Tier routes via public internet -- higher latency, regional LB only, lower cost. They're not the same performance (C), and the description in A is reversed. Standard does NOT support global LB (D).
</details>

---

### Q46. You are responsible for monitoring all changes in your Cloud Storage and Firestore instances. For each change, you need to invoke an action that will verify the compliance of the change in near real time. You want to accomplish this with minimal setup. What should you do?

A) Use the trigger mechanism in each datastore to invoke the security script.
B) Use Cloud Function events, and call the security script from the Cloud Function triggers.
C) Redirect your data-changing queries to an App Engine application, and call the security script from the application.
D) Use a Python script to get logs of the datastores, analyze them, and invoke the security script.

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud Functions natively supports **event-driven triggers** for both Cloud Storage (object finalize, delete, metadata update) and Firestore (document create, update, delete, write). This gives you near real-time invocation with minimal setup -- just deploy a function per trigger. Option A is vague and Cloud Storage doesn't have a built-in "trigger mechanism." Option C requires rearchitecting the application. Option D is polling-based, not real-time, and requires maintaining a script.

> **Exam tip:** Cloud Functions (2nd gen) uses Eventarc under the hood. For "near real-time" + "minimal setup" + event-driven, Cloud Functions is almost always the answer.
</details>

---

### Q47. Your application needs to process a significant rate of transactions. The rate of transactions exceeds the processing capabilities of a single VM. You want to spread transactions across multiple servers in real time and in the most cost-effective manner. What should you do?

A) Send transactions to BigQuery. On the VMs, poll for transactions that do not have the 'processed' key, and mark them 'processed' when done.
B) Set up Cloud SQL with a memory cache for speed. On your multiple servers, poll for transactions that do not have the 'processed' key, and mark them 'processed' when done.
C) Send transactions to Pub/Sub. Process them in VMs in a managed instance group.
D) Record transactions in Cloud Bigtable, and poll for new transactions from the VMs.

<details>
<summary>Answer</summary>

**Correct: C)**

**Pub/Sub + MIG** is the classic pattern for distributing work across multiple consumers. Pub/Sub provides reliable message delivery with at-least-once semantics, decouples producers from consumers, and handles backpressure automatically. The MIG can autoscale based on the Pub/Sub backlog metric (`pubsub.googleapis.com/subscription/num_undelivered_messages`). Options A, B, and D all use **polling**, which is inefficient, adds latency, and wastes resources.

> **Exam tip:** "Distribute work across multiple servers in real time" = **Pub/Sub**. Polling a database is never the right answer for real-time distribution.
</details>

---

### Q48. Your team needs to directly connect your on-premises resources to several virtual machines inside a VPC. You want to provide your team with fast and secure access to the VMs with minimal maintenance and cost. What should you do?

A) Set up Cloud Interconnect.
B) Use Cloud VPN to create a bridge between the VPC and your network.
C) Assign a public IP address to each VM, and assign a strong password to each one.
D) Start a Compute Engine VM, install a software router, and create a direct tunnel to each VM.

<details>
<summary>Answer</summary>

**Correct: B)**

**Cloud VPN** provides encrypted IPsec tunnels between on-premises and GCP at a fraction of the cost of Cloud Interconnect. It's managed by Google (minimal maintenance) and secure by default. Cloud Interconnect (A) provides higher bandwidth but is significantly more expensive and complex to set up -- overkill when the question asks for minimal cost. Public IPs with passwords (C) are insecure. A software router VM (D) is high maintenance.

> **Exam tip:** Cost ranking: **Cloud VPN < Partner Interconnect < Dedicated Interconnect**. Choose VPN when the question emphasizes "minimal cost" or doesn't require high bandwidth (>10 Gbps).
</details>

---

### Q49. Your application is creating a Cloud IoT application requiring data storage of up to 10 PB. The application must support high-speed reads and writes of small pieces of data, but your data schema is simple. You want to use the most economical solution for data storage. What should you do?

A) Store the data in Cloud Spanner, and add an in-memory cache for speed.
B) Store the data in Cloud Storage, and distribute the data through Cloud CDN for speed.
C) Store the data in Cloud Bigtable, and implement the business logic in the programming language of your choice.
D) Use BigQuery, and implement the business logic in SQL.

<details>
<summary>Answer</summary>

**Correct: C)**

**Cloud Bigtable** is purpose-built for this exact use case: IoT data, petabyte scale, high-throughput reads/writes of small records, simple key-value/wide-column schema. It handles millions of operations per second at single-digit millisecond latency. Cloud Spanner (A) is far more expensive and designed for relational/transactional workloads. Cloud Storage (B) is object storage, not suitable for high-speed small random reads/writes. BigQuery (D) is an analytics warehouse optimized for large scans, not high-speed point writes.

> **Exam tip:** IoT + high throughput + simple schema + petabyte scale = **Cloud Bigtable**. Every time.
</details>

---

### Q50. You have created a Kubernetes deployment on GKE that has a backend service. You also have pods that run the frontend service. You want to ensure that there is no interruption in communication between your frontend and backend service pods if they are moved or restarted. What should you do?

A) Create a service that groups your pods in the backend service, and tell your frontend pods to communicate through that service.
B) Create a DNS entry with a fixed IP address that the frontend service can use to reach the backend service.
C) Assign static internal IP addresses that the frontend service can use to reach the backend pods.
D) Assign static external IP addresses that the frontend service can use to reach the backend pods.

<details>
<summary>Answer</summary>

**Correct: A)**

A **Kubernetes Service** (e.g., `ClusterIP`) provides a stable virtual IP and DNS name that automatically routes traffic to healthy backend pods via label selectors. When pods restart or get rescheduled, the Service updates its endpoints automatically -- zero interruption. Static IPs (C, D) don't work because pods get new IPs on restart and you can't assign static IPs to pods. A manual DNS entry (B) wouldn't update automatically when pods move.

> **Exam tip:** In Kubernetes, **Services** are the answer to stable communication between pods. `ClusterIP` for internal, `LoadBalancer` or `Ingress` for external.
</details>

---

### Q51. You are responsible for the user-management service for your global company. The service will add, update, delete, and list addresses. Each of these operations is implemented by a Docker container microservice. The processing load can vary from low to very high. You want to deploy the service on Google Cloud for scalability and minimal administration. What should you do?

A) Deploy your Docker containers into Cloud Run.
B) Start each Docker container as a managed instance group.
C) Deploy your Docker containers into Google Kubernetes Engine.
D) Combine the four microservices into one Docker image, and deploy it to the App Engine instance.

<details>
<summary>Answer</summary>

**Correct: A)**

**Cloud Run** is serverless, runs Docker containers directly, auto-scales from zero to thousands of instances, and requires zero cluster management. It's the lowest-admin option for containerized microservices with variable load. GKE (C) works but requires managing a cluster -- more admin overhead. MIGs (B) require managing VMs. Combining microservices into one image (D) defeats the purpose of microservices architecture and App Engine isn't the right fit for Docker containers.

> **Exam tip:** "Docker containers" + "minimal administration" + "variable load" = **Cloud Run**. If the question adds "complex orchestration" or "stateful workloads," that's when GKE becomes the answer.
</details>

---

### Q52. You provide a service that you need to open to everyone in your partner network. You have a server and an IP address where the application is located. You do not want to have to change the IP address on your DNS server if your server crashes or is replaced. You also want to avoid downtime and deliver a solution for minimal cost and setup. What should you do?

A) Create a script that updates the IP address for the domain when the server crashes or is replaced.
B) Reserve a static internal IP address, and assign it using Cloud DNS.
C) Reserve a static external IP address, and assign it using Cloud DNS.
D) Use the Bring Your Own IP (BYOIP) method to use your own IP address.

<details>
<summary>Answer</summary>

**Correct: C)**

A **reserved static external IP** persists even when the VM is deleted and recreated. You assign it to your replacement VM and the DNS record (in Cloud DNS) stays the same. Partners always reach the same IP. Internal IP (B) won't work because partners are external. A script (A) introduces downtime and complexity. BYOIP (D) is for bringing existing IP ranges to Google Cloud -- overcomplicated for this scenario.

> **Exam tip:** "Don't want to change IP if server is replaced" = **reserve a static external IP**. Static IPs can be reassigned to new VMs.
</details>

---

### Q53. Your team is building the development, test, and production environments for your project deployment in Google Cloud. You need to efficiently deploy and manage these environments and ensure that they are consistent. You want to follow Google-recommended practices. What should you do?

A) Create a Cloud Shell script that uses gcloud commands to deploy the environments.
B) Create one Terraform configuration for all environments. Parameterize the differences between environments.
C) For each environment, create a Terraform configuration. Use them for repeated deployment. Reconcile the templates periodically.
D) Use the Cloud Foundation Toolkit to create one deployment template that will work for all environments, and deploy with Terraform.

<details>
<summary>Answer</summary>

**Correct: D)**

The **Cloud Foundation Toolkit (CFT)** provides Google-opinionated, production-ready Terraform modules that follow Google's best practices. Using CFT with one template and deploying with Terraform ensures consistency across environments and follows Google-recommended practices. Option B is good IaC practice but doesn't leverage Google's best-practice templates. Option C creates drift risk with separate configs. Option A uses imperative scripting rather than declarative IaC.

> **Exam tip:** "Google-recommended practices" + infrastructure deployment = **Cloud Foundation Toolkit + Terraform**. CFT is Google's answer to "how should I set up my GCP environments."
</details>

---

### Q54. You are running several related applications on Compute Engine VM instances. You want to follow Google-recommended practices and expose each application through a DNS name. What should you do?

A) Use the Compute Engine internal DNS service to assign DNS names to your VM instances, and make the names known to your users.
B) Assign each VM instance an alias IP address range, and then make the internal DNS names public.
C) Assign Google Cloud routes to your VM instances, assign DNS names to the routes, and make the DNS names public.
D) Use Cloud DNS to translate your domain names into your IP addresses.

<details>
<summary>Answer</summary>

**Correct: D)**

**Cloud DNS** is Google's managed, authoritative DNS service designed for publishing DNS records. You create a managed zone, add A records mapping your domain names to your VM IPs, and it handles global DNS resolution. Internal DNS (A) is auto-generated and intended for within-VPC communication, not for exposing apps externally. Alias IPs (B) are for assigning multiple IPs to a single NIC, not for DNS. Routes (C) are for network routing, not DNS.

> **Exam tip:** "Expose through DNS" + "Google-recommended" = **Cloud DNS**. Internal DNS (`*.internal`) is for VPC-internal resolution only.
</details>

---

### Q55. You are creating an environment for researchers to run ad hoc SQL queries. The researchers work with large quantities of data. Although they will use the environment for an hour a day on average, the researchers need access to the functional environment at any time during the day. You need to deliver a cost-effective solution. What should you do?

A) Store the data in Cloud Bigtable, and run SQL queries provided by Bigtable schema.
B) Store the data in BigQuery, and run SQL queries in BigQuery.
C) Create a Dataproc cluster, store the data in HDFS storage, and run SQL queries in Spark.
D) Create a Dataproc cluster, store the data in Cloud Storage, and run SQL queries in Spark.

<details>
<summary>Answer</summary>

**Correct: B)**

**BigQuery** is serverless, always available, and uses on-demand pricing (pay per query, not per hour). If researchers only query for ~1 hour/day, you only pay for those queries. With Dataproc (C, D), you'd either pay for an idle cluster 23 hours/day or deal with startup delays when spinning up on demand. Bigtable (A) doesn't support SQL natively. BigQuery is purpose-built for ad hoc SQL analytics on large datasets.

> **Exam tip:** "Ad hoc SQL" + "large data" + "cost-effective" + "intermittent usage" = **BigQuery**. Its serverless, pay-per-query model beats always-on clusters for sporadic workloads.
</details>

---

### Q56. You are migrating your workload from on-premises deployment to Google Kubernetes Engine (GKE). You want to minimize costs and stay within budget. What should you do?

A) Configure Autopilot in GKE to monitor node utilization and eliminate idle nodes.
B) Configure the needed capacity; the sustained use discount will make you stay within budget.
C) Scale individual nodes up and down with the Horizontal Pod Autoscaler.
D) Create several nodes using Compute Engine, add them to a managed instance group, and set the group to scale up and down depending on load.

<details>
<summary>Answer</summary>

**Correct: A)**

**GKE Autopilot** is Google's fully managed Kubernetes mode where you pay per pod resource request, not per node. Google manages the nodes, optimizes bin-packing, and eliminates idle node costs. This directly minimizes costs. Option B assumes sustained use discounts alone will keep you in budget -- passive, not active cost management. HPA (C) scales pods, not nodes -- it won't reduce infrastructure costs if nodes remain idle. Option D bypasses GKE entirely and manually recreates orchestration.

> **Exam tip:** GKE Autopilot = pay per pod, Google manages nodes. GKE Standard = you manage nodes, pay per node. For cost optimization questions, Autopilot is usually the answer.
</details>

---

### Q57. Your application allows users to upload pictures. You need to convert each picture to your internal optimized binary format and store it. You want to use the most efficient, cost-effective solution. What should you do?

A) Store uploaded files in Cloud Bigtable, monitor Bigtable entries, and then run a Cloud Function to convert the files and store them in Bigtable.
B) Store uploaded files in Firestore, monitor Firestore entries, and then run a Cloud Function to convert the files and store them in Firestore.
C) Store uploaded files in Filestore, monitor Filestore entries, and then run a Cloud Function to convert the files and store them in Filestore.
D) Save uploaded files in a Cloud Storage bucket, and monitor the bucket for uploads. Run a Cloud Function to convert the files and to store them in a Cloud Storage bucket.

<details>
<summary>Answer</summary>

**Correct: D)**

**Cloud Storage** is designed for storing files (objects) like images, and Cloud Functions has a native event trigger for Cloud Storage (`google.storage.object.finalize`). This is the standard event-driven file processing pattern: upload triggers function, function processes file, function writes result back. Bigtable (A) and Firestore (B) are databases, not file storage. Filestore (C) is an NFS file server -- overkill and more expensive for this use case.

> **Exam tip:** File upload + processing = **Cloud Storage + Cloud Function trigger**. This is one of the most classic GCP patterns.
</details>

---

### Q58. You are migrating your on-premises solution to Google Cloud. As a first step, the new cloud solution will need to ingest 100 TB of data. Your daily uploads will be within your current bandwidth limit of 100 Mbps. You want to follow Google-recommended practices for the most cost-effective way to implement the migration. What should you do?

A) Set up Partner Interconnect for the duration of the first upload.
B) Obtain a Transfer Appliance, copy the data to it, and ship it to Google.
C) Set up Dedicated Interconnect for the duration of your first upload, and then drop back to regular bandwidth.
D) Divide your data between 100 computers, and upload each data portion to a bucket. Then run a script to merge the uploads together.

<details>
<summary>Answer</summary>

**Correct: B)**

At 100 Mbps, uploading 100 TB would take approximately **93 days** (100 TB = 800,000 Gb / 100 Mbps = 8,000,000 seconds). A **Transfer Appliance** is a physical device Google ships to you -- you load your data and ship it back. It's designed for exactly this scenario: large one-time transfers where network bandwidth is insufficient. Interconnect options (A, C) are expensive recurring commitments for a one-time transfer. Option D doesn't increase total bandwidth.

> **Exam tip:** Use this rule of thumb: if transferring over the network would take **weeks or more**, use a **Transfer Appliance**. For ongoing high-bandwidth needs, use Interconnect.
</details>

---

## Answer Key

| Q | Answer | Source | Topic |
|---|--------|--------|-------|
| 1 | C | 07-Q13 | Compute selection -- Cloud Run |
| 2 | B | 07-Q14 | Spot VMs for batch processing |
| 3 | B | 07-Q15 | Custom machine types |
| 4 | B | 07-Q16 | Cloud Spanner multi-region |
| 5 | C | 07-Q17 | Bigtable for IoT |
| 6 | D | 07-Q18 | Archive storage class |
| 7 | C | 07-Q19 | Global HTTPS LB + Premium Tier |
| 8 | B | 07-Q20 | Standard Network Tier |
| 9 | B | 07-Q21 | Firestore Native mode |
| 10 | C | 07-Q22 | UDP -- Passthrough NLB |
| 11 | B | 07-Q23 | Disk auto-delete flag |
| 12 | B | 07-Q24 | OS Login + osAdminLogin |
| 13 | C | 07-Q25 | Regional GKE node count |
| 14 | C | 07-Q26 | GKE Autopilot |
| 15 | B | 07-Q27 | Private GKE cluster flags |
| 16 | B | 07-Q28 | Cloud Run traffic splitting |
| 17 | B | 07-Q29 | Eventarc + Cloud Run |
| 18 | B | 07-Q30 | Cloud SQL regional HA |
| 19 | B | 07-Q31 | Shared VPC |
| 20 | B | 07-Q32 | Firewall rule priority |
| 21 | B | 07-Q33 | VPC peering non-transitive |
| 22 | C | 07-Q34 | HA VPN 99.99% |
| 23 | B | 07-Q35 | Terraform plan |
| 24 | C | 07-Q36 | Config Connector |
| 25 | A | 07-Q37 | Cloud Functions 2nd gen deploy |
| 26 | C | 11-Q11 | Cloud Run for containers |
| 27 | D | 11-Q12 | Hyperdisk Extreme |
| 28 | B | 11-Q13 | Hyperdisk vs PD |
| 29 | B | 11-Q14 | Managed Kafka |
| 30 | C | 11-Q15 | Filestore shared filesystem |
| 31 | B | 11-Q16 | NetApp Volumes |
| 32 | B | 11-Q17 | Multi-region Cloud Storage |
| 33 | B | 11-Q18 | Cloud NGFW features |
| 34 | B | 11-Q19 | Secure Tags vs network tags |
| 35 | A | 11-Q20 | Cloud NGFW ingress rule |
| 36 | B | 11-Q21 | Fabric FAST |
| 37 | B | 11-Q22 | Terraform state GCS backend |
| 38 | B | 11-Q23 | Terraform force new |
| 39 | B | 11-Q24 | Config Connector |
| 40 | C | 11-Q25 | Private GKE cluster |
| 41 | B | 11-Q26 | Eventarc + Cloud Storage |
| 42 | B | 11-Q27 | Passthrough LB client IP |
| 43 | B | 11-Q28 | Knative serving |
| 44 | B | 11-Q29 | Cloud SQL cross-region replicas |
| 45 | B | 11-Q30 | Network Service Tiers |
| 46 | B | 08-Q3 | Cloud Functions event triggers |
| 47 | C | 08-Q4 | Pub/Sub + MIG distribution |
| 48 | B | 08-Q5 | Cloud VPN minimal cost |
| 49 | C | 08-Q7 | Bigtable for IoT 10 PB |
| 50 | A | 08-Q8 | GKE Kubernetes Service |
| 51 | A | 08-Q9 | Docker microservices Cloud Run |
| 52 | C | 08-Q10 | Static external IP + Cloud DNS |
| 53 | D | 08-Q11 | Cloud Foundation Toolkit + Terraform |
| 54 | D | 08-Q13 | Cloud DNS for applications |
| 55 | B | 08-Q15 | BigQuery ad hoc SQL |
| 56 | A | 08-Q16 | GKE Autopilot cost optimization |
| 57 | D | 08-Q17 | Cloud Storage + Cloud Function |
| 58 | B | 08-Q18 | Transfer Appliance 100 TB |
