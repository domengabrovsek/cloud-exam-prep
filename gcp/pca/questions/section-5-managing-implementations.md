# Section 5: Managing Implementation

> **Exam weight:** ~12.5% of the PCA exam.
>
> This section covers API management, migration tooling, Terraform/IaC, Cloud SDKs, and emulators.

**Total questions: 25**

---

### Q1.

A company exposes internal microservices to external partners through APIs. They need rate limiting, API key authentication, developer portal for onboarding, and analytics on API usage. The APIs are served by Cloud Run services. What is the recommended API management solution?

A) Cloud Endpoints with ESP (Extensible Service Proxy) deployed as a sidecar on Cloud Run
B) Apigee API Management with Cloud Run backends, using Apigee's developer portal and analytics
C) Cloud Armor WAF rules on the HTTPS load balancer for rate limiting, with Identity Platform for API keys
D) A custom API gateway built on Cloud Functions that handles authentication and rate limiting

<details>
<summary>Answer</summary>

**Correct: B)**

Apigee is Google Cloud's full-featured API management platform. It provides rate limiting (quota policies), API key management, a built-in developer portal for partner onboarding, and comprehensive API analytics. It integrates with Cloud Run backends natively. This matches all four requirements.

- **A) is wrong** -- Cloud Endpoints provides basic API management (authentication, monitoring) but lacks a developer portal and has limited rate-limiting capabilities compared to Apigee. For external partner-facing APIs with onboarding requirements, Apigee is the right choice.
- **C) is wrong** -- Cloud Armor provides DDoS protection and WAF rules, not API management. It can do basic rate limiting but doesn't provide API key management, developer portals, or API analytics. Identity Platform is for user authentication, not API key management.
- **D) is wrong** -- A custom API gateway is a maintenance burden and reinvents functionality that Apigee provides out of the box. It would require building rate limiting, key management, analytics, and a portal from scratch.

**Exam tip:** Apigee = full API management (external APIs, partner portals, monetization). Cloud Endpoints = lightweight API gateway (internal APIs, simpler requirements). If the question mentions "developer portal" or "partner onboarding," it's Apigee.

Docs: https://cloud.google.com/apigee/docs/api-platform/get-started/what-apigee
</details>

---

### Q2.

Your organization uses Apigee for API management. A new regulatory requirement mandates that all API traffic within the EU must stay within EU boundaries, including the API gateway processing. How should you configure Apigee?

A) Deploy Apigee in a single EU region and restrict API consumers to EU IP ranges using Cloud Armor
B) Use Apigee hybrid with a runtime plane deployed in a GKE cluster in an EU region, keeping the management plane in Google-managed infrastructure
C) Configure Apigee with EU data residency and deploy the Apigee instance in an EU region with regional API routing
D) Use Cloud Endpoints instead of Apigee, deployed in an EU Cloud Run service

<details>
<summary>Answer</summary>

**Correct: C)**

Apigee supports data residency controls and regional deployment. By deploying the Apigee instance in an EU region and configuring data residency settings, all API traffic processing and analytics data stays within the EU. This meets regulatory requirements without sacrificing Apigee's features.

- **A) is wrong** -- Restricting consumers by IP range doesn't ensure that API gateway processing stays in the EU. Also, legitimate EU users may use VPNs or have non-EU IPs. The requirement is about where processing occurs, not where requests originate.
- **B) is wrong** -- Apigee hybrid deploys the runtime in your own GKE cluster, which gives location control. However, the management plane (analytics, configuration) still runs in Google-managed infrastructure, which may not meet strict EU data residency requirements for all data. Option C with native Apigee data residency is simpler and more complete.
- **D) is wrong** -- Replacing Apigee with Cloud Endpoints loses all the API management features (rate limiting, developer portal, analytics). Cloud Endpoints doesn't offer the same data residency controls. This is a downgrade in functionality to solve a configuration problem.

**Exam tip:** Apigee data residency = control where your API data is processed and stored. Apigee hybrid = run the runtime plane in your own infrastructure. Know the difference: data residency is about compliance, hybrid is about infrastructure control.

Docs: https://cloud.google.com/apigee/docs/api-platform/get-started/data-residency
</details>

---

### Q3.

A financial services company wants to monetize their data APIs by charging partners per API call with tiered pricing (first 10K calls free, 10K-100K at $0.01/call, 100K+ at $0.005/call). They need usage tracking, invoicing, and a self-service portal. What should they use?

A) Apigee with monetization features, configuring rate plans with tiered pricing, and the integrated developer portal for self-service
B) Cloud Endpoints with a custom billing system built on Cloud Functions that tracks usage and generates invoices
C) An API gateway on GKE with a custom metering system using Pub/Sub and BigQuery for usage tracking
D) Apigee with API products and quotas set to the tier limits, sending usage data to Stripe for billing

<details>
<summary>Answer</summary>

**Correct: A)**

Apigee has built-in monetization features specifically designed for API-as-a-product businesses. It supports tiered pricing (rate plans), usage tracking per developer/app, revenue reporting, and integrates with the developer portal for self-service subscription management. This is a core Apigee differentiator.

- **B) is wrong** -- Building a custom billing system is significant engineering effort (metering accuracy, invoice generation, payment processing, tax handling). Cloud Endpoints doesn't have monetization features. This reinvents what Apigee provides natively.
- **C) is wrong** -- A custom metering system on GKE/Pub/Sub/BigQuery is possible but is a custom build of what Apigee offers as a managed feature. It requires ongoing maintenance and doesn't include a developer portal or self-service subscription management.
- **D) is wrong** -- While Apigee + Stripe could work, Apigee already has built-in monetization. Sending data to Stripe adds an external dependency and integration complexity. Quotas are enforcement mechanisms, not billing mechanisms -- the tier limits should be in the rate plan, not quota policies.

