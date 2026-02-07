# GCP Associate Cloud Engineer -- Practice Questions (60 Questions)

> **Exam simulation:** 60 scenario-based questions weighted by domain importance. Aim for 85%+ before taking the real exam. Answers and explanations are hidden behind expandable sections.

| Domain | Questions | Weight |
|--------|-----------|--------|
| 1 -- Setting Up a Cloud Solution Environment | Q1--Q12 | ~20% |
| 2 -- Planning and Configuring a Cloud Solution | Q13--Q22 | ~17% |
| 3 -- Deploying and Implementing a Cloud Solution | Q23--Q37 | ~25% |
| 4 -- Ensuring Successful Operation | Q38--Q49 | ~20% |
| 5 -- Configuring Access and Security | Q50--Q60 | ~18% |

---

## Domain 1: Setting Up a Cloud Solution Environment

### Q1. Your company has just acquired a Google Workspace subscription. The CTO wants to organize GCP projects by department (Engineering, Marketing, Finance) and by environment (dev, staging, prod) within each department. What should you create first under the Organization node?

A) One project per department with labels for environments
B) Folders for each department, with nested folders for each environment
C) Separate billing accounts for each department
D) Separate VPCs for each department

<details>
<summary>Answer</summary>

**Correct: B)**

Folders are the right way to organize resources hierarchically under an Organization node. You can nest folders (up to 10 levels deep), so you would create department folders at the top level and environment folders (dev, staging, prod) nested inside each. This allows you to apply different IAM policies and org policies at each level. Option A uses labels, which help with billing attribution but do not provide IAM or policy inheritance. Option C is about billing, not resource organization. Option D is about networking, not hierarchy.
</details>

---

### Q2. A security team member needs to set an organization policy that prevents any VM in the organization from having an external IP address. Which role should they have?

A) `roles/resourcemanager.organizationAdmin`
B) `roles/compute.networkAdmin`
C) `roles/orgpolicy.policyAdmin`
D) `roles/iam.securityAdmin`

<details>
<summary>Answer</summary>

**Correct: C)**

The `roles/orgpolicy.policyAdmin` role is specifically required to create and manage organization policies. The constraint they would set is `constraints/compute.vmExternalIpAccess`. Option A (Organization Admin) manages the org structure and IAM at the org level but does not inherently include org policy management. Option B (Compute Network Admin) manages networking resources, not org policies. Option D (Security Admin) manages IAM policies, not org policy constraints.
</details>

---

### Q3. You need to link a newly created project to a billing account. You have `roles/billing.user` on the billing account. What additional role do you need on the project to complete the linking?

A) `roles/viewer`
B) `roles/editor`
C) `roles/billing.projectManager` or `roles/owner`
D) `roles/billing.admin`

<details>
<summary>Answer</summary>

**Correct: C)**

Linking a project to a billing account requires roles on both the billing account AND the project. On the billing account side, you need `roles/billing.user` (which you have). On the project side, you need either `roles/owner` or `roles/billing.projectManager`. The latter is the least-privilege option for this specific task. Option A (Viewer) has no billing permissions. Option B (Editor) cannot modify billing settings. Option D (Billing Admin) is on the billing account, not the project, and you already have `roles/billing.user` on the billing account.
</details>

---

### Q4. Your finance team wants to be alerted when a project's monthly spend reaches 80% and 100% of its $5,000 budget. They also want spending to automatically stop when the budget is exceeded. What should you configure?

A) A budget with threshold rules at 80% and 100%, which will automatically stop all resources
B) A budget with threshold rules at 80% and 100%, with notifications sent to a Pub/Sub topic that triggers a Cloud Function to disable billing on the project
C) A quota limit of $5,000 on the project
D) A budget with threshold rules at 80% and 100%, with email notifications only

<details>
<summary>Answer</summary>

**Correct: B)**

Budget alerts do NOT stop spending by default -- they only send notifications. To automatically stop spending, you must configure a Pub/Sub topic connected to a Cloud Function that programmatically disables billing on the project. Option A is incorrect because budgets never automatically stop resources. Option C is incorrect because quotas limit resource creation (e.g., number of CPUs), not dollar spending. Option D would alert the team but would not stop spending automatically as required.
</details>

---

### Q5. Your organization wants to analyze cloud costs by querying detailed billing data with SQL. What should you configure?

A) Export billing data to Cloud Storage as CSV and query with Dataflow
B) Enable billing export to BigQuery from the Billing section of the Cloud Console
C) Use `gcloud billing export create` to export to BigQuery
D) Enable billing export to Cloud Storage and connect Looker Studio directly to the CSV files

<details>
<summary>Answer</summary>

**Correct: B)**

