# GCP ACE - Answered Practice Questions

---

## Q1: Auditors Need BigQuery View Access

**Question:** Your organization plans to migrate its financial transaction monitoring application to Google Cloud. Auditors need to view the data and run reports in BigQuery, but they are not allowed to perform transactions in the application. You are leading the migration and want the simplest solution that will require the least amount of maintenance. What should you do?

- A. Assign `roles/bigquery.dataViewer` to the individual auditors.
- B. Create a group for auditors and assign `roles/viewer` to them.
- C. Create a group for auditors, and assign `roles/bigquery.dataViewer` to them.
- D. Assign a custom role to each auditor that allows view-only access to BigQuery.

**Answer: C**

**Explanation:** Two principles are at play -- **least privilege** and **least maintenance**. Option C satisfies both. Using a **group** means you manage membership once instead of per-user (eliminates A and D for maintenance). `roles/bigquery.dataViewer` is a predefined role scoped specifically to BigQuery read access (eliminates B, where `roles/viewer` is a basic role granting read access to *all* resources, far too broad). Custom roles (D) require ongoing maintenance when permissions change.

> **Exam tip:** Whenever you see "least maintenance" + IAM, think **groups + predefined roles**.

---

## Q2: First Project with Sensitive Information

**Question:** You are managing your company's first Google Cloud project. Project leads, developers, and internal testers will participate in the project, which includes sensitive information. You need to ensure that only specific members of the development team have access to sensitive information. You want to assign the appropriate IAM roles that also require the least amount of maintenance. What should you do?

- A. Assign a basic role to each user.
- B. Create groups. Assign a basic role to each group, and then assign users to groups.
- C. Create groups. Assign a Custom role to each group, including those who should have access to sensitive data. Assign users to groups.
- D. Create groups. Assign an IAM Predefined role to each group as required, including those who should have access to sensitive data. Assign users to groups.

**Answer: D**

**Explanation:** Groups are essential for maintainability (eliminates A). Basic roles (B) are too broad and cannot differentiate access to sensitive data -- `roles/editor` and `roles/viewer` apply to all resources. Custom roles (C) work but add maintenance overhead (you must update permissions manually when Google adds new features). **Predefined roles** (D) are maintained by Google, scoped to specific services, and allow you to grant different levels of access to different groups (e.g., one group gets `bigquery.dataViewer`, another gets `bigquery.dataEditor`).

> **Exam tip:** Google best practice hierarchy: **Predefined roles > Custom roles > Basic roles**. Only use custom roles when no predefined role fits.

---

## Q3: Monitor Cloud Storage and Firestore Changes

**Question:** You are responsible for monitoring all changes in your Cloud Storage and Firestore instances. For each change, you need to invoke an action that will verify the compliance of the change in near real time. You want to accomplish this with minimal setup. What should you do?

- A. Use the trigger mechanism in each datastore to invoke the security script.
- B. Use Cloud Function events, and call the security script from the Cloud Function triggers.
- C. Redirect your data-changing queries to an App Engine application, and call the security script from the application.
- D. Use a Python script to get logs of the datastores, analyze them, and invoke the security script.

**Answer: B**

**Explanation:** Cloud Functions natively supports **event-driven triggers** for both Cloud Storage (object finalize, delete, metadata update) and Firestore (document create, update, delete, write). This gives you near real-time invocation with minimal setup -- just deploy a function per trigger. Option A is vague and Cloud Storage doesn't have a built-in "trigger mechanism." Option C requires rearchitecting the application. Option D is polling-based, not real-time, and requires maintaining a script.

> **Exam tip:** Cloud Functions (2nd gen) uses Eventarc under the hood. For "near real-time" + "minimal setup" + event-driven, Cloud Functions is almost always the answer.

---

## Q4: High Transaction Rate Across Multiple Servers

**Question:** Your application needs to process a significant rate of transactions. The rate of transactions exceeds the processing capabilities of a single VM. You want to spread transactions across multiple servers in real time and in the most cost-effective manner. What should you do?

- A. Send transactions to BigQuery. On the VMs, poll for transactions that do not have the 'processed' key, and mark them 'processed' when done.
- B. Set up Cloud SQL with a memory cache for speed. On your multiple servers, poll for transactions that do not have the 'processed' key, and mark them 'processed' when done.
- C. Send transactions to Pub/Sub. Process them in VMs in a managed instance group.
- D. Record transactions in Cloud Bigtable, and poll for new transactions from the VMs.

