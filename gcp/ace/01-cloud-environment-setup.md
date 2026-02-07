# Domain 1: Setting Up a Cloud Solution Environment (~20% of Exam)

> **Quick context:** This domain tests whether you can set up the foundational building blocks -- org structure, IAM, billing, monitoring -- before deploying any workloads. Expect 4-5 questions on this domain.

---

## 1.1 Setting Up Cloud Projects and Accounts

### Creating a Resource Hierarchy

Google Cloud uses a hierarchical structure to organize resources. Every resource belongs to exactly one project, and projects can be organized under folders and an organization node.

**Hierarchy (top to bottom):**

```
Organization (company-level, tied to Cloud Identity / Workspace domain)
  └── Folders (departments, teams, environments -- can be nested up to 10 levels)
       └── Projects (the fundamental unit for grouping resources, billing, and APIs)
            └── Resources (VMs, buckets, datasets, etc.)
```

**Key facts for the exam:**

- An **Organization** node is automatically created when you set up Google Workspace or Cloud Identity. It is the root of the hierarchy.
- **Folders** require an Organization node to exist. You cannot create folders without one.
- **Projects** can exist without an Organization (standalone), but in enterprise setups they sit under folders or directly under the org.
- Each project has three identifiers: **project name** (human-friendly, not unique), **project ID** (globally unique, immutable after creation, chosen by you), and **project number** (globally unique, auto-assigned, numeric).
- IAM policies are **inherited downward**. A policy set on the org applies to all folders, projects, and resources beneath it.
- A **deny policy** at a higher level cannot be overridden by an allow at a lower level (deny always wins when using deny policies).

**Key gcloud commands:**

```bash
# Create a new project
gcloud projects create my-project-id --name="My Project" --folder=123456789

# Create a project under the organization directly
gcloud projects create my-project-id --name="My Project" --organization=987654321

# List all projects
gcloud projects list

# Describe a specific project
gcloud projects describe my-project-id

# Delete a project (30-day recovery window)
gcloud projects delete my-project-id

# Undelete a project within the 30-day window
gcloud projects undelete my-project-id

# Create a folder (requires Organization)
gcloud resource-manager folders create --display-name="Production" --organization=987654321

# Create a nested folder
gcloud resource-manager folders create --display-name="Team-A" --folder=111222333

# List folders
gcloud resource-manager folders list --organization=987654321

# Move a project to a different folder
gcloud projects move my-project-id --folder=111222333
```

**Exam tips:**

- The **Organization Admin** role (`roles/resourcemanager.organizationAdmin`) is needed to define the org structure and assign org-level IAM policies.
- The **Folder Admin** role (`roles/resourcemanager.folderAdmin`) is needed to create and manage folders.
- The **Folder Creator** role (`roles/resourcemanager.folderCreator`) lets someone create folders but not manage them after creation.
- Deleted projects enter a **30-day shutdown period** where they can be recovered. After that, the project ID becomes available again eventually.
- A common exam pattern: "Least-privilege way to let a team lead organize their projects" -- answer is usually Folder Admin or Folder Creator on a specific folder, not Organization Admin.

**Docs:** [Resource Hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)

---

### Applying Organizational Policies to the Resource Hierarchy

Organization Policies are **constraints** that govern how resources can be configured across the hierarchy. They are distinct from IAM (which controls *who* can do things); org policies control *what* can be done.

**How they work:**

- Org policies are set using the **Organization Policy Service**.
- They use **constraints** -- either **boolean** (on/off) or **list** (allow/deny specific values).
- Policies are inherited downward and can be overridden at lower levels (unless inheritance is enforced).

**Common constraints that appear on the exam:**

| Constraint | What It Does |
|---|---|
| `constraints/compute.disableSerialPortAccess` | Blocks serial port access on VMs |
| `constraints/compute.vmExternalIpAccess` | Controls which VMs can have external IPs |
| `constraints/iam.allowedPolicyMemberDomains` | Restricts which domains can be granted IAM roles |
| `constraints/compute.restrictVpnPeerIPs` | Controls allowed VPN peer IPs |
| `constraints/gcp.resourceLocations` | Restricts where resources can be created (regions/zones) |
| `constraints/iam.disableServiceAccountKeyCreation` | Prevents creation of user-managed SA keys |
| `constraints/compute.skipDefaultNetworkCreation` | Prevents auto-creation of the default VPC |