Billing export to BigQuery is the recommended approach for analyzing billing data with SQL. It is configured in the Cloud Console under Billing > Billing export (there is no gcloud command for configuring this). Once enabled, you can run SQL queries directly in BigQuery. Option A uses Cloud Storage and Dataflow, which is unnecessarily complex. Option C is wrong because there is no such gcloud command. Option D uses Cloud Storage CSV export, which is a legacy approach and does not support direct SQL queries.
</details>

---

### Q6. A teammate tries to create a Compute Engine VM but receives the error: "Compute Engine API has not been used in project X before or it is disabled." What is the most likely cause and fix?

A) The project's billing account has been disabled; re-enable billing
B) The Compute Engine API is not enabled in the project; run `gcloud services enable compute.googleapis.com`
C) The user does not have the `roles/compute.instanceAdmin` role; grant the role
D) The project has exceeded its Compute Engine quota; request a quota increase

<details>
<summary>Answer</summary>

**Correct: B)**

The error message explicitly states the API has not been used or is disabled. Every GCP service requires its API to be enabled in the project before use. The fix is `gcloud services enable compute.googleapis.com`. Option A would produce a different error about billing. Option C would produce a permission denied error, not an API disabled error. Option D would produce a quota exceeded error.
</details>

---

### Q7. Your company uses Active Directory on-premises for identity management. You need to synchronize user identities to Google Cloud so developers can log in with their corporate credentials. What should you use?

A) Cloud IAM federation with Active Directory
B) Google Cloud Directory Sync (GCDS) for identities and SAML federation for authentication
C) Google Cloud Directory Sync (GCDS) for both identities and passwords
D) Import Active Directory users directly into Cloud Console

<details>
<summary>Answer</summary>

**Correct: B)**

GCDS performs a one-way sync from Active Directory (or LDAP) to Cloud Identity, synchronizing user and group identities. However, GCDS does NOT sync passwords. For authentication (login), you need a separate SAML-based federation (using AD FS or another IdP). Option A is too vague and does not describe the correct pattern. Option C is wrong because GCDS never syncs passwords. Option D is not a real feature -- you cannot import AD users directly into the Cloud Console.
</details>

---

### Q8. You set up a metrics scope in Cloud Monitoring for your scoping project `monitoring-hub`. You want to view metrics from three other projects: `project-a`, `project-b`, and `project-c`. What should you do?

A) Create a separate Cloud Monitoring workspace for each project
B) Export metrics from all projects to BigQuery and query them there
C) Add `project-a`, `project-b`, and `project-c` as monitored projects in the `monitoring-hub` metrics scope
D) Install the Ops Agent in each project and point it to the `monitoring-hub` project

<details>
<summary>Answer</summary>

**Correct: C)**

A metrics scope defines which projects are monitored together. One project acts as the scoping project (host), and you add other projects as monitored projects. This is done in the Cloud Console under Monitoring > Settings. A single scoping project can monitor up to 375 other projects. Option A creates separate, disconnected monitoring views. Option B would work for analysis but is not the standard approach for centralized monitoring. Option D is about collecting VM-level metrics, not about cross-project monitoring configuration.
</details>

---

### Q9. What is the retention period and cost model for Admin Activity audit logs in GCP?

A) 30-day retention, must be explicitly enabled, charged per GB ingested
B) 400-day retention, always enabled, free of charge
C) 400-day retention, must be explicitly enabled, free of charge
D) 90-day retention, always enabled, charged per GB ingested

<details>
<summary>Answer</summary>

**Correct: B)**

Admin Activity audit logs are always enabled (you cannot disable them), retained for 400 days in the `_Required` log bucket, and are free of charge. They record configuration changes like creating, deleting, or updating resources. This is distinct from Data Access audit logs, which must be explicitly enabled (except for BigQuery), have a 30-day default retention in the `_Default` bucket, and cost money based on volume.
</details>

---

### Q10. You try to create 10 VMs in `us-east1` but only 6 are created successfully. The remaining 4 fail with a quota error. What should you do?

A) Switch to a different machine type that uses fewer resources
B) Check the CPU quota for `us-east1` in the project and request a quota increase
C) Create the remaining VMs in a different region
D) Create a new project and distribute VMs across both projects

<details>
<summary>Answer</summary>

**Correct: B)**

Quotas are enforced per project and per region. The most likely cause is that the CPU quota in `us-east1` has been exceeded. You should check the quota under IAM & Admin > Quotas, filter for the relevant service and region, and request an increase. Option A might work as a workaround but does not address the underlying quota limitation. Options C and D are workarounds that avoid the actual issue. Quota increases are the standard approach and are typically approved within 24-48 hours.
</details>

---

### Q11. An organization wants to ensure that no GCP resources can be created outside of the EU. What is the most effective way to enforce this?