**Exam tip:** Apigee monetization is a PCA exam topic. If the question mentions "API monetization," "charging per API call," or "tiered pricing for APIs," Apigee is the answer.

Docs: https://cloud.google.com/apigee/docs/api-platform/monetization/overview
</details>

---

### Q4.

An API team is designing a versioning strategy for their Apigee-managed APIs. They need to support two major versions simultaneously (v1 and v2) while eventually deprecating v1. Traffic should be gradually migrated from v1 to v2. What is the recommended approach?

A) Use Apigee API proxy revisions to manage v1 and v2 as different revisions of the same proxy
B) Create separate Apigee API proxies (my-api-v1, my-api-v2) with separate basepaths (/v1, /v2), and use Apigee's traffic management policies to gradually shift traffic
C) Use a single API proxy with conditional flows that route based on a version header
D) Deploy v2 as a new Apigee environment and migrate partners environment by environment

<details>
<summary>Answer</summary>

**Correct: B)**

Separate API proxies with versioned basepaths (/v1, /v2) is the Apigee best practice for major version management. Each proxy can have independent policies, backend routing, and lifecycle management. Apigee's traffic management (route rules, weighted routing) enables gradual migration. This approach allows deprecating v1 independently.

- **A) is wrong** -- Proxy revisions are for iterating on the same version (like deploying bug fixes), not for major version differences. Revisions share the same basepath, so you can't serve v1 and v2 simultaneously with different URLs.
- **C) is wrong** -- A single proxy with conditional flows becomes complex as versions diverge. All version logic is coupled in one proxy, making it harder to deprecate v1 independently. Header-based versioning is also less discoverable than URL-based versioning.
- **D) is wrong** -- Environments in Apigee are for lifecycle stages (dev, staging, prod), not API versions. Using environments for versioning misuses the abstraction and complicates deployment management.

**Exam tip:** Apigee versioning: proxy revisions = minor updates to same version, separate proxies = major version differences. URL-based versioning (/v1, /v2) is the recommended pattern.

Docs: https://cloud.google.com/apigee/docs/api-platform/fundamentals/best-practices-for-api-proxy-design-and-development
</details>

---

### Q5.

A company is migrating 500 VMs from VMware on-premises to GCP. They need minimal downtime during cutover and want to test VMs in GCP before committing to the migration. Which migration tool and approach should they use?

A) Use `gcloud compute images import` to convert VMware VMDKs to GCE images, then create instances from the images
B) Use Migrate to Virtual Machines (M2VM) to set up continuous replication from VMware to GCP, perform test clones in GCP, and execute final cutover with minimal downtime
C) Use a third-party tool like CloudEndure to replicate VMs, as GCP doesn't have native VMware migration tooling
D) Export VMs as OVA files, upload to Cloud Storage, and import using `gcloud compute instances import`

<details>
<summary>Answer</summary>

**Correct: B)**

Migrate to Virtual Machines (M2VM) provides continuous block-level replication from VMware (and other sources) to GCP. It supports test clones (running the migrated VM in GCP without affecting replication), which lets you validate before cutover. Final cutover has minimal downtime because replication keeps data current until the switch.

- **A) is wrong** -- Importing VMDKs is a manual, offline process. For 500 VMs, it's operationally prohibitive. There's no continuous replication, so downtime equals the time to export, upload, import, and configure each VM.
- **C) is wrong** -- GCP does have native VMware migration tooling (Migrate to Virtual Machines). While CloudEndure (now AWS Application Migration Service) exists, it's an AWS product and doesn't natively target GCP.
- **D) is wrong** -- OVA export/upload/import is manual and offline, similar to option A. For 500 VMs, this approach would take months and require extensive downtime per VM.

**Exam tip:** Migrate to Virtual Machines = continuous replication + test clones + minimal downtime cutover. Key features: source compatibility (VMware, AWS, Azure), continuous replication, test before commit. For VM migration questions, M2VM is the standard answer.

Docs: https://cloud.google.com/migrate/virtual-machines/docs/5.0/how-to/migrate-connector-overview
</details>

---

### Q6.

A company has 20 Oracle databases (total 50 TB) that need to migrate to Cloud SQL for PostgreSQL. Some databases have complex PL/SQL stored procedures and Oracle-specific features. What is the recommended migration approach?

A) Use Database Migration Service (DMS) to perform a direct Oracle-to-Cloud SQL migration with continuous replication
B) Use the Ora2Pg migration tool to convert schema and PL/SQL to PostgreSQL, assess and fix compatibility issues, then use DMS for data migration with minimal downtime
C) Export Oracle data as CSV, load into Cloud SQL using `pg_restore`, and manually rewrite stored procedures
D) Migrate to Cloud SQL for MySQL first (easier from Oracle), then migrate from MySQL to PostgreSQL

<details>
<summary>Answer</summary>

**Correct: B)**

Oracle-to-PostgreSQL migration requires schema conversion (Oracle data types, PL/SQL to PL/pgSQL, Oracle-specific functions). Ora2Pg is the industry-standard open-source tool for this conversion. After schema compatibility is addressed, DMS handles the data migration with continuous replication for minimal downtime. This is the standard heterogeneous database migration pattern.

- **A) is wrong** -- DMS supports heterogeneous migrations (Oracle to PostgreSQL), but it doesn't handle PL/SQL conversion automatically. The stored procedures and Oracle-specific features need schema conversion tooling (Ora2Pg or similar) before DMS can migrate the data.
- **C) is wrong** -- CSV export loses data types, constraints, indexes, and sequences. `pg_restore` works with PostgreSQL dump formats, not CSV. For 50 TB, CSV export/import would be extremely slow and error-prone. Manual stored procedure rewriting without a conversion tool is inefficient.
- **D) is wrong** -- Migrating to MySQL first adds an unnecessary intermediate step with its own compatibility issues. Oracle-to-MySQL has different conversion challenges than Oracle-to-PostgreSQL. Two migrations doubles the risk and effort.