**Key gcloud commands:**

```bash
# List available constraints
gcloud org-policies list-available-constraints --organization=987654321

# Describe a specific org policy on the organization
gcloud org-policies describe constraints/compute.disableSerialPortAccess \
  --organization=987654321

# Set a boolean constraint (e.g., disable serial port access on a project)
gcloud org-policies set-policy policy.yaml --project=my-project-id

# Reset a policy to inherit from parent
gcloud org-policies reset constraints/compute.disableSerialPortAccess \
  --project=my-project-id
```

Example `policy.yaml` for a boolean constraint:

```yaml
constraint: constraints/compute.disableSerialPortAccess
booleanPolicy:
  enforced: true
```

Example `policy.yaml` for a list constraint (restrict resource locations):

```yaml
constraint: constraints/gcp.resourceLocations
listPolicy:
  allowedValues:
    - in:us-locations
    - in:eu-locations
```

**Exam tips:**

- Org policies are **not the same as IAM policies**. IAM = who can do what. Org policies = what is allowed regardless of who.
- If an org policy at the org level says "no external IPs on VMs," a Project Owner cannot override it unless the policy allows inheritance override.
- The **Organization Policy Administrator** role (`roles/orgpolicy.policyAdmin`) is required to set org policies.
- A common scenario: "How to ensure no resources are created outside of Europe?" -- answer is the `gcp.resourceLocations` constraint.

