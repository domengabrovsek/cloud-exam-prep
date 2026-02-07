# Section 3: Ensuring Successful Operation of a Cloud Solution

> **Exam weight:** ~27% of the standard ACE exam, ~40% of the renewal exam.
>
> This section covers managing Compute Engine resources, managing GKE resources, managing Cloud Run resources, managing storage and database solutions, managing networking resources, monitoring and logging, and using Cloud Observability tools (Cloud Trace, Cloud Profiler, Service Health, Active Assist, Gemini Cloud Assist).

**Total questions: 34**

---

### Q1. You take a snapshot of a 500 GB persistent disk. You then make 10 GB of changes and take another snapshot. How much new data does the second snapshot store?

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

### Q2. You want to create a reusable boot image for your application so that new VMs always start with the same pre-configured software. You have a running VM with the software installed. What is the recommended approach?

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

### Q3. Your GKE cluster needs a new node pool with GPU-enabled nodes for machine learning workloads, while the existing node pool continues to serve the web application. What should you do?

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

### Q4. Your GKE deployment experiences variable traffic. During peak hours, pods hit 90% CPU utilization. During off-hours, most pods are idle. You want to automatically adjust the number of pod replicas based on CPU usage. What should you configure?

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

### Q5. You deployed a Cloud Run service revision `v2` that has a bug. You need to immediately route all traffic back to the stable `v1` revision. What command should you run?

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

### Q6. You have a Cloud Storage bucket with objects that should transition from Standard to Nearline after 30 days, then to Coldline after 90 days, and be deleted after 2 years. How should you configure this?

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

### Q7. You have a subnet with CIDR range 10.0.1.0/24 (256 IPs) that is running out of addresses. You need to expand it to accommodate more VMs. What should you do?

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

### Q8. Your VMs have no external IP addresses but need to download packages from the internet for patching. You do not want to assign public IPs. What should you configure?

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

### Q9. Your team needs to create a private DNS zone that resolves `internal.example.com` for VMs within your VPC but is not visible to the public internet. What should you configure?

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

### Q10. You need to enable Data Access audit logs for Cloud Storage in your project. You already have Admin Activity logs. What do you need to do?

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

### Q11. You need to collect application logs and system metrics from Compute Engine VMs running a custom web application. What is the recommended agent to install?

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

### Q12. You want to export all audit logs from your organization to BigQuery for long-term compliance analysis. The export should include logs from all current and future projects. What should you configure?

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

### Q13. You need to SSH into a Compute Engine VM that has no external IP address. The VM is in a private subnet. What's the recommended approach?

A) Assign a temporary external IP, SSH in, then remove it
B) Use Identity-Aware Proxy (IAP) TCP forwarding with `gcloud compute ssh --tunnel-through-iap`
C) Set up a Cloud VPN just for SSH access
D) Connect through the serial console

<details>
<summary>Answer</summary>

**Correct: B)**

IAP TCP forwarding creates an encrypted tunnel from your workstation to the VM through Google's network -- no external IP needed. You just need `roles/iap.tunnelResourceAccessor`. Temporary IPs (A) are a security risk. VPN (C) is overkill for SSH. Serial console (D) is for troubleshooting boot issues, not regular access.
</details>

---

### Q14. You need to create a snapshot schedule that takes daily snapshots of a Compute Engine disk and retains them for 14 days. What resource do you create?

A) A cron job using Cloud Scheduler that runs `gcloud compute disks snapshot`
B) A resource policy with a snapshot schedule
C) A Cloud Function triggered on a timer
D) A Cloud Monitoring alert that triggers snapshot creation

<details>
<summary>Answer</summary>

**Correct: B)**

Resource policies define snapshot schedules natively -- frequency, retention period, and which disks to apply them to. `gcloud compute resource-policies create snapshot-schedule` is the command. Cloud Scheduler (A) and Cloud Functions (C) work but are unnecessarily complex. Monitoring alerts (D) don't create snapshots.
</details>

---

### Q15. You need to allow your GKE cluster to pull container images from Artifact Registry in the same project. What's the simplest way to configure this?