**Exam tip:** Heterogeneous database migration (different engines) = schema conversion first, then data migration. Homogeneous (same engine) = DMS can handle end-to-end. Know that Oracle-to-PostgreSQL is a common PCA exam scenario.

Docs: https://cloud.google.com/database-migration/docs/oracle-to-postgresql/overview
</details>

---

### Q7.

Your company needs to migrate a Hadoop cluster (500 TB of data in HDFS, 200 MapReduce and Spark jobs) to GCP. The data science team wants to modernize while migrating. What is the recommended approach?

A) Lift-and-shift the Hadoop cluster to Dataproc, keeping the same HDFS storage
B) Migrate data from HDFS to Cloud Storage using the Hadoop DistCp tool, then run existing Spark/MapReduce jobs on ephemeral Dataproc clusters that read from Cloud Storage
C) Rewrite all MapReduce jobs as Dataflow pipelines and store data in BigQuery
D) Use a Hadoop-compatible file system (HCFS) connector to access HDFS from Dataproc running in GCP

<details>
<summary>Answer</summary>

**Correct: B)**

Separating storage (Cloud Storage) from compute (ephemeral Dataproc) is the cloud-native pattern. DistCp efficiently migrates HDFS data to Cloud Storage. Existing Spark/MapReduce jobs run on Dataproc with minimal changes (just change `hdfs://` paths to `gs://`). Ephemeral clusters reduce costs dramatically (pay only when processing).

- **A) is wrong** -- Keeping HDFS on Dataproc persistent clusters is a lift-and-shift that doesn't leverage cloud economics. You'd pay for always-on clusters to maintain HDFS, losing the cost savings of ephemeral compute.
- **C) is wrong** -- Rewriting 200 jobs as Dataflow pipelines is a massive effort. Not all MapReduce/Spark patterns translate easily to Dataflow. This approach modernizes too aggressively, delaying the migration. Migrate first, then selectively modernize.
- **D) is wrong** -- HCFS connector accessing on-premises HDFS from GCP Dataproc would have severe latency issues for 500 TB of data. Cross-network data access for compute-intensive workloads is impractical.

**Exam tip:** Hadoop to GCP: separate storage (Cloud Storage) from compute (Dataproc). Use DistCp for data migration. Ephemeral Dataproc clusters = pay per job, not 24/7. This is the standard migration pattern.

Docs: https://cloud.google.com/architecture/hadoop-migration-overview
</details>

---

### Q8.

A company uses Transfer Appliance to migrate 200 TB of data to GCP because their internet bandwidth is only 100 Mbps. While waiting for the appliance, they want to start migrating the most critical 5 TB of data immediately. What should they use for the 5 TB?

A) `gsutil -m rsync` to incrementally sync the 5 TB over the existing internet connection
B) Storage Transfer Service with a scheduled transfer from the on-premises data source
C) Wait for the Transfer Appliance to avoid data consistency issues
D) Set up a VPN and use `gcloud storage cp` for the transfer

<details>
<summary>Answer</summary>

**Correct: A)**

At 100 Mbps, transferring 5 TB takes approximately 4.6 days -- acceptable for critical data. `gsutil -m rsync` (or `gcloud storage rsync`) provides multi-threaded, resumable transfer with incremental sync capability. If any files change during transfer, re-running rsync only transfers the differences. This gets critical data to GCP quickly.

- **B) is wrong** -- Storage Transfer Service is designed for large-scale, scheduled transfers from other cloud providers (AWS S3, Azure) or HTTP/HTTPS sources, not for on-premises data sources. For on-premises to Cloud Storage, `gsutil`/`gcloud storage` or Transfer Appliance are the tools.
- **C) is wrong** -- Waiting delays the migration of critical data unnecessarily. Transfer Appliances can take weeks for shipping and processing. The 5 TB can be transferred in under a week over the existing connection.
- **D) is wrong** -- A VPN provides secure network connectivity but doesn't speed up the transfer. `gcloud storage cp` works but `gsutil -m rsync` is better because it's incremental and handles restarts gracefully. Also, VPN setup adds time.

**Exam tip:** Data migration tool selection: <10 TB + decent bandwidth = gsutil/gcloud storage. 10-200 TB = Storage Transfer Service (cloud-to-cloud) or Transfer Appliance. 200+ TB = Transfer Appliance. Know the bandwidth calculation: 1 TB at 100 Mbps ~ 22 hours.

Docs: https://cloud.google.com/storage/docs/gsutil/commands/rsync
</details>

---

### Q9.

You are advising a team that needs to test their Cloud Functions locally before deploying. The functions interact with Firestore, Pub/Sub, and Cloud Storage. What is the recommended local development approach?

A) Deploy to a test project and test in the cloud environment
B) Use the Firebase Local Emulator Suite which provides emulators for Firestore, Pub/Sub, and Cloud Storage, allowing full local testing
C) Mock all GCP service calls using a testing framework like `unittest.mock` in Python
D) Use Docker containers running unofficial GCP service emulators from GitHub

<details>
<summary>Answer</summary>

**Correct: B)**

The Firebase Local Emulator Suite provides official emulators for Firestore, Pub/Sub, Cloud Storage, and other services. Functions can interact with these emulators identically to production services. This enables fast, free, offline development and testing without deploying to GCP.

