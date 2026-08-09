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

Dedicated Interconnect is a direct physical connection into Google's network, available at 10, 100 and 400 Gbps link speeds, with the lowest and most predictable latency of the connectivity options. Real-time trading is exactly the case that justifies it.

- **A) is wrong** -- Cloud VPN runs over the public internet, so latency varies with internet conditions, and its documented throughput ceiling is expressed in packets per second rather than a flat gigabit figure.
- **C) is wrong** -- Partner Interconnect inserts a service provider into the path, adding a hop and some latency. It is the answer when you cannot meet Google's colocation requirement, which this stem does not say.
- **D) is wrong** -- Cloud CDN caches content at the edge. It is not a private connectivity product.

**Exam tip:** two facts pick the connectivity option: required bandwidth and whether the customer can colocate in a Google peering facility. Colocation possible plus multi-gigabit means Dedicated; no colocation means Partner; low bandwidth or budget-constrained means HA VPN. "Strict latency" always rules out VPN.

Docs: https://cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview
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

Partner Interconnect exists for exactly this shape: private connectivity without colocation presence, through a supported service provider, with VLAN attachments sized from tens of megabits upward. Fifteen branches needing 50-500 Mbps each fit it cleanly.

- **A) is wrong** -- Dedicated Interconnect requires colocation in a Google peering facility, which the company does not have, and 10 Gbps per branch is far beyond the stated need.
- **C) is wrong** -- VPN over the public internet is workable for some branches but gives variable latency and less predictable performance across 15 production sites.
- **D) is wrong** -- Cloud Router is the BGP component used alongside Interconnect or VPN. It is not a connectivity product on its own.

**Exam tip:** "no colocation presence" is the single phrase that forces Partner Interconnect. Also watch for Cloud Router offered as if it were a connection: it only exchanges routes, so it is always a supporting component, never the answer to "which connectivity option".

Docs: https://cloud.google.com/network-connectivity/docs/interconnect/concepts/partner-overview
</details>

---

### Q3. You are designing a network for a SaaS platform that serves customers globally. The platform has backend services in us-central1 and europe-west1. You need to route users to the closest healthy backend with a single anycast IP address. Which load balancer should you use?

A) Regional external Application Load Balancer
B) Global external Application Load Balancer
C) Global external proxy Network Load Balancer
D) Regional internal passthrough Network Load Balancer

<details>
<summary>Answer</summary>

**Correct: B)**

The global external Application Load Balancer presents one anycast IP and routes HTTP(S) traffic to the closest healthy backend across regions, with URL maps, SSL termination and Cloud CDN integration available on top.

- **A) is wrong** -- a regional external Application Load Balancer serves one region and has no global anycast address.
- **C) is wrong** -- the global external proxy Network Load Balancer handles TCP and SSL but has no HTTP-layer routing, so no URL maps or host rules.
- **D) is wrong** -- a regional internal passthrough Network Load Balancer is internal-only and regional, so it is not internet-facing at all.

**Exam tip:** decode the load balancer names literally. Global or regional is the scope, external or internal is the audience, Application or Network is the layer, and proxy or passthrough is whether the connection terminates. Match each word to a phrase in the stem and the answer usually falls out without any product knowledge.

Docs: https://cloud.google.com/load-balancing/docs/application-load-balancer
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

Cross-Cloud Interconnect provisions dedicated private connectivity directly between Google Cloud and another cloud provider at 10 or 100 Gbps, with no on-premises hop in the middle.

- **A) is wrong** -- VPN between the two providers' gateways sends traffic across the public internet, which the requirement rules out.
- **B) is wrong** -- routing GCP to on-premises to AWS adds two extra legs of latency, cost and failure surface for a link that can be direct.
- **D) is wrong** -- it also drags on-premises infrastructure into a path that does not need it.

**Exam tip:** cloud-to-cloud private connectivity is Cross-Cloud Interconnect. The distractor pattern is always "hairpin through on-premises", which technically works and is always the wrong architecture when a direct product exists.

Docs: https://cloud.google.com/network-connectivity/docs/interconnect/concepts/cci-overview
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

Cloud Armor covers all three requirements in one service: DDoS protection at the edge, preconfigured WAF rules covering OWASP categories including SQL injection and cross-site scripting, and geo-based rules that allow or deny by source country.

- **A) is wrong** -- VPC firewall rules work at L3/L4 on the VPC, so they cannot express country-based policy on inbound HTTP traffic.
- **B) is wrong** -- Cloud NAT is an egress product and IAP protects internal applications by identity. Neither defends a public API from DDoS or injection.
- **D) is wrong** -- VPC Service Controls guards Google Cloud API boundaries, not application HTTP traffic, and Cloud IDS only detects; it does not block inline.

**Exam tip:** keep the four network security products in separate boxes. Cloud Armor is inline L7 protection on the load balancer, VPC firewall is L3/L4 inside the VPC, VPC Service Controls is an API perimeter against exfiltration, Cloud IDS is passive detection. Only Cloud Armor blocks a web attack.

Docs: https://cloud.google.com/armor/docs/security-policy-overview
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

VPC peering is non-transitive, so Spoke A cannot reach Spoke B through the hub by peering alone. Putting a network virtual appliance in the hub and pointing custom routes at it gives spoke-to-spoke connectivity with a natural place to inspect and enforce policy.

- **A) is wrong** -- pairwise peering between every pair of spokes that needs to talk grows quadratically and runs into per-VPC peering limits.
- **C) is wrong** -- collapsing 25 spokes into one Shared VPC removes the network boundary between them, which may be the very thing the design exists to preserve.
- **D) is wrong** -- a VPN mesh adds encryption overhead and tunnel management, and scales worse than peering plus an appliance.

**Exam tip:** non-transitive peering is one of the most tested VPC facts. Whenever a stem needs traffic to pass through a hub, the answer is an appliance with custom routes, or Network Connectivity Center, never plain peering. Also worth knowing that Network Connectivity Center is the managed answer to this shape at larger scale.