**Answer: C**

**Explanation:** **Pub/Sub + MIG** is the classic pattern for distributing work across multiple consumers. Pub/Sub provides reliable message delivery with at-least-once semantics, decouples producers from consumers, and handles backpressure automatically. The MIG can autoscale based on the Pub/Sub backlog metric (`pubsub.googleapis.com/subscription/num_undelivered_messages`). Options A, B, and D all use **polling**, which is inefficient, adds latency, and wastes resources.

> **Exam tip:** "Distribute work across multiple servers in real time" = **Pub/Sub**. Polling a database is never the right answer for real-time distribution.

---

## Q5: Connect On-Premises to VPC

**Question:** Your team needs to directly connect your on-premises resources to several virtual machines inside a VPC. You want to provide your team with fast and secure access to the VMs with minimal maintenance and cost. What should you do?

- A. Set up Cloud Interconnect.
- B. Use Cloud VPN to create a bridge between the VPC and your network.
- C. Assign a public IP address to each VM, and assign a strong password to each one.
- D. Start a Compute Engine VM, install a software router, and create a direct tunnel to each VM.

**Answer: B**

**Explanation:** **Cloud VPN** provides encrypted IPsec tunnels between on-premises and GCP at a fraction of the cost of Cloud Interconnect. It's managed by Google (minimal maintenance) and secure by default. Cloud Interconnect (A) provides higher bandwidth but is significantly more expensive and complex to set up -- overkill when the question asks for minimal cost. Public IPs with passwords (C) are insecure. A software router VM (D) is high maintenance.

> **Exam tip:** Cost ranking: **Cloud VPN < Partner Interconnect < Dedicated Interconnect**. Choose VPN when the question emphasizes "minimal cost" or doesn't require high bandwidth (>10 Gbps).

---

## Q6: Cloud Storage Lifecycle Policies

**Question:** You are implementing Cloud Storage for your organization. You need to follow your organization's regulations: 1) Archive data older than one year. 2) Delete data older than 5 years. 3) Use standard storage for all other data. You want to implement these guidelines automatically and in the simplest manner available. What should you do?

- A. Set up Object Lifecycle management policies.
- B. Run a script daily. Copy data that is older than one year to an archival bucket, and delete five-year-old data.
- C. Run a script daily. Set storage class to ARCHIVE for data that is older than one year, and delete five-year-old data.
- D. Set up default storage class for three buckets named: STANDARD, ARCHIVE, DELETED. Use a script to move the data in the appropriate bucket when its condition matches your company guidelines.

**Answer: A**

**Explanation:** **Object Lifecycle Management** is a built-in Cloud Storage feature that automatically transitions storage classes and deletes objects based on conditions like age. You define rules (e.g., `SetStorageClass` to ARCHIVE after 365 days, `Delete` after 1825 days) and Google handles execution. No scripts, no cron jobs, no maintenance. Options B, C, and D all require custom scripts that must be maintained, monitored, and are error-prone.

> **Exam tip:** Whenever you see "automatically" + Cloud Storage class transitions or deletion based on age, the answer is **Object Lifecycle Management**.

---

## Q7: IoT Application with 10 PB of Data

**Question:** You are creating a Cloud IoT application requiring data storage of up to 10 PB. The application must support high-speed reads and writes of small pieces of data, but your data schema is simple. You want to use the most economical solution for data storage. What should you do?

- A. Store the data in Cloud Spanner, and add an in-memory cache for speed.
- B. Store the data in Cloud Storage, and distribute the data through Cloud CDN for speed.
- C. Store the data in Cloud Bigtable, and implement the business logic in the programming language of your choice.
- D. Use BigQuery, and implement the business logic in SQL.

**Answer: C**

**Explanation:** **Cloud Bigtable** is purpose-built for this exact use case: IoT data, petabyte scale, high-throughput reads/writes of small records, simple key-value/wide-column schema. It handles millions of operations per second at single-digit millisecond latency. Cloud Spanner (A) is far more expensive and designed for relational/transactional workloads. Cloud Storage (B) is object storage, not suitable for high-speed small random reads/writes. BigQuery (D) is an analytics warehouse optimized for large scans, not high-speed point writes.