- **A) is wrong** -- Testing in a cloud project works but is slow (deploy time), costs money, and requires internet connectivity. Local emulators provide faster feedback loops.
- **C) is wrong** -- Mocking all service calls tests your application logic but not the integration with GCP services. Mocks can pass while real service interactions fail (wrong data formats, missing permissions, incorrect API usage). Emulators provide higher-fidelity testing.
- **D) is wrong** -- Unofficial emulators may not accurately replicate GCP service behavior and aren't maintained by Google. The Firebase Local Emulator Suite is the official, supported solution.

**Exam tip:** Firebase Local Emulator Suite is the official local testing tool for Firestore, Pub/Sub, Cloud Storage, Auth, and Functions. For Spanner and Bigtable, there are separate official emulators. Know which services have emulators.

Docs: https://firebase.google.com/docs/emulator-suite
</details>

---

### Q10.

A team is using Gemini Code Assist (formerly Duet AI for Developers) in their IDE. They want to use it to help with writing Terraform configurations for GCP infrastructure. Which capabilities does Gemini Code Assist provide for this use case? (Choose TWO)

A) Generating Terraform resource blocks from natural language descriptions of desired infrastructure
B) Automatically applying Terraform configurations to GCP projects
C) Explaining existing Terraform code and suggesting improvements for GCP best practices
D) Replacing the need for `terraform plan` by predicting the changes
E) Managing Terraform state files in Cloud Storage

<details>
<summary>Answer</summary>

**Correct: A) and C)**

Gemini Code Assist can generate Terraform code from natural language prompts (e.g., "create a GKE cluster with Autopilot in us-central1") and can explain existing Terraform configurations, suggest improvements based on GCP best practices, and help debug Terraform errors.

- **B) is wrong** -- Gemini Code Assist is an IDE assistant, not an infrastructure automation tool. It generates code suggestions; it doesn't execute `terraform apply` or modify cloud resources.
- **D) is wrong** -- `terraform plan` queries actual GCP APIs to determine the real diff between state and desired configuration. Gemini cannot replicate this because it doesn't have access to your Terraform state or live GCP environment.
- **E) is wrong** -- Terraform state management is handled by Terraform backends (Cloud Storage, Terraform Cloud, etc.). Gemini Code Assist doesn't manage infrastructure state.

**Exam tip:** Gemini Code Assist = AI pair programmer (code generation, explanation, debugging). It doesn't replace DevOps tools (Terraform, Cloud Build, Cloud Deploy). Know the boundary between AI assistance and infrastructure automation.

Docs: https://cloud.google.com/gemini/docs/discover/write-code-gemini
</details>

---

### Q11.

A development team is testing their application that uses Cloud Spanner. Running tests against a real Spanner instance is slow and expensive. What is the recommended approach for local testing?

A) Use the Cloud Spanner emulator for local development and integration tests, reserving real Spanner instances for pre-production validation
B) Create a small single-node Spanner instance in a test project for all developers to share
C) Use SQLite as a local substitute since it supports similar SQL syntax
D) Skip Spanner-specific testing and rely on production monitoring to catch issues

<details>
<summary>Answer</summary>

**Correct: A)**

The Cloud Spanner emulator is an official Google-provided local emulator that implements the Spanner API. It supports the same SQL dialect, schema definitions, and client library interfaces. It runs locally with no cost, enabling fast iteration. Real Spanner instances should be used for pre-production validation to test with real distributed behavior.

- **B) is wrong** -- A shared Spanner instance creates contention between developers, costs money (even a single-node instance is ~$650/month), and introduces flaky tests due to shared state.
- **C) is wrong** -- SQLite has a completely different SQL dialect, doesn't support Spanner-specific features (interleaved tables, commit timestamps, stale reads), and doesn't implement the Spanner client library interface. Tests would not be representative.
- **D) is wrong** -- Skipping tests for a critical database component is negligent. Production monitoring is reactive; testing is preventive. Catching Spanner query issues in production means user-facing outages.

**Exam tip:** Know which GCP services have official emulators: Spanner, Firestore, Pub/Sub, Bigtable, Cloud Storage (via Firebase). Emulators are always preferred for local development over shared cloud instances.

Docs: https://cloud.google.com/spanner/docs/emulator
</details>

---

### Q12.

Your organization wants to use Gemini Code Assist but has concerns about code confidentiality. Developers work on proprietary algorithms. What should you communicate about Gemini Code Assist's data handling?

A) All code sent to Gemini is used to train Google's AI models, so proprietary code should not be used with it
B) Gemini Code Assist for enterprise (with a subscription) does not use customer code as training data, processes code securely, and is covered by Google Cloud's data privacy commitments
C) Code suggestions are generated entirely on the local machine with no data sent to Google
D) Only code in public repositories is used for suggestions; private code is never processed

<details>
<summary>Answer</summary>

**Correct: B)**

Gemini Code Assist enterprise subscriptions include Google Cloud's data privacy commitments: customer code is not used to train foundation models, prompts and responses are encrypted in transit and processed securely, and the service is covered by the same data processing terms as other Google Cloud services.

- **A) is wrong** -- This was a common concern with early AI coding assistants, but Gemini Code Assist for enterprise explicitly does not use customer code for model training. Google has published clear data governance commitments for enterprise customers.
- **C) is wrong** -- Gemini Code Assist sends code context to Google's servers for processing by the Gemini model. It's not a local-only tool. However, enterprise data handling ensures code is not retained for training.
- **D) is wrong** -- Gemini processes the code you send it (including private code) to generate suggestions. The key point is that this processing doesn't result in the code being used for training, not that private code is never processed.

**Exam tip:** For enterprise AI assistant questions, know the data governance model: customer code is NOT used for training, is processed securely, and is covered by Google Cloud's DPA. This is a key differentiator from consumer AI tools.

Docs: https://cloud.google.com/gemini/docs/discover/data-governance
</details>

---

### Q13.

A platform team manages 200 GCP projects using Terraform. Their Terraform codebase has grown to 50,000 lines in a single state file, and `terraform plan` takes 20 minutes. Developers frequently encounter state locking conflicts. How should they restructure? (Choose TWO)