A) Create a custom IAM role that only allows resource creation in EU regions
B) Use VPC firewall rules to block traffic from non-EU regions
C) Apply the `constraints/gcp.resourceLocations` organization policy constraint at the organization level
D) Train developers to only select EU regions when creating resources

<details>
<summary>Answer</summary>

**Correct: C)**

The `constraints/gcp.resourceLocations` organization policy constraint restricts where resources can be created. Setting it at the organization level with only EU locations allowed ensures enforcement across all folders, projects, and resources. Option A would be extremely complex and brittle -- IAM controls who can do things, not where. Option B is about network traffic, not resource creation locations. Option D relies on human behavior and cannot be enforced.
</details>

---

### Q12. A billing administrator needs to understand which team is responsible for a spike in Cloud Storage costs. The organization uses labels on their GCS buckets. Where should they look?

A) Cloud Monitoring dashboards
B) Billing export data in BigQuery, filtering by labels
C) The Cloud Storage usage section in the Console
D) Cloud Audit Logs

<details>
<summary>Answer</summary>

**Correct: B)**

Labels applied to resources appear in billing export data. By querying the BigQuery billing export and filtering or grouping by labels, the billing admin can attribute costs to specific teams. Option A shows metrics but not billing cost breakdowns by label. Option C shows storage usage but does not break down costs by label. Option D shows who performed actions, not cost attribution.
</details>

---

## Domain 2: Planning and Configuring a Cloud Solution

### Q13. A startup wants to run a containerized web API. They want zero infrastructure management, the ability to scale to zero during off-hours to save costs, and they need to handle multiple concurrent requests per instance. Which service should they use?

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

### Q14. A company needs to run a batch data processing job that processes 500 GB of data weekly. The job is fault-tolerant and can be restarted if interrupted. They want to minimize cost. What compute option should they use?

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

### Q15. Your application requires exactly 6 vCPUs and 20 GB of RAM. The closest predefined machine type offers 8 vCPUs and 32 GB RAM. How can you avoid overprovisioning?

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

### Q16. A financial services company needs a relational database that provides strong consistency, horizontal scalability, and 99.999% availability across multiple regions. Which database service should they choose?

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

### Q17. You need to store 50 TB of IoT sensor data that arrives at millions of writes per second. The data will be queried by row key for real-time dashboards. Which storage option is best?

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

### Q18. A company stores compliance archives that must be retained for 7 years and are accessed less than once per year. They want to minimize storage cost. Which Cloud Storage class should they use?

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

### Q19. Your web application serves users globally and needs an HTTPS load balancer with DDoS protection via Cloud Armor. It also needs Cloud CDN for static assets. Which load balancer and network tier should you use?

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

### Q20. Your application serves users in a single region and cost optimization is the top priority. Latency is not critical. Which Network Service Tier should you use?

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

### Q21. A mobile application needs a database with real-time synchronization, offline support, and automatic scaling. The data model is document-oriented. Which database should you choose?

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

### Q22. You need to expose a UDP-based game server running on Compute Engine to external players. Which load balancer should you use?

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

## Domain 3: Deploying and Implementing a Cloud Solution

### Q23. You are creating a Compute Engine VM that will host a database. The data disk must survive if the VM is accidentally deleted. Which flag should you use when creating the additional data disk?

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

### Q24. You want to enable OS Login on all VMs in your project so that SSH access is managed through IAM. A developer needs SSH access with sudo privileges. What should you do?

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

### Q25. You create a regional GKE cluster with `--region=us-central1 --num-nodes=2`. How many total nodes will be created?

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

### Q26. Your team uses Kubernetes for all workloads. You want Google to manage node pools, security patching, and infrastructure, and you want to pay per-pod resource requests rather than per-node. Which GKE mode should you choose?

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

### Q27. You need to create a GKE cluster where nodes have no public IP addresses and the control plane is only accessible from your corporate network (203.0.113.0/24). Which flags should you use?

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

### Q28. You deployed a new version of your Cloud Run service and want to send 10% of traffic to the new revision while keeping 90% on the previous stable revision. Which command achieves this?

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

### Q29. You want a Cloud Run service to be triggered whenever a new object is uploaded to a specific Cloud Storage bucket. What is the recommended approach?

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

### Q30. You are creating a Cloud SQL PostgreSQL instance for a production application. The application requires automatic failover if the primary zone goes down. Which flag should you set?

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

### Q31. Your organization has a centralized networking team that manages all VPC networks in a host project. Application teams deploy resources in their own service projects but need to use the shared network. What should you set up?

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

### Q32. You create two firewall rules on the same VPC. Rule A allows TCP:443 from 0.0.0.0/0 with priority 1000. Rule B denies TCP:443 from 0.0.0.0/0 with priority 900. A user tries to access port 443 on a VM with both rules applied. What happens?

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