> **Exam tip:** IoT + high throughput + simple schema + petabyte scale = **Cloud Bigtable**. Every time.

---

## Q8: GKE Frontend-Backend Communication

**Question:** You have created a Kubernetes deployment on GKE that has a backend service. You also have pods that run the frontend service. You want to ensure that there is no interruption in communication between your frontend and backend service pods if they are moved or restarted. What should you do?

- A. Create a service that groups your pods in the backend service, and tell your frontend pods to communicate through that service.
- B. Create a DNS entry with a fixed IP address that the frontend service can use to reach the backend service.
- C. Assign static internal IP addresses that the frontend service can use to reach the backend pods.
- D. Assign static external IP addresses that the frontend service can use to reach the backend pods.

**Answer: A**

**Explanation:** A **Kubernetes Service** (e.g., `ClusterIP`) provides a stable virtual IP and DNS name that automatically routes traffic to healthy backend pods via label selectors. When pods restart or get rescheduled, the Service updates its endpoints automatically -- zero interruption. Static IPs (C, D) don't work because pods get new IPs on restart and you can't assign static IPs to pods. A manual DNS entry (B) wouldn't update automatically when pods move.

> **Exam tip:** In Kubernetes, **Services** are the answer to stable communication between pods. `ClusterIP` for internal, `LoadBalancer` or `Ingress` for external.

---

## Q9: Docker Microservices with Variable Load

**Question:** You are responsible for the user-management service for your global company. The service will add, update, delete, and list addresses. Each of these operations is implemented by a Docker container microservice. The processing load can vary from low to very high. You want to deploy the service on Google Cloud for scalability and minimal administration. What should you do?

- A. Deploy your Docker containers into Cloud Run.
- B. Start each Docker container as a managed instance group.
- C. Deploy your Docker containers into Google Kubernetes Engine.
- D. Combine the four microservices into one Docker image, and deploy it to the App Engine instance.

**Answer: A**

**Explanation:** **Cloud Run** is serverless, runs Docker containers directly, auto-scales from zero to thousands of instances, and requires zero cluster management. It's the lowest-admin option for containerized microservices with variable load. GKE (C) works but requires managing a cluster -- more admin overhead. MIGs (B) require managing VMs. Combining microservices into one image (D) defeats the purpose of microservices architecture and App Engine isn't the right fit for Docker containers.

> **Exam tip:** "Docker containers" + "minimal administration" + "variable load" = **Cloud Run**. If the question adds "complex orchestration" or "stateful workloads," that's when GKE becomes the answer.

---

## Q10: Static IP for Partner-Facing Service

**Question:** You provide a service that you need to open to everyone in your partner network. You have a server and an IP address where the application is located. You do not want to have to change the IP address on your DNS server if your server crashes or is replaced. You also want to avoid downtime and deliver a solution for minimal cost and setup. What should you do?

- A. Create a script that updates the IP address for the domain when the server crashes or is replaced.
- B. Reserve a static internal IP address, and assign it using Cloud DNS.
- C. Reserve a static external IP address, and assign it using Cloud DNS.
- D. Use the Bring Your Own IP (BYOIP) method to use your own IP address.

**Answer: C**

**Explanation:** A **reserved static external IP** persists even when the VM is deleted and recreated. You assign it to your replacement VM and the DNS record (in Cloud DNS) stays the same. Partners always reach the same IP. Internal IP (B) won't work because partners are external. A script (A) introduces downtime and complexity. BYOIP (D) is for bringing existing IP ranges to Google Cloud -- overcomplicated for this scenario.

> **Exam tip:** "Don't want to change IP if server is replaced" = **reserve a static external IP**. Static IPs can be reassigned to new VMs.

---

## Q11: Consistent Dev/Test/Prod Environments

**Question:** Your team is building the development, test, and production environments for your project deployment in Google Cloud. You need to efficiently deploy and manage these environments and ensure that they are consistent. You want to follow Google-recommended practices. What should you do?

- A. Create a Cloud Shell script that uses gcloud commands to deploy the environments.
- B. Create one Terraform configuration for all environments. Parameterize the differences between environments.
- C. For each environment, create a Terraform configuration. Use them for repeated deployment. Reconcile the templates periodically.
- D. Use the Cloud Foundation Toolkit to create one deployment template that will work for all environments, and deploy with Terraform.