A) Split the monolithic Terraform configuration into smaller, domain-specific modules with separate state files (networking, compute, data, IAM)
B) Increase the Cloud Storage backend's lock timeout to reduce conflicts
C) Use Terragrunt to orchestrate multiple Terraform modules with dependency management and keep state files smaller
D) Migrate to Pulumi for better performance with large state files
E) Remove the state lock and use a CI/CD pipeline to serialize Terraform applies

<details>
<summary>Answer</summary>

**Correct: A) and C)**

- **A)** Splitting into domain-specific modules with separate state files reduces the size of each state file, making `terraform plan` faster and reducing lock contention (different domains can be modified independently).
- **C)** Terragrunt provides a framework for managing multiple Terraform modules, handling dependencies between them, and managing separate state files. It's specifically designed for the "many modules, many state files" pattern.

- **B) is wrong** -- Increasing lock timeout doesn't solve the problem; it makes it worse. Developers wait longer when there's a conflict instead of failing fast. The root cause is that too many changes go through one state file.
- **D) is wrong** -- Migrating 50,000 lines of Terraform to Pulumi is a massive effort that doesn't address the architectural problem. Even in Pulumi, a single large stack would have similar performance issues.
- **E) is wrong** -- Removing the state lock is dangerous. Concurrent Terraform applies without locking can corrupt state, leading to resource conflicts, orphaned resources, or failed deployments. Serializing via CI/CD is fragile (what if two pipelines run simultaneously?).

**Exam tip:** Terraform at scale: split state files by domain/team, use Terragrunt for orchestration, use remote backends with locking. Monolithic state = slow plans + lock conflicts. The exam tests Terraform operational patterns.

Docs: https://cloud.google.com/docs/terraform/best-practices/general-style-structure
</details>

---

### Q14.

A team is writing Terraform for a new GCP project. They need to create a VPC, GKE cluster, Cloud SQL instance, and associated IAM bindings. The GKE cluster depends on the VPC, and Cloud SQL needs a private IP in the VPC. What is the best way to structure the Terraform code?

A) Put all resources in a single `main.tf` file with explicit `depends_on` for all dependencies
B) Use separate Terraform modules for networking (VPC, subnets, firewall rules), compute (GKE), database (Cloud SQL), and IAM, with module outputs as inputs to dependent modules
C) Create separate Terraform workspaces for each component and apply them in the correct order manually
D) Use `terraform apply -target` to apply resources in the correct dependency order

<details>
<summary>Answer</summary>

**Correct: B)**

Modular Terraform with explicit inputs/outputs is the best practice. The networking module outputs VPC and subnet IDs. The compute module takes these as inputs for GKE. The database module takes VPC information for private IP configuration. Terraform automatically determines the correct apply order from the dependency graph.

- **A) is wrong** -- A single file with explicit `depends_on` doesn't scale and is hard to maintain. Terraform normally infers dependencies from resource references; manual `depends_on` suggests the code structure is wrong. Single-file configs become unreadable beyond a few resources.
- **C) is wrong** -- Separate workspaces with manual ordering is error-prone and doesn't enforce dependency order. Workspaces are designed for environment separation (dev/staging/prod), not component separation.
- **D) is wrong** -- `-target` is for exceptional situations (debugging, recovering from partial failures), not for normal operation. Relying on `-target` means Terraform can't manage the full dependency graph, leading to drift and inconsistencies.

**Exam tip:** Terraform best practices: modules for logical grouping, outputs for inter-module dependencies, never use `-target` in normal workflows, minimize `depends_on` (let Terraform infer dependencies).

Docs: https://cloud.google.com/docs/terraform/best-practices/general-style-structure#module-structure
</details>

---

### Q15.

Your team uses Terraform with a Cloud Storage backend for state. A junior developer accidentally ran `terraform destroy` on the production project, deleting critical resources. Fortunately, data was backed up. How should you prevent this from happening again? (Choose TWO)

A) Enable Cloud Storage object versioning on the state bucket to allow state recovery
B) Implement a CI/CD pipeline (Cloud Build) for Terraform applies, removing `terraform apply/destroy` permissions from developer workstations
C) Add `prevent_destroy` lifecycle rules to critical resources in Terraform
D) Use Terraform workspaces to separate production from development
E) Disable the `destroy` command in Terraform configuration

<details>
<summary>Answer</summary>

**Correct: B) and C)**

- **B)** Running Terraform through a CI/CD pipeline with proper IAM ensures developers can't run `terraform destroy` directly. The pipeline can include plan review, approval gates, and restricted permissions.
- **C)** `prevent_destroy` lifecycle rules on critical Terraform resources cause `terraform destroy` (and `terraform apply` that would destroy the resource) to fail with an error. This is a code-level safeguard.

- **A) is wrong** -- Object versioning on the state bucket helps recover the state file but doesn't prevent the destruction of actual GCP resources. By the time you restore state, the resources are already deleted.
- **D) is wrong** -- Workspaces separate state but don't prevent running `destroy` on any workspace, including production. A developer could still `terraform workspace select prod && terraform destroy`.
- **E) is wrong** -- There is no Terraform configuration option to globally disable the `destroy` command. `prevent_destroy` is per-resource, not global.

**Exam tip:** Terraform safety: CI/CD pipeline for applies (no direct access), `prevent_destroy` on critical resources, state versioning for recovery (not prevention), and IAM to restrict who can apply changes.

Docs: https://cloud.google.com/docs/terraform/best-practices/general-style-structure#prevent-destruction
</details>

---

### Q16.

A company wants to adopt Infrastructure as Code but has 500 existing GCP resources created manually through the console. They want to bring these under Terraform management without recreating them. What is the recommended approach?