Docs: https://cloud.google.com/vpc/docs/vpc-peering and https://cloud.google.com/network-connectivity/docs/network-connectivity-center
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

A Cloud DNS forwarding zone for corp.example.com sends queries for that domain to the on-premises resolver at 10.0.1.10 over the VPN or Interconnect. That is the supported hybrid DNS pattern and it stays current automatically as Active Directory records change.

- **B) is wrong** -- hand-maintained A records in a private zone duplicate a directory that changes constantly, so they go stale.
- **C) is wrong** -- editing resolv.conf per VM bypasses Cloud DNS, has to be repeated on every instance, and breaks the metadata-server resolution path VMs rely on.
- **D) is wrong** -- a DNS peering zone forwards resolution to another VPC's Cloud DNS, not to an on-premises server.

**Exam tip:** learn the three Cloud DNS zone types by direction. Forwarding zone sends queries out to on-premises resolvers; peering zone sends them to another VPC; private zone answers them itself. "Resolve an on-premises domain" is always a forwarding zone.

Docs: https://cloud.google.com/dns/docs/zones/forwarding-zones
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

Splitting the traffic in the URL map is the durable answer: static asset paths go to a backend service with Cloud CDN enabled, live stream paths go to a separate backend service with CDN off. Caching behaviour is then a property of the architecture rather than of a header being correct.

- **A) is wrong** -- enabling CDN everywhere and relying on max-age=0 means one header mistake caches a live stream, and it puts CDN in the path of traffic that gains nothing from it.
- **C) is wrong** -- same fragility, with the correctness of the whole design resting on origin headers.
- **D) is wrong** -- a third-party CDN adds a vendor and an integration for control that URL map routing already provides.

**Exam tip:** when content has genuinely different caching requirements, separate backend services under one URL map beats one backend service with clever headers. The exam consistently prefers configuration that cannot be defeated by a single mistake.

Docs: https://cloud.google.com/cdn/docs/caching and https://cloud.google.com/cdn/docs/overview
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

Cloud Run reaches a VPC either through Direct VPC egress, which places the service's instances on a subnet, or through a Serverless VPC Access connector. Under Shared VPC the connector lives in the host project, and principals in service projects need `roles/vpcaccess.user`. Either path lets Cloud Run reach the Cloud SQL private IP.

- **B) is wrong** -- Shared VPC service projects attach to the host network through Shared VPC association, not through VPC peering.
- **C) is wrong** -- giving Cloud SQL a public IP exposes the database to the internet to solve a routing problem that has a private answer.
- **D) is wrong** -- building a VPN into a network the project can already be attached to is unnecessary complexity.

**Exam tip:** serverless to private IP is always Direct VPC egress or a Serverless VPC Access connector. Direct VPC egress is generally the newer, higher-throughput choice; the connector is what you see in older material and in Shared VPC examples. Neither involves peering.

Docs: https://cloud.google.com/run/docs/configuring/vpc-direct-vpc and https://cloud.google.com/run/docs/configuring/connecting-vpc
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

Adaptive Protection builds a model of normal traffic and surfaces suggested rules for anomalous L7 patterns, which is what catches attacks that static WAF signatures miss. Rate limiting throttles or bans clients by request rate, so the two work together: one identifies the pattern, the other caps the volume.

- **C) is wrong** -- turning off the load balancer completes the attacker's objective for them.
- **D) is wrong** -- the global load balancer's anycast address fronts every region, so moving backends changes nothing about where the attack lands.
- **E) is wrong** -- switching network tier does not mitigate an L7 attack, and it would give up the global anycast frontend. Note that Cloud Armor itself works with load balancers in either Premium or Standard Tier, so "Standard Tier does not support Cloud Armor" is not the reason to reject this option.

**Exam tip:** two Cloud Armor features answer "sophisticated attack getting past the WAF": Adaptive Protection for anomaly detection and rate limiting for volumetric control. Do not eliminate an option by assuming a tier restriction; Cloud Armor is supported on both Premium and Standard Tier, and the real difference between tiers is routing scope.

Docs: https://cloud.google.com/armor/docs/adaptive-protection-overview and https://cloud.google.com/armor/docs/rate-limiting-overview
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

Standard holds the hot first 30 days. Nearline takes the 30-365 day band, and Archive takes everything past a year, which is where almost all of the 500 TB eventually sits and where the cost difference is decided. The 7-year delete rule satisfies retention.

- **A) is wrong** -- keeping seven years of rarely read logs in Standard is the most expensive possible configuration.
- **B) is wrong** -- it puts the long-term tier in Coldline. Coldline is for data read at most once a quarter, while this data is "almost never accessed", which is Google's description of Archive. Since the over-365-day band is the bulk of the volume, that single difference decides the question.
- **D) is wrong** -- Coldline from day one means retrieval charges on the heavily queried recent data, plus a 90-day minimum storage duration.

**Exam tip:** learn Google's own access-frequency wording, because the exam quotes it. Nearline is "once a month or less", Coldline is "at most once a quarter", Archive is "less than once a year", with minimum storage durations of 30, 90 and 365 days. Note that this stem describes the middle tier as quarterly, which maps to Coldline in isolation; the option still wins on the long-term tier, so grade every tier before choosing.

Docs: https://cloud.google.com/storage/docs/storage-classes and https://cloud.google.com/storage/docs/lifecycle
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

Point-in-time recovery on Cloud SQL for PostgreSQL replays write-ahead logs, so recovery to a chosen instant is possible and the effective RPO is seconds, inside the 5-minute target. A cross-region read replica can be promoted within minutes if the region is lost, which covers the 1-hour RTO.

- **A) is wrong** -- binary logging is the MySQL mechanism. On PostgreSQL the equivalent is write-ahead logging, so the option is describing the wrong engine.
- **C) is wrong** -- daily exports give an RPO of up to 24 hours, far outside 5 minutes.
- **D) is wrong** -- HA protects against zonal failure only, and by itself defines neither RPO nor RTO for a regional loss.