### Q33. Your VPC-A is peered with VPC-B, and VPC-B is peered with VPC-C. A VM in VPC-A tries to communicate with a VM in VPC-C. What happens?

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

### Q34. You need to connect your on-premises data center to GCP with 99.99% availability. The connection must use dynamic routing with BGP. Which solution should you use?

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

### Q35. You have written Terraform configuration files for your infrastructure. You modified the configuration and want to see what changes will be made before applying them. Which command should you run?

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

### Q36. Your team manages GCP resources using Kubernetes-native workflows. They want to create and manage GCP resources (like Cloud Storage buckets and Cloud SQL instances) using `kubectl apply` and YAML manifests. Which tool should they use?

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

### Q37. You need to deploy a Cloud Function that processes files uploaded to a Cloud Storage bucket. The function should use the Python 3.12 runtime. Which command is correct?

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

## Domain 4: Ensuring Successful Operation of a Cloud Solution

### Q38. You take a snapshot of a 500 GB persistent disk. You then make 10 GB of changes and take another snapshot. How much new data does the second snapshot store?

A) 500 GB (each snapshot is a full copy)
B) 510 GB (the original plus changes)
C) Approximately 10 GB (only the changed blocks)
D) 250 GB (snapshots compress data by 50%)

<details>
<summary>Answer</summary>

**Correct: C)**

Snapshots in GCP are incremental. Only the first snapshot is a full copy of the disk. Subsequent snapshots store only the blocks that have changed since the previous snapshot. So the second snapshot stores approximately 10 GB of changed blocks. Importantly, deleting an earlier snapshot is safe -- GCP automatically moves any data needed by later snapshots. Option A is incorrect because only the first snapshot is full. Options B and D use incorrect calculations.
</details>

---

### Q39. You want to create a reusable boot image for your application so that new VMs always start with the same pre-configured software. You have a running VM with the software installed. What is the recommended approach?

A) Create a snapshot of the boot disk and use the snapshot to create new VMs
B) Stop the VM, create a custom image from its boot disk, and use the image in instance templates
C) Export the VM's disk to Cloud Storage and import it for each new VM
D) Write a startup script that installs all software on every new VM

<details>
<summary>Answer</summary>

**Correct: B)**

The best practice is to stop the VM (for file system consistency), create a custom image from its boot disk, and use that image in instance templates for creating new VMs. Images are designed for creating boot disks and can be organized into image families (where the latest non-deprecated image is used automatically). Option A (snapshots) is primarily for backup/restore, not for creating reusable boot images. Option C is overly complex. Option D is slow and error-prone compared to using a pre-built image.
</details>

---

### Q40. Your GKE cluster needs a new node pool with GPU-enabled nodes for machine learning workloads, while the existing node pool continues to serve the web application. What should you do?

A) Delete the existing cluster and create a new one with GPU nodes
B) Modify the existing node pool to change the machine type to one with GPUs
C) Create a new node pool with GPU-enabled machine types in the same cluster
D) Create a separate GKE cluster for the ML workloads

<details>
<summary>Answer</summary>

**Correct: C)**

A GKE cluster can have multiple node pools with different machine types. You should create a new node pool with GPU-enabled nodes (e.g., `n1-standard-4` with `--accelerator=type=nvidia-tesla-t4,count=1`) and use node selectors or taints/tolerations to schedule ML workloads on GPU nodes. Option A destroys the existing workloads unnecessarily. Option B is not possible -- you cannot change a node pool's machine type after creation. Option D adds unnecessary overhead and management complexity.
</details>

---

### Q41. Your GKE deployment experiences variable traffic. During peak hours, pods hit 90% CPU utilization. During off-hours, most pods are idle. You want to automatically adjust the number of pod replicas based on CPU usage. What should you configure?

A) Vertical Pod Autoscaler (VPA)
B) Horizontal Pod Autoscaler (HPA)
C) Cluster Autoscaler
D) GKE Autopilot

<details>
<summary>Answer</summary>

**Correct: B)**

The Horizontal Pod Autoscaler (HPA) automatically scales the number of pod replicas based on observed CPU utilization (or custom metrics). It adds more pods during peak traffic and removes them during off-hours. Option A (VPA) adjusts CPU and memory resource requests per pod (makes pods bigger), not the number of replicas. Option C (Cluster Autoscaler) scales the number of nodes, not pods. Option D (Autopilot) is a cluster mode, not an autoscaling mechanism for pods. Note: HPA and Cluster Autoscaler often work together -- HPA creates more pods, and if nodes are full, the Cluster Autoscaler adds more nodes.
</details>

---