A) Create a Kubernetes secret with registry credentials and attach it to each pod
B) No configuration needed -- the default node service account has read access to Artifact Registry in the same project
C) Configure Workload Identity for every pod that needs image access
D) Make the Artifact Registry repository public

<details>
<summary>Answer</summary>

**Correct: B)**

By default, the Compute Engine default service account (used by GKE nodes) has `roles/artifactregistry.reader` scope on repositories in the same project. Image pulling works out of the box. Kubernetes secrets (A) and Workload Identity (C) are needed for cross-project access. Public repos (D) are a security risk.
</details>

---

### Q16. You need to add a new node pool to your GKE cluster with `e2-highmem-8` machines for memory-intensive workloads. The existing pool uses `e2-standard-4`. What command do you use?

A) `gcloud container node-pools update default-pool --machine-type=e2-highmem-8`
B) `gcloud container clusters resize my-cluster --machine-type=e2-highmem-8`
C) `gcloud container node-pools create highmem-pool --cluster=my-cluster --machine-type=e2-highmem-8`
D) `kubectl set node-type e2-highmem-8`

<details>
<summary>Answer</summary>

**Correct: C)**

You cannot change the machine type of an existing node pool -- you must create a new one. `node-pools create` with `--machine-type` adds a new pool. Update (A) can't change machine type. Cluster resize (B) changes node count, not type. kubectl (D) has no such command.
</details>

---

### Q17. Your GKE cluster runs on Autopilot. A developer requests more CPU for their Pod. How do they get it?

A) Add more nodes to the cluster
B) Modify the Pod's resource `requests` in the deployment spec
C) Change the node pool machine type
D) Enable Horizontal Pod Autoscaler

<details>
<summary>Answer</summary>

**Correct: B)**

In GKE Autopilot, you don't manage nodes -- Google does. Pods get resources based on their `requests` spec. To get more CPU, increase the `resources.requests.cpu` value in the Pod/Deployment manifest. Autopilot adjusts nodes automatically. You can't manage nodes (A, C) in Autopilot. HPA (D) scales Pod count, not individual Pod resources.
</details>

---

### Q18. You have a Cloud Run service at revision `v1` (serving 100% traffic) and deploy `v2`. You want to test `v2` with a specific URL without sending any production traffic to it. What do you use?

A) `gcloud run services update-traffic --to-revisions=v2=0`
B) `gcloud run deploy --tag=canary`
C) `gcloud run services update-traffic --to-revisions=v2=10`
D) Deploy to a separate Cloud Run service

<details>
<summary>Answer</summary>

**Correct: B)**

Revision tags create a dedicated URL for a specific revision (e.g., `canary---myservice-xyz.run.app`) without receiving any production traffic. You can test it directly via the tagged URL. 0% traffic (A) makes it unreachable. 10% traffic (C) sends production traffic. Separate service (D) is unnecessary overhead.
</details>

---

### Q19. You need to configure autoscaling for a Cloud Run service. The service should always have at least 2 instances warm and scale up to a maximum of 100. Which flags do you use?

A) `--min-instances=2 --max-instances=100`
B) `--concurrency=2 --max-instances=100`
C) `--cpu-throttling --min-instances=2`
D) `--min-replicas=2 --max-replicas=100`

<details>
<summary>Answer</summary>

**Correct: A)**

`--min-instances` keeps instances warm to avoid cold starts. `--max-instances` caps the scale-up. Concurrency (B) controls requests per instance, not instance count. CPU throttling (C) controls whether CPU is allocated outside requests. The correct flag names use "instances" not "replicas" (D).
</details>

---

### Q20. You need to manage lifecycle policies for a Cloud Storage bucket: move objects to Nearline after 30 days, to Coldline after 90 days, and delete after 365 days. How many lifecycle rules do you need?

A) 1 rule with 3 conditions
B) 3 separate rules
C) 2 rules (transition and deletion are separate)
D) You can only have 1 lifecycle rule per bucket

<details>
<summary>Answer</summary>

**Correct: B)**

Each action (SetStorageClass to Nearline, SetStorageClass to Coldline, Delete) requires its own rule with its own condition. Lifecycle management supports multiple rules per bucket. One rule can only have one action.
</details>

---

### Q21. You need to estimate the cost of a BigQuery query that scans a 5 TB table before running it. What do you use?