**Exam tip:** match the log mechanism to the engine: binary logs for MySQL, write-ahead logs for PostgreSQL, transaction logs for SQL Server. An option that names the wrong one is wrong regardless of how good the rest of it sounds. And remember HA is zonal, replicas are regional.

Docs: https://cloud.google.com/sql/docs/postgres/backup-recovery/pitr and https://cloud.google.com/sql/docs/postgres/replication/cross-region-replicas
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

A locked retention policy makes objects undeletable and unmodifiable until the retention period expires, and locking is irreversible, which is what regulatory immutability means. Standard class serves the frequent reads of the first six months, and a lifecycle rule moves objects to Coldline afterwards.

- **B) is wrong** -- Object Versioning preserves earlier versions but does not stop someone deleting every version. It is a recovery feature, not an immutability control.
- **C) is wrong** -- Persistent Disk snapshots are not an archival product and are expensive at 10 TB per week.
- **D) is wrong** -- Archive from day one means paying retrieval charges throughout the six months of heavy reading.

**Exam tip:** immutability is retention policy plus lock, and nothing else. Versioning, IAM deny and ACLs can all be undone by someone with enough privilege; a locked retention policy cannot be shortened or removed by anyone, including the organisation admin.

Docs: https://cloud.google.com/storage/docs/bucket-lock and https://cloud.google.com/storage/docs/lifecycle
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

BigQuery applies long-term storage pricing automatically to any table or partition not modified for 90 consecutive days, roughly halving the storage rate. Partitioning by date means old partitions age into that rate on their own, with no pipeline, no data movement and no change to query behaviour.

- **A) is wrong** -- a 90-day expiration deletes data that must be kept for three years.
- **C) is wrong** -- exporting to Parquet and deleting from BigQuery adds a pipeline to maintain and loses seamless querying of the history.
- **D) is wrong** -- BigQuery storage pricing does not vary enough by region to pay for the added complexity, and splitting datasets fragments queries.

**Exam tip:** long-term storage is automatic and per-partition, which is why partitioning is the answer rather than any manual tiering. The trap is the option that saves storage by deleting data the stem says must be retained; always check the retention requirement before optimising.

Docs: https://cloud.google.com/bigquery/docs/partitioned-tables and https://cloud.google.com/bigquery/pricing
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

A locked retention policy on the bucket prevents deletion or overwriting of any object until its retention period elapses, and once locked it cannot be shortened or removed by anyone, including project owners and storage admins.

- **A) is wrong** -- versioning keeps prior versions but does not prevent deleting the live version or purging all versions.
- **C) is wrong** -- IAM deny policies are strong, but an organisation administrator can amend them, so the guarantee is weaker than an irreversible retention lock.
- **D) is wrong** -- ACLs are editable by bucket owners and provide no immutability at all.

**Exam tip:** the phrase "including project owners" or "even administrators" is the tell for bucket lock. Anything an administrator can change is not the answer to a question about protection from privileged users.

Docs: https://cloud.google.com/storage/docs/bucket-lock
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

Bigtable replication between clusters in two regions is continuous and near real time, and a multi-cluster routing app profile makes the client library route to the nearest available cluster and fail over automatically. That gives near-zero RPO and a failover that needs no operator action.

- **A) is wrong** -- daily exports leave up to 24 hours of writes unprotected.
- **C) is wrong** -- manual backup and restore inserts human response time into both RPO and RTO.
- **D) is wrong** -- application-level dual writes push conflict resolution and consistency handling into the application, which is the complexity replication exists to remove.

**Exam tip:** on Bigtable, the app profile is the failover control, not the replication setting. Single-cluster routing pins traffic to one cluster for read-your-writes consistency; multi-cluster routing gives automatic failover but eventual consistency. The stem's RPO and RTO tell you which to pick.

Docs: https://cloud.google.com/bigtable/docs/replication-overview and https://cloud.google.com/bigtable/docs/app-profiles
</details>

---

### Q17. A startup wants to migrate 200 TB of on-premises data to Cloud Storage. Their internet bandwidth is 1 Gbps. They need the data migrated within 30 days. Which transfer method should you recommend?

A) Use gsutil rsync over the internet
B) Use Transfer Appliance
C) Use Storage Transfer Service with a scheduled transfer
D) Use a Dedicated Interconnect provisioned for the migration

<details>
<summary>Answer</summary>

**Correct: B)**

At 1 Gbps, 200 TB takes roughly 18.5 days at 100% link utilisation (200 TB x 8 bits / 1 Gbps / 86400 seconds), and no production link sustains 100%. Realistically this overruns 30 days while saturating the company's only internet connection for a month. Transfer Appliance is a physical device shipped to your data centre, loaded, and shipped back; the TA300 model holds 300 TB, and the full round trip plus ingest fits inside 30 days.

- **A) is wrong** -- gsutil hits the same internet bandwidth ceiling, and `gcloud storage` has superseded gsutil as the current CLI surface.
- **C) is wrong** -- Storage Transfer Service also moves data over the network, so it runs into the identical constraint.
- **D) is wrong** -- Dedicated Interconnect takes weeks to provision and is heavy capital work for a one-time move.

**Exam tip:** do the bandwidth arithmetic before picking. Divide the data volume by the link rate, then assume you only get 50-70% of nominal throughput. If the result is a meaningful fraction of the deadline, the answer is Transfer Appliance. Current models are TA40 (40 TB) and TA300 (300 TB).

Docs: https://cloud.google.com/transfer-appliance/docs and https://cloud.google.com/storage-transfer/docs/overview
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

A backend bucket behind the global external Application Load Balancer lets Cloud CDN cache the objects at Google's edge. With a 100:1 read-to-write ratio and content that rarely changes, hit rates are high, which cuts both latency and origin egress.

- **A) is wrong** -- Autoclass moves objects between storage classes based on access. It changes storage cost, not serving latency or egress.
- **C) is wrong** -- per-region buckets means building and running replication, and still no edge caching.
- **D) is wrong** -- Requester Pays moves the bill to the caller. Total egress and latency are unchanged.

