# Section 1: Setting Up a Cloud Solution Environment

> **Exam weight:** ~23% of the standard ACE exam. **Not included on the renewal exam.**
>
> This section covers creating projects and accounts within a resource hierarchy, managing billing configuration, installing and configuring the CLI (Cloud SDK), setting up Cloud Observability, and working with organization policies, quotas, and APIs.

**Total questions: 25**

---

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

### Q13. Your company has just set up a Google Cloud organization. You need to ensure that all resources across every project are created only in EU regions. What should you configure?

A) IAM deny policy blocking `compute.instances.create` outside EU
B) Organization policy with `gcp.resourceLocations` constraint set to EU locations
C) VPC firewall rules blocking traffic from non-EU regions
D) Budget alerts for spending in non-EU regions

<details>
<summary>Answer</summary>

**Correct: B)**

The `gcp.resourceLocations` organization policy constraint restricts where resources can be created. Applied at the org level, it affects all folders and projects beneath it. IAM (A) controls who can do what, not where. Firewalls (C) control network traffic. Budgets (D) are notifications only.
</details>

---

### Q14. You need to inventory all resources across your organization -- VMs, buckets, databases, IAM policies -- and analyze them for compliance. Which service should you use?

A) Cloud Monitoring dashboards
B) Cloud Asset Inventory
C) Resource Manager API
D) Cloud Billing export to BigQuery

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud Asset Inventory provides a complete inventory of all Google Cloud resources and IAM policies across your organization. You can search, export, and analyze assets. Cloud Monitoring (A) tracks metrics, not resource inventory. Resource Manager (C) manages projects/folders but doesn't inventory all resource types. Billing export (D) tracks costs, not resource configurations.
</details>

---

### Q15. Your team wants to use Gemini Cloud Assist to analyze resource configurations and get recommendations. What does Gemini Cloud Assist help with in the context of Cloud Asset Inventory?

A) Automatically fixing misconfigured resources
B) Generating natural language summaries and analysis of your cloud resources
C) Creating Terraform configurations from existing resources
D) Replacing Cloud Monitoring alerting policies

<details>
<summary>Answer</summary>

**Correct: B)**

Gemini Cloud Assist integrates with Cloud Asset Inventory to provide natural language analysis of your resources, helping you understand configurations, identify issues, and get recommendations. It doesn't auto-fix resources (A), generate Terraform (C), or replace monitoring (D).
</details>

---

### Q16. You're setting up a new standalone Google Cloud organization without Google Workspace. What do you use to manage users and groups?

A) Google Workspace Admin Console
B) Cloud Identity Free or Premium
C) Active Directory directly
D) IAM console

<details>
<summary>Answer</summary>

**Correct: B)**

Cloud Identity (Free or Premium) provides user and group management for organizations that don't use Google Workspace. It gives you an organization node and the ability to manage identities. Google Workspace (A) is an alternative but the question specifies "without Google Workspace." Active Directory (C) is on-premises and needs GCDS to sync. IAM console (D) manages permissions, not user identities.
</details>

---

### Q17. You need to confirm whether Cloud Spanner is available in the `asia-southeast2` region before deploying. Where do you check?

A) `gcloud services list --available`
B) The Google Cloud Locations page or `gcloud spanner instance-configs list`
C) Cloud Asset Inventory
D) The Pricing Calculator

<details>
<summary>Answer</summary>

**Correct: B)**

Product availability varies by region. You can check the Google Cloud Locations page (cloud.google.com/about/locations) or use service-specific commands like `gcloud spanner instance-configs list` to see available configurations and regions. `gcloud services list` (A) shows APIs, not regional availability. Cloud Asset Inventory (C) shows existing resources. Pricing Calculator (D) shows pricing, not availability.
</details>

---

### Q18. Your project's Compute Engine API quota for `CPUS` in `us-central1` is 24, but your team needs 48. What should you do?

A) Create a second project and split the workload
B) Request a quota increase through the Google Cloud Console or `gcloud` CLI
C) Switch to a different region with higher default quotas
D) Contact Google Cloud support to remove all quotas

<details>
<summary>Answer</summary>

**Correct: B)**

