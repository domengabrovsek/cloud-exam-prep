# Section 4: Configuring Access and Security

> **Exam weight:** ~20% of the standard ACE exam, ~20% of the renewal exam.
>
> This section covers managing IAM (viewing policies, assigning roles, managing custom roles), managing service accounts, managing and viewing audit logs, and configuring access controls (Workload Identity, IAM deny policies, organization policy constraints for key management).

**Total questions: 23**

---

### Q1. A junior developer needs to view VM instances and their configurations but should not be able to create, modify, or delete any VMs. What is the least-privilege predefined role you should grant?

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

### Q2. You have a predefined role that is too broad and a narrower predefined role that is missing one permission your application needs. What should you do?

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

### Q3. You need to create a service account for a Cloud Run microservice that reads objects from a Cloud Storage bucket and publishes messages to Pub/Sub. Following least privilege, which roles should you grant?

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

### Q4. A developer needs to deploy a Cloud Run service that runs as a specific service account `app-sa@project.iam.gserviceaccount.com`. The developer has `roles/run.admin` on the project. The deployment fails with a permission error related to the service account. What is missing?

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

### Q5. A CI/CD pipeline running on GitHub Actions needs to deploy resources to GCP. You want to avoid storing long-lived service account keys in GitHub. What is the recommended approach?

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

### Q6. Your organization's default Compute Engine service account has `roles/editor` on the project. A new VM is created without specifying a service account. What is the security risk?

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

### Q7. You need to allow a user to generate short-lived access tokens for a service account to test an API integration, without giving them the ability to deploy resources as that service account. Which role should you grant on the service account?

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

### Q8. A GKE pod needs to write data to a Cloud Storage bucket. Your company policy prohibits service account keys. What is the recommended approach?

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

### Q9. Your organization wants to prevent anyone from creating service account keys across all projects. What is the most effective way to enforce this?

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

### Q10. A user has `roles/editor` at the organization level through inheritance. You need to prevent them from deleting Cloud Storage objects in a specific production project while keeping all their other permissions. What should you do?

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

### Q11. You are using `gcloud` to impersonate a service account for testing. Which command sets impersonation for all subsequent gcloud commands in the session?

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

### Q12. You need to view the current IAM policy for a project to audit who has access. Which command do you use?

A) `gcloud iam roles list --project=my-project`
B) `gcloud projects get-iam-policy my-project`
C) `gcloud iam service-accounts list --project=my-project`
D) `gcloud resource-manager get-policy my-project`

<details>
<summary>Answer</summary>

**Correct: B)**

`gcloud projects get-iam-policy` returns the full IAM policy for a project -- all bindings of members to roles. `iam roles list` (A) lists available roles, not who has them. `service-accounts list` (C) lists SAs, not the full policy. `resource-manager get-policy` (D) isn't the correct command syntax.
</details>

---

### Q13. You need to create a custom IAM role that combines permissions from two predefined roles but excludes a few specific permissions. What's the correct approach?

A) Assign both predefined roles and use an IAM deny policy to block the unwanted permissions
B) Create a custom role specifying only the exact permissions needed
C) Edit one of the predefined roles to remove permissions
D) Use IAM Conditions to filter out unwanted permissions

<details>
<summary>Answer</summary>

**Correct: B)**

Custom roles let you pick exactly the permissions you need -- no more, no less. `gcloud iam roles create` with a list of specific permissions. You cannot edit predefined roles (C). Deny policies (A) work but are more complex than necessary. IAM Conditions (D) filter based on attributes (time, resource), not permission removal.
</details>

---

### Q14. A service account needs to interact with Cloud Storage and Pub/Sub but nothing else. Following the principle of least privilege, how should you configure it?

A) Grant `roles/editor` to the service account
B) Grant `roles/storage.objectAdmin` and `roles/pubsub.editor` to the service account
C) Grant only the specific predefined roles needed: `roles/storage.objectViewer` (if read-only) or `roles/storage.objectCreator` (if write) and `roles/pubsub.publisher` or `roles/pubsub.subscriber` as needed
D) Use the default service account since it already has these permissions

<details>
<summary>Answer</summary>

**Correct: C)**

Least privilege means granting the narrowest roles that match actual needs. If the SA only reads from Storage and publishes to Pub/Sub, use `objectViewer` + `publisher`, not the broader `objectAdmin` + `editor`. Editor (A) and default SA (D) are far too broad. Option B is better than A/D but still broader than necessary.
</details>

---

### Q15. You create a new service account and need to assign it to an existing Compute Engine VM. What are the two things you need?