**Answer: D**

**Explanation:** The **Cloud Foundation Toolkit (CFT)** provides Google-opinionated, production-ready Terraform modules that follow Google's best practices. Using CFT with one template and deploying with Terraform ensures consistency across environments and follows Google-recommended practices. Option B is good IaC practice but doesn't leverage Google's best-practice templates. Option C creates drift risk with separate configs. Option A uses imperative scripting rather than declarative IaC.

> **Exam tip:** "Google-recommended practices" + infrastructure deployment = **Cloud Foundation Toolkit + Terraform**. CFT is Google's answer to "how should I set up my GCP environments."

---

## Q12: IP Range Exhausted in Subnet

**Question:** You receive an error message when you try to start a new VM: "You have exhausted the IP range in your subnet." You want to resolve the error with the least amount of effort. What should you do?

- A. Create a new subnet and start your VM there.
- B. Expand the CIDR range in your subnet, and restart the VM that issued the error.
- C. Create another subnet, and move several existing VMs into the new subnet.
- D. Restart the VM using exponential backoff until the VM starts successfully.

**Answer: B**

**Explanation:** GCP allows you to **expand a subnet's CIDR range** without downtime or disruption to existing VMs. This is a single `gcloud compute networks subnets expand-ip-range` command -- least effort. Creating a new subnet (A) works but is more effort (you may need to update firewall rules, routes, etc.). Moving VMs (C) is even more effort. Exponential backoff (D) won't help -- the IPs are exhausted, retrying won't free them.

> **Exam tip:** GCP subnets can be **expanded but never shrunk**. This is a frequently tested fact. `expand-ip-range` is the command.

---

## Q13: Expose Applications Through DNS Names

**Question:** You are running several related applications on Compute Engine VM instances. You want to follow Google-recommended practices and expose each application through a DNS name. What should you do?

- A. Use the Compute Engine internal DNS service to assign DNS names to your VM instances, and make the names known to your users.
- B. Assign each VM instance an alias IP address range, and then make the internal DNS names public.
- C. Assign Google Cloud routes to your VM instances, assign DNS names to the routes, and make the DNS names public.
- D. Use Cloud DNS to translate your domain names into your IP addresses.

**Answer: D**

**Explanation:** **Cloud DNS** is Google's managed, authoritative DNS service designed for publishing DNS records. You create a managed zone, add A records mapping your domain names to your VM IPs, and it handles global DNS resolution. Internal DNS (A) is auto-generated and intended for within-VPC communication, not for exposing apps externally. Alias IPs (B) are for assigning multiple IPs to a single NIC, not for DNS. Routes (C) are for network routing, not DNS.

> **Exam tip:** "Expose through DNS" + "Google-recommended" = **Cloud DNS**. Internal DNS (`*.internal`) is for VPC-internal resolution only.

---

## Q14: Optimize Resource Consumption Analysis

**Question:** You are charged with optimizing Google Cloud resource consumption. Specifically, you need to investigate the resource consumption charges and present a summary of your findings. You want to do it in the most efficient way possible. What should you do?

- A. Rename resources to reflect the owner and purpose. Write a Python script to analyze resource consumption.
- B. Attach labels to resources to reflect the owner and purpose. Export Cloud Billing data into BigQuery, and analyze it with Data Studio.
- C. Assign tags to resources to reflect the owner and purpose. Export Cloud Billing data into BigQuery, and analyze it with Data Studio.
- D. Create a script to analyze resource usage based on the project to which the resources belong. In this script, use the IAM accounts and services accounts that control given resources.

**Answer: B**

**Explanation:** **Labels** are key-value pairs designed for organizing resources and attributing costs in billing. Billing export to BigQuery captures label data, and Data Studio (now Looker Studio) provides visualization. This is the standard Google-recommended workflow for cost analysis. **Tags** (C) are for firewall rules and network policies, not billing attribution -- this is a common exam trap. Renaming resources (A) is disruptive. Custom scripts (D) are inefficient and hard to maintain.

> **Exam tip:** **Labels** = billing, organization, cost attribution. **Tags** (network tags) = firewall rules. Don't confuse them -- the exam tests this distinction.

---

## Q15: Ad Hoc SQL on Large Data, Cost-Effective