**Exam tip:** separate the three cost levers on Cloud Storage. Storage class (and Autoclass) changes what you pay to keep bytes; CDN changes what you pay to serve them and how fast; Requester Pays changes who pays. A stem naming latency and egress together is a CDN question.

Docs: https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket and https://cloud.google.com/storage/docs/autoclass
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

For a workload running continuously for three years, committed use discounts give the deepest savings. Compute Engine CUDs reach up to 55% for most machine series and up to 70% for memory-optimized types, with the 3-year term discounting more than the 1-year term. Renewing 1-year commitments annually gives a smaller discount but keeps flexibility, so both are defensible cost strategies for predictable long-running workloads.

- **A) is wrong** -- sustained use discounts are automatic but capped at 30%, apply only to resources used for more than 25% of a billing month, and cannot be combined with a CUD.
- **C) is wrong** -- Spot VMs can be reclaimed at any time, which a continuously running workload cannot accept.
- **D) is wrong** -- preemptible VMs are the earlier generation of Spot, with the same interruption behaviour plus a 24-hour maximum runtime.

**Exam tip:** rank the discount mechanisms by what they demand of you. Spot demands interruption tolerance, CUDs demand a spend commitment, SUDs demand nothing and therefore pay least. "Runs continuously" rules out Spot immediately and points at a commitment.

Docs: https://cloud.google.com/docs/cuds and https://cloud.google.com/compute/docs/sustained-use-discounts
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

TPU v5p pods are Google's largest training accelerators, and the inter-chip interconnect between chips is what makes model and data parallelism efficient at 70 billion parameters. TensorFlow has first-class TPU support, so the framework is not a constraint here.

- **A) is wrong** -- A3 Mega with H100 GPUs is a strong training platform, but for very large TensorFlow models TPU pods generally win on price-performance at this scale.
- **B) is wrong** -- C3 high-cpu has no accelerators at all, which rules it out for LLM training.
- **D) is wrong** -- GKE Autopilot with GPU node pools adds orchestration without providing the TPU-class interconnect the parallelism strategy depends on.

**Exam tip:** for accelerator questions, read the framework and the interconnect requirement. TensorFlow or JAX at very large scale favours TPU pods; PyTorch with custom CUDA kernels forces NVIDIA GPUs. The interconnect, not the individual chip, is what decides large-model training performance.

Docs: https://cloud.google.com/tpu/docs/v5p and https://cloud.google.com/tpu/docs/intro-to-tpu
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

Cloud Run fits a containerised application with a clear daily traffic shape: with minimum instances at zero it costs nothing overnight, scales up automatically for business hours, and there is no cluster or VM image to maintain.

- **A) is wrong** -- a managed instance group means owning images, instance templates and scaling policies, which is more operational work than the stem allows.
- **B) is wrong** -- Autopilot removes node management but still asks you to run Kubernetes objects, and the cluster carries a baseline cost even when idle.
- **D) is wrong** -- Cloud Run functions target event-driven handlers rather than a full web application, with tighter limits on request handling.

**Exam tip:** the phrase "pay only for actual usage" plus "containers" is Cloud Run. Cloud Run functions is for single event handlers, GKE is for when you need Kubernetes primitives, and a MIG is for when you need the VM itself. Note the rename: Cloud Functions is now Cloud Run functions.

Docs: https://cloud.google.com/run/docs/about-instance-autoscaling and https://cloud.google.com/run/docs/overview/what-is-cloud-run
</details>

---

### Q22. A gaming company needs to run game servers that require consistent, high single-thread performance and low latency. The servers are stateful and each instance handles up to 64 concurrent player connections. Which Compute Engine machine family should you recommend?

A) E2 general-purpose (cost-optimized)
B) C2D compute-optimized
C) N2 general-purpose with sole-tenant nodes
D) M3 memory-optimized

<details>
<summary>Answer</summary>

**Correct: B)**

C2D is a compute-optimized machine family, built for sustained high per-core performance with all-core turbo frequencies. Game servers that depend on single-thread performance and low latency benefit directly from it.

- **A) is wrong** -- E2 uses shared-core and burstable configurations, so per-core performance is inconsistent by design.
- **C) is wrong** -- N2 on sole-tenant nodes buys dedicated physical hardware for licensing or compliance reasons. It does not raise per-core performance.
- **D) is wrong** -- M3 memory-optimized targets in-memory databases and SAP, trading per-core speed for memory capacity.

**Exam tip:** know which family is which, because the exam leans on it. Compute-optimized is H3, H4D, C2 and C2D. C3, C3D, C4, N2 and E2 are general-purpose despite C3/C4 sounding like the C2 line. M-series is memory-optimized. "Consistent high single-thread performance" means compute-optimized; "large in-memory dataset" means memory-optimized.

Docs: https://cloud.google.com/compute/docs/machine-resource and https://cloud.google.com/compute/docs/compute-optimized-machines
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

Two node pools let each workload get the reliability it needs. The API service runs on standard VMs; the interruption-tolerant background job runs on Spot VMs at up to 91% off. A taint on the Spot pool keeps API pods away, and a matching toleration on the job pods lets them land there.

- **A) is wrong** -- one pool of on-demand VMs forgoes the saving that the fault-tolerant workload could have taken.
- **C) is wrong** -- the stem specifies GKE Standard, and Autopilot gives less direct control over node pool shape.
- **D) is wrong** -- two clusters duplicate control planes and system workloads for a separation that node pools already provide.

**Exam tip:** taints and tolerations plus node affinity is the standard answer for putting different workload classes on different hardware in one cluster. Remember the direction: the taint goes on the node pool to repel pods, the toleration goes on the pod to allow it.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/spot-vms
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

Node auto-provisioning creates and removes node pools to match pending workloads, so the cluster can drop to zero worker nodes between daily runs. Spark executors are fault-tolerant, so putting them on Spot VMs takes the discount without risking the job, with the driver on a small on-demand pool.