### Q42. You deployed a Cloud Run service revision `v2` that has a bug. You need to immediately route all traffic back to the stable `v1` revision. What command should you run?

A) `gcloud run deploy my-service --image=v1-image --region=us-central1`
B) `gcloud run services update-traffic my-service --to-revisions=v1=100 --region=us-central1`
C) `gcloud run revisions delete v2 --region=us-central1`
D) `gcloud run services rollback my-service --region=us-central1`

<details>
<summary>Answer</summary>

**Correct: B)**

Using `update-traffic` to send 100% of traffic to the stable `v1` revision is the fastest way to roll back. This is an instant traffic shift with no deployment needed. Option A deploys a new revision from the old image, which takes time and creates a new revision instead of simply switching traffic. Option C deletes the revision entirely, which is not needed for a rollback and could lose the ability to inspect the buggy revision. Option D is not a valid gcloud command -- there is no `rollback` subcommand for Cloud Run services.
</details>

---

### Q43. You have a Cloud Storage bucket with objects that should transition from Standard to Nearline after 30 days, then to Coldline after 90 days, and be deleted after 2 years. How should you configure this?

A) Manually move objects between storage classes on a schedule using a cron job
B) Set up Object Lifecycle Management rules with `SetStorageClass` and `Delete` actions based on age conditions
C) Create three separate buckets with different storage classes and move objects between them
D) Use the Autoclass feature and set a deletion policy

<details>
<summary>Answer</summary>

**Correct: B)**

Object Lifecycle Management rules automate transitions and deletions based on conditions like object age. You configure rules with `SetStorageClass` actions (Standard to Nearline at 30 days, Nearline to Coldline at 90 days) and a `Delete` action at 730 days. Transitions can only go "colder" (Standard > Nearline > Coldline > Archive). Option A is manual and error-prone. Option C is unnecessarily complex. Option D -- Autoclass handles transitions automatically based on access patterns but does not support scheduled deletion, and the transitions would not follow the exact schedule specified.
</details>

---

### Q44. You have a subnet with CIDR range 10.0.1.0/24 (256 IPs) that is running out of addresses. You need to expand it to accommodate more VMs. What should you do?

A) Delete the subnet and recreate it with a larger range
B) Create a second subnet in the same region with a different IP range
C) Expand the subnet using `gcloud compute networks subnets expand-ip-range` with a smaller prefix length
D) Contact Google support to increase the subnet's IP capacity

<details>
<summary>Answer</summary>

**Correct: C)**

You can expand a subnet's CIDR range using `gcloud compute networks subnets expand-ip-range --prefix-length=20`, which would grow from /24 (256 IPs) to /20 (4,096 IPs). Expansion does not disrupt existing connections. However, you can NEVER shrink a subnet -- plan carefully. Option A would require deleting all resources in the subnet first, causing downtime. Option B adds a second subnet but does not expand the existing one, which may not work if resources must stay in the same subnet. Option D is unnecessary because this is a self-service operation.
</details>

---

### Q45. Your VMs have no external IP addresses but need to download packages from the internet for patching. You do not want to assign public IPs. What should you configure?

A) An HTTP(S) proxy server on a VM with an external IP
B) Cloud NAT with a Cloud Router on the VPC
C) A VPN tunnel to an on-premises internet gateway
D) Private Google Access on the subnet

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud NAT provides outbound internet access for VMs without external IP addresses. It requires a Cloud Router (but does not use BGP for NAT). Cloud NAT is regional and supports both VM instances and GKE nodes. It only allows outbound connections -- it does not enable inbound access from the internet. Option A works but adds operational overhead. Option C is unnecessarily complex. Option D enables access to Google APIs and services only, not the general internet.
</details>

---

### Q46. Your team needs to create a private DNS zone that resolves `internal.example.com` for VMs within your VPC but is not visible to the public internet. What should you configure?

A) A public Cloud DNS zone for `internal.example.com`
B) A private Cloud DNS zone for `internal.example.com` associated with your VPC network
C) DNS records in the VM's `/etc/hosts` files
D) A Cloud Router with custom DNS settings

<details>
<summary>Answer</summary>

**Correct: B)**

A private Cloud DNS managed zone is visible only within the specified VPC networks. You create it with `--visibility=private --networks=VPC_NAME`. This allows internal DNS resolution without exposing records publicly. Option A makes the zone publicly resolvable, which is not desired. Option C is unmanageable at scale and does not provide DNS for the entire VPC. Option D -- Cloud Router does not manage DNS settings.
</details>

---

### Q47. You need to enable Data Access audit logs for Cloud Storage in your project. You already have Admin Activity logs. What do you need to do?

