# Domain 5: Configuring Access and Security (~17.5% of Exam)

[Back to README](./README.md)

---

## Table of Contents

- [5.1 Managing Identity and Access Management (IAM)](#51-managing-identity-and-access-management-iam)
  - [IAM Overview](#iam-overview)
  - [Viewing and Creating IAM Policies](#viewing-and-creating-iam-policies)
  - [Role Types](#role-types)
  - [Custom IAM Roles](#custom-iam-roles)
  - [IAM Policy Hierarchy and Inheritance](#iam-policy-hierarchy-and-inheritance)
  - [IAM Conditions](#iam-conditions)
  - [IAM Deny Policies](#iam-deny-policies)
  - [Policy Troubleshooter](#policy-troubleshooter)
- [5.2 Managing Service Accounts](#52-managing-service-accounts)
  - [Service Account Overview](#service-account-overview)
  - [Creating Service Accounts](#creating-service-accounts)
  - [Using Service Accounts with Minimum Permissions](#using-service-accounts-with-minimum-permissions)
  - [Assigning Service Accounts to Resources](#assigning-service-accounts-to-resources)
  - [Managing IAM of a Service Account](#managing-iam-of-a-service-account)
  - [Service Account Impersonation](#service-account-impersonation)
  - [Short-Lived Service Account Credentials](#short-lived-service-account-credentials)
  - [User-Managed vs Google-Managed Service Accounts](#user-managed-vs-google-managed-service-accounts)
  - [Default Service Accounts and Their Risks](#default-service-accounts-and-their-risks)
  - [Service Account Keys](#service-account-keys)
  - [Workload Identity Federation](#workload-identity-federation)
  - [Organization Policy Constraints for IAM](#organization-policy-constraints-for-iam)
- [Exam Tips Summary](#exam-tips-summary)

---

## 5.1 Managing Identity and Access Management (IAM)

### IAM Overview

IAM lets you control **who** (identity) has **what access** (role) to **which resource**. An IAM policy is a collection of **bindings** that associate one or more **members** (principals) with a **role**.

Members (principals) can be:
- Google Account (user:alice@example.com)
- Service Account (serviceAccount:sa@project.iam.gserviceaccount.com)
- Google Group (group:devs@example.com)
- Google Workspace domain (domain:example.com)
- Cloud Identity domain
- allAuthenticatedUsers, allUsers (use with extreme caution)

Docs: [IAM Overview](https://cloud.google.com/iam/docs/overview)

---

### Viewing and Creating IAM Policies

An IAM **policy** is attached to a resource. It contains a list of **bindings** (role + members). Policies are **additive only** in the allow policy system -- there is no "deny" in allow policies (for deny, see [IAM Deny Policies](#iam-deny-policies)).

#### View the IAM policy for a project

```bash
# Get the full IAM policy for a project (JSON format)
gcloud projects get-iam-policy PROJECT_ID

# Get the policy in a specific format
gcloud projects get-iam-policy PROJECT_ID --format=json > policy.json
gcloud projects get-iam-policy PROJECT_ID --format=yaml

# Filter to see bindings for a specific member
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:alice@example.com" \
  --format="table(bindings.role)"
```

#### View IAM policy for other resources

```bash
# For organizations
gcloud organizations get-iam-policy ORG_ID

# For folders
gcloud resource-manager folders get-iam-policy FOLDER_ID

# For specific resources (e.g., a Cloud Storage bucket)
gcloud storage buckets get-iam-policy gs://BUCKET_NAME

# For Pub/Sub topics
gcloud pubsub topics get-iam-policy TOPIC_NAME
```

#### Add an IAM policy binding (grant access)

```bash
# Grant a role to a user on a project
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/viewer"

# Grant a role to a service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Grant a role to a group
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:devs@example.com" \
  --role="roles/editor"

# Grant at org level
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="user:admin@example.com" \
  --role="roles/resourcemanager.organizationAdmin"

# Grant at folder level
gcloud resource-manager folders add-iam-policy-binding FOLDER_ID \
  --member="user:alice@example.com" \
  --role="roles/viewer"
```

#### Remove an IAM policy binding (revoke access)

```bash
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/viewer"
```

#### Set the entire IAM policy (overwrite -- use carefully)

```bash
# Export, edit, then import
gcloud projects get-iam-policy PROJECT_ID --format=json > policy.json
# ... edit policy.json ...
gcloud projects set-iam-policy PROJECT_ID policy.json
```

> **Exam tip:** `set-iam-policy` replaces the entire policy. Use `add-iam-policy-binding` / `remove-iam-policy-binding` for incremental changes. Always use the `etag` field to prevent concurrent modification conflicts.

Docs: [Manage access to projects, folders, and organizations](https://cloud.google.com/iam/docs/granting-changing-revoking-access)

---

### Role Types

IAM has three types of roles: **Basic**, **Predefined**, and **Custom**.

#### Basic Roles (formerly "Primitive Roles")

These are the original, coarse-grained roles that existed before Cloud IAM:

| Role | Role ID | Description |
|------|---------|-------------|
| **Viewer** | `roles/viewer` | Read-only access to all resources |
| **Editor** | `roles/editor` | Viewer + can modify/create/delete most resources |
| **Owner** | `roles/owner` | Editor + manage roles/permissions + set up billing |

There is also `roles/browser` which allows listing projects and folders only (no resource access).

```bash
# List all basic roles
gcloud iam roles list --filter="name:roles/viewer OR name:roles/editor OR name:roles/owner"

# See what permissions a basic role includes
gcloud iam roles describe roles/editor
```

> **Exam tip:** Basic roles (Owner/Editor/Viewer) are **too broad for production use**. They grant permissions across ALL services in a project. The exam frequently tests whether you know to choose predefined roles over basic roles. Editor, for example, includes thousands of permissions. Always prefer predefined roles following the **principle of least privilege**.

> **Exam tip:** `roles/owner` is the only role that can manage IAM policies and set up billing for a project. It cannot be granted via `gcloud` at the organization level -- organization-level owner must be granted in the Console.

#### Predefined Roles

Predefined roles are **service-specific** and **granular**. Google creates and maintains them. They follow the pattern `roles/SERVICE.ROLE`.

Common examples:

| Role | Description |
|------|-------------|
| `roles/compute.instanceAdmin.v1` | Full control of Compute Engine instances |
| `roles/compute.networkAdmin` | Manage networking resources (not instances) |
| `roles/storage.objectViewer` | Read GCS objects |
| `roles/storage.objectCreator` | Create GCS objects (no read/delete) |
| `roles/storage.admin` | Full control of GCS |
| `roles/bigquery.dataViewer` | Read BigQuery datasets/tables |
| `roles/bigquery.jobUser` | Run BigQuery jobs |
| `roles/run.invoker` | Invoke a Cloud Run service |
| `roles/cloudfunctions.invoker` | Invoke a Cloud Function |
| `roles/logging.viewer` | View logs |
| `roles/monitoring.viewer` | View monitoring data |
| `roles/iam.serviceAccountUser` | Attach/act as a service account |
| `roles/iam.serviceAccountTokenCreator` | Create tokens for a service account |

```bash
# List all predefined roles
gcloud iam roles list

# List roles for a specific service
gcloud iam roles list --filter="name:roles/storage"

# Describe a predefined role to see its permissions
gcloud iam roles describe roles/storage.objectViewer
```

> **Exam tip:** Know the difference between `roles/storage.objectViewer` (read objects) vs `roles/storage.objectAdmin` (read/write/delete objects) vs `roles/storage.admin` (full control including bucket-level operations and IAM).

Docs: [Understanding roles](https://cloud.google.com/iam/docs/understanding-roles)

---

### Custom IAM Roles

Custom roles let you create a role with only the specific permissions you need. Use them when predefined roles are either too broad or don't match your exact requirements.

#### When to use custom roles
- Predefined roles grant more permissions than needed (least privilege)
- You need a unique combination of permissions across services
- Compliance requirements demand fine-grained access control

#### Limitations
- Custom roles can only be created at the **organization** or **project** level (NOT at the folder level)
- Maximum of 300 custom roles per organization, 300 per project
- Some permissions cannot be used in custom roles (marked `NOT_APPLICABLE` or `TESTING`)
- Custom roles must be manually maintained when APIs change (Google does not auto-update them)

#### Creating custom roles

```bash
# Create a custom role from a YAML file
gcloud iam roles create myCustomRole \
  --project=PROJECT_ID \
  --file=role-definition.yaml

# Create a custom role inline
gcloud iam roles create myStorageReader \
  --project=PROJECT_ID \
  --title="My Storage Reader" \
  --description="Can list buckets and read objects" \
  --permissions="storage.buckets.list,storage.objects.get,storage.objects.list" \
  --stage="GA"

# Create at the org level
gcloud iam roles create myOrgRole \
  --organization=ORG_ID \
  --file=role-definition.yaml
```

Example YAML definition file (`role-definition.yaml`):

```yaml
title: "My Custom Role"
description: "Custom role for app team"
stage: "GA"
includedPermissions:
  - storage.buckets.list
  - storage.objects.get
  - storage.objects.list
  - compute.instances.get
  - compute.instances.list
```

#### Managing custom roles

```bash
# List custom roles in a project
gcloud iam roles list --project=PROJECT_ID

# List custom roles in an organization
gcloud iam roles list --organization=ORG_ID

# Describe a custom role
gcloud iam roles describe myCustomRole --project=PROJECT_ID

# Update a custom role
gcloud iam roles update myCustomRole \
  --project=PROJECT_ID \
  --add-permissions="compute.instances.start,compute.instances.stop"

# Disable a custom role (set stage to DISABLED)
gcloud iam roles update myCustomRole \
  --project=PROJECT_ID \
  --stage=DISABLED

# Delete a custom role (can be undeleted within 7 days)
gcloud iam roles delete myCustomRole --project=PROJECT_ID

# Undelete (within 7 days)
gcloud iam roles undelete myCustomRole --project=PROJECT_ID
```

#### Role stages

| Stage | Meaning |
|-------|---------|
| `ALPHA` | Early testing |
| `BETA` | Tested, not yet GA |
| `GA` | Generally available |
| `DISABLED` | Role is disabled, permissions not enforced |
| `EAP` | Early Access Program |

> **Exam tip:** Custom roles can only be created at the **organization** or **project** level. You CANNOT create custom roles at the folder level. Deleted custom roles can be undeleted within 7 days, but the role ID cannot be reused for 37 days after permanent deletion.

Docs: [Creating and managing custom roles](https://cloud.google.com/iam/docs/creating-custom-roles)

---

### IAM Policy Hierarchy and Inheritance

IAM policies are inherited **downward** through the resource hierarchy:

```
Organization
  └── Folder
        └── Project
              └── Resource (e.g., GCS bucket, VM, BigQuery dataset)
```

Key rules:
1. **Policies are inherited** from parent to child. A role granted at the org level applies to every folder, project, and resource under it.
2. **Policies are additive (union).** If a user has `roles/storage.objectViewer` at the org level and `roles/storage.objectAdmin` at the project level, their effective permissions at the project level are the union of both.
3. **You cannot remove an inherited permission** with an allow policy at a lower level. Allow policies only add, never subtract. (Use [deny policies](#iam-deny-policies) to explicitly block.)
4. **More restrictive policies at a child do NOT override** more permissive policies inherited from a parent (in the allow system).

```
Example:
  Org: Alice has roles/viewer
    └── Folder: (no additional bindings for Alice)
          └── Project: Alice has roles/storage.admin
                └── Effective: Alice has roles/viewer + roles/storage.admin
```

> **Exam tip:** If a question asks "How to prevent a user from accessing a resource when they have an inherited Editor role?", the answer involves either **IAM deny policies** or **removing the binding at the parent level**. You cannot simply add a more restrictive allow policy at a child level.

Docs: [Resource hierarchy for access control](https://cloud.google.com/iam/docs/resource-hierarchy-access-control)

---

### IAM Conditions

IAM conditions allow you to grant access **conditionally** based on attributes of the request. Conditions are added to role bindings using Common Expression Language (CEL).

#### Supported condition attributes
- **Resource attributes:** resource.type, resource.name, resource.service
- **Request attributes:** request.time (for time-based access)
- **Access levels:** from Access Context Manager (VPC Service Controls)

#### Examples

```bash
# Grant access only until a specific date (temporary access)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:contractor@example.com" \
  --role="roles/storage.objectViewer" \
  --condition='expression=request.time < timestamp("2026-03-01T00:00:00Z"),title=temp-access,description=Temporary access until March 2026'

# Grant access to a specific resource name prefix
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/storage.objectViewer" \
  --condition='expression=resource.name.startsWith("projects/_/buckets/my-bucket/objects/public/"),title=public-only'

# Grant Compute Admin only for instances with a specific tag/prefix
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:devs@example.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=resource.name.startsWith("projects/PROJECT_ID/zones/us-central1-a/instances/dev-"),title=dev-instances-only'
```

> **Exam tip:** IAM conditions are commonly tested for **time-based access** (e.g., granting temporary access to a contractor) and **resource-based restrictions** (e.g., only allow access to certain buckets or instances). Not all resource types support conditions -- check the docs.

Docs: [IAM Conditions overview](https://cloud.google.com/iam/docs/conditions-overview)

---

### IAM Deny Policies

Deny policies let you **explicitly deny** specific permissions, overriding any allow policies. This was a major gap before -- allow policies are additive, so you previously could not block inherited permissions.

#### Key characteristics
- Deny policies are evaluated **before** allow policies
- Deny rules can include **exception principals** (principals exempt from the deny)
- Deny policies are attached to organizations, folders, or projects (not individual resources)
- Up to 500 deny rules per deny policy, up to 5 deny policies per resource
- Deny policies support conditions (same CEL syntax as allow conditions)

#### Evaluation order

```
1. Check deny policies (top-down through hierarchy) --> if denied, ACCESS DENIED
2. Check allow policies (inherited + direct) --> if allowed, ACCESS GRANTED
3. Default: ACCESS DENIED (implicit deny)
```

#### Managing deny policies (uses REST API or gcloud beta)

```bash
# List deny policies on a project
gcloud iam policies list \
  --attachment-point="cloudresourcemanager.googleapis.com/projects/PROJECT_ID" \
  --kind=denypolicies

# Create a deny policy from a JSON file
gcloud iam policies create my-deny-policy \
  --attachment-point="cloudresourcemanager.googleapis.com/projects/PROJECT_ID" \
  --kind=denypolicies \
  --policy-file=deny-policy.json
```

Example deny policy JSON:

```json
{
  "displayName": "Deny delete on production buckets",
  "rules": [
    {
      "denyRule": {
        "deniedPrincipals": [
          "principalSet://goog/group/devs@example.com"
        ],
        "exceptionPrincipals": [
          "principal://goog/subject/admin@example.com"
        ],
        "deniedPermissions": [
          "storage.googleapis.com/buckets.delete",
          "storage.googleapis.com/objects.delete"
        ],
        "denialCondition": {
          "expression": "resource.name.contains('/prod-')"
        }
      }
    }
  ]
}
```

> **Exam tip:** IAM allow policies are **additive** -- you cannot deny with allow policies alone. If the exam asks how to prevent a specific action for a user who has inherited Editor, the answer is **IAM deny policies**. Deny policies are evaluated before allow policies.

Docs: [Deny policies overview](https://cloud.google.com/iam/docs/deny-overview)

---

### Policy Troubleshooter

The IAM Policy Troubleshooter helps you understand **why a principal has or does not have access** to a resource. It evaluates all applicable allow and deny policies across the resource hierarchy.

#### Using via gcloud

```bash
# Check if a member has a specific permission on a resource
gcloud policy-troubleshoot iam \
  --principal-email="user:alice@example.com" \
  --resource-name="//cloudresourcemanager.googleapis.com/projects/PROJECT_ID" \
  --permission="compute.instances.delete"
```

Output tells you:
- Whether access is GRANTED or DENIED
- Which policy bindings grant or deny the permission
- At which level in the hierarchy the binding exists
- Whether any conditions affect the result

#### Using via Console
Navigate to **IAM & Admin > Troubleshoot** or access it from the IAM page by clicking the "Troubleshoot" icon next to a member.

> **Exam tip:** When a question describes a user who unexpectedly does or does not have access, the Policy Troubleshooter is the right diagnostic tool. It checks allow policies, deny policies, and the full resource hierarchy.

Docs: [Troubleshoot access](https://cloud.google.com/iam/docs/troubleshooting-access)

---

## 5.2 Managing Service Accounts

### Service Account Overview

A service account is a special type of account used by **applications, VMs, and services** (not humans). It is identified by an email address:

```
SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

Service accounts are **both an identity and a resource**:
- **As an identity:** A service account can be granted roles on resources (e.g., give it `roles/storage.objectViewer`)
- **As a resource:** Other identities can be granted roles ON the service account (e.g., who can "act as" or "manage" it)

Docs: [Service accounts overview](https://cloud.google.com/iam/docs/service-account-overview)

---

### Creating Service Accounts

```bash
# Create a service account
gcloud iam service-accounts create my-app-sa \
  --display-name="My Application Service Account" \
  --description="Used by the backend application"

# List service accounts in a project
gcloud iam service-accounts list --project=PROJECT_ID

# Describe a service account
gcloud iam service-accounts describe my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Disable a service account (temporary, reversible)
gcloud iam service-accounts disable my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Enable a disabled service account
gcloud iam service-accounts enable my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Delete a service account (can be undeleted within 30 days)
gcloud iam service-accounts delete my-app-sa@PROJECT_ID.iam.gserviceaccount.com
```

> **Exam tip:** Each project can have up to 100 user-managed service accounts. Service account names must be 6-30 characters. After deletion, you have 30 days to undelete, and the email cannot be reused for 30 days.

Docs: [Create service accounts](https://cloud.google.com/iam/docs/service-accounts-create)

---

### Using Service Accounts with Minimum Permissions

Follow the **principle of least privilege**: grant only the permissions the service account actually needs.

```bash
# Grant a predefined role with minimal scope
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-app-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Instead of roles/editor, use specific roles
# BAD:  --role="roles/editor"
# GOOD: --role="roles/storage.objectViewer"
# GOOD: --role="roles/pubsub.publisher"
```

Use **IAM Recommender** to identify and remove excess permissions:
- Console: IAM page shows recommendations (shield icon with exclamation mark)
- Suggests replacing broad roles with more specific ones based on actual usage over 90 days

> **Exam tip:** The exam loves questions about least privilege. If a question says "the application only needs to read Cloud Storage objects", the answer is `roles/storage.objectViewer`, not `roles/storage.admin` or `roles/editor`.

Docs: [IAM Recommender](https://cloud.google.com/iam/docs/recommender-overview)

---

### Assigning Service Accounts to Resources

When you attach a service account to a resource, that resource runs as the service account identity.

#### Compute Engine VMs

```bash
# Create a VM with a specific service account
gcloud compute instances create my-vm \
  --service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --zone=us-central1-a

# Change the service account of a stopped VM
gcloud compute instances set-service-account my-vm \
  --service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --zone=us-central1-a
```

> **Note on scopes:** Access scopes are a legacy method of limiting VM permissions. The recommended approach is to use `--scopes=cloud-platform` (full scope) and control access entirely through IAM roles. Scopes only restrict, never expand, what IAM allows.

#### Cloud Run

```bash
# Deploy a Cloud Run service with a specific service account
gcloud run deploy my-service \
  --image=gcr.io/PROJECT_ID/my-image \
  --service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --region=us-central1
```

#### Cloud Functions

```bash
# Deploy with a specific service account
gcloud functions deploy my-function \
  --runtime=python312 \
  --trigger-http \
  --service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --region=us-central1
```

#### GKE (Workload Identity)

```bash
# Enable Workload Identity on a cluster
gcloud container clusters update my-cluster \
  --workload-pool=PROJECT_ID.svc.id.goog \
  --zone=us-central1-a

# Bind Kubernetes SA to GCP SA
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"
```

> **Exam tip:** The user or service account **deploying** the resource needs `roles/iam.serviceAccountUser` on the target service account in order to attach it. This is a common exam question.

Docs: [Attach service accounts to resources](https://cloud.google.com/iam/docs/attach-service-accounts)

---

### Managing IAM of a Service Account

Because a service account is both an identity and a resource, there are two distinct IAM dimensions:

#### 1. Roles granted TO the service account (what it can do)

```bash
# View what roles a service account has on a project
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:my-app-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --format="table(bindings.role)"
```

#### 2. Roles granted ON the service account (who can use/manage it)

```bash
# View who has access to act as / manage the service account
gcloud iam service-accounts get-iam-policy \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Grant a user the ability to act as the service account
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --member="user:developer@example.com" \
  --role="roles/iam.serviceAccountUser"

# Grant a user the ability to create tokens for the service account
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --member="user:developer@example.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```

#### Key roles ON a service account

| Role | What it allows |
|------|---------------|
| `roles/iam.serviceAccountUser` | **Act as** the SA: attach it to VMs, Cloud Run, Cloud Functions, etc. Also allows running operations as the SA. |
| `roles/iam.serviceAccountTokenCreator` | **Create tokens** for the SA: generate access tokens, ID tokens, sign blobs, sign JWTs. Needed for impersonation. |
| `roles/iam.serviceAccountAdmin` | **Full management** of the SA: create, delete, update, manage keys, manage IAM policy of the SA. Does NOT grant the ability to act as the SA. |
| `roles/iam.serviceAccountKeyAdmin` | Create and manage keys for the SA. |

> **Exam tip:** There is a critical distinction between **"acting as"** a service account (`serviceAccountUser`) and **"managing"** a service account (`serviceAccountAdmin`). A service account admin can delete the SA or manage its keys, but CANNOT deploy resources as that SA without also having `serviceAccountUser`. Conversely, `serviceAccountUser` allows deploying resources that run as the SA, but not managing the SA itself.

> **Exam tip:** `roles/iam.serviceAccountUser` is needed to **attach** a service account to a resource (VM, Cloud Run, etc.). `roles/iam.serviceAccountTokenCreator` is needed to **impersonate** a service account (generate short-lived credentials).

Docs: [Manage access to service accounts](https://cloud.google.com/iam/docs/manage-access-service-accounts)

---

### Service Account Impersonation

Service account impersonation allows a user or another service account to **act as** a service account by generating short-lived credentials, without needing a downloaded key file.

#### Why use impersonation?
- Avoids the security risk of key files
- Provides short-lived credentials (default 1 hour, max 12 hours)
- Creates an auditable trail (logs show who impersonated whom)
- Supports the principle of least privilege and separation of duties

#### How it works

```bash
# Impersonate a service account in any gcloud command
gcloud storage ls gs://my-bucket \
  --impersonate-service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Set impersonation for all gcloud commands in the session
gcloud config set auth/impersonate_service_account \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Clear impersonation
gcloud config unset auth/impersonate_service_account
```

#### Prerequisites
The caller needs `roles/iam.serviceAccountTokenCreator` on the target service account.

```bash
# Grant impersonation ability
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --member="user:developer@example.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```

#### Chained impersonation (delegation chain)
SA-A impersonates SA-B which impersonates SA-C. Each SA in the chain needs `serviceAccountTokenCreator` on the next SA.

> **Exam tip:** Service account impersonation is **always preferred over downloading service account keys**. Keys are long-lived credentials that can be leaked. Impersonation generates short-lived tokens and leaves an audit trail.

Docs: [Service account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation)

---

### Short-Lived Service Account Credentials

Instead of using long-lived service account keys, you can generate **short-lived credentials** (tokens) that expire automatically.

#### Types of short-lived credentials

| Type | Use case | Default lifetime |
|------|----------|-----------------|
| **Access token** | Call Google APIs | 1 hour (max 12 hours) |
| **ID token** | Authenticate to services (Cloud Run, Cloud Functions) | 1 hour |
| **Self-signed JWT** | Call Google APIs without token exchange | 1 hour |
| **Self-signed blob** | Sign arbitrary data | N/A |

```bash
# Generate an access token
gcloud auth print-access-token \
  --impersonate-service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Generate an ID token (e.g., for calling an authenticated Cloud Run service)
gcloud auth print-identity-token \
  --impersonate-service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --audiences=https://my-service-xxxxx-uc.a.run.app
```

> **Exam tip:** When a question involves authenticating to a Cloud Run or Cloud Function that requires authentication, the answer typically involves generating an **ID token** (not an access token). Access tokens are for Google APIs; ID tokens are for authenticating to services.

Docs: [Create short-lived credentials](https://cloud.google.com/iam/docs/create-short-lived-credentials-direct)

---

### User-Managed vs Google-Managed Service Accounts

| Aspect | User-Managed | Google-Managed |
|--------|-------------|----------------|
| **Created by** | You (the user) | Google automatically |
| **Email format** | `NAME@PROJECT_ID.iam.gserviceaccount.com` | Varies (e.g., `PROJECT_NUMBER-compute@developer.gserviceaccount.com`) |
| **Manageable** | Full control (create, delete, grant roles, create keys) | Limited (can grant/revoke roles, but cannot delete or create keys) |
| **Examples** | Any SA you create | Compute Engine default SA, App Engine default SA, Google APIs service agent |
| **Count limit** | 100 per project | N/A (created automatically as needed) |

**Google-managed service agents** follow the pattern:
- `service-PROJECT_NUMBER@SERVICE.iam.gserviceaccount.com`
- Example: `service-123456@compute-system.iam.gserviceaccount.com`

These agents are used internally by Google services to perform actions on your behalf. You should generally not modify their permissions unless specifically required.

> **Exam tip:** Do not confuse Google-managed **service agents** (managed by Google for internal service operations) with the Compute Engine **default service account** (which is technically user-manageable and should be replaced in production).

Docs: [Service agents](https://cloud.google.com/iam/docs/service-agents)

---

### Default Service Accounts and Their Risks

When you enable certain APIs, GCP automatically creates a **default service account**:

| Service | Default SA email | Default role |
|---------|-----------------|--------------|
| Compute Engine | `PROJECT_NUMBER-compute@developer.gserviceaccount.com` | `roles/editor` on the project |
| App Engine | `PROJECT_ID@appspot.gserviceaccount.com` | `roles/editor` on the project |

#### Risks of default service accounts
1. **Overly permissive:** Default SAs are granted `roles/editor`, which gives near-full access to all resources in the project
2. **Shared across workloads:** All VMs use the same default SA unless you specify otherwise, violating isolation
3. **Difficult to audit:** When multiple workloads share one SA, you cannot tell which workload performed an action

#### Best practices

```bash
# DO: Create a dedicated service account per workload
gcloud iam service-accounts create my-web-app-sa \
  --display-name="Web App SA"

# DO: Grant only needed roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-web-app-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# DO: Use the dedicated SA when creating a VM
gcloud compute instances create web-server \
  --service-account=my-web-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform

# CONSIDER: Remove the Editor role from the default SA
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/editor"
```

> **Exam tip:** The exam frequently tests this: if a question mentions using the "default service account" in production, it is almost always the wrong answer. Create a **dedicated service account with minimum permissions** instead.

Docs: [Default service accounts](https://cloud.google.com/iam/docs/service-account-types#default)

---

### Service Account Keys

Service account keys are **long-lived credentials** (JSON key files) that allow applications to authenticate as a service account.

#### Types of keys
- **Google-managed keys:** Rotated automatically by Google. Used internally. You cannot download these.
- **User-managed keys:** You create and download them. You are responsible for rotation and security.

```bash
# Create and download a key (AVOID when possible)
gcloud iam service-accounts keys create key.json \
  --iam-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# List keys for a service account
gcloud iam service-accounts keys list \
  --iam-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com

# Delete a specific key
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com
```

#### Why to avoid user-managed keys
- Long-lived credentials that do not expire automatically (can be active for years)
- Can be leaked via source code, logs, or insecure storage
- Difficult to track and rotate at scale
- No way to know if a key has been compromised until it's used maliciously

#### Alternatives to keys (preferred)

| Scenario | Alternative |
|----------|------------|
| Application running on GCP (VM, Cloud Run, GKE, etc.) | Attached service account (automatic credential injection via metadata server) |
| Local development | `gcloud auth application-default login` or impersonation |
| GKE pods | Workload Identity |
| CI/CD from external platforms (GitHub Actions, GitLab, etc.) | Workload Identity Federation |
| Another GCP service calling Google APIs | Service account attached to the resource |
| User needs temporary SA access | Service account impersonation |

> **Exam tip:** Service account keys should be a **last resort**. The exam tests this heavily. Whenever you see a question about authenticating an application, prefer: (1) attached service accounts for GCP resources, (2) Workload Identity for GKE, (3) Workload Identity Federation for external workloads, (4) impersonation for users. Only use keys when none of these options are available.

Docs: [Best practices for managing service account keys](https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys)

---

### Workload Identity Federation

Workload Identity Federation lets **external workloads** (AWS, Azure, on-premises, GitHub Actions, etc.) authenticate to Google Cloud **without service account keys** by exchanging external credentials for short-lived GCP tokens.

#### How it works

```
External Workload (e.g., GitHub Actions)
  --> presents external token (OIDC/SAML/AWS)
    --> Workload Identity Pool validates token
      --> maps to a GCP service account
        --> returns short-lived GCP access token
```

#### Key components
- **Workload Identity Pool:** A container for external identities
- **Workload Identity Provider:** Configuration for a specific identity provider (AWS, Azure AD, OIDC provider)
- **Attribute mapping:** Maps external token claims to Google attributes

```bash
# Create a Workload Identity Pool
gcloud iam workload-identity-pools create my-pool \
  --location="global" \
  --display-name="My Pool"

# Create an OIDC provider (e.g., for GitHub Actions)
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository"

# Allow the external identity to impersonate a service account
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/my-pool/attribute.repository/my-org/my-repo"
```

#### Workload Identity for GKE (different from Federation)

Workload Identity for GKE maps **Kubernetes service accounts** to **Google Cloud service accounts**, so pods authenticate as GCP SAs without needing keys.

```bash
# The GKE-specific binding uses the workload pool format
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"

# Annotate the Kubernetes service account
kubectl annotate serviceaccount KSA_NAME \
  --namespace=NAMESPACE \
  iam.gke.io/gcp-service-account=my-app-sa@PROJECT_ID.iam.gserviceaccount.com
```

> **Exam tip:** Workload Identity Federation is the answer whenever the question involves authenticating an **external workload** (on-premises, AWS, CI/CD pipelines) to GCP without service account keys. Workload Identity (for GKE) is the answer for **GKE pods** that need to call Google APIs.

Docs: [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) | [Workload Identity for GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)

---

### Organization Policy Constraints for IAM

Organization policies allow administrators to set **guardrails** across the organization. These are different from IAM policies -- they restrict what can be configured, not who can do it.

#### Key IAM-related constraints

| Constraint | Effect |
|-----------|--------|
| `constraints/iam.disableServiceAccountKeyCreation` | Prevents creation of user-managed service account keys |
| `constraints/iam.disableServiceAccountCreation` | Prevents creation of new service accounts |
| `constraints/iam.allowedPolicyMemberDomains` | Restricts which domains can be added to IAM policies (prevent external sharing) |
| `constraints/iam.disableWorkloadIdentityClusterCreation` | Prevents creation of GKE clusters with Workload Identity |

```bash
# List all org policy constraints
gcloud org-policies list --organization=ORG_ID

# Describe a specific constraint
gcloud org-policies describe constraints/iam.disableServiceAccountKeyCreation \
  --organization=ORG_ID

# Set a boolean constraint (enforce it)
gcloud org-policies set-policy policy.yaml --project=PROJECT_ID
```

Example policy YAML to disable SA key creation:

```yaml
name: projects/PROJECT_ID/policies/iam.disableServiceAccountKeyCreation
spec:
  rules:
    - enforce: true
```

> **Exam tip:** If a question asks how to prevent service account key creation across an entire organization, the answer is the **Organization Policy constraint** `constraints/iam.disableServiceAccountKeyCreation`, not an IAM policy. Organization policies restrict configurations; IAM policies restrict who can perform actions.

Docs: [Organization Policy constraints](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) | [IAM-related constraints](https://cloud.google.com/resource-manager/docs/organization-policy/restricting-service-accounts)

---

## Exam Tips Summary

### Must-Know Concepts

1. **Principle of least privilege:** Always grant the minimum permissions needed. Use predefined roles, not basic roles. Use dedicated service accounts, not the default SA.

2. **Basic roles are too broad for production:** `roles/editor` includes thousands of permissions across all services. The exam expects you to choose predefined roles like `roles/storage.objectViewer` over `roles/editor`.

3. **IAM policies are additive:** You cannot deny access with allow policies. A permission granted at a parent level (org or folder) CANNOT be removed at a child level by an allow policy. Use **deny policies** for explicit deny.

4. **Custom roles: org or project only.** You CANNOT create custom roles at the folder level. Custom roles require manual maintenance.

5. **Service account impersonation over keys.** Always prefer impersonation, Workload Identity, or attached service accounts over downloading JSON key files. Keys are a last resort.

6. **Deny policies are evaluated before allow policies.** The evaluation order is: deny policies first, then allow policies, then implicit deny.

7. **serviceAccountUser vs serviceAccountTokenCreator:**
   - `serviceAccountUser` = attach/deploy resources as the SA (act as)
   - `serviceAccountTokenCreator` = generate tokens/impersonate the SA

8. **"Acting as" vs "managing" a service account:**
   - Acting as = running workloads with the SA identity (`serviceAccountUser`)
   - Managing = creating, deleting, updating the SA and its keys (`serviceAccountAdmin`)
   - These are independent -- having one does not grant the other

9. **Default service accounts are risky:** They have `roles/editor` by default. Always create dedicated SAs with minimum permissions for production workloads.

10. **Workload Identity Federation for external workloads:** GitHub Actions, AWS, Azure, on-prem -- all can authenticate to GCP without keys using WIF. GKE pods use Workload Identity (via `roles/iam.workloadIdentityUser`).

### Quick Decision Tree

```
Q: How should an application authenticate to GCP?

  Running on GCP (VM, Cloud Run, GKE, Cloud Functions)?
    --> Use attached service account (automatic via metadata server)
    --> For GKE specifically: use Workload Identity

  Running externally (on-prem, AWS, Azure, CI/CD)?
    --> Use Workload Identity Federation (no keys needed)

  Developer needs temporary SA access?
    --> Use service account impersonation

  Local development?
    --> gcloud auth application-default login
    --> Or impersonate a service account

  None of the above work?
    --> Service account key (last resort)
```

### Key Command Reference

```bash
# --- IAM Roles ---
gcloud iam roles list                                     # List predefined roles
gcloud iam roles list --project=PROJECT_ID                # List custom roles
gcloud iam roles describe ROLE_ID                         # See role permissions
gcloud iam roles create ROLE_ID --project=PROJECT_ID      # Create custom role
gcloud iam roles update ROLE_ID --project=PROJECT_ID      # Update custom role
gcloud iam roles delete ROLE_ID --project=PROJECT_ID      # Delete custom role

# --- IAM Policies ---
gcloud projects get-iam-policy PROJECT_ID                 # View project policy
gcloud projects add-iam-policy-binding PROJECT_ID ...     # Grant a role
gcloud projects remove-iam-policy-binding PROJECT_ID ...  # Revoke a role
gcloud projects set-iam-policy PROJECT_ID policy.json     # Overwrite entire policy

# --- Service Accounts ---
gcloud iam service-accounts create SA_NAME                # Create SA
gcloud iam service-accounts list                          # List SAs
gcloud iam service-accounts describe SA_EMAIL             # Describe SA
gcloud iam service-accounts delete SA_EMAIL               # Delete SA
gcloud iam service-accounts disable SA_EMAIL              # Disable SA
gcloud iam service-accounts enable SA_EMAIL               # Re-enable SA

# --- SA IAM (who can use the SA) ---
gcloud iam service-accounts get-iam-policy SA_EMAIL       # View SA policy
gcloud iam service-accounts add-iam-policy-binding SA_EMAIL --member=... --role=...

# --- SA Keys ---
gcloud iam service-accounts keys create key.json --iam-account=SA_EMAIL
gcloud iam service-accounts keys list --iam-account=SA_EMAIL
gcloud iam service-accounts keys delete KEY_ID --iam-account=SA_EMAIL

# --- Troubleshooting ---
gcloud policy-troubleshoot iam --principal-email=... --resource-name=... --permission=...

# --- Impersonation ---
gcloud COMMAND --impersonate-service-account=SA_EMAIL
gcloud config set auth/impersonate_service_account SA_EMAIL
```

---

## Official Documentation Links

| Topic | Link |
|-------|------|
| IAM Overview | https://cloud.google.com/iam/docs/overview |
| Understanding Roles | https://cloud.google.com/iam/docs/understanding-roles |
| Granting/Revoking Access | https://cloud.google.com/iam/docs/granting-changing-revoking-access |
| Custom Roles | https://cloud.google.com/iam/docs/creating-custom-roles |
| Resource Hierarchy | https://cloud.google.com/iam/docs/resource-hierarchy-access-control |
| IAM Conditions | https://cloud.google.com/iam/docs/conditions-overview |
| Deny Policies | https://cloud.google.com/iam/docs/deny-overview |
| Troubleshoot Access | https://cloud.google.com/iam/docs/troubleshooting-access |
| Service Accounts Overview | https://cloud.google.com/iam/docs/service-account-overview |
| Create Service Accounts | https://cloud.google.com/iam/docs/service-accounts-create |
| Manage SA Access | https://cloud.google.com/iam/docs/manage-access-service-accounts |
| Attach SA to Resources | https://cloud.google.com/iam/docs/attach-service-accounts |
| SA Impersonation | https://cloud.google.com/iam/docs/service-account-impersonation |
| Short-Lived Credentials | https://cloud.google.com/iam/docs/create-short-lived-credentials-direct |
| Service Agents | https://cloud.google.com/iam/docs/service-agents |
| SA Key Best Practices | https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys |
| Workload Identity Federation | https://cloud.google.com/iam/docs/workload-identity-federation |
| Workload Identity for GKE | https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity |
| Org Policy Constraints | https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints |
| IAM Recommender | https://cloud.google.com/iam/docs/recommender-overview |

---

[Back to README](./README.md)