- **A) is wrong** -- a floor of three nodes is paid for every hour of every day, including the many hours with no job running.
- **B) is wrong** -- Autopilot can run this, but gives less direct control over node selection and Spot configuration, which is where the saving comes from.
- **D) is wrong** -- a static 50-node pool is both idle-heavy and over-provisioned.

**Exam tip:** cluster autoscaler resizes node pools that already exist; node auto-provisioning creates the pools themselves. When a stem says workloads vary in shape or the cluster should reach zero between jobs, that difference is what the question is testing.

Docs: https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-provisioning and https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler
</details>

---

### Q25. Your company is migrating a monolithic application to GKE. The application needs to run across multiple clusters in different regions for high availability. The team needs centralized fleet management, consistent security policies, and service mesh capabilities across all clusters. Which solution should you recommend?

A) Deploy independent GKE Standard clusters in each region and manage them separately
B) Use GKE Autopilot clusters with Multi-Cluster Ingress
C) Use GKE Enterprise with fleet management, Config Sync, and Cloud Service Mesh
D) Deploy the application on Cloud Run in multiple regions instead of GKE

<details>
<summary>Answer</summary>

**Correct: C)**

GKE Enterprise (the product formerly sold as Anthos) gives fleet management for centralised visibility and control across clusters in several regions. Config Sync applies configuration and policy from Git across the whole fleet, and Cloud Service Mesh (formerly Anthos Service Mesh, built on Istio) supplies cross-cluster traffic management, observability and mTLS.

- **A) is wrong** -- independently managed clusters give neither central policy nor a shared mesh, which are two of the three requirements.
- **B) is wrong** -- Multi Cluster Ingress routes traffic across clusters but does not manage fleet-wide policy or provide a service mesh.
- **D) is wrong** -- Cloud Run is not a target for migrating a monolith that needs Kubernetes features.

**Exam tip:** the trio "fleet management, consistent policy, service mesh" is the GKE Enterprise fingerprint. Learn the current names, since older material uses the old ones: Anthos is GKE Enterprise, Anthos Config Management is Config Sync and Policy Controller, Anthos Service Mesh and Traffic Director are Cloud Service Mesh.

Docs: https://cloud.google.com/kubernetes-engine/fleet-management/docs and https://cloud.google.com/service-mesh/docs/overview
</details>

---

### Q26. You are designing a GKE architecture for a fintech company. They need to ensure that their Kubernetes workloads meet PCI-DSS compliance requirements. The cluster must provide workload isolation, binary authorization for container images, and network policy enforcement. Which configuration should you use?

A) GKE Autopilot with Binary Authorization enabled
B) GKE Standard with GKE Sandbox (gVisor), Binary Authorization, and Dataplane V2 network policies enabled
C) GKE Standard with default settings and namespace-level RBAC
D) Cloud Run with container image signing

<details>
<summary>Answer</summary>

**Correct: B)**

Option B is the only choice that explicitly addresses all three stated requirements. GKE Sandbox (gVisor) runs containers against a user-space kernel, which is the workload isolation PCI-DSS asks for in multi-tenant clusters. Binary Authorization gates deployment on signed, attested images. Dataplane V2 (Cilium and eBPF) enforces pod-to-pod network policy and is the recommended plugin; Calico is the legacy dataplane.

- **A) is wrong**, but for incompleteness rather than impossibility. Autopilot does support GKE Sandbox, Binary Authorization, and network policy through Dataplane V2 by default, but the option as written enables only Binary Authorization and says nothing about isolation or network policy, so it leaves two of the three requirements unanswered.
- **C) is wrong** -- default Standard settings with namespace RBAC provide none of the three controls. RBAC governs API access, not pod isolation or network traffic.
- **D) is wrong** -- Cloud Run supports Binary Authorization but offers neither Kubernetes network policy nor gVisor-level workload isolation.

**Exam tip:** be careful with "Autopilot cannot do X" reasoning. Autopilot supports GKE Sandbox, Spot Pods, GPUs, DaemonSets and network policy, and much older study material says otherwise. The genuine Standard-only cases are node-level access: custom sysctls, privileged DaemonSets, and specific node OS or kernel tuning.

Docs: https://cloud.google.com/kubernetes-engine/docs/resources/autopilot-standard-feature-comparison and https://cloud.google.com/kubernetes-engine/docs/concepts/sandbox-pods
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

Peak hours are known, so scheduled autoscaling pre-provisions capacity before the surge instead of chasing it, and reactive CPU-based autoscaling stays in place to absorb anything the schedule did not predict. That combination avoids the boot-plus-health-check delay that makes a pure reactive policy miss a 2-minute scaling target.

- **A) is wrong** -- CPU-based scaling alone still has to create, boot and health-check instances after load has already arrived.
- **B) is wrong** -- a custom application metric is more precise than CPU, but it is still reactive and does not remove the provisioning delay.
- **D) is wrong** -- predictive autoscaling extrapolates from history, so it does not cover a peak that is unusual or newly scheduled.

**Exam tip:** when a stem gives both a known schedule and a tight scale-up window, the answer combines scheduled and reactive scaling rather than choosing between them. Scheduling handles the predictable part; reactive scaling is the safety net for the rest.

Docs: https://cloud.google.com/compute/docs/autoscaler/scaling-schedules and https://cloud.google.com/compute/docs/autoscaler
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

Cloud Run gives zero infrastructure management, scaling that includes scaling to zero, and request-based billing. Direct VPC egress puts the service on a VPC subnet so it can reach the Cloud SQL private IP, and a high maximum instance count leaves room for the 10x seasonal peak.

- **B) is wrong** -- Autopilot still means running Kubernetes objects and does not bill per request.
- **C) is wrong** -- App Engine Flexible runs on VMs with per-hour billing and scales more slowly than Cloud Run.
- **D) is wrong** -- a managed instance group is infrastructure to manage and is billed by instance-time, not per request.