**Docs:** [Organization Policy Service](https://cloud.google.com/resource-manager/docs/organization-policy/overview)

---

### Granting Members IAM Roles Within a Project

IAM in GCP follows the model: **Member + Role + Resource = Policy Binding**.

**Member types (principals):**

| Type | Format |
|---|---|
| Google Account | `user:alice@example.com` |
| Service Account | `serviceAccount:sa@project.iam.gserviceaccount.com` |
| Google Group | `group:devs@example.com` |
| Google Workspace Domain | `domain:example.com` |
| allAuthenticatedUsers | Any Google account (avoid in prod) |
| allUsers | Anyone on the internet (public) |

**Role types:**

| Type | Example | Notes |
|---|---|---|
| **Basic** (formerly Primitive) | `roles/owner`, `roles/editor`, `roles/viewer` | Very broad; avoid in production |
| **Predefined** | `roles/compute.instanceAdmin`, `roles/storage.objectViewer` | Google-managed, service-specific |
| **Custom** | `roles/myCustomRole` | You define the exact permissions |

**Key gcloud commands:**

```bash
# View current IAM policy on a project
gcloud projects get-iam-policy my-project-id

# Grant a role to a user
gcloud projects add-iam-policy-binding my-project-id \
  --member="user:alice@example.com" \
  --role="roles/compute.instanceAdmin.v1"

# Grant a role to a group
gcloud projects add-iam-policy-binding my-project-id \
  --member="group:devs@example.com" \
  --role="roles/viewer"

# Grant a role to a service account
gcloud projects add-iam-policy-binding my-project-id \
  --member="serviceAccount:my-sa@my-project-id.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

# Remove a role binding
gcloud projects remove-iam-policy-binding my-project-id \
  --member="user:alice@example.com" \
  --role="roles/compute.instanceAdmin.v1"

# Create a custom role at the project level
gcloud iam roles create myCustomRole --project=my-project-id \
  --title="My Custom Role" \
  --description="Custom role for specific tasks" \
  --permissions="compute.instances.get,compute.instances.list"

# Create a custom role at the org level
gcloud iam roles create myOrgRole --organization=987654321 \
  --title="My Org Role" \
  --permissions="resourcemanager.projects.get"

# List predefined roles
gcloud iam roles list

# Describe a role to see its permissions
gcloud iam roles describe roles/compute.instanceAdmin.v1

# Create a service account
gcloud iam service-accounts create my-sa \
  --display-name="My Service Account" \
  --project=my-project-id

# List service accounts in a project
gcloud iam service-accounts list --project=my-project-id
```

**Exam tips:**

- **Prefer groups over individual users.** If a question asks the best way to manage access for a team, the answer is almost always "add users to a Google Group and grant the role to the group."
- **Least privilege** is always the right answer. If a question offers `roles/editor` and a more specific predefined role that covers the need, pick the predefined role.
- **Owner** (`roles/owner`) is the only basic role that can manage IAM policies and set up billing. `roles/editor` cannot modify IAM.
- IAM policies are **additive only** (with standard allow policies). You cannot deny access with a standard IAM policy -- you need **IAM Deny Policies** for that.
- **IAM conditions** allow time-based or resource-based conditional access (e.g., "only allow access to resources with a specific tag, only during business hours").
- Service accounts are both **identities** (they act as a principal) and **resources** (you grant roles on them). The `Service Account User` role (`roles/iam.serviceAccountUser`) lets someone impersonate a SA.
- Each project can have up to **100 service accounts**.

**Docs:** [IAM Overview](https://cloud.google.com/iam/docs/overview) | [Understanding Roles](https://cloud.google.com/iam/docs/understanding-roles)

---

### Managing Users and Groups in Cloud Identity (Manually and Automated)

**Cloud Identity** is Google's Identity-as-a-Service (IDaaS) product. It provides the user and group directory that GCP IAM references. Google Workspace includes Cloud Identity; if you don't use Workspace, you can use the free Cloud Identity Free tier.

**Manual management:**

- Users and groups are managed in the **Google Admin Console** (`admin.google.com`), not in the Cloud Console.
- Admins can create users, assign them to groups, and manage organizational units (OUs).
- Groups created in Cloud Identity / Workspace are available as IAM principals (`group:name@domain.com`).
- You can also create Google Groups directly for collaboration in the Cloud Console under **IAM & Admin > Groups**.

**Automated management:**

| Method | Use Case |
|---|---|
| **Google Cloud Directory Sync (GCDS)** | One-way sync from Active Directory / LDAP to Cloud Identity. Runs on-prem. Does not sync passwords. |
| **Azure AD / Entra ID provisioning** | SAML + SCIM integration for orgs using Microsoft identity |
| **Admin SDK Directory API** | Programmatic user/group management |
| **Terraform (google_cloud_identity_group)** | IaC-driven group management |
| **Cloud Identity API** | Create and manage groups programmatically |

**Key gcloud commands:**

```bash
# List groups (requires Cloud Identity API enabled)
gcloud identity groups list --organization=987654321

# Create a group
gcloud identity groups create "dev-team@example.com" \
  --organization=987654321 \
  --display-name="Dev Team" \
  --group-type="security"

# Add a member to a group
gcloud identity groups memberships add \
  --group-email="dev-team@example.com" \
  --member-email="alice@example.com" \
  --roles=MEMBER

# List members of a group
gcloud identity groups memberships list --group-email="dev-team@example.com"
```

**Exam tips:**

- **GCDS syncs identities, not passwords.** For SSO, you need a separate SAML-based federation (e.g., using Cloud Identity as the service provider with AD FS or Okta as the IdP).
- If asked about syncing an on-prem Active Directory with GCP, the answer is **GCDS** for identities + **SAML federation** for authentication.
- The **Super Admin** role in Cloud Identity / Workspace has ultimate control and should be heavily protected (hardware security keys, limited number of super admins).
- Questions about "how to manage access for 500 developers" -- the answer involves Cloud Identity groups, not adding individuals.

**Docs:** [Cloud Identity Overview](https://cloud.google.com/identity/docs/overview) | [Google Cloud Directory Sync](https://support.google.com/a/answer/106368)

---

### Enabling APIs Within Projects

Every GCP service requires its API to be enabled in the project before you can use it. New projects have a small set of APIs enabled by default.

**Key facts:**

- APIs are enabled **per project**.
- Some APIs are enabled by default (e.g., Compute Engine API on some project types), but most are not.
- Enabling an API does not incur charges by itself -- you are charged when you use resources.
- You need the **Service Usage Admin** role (`roles/serviceusage.serviceUsageAdmin`) or **Owner** to enable/disable APIs.

**Key gcloud commands:**

```bash
# Enable an API
gcloud services enable compute.googleapis.com --project=my-project-id

# Enable multiple APIs at once
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  container.googleapis.com \
  cloudfunctions.googleapis.com \
  --project=my-project-id

# List enabled APIs in a project
gcloud services list --enabled --project=my-project-id

# List all available APIs (enabled and disabled)
gcloud services list --available --project=my-project-id

# Disable an API (must have no dependent resources, or use --force)
gcloud services disable compute.googleapis.com --project=my-project-id
```

**Exam tips:**

- If you get an error like `PERMISSION_DENIED: <API> has not been used in project X before or it is disabled`, the first thing to check is whether the API is enabled.
- You can also enable APIs from the Cloud Console under **APIs & Services > Library**.
- **Org policies** can restrict which APIs are allowed (`constraints/serviceuser.services`).
- Disabling an API does **not** delete resources created by that API, but those resources become inaccessible until the API is re-enabled.

**Docs:** [Enabling and Disabling APIs](https://cloud.google.com/service-usage/docs/enable-disable)

---

### Provisioning and Setting Up Products in Google Cloud's Operations Suite

Google Cloud's operations suite (formerly Stackdriver) provides monitoring, logging, error reporting, tracing, and profiling.

**Core components:**

| Product | Purpose |
|---|---|
| **Cloud Monitoring** | Metrics, dashboards, uptime checks, alerting |
| **Cloud Logging** | Log ingestion, storage, analysis, export |
| **Cloud Trace** | Distributed tracing for latency analysis |
| **Cloud Profiler** | Continuous CPU/memory profiling |
| **Error Reporting** | Aggregates and tracks application errors |

**Cloud Monitoring -- key concepts:**

- **Metrics Scopes (formerly Stackdriver Workspaces):** A metrics scope defines which projects are monitored together. One project is the **scoping project** (host), and it can monitor up to 375 other **monitored projects**.
- **Alerting policies:** Define conditions (e.g., CPU > 80% for 5 min), notification channels (email, SMS, PagerDuty, Pub/Sub), and documentation.
- **Uptime checks:** Probe your endpoints from global locations at configurable intervals (1, 5, 10, or 15 min).
- **Custom metrics:** You can write custom metrics via the API or using OpenTelemetry.

**Cloud Logging -- key concepts:**

- All GCP services emit **audit logs** automatically (Admin Activity logs are always on and free).
- **Log types:** Admin Activity (always on, free, 400-day retention), Data Access (must be enabled, 30-day default retention), System Events (always on, free), Policy Denied.
- **Log Router:** Every log entry passes through the Log Router, which uses **sinks** to route logs to destinations (Cloud Logging buckets, Cloud Storage, BigQuery, Pub/Sub).
- **Exclusion filters** on the Log Router can reduce logging costs by dropping unwanted log entries before they are stored.
- Default retention: 30 days in the `_Default` bucket; `_Required` bucket is 400 days (not configurable).

**Key gcloud commands:**

```bash
# Create an alerting policy from a JSON/YAML file
gcloud alpha monitoring policies create --policy-from-file=alert-policy.json

# List alerting policies
gcloud alpha monitoring policies list

# Create a notification channel
gcloud alpha monitoring channels create --type=email --display-name="Ops Team" \
  --channel-labels=email_address=ops@example.com

# Create a log sink to BigQuery
gcloud logging sinks create my-bq-sink \
  bigquery.googleapis.com/projects/my-project-id/datasets/audit_logs \
  --log-filter='logName="projects/my-project-id/logs/cloudaudit.googleapis.com%2Factivity"'

# Create a log sink to Cloud Storage
gcloud logging sinks create my-gcs-sink \
  storage.googleapis.com/my-log-bucket \
  --log-filter='resource.type="gce_instance"'

# List log sinks
gcloud logging sinks list

# View logs
gcloud logging read "resource.type=gce_instance AND severity>=ERROR" \
  --limit=50 --project=my-project-id

# Create an uptime check (typically done in Console or via API)
# There is no direct gcloud command; use the Console or monitoring API
```

**Exam tips:**

- **Admin Activity audit logs** are always enabled, free, and retained for 400 days. You cannot disable them.
- **Data Access audit logs** must be explicitly enabled (except for BigQuery, where they are enabled by default). They can generate high volume and cost.
- When a question mentions "centralized logging across multiple projects," the answer is typically log sinks at the org or folder level routing to a shared project.
- **Metrics scope** setup: go to the scoping project's Monitoring section and add the other projects you want to monitor. This is done in the Cloud Console under Monitoring > Settings.
- The sink's service account (auto-created) needs write permissions on the destination. For BigQuery sinks, grant `roles/bigquery.dataEditor` to the sink's writer identity.

**Docs:** [Cloud Monitoring](https://cloud.google.com/monitoring/docs) | [Cloud Logging](https://cloud.google.com/logging/docs) | [Audit Logs](https://cloud.google.com/logging/docs/audit)

---

### Assessing Quotas and Requesting Increases

GCP enforces **quotas** (also called limits) to protect shared infrastructure and prevent runaway resource consumption.

**Types of quotas:**

| Type | Description | Example |
|---|---|---|
| **Rate quotas** | Limit API calls per time period | Compute Engine API: 20 requests/second |
| **Allocation quotas** | Limit number of resources you can create | Max 8 CPUs per region (default for new projects) |

**Key facts:**

- Quotas are enforced **per project** and often **per region**.
- Default quotas are conservative for new projects and can be increased via a request.
- Some quotas are hard limits that cannot be increased.
- You can view and request quota increases in the Cloud Console under **IAM & Admin > Quotas**.

**Key gcloud commands:**

```bash
# List all quotas for a project (filtered to Compute Engine as an example)
gcloud compute project-info describe --project=my-project-id

# List quotas for a specific region
gcloud compute regions describe us-central1 --project=my-project-id

# Request a quota increase (done via Console or via the Service Usage API)
# There is no single gcloud command -- use the Console:
# IAM & Admin > Quotas > filter to the quota > Edit Quotas

# You can also use the gcloud beta to update quotas via the API:
gcloud beta quotas info list --service=compute.googleapis.com --project=my-project-id
```

**Exam tips:**

- If you see an error like `QUOTA_EXCEEDED`, check the project's quotas for that region.
- **Quotas are not the same as budgets.** Quotas limit resource creation; budgets limit spending.
- Quota increases are **not instant** -- they require Google approval and can take 24-48 hours.
- To see all quotas at a glance, use the Cloud Console **Quotas page** with filters for the specific service and region.
- A common exam scenario: "You try to create a VM but get a quota error." Answer: check the CPU/disk quota in that region and request an increase.
- Some quotas can be managed through **quota override** using the Service Usage API or Terraform.

**Docs:** [Working with Quotas](https://cloud.google.com/docs/quotas/view-manage)

---

## 1.2 Managing Billing Configuration

### Creating One or More Billing Accounts

A billing account defines **who pays** for GCP resource usage. It is linked to a payment instrument (credit card, invoice, etc.).

**Types of billing accounts:**

| Type | Description |
|---|---|
| **Self-serve (online)** | Credit/debit card; anyone can create one |
| **Invoiced (offline)** | Invoice-based; requires Google Sales engagement; typically for larger orgs |

**Key facts:**

- A billing account is **separate from projects** -- it is a resource that can be managed under the organization.
- One billing account can be linked to **many projects**.
- An organization can have **multiple billing accounts** (e.g., one per department).
- Billing accounts can exist **outside an organization**, but best practice is to put them under one.
- The **Billing Account Creator** role (`roles/billing.creator`) at the org level is needed to create new billing accounts.
- The **Billing Account Administrator** role (`roles/billing.admin`) manages an existing billing account (link/unlink projects, manage payments, manage users).

**Key gcloud commands:**

```bash
# List billing accounts you have access to
gcloud billing accounts list

# Describe a billing account
gcloud billing accounts describe 01ABCD-EFGH12-34IJKL

# List projects linked to a billing account
gcloud billing projects list --billing-account=01ABCD-EFGH12-34IJKL
```

**Exam tips:**

- **Billing Account Creator** can create billing accounts but does not automatically have admin rights over them -- that role must be granted separately.
- A project without a linked billing account can **only use free-tier resources**. Paid resources will fail to provision.
- For large enterprises, the pattern is: one billing account per cost center or department, with the **billing account admin** managing each.
- When an organization is created, the **Organization Admin** does not automatically get billing roles -- these must be granted explicitly.

**Docs:** [Create, Modify, or Close a Billing Account](https://cloud.google.com/billing/docs/how-to/manage-billing-account)

---

### Linking Projects to a Billing Account

Every project that uses paid resources must be linked to exactly one billing account.

**Key facts:**

- A project can only be linked to **one billing account at a time**, but you can change which billing account it's linked to.
- Linking/unlinking requires the **Billing Account User** role (`roles/billing.user`) on the billing account AND **Project Owner** or **Project Billing Manager** (`roles/billing.projectManager`) on the project.
- If a billing account is disabled or a project is unlinked, **all paid resources in that project will be stopped** (VMs shut down, etc.) after a grace period.

**Key gcloud commands:**

```bash
# Link a project to a billing account
gcloud billing projects link my-project-id \
  --billing-account=01ABCD-EFGH12-34IJKL

# Unlink a project from its billing account (disables billing)
gcloud billing projects unlink my-project-id

# Check which billing account a project is linked to
gcloud billing projects describe my-project-id
```

**Exam tips:**

- To link a project to a billing account you need **both**: a role on the billing account (Billing Account User) **and** a role on the project (Owner or Project Billing Manager). This is a very common exam question.
- **Project Billing Manager** (`roles/billing.projectManager`) is the least-privilege role that can link/unlink billing. It cannot change anything else about the project.
- If a question says "a developer needs to deploy resources but billing is not enabled," the answer is to link the project to a billing account -- the developer should request this from someone with the appropriate billing roles.

**Docs:** [Enable Billing for a Project](https://cloud.google.com/billing/docs/how-to/modify-project)

---

### Establishing Billing Budgets and Alerts

Budgets allow you to track spending against a planned amount and receive alerts at configurable thresholds.

**How budgets work:**

- A budget is set on a **billing account** and can optionally be scoped to specific **projects**, **services**, or **labels**.
- You set a **budget amount** (fixed or based on last month's spend).
- You configure **threshold rules** (e.g., alert at 50%, 90%, 100% of budget).
- Alerts are sent to **Billing Account Admins and Users** by email, and optionally to **Pub/Sub** for programmatic actions and to **Monitoring notification channels**.

**Key gcloud commands:**

```bash
# Create a budget (using gcloud beta)
gcloud billing budgets create \
  --billing-account=01ABCD-EFGH12-34IJKL \
  --display-name="Monthly Dev Budget" \
  --budget-amount=1000USD \
  --threshold-rule=percent=0.5 \
  --threshold-rule=percent=0.9 \
  --threshold-rule=percent=1.0

# Create a budget scoped to specific projects
gcloud billing budgets create \
  --billing-account=01ABCD-EFGH12-34IJKL \
  --display-name="Project X Budget" \
  --budget-amount=500USD \
  --filter-projects="projects/my-project-id" \
  --threshold-rule=percent=0.5 \
  --threshold-rule=percent=1.0

# List budgets
gcloud billing budgets list --billing-account=01ABCD-EFGH12-34IJKL

# Describe a budget
gcloud billing budgets describe BUDGET_ID --billing-account=01ABCD-EFGH12-34IJKL

# Update a budget
gcloud billing budgets update BUDGET_ID --billing-account=01ABCD-EFGH12-34IJKL \
  --budget-amount=2000USD
```

**Programmatic budget actions (using Pub/Sub):**

```
Budget Alert → Pub/Sub Topic → Cloud Function → (action)
```

The Cloud Function can:
- Disable billing on the project (`cloudbilling.projects.updateBillingInfo`)
- Cap spending by removing the billing account link
- Send a Slack notification
- Scale down resources

**Exam tips:**

- **CRITICAL:** Budget alerts **do not stop spending by default.** They only send notifications. If the question asks "how to automatically stop spending when a budget is exceeded," the answer involves Pub/Sub + Cloud Functions to programmatically disable billing.
- Budget alerts can be sent to **email** (default to billing admins/users), **Pub/Sub topics**, or **Cloud Monitoring notification channels**.
- You can set alerts on **forecasted spend** as well (e.g., "alert me when forecasted spend for the month is expected to exceed 120% of budget").
- Budgets can be scoped by **project**, **service** (e.g., only Compute Engine), **label** (e.g., `env:dev`), or **credit type**.

**Docs:** [Set Budgets and Budget Alerts](https://cloud.google.com/billing/docs/how-to/budgets) | [Programmatic Budget Notifications](https://cloud.google.com/billing/docs/how-to/budgets-programmatic-notifications)

---

### Setting Up Billing Exports

Billing export lets you send detailed billing data to BigQuery or Cloud Storage for analysis, visualization, and cost management.

**Export types:**

| Export Target | Data | Use Case |
|---|---|---|
| **BigQuery (Standard usage cost)** | Detailed line items with resource-level cost data | Cost analysis with SQL, connect to Looker Studio |
| **BigQuery (Detailed usage cost)** | Same as standard + resource-level details (e.g., individual VM costs) | Granular chargeback and showback |
| **BigQuery (Pricing)** | The pricing table (list prices, SKUs) | Compare pricing programmatically |
| **Cloud Storage (CSV/JSON)** | Daily cost export files | Legacy; BigQuery export is preferred |

**Setup steps:**

1. Create a BigQuery dataset in a project (the dataset region cannot be changed later).
2. Go to **Billing > Billing export** in the Cloud Console.
3. Select the export type and specify the BigQuery dataset.
4. GCP will begin populating data (may take a few hours for initial data; not retroactive beyond the enable date for standard export).

**Key facts:**

- There is **no gcloud command** for configuring billing export -- it is done through the Cloud Console or the Billing API.
- Billing export to BigQuery is **free** (no charge for the export itself; you pay for BigQuery storage and queries).
- The BigQuery dataset used for export must be in the same **billing account** context.
- The **Billing Account Administrator** role is needed to set up billing export.
- Standard usage cost export data is available for querying in BigQuery after about **24 hours**.

**Example BigQuery query (common in exam scenarios):**

```sql
-- Total cost by project for the current month
SELECT
  project.name AS project_name,
  SUM(cost) AS total_cost,
  SUM(IFNULL((SELECT SUM(c.amount) FROM UNNEST(credits) c), 0)) AS total_credits
FROM
  `my-project.my_dataset.gcp_billing_export_v1_01ABCD_EFGH12_34IJKL`
WHERE
  invoice.month = FORMAT_DATE('%Y%m', CURRENT_DATE())
GROUP BY
  project_name
ORDER BY
  total_cost DESC;
```

**Exam tips:**

- **BigQuery export is the recommended approach** over Cloud Storage export. If the question says "analyze billing data," the answer is BigQuery export.
- Billing export is **not retroactive** -- it only captures data from the date it is enabled. This is a common trick in exam questions.
- To visualize billing data, the standard pattern is: BigQuery export + **Looker Studio** (formerly Data Studio) dashboards.
- **Cloud Storage export** is considered legacy. For new setups, always prefer BigQuery.
- The export dataset should be in a **dedicated project** that is not used for workloads, so billing admins can access it without needing access to workload projects.
- Labels applied to resources show up in billing export data, making them essential for cost allocation.

**Docs:** [Export Billing Data to BigQuery](https://cloud.google.com/billing/docs/how-to/export-data-bigquery) | [Billing Export Overview](https://cloud.google.com/billing/docs/how-to/export-data-bigquery-setup)

---

## Quick Reference: Key Roles for Domain 1

| Role | What It Grants |
|---|---|
| `roles/resourcemanager.organizationAdmin` | Full control over the organization, set IAM at org level |
| `roles/resourcemanager.folderAdmin` | Create, update, delete, move folders; set IAM on folders |
| `roles/resourcemanager.folderCreator` | Create folders (but not manage them after creation) |
| `roles/resourcemanager.projectCreator` | Create new projects |
| `roles/orgpolicy.policyAdmin` | Create and manage organization policies |
| `roles/billing.creator` | Create new billing accounts |
| `roles/billing.admin` | Manage a billing account (payments, users, link projects) |
| `roles/billing.user` | Link projects to a billing account |
| `roles/billing.projectManager` | Link/unlink billing on a specific project (least privilege) |
| `roles/billing.viewer` | View billing account cost and transactions |
| `roles/iam.organizationRoleAdmin` | Create custom roles at the org level |
| `roles/serviceusage.serviceUsageAdmin` | Enable/disable APIs |
| `roles/monitoring.admin` | Full access to Cloud Monitoring |
| `roles/logging.admin` | Full access to Cloud Logging |

---

## Exam Cheat Sheet: Domain 1

1. **Resource hierarchy flows down:** Org > Folders > Projects > Resources. IAM policies are inherited downward.
2. **Org policies are not IAM.** Org policies restrict *what* can be done; IAM restricts *who* can do it.
3. **Billing accounts are separate from projects.** Link them with `gcloud billing projects link`.
4. **Budgets do not stop spending.** You need Pub/Sub + Cloud Functions for automatic enforcement.
5. **Billing export to BigQuery is not retroactive.** Enable it early.
6. **Least privilege is always the answer.** Use predefined roles over basic roles, groups over individuals.
7. **API must be enabled** before using any service in a project.
8. **Admin Activity audit logs** are always on and free (400-day retention).
9. **Data Access audit logs** must be enabled (except BigQuery) and cost money.
10. **GCDS syncs identities, not passwords.** Use SAML federation for SSO.