You can request quota increases through the Cloud Console (IAM & Admin > Quotas) or via `gcloud alpha services quota update`. Splitting across projects (A) adds complexity. Switching regions (C) doesn't solve the problem and default quotas are similar. Quotas can't be removed entirely (D) -- they're a safety mechanism.
</details>

---

### Q19. You're setting up networking for a new project. You need a VPC where you control the subnet IP ranges in specific regions. What type of VPC should you create?

A) Auto mode VPC
B) Custom mode VPC
C) Default VPC
D) Shared VPC

<details>
<summary>Answer</summary>

**Correct: B)**

Custom mode VPC gives you full control over subnet creation -- you choose which regions get subnets and define the IP ranges. Auto mode (A) automatically creates a subnet in every region with predefined ranges. Default VPC (C) is an auto mode VPC created automatically with each project. Shared VPC (D) is an organizational construct for sharing networks, not a VPC mode.
</details>

---

### Q20. You're setting up billing exports for cost analysis. Your team needs to run SQL queries against the billing data. Where should you export?

A) Cloud Storage in CSV format
B) BigQuery
C) Cloud Logging
D) Pub/Sub

<details>
<summary>Answer</summary>

**Correct: B)**

BigQuery billing export allows you to run SQL queries directly against your billing data. You get detailed usage records and can join with other datasets. Cloud Storage (A) exports are for archival, not interactive querying. Cloud Logging (C) is for logs, not billing. Pub/Sub (D) is for streaming, not billing export.
</details>

---

### Q21. You've created a budget in Google Cloud with alert thresholds at 50%, 90%, and 100%. At 100%, you need to automatically disable billing for non-production projects. What architecture achieves this?

A) Budget alert emails to the finance team who manually disable billing
B) Budget linked to Pub/Sub, triggering a Cloud Function that calls the Billing API to disable billing
C) Organization policy that caps spending at budget limit
D) Quota limits on all resources matching the budget amount

<details>
<summary>Answer</summary>

**Correct: B)**

Budgets can send notifications to Pub/Sub topics. A Cloud Function triggered by the Pub/Sub message can call the Cloud Billing API to programmatically unlink the billing account from the project. Email alerts (A) are manual. There's no org policy for spending caps (C). Quotas (D) limit resource count, not dollar amounts.
</details>

---

### Q22. You need to set up Google Cloud Observability for a new project. Which products are part of the Observability suite?

A) Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler, Error Reporting
B) Cloud Monitoring, Cloud Logging, BigQuery, Cloud DNS
C) Cloud Monitoring, Cloud IAM, Cloud Audit Logs, VPC Flow Logs
D) Cloud Logging, Cloud Build, Cloud Deploy, Artifact Registry

<details>
<summary>Answer</summary>

**Correct: A)**

Google Cloud Observability (formerly Stackdriver) includes Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler, and Error Reporting. These are the core observability tools. BigQuery (B) is analytics. IAM (C) is security. Cloud Build/Deploy (D) are CI/CD tools.
</details>

---

### Q23. You are charged with optimizing Google Cloud resource consumption. Specifically, you need to investigate the resource consumption charges and present a summary of your findings. You want to do it in the most efficient way possible. What should you do?

A) Rename resources to reflect the owner and purpose. Write a Python script to analyze resource consumption.
B) Attach labels to resources to reflect the owner and purpose. Export Cloud Billing data into BigQuery, and analyze it with Data Studio.
C) Assign tags to resources to reflect the owner and purpose. Export Cloud Billing data into BigQuery, and analyze it with Data Studio.
D) Create a script to analyze resource usage based on the project to which the resources belong. In this script, use the IAM accounts and services accounts that control given resources.

<details>
<summary>Answer</summary>

**Correct: B)**

**Labels** are key-value pairs designed for organizing resources and attributing costs in billing. Billing export to BigQuery captures label data, and Data Studio (now Looker Studio) provides visualization. This is the standard Google-recommended workflow for cost analysis. **Tags** (C) are for firewall rules and network policies, not billing attribution -- this is a common exam trap. Renaming resources (A) is disruptive. Custom scripts (D) are inefficient and hard to maintain.