**Exam tip:** always set a maximum instance count on Cloud Run. Unbounded scaling turns a traffic spike into both a cost incident and a way to overwhelm whatever the service calls, and Cloud SQL connection limits are the usual downstream casualty.

Docs: https://cloud.google.com/run/docs/configuring/vpc-direct-vpc and https://cloud.google.com/run/docs/about-instance-autoscaling
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

SSD-backed Persistent Disk delivers the consistent sub-millisecond latency that random read and write patterns need, and ReadWriteOnce matches a volume attached to a single node, which is the normal shape for a stateful workload. GKE provisions it dynamically from a StorageClass through a PersistentVolumeClaim.

- **A) is wrong** -- pd-standard is HDD-backed, with latency an order of magnitude above the requirement.
- **C) is wrong** -- Filestore provides ReadWriteMany over NFS, which is a different access mode and slower for random I/O than SSD block storage.
- **D) is wrong** -- Cloud Storage FUSE presents object storage as files; latency is far higher and it suits sequential, read-heavy access.

**Exam tip:** the access mode is a strong filter on its own. ReadWriteOnce means block storage (Persistent Disk or Hyperdisk); ReadWriteMany means a shared file system (Filestore). Then let the latency requirement choose between HDD and SSD. Note that Hyperdisk is the current generation on newer machine families.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes and https://cloud.google.com/compute/docs/disks
</details>

---

### Q30. Your company needs to run 100 parallel rendering jobs. Each job requires 96 vCPUs, 360 GB of RAM, and 4 NVIDIA T4 GPUs, and runs for approximately 4 hours. Jobs can be checkpointed and restarted. What is the most cost-effective compute configuration?

A) 100 n2-highmem-96 instances with T4 GPUs attached, using on-demand pricing
B) 100 n1-custom instances with 4 T4 GPUs attached, using Spot VMs with checkpointing to Cloud Storage
C) GKE Autopilot with GPU node pools
D) 100 c2d-highcpu-112 instances without GPUs, using software rendering

<details>
<summary>Answer</summary>

**Correct: B)**

T4 GPUs attach to N1 machine types, so an N1 custom configuration is what actually satisfies the stated 96 vCPU / 360 GB / 4x T4 requirement. The decisive fact is that the jobs are checkpointable and restartable, which is the workload profile Spot VMs are priced for: up to 91% off on-demand, in exchange for reclamation at any time. Checkpointing to Cloud Storage means a preempted job resumes rather than restarts.

- **A) is wrong** -- it specifies the right hardware but pays on-demand rates for a workload that tolerates preemption, so it costs several times more for no benefit.
- **C) is wrong** -- Autopilot adds Kubernetes to what is a plain batch workload and gives less direct control over GPU and machine selection.
- **D) is wrong** -- dropping the GPUs entirely means software rendering, which runs far longer and costs more in CPU hours than the GPUs saved.

**Exam tip:** match the accelerator to the machine family before comparing prices. T4 and V100 attach to N1; A100 comes with A2; H100 with A3; L4 with G2. An option pairing an A2 machine with a T4 requirement is wrong on hardware alone, whatever its pricing story. Then look for "checkpointable", "fault-tolerant" or "can be restarted", which is the exam signalling Spot. Note that Spot VMs give no guaranteed notice period by default.

Docs: https://cloud.google.com/compute/docs/gpus and https://cloud.google.com/compute/docs/instances/spot
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

GKE Standard is required when workloads need node-level customisation such as custom sysctl settings, because Autopilot manages the nodes and does not permit kernel parameter changes. The limited Kubernetes expertise is a real constraint, but it does not change what is technically possible; the team should close that gap with training and managed tooling such as Config Sync.

- **A) is wrong** -- Autopilot is easier to operate but cannot satisfy the sysctl requirement, and requirements outrank convenience.
- **C) is wrong** -- Cloud Run cannot host 15 interconnected microservices with custom networking as effectively as GKE.
- **D) is wrong** -- the blocker is the kernel parameter change, not DaemonSets. Autopilot does run user DaemonSets; the allowlist people remember applies to privileged workloads, not to DaemonSets in general.

**Exam tip:** when a stem pairs a hard technical requirement with a soft team-capability preference, the hard requirement wins. For Autopilot the genuine exclusions are all node-level: custom sysctls, privileged access, and node OS or kernel tuning.

Docs: https://cloud.google.com/kubernetes-engine/docs/resources/autopilot-standard-feature-comparison and https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview
</details>

---

### Q32. You are designing a compute solution for a scientific simulation that requires 3 TB of RAM and 128 vCPUs. The simulation runs for 72 hours and the dataset must remain in memory throughout. Which machine type family should you use?

A) N2 general-purpose with extended memory
B) M3 memory-optimized
C) C3 compute-optimized
D) A3 accelerator-optimized

<details>
<summary>Answer</summary>

**Correct: B)**

The M-series is the memory-optimized family, built for workloads whose dataset must stay resident in RAM. `m3-ultramem-128` provides 128 vCPUs with roughly 3.9 TB of memory, which covers the stated 3 TB and 128 vCPU requirement in a single instance.

- **A) is wrong** -- N2 with extended memory allows custom memory ratios but tops out well below 3 TB per instance.
- **C) is wrong** -- C3 is general-purpose despite the name and is tuned for per-core throughput, not memory capacity.
- **D) is wrong** -- A3 accelerator-optimized exists to carry H100 GPUs; its system memory is sized to feed the accelerators, not to hold a multi-terabyte working set.

**Exam tip:** for memory questions, check the ceiling of the family you pick against the number in the stem. M3 reaches about 3.9 TB, M2 about 12 TB, M4 up to 6 TB with 224 vCPUs, and X4 goes higher still. If a scenario needs both very high memory and very high vCPU counts, M3 is often the wrong pick, because its vCPU count caps at 128.

Docs: https://cloud.google.com/compute/docs/memory-optimized-machines and https://cloud.google.com/compute/docs/machine-resource
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