A) Cloud Billing dashboard
B) `bq query --dry_run` flag
C) `EXPLAIN` keyword in the SQL query
D) BigQuery Pricing Calculator

<details>
<summary>Answer</summary>

**Correct: B)**

`--dry_run` returns the estimated bytes that would be scanned without executing the query. At $5/TB on-demand pricing, 5 TB = $25. This is the standard way to estimate before running. Billing dashboard (A) shows past costs. EXPLAIN (C) shows query plan, not bytes. Pricing Calculator (D) is for overall project estimates.
</details>

---

### Q22. You need to back up a Cloud SQL instance and be able to restore to a specific point in time (3 hours ago). What must be enabled?

A) Automated daily backups only
B) Point-in-time recovery (PITR) with binary logging enabled
C) A Cloud Function that exports the database hourly
D) A read replica that can be promoted

<details>
<summary>Answer</summary>

**Correct: B)**

PITR requires binary logging (MySQL) or WAL archiving (PostgreSQL) to be enabled. It allows restoration to any second within the retention window. Daily backups (A) could lose up to 24 hours. Hourly exports (C) lose up to 1 hour. Read replicas (D) replicate deletions too.
</details>

---

### Q23. You want to use Database Center to manage your Google Cloud database fleet. What does Database Center provide?

A) A service for creating databases automatically
B) A centralized dashboard to inventory, monitor, and manage all database instances across projects
C) A migration tool for moving on-premises databases to Cloud SQL
D) A backup service for all databases

<details>
<summary>Answer</summary>

**Correct: B)**

Database Center provides a unified view of all your database instances across Google Cloud -- Cloud SQL, Spanner, AlloyDB, Bigtable, Firestore, Memorystore. It helps with inventory management, compliance monitoring, and operational insights. It doesn't create databases (A), migrate them (C), or replace individual backup mechanisms (D).
</details>

---

### Q24. You need to review the status of a running Dataflow pipeline job. How do you check it?

A) `gcloud dataflow jobs list` and `gcloud dataflow jobs describe JOB_ID`
B) `kubectl get jobs`
C) Cloud SQL admin console
D) `bq show --job JOB_ID`

<details>
<summary>Answer</summary>

**Correct: A)**

`gcloud dataflow jobs list` shows all Dataflow jobs, and `describe` gives detailed status including pipeline steps, errors, and metrics. You can also view in the Cloud Console under Dataflow > Jobs. kubectl (B) is for Kubernetes. Cloud SQL (C) is databases. `bq show` (D) is for BigQuery jobs.
</details>

---

### Q25. You need to add a custom static route in your VPC to send traffic destined for `10.128.0.0/16` to a specific VM acting as a network appliance. What do you create?

A) A firewall rule allowing traffic to `10.128.0.0/16`
B) A custom static route with destination `10.128.0.0/16` and next hop set to the appliance VM instance
C) A Cloud NAT gateway for the `10.128.0.0/16` range
D) A VPC peering connection to `10.128.0.0/16`

<details>
<summary>Answer</summary>

**Correct: B)**

`gcloud compute routes create` with `--destination-range=10.128.0.0/16` and `--next-hop-instance=appliance-vm` creates a custom static route directing matching traffic to the specified VM. Firewall rules (A) control access, not routing. Cloud NAT (C) is for outbound internet. VPC Peering (D) connects entire VPCs, not specific ranges.
</details>

---

### Q26. You need to set up Cloud DNS for your private VPC so that VMs can resolve `internal.mycompany.com`. External users should NOT be able to resolve this domain. What type of DNS zone do you create?

A) Public managed zone
B) Private managed zone attached to the VPC
C) Forwarding zone to an external DNS server
D) CNAME record in a public zone

<details>
<summary>Answer</summary>

**Correct: B)**

A private managed zone is only visible to the VPCs you attach it to. VMs within those VPCs can resolve records, but the domain is invisible externally. Public zones (A, D) are internet-facing. Forwarding zones (C) redirect queries to another DNS server, not what's needed here.
</details>

---

### Q27. Your Cloud NAT gateway is configured, but VMs in a new subnet can't reach the internet. VMs in other subnets work fine. What's likely wrong?

