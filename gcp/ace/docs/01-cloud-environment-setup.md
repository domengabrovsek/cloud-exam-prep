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

### Setting Up Standalone Organizations

You can create an Organization node without Google Workspace by using **Cloud Identity Free**. This is common for companies that want to use GCP with a centralized org structure but don't need Google Workspace email and collaboration tools.

**How standalone organizations work:**

- An Organization node is automatically created when you set up Cloud Identity (or Google Workspace).
- With **Cloud Identity Free**, you get identity management (users, groups, 2FA, SSO) but no Gmail, Drive, or Workspace apps.
- Cloud Identity Free supports up to 50 users for free; if you need more, you must upgrade to a paid Cloud Identity tier.
- The organization domain must be verified -- you prove you own the domain via DNS TXT records or HTML file upload.

**Steps to set up a standalone organization:**

1. Go to `cloud.google.com` and sign in with a Gmail account or any Google account.
2. Navigate to **Cloud Console > IAM & Admin > Identity & Organization**.
3. Follow the prompts to set up Cloud Identity. You will be asked to provide a domain you own (e.g., `example.com`).
4. Verify domain ownership via DNS TXT record or HTML file upload (similar to Google Search Console verification).
5. Complete the Cloud Identity account setup in the Google Admin Console (`admin.google.com`).
6. The Organization node is **automatically created** once Cloud Identity setup is complete.
7. Verify the organization exists:

```bash
# List organizations you have access to
gcloud organizations list

# Describe the organization
gcloud organizations describe 123456789012
```

**Difference between Google Workspace-backed and standalone orgs:**

| Aspect | Google Workspace | Cloud Identity Free |
|---|---|---|
| **Organization node** | Created automatically | Created automatically |
| **Identity management** | Yes (users, groups, SSO, 2FA) | Yes (users, groups, SSO, 2FA) |
| **Gmail, Drive, Docs, etc.** | Included | Not included |
| **Cost** | Paid per user | Free (up to 50 users) |
| **Use case** | Orgs that need collaboration tools + GCP | Orgs that only need GCP with centralized identity |

**Exam tips:**

- **Cloud Identity Free is sufficient for GCP-only organizations.** You do not need to pay for Google Workspace to get an Organization node.
- The Organization node is tied to a **domain** (e.g., `example.com`), not to individual user accounts. You cannot create an Organization using a personal Gmail account (like `john@gmail.com`).
- Once the Organization is created, the **first Super Admin** (the person who set up Cloud Identity) becomes the de facto owner. This user should grant `roles/resourcemanager.organizationAdmin` to other admins.
- **Domain verification is required** before Cloud Identity can be fully activated and the Organization node is created.
- If you delete an Organization, all resources under it remain, but they become standalone projects. The Organization **cannot be easily deleted** once created -- it requires contacting Google Support.