> **Exam tip:** **Labels** = billing, organization, cost attribution. **Tags** (network tags) = firewall rules. Don't confuse them -- the exam tests this distinction.
</details>

---

### Q24. You are setting up billing for your project. You want to prevent excessive consumption of resources due to an error or malicious attack and prevent billing spikes or surprises. What should you do?

A) Set up budgets and alerts in your project.
B) Set up quotas for the resources that your project will be using.
C) Set up a spending limit on the credit card used in your billing account.
D) Label all resources according to best practices, regularly export the billing reports, and analyze them with BigQuery.

<details>
<summary>Answer</summary>

**Correct: B)**

**Quotas** are the only option that actually **prevents** excessive resource consumption. Quotas limit how many resources can be created (allocation quotas) or how fast APIs can be called (rate quotas). Budgets and alerts (A) only **notify** you -- they do NOT stop spending. Credit card limits (C) are outside GCP's control and can cause account suspension. Labels and exports (D) are reactive analysis, not prevention.

> **Exam tip:** This is a critical distinction: **Budgets = alerts only, do NOT stop spending. Quotas = actually limit resource creation.** The exam loves testing this.
</details>

---

### Q25. Your project team needs to estimate the spending for your Google Cloud project for the next quarter. You know the project requirements. You want to produce your estimate as quickly as possible. What should you do?

A) Build a simple machine learning model that will predict your next month's spend.
B) Estimate the number of hours of compute time required, and then multiply by the VM per-hour pricing.
C) Use the Google Cloud Pricing Calculator to enter your predicted consumption for all groups of resources.
D) Use the Google Cloud Pricing Calculator to enter your consumption for all groups of resources, and then adjust for volume discounts.

<details>
<summary>Answer</summary>

**Correct: C)**

The **Google Cloud Pricing Calculator** is designed for exactly this -- you input expected resource usage (VMs, storage, networking, etc.) and it outputs a cost estimate. It already factors in standard pricing tiers. Option B only covers compute, ignoring storage, networking, and other services. Option A is wildly over-engineered for a cost estimate. Option D adds an unnecessary step -- the calculator already handles pricing tiers; manually adjusting for volume discounts is redundant and could introduce errors.

> **Exam tip:** "Estimate costs quickly" = **Google Cloud Pricing Calculator**. It's at [cloud.google.com/products/calculator](https://cloud.google.com/products/calculator).
</details>

---

## Answer Key

| Q | Answer | Source | Topic |
|---|--------|--------|-------|
| 1 | B | 07-Q1 | Resource hierarchy -- folders |
| 2 | C | 07-Q2 | Organization policy admin role |
| 3 | C | 07-Q3 | Billing account linking |
| 4 | B | 07-Q4 | Budget alerts + Pub/Sub automation |
| 5 | B | 07-Q5 | Billing export to BigQuery |
| 6 | B | 07-Q6 | Enabling APIs |
| 7 | B | 07-Q7 | GCDS + SAML federation |
| 8 | C | 07-Q8 | Cloud Monitoring metrics scope |
| 9 | B | 07-Q9 | Admin Activity audit log retention |
| 10 | B | 07-Q10 | Quota management |
| 11 | C | 07-Q11 | Org policy -- resource locations |
| 12 | B | 07-Q12 | Billing labels in BigQuery |
| 13 | B | 11-Q1 | Org policy -- resource locations |
| 14 | B | 11-Q2 | Cloud Asset Inventory |
| 15 | B | 11-Q3 | Gemini Cloud Assist |
| 16 | B | 11-Q4 | Cloud Identity |
| 17 | B | 11-Q5 | Product regional availability |
| 18 | B | 11-Q6 | Quota increase request |
| 19 | B | 11-Q7 | Custom mode VPC |
| 20 | B | 11-Q8 | Billing export to BigQuery |
| 21 | B | 11-Q9 | Budget + Pub/Sub + Cloud Function |
| 22 | A | 11-Q10 | Observability suite products |
| 23 | B | 08-Q14 | Labels + billing export analysis |
| 24 | B | 08-Q19 | Quotas to prevent overspend |
| 25 | C | 08-Q20 | Pricing Calculator |