A) The Cloud Router is misconfigured
B) The new subnet isn't included in the Cloud NAT configuration -- it's set to specific subnets, not all subnets
C) Cloud NAT has a maximum number of supported subnets
D) The VMs need external IP addresses for Cloud NAT to work

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud NAT can be configured to apply to all subnets or specific subnets. If set to specific subnets, new subnets aren't automatically included -- you must add them. Cloud Router (A) issues would affect all subnets. There's no hard subnet limit (C). Cloud NAT specifically works with VMs that DON'T have external IPs (D).
</details>

---

### Q28. You're investigating slow response times in your application. You want to trace requests across multiple services to find the bottleneck. Which tool should you use?

A) Cloud Monitoring metrics explorer
B) Cloud Logging log viewer
C) Cloud Trace
D) Cloud Profiler

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud Trace collects latency data from distributed applications, showing you how requests propagate across services and where bottlenecks occur. It visualizes the request lifecycle as trace spans. Cloud Monitoring (A) shows metrics but not request flows. Cloud Logging (B) shows log entries. Cloud Profiler (D) analyzes CPU and memory usage of code, not request latency across services.
</details>

---

### Q29. Your application's Cloud SQL queries are slow. You want to identify which specific queries are taking the longest and see index recommendations. Which tool do you use?

A) Cloud Trace
B) Cloud Profiler
C) Query Insights and index advisor
D) Cloud Monitoring custom metrics

<details>
<summary>Answer</summary>

**Correct: C)**

Query Insights (available in Cloud SQL) shows query performance metrics -- top queries by latency, load, and frequency. The index advisor recommends indexes to improve query performance. Cloud Trace (A) traces distributed requests, not SQL queries. Cloud Profiler (B) profiles application code. Custom metrics (D) require manual instrumentation.
</details>

---

### Q30. You want to view the current status and known issues affecting Google Cloud services in your regions. Where do you look?

A) Cloud Monitoring uptime checks
B) Personalized Service Health dashboard
C) Cloud Logging for error rates
D) `gcloud services list`

<details>
<summary>Answer</summary>

**Correct: B)**

The Personalized Service Health dashboard shows incidents and disruptions specifically affecting the Google Cloud services and regions your projects use. It's personalized based on your actual resource footprint. Uptime checks (A) monitor YOUR services. Cloud Logging (C) shows your logs. `gcloud services list` (D) shows enabled APIs.
</details>

---

### Q31. You want to use Active Assist to optimize your GCP resource utilization. What kind of recommendations does Active Assist provide?

A) Only billing optimization recommendations
B) Recommendations across cost, security, performance, and manageability -- including idle VM identification, rightsizing, and IAM recommendations
C) Only security vulnerability scanning
D) Only network performance optimization

<details>
<summary>Answer</summary>

**Correct: B)**

Active Assist is a portfolio of tools providing recommendations across multiple pillars: cost (idle resources, rightsizing VMs, committed use discounts), security (IAM recommender, overly permissive roles), performance (VM machine type recommendations), and manageability. It's not limited to any single category.
</details>

---

### Q32. You want to use Gemini Cloud Assist within Cloud Monitoring. What can it help you do?

A) Automatically resolve all alerts
B) Write monitoring queries, explain metrics, troubleshoot issues using natural language, and create alert policies
C) Replace Cloud Monitoring entirely
D) Only create uptime checks

<details>
<summary>Answer</summary>

**Correct: B)**

Gemini Cloud Assist in Cloud Monitoring helps you write MQL/PromQL queries using natural language, explains what metrics mean, helps troubleshoot performance issues, and assists in creating alert policies. It doesn't auto-resolve alerts (A), replace Monitoring (C), or only handle uptime checks (D).
</details>

---

### Q33. You are implementing Cloud Storage for your organization. You need to follow your organization's regulations: 1) Archive data older than one year. 2) Delete data older than 5 years. 3) Use standard storage for all other data. You want to implement these guidelines automatically and in the simplest manner available. What should you do?