A) `roles/iam.serviceAccountUser` on the SA + stop the VM, change the SA, start the VM
B) Just run `gcloud compute instances set-service-account` -- no additional role needed
C) `roles/iam.serviceAccountAdmin` + restart the VM
D) Create a new VM -- you can't change a VM's service account after creation

<details>
<summary>Answer</summary>

**Correct: A)**

To assign a SA to a VM, you need `roles/iam.serviceAccountUser` on the service account (to "act as" it). The VM must be stopped first, then you can change the SA with `gcloud compute instances set-service-account`, then start it again. You can change a VM's SA (D is wrong), but the VM must be stopped. `serviceAccountAdmin` (C) manages the SA itself, not the ability to use it.
</details>

---

### Q16. A developer needs to test their application locally using a service account's permissions. They want to avoid downloading a key file. What should they use?

A) `gcloud auth application-default login --impersonate-service-account=SA_EMAIL`
B) Download the SA key and set `GOOGLE_APPLICATION_CREDENTIALS`
C) `gcloud auth login` with their own account
D) Create a VM with the SA attached and develop on that VM

<details>
<summary>Answer</summary>

**Correct: A)**

Impersonating a service account via `--impersonate-service-account` generates short-lived credentials without needing a key file. The developer needs `roles/iam.serviceAccountTokenCreator` on the SA. Downloading keys (B) is a security risk. Own account (C) doesn't test SA permissions. Developing on a VM (D) is impractical for local development.
</details>

---

### Q17. You need to generate a short-lived access token for a service account to pass to an external system that needs temporary GCP access. What do you use?

A) `gcloud iam service-accounts keys create` to create a persistent key
B) `gcloud auth print-access-token --impersonate-service-account=SA_EMAIL`
C) `gcloud iam service-accounts enable SA_EMAIL`
D) Store the SA credentials in Secret Manager

<details>
<summary>Answer</summary>

**Correct: B)**

`gcloud auth print-access-token` with impersonation generates a short-lived OAuth2 access token (default 1 hour). No persistent keys involved. Creating keys (A) produces long-lived credentials -- a security risk. Enable (C) activates a disabled SA but doesn't generate tokens. Secret Manager (D) stores secrets but doesn't generate tokens.
</details>

---

### Q18. Your GKE application needs to access Cloud Spanner. Following Google best practices, how should you authenticate the application?

A) Mount a service account key as a Kubernetes secret
B) Use the node's default service account
C) Configure Workload Identity to bind the Kubernetes service account to a GCP service account with Spanner permissions
D) Embed the service account key in the container image

<details>
<summary>Answer</summary>

**Correct: C)**

Workload Identity is Google's recommended way for GKE workloads to authenticate to GCP services. It binds a Kubernetes SA to a GCP SA -- no keys needed, follows least privilege per workload. Keys in secrets (A) or images (D) are security risks. The node's default SA (B) gives the same permissions to all pods on the node.
</details>

---

### Q19. You have a service account that was compromised. You need to immediately revoke all access. What's the fastest action?

A) Delete all service account keys
B) Remove all IAM bindings for the service account
C) Disable the service account
D) Delete the service account

<details>
<summary>Answer</summary>

**Correct: C)**

`gcloud iam service-accounts disable SA_EMAIL` immediately prevents the SA from authenticating -- all existing tokens become invalid. It's fast and reversible. Deleting keys (A) doesn't revoke existing tokens immediately. Removing IAM bindings (B) is slow if the SA has many bindings across projects. Deleting (D) works but is irreversible and may break dependencies.
</details>

---

### Q20. You need to ensure that a specific IAM permission is NEVER granted to anyone in your organization, regardless of allow policies. What mechanism do you use?

A) Remove the permission from all custom roles
B) Create an organization-level IAM deny policy
C) Set an org policy constraint
D) Revoke the permission from all users manually

<details>
<summary>Answer</summary>

**Correct: B)**

IAM deny policies are evaluated BEFORE allow policies. A deny at the org level overrides any allow at any level below. Removing from custom roles (A) doesn't affect predefined roles. Org policies (C) constrain resource creation, not IAM permissions. Manual revocation (D) doesn't prevent future grants.
</details>

---

### Q21. A team member has the `roles/iam.serviceAccountUser` role on a service account and `roles/compute.instanceAdmin` on a project. What can they do?

A) Only manage VMs, not use the service account
B) Create VMs that run as that service account, effectively gaining the SA's permissions on resources those VMs access
C) Only manage the service account itself (create keys, disable, delete)
D) Nothing -- these roles conflict with each other