A) Nothing -- Data Access audit logs are enabled by default for all services
B) Edit the project's IAM policy to add `auditConfigs` for `storage.googleapis.com` with DATA_READ and DATA_WRITE log types
C) Enable them in Cloud Monitoring under Audit Logs settings
D) Run `gcloud logging enable-data-access --service=storage.googleapis.com`

<details>
<summary>Answer</summary>

**Correct: B)**

Data Access audit logs must be explicitly enabled (they are NOT on by default, with the exception of BigQuery). You enable them by editing the project's IAM policy to add an `auditConfigs` section specifying the service and log types (ADMIN_READ, DATA_READ, DATA_WRITE). This is done via `gcloud projects get-iam-policy`, editing the YAML, and `gcloud projects set-iam-policy`. Option A is incorrect -- they are not enabled by default (except BigQuery). Option C is wrong -- this is not configured in Cloud Monitoring. Option D uses a non-existent gcloud command.
</details>

---

### Q48. You need to collect application logs and system metrics from Compute Engine VMs running a custom web application. What is the recommended agent to install?

A) Legacy Cloud Logging agent (google-fluentd)
B) Legacy Cloud Monitoring agent (collectd-based)
C) Ops Agent
D) Prometheus Node Exporter

<details>
<summary>Answer</summary>

**Correct: C)**

The Ops Agent is the recommended agent for collecting both logs and metrics from Compute Engine VMs. It replaces the legacy separate Monitoring agent (collectd-based) and Logging agent (google-fluentd). The Ops Agent uses Fluent Bit for logging and OpenTelemetry for metrics collection. Options A and B are legacy agents that have been superseded. Option D (Prometheus Node Exporter) collects metrics in Prometheus format but does not handle logging and is not the recommended GCP approach for Compute Engine.
</details>

---

### Q49. You want to export all audit logs from your organization to BigQuery for long-term compliance analysis. The export should include logs from all current and future projects. What should you configure?

A) A log sink in each individual project pointing to BigQuery
B) An aggregated log sink at the organization level pointing to BigQuery
C) Enable billing export to BigQuery and filter for audit log entries
D) A Cloud Scheduler job that copies logs from Cloud Logging to BigQuery daily

<details>
<summary>Answer</summary>

**Correct: B)**

An aggregated log sink at the organization level captures logs from all projects (current and future) and routes them to a destination. Creating it at the org level means new projects are automatically included. The sink's writer identity service account needs `roles/bigquery.dataEditor` on the destination dataset. Option A requires creating a sink per project and does not automatically include new projects. Option C exports billing data, not audit logs. Option D is overly complex and not how log export works in GCP.
</details>

---

## Domain 5: Configuring Access and Security

### Q50. A junior developer needs to view VM instances and their configurations but should not be able to create, modify, or delete any VMs. What is the least-privilege predefined role you should grant?

A) `roles/viewer`
B) `roles/compute.viewer`
C) `roles/compute.instanceAdmin.v1`
D) `roles/editor`

<details>
<summary>Answer</summary>

**Correct: B)**

`roles/compute.viewer` grants read-only access to Compute Engine resources, allowing the developer to view VM instances and their configurations without the ability to modify them. Option A (`roles/viewer`) is a basic role that grants read access across ALL services in the project, which is more than needed. Option C grants full instance administration including create, modify, and delete. Option D (`roles/editor`) grants modify access across all services, far exceeding the requirement.
</details>

---

### Q51. You have a predefined role that is too broad and a narrower predefined role that is missing one permission your application needs. What should you do?

A) Grant both predefined roles to the service account
B) Create a custom role with only the specific permissions needed
C) Grant `roles/editor` since it includes all permissions
D) Ask Google to modify the predefined role

<details>
<summary>Answer</summary>

**Correct: B)**

Custom roles allow you to define an exact set of permissions, following the principle of least privilege. You include only the permissions the application actually needs. Custom roles can be created at the project or organization level (but not folder level). Option A would grant more permissions than needed by combining two roles. Option C violates least privilege -- Editor includes thousands of permissions. Option D -- Google does not modify predefined roles based on user requests.
</details>

---

### Q52. You need to create a service account for a Cloud Run microservice that reads objects from a Cloud Storage bucket and publishes messages to Pub/Sub. Following least privilege, which roles should you grant?

A) `roles/editor`
B) `roles/storage.objectViewer` and `roles/pubsub.publisher`
C) `roles/storage.admin` and `roles/pubsub.admin`
D) `roles/storage.objectAdmin` and `roles/pubsub.editor`

<details>
<summary>Answer</summary>

**Correct: B)**

The service only needs to read Cloud Storage objects (not create or delete) and publish to Pub/Sub (not create topics or manage subscriptions). `roles/storage.objectViewer` provides read access to objects, and `roles/pubsub.publisher` provides the ability to publish messages. Option A is far too broad. Options C and D both grant more permissions than needed -- admin roles include management capabilities that are not required.
</details>