**Question:** You are creating an environment for researchers to run ad hoc SQL queries. The researchers work with large quantities of data. Although they will use the environment for an hour a day on average, the researchers need access to the functional environment at any time during the day. You need to deliver a cost-effective solution. What should you do?

- A. Store the data in Cloud Bigtable, and run SQL queries provided by Bigtable schema.
- B. Store the data in BigQuery, and run SQL queries in BigQuery.
- C. Create a Dataproc cluster, store the data in HDFS storage, and run SQL queries in Spark.
- D. Create a Dataproc cluster, store the data in Cloud Storage, and run SQL queries in Spark.

**Answer: B**

**Explanation:** **BigQuery** is serverless, always available, and uses on-demand pricing (pay per query, not per hour). If researchers only query for ~1 hour/day, you only pay for those queries. With Dataproc (C, D), you'd either pay for an idle cluster 23 hours/day or deal with startup delays when spinning up on demand. Bigtable (A) doesn't support SQL natively. BigQuery is purpose-built for ad hoc SQL analytics on large datasets.

> **Exam tip:** "Ad hoc SQL" + "large data" + "cost-effective" + "intermittent usage" = **BigQuery**. Its serverless, pay-per-query model beats always-on clusters for sporadic workloads.

---

## Q16: Migrate to GKE and Minimize Costs

**Question:** You are migrating your workload from on-premises deployment to Google Kubernetes Engine (GKE). You want to minimize costs and stay within budget. What should you do?

- A. Configure Autopilot in GKE to monitor node utilization and eliminate idle nodes.
- B. Configure the needed capacity; the sustained use discount will make you stay within budget.
- C. Scale individual nodes up and down with the Horizontal Pod Autoscaler.
- D. Create several nodes using Compute Engine, add them to a managed instance group, and set the group to scale up and down depending on load.

**Answer: A**

**Explanation:** **GKE Autopilot** is Google's fully managed Kubernetes mode where you pay per pod resource request, not per node. Google manages the nodes, optimizes bin-packing, and eliminates idle node costs. This directly minimizes costs. Option B assumes sustained use discounts alone will keep you in budget -- passive, not active cost management. HPA (C) scales pods, not nodes -- it won't reduce infrastructure costs if nodes remain idle. Option D bypasses GKE entirely and manually recreates orchestration.

> **Exam tip:** GKE Autopilot = pay per pod, Google manages nodes. GKE Standard = you manage nodes, pay per node. For cost optimization questions, Autopilot is usually the answer.

---

## Q17: Upload, Convert, and Store Pictures

**Question:** Your application allows users to upload pictures. You need to convert each picture to your internal optimized binary format and store it. You want to use the most efficient, cost-effective solution. What should you do?

- A. Store uploaded files in Cloud Bigtable, monitor Bigtable entries, and then run a Cloud Function to convert the files and store them in Bigtable.
- B. Store uploaded files in Firestore, monitor Firestore entries, and then run a Cloud Function to convert the files and store them in Firestore.
- C. Store uploaded files in Filestore, monitor Filestore entries, and then run a Cloud Function to convert the files and store them in Filestore.
- D. Save uploaded files in a Cloud Storage bucket, and monitor the bucket for uploads. Run a Cloud Function to convert the files and to store them in a Cloud Storage bucket.

**Answer: D**

**Explanation:** **Cloud Storage** is designed for storing files (objects) like images, and Cloud Functions has a native event trigger for Cloud Storage (`google.storage.object.finalize`). This is the standard event-driven file processing pattern: upload triggers function, function processes file, function writes result back. Bigtable (A) and Firestore (B) are databases, not file storage. Filestore (C) is an NFS file server -- overkill and more expensive for this use case.

> **Exam tip:** File upload + processing = **Cloud Storage + Cloud Function trigger**. This is one of the most classic GCP patterns.

---

## Q18: Migrate 100 TB to Google Cloud

**Question:** You are migrating your on-premises solution to Google Cloud. As a first step, the new cloud solution will need to ingest 100 TB of data. Your daily uploads will be within your current bandwidth limit of 100 Mbps. You want to follow Google-recommended practices for the most cost-effective way to implement the migration. What should you do?