A) Use `terraform import` for each resource to import existing resources into Terraform state, then write matching Terraform configuration
B) Use Google Cloud's Config Connector to automatically generate Terraform code from existing resources
C) Delete all resources and recreate them with Terraform
D) Use `gcloud` commands to export resource configurations and manually convert them to Terraform HCL

<details>
<summary>Answer</summary>

**Correct: A)**

`terraform import` brings existing resources under Terraform management by adding them to the state file. After importing, you write (or generate) the corresponding Terraform configuration. For bulk imports, tools like `terraformer` (open-source) can generate both state and configuration for many resources automatically.

- **B) is wrong** -- Config Connector is a Kubernetes add-on for managing GCP resources through Kubernetes manifests, not for generating Terraform code. It's a different IaC paradigm, not a migration tool.
- **C) is wrong** -- Deleting and recreating production resources causes downtime and data loss. This is never acceptable for production infrastructure.
- **D) is wrong** -- While `gcloud` can describe resources, manually converting output to Terraform HCL for 500 resources is extremely tedious and error-prone. `terraform import` (or terraformer for bulk) is the standard approach.

**Exam tip:** `terraform import` = bring existing resources into Terraform state. After import, you must write the matching configuration. For bulk import, mention tools like `terraformer` or the newer `terraform import` block (Terraform 1.5+) that can generate configuration.

Docs: https://cloud.google.com/docs/terraform/import
</details>

---

### Q17.

A team is developing a Terraform module that creates GKE clusters. The module needs to be reusable across development, staging, and production environments with different configurations (machine types, node counts, networking). What is the best way to parameterize the module?

A) Use Terraform variables with type constraints and validation rules, providing environment-specific values through `.tfvars` files
B) Use conditional expressions with `count` based on an environment variable to create different resources per environment
C) Create three separate modules (one per environment) with hardcoded values
D) Use Terraform workspaces and `terraform.workspace` interpolation to determine configuration values

<details>
<summary>Answer</summary>

**Correct: A)**

Variables with type constraints (string, number, object) and validation rules ensure the module receives correct inputs. `.tfvars` files per environment (`dev.tfvars`, `prod.tfvars`) provide environment-specific values cleanly. This approach is maintainable, type-safe, and follows Terraform best practices.

- **B) is wrong** -- Using `count` with conditionals creates different resource structures per environment, making the module harder to understand and maintain. If environments need different resources (not just different configurations), separate modules are better.
- **C) is wrong** -- Three separate modules with hardcoded values violate DRY (Don't Repeat Yourself). Any change to the GKE configuration must be replicated across all three modules, increasing maintenance burden and risk of drift.
- **D) is wrong** -- Using `terraform.workspace` for configuration values tightly couples the module to workspace naming conventions. It makes the module less reusable and harder to test. Modules should be parameterized via variables, not workspace state.

**Exam tip:** Terraform module best practices: parameterize with variables + type constraints + validation, use `.tfvars` for environment values, never hardcode environment-specific values in modules.

Docs: https://cloud.google.com/docs/terraform/best-practices/general-style-structure#variables
</details>

---

### Q18.

Your team uses Terraform to manage GCP infrastructure. They want to enforce that all Cloud Storage buckets have uniform bucket-level access enabled and all GKE clusters have workload identity enabled. How should you enforce these standards?

A) Write a custom pre-commit hook that greps Terraform files for the required settings
B) Use Terraform Sentinel (or Open Policy Agent with conftest) to write policy-as-code that validates Terraform plans against organizational standards
C) Rely on code reviews to catch non-compliant configurations
D) Create GCP organization policies that enforce these settings and let Terraform fail if non-compliant

<details>
<summary>Answer</summary>

**Correct: B)**

Policy-as-code tools like OPA/conftest or HashiCorp Sentinel validate Terraform plans against organizational policies before applying. This catches non-compliant configurations during CI/CD, before they reach GCP. Policies can check for uniform bucket access, workload identity, and any other standard.

- **A) is wrong** -- Grepping Terraform files is fragile. It can't handle modules, variables, conditional expressions, or default values. It also can't validate the plan output, only the source code text.
- **C) is wrong** -- Code reviews are important but don't scale. Reviewers can miss settings, especially in large changesets. Automated policy enforcement is more reliable and consistent.
- **D) is wrong** -- While GCP org policies CAN enforce some settings (like uniform bucket access), not all Terraform configurations map to org policies (e.g., workload identity on GKE is an API-level setting, not an org policy). Also, failing at apply time is slower feedback than failing at plan time. Policy-as-code catches issues earlier.

**Exam tip:** Policy-as-code (OPA/Sentinel) validates before apply. Org policies enforce at the GCP API level. Both have value, but policy-as-code catches issues earlier in the pipeline. For Terraform-specific enforcement, policy-as-code is the answer.

Docs: https://cloud.google.com/docs/terraform/policy-validation
</details>

---

### Q19.

A developer needs to interact with the GCP Compute Engine API from a Python application. They need to list all VMs in a project, create new VMs, and delete VMs. What is the recommended approach?

A) Use the `google-cloud-compute` Python client library, which provides idiomatic Python classes for Compute Engine resources
B) Use the `requests` library to make direct REST API calls to `compute.googleapis.com`
C) Shell out to `gcloud compute` commands from Python using `subprocess`
D) Use the `google-api-python-client` (discovery-based) library for all Google API interactions

<details>
<summary>Answer</summary>

**Correct: A)**

The `google-cloud-compute` library is the modern, idiomatic Cloud Client Library for Compute Engine. It provides typed Python classes, proper error handling, automatic pagination, and retries. It's generated from the API specification and is the recommended way to interact with GCP services from Python.