---

### Q53. A developer needs to deploy a Cloud Run service that runs as a specific service account `app-sa@project.iam.gserviceaccount.com`. The developer has `roles/run.admin` on the project. The deployment fails with a permission error related to the service account. What is missing?

A) The developer needs `roles/iam.serviceAccountAdmin` on the service account
B) The developer needs `roles/iam.serviceAccountUser` on the service account
C) The developer needs `roles/iam.serviceAccountTokenCreator` on the service account
D) The service account needs `roles/run.admin` on the project

<details>
<summary>Answer</summary>

**Correct: B)**

To attach (or "act as") a service account when deploying a resource like Cloud Run, the deploying user needs `roles/iam.serviceAccountUser` on the target service account. This role grants the ability to set the service account identity on resources. Option A (serviceAccountAdmin) allows managing the SA (create, delete, update keys) but NOT acting as it. Option C (serviceAccountTokenCreator) allows generating tokens for impersonation but is not what is needed for attaching a SA to a resource. Option D is about the SA's permissions, not the developer's ability to use the SA.
</details>

---

### Q54. A CI/CD pipeline running on GitHub Actions needs to deploy resources to GCP. You want to avoid storing long-lived service account keys in GitHub. What is the recommended approach?

A) Store a service account key JSON file as a GitHub encrypted secret
B) Use Workload Identity Federation with an OIDC provider for GitHub Actions
C) Create a service account key, rotate it monthly, and store it in Secret Manager
D) Use a personal Google account's credentials in the CI/CD pipeline

<details>
<summary>Answer</summary>

**Correct: B)**

Workload Identity Federation allows external workloads (like GitHub Actions) to authenticate to GCP without service account keys. GitHub Actions provides OIDC tokens that can be exchanged for short-lived GCP access tokens. You create a Workload Identity Pool and Provider, then allow the GitHub identity to impersonate a GCP service account. Option A uses long-lived keys, which is exactly what we want to avoid. Option C still uses keys, just with rotation. Option D uses personal credentials, which is insecure and not scalable.
</details>

---

### Q55. Your organization's default Compute Engine service account has `roles/editor` on the project. A new VM is created without specifying a service account. What is the security risk?

A) The VM has no access to any GCP resources
B) The VM inherits the Editor role, giving it near-full access to all resources in the project
C) The VM only has read access to Compute Engine resources
D) The VM cannot communicate with other GCP services

<details>
<summary>Answer</summary>

**Correct: B)**

When no service account is specified, VMs use the default Compute Engine service account, which is automatically granted `roles/editor` on the project. This means the VM (and any application running on it) has the ability to create, modify, and delete most resources across all services in the project. This is a significant security risk, especially if the VM is compromised. Best practice is to create dedicated service accounts with minimum permissions for each workload.
</details>

---

### Q56. You need to allow a user to generate short-lived access tokens for a service account to test an API integration, without giving them the ability to deploy resources as that service account. Which role should you grant on the service account?

A) `roles/iam.serviceAccountUser`
B) `roles/iam.serviceAccountTokenCreator`
C) `roles/iam.serviceAccountAdmin`
D) `roles/iam.serviceAccountKeyAdmin`

<details>
<summary>Answer</summary>

**Correct: B)**

`roles/iam.serviceAccountTokenCreator` allows a user to generate access tokens, ID tokens, sign blobs, and sign JWTs for the service account. This is exactly what is needed for impersonation and testing. Option A (`serviceAccountUser`) allows attaching/deploying resources as the SA, which is more than needed and not what was requested. Option C (`serviceAccountAdmin`) allows managing the SA itself (create, delete, update) but not generating tokens. Option D (`serviceAccountKeyAdmin`) allows creating and managing long-lived keys, not short-lived tokens.
</details>

---

### Q57. A GKE pod needs to write data to a Cloud Storage bucket. Your company policy prohibits service account keys. What is the recommended approach?

A) Mount a service account key as a Kubernetes secret
B) Use the node's default service account with `roles/storage.admin`
C) Configure Workload Identity to map the Kubernetes service account to a Google Cloud service account with `roles/storage.objectCreator`
D) Use the pod's environment variables to pass credentials

<details>
<summary>Answer</summary>

**Correct: C)**

Workload Identity is the recommended way for GKE pods to authenticate to Google Cloud services without keys. You map a Kubernetes service account (KSA) to a Google Cloud service account (GSA) using `roles/iam.workloadIdentityUser`. The GSA is granted the minimum required role (`roles/storage.objectCreator` for writing). Option A uses service account keys, which violates the company policy. Option B uses the node's default SA, which is shared across all pods on the node and likely over-permissioned. Option D is vague but would still require some form of credentials.
</details>