<details>
<summary>Answer</summary>

**Correct: B)**

`serviceAccountUser` lets you deploy resources (like VMs) that "act as" the service account. Combined with `compute.instanceAdmin`, they can create VMs running as that SA. This is why `serviceAccountUser` is a powerful role -- it enables privilege escalation through the SA's permissions. The roles don't conflict (D). `serviceAccountAdmin` (not User) manages the SA itself (C).
</details>

---

### Q22. Your organization plans to migrate its financial transaction monitoring application to Google Cloud. Auditors need to view the data and run reports in BigQuery, but they are not allowed to perform transactions in the application. You are leading the migration and want the simplest solution that will require the least amount of maintenance. What should you do?

A) Assign `roles/bigquery.dataViewer` to the individual auditors.
B) Create a group for auditors and assign `roles/viewer` to them.
C) Create a group for auditors, and assign `roles/bigquery.dataViewer` to them.
D) Assign a custom role to each auditor that allows view-only access to BigQuery.

<details>
<summary>Answer</summary>

**Correct: C)**

Two principles are at play -- **least privilege** and **least maintenance**. Option C satisfies both. Using a **group** means you manage membership once instead of per-user (eliminates A and D for maintenance). `roles/bigquery.dataViewer` is a predefined role scoped specifically to BigQuery read access (eliminates B, where `roles/viewer` is a basic role granting read access to *all* resources, far too broad). Custom roles (D) require ongoing maintenance when permissions change.

> **Exam tip:** Whenever you see "least maintenance" + IAM, think **groups + predefined roles**.
</details>

---

### Q23. You are managing your company's first Google Cloud project. Project leads, developers, and internal testers will participate in the project, which includes sensitive information. You need to ensure that only specific members of the development team have access to sensitive information. You want to assign the appropriate IAM roles that also require the least amount of maintenance. What should you do?

A) Assign a basic role to each user.
B) Create groups. Assign a basic role to each group, and then assign users to groups.
C) Create groups. Assign a Custom role to each group, including those who should have access to sensitive data. Assign users to groups.
D) Create groups. Assign an IAM Predefined role to each group as required, including those who should have access to sensitive data. Assign users to groups.

<details>
<summary>Answer</summary>

**Correct: D)**

Groups are essential for maintainability (eliminates A). Basic roles (B) are too broad and cannot differentiate access to sensitive data -- `roles/editor` and `roles/viewer` apply to all resources. Custom roles (C) work but add maintenance overhead (you must update permissions manually when Google adds new features). **Predefined roles** (D) are maintained by Google, scoped to specific services, and allow you to grant different levels of access to different groups (e.g., one group gets `bigquery.dataViewer`, another gets `bigquery.dataEditor`).

> **Exam tip:** Google best practice hierarchy: **Predefined roles > Custom roles > Basic roles**. Only use custom roles when no predefined role fits.
</details>

---

## Answer Key

| Q | Answer | Source | Topic |
|---|--------|--------|-------|
| 1 | B | 07-Q50 | Least privilege -- compute.viewer |
| 2 | B | 07-Q51 | Custom IAM roles |
| 3 | B | 07-Q52 | Least privilege SA roles |
| 4 | B | 07-Q53 | serviceAccountUser for deployment |
| 5 | B | 07-Q54 | Workload Identity Federation |
| 6 | B | 07-Q55 | Default SA security risk |
| 7 | B | 07-Q56 | serviceAccountTokenCreator |
| 8 | C | 07-Q57 | GKE Workload Identity |
| 9 | B | 07-Q58 | Org policy disable key creation |
| 10 | C | 07-Q59 | IAM deny policies |
| 11 | B | 07-Q60 | gcloud SA impersonation |
| 12 | B | 11-Q51 | View project IAM policy |
| 13 | B | 11-Q52 | Custom IAM roles |
| 14 | C | 11-Q53 | Least privilege SA permissions |
| 15 | A | 11-Q54 | Assign SA to VM |
| 16 | A | 11-Q55 | SA impersonation without keys |
| 17 | B | 11-Q56 | Short-lived access tokens |
| 18 | C | 11-Q57 | Workload Identity for GKE |
| 19 | C | 11-Q58 | Disable compromised SA |
| 20 | B | 11-Q59 | IAM deny policies |
| 21 | B | 11-Q60 | serviceAccountUser privilege escalation |
| 22 | C | 08-Q1 | Groups + predefined roles for auditors |
| 23 | D | 08-Q2 | Groups + predefined roles for projects |