- **B) is wrong** -- Direct REST API calls require manual handling of authentication (OAuth2 token management), pagination, retries, error parsing, and request construction. The client library handles all of this automatically.
- **C) is wrong** -- Shelling out to `gcloud` from Python is fragile (parsing text output), slow (process creation overhead), and difficult to test. Client libraries are designed for programmatic access; `gcloud` is designed for humans and scripts.
- **D) is wrong** -- The `google-api-python-client` (discovery-based) is the older client library. While functional, it's dynamically typed (no IDE autocompletion), has a different API style, and is not recommended for new development. The Cloud Client Libraries (`google-cloud-*`) are preferred.

**Exam tip:** Cloud Client Libraries (`google-cloud-*`) are the recommended SDKs. Discovery-based libraries (`google-api-python-client`) are the older generation. Know the difference for the exam. Client libraries exist for Python, Java, Go, Node.js, C#, Ruby, and PHP.

Docs: https://cloud.google.com/python/docs/reference/compute/latest
</details>

---

### Q20.

A team is building a Cloud Function that processes Pub/Sub messages and writes to Firestore. They want to write integration tests that verify the end-to-end flow locally without deploying to GCP. What combination should they use?

A) Firebase Local Emulator Suite (Pub/Sub emulator + Firestore emulator + Functions emulator) with test scripts that publish messages and verify Firestore writes
B) Unit tests with mocked Pub/Sub and Firestore clients
C) Deploy to a test project and use Cloud Scheduler to trigger test messages
D) Use Docker Compose to run unofficial Pub/Sub and Firestore containers

<details>
<summary>Answer</summary>

**Correct: A)**

The Firebase Local Emulator Suite provides official emulators for Pub/Sub, Firestore, and Cloud Functions. Running all three emulators locally creates a complete local environment where you can publish test messages to Pub/Sub, trigger the Function, and verify the Firestore writes -- all without cloud access or cost.

- **B) is wrong** -- Unit tests with mocks verify application logic but not the integration between services. Mock behavior may not match real service behavior, leading to tests that pass locally but fail in production.
- **C) is wrong** -- Deploying to a test project is slow (deploy time), costs money, requires network access, and makes the test cycle longer. Local emulators provide faster feedback.
- **D) is wrong** -- Unofficial containers may not accurately emulate GCP service behavior and aren't maintained by Google. The official Firebase emulators are purpose-built for this use case.

**Exam tip:** Firebase Local Emulator Suite covers: Functions, Firestore, Pub/Sub, Cloud Storage, Auth, Hosting, Realtime Database. For integration testing of serverless GCP applications, this is the go-to tool.

Docs: https://firebase.google.com/docs/emulator-suite
</details>

---

### Q21.

Your organization manages API traffic through Apigee and wants to implement an API-first design approach. Development teams create APIs before building implementations. How should the API lifecycle be structured?

A) Design APIs in Apigee's API proxy editor, generate OpenAPI specs from the proxy, then build backend implementations
B) Design APIs using OpenAPI specifications in Apigee's API hub, mock the APIs for consumer testing, build backend implementations, then deploy API proxies connecting specs to backends
C) Build backend services first, then auto-generate API specs using Swagger tools, and import into Apigee
D) Use gRPC protocol buffers as the API contract and convert to REST at the Apigee layer

<details>
<summary>Answer</summary>

**Correct: B)**

API-first design means the API specification comes first. Apigee's API hub provides a registry for managing API specs. Mocking allows API consumers (frontend teams, partners) to develop against the API contract before the backend exists. Once the backend is built, Apigee proxies connect the spec to the implementation.

- **A) is wrong** -- The API proxy editor is for implementation, not design. Starting with proxy configuration means you're building the gateway before defining the contract, which is implementation-first, not API-first.
- **C) is wrong** -- Building backends first and generating specs is the opposite of API-first. Auto-generated specs from implementations often expose internal details and don't represent the ideal consumer experience.
- **D) is wrong** -- While gRPC with Apigee transcoding is a valid architecture, it doesn't address the API-first lifecycle question. The question is about process (design -> mock -> build -> deploy), not protocol choice.

**Exam tip:** API-first = spec first, implementation second. Apigee API hub = central registry for API specs. Mocking = let consumers develop before backends are ready. This lifecycle is a PCA exam topic.

Docs: https://cloud.google.com/apigee/docs/api-hub/what-is-api-hub
</details>

---

### Q22.

A team needs to manage Terraform state for multiple environments (dev, staging, prod) across 10 microservices. Each microservice has its own Terraform configuration. What is the recommended state management strategy?

A) One Cloud Storage bucket with separate state files using key prefixes: `{env}/{service}/terraform.tfstate`
B) Separate Cloud Storage buckets per environment, with state files per service
C) A single state file containing all environments and services
D) Terraform Cloud for state management with workspaces per environment per service

<details>
<summary>Answer</summary>

**Correct: A)**

A single Cloud Storage bucket with prefix-based organization provides centralized management with logical separation. Each state file is independent (separate locking, separate access), while the bucket provides unified access control, versioning, and lifecycle management. The prefix structure `{env}/{service}/terraform.tfstate` scales cleanly.

- **B) is wrong** -- Separate buckets per environment add management overhead (bucket policies, versioning configuration, access controls for each bucket). The same logical separation is achieved with prefixes in a single bucket, with simpler management.
- **C) is wrong** -- A single state file for everything is the exact problem described in earlier questions (slow plans, lock contention, blast radius). This is the worst option.
- **D) is wrong** -- Terraform Cloud is a valid option but introduces a SaaS dependency and licensing costs. For organizations already on GCP, Cloud Storage backend provides the same functionality (locking, versioning, encryption) without additional services.

**Exam tip:** Terraform state strategy: one bucket, prefix-based organization, separate state per service per environment. Enable versioning on the bucket. Use Cloud Storage's IAM for access control to state files.