**Docs:** [Cloud Identity Overview](https://cloud.google.com/identity/docs/overview) | [Set Up Cloud Identity](https://cloud.google.com/resource-manager/docs/creating-managing-organization)

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

### Using Cloud Asset Inventory and Gemini Cloud Assist

**Cloud Asset Inventory** provides a complete inventory of all GCP resources, IAM policies, and organization policies across your organization, folders, and projects. It is essential for resource governance, security audits, and compliance.

**What is Cloud Asset Inventory?**

- A **metadata database** that tracks all resources in your GCP organization in near real-time.
- Captures resource configurations, IAM policies, and organization policies.
- Supports querying, exporting, and analyzing assets at scale.
- Does **not** store resource data (e.g., VM disk contents or database rows) -- only metadata (e.g., "this VM exists in us-central1-a with 4 vCPUs").

**Key capabilities:**

| Capability | Description |
|---|---|
| **Search all resources** | Find all resources of a specific type (e.g., "all VMs") across the entire org |
| **Search all IAM policies** | Find who has access to what (e.g., "which users have Owner on any project?") |
| **Export inventory** | Export full asset snapshots to BigQuery or Cloud Storage for analysis |
| **Analyze IAM policies** | Use the IAM Policy Analyzer to trace effective permissions and access paths |
| **Get resource history** | See the state of a resource at a previous point in time (up to 35 days of history) |

**Key gcloud commands:**

```bash
# Search for all Compute Engine instances across the organization
gcloud asset search-all-resources \
  --scope=organizations/123456789012 \
  --asset-types=compute.googleapis.com/Instance

# Search for all Cloud Storage buckets in a project
gcloud asset search-all-resources \
  --scope=projects/my-project-id \
  --asset-types=storage.googleapis.com/Bucket

# Search for all IAM policies granting the Owner role
gcloud asset search-all-iam-policies \
  --scope=organizations/123456789012 \
  --query="policy:roles/owner"

# Search for IAM policies granting access to a specific user
gcloud asset search-all-iam-policies \
  --scope=organizations/123456789012 \
  --query="policy:user:alice@example.com"

# Export all assets to BigQuery
gcloud asset export \
  --organization=123456789012 \
  --output-bigquery-table=projects/my-project-id/datasets/my_dataset/asset_export \
  --content-type=resource

# Export all IAM policies to Cloud Storage
gcloud asset export \
  --organization=123456789012 \
  --output-path=gs://my-bucket/asset-export.json \
  --content-type=iam-policy

# Analyze IAM policy to check if a principal has access to a resource
gcloud asset analyze-iam-policy \
  --organization=123456789012 \
  --full-resource-name="//compute.googleapis.com/projects/my-project-id/zones/us-central1-a/instances/my-vm" \
  --identity="user:alice@example.com"
```

**Gemini Cloud Assist:**

**Gemini Cloud Assist** is an AI-powered assistant that helps you analyze resources, troubleshoot issues, and get configuration recommendations using natural language queries.

**How Gemini Cloud Assist works with Cloud Asset Inventory:**

- Gemini can query Cloud Asset Inventory to answer questions like "What VMs are running in us-east1?" or "Which service accounts have not been used in 90 days?"
- It can analyze IAM policies and recommend least-privilege changes (e.g., "Which users have overly broad permissions?").
- It provides compliance checking (e.g., "Are there any public Cloud Storage buckets?").
- It offers configuration recommendations (e.g., "Which VMs are underutilized and can be resized?").

**Use cases for governance:**

- **Compliance audits:** "Show me all resources in the EU."
- **Security reviews:** "Which service accounts have the Owner role?"
- **Cost optimization:** "List all idle VMs."
- **Policy enforcement:** "Are there any VMs with external IPs in the production folder?"

**Exam tips:**

- Cloud Asset Inventory is **separate from Cloud Monitoring metrics**. It tracks resource metadata and IAM policies, not performance metrics.
- The **Cloud Asset Viewer** role (`roles/cloudasset.viewer`) is required to search and export assets.
- Asset history is retained for **35 days** (also called the "temporal window"). You can query historical state during this window.
- Asset exports to BigQuery are useful for **custom compliance reporting** and **showback/chargeback** by tagging resources with labels and querying them.
- The `gcloud asset search-all-resources` command uses a query syntax similar to Cloud Console search (e.g., `--query="labels.env=prod"`).
- **IAM Policy Analyzer** (`gcloud asset analyze-iam-policy`) is the tool for answering "who has access to this resource?" and "what can this user access?" questions.
- Gemini Cloud Assist integrates with Cloud Asset Inventory to provide **natural language querying** and **AI-driven insights** -- this is a newer feature and may appear in updated exam content.

**Docs:** [Cloud Asset Inventory Overview](https://cloud.google.com/asset-inventory/docs/overview) | [Searching Resources and Policies](https://cloud.google.com/asset-inventory/docs/searching-resources) | [Gemini Cloud Assist](https://cloud.google.com/gemini/docs)

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

### Confirming Availability of Products in Geographical Locations

Not all GCP products are available in all regions and zones. Before deploying resources, you must confirm that the product you need is available in your target geographical location.

**Google Cloud Locations:**

- Google Cloud has **regions** (independent geographic areas, e.g., `us-central1`, `europe-west1`) and **zones** (isolated locations within a region, e.g., `us-central1-a`, `us-central1-b`).
- The official reference for all locations is the **Google Cloud Locations page**: `cloud.google.com/about/locations`
- Each product has different regional availability. Some products are global, some regional, some zonal.

**How to check product availability:**

1. **Use the Cloud Locations page:**
   - Visit `cloud.google.com/about/locations`
   - Filter by region to see which products are available there.
   - Example: Cloud Spanner is only available in specific regions; Cloud SQL supports different database engines in different regions.

2. **Use gcloud commands to list regions/zones for specific products:**

```bash
# List all available Compute Engine regions
gcloud compute regions list

# Describe a specific region (shows available CPUs, quotas, etc.)
gcloud compute regions describe us-central1

# List all available zones
gcloud compute zones list

# List available zones in a specific region
gcloud compute zones list --filter="region:us-central1"

# List Cloud Spanner instance configurations (regional and multi-regional)
gcloud spanner instance-configs list

# Describe a specific Cloud Spanner configuration to see which regions it covers
gcloud spanner instance-configs describe regional-us-central1

# List available GKE versions per region
gcloud container get-server-config --region=us-central1

# List available Cloud SQL regions (via API or Console; no direct gcloud command)
# Check the Cloud Console or use the SQL Admin API
```

3. **Product-specific documentation:**
   - Each product's documentation includes a "Locations" or "Regions and Zones" page.
   - Example: Cloud SQL documentation lists supported regions and database versions per region.

**Resource scope: Global vs Regional vs Zonal**

Understanding resource scope is critical for the exam. Here is a reference table:

| Scope | What It Means | Examples |
|---|---|---|
| **Global** | Resource is not tied to a specific region or zone; accessible from anywhere | IAM policies, Cloud DNS, global load balancers, VPC networks, firewall rules, global Cloud Storage buckets (but data is stored in a location) |
| **Regional** | Resource exists in a specific region (replicated across zones in that region) | Cloud SQL instances, GKE clusters (regional), Cloud Storage regional buckets, external IP addresses (regional), subnets |
| **Zonal** | Resource exists in a specific zone only (not replicated) | Compute Engine VM instances, persistent disks, GKE node pools (zonal clusters) |

**Key facts for the exam:**

- A **zonal resource** (like a VM) can only be used in the zone where it was created. To move it, you must recreate it in another zone or use snapshots/images.
- A **regional resource** (like a regional GKE cluster) is replicated across multiple zones in the region for high availability.
- **Global resources** like VPC networks and IAM policies are shared across all regions. However, subnets within a VPC are regional.
- **Cloud Storage buckets** are created with a location (single region, dual-region, or multi-region). Data is stored in that location, but the bucket name is globally unique.
- Some products are only available in specific regions (e.g., Cloud Spanner multi-region `nam-eur-asia1` spans North America, Europe, and Asia).

**Exam tips:**

- Questions like "Which products can be used in `us-west1`?" require you to know the difference between global, regional, and zonal scope.
- If a question mentions latency or proximity to users, the answer usually involves deploying resources in a region **close to the users** (e.g., `asia-southeast1` for users in Singapore).
- **Compliance and data residency** scenarios often test your knowledge of `constraints/gcp.resourceLocations` org policy to restrict where resources can be created.
- A common trick: "Can you attach a persistent disk in `us-central1-a` to a VM in `us-central1-b`?" -- **No**, disks are zonal and must be in the same zone as the VM. You can snapshot the disk and create a new disk in the target zone.
- **Regional persistent disks** (replica PDs) are an exception -- they replicate across two zones in a region and can be attached to VMs in either zone.
- When designing for high availability, the pattern is: **regional resources** (like regional GKE or Cloud SQL with HA) or **multi-zonal deployments** (like a managed instance group spread across zones).

**gcloud examples for availability checks:**

```bash
# Check if a specific machine type is available in a zone
gcloud compute machine-types list --filter="zone:us-central1-a AND name:n1-standard-4"

# List all GPU types available in a zone
gcloud compute accelerator-types list --filter="zone:us-central1-a"

# Check available Cloud SQL tiers in a region (done via Console or SQL Admin API)
# No direct gcloud command; use: gcloud sql tiers list --project=my-project-id
```

**Docs:** [Google Cloud Locations](https://cloud.google.com/about/locations) | [Regions and Zones](https://cloud.google.com/compute/docs/regions-zones) | [Geography and Regions](https://cloud.google.com/docs/geography-and-regions)

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