---

### Q58. Your organization wants to prevent anyone from creating service account keys across all projects. What is the most effective way to enforce this?

A) Remove `roles/iam.serviceAccountKeyAdmin` from all users
B) Apply the `constraints/iam.disableServiceAccountKeyCreation` organization policy at the organization level
C) Create an IAM deny policy that blocks the `iam.serviceAccountKeys.create` permission
D) Send a company-wide email instructing everyone not to create keys

<details>
<summary>Answer</summary>

**Correct: B)**

The organization policy constraint `constraints/iam.disableServiceAccountKeyCreation` prevents the creation of user-managed service account keys across the entire organization (or any level of the hierarchy where it is applied). This is a preventive control that applies regardless of the user's IAM roles. Option A is difficult to maintain and could miss users who get the permission through custom roles or basic roles. Option C could work for specific principals but is more complex to manage than an org policy. Option D relies on human compliance and is unenforceable.
</details>

---

### Q59. A user has `roles/editor` at the organization level through inheritance. You need to prevent them from deleting Cloud Storage objects in a specific production project while keeping all their other permissions. What should you do?

A) Remove `roles/editor` from the user at the organization level
B) Grant `roles/storage.objectViewer` at the project level to override the Editor role
C) Create an IAM deny policy on the project that denies `storage.objects.delete` for the user
D) Move the project out of the organization

<details>
<summary>Answer</summary>

**Correct: C)**

IAM deny policies explicitly block specific permissions and are evaluated BEFORE allow policies. By creating a deny policy on the project that denies `storage.objects.delete` for this user, you effectively block object deletion while keeping all other Editor permissions intact. Option A would affect ALL projects, not just the production project, and could disrupt the user's work elsewhere. Option B does not work -- IAM policies are additive, so adding a viewer role does not remove delete permissions from the inherited Editor role. Option D is extreme and does not solve the underlying problem.
</details>

---

### Q60. You are using `gcloud` to impersonate a service account for testing. Which command sets impersonation for all subsequent gcloud commands in the session?

A) `gcloud auth activate-service-account --key-file=sa-key.json`
B) `gcloud config set auth/impersonate_service_account my-sa@project.iam.gserviceaccount.com`
C) `gcloud auth login --service-account=my-sa@project.iam.gserviceaccount.com`
D) `gcloud iam service-accounts impersonate my-sa@project.iam.gserviceaccount.com`

<details>
<summary>Answer</summary>

**Correct: B)**

`gcloud config set auth/impersonate_service_account SA_EMAIL` configures all subsequent gcloud commands to impersonate the specified service account. The caller must have `roles/iam.serviceAccountTokenCreator` on the target service account. This generates short-lived credentials and leaves an audit trail. To clear it, use `gcloud config unset auth/impersonate_service_account`. Option A activates a service account using a downloaded key file, which is not impersonation -- it is direct authentication with long-lived credentials. Option C is for user login, not service account impersonation. Option D is not a valid gcloud command.
</details>

---

## Answer Key

| Q | Answer | Q | Answer | Q | Answer | Q | Answer |
|---|--------|---|--------|---|--------|---|--------|
| 1 | B | 16 | B | 31 | B | 46 | B |
| 2 | C | 17 | C | 32 | B | 47 | B |
| 3 | C | 18 | D | 33 | B | 48 | C |
| 4 | B | 19 | C | 34 | C | 49 | B |
| 5 | B | 20 | B | 35 | B | 50 | B |
| 6 | B | 21 | B | 36 | C | 51 | B |
| 7 | B | 22 | C | 37 | A | 52 | B |
| 8 | C | 23 | B | 38 | C | 53 | B |
| 9 | B | 24 | B | 39 | B | 54 | B |
| 10 | B | 25 | C | 40 | C | 55 | B |
| 11 | C | 26 | C | 41 | B | 56 | B |
| 12 | B | 27 | B | 42 | B | 57 | C |
| 13 | C | 28 | B | 43 | B | 58 | B |
| 14 | B | 29 | B | 44 | C | 59 | C |
| 15 | B | 30 | B | 45 | B | 60 | B |

---

## Scoring Guide

| Score | Readiness |
|-------|-----------|
| 54-60 (90%+) | You are well prepared. Go take the exam. |
| 48-53 (80-89%) | Solid foundation. Review the topics you missed. |
| 42-47 (70-79%) | Borderline passing. Review all domain study guides for weak areas. |
| Below 42 (<70%) | More study needed. Focus on the domains where you missed the most questions. |

---

*Questions aligned with the [official exam guide](https://cloud.google.com/learn/certification/guides/cloud-engineer/). Cross-reference with the domain study guides (01 through 05) for deeper coverage.*