Docs: https://cloud.google.com/docs/terraform/best-practices/general-style-structure#state
</details>

---

### Q23.

A development team is building a data pipeline using Cloud Dataflow. They want to test their Apache Beam pipeline locally before deploying to the Dataflow runner. What is the correct local testing approach?

A) Use the DirectRunner to execute the pipeline locally with small test datasets, then switch to the DataflowRunner for production
B) Use the Dataflow emulator provided in the Cloud SDK
C) Deploy to Dataflow with `--maxNumWorkers=1` to simulate local execution
D) Test individual `DoFn` transforms with unit tests only, skipping pipeline-level tests

<details>
<summary>Answer</summary>

**Correct: A)**

Apache Beam's DirectRunner executes pipelines locally on the developer's machine. It supports the same pipeline API as the DataflowRunner, so pipelines developed locally translate directly to Dataflow. Test with small datasets locally using DirectRunner, then deploy with DataflowRunner for production scale.

- **B) is wrong** -- There is no official Dataflow emulator in the Cloud SDK. The DirectRunner serves this purpose for Apache Beam pipelines.
- **C) is wrong** -- Deploying to Dataflow with one worker still runs in the cloud (costs money, takes time to provision). It's not a local test -- it's a minimal cloud deployment.
- **D) is wrong** -- Unit testing individual DoFns is important but insufficient. Pipeline-level tests verify the pipeline topology (transforms connected correctly, windowing, triggers). Both unit tests and pipeline tests are needed.

**Exam tip:** Apache Beam runners: DirectRunner (local testing), DataflowRunner (production on GCP), SparkRunner (Apache Spark), FlinkRunner (Apache Flink). The same pipeline code works across runners. Local testing always uses DirectRunner.

Docs: https://cloud.google.com/dataflow/docs/guides/developing-pipelines
</details>

---

### Q24.

Your organization has a REST API rate limit policy that allows 100 requests per minute per API key. During a traffic spike, a critical partner exceeded the limit and was throttled, causing business impact. How should you handle this in Apigee? (Choose TWO)

A) Increase the global rate limit to 1000 requests per minute for all API consumers
B) Implement tiered rate limiting using Apigee API products, assigning the critical partner to a higher-tier product with a 500 req/min quota
C) Use Apigee's SpikeArrest policy for protection against traffic bursts, combined with Quota policy for longer-term rate limiting, and provide the partner with a burst allowance
D) Remove rate limiting entirely for authenticated partners
E) Implement a waiting queue using Pub/Sub for requests that exceed the rate limit

<details>
<summary>Answer</summary>

**Correct: B) and C)**

- **B)** Apigee API products allow different rate limits per consumer tier. Critical partners get higher quotas through premium API products, while standard consumers keep the default limits. This is business-driven API management.
- **C)** SpikeArrest handles sudden traffic bursts (smoothing), while Quota handles sustained rate limiting. Configuring both provides protection against abuse while allowing legitimate traffic bursts.

- **A) is wrong** -- Increasing the global limit for everyone reduces protection against abuse and overconsumption. The issue is that one critical partner needs a higher limit, not that the limit is too low for everyone.
- **D) is wrong** -- Removing rate limiting for authenticated partners exposes the backend to overconsumption and potential outages. Even trusted partners should have reasonable limits to protect backend services.
- **E) is wrong** -- A Pub/Sub queue changes the API from synchronous to asynchronous, which is a fundamental API contract change. Partners expecting synchronous responses would need to be re-architected. This is disproportionate to the problem.

**Exam tip:** Apigee traffic management: SpikeArrest = burst protection (requests per second/minute smoothing), Quota = sustained limits (per minute/hour/day). API products = different limits for different consumers. Know all three.

Docs: https://cloud.google.com/apigee/docs/api-platform/develop/comparing-quota-spike-arrest-and-concurrent-rate-limit-policies
</details>

---

### Q25.

A team is migrating their application from AWS to GCP. The application uses AWS SDK calls extensively (S3, DynamoDB, SQS). They want to minimize code changes during migration. What approach should they take?

A) Use GCP's AWS-compatible APIs that accept AWS SDK calls natively
B) Create an abstraction layer (interface/adapter pattern) over cloud-specific SDK calls, implement GCP adapters using Cloud Client Libraries, and swap the implementations
C) Use a multi-cloud SDK like Apache Libcloud that abstracts both AWS and GCP
D) Find-and-replace AWS SDK calls with GCP equivalents using automated code transformation

<details>
<summary>Answer</summary>

**Correct: B)**

The adapter pattern provides a clean migration path. Define interfaces for storage, database, and messaging operations. Create AWS implementations (current code, refactored behind the interface) and GCP implementations (using Cloud Client Libraries). Swap implementations during migration. This also enables testing both implementations in parallel.

- **A) is wrong** -- GCP does not provide AWS-compatible APIs that accept AWS SDK calls. Each cloud has its own APIs and SDKs. (Some specific services like S3-compatible storage exist, but not broadly across all services.)
- **C) is wrong** -- Apache Libcloud provides a lowest-common-denominator abstraction that doesn't expose cloud-specific features. It's limited in functionality and often requires cloud-specific code anyway for advanced features.
- **D) is wrong** -- Automated find-and-replace of SDK calls is fragile because AWS and GCP SDKs have different patterns, authentication models, error handling, and feature sets. S3 and Cloud Storage have different APIs; DynamoDB and Firestore/Bigtable have fundamentally different data models.

**Exam tip:** Cloud migration with code changes: adapter pattern is the best practice. It enables gradual migration, parallel testing, and rollback capability. Direct SDK replacement is risky for anything beyond trivial usage.

Docs: https://cloud.google.com/architecture/migration-to-gcp-getting-started
</details>

---