A Spanner multi-region configuration replicates synchronously across regions and keeps serving reads and writes when one region fails, with no promotion step and therefore an effective RTO near zero. That is what a 5-minute RTO mandate needs.

- **A) is wrong** -- regional with daily backups means restoring from a backup, which is measured in hours.
- **C) is wrong** -- a cross-region restore procedure is manual and again bounded by restore time, not minutes.
- **D) is wrong** -- swapping to Cloud SQL changes the database rather than answering the availability requirement, and its cross-region replica still needs promotion.

**Exam tip:** Spanner has no failover step to configure, which is the whole point of a multi-region configuration. Regional Spanner carries a 99.99% SLA and multi-region 99.999%, so the availability number in a stem often selects the configuration on its own.

Docs: https://cloud.google.com/spanner/docs/instance-configurations and https://cloud.google.com/spanner/docs/replication
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

Vertex AI Training provisions and manages distributed training clusters, Vertex AI Vizier performs black-box hyperparameter optimisation, and Vertex AI Endpoints serve the resulting model with autoscaling. That is the full path from data to served model with no infrastructure to run. (The platform is now marketed as Gemini Enterprise Agent Platform; the exam guide still uses the Vertex AI names.)

- **A) is wrong** -- custom GPU VMs and a hand-built pipeline is exactly the infrastructure management the stem excludes.
- **C) is wrong** -- Kubeflow on GKE is flexible but means owning a Kubernetes platform.
- **D) is wrong** -- Cloud Run functions are not a training runtime and have execution time limits far below a training job.

**Exam tip:** in Vertex AI questions, map each named requirement to its component: distributed training to Training, hyperparameter tuning to Vizier, orchestration to Pipelines, serving to Endpoints, drift detection to Model Monitoring. The correct option usually names the component for every requirement in the stem.

Docs: https://cloud.google.com/vertex-ai/docs/training/overview and https://cloud.google.com/vertex-ai/docs/vizier/overview
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

Custom CUDA kernels are compiled for NVIDIA's architecture and call NVIDIA-specific APIs directly, so they cannot run on TPUs even through the PyTorch-XLA bridge. A2 VMs with A100 GPUs run the code unchanged and scale to the required accelerator count.

- **A) is wrong** -- "TPUs are always faster for transformers" is an overgeneralisation, and it ignores the CUDA dependency that decides this question.
- **C) is wrong** -- PyTorch-XLA covers standard PyTorch operations, not hand-written CUDA kernels.
- **D) is wrong** -- H100 GPUs would also run the kernels, but the question asks which factor decides, and that factor is CUDA compatibility rather than GPU generation.

**Exam tip:** a stated dependency on custom CUDA kernels eliminates TPUs outright. More generally, look for the one hard technical constraint in the stem and use it to eliminate first; performance comparisons between the survivors come second.

Docs: https://cloud.google.com/compute/docs/gpus and https://cloud.google.com/tpu/docs/intro-to-tpu
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

Vertex AI Pipelines runs pipelines authored with the Kubeflow Pipelines SDK on managed infrastructure. The pipeline is code, so it version-controls; runs record artifacts and lineage, so they reproduce; and conditional steps express "deploy only if the evaluation beats the baseline". Cloud Scheduler triggers the daily run.

- **A) is wrong** -- a scheduler plus a function plus a VM is hand-built orchestration with no artifact tracking or conditional pipeline semantics.
- **C) is wrong** -- a cron-driven bash script is neither reproducible nor managed.
- **D) is wrong** -- AutoML with scheduled retraining does not express custom conditional deployment logic.

**Exam tip:** "reproducible", "version-controlled" and "conditional deployment" together describe a pipeline product, not a scheduler. On Google Cloud that is Vertex AI Pipelines for ML work and Cloud Composer for general data orchestration.

Docs: https://cloud.google.com/vertex-ai/docs/pipelines/introduction
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

AI Hypercomputer with TPU v5p multislice lets a single training job span many pods, with high-bandwidth inter-chip interconnect inside a slice and efficient communication across slices. That is what makes thousands of accelerators usable on one 175-billion-parameter job.

- **A) is wrong** -- A2 VMs communicating over standard VPC networking makes the network the bottleneck long before the accelerators are saturated.
- **C) is wrong** -- eight H100s in one VM is orders of magnitude short of the required scale.
- **D) is wrong** -- spreading a tightly coupled training job across regions adds latency that destroys distributed training throughput.

**Exam tip:** for very large training jobs the interconnect is the answer, not the chip. Any option that scales out over ordinary VPC networking or across regions is wrong on communication cost alone. Multislice is the term for scaling a single job beyond one TPU pod.

Docs: https://cloud.google.com/tpu/docs/multislice-introduction and https://cloud.google.com/tpu/docs/intro-to-tpu
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

Model Garden is the curated catalogue of first-party, open-weight and third-party models, with model cards for comparison and one-click deployment. It is built for exactly this evaluate-before-committing stage, without standing up each model separately.

- **B) is wrong** -- AutoML trains a custom model on your data. It is not a catalogue of pre-trained foundation models.
- **C) is wrong** -- the Natural Language API is a prebuilt API for specific NLP tasks, not a model exploration surface.
- **D) is wrong** -- BigQuery ML runs models inside BigQuery but offers no foundation model catalogue to compare.

**Exam tip:** map the generative AI verbs to products. Browse and compare models is Model Garden. Ground answers in your documents is Agent Builder. Teach house style from examples is fine-tuning. Task-specific prebuilt capability is one of the pretrained APIs such as Vision, Natural Language or Document AI.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models
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

Agent Builder with a data store ingests the PDFs and HTML and answers questions using retrieval-augmented generation, so responses are grounded in the documentation and carry source citations. No ML expertise and no training is required.