- A. Set up Partner Interconnect for the duration of the first upload.
- B. Obtain a Transfer Appliance, copy the data to it, and ship it to Google.
- C. Set up Dedicated Interconnect for the duration of your first upload, and then drop back to regular bandwidth.
- D. Divide your data between 100 computers, and upload each data portion to a bucket. Then run a script to merge the uploads together.

**Answer: B**

**Explanation:** At 100 Mbps, uploading 100 TB would take approximately **93 days** (100 TB = 800,000 Gb / 100 Mbps = 8,000,000 seconds). A **Transfer Appliance** is a physical device Google ships to you -- you load your data and ship it back. It's designed for exactly this scenario: large one-time transfers where network bandwidth is insufficient. Interconnect options (A, C) are expensive recurring commitments for a one-time transfer. Option D doesn't increase total bandwidth.

> **Exam tip:** Use this rule of thumb: if transferring over the network would take **weeks or more**, use a **Transfer Appliance**. For ongoing high-bandwidth needs, use Interconnect.

---

## Q19: Prevent Excessive Resource Consumption

**Question:** You are setting up billing for your project. You want to prevent excessive consumption of resources due to an error or malicious attack and prevent billing spikes or surprises. What should you do?

- A. Set up budgets and alerts in your project.
- B. Set up quotas for the resources that your project will be using.
- C. Set up a spending limit on the credit card used in your billing account.
- D. Label all resources according to best practices, regularly export the billing reports, and analyze them with BigQuery.

**Answer: B**

**Explanation:** **Quotas** are the only option that actually **prevents** excessive resource consumption. Quotas limit how many resources can be created (allocation quotas) or how fast APIs can be called (rate quotas). Budgets and alerts (A) only **notify** you -- they do NOT stop spending. Credit card limits (C) are outside GCP's control and can cause account suspension. Labels and exports (D) are reactive analysis, not prevention.

> **Exam tip:** This is a critical distinction: **Budgets = alerts only, do NOT stop spending. Quotas = actually limit resource creation.** The exam loves testing this.

---

## Q20: Estimate Spending for Next Quarter

**Question:** Your project team needs to estimate the spending for your Google Cloud project for the next quarter. You know the project requirements. You want to produce your estimate as quickly as possible. What should you do?

- A. Build a simple machine learning model that will predict your next month's spend.
- B. Estimate the number of hours of compute time required, and then multiply by the VM per-hour pricing.
- C. Use the Google Cloud Pricing Calculator to enter your predicted consumption for all groups of resources.
- D. Use the Google Cloud Pricing Calculator to enter your consumption for all groups of resources, and then adjust for volume discounts.

**Answer: C**

**Explanation:** The **Google Cloud Pricing Calculator** is designed for exactly this -- you input expected resource usage (VMs, storage, networking, etc.) and it outputs a cost estimate. It already factors in standard pricing tiers. Option B only covers compute, ignoring storage, networking, and other services. Option A is wildly over-engineered for a cost estimate. Option D adds an unnecessary step -- the calculator already handles pricing tiers; manually adjusting for volume discounts is redundant and could introduce errors.

> **Exam tip:** "Estimate costs quickly" = **Google Cloud Pricing Calculator**. It's at [cloud.google.com/products/calculator](https://cloud.google.com/products/calculator).

---

## Score Card

| # | Topic | Answer |
|---|-------|--------|
| 1 | IAM for auditors (BigQuery) | C |
| 2 | IAM for project with sensitive data | D |
| 3 | Monitor Storage + Firestore changes | B |
| 4 | Distribute transactions across VMs | C |
| 5 | Connect on-prem to VPC | B |
| 6 | Cloud Storage lifecycle | A |
| 7 | IoT 10 PB storage | C |
| 8 | GKE frontend-backend communication | A |
| 9 | Docker microservices, variable load | A |
| 10 | Static IP for partner service | C |
| 11 | Consistent dev/test/prod environments | D |
| 12 | IP range exhausted in subnet | B |
| 13 | Expose apps through DNS | D |
| 14 | Optimize resource consumption analysis | B |
| 15 | Ad hoc SQL, large data, cost-effective | B |
| 16 | Migrate to GKE, minimize costs | A |
| 17 | Upload and convert pictures | D |
| 18 | Migrate 100 TB initial load | B |
| 19 | Prevent excessive consumption | B |
| 20 | Estimate quarterly spending | C |