A) Set up Object Lifecycle management policies.
B) Run a script daily. Copy data that is older than one year to an archival bucket, and delete five-year-old data.
C) Run a script daily. Set storage class to ARCHIVE for data that is older than one year, and delete five-year-old data.
D) Set up default storage class for three buckets named: STANDARD, ARCHIVE, DELETED. Use a script to move the data in the appropriate bucket when its condition matches your company guidelines.

<details>
<summary>Answer</summary>

**Correct: A)**

**Object Lifecycle Management** is a built-in Cloud Storage feature that automatically transitions storage classes and deletes objects based on conditions like age. You define rules (e.g., `SetStorageClass` to ARCHIVE after 365 days, `Delete` after 1825 days) and Google handles execution. No scripts, no cron jobs, no maintenance. Options B, C, and D all require custom scripts that must be maintained, monitored, and are error-prone.

> **Exam tip:** Whenever you see "automatically" + Cloud Storage class transitions or deletion based on age, the answer is **Object Lifecycle Management**.
</details>

---

### Q34. You receive an error message when you try to start a new VM: "You have exhausted the IP range in your subnet." You want to resolve the error with the least amount of effort. What should you do?

A) Create a new subnet and start your VM there.
B) Expand the CIDR range in your subnet, and restart the VM that issued the error.
C) Create another subnet, and move several existing VMs into the new subnet.
D) Restart the VM using exponential backoff until the VM starts successfully.

<details>
<summary>Answer</summary>

**Correct: B)**

GCP allows you to **expand a subnet's CIDR range** without downtime or disruption to existing VMs. This is a single `gcloud compute networks subnets expand-ip-range` command -- least effort. Creating a new subnet (A) works but is more effort (you may need to update firewall rules, routes, etc.). Moving VMs (C) is even more effort. Exponential backoff (D) won't help -- the IPs are exhausted, retrying won't free them.

> **Exam tip:** GCP subnets can be **expanded but never shrunk**. This is a frequently tested fact. `expand-ip-range` is the command.
</details>

---

## Answer Key

| Q | Answer | Source | Topic |
|---|--------|--------|-------|
| 1 | C | 07-Q38 | Incremental snapshots |
| 2 | B | 07-Q39 | Custom images + instance templates |
| 3 | C | 07-Q40 | GKE GPU node pool |
| 4 | B | 07-Q41 | Horizontal Pod Autoscaler |
| 5 | B | 07-Q42 | Cloud Run traffic rollback |
| 6 | B | 07-Q43 | Object Lifecycle Management |
| 7 | C | 07-Q44 | Subnet IP range expansion |
| 8 | B | 07-Q45 | Cloud NAT |
| 9 | B | 07-Q46 | Private Cloud DNS zone |
| 10 | B | 07-Q47 | Data Access audit logs |
| 11 | C | 07-Q48 | Ops Agent |
| 12 | B | 07-Q49 | Aggregated org-level log sink |
| 13 | B | 11-Q31 | IAP TCP forwarding |
| 14 | B | 11-Q32 | Snapshot schedule resource policy |
| 15 | B | 11-Q33 | GKE + Artifact Registry default |
| 16 | C | 11-Q34 | GKE node pool creation |
| 17 | B | 11-Q35 | Autopilot Pod resource requests |
| 18 | B | 11-Q36 | Cloud Run revision tags |
| 19 | A | 11-Q37 | Cloud Run min/max instances |
| 20 | B | 11-Q38 | Lifecycle rules count |
| 21 | B | 11-Q39 | BigQuery dry run |
| 22 | B | 11-Q40 | Cloud SQL PITR |
| 23 | B | 11-Q41 | Database Center |
| 24 | A | 11-Q42 | Dataflow job status |
| 25 | B | 11-Q43 | Custom static routes |
| 26 | B | 11-Q44 | Private Cloud DNS zone |
| 27 | B | 11-Q45 | Cloud NAT subnet config |
| 28 | C | 11-Q46 | Cloud Trace |
| 29 | C | 11-Q47 | Query Insights + index advisor |
| 30 | B | 11-Q48 | Service Health dashboard |
| 31 | B | 11-Q49 | Active Assist |
| 32 | B | 11-Q50 | Gemini Cloud Assist monitoring |
| 33 | A | 08-Q6 | Object Lifecycle Management |
| 34 | B | 08-Q12 | Subnet CIDR expansion |