- **A) is wrong** -- fine-tuning teaches style and format, not facts, and it needs ML expertise for a documentation Q&A use case that RAG solves directly.
- **C) is wrong** -- the Natural Language API extracts entities and sentiment. It does not hold a grounded conversation.
- **D) is wrong** -- training a custom BERT model is substantial ML work for functionality that is available out of the box.

**Exam tip:** citations are the tell. Only retrieval-based grounding can point at the source document, so any stem asking for cited or verifiable answers is a RAG question and therefore Agent Builder with a data store, never fine-tuning.

Docs: https://cloud.google.com/generative-ai-app-builder/docs/introduction and https://cloud.google.com/vertex-ai/generative-ai/docs/agent-builder/overview
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

The Cloud Vision API is a prebuilt service that returns labels with confidence scores for uploaded images. It needs no training data and no ML team, and integrating a REST call fits comfortably inside a one-week deadline.

- **A) is wrong** -- AutoML Vision needs a collected and labelled training dataset plus training time, which the deadline does not allow.
- **C) is wrong** -- deploying a custom ResNet means model selection, optimisation and serving decisions the team has no one to make.
- **D) is wrong** -- Gemini can describe images, but it returns natural language rather than structured labels with confidence scores, and costs more per request for this task.

**Exam tip:** the pretrained APIs (Vision, Natural Language, Speech, Translation, Document AI) are the answer whenever the stem combines a short deadline, no ML team, and a common task. AutoML enters only when the domain is specialised enough that generic labels will not do, and the stem gives you labelled data.

Docs: https://cloud.google.com/vision/docs
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

Document AI ships specialised processors for invoices, receipts and contracts that return structured fields (line items, totals, dates, parties, clauses) rather than raw text, and it handles PDFs, images and scans with OCR built in.

- **A) is wrong** -- Vision API OCR returns the text on the page. It does not know which number is the invoice total.
- **C) is wrong** -- AutoML would need labelled training data and training time for something Document AI already does out of the box.
- **D) is wrong** -- BigQuery ML has no document parsing capability.

**Exam tip:** the distinction to hold is OCR versus document understanding. If the stem only needs text off a page, Vision API OCR is enough; if it needs named fields extracted from a known document type, it is Document AI. The document type being named in the stem is usually the giveaway.

Docs: https://cloud.google.com/document-ai/docs
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

Ten thousand examples of the company's own writing is a fine-tuning dataset. Fine-tuning adjusts the model toward that voice and style far more consistently than prompt instructions can, and Vertex AI provides the managed tuning infrastructure.

- **A) is wrong** -- a system prompt can approximate a house style, but it will not match it as reliably as tuning on 10,000 real examples.
- **C) is wrong** -- Agent Builder retrieves and answers from documents. It is not a mechanism for generating new copy in a learned style.
- **D) is wrong** -- the Translation API converts between languages, not between writing styles.

**Exam tip:** the fine-tuning versus RAG decision comes down to style versus facts. Style, tone, format or a domain-specific output shape means fine-tuning; current, citable, changing information means retrieval. A stem that hands you thousands of curated examples is pointing at fine-tuning.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/models/tune-models
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

Safety filters run on every prompt and response, classifying content across categories such as harassment, hate speech, sexually explicit and dangerous content, and blocking at a configured threshold. Vertex AI Evaluation benchmarks model outputs against test datasets before deployment, covering quality, safety and bias. Together they give a pre-deployment gate and a runtime control.

- **A) is wrong** -- Model Monitoring tracks feature and prediction drift for tabular and custom models. It is not a content safety mechanism.
- **D) is wrong** -- audit logs record who called the API and when. They do not inspect the content of prompts or responses.
- **E) is wrong** -- VPC Service Controls restricts which networks and projects can reach the API. It says nothing about what the model produces.

**Exam tip:** separate access control from content control. Audit logs, IAM and VPC Service Controls answer "who can call it"; safety filters and evaluation answer "what comes out". A stem naming safety, bias or policy compliance is asking the second question. Note that Model Armor, the prompt and response screening product, ships under Security Command Center.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/configure-safety-filters and https://cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-overview
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

AutoML Vision trains a custom classifier on the 50,000 labelled images and deploys it to an endpoint inside the customer's own project, so the data and the model stay within the project boundary the regulator cares about, and the model is specialised to the imaging domain.

- **A) is wrong** -- Vision API is a shared prebuilt service, not specialised for medical imaging, and it sends data to a shared Google endpoint.
- **C) is wrong** -- Gemini is general-purpose rather than tuned for medical anomaly detection, and it does not satisfy the stated isolation requirement.
- **D) is wrong** -- a pre-trained model with no fine-tuning will not reach the accuracy a medical imaging task needs.

**Exam tip:** two constraints drive this. A stated data-residency or isolation requirement rules out shared prebuilt APIs and forces a model deployed in your own project. A substantial labelled dataset plus a specialised domain is what justifies AutoML or custom training over a general model.

Docs: https://cloud.google.com/vertex-ai/docs/image-data/classification/train-model and https://cloud.google.com/vertex-ai/docs/predictions/overview
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

Vertex AI Model Monitoring watches deployed models for feature drift and prediction drift and raises alerts when the configured thresholds are crossed. Routing those alerts through a Cloud Monitoring notification channel to a function that starts the retraining pipeline closes the loop without a human in it.

- **A) is wrong** -- weekly manual review is slow and does not meet the stated requirement for an automated system.
- **C) is wrong** -- unconditional daily retraining burns compute when nothing has changed and may still lag a sharp shift in fraud patterns.
- **D) is wrong** -- randomly splitting traffic between versions neither detects degradation nor corrects it, and can serve predictions from a stale model.

**Exam tip:** the MLOps closed loop is monitor, alert, retrain, redeploy. Model Monitoring is the detection half; the automation half is an alert-driven trigger into a pipeline. An option that only detects, or only retrains on a fixed schedule, is answering half the question.

Docs: https://cloud.google.com/vertex-ai/docs/model-monitoring/overview and https://cloud.google.com/vertex-ai/docs/pipelines/introduction
</details>

---
