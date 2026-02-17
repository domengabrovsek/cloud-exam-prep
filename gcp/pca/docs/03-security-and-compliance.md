# Section 3: Designing for Security and Compliance (~17.5% of Exam)

> **Quick context:** This section tests architect-level security design -- IAM at scale, encryption strategies, VPC Service Controls, compliance frameworks, and securing AI workloads. Expect 9-11 questions.

---

## 3.1 Designing for Security

### Identity and Access Management (IAM) at Scale

**Groups-based access management:**
The recommended pattern for enterprise IAM is to manage access through Google Groups (or Cloud Identity groups), not individual user assignments:

```
Google Group (e.g., gcp-developers@company.com)
  └── Granted roles/viewer on folder "Development"
       └── All group members inherit access
```

Benefits: centralized management, audit trail, lifecycle management, integration with identity providers.

```bash
# Grant a role to a group
gcloud projects add-iam-policy-binding my-project \
  --member="group:gcp-developers@company.com" \
  --role="roles/viewer"

# List IAM policy
gcloud projects get-iam-policy my-project --format=json
```

**Predefined vs Custom Roles:**

| Role Type | When to Use | Example |
|-----------|------------|---------|
| Basic (Owner/Editor/Viewer) | Never in production -- too broad | Testing only |
| Predefined | Default choice -- Google maintains | roles/compute.instanceAdmin.v1 |
| Custom | When predefined roles are too broad/narrow | Specific permission combinations |

```bash
# Create a custom role
gcloud iam roles create myCustomRole \
  --project=my-project \
  --title="Custom VM Starter" \
  --description="Can start and stop VMs only" \
  --permissions=compute.instances.start,compute.instances.stop,compute.instances.get

# List predefined roles for a service
gcloud iam roles list --filter="name:roles/compute.*"
```

**IAM Conditions (CEL expressions):**
Conditions allow granting access only when specific criteria are met:

| Condition Type | Example | Use Case |
|---------------|---------|----------|
| Resource type | `resource.type == "storage.googleapis.com/Bucket"` | Limit role to specific resource types |
| Resource name | `resource.name.startsWith("projects/my-project/zones/us-central1-a")` | Regional restrictions |
| Time-based | `request.time < timestamp("2026-06-01T00:00:00Z")` | Temporary access |
| Resource tags | `resource.matchTag("env", "production")` | Tag-based access control |
| IP-based | Via Access Context Manager (not direct IAM) | Network-based access |

```bash
# Grant role with time-based condition
gcloud projects add-iam-policy-binding my-project \
  --member="user:contractor@example.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=request.time < timestamp("2026-06-01T00:00:00Z"),title=temp-access,description=Temporary contractor access'
```

**IAM Deny Policies:**
- Evaluated BEFORE allow policies -- deny always wins
- Use cases: prevent accidental privilege escalation, enforce guardrails
- Can deny specific permissions regardless of allow policies
- Applied at org, folder, or project level

```bash
# Create a deny policy (using gcloud beta)
gcloud iam policies create my-deny-policy \
  --attachment-point=cloudresourcemanager.googleapis.com/projects/my-project \
  --kind=denypolicies \
  --policy-file=deny-policy.json
```

Example deny policy JSON:
```json
{
  "displayName": "Prevent public access",
  "rules": [{
    "denyRule": {
      "deniedPrincipals": ["principalSet://goog/public:all"],
      "deniedPermissions": ["iam.googleapis.com/roles.create"]
    }
  }]
}
```

**IAM Recommender:**
- Analyzes actual permission usage over 90 days
- Suggests removing unused permissions
- Recommends replacing overly broad roles with more specific ones
- Part of Active Assist

```bash
# List IAM recommendations
gcloud recommender recommendations list \
  --recommender=google.iam.policy.Recommender \
  --project=my-project \
  --location=global
```

**Exam tips:**
- "Least privilege at scale" → Groups + predefined roles + IAM Recommender
- "Temporary access" → IAM Conditions with time-based expression
- "Prevent specific actions globally" → IAM Deny Policies at org level
- Basic roles (Owner/Editor/Viewer) should NEVER be used in production -- exam expects predefined roles
- Deny policies are evaluated before allow -- this is a key exam concept
- IAM Recommender suggests changes; you must apply them (it's not automatic)

**Docs:** [IAM](https://cloud.google.com/iam/docs), [IAM Conditions](https://cloud.google.com/iam/docs/conditions-overview), [Deny Policies](https://cloud.google.com/iam/docs/deny-overview), [Recommender](https://cloud.google.com/recommender/docs/overview)

---

### Resource Hierarchy for Security

**Security boundary design:**

```
Organization (company.com)
├── Folder: Production
│   ├── Folder: Sensitive-data (org policies: CMEK required, VPC-SC enforced)
│   │   ├── Project: prod-healthcare (HIPAA workloads)
│   │   └── Project: prod-financial (PCI DSS workloads)
│   └── Folder: Standard
│       ├── Project: prod-web
│       └── Project: prod-api
├── Folder: Non-production
│   ├── Project: staging-all
│   └── Project: dev-all
├── Folder: Shared-services
│   ├── Project: shared-vpc-host
│   ├── Project: shared-monitoring
│   └── Project: shared-security (SCC, KMS)
└── Folder: Sandbox
    └── Project: sandbox-* (self-service, restricted budget)
```

**Key patterns:**
- **Shared VPC Host Project**: Centralized network management, teams get service projects
- **Shared Security Project**: Centralized KMS key rings, SCC, log sinks
- **Environment Isolation**: Separate folders for prod/non-prod with different org policies
- **Data classification**: Folders for sensitive data with stricter controls

**IAM inheritance:**
- Policies set at org level apply to ALL folders, projects, resources below
- Policies set at folder level apply to all projects/resources in that folder
- More permissive policies at lower levels ADD to (never remove from) inherited policies
- Use deny policies to enforce restrictions that can't be overridden

**Exam tips:**
- "Centralize network management" → Shared VPC with dedicated host project
- "Different security policies for different environments" → folders with org policies
- "Data classification enforcement" → folders + VPC Service Controls + CMEK
- IAM policies are additive going down -- you can grant more but not less (without deny policies)

**Docs:** [Resource Hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy), [Best Practices for Enterprise Organizations](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations)

---

### Data Security

**Encryption at rest -- three levels:**

| Type | Key Management | Who Manages | Use Case |
|------|---------------|-------------|----------|
| Google default encryption | Google-managed keys | Google | Default, no action needed |
| CMEK (Customer-Managed Encryption Keys) | Cloud KMS | Customer | Compliance, key lifecycle control |
| CSEK (Customer-Supplied Encryption Keys) | Customer provides key per API call | Customer | Full key control, customer holds key material |

**Cloud KMS key hierarchy:**

```
Project
  └── Key Ring (regional -- cannot be deleted)
       └── Crypto Key (rotation policy, protection level)
            └── Key Version (the actual cryptographic material)
                 States: ENABLED → DISABLED → SCHEDULED_FOR_DESTRUCTION → DESTROYED
```

```bash
# Create a key ring
gcloud kms keyrings create my-ring --location=us-central1

# Create a crypto key with automatic rotation
gcloud kms keys create my-key \
  --keyring=my-ring \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time=2026-04-01T00:00:00Z

# Encrypt data
gcloud kms encrypt \
  --key=my-key --keyring=my-ring --location=us-central1 \
  --plaintext-file=secret.txt --ciphertext-file=secret.enc

# Disable a key version (data becomes inaccessible)
gcloud kms keys versions disable 1 \
  --key=my-key --keyring=my-ring --location=us-central1
```

**Cloud HSM** (Hardware Security Module):
- FIPS 140-2 Level 3 certified
- Keys never leave the HSM
- Higher cost, same API as software KMS
- Set `--protection-level=hsm` when creating key

**Cloud EKM** (External Key Manager):
- Keys stored in customer's external key management system (Thales, Fortanix, etc.)
- Google Cloud never sees the key material
- Adds latency (external call for every operation)
- For extreme compliance requirements (data sovereignty, hold-your-own-key)

**Key Access Justifications:**
- Shows reason why Google needs to access keys (customer support, maintenance, etc.)
- Customer can approve/deny access
- Only available with EKM

**Secret Manager:**
- Store API keys, passwords, certificates
- Automatic versioning, rotation hooks, fine-grained IAM
- Integration with Cloud Run, GKE (as mounted volumes), Cloud Functions

```bash
# Create a secret
gcloud secrets create my-secret --replication-policy=automatic

# Add a secret version
echo -n "my-password" | gcloud secrets versions add my-secret --data-file=-

# Access a secret
gcloud secrets versions access latest --secret=my-secret

# Grant access to a service account
gcloud secrets add-iam-policy-binding my-secret \
  --member="serviceAccount:my-sa@my-project.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

**Sensitive Data Protection (formerly DLP):**
- Inspect, classify, de-identify sensitive data
- Supports 150+ built-in detectors (SSN, credit card, email, etc.)
- De-identification methods: masking, tokenization, bucketing, format-preserving encryption
- Integrates with BigQuery, Cloud Storage, Datastore

**Exam tips:**
- "Customer must control encryption keys" → CMEK (Cloud KMS)
- "Customer must hold key material outside Google" → CSEK or Cloud EKM
- "FIPS 140-2 Level 3 compliance" → Cloud HSM
- "Store application secrets" → Secret Manager (not Cloud KMS -- KMS is for encryption keys)
- Key rings cannot be deleted once created
- Destroying a key version has a 24-hour default scheduled destruction period (configurable)
- CMEK + Cloud EKM = strongest key control for data sovereignty

**Docs:** [Cloud KMS](https://cloud.google.com/kms/docs), [Secret Manager](https://cloud.google.com/secret-manager/docs), [Sensitive Data Protection](https://cloud.google.com/sensitive-data-protection/docs), [Cloud EKM](https://cloud.google.com/kms/docs/ekm)

---

### Separation of Duties

**Principle:** No single person should have complete control over a critical process.

**Implementation patterns:**

| Pattern | Implementation |
|---------|---------------|
| Key management vs data access | KMS admin role ≠ data accessor role |
| Network management vs application deployment | Network Admin ≠ Compute Admin |
| Security admin vs project admin | Security Admin manages policies, Project Admin manages resources |
| Billing admin vs resource creator | Billing Admin manages accounts, devs cannot modify billing |
| Infrastructure provisioning vs approval | Terraform plan (dev) → apply (ops) via CI/CD |

```bash
# Example: separate KMS admin from data user
# KMS Admin can manage keys but NOT use them to encrypt/decrypt
gcloud kms keyrings add-iam-policy-binding my-ring \
  --location=us-central1 \
  --member="group:kms-admins@company.com" \
  --role="roles/cloudkms.admin"

# Data team can encrypt/decrypt but NOT manage keys
gcloud kms keyrings add-iam-policy-binding my-ring \
  --location=us-central1 \
  --member="group:data-team@company.com" \
  --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"
```

**Exam tips:**
- "Ensure no single person can access and manage encryption keys" → separate roles/cloudkms.admin from roles/cloudkms.cryptoKeyEncrypterDecrypter
- Separation of duties questions test whether you understand that admin roles and usage roles should be held by different principals

**Docs:** [Separation of Duties](https://cloud.google.com/iam/docs/understanding-roles#separation_of_duties)

---

### Security Controls

**VPC Service Controls (VPC-SC):**

The most heavily tested security feature on the PCA exam. VPC-SC creates security perimeters around Google Cloud resources to prevent data exfiltration.

```
                    VPC Service Perimeter
┌──────────────────────────────────────────────┐
│  Project A          Project B                │
│  ┌──────────┐       ┌──────────┐            │
│  │ BigQuery │ ←───→ │ GCS      │  ✅ Allowed│
│  └──────────┘       └──────────┘            │
│                                              │
└──────────────────────────────────────────────┘
         ↕ BLOCKED
┌──────────────────┐
│ Project C        │
│ (outside perimeter)│
│ Cannot access A/B │
└──────────────────┘
```

**Key concepts:**
- **Service perimeter**: Boundary around projects/VPCs that restricts data movement
- **Access levels**: Conditions under which perimeter access is allowed (IP range, device, identity)
- **Ingress rules**: Allow specific external access INTO the perimeter
- **Egress rules**: Allow specific access FROM the perimeter to external resources
- **Perimeter bridges**: Allow two perimeters to communicate (bidirectional)
- **Dry-run mode**: Test perimeter config without enforcement

```bash
# Create an access policy
gcloud access-context-manager policies create \
  --organization=123456789 --title="My Policy"

# Create a service perimeter
gcloud access-context-manager perimeters create my-perimeter \
  --policy=POLICY_ID \
  --title="Production Perimeter" \
  --resources=projects/111,projects/222 \
  --restricted-services=bigquery.googleapis.com,storage.googleapis.com \
  --access-levels=accessPolicies/POLICY_ID/accessLevels/corp-network

# Create an access level (IP-based)
gcloud access-context-manager levels create corp-network \
  --policy=POLICY_ID \
  --title="Corporate Network" \
  --basic-level-spec=level-spec.yaml

# Update perimeter with ingress rule
gcloud access-context-manager perimeters update my-perimeter \
  --policy=POLICY_ID \
  --set-ingress-policies=ingress.yaml

# Dry-run mode (test without enforcement)
gcloud access-context-manager perimeters dry-run create my-perimeter \
  --policy=POLICY_ID \
  --resources=projects/111 \
  --restricted-services=bigquery.googleapis.com
```

**VPC-SC supported services (commonly tested):**
BigQuery, Cloud Storage, Cloud SQL, Spanner, Bigtable, Pub/Sub, Dataflow, Dataproc, Cloud KMS, Vertex AI, Artifact Registry, Container Registry, Secret Manager, Cloud Functions, Cloud Run.

**Organization Policies as Security Guardrails:**

| Constraint | Purpose |
|-----------|---------|
| `constraints/compute.vmExternalIpAccess` | Control which VMs can have external IPs |
| `constraints/iam.allowedPolicyMemberDomains` | Restrict who can be granted IAM roles |
| `constraints/compute.restrictVpnPeerIPs` | Control VPN peer addresses |
| `constraints/gcp.resourceLocations` | Restrict resource creation to specific regions |
| `constraints/iam.disableServiceAccountKeyCreation` | Prevent SA key creation |
| `constraints/compute.skipDefaultNetworkCreation` | No default VPC in new projects |
| `constraints/sql.restrictPublicIp` | No public IPs on Cloud SQL |
| `constraints/storage.uniformBucketLevelAccess` | Enforce uniform access on GCS |

**Hierarchical Firewall Policies:**
- Applied at organization or folder level
- Evaluated BEFORE VPC firewall rules
- Can "goto_next" to allow VPC-level rules to decide, or "allow"/"deny" to override

```bash
# Create a hierarchical firewall policy
gcloud compute firewall-policies create \
  --organization=123456789 \
  --short-name=org-firewall

# Add a rule
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=org-firewall \
  --direction=INGRESS \
  --action=deny \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:22 \
  --description="Deny SSH from internet globally"

# Associate with a folder
gcloud compute firewall-policies associations create \
  --firewall-policy=org-firewall \
  --folder=123456789
```

**Security Command Center (SCC):**
- Central security and risk dashboard
- **Standard tier** (free): Security Health Analytics, Web Security Scanner
- **Premium tier**: Event Threat Detection, Container Threat Detection, Virtual Machine Threat Detection, Rapid Vulnerability Detection
- Findings from multiple sources: Security Health Analytics, third-party tools
- Compliance reports (CIS benchmarks, PCI DSS, NIST)

**Exam tips:**
- "Prevent data exfiltration from BigQuery/GCS" → VPC Service Controls
- "Allow specific external access to perimeter" → VPC-SC ingress rules
- "Test perimeter before enforcement" → dry-run mode
- "Enforce policy across entire organization" → org policy constraints
- "Override VPC-level firewall rules" → hierarchical firewall policies
- VPC-SC does NOT work with App Engine -- this is an exam trap
- SCC Premium is needed for threat detection -- Standard only does health analytics

**Docs:** [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs), [Organization Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview), [Hierarchical Firewall Policies](https://cloud.google.com/firewall/docs/firewall-policies-overview), [Security Command Center](https://cloud.google.com/security-command-center/docs)

---

### Managing CMEK with Cloud KMS

(Detailed coverage in Data Security section above -- this section covers integration patterns)

**CMEK integration with services:**

| Service | CMEK Support | Key Scope |
|---------|-------------|-----------|
| Cloud Storage | Per bucket | Bucket-level |
| BigQuery | Per dataset | Dataset-level |
| Cloud SQL | Per instance | Instance-level |
| Spanner | Per database | Database-level |
| Compute Engine (disk) | Per disk | Disk-level |
| GKE | Boot disk + etcd | Cluster-level |
| Pub/Sub | Per topic | Topic-level |
| Secret Manager | Per secret | Secret-level |
| Vertex AI | Per resource | Resource-level |
| Artifact Registry | Per repository | Repository-level |

```bash
# Create a Cloud SQL instance with CMEK
gcloud sql instances create my-instance \
  --database-version=POSTGRES_15 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --disk-encryption-key=projects/my-project/locations/us-central1/keyRings/my-ring/cryptoKeys/my-key

# Create a GCS bucket with CMEK
gcloud storage buckets create gs://my-bucket \
  --location=us-central1 \
  --default-encryption-key=projects/my-project/locations/us-central1/keyRings/my-ring/cryptoKeys/my-key

# Create a BigQuery dataset with CMEK
bq mk --dataset \
  --default_kms_key=projects/my-project/locations/us/keyRings/my-ring/cryptoKeys/my-key \
  my-project:my_dataset
```

**Org policy to enforce CMEK:**
```bash
# Require CMEK for all Cloud Storage buckets
gcloud resource-manager org-policies set-policy cmek-policy.yaml \
  --organization=123456789
```

**Exam tips:**
- The KMS service account for each Google service needs `roles/cloudkms.cryptoKeyEncrypterDecrypter` on the key
- Key ring location must match resource location (us-central1 key ring for us-central1 resources)
- If a key is destroyed, all data encrypted with it becomes permanently inaccessible (crypto-shredding)
- CMEK adds a small latency overhead for key retrieval
- Use org policies to enforce CMEK at the organizational level

**Docs:** [CMEK](https://cloud.google.com/kms/docs/cmek), [Org Policy for CMEK](https://cloud.google.com/kms/docs/cmek-org-policy)

---

### Secure Remote Access

**Identity-Aware Proxy (IAP):**
- Provides identity-based access to applications and VMs
- No VPN required -- works over HTTPS
- Integrates with IAM for authorization
- Supports TCP forwarding (SSH, RDP) and HTTP(S) applications

```bash
# SSH to a VM through IAP (no external IP needed)
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap

# Create a TCP forwarding tunnel
gcloud compute start-iap-tunnel my-vm 3389 --local-host-port=localhost:3389 --zone=us-central1-a

# Set IAM policy for IAP access
gcloud iap web add-iam-policy-binding \
  --member="user:dev@company.com" \
  --role="roles/iap.httpsResourceAccessor" \
  --resource-type=app-engine
```

**Service Account Impersonation:**
- Short-lived credentials instead of long-lived keys
- User/SA assumes another SA's identity temporarily
- Requires `roles/iam.serviceAccountTokenCreator` on target SA

```bash
# Impersonate a service account
gcloud compute instances list \
  --impersonate-service-account=my-sa@my-project.iam.gserviceaccount.com

# Generate an access token for a service account
gcloud auth print-access-token \
  --impersonate-service-account=my-sa@my-project.iam.gserviceaccount.com
```

**Chrome Enterprise Premium (formerly BeyondCorp Enterprise):**
- Zero-trust access model
- Context-aware access based on device posture, user identity, location
- Integrates with IAP and Access Context Manager
- Device trust evaluation (managed, encrypted, up-to-date OS)

**Workload Identity Federation:**
- Allows external identities (AWS, Azure, GitHub Actions, OIDC, SAML) to access GCP without service account keys
- Maps external identity to a GCP service account
- Eliminates need for long-lived credentials

| Provider | Protocol | Use Case |
|----------|----------|----------|
| AWS | AWS STS | Cross-cloud access |
| Azure AD | OIDC | Cross-cloud access |
| GitHub Actions | OIDC | CI/CD without SA keys |
| GitLab CI | OIDC | CI/CD without SA keys |
| Active Directory | SAML | On-premises workloads |
| Any OIDC provider | OIDC | Custom identity providers |

```bash
# Create a workload identity pool
gcloud iam workload-identity-pools create github-pool \
  --location=global \
  --description="GitHub Actions pool"

# Create a provider for GitHub
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --workload-identity-pool=github-pool \
  --location=global \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository"

# Bind to a service account
gcloud iam service-accounts add-iam-policy-binding my-sa@my-project.iam.gserviceaccount.com \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/attribute.repository/my-org/my-repo" \
  --role="roles/iam.workloadIdentityUser"
```

**OS Login:**
- SSH access management through IAM (replaces SSH key management)
- Users login with their Google identity
- Two roles: `roles/compute.osLogin` (standard) and `roles/compute.osAdminLogin` (sudo)
- Supports 2FA with `roles/compute.osLogin` + OS Login 2FA

**Exam tips:**
- "Access VMs without external IP" → IAP TCP forwarding
- "Eliminate service account keys" → Workload Identity Federation
- "Zero-trust access to web apps" → Chrome Enterprise Premium + IAP
- "Temporary elevated access" → service account impersonation
- "Manage SSH access through IAM" → OS Login
- Workload Identity Federation is the preferred alternative to SA keys for external workloads

**Docs:** [IAP](https://cloud.google.com/iap/docs), [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation), [OS Login](https://cloud.google.com/compute/docs/instances/managing-instance-access)

---

### Securing Software Supply Chain

**Binary Authorization:**
- Admission control for GKE -- only allows deploying container images that have been signed by trusted authorities
- Based on attestations (cryptographic signatures)
- Integrates with Artifact Registry vulnerability scanning

```bash
# Enable Binary Authorization on a GKE cluster
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --binauthz-evaluation-mode=PROJECT_SINGLETON_POLICY_ENFORCE

# Set default policy to require attestation
gcloud container binauthz policy export > policy.yaml
# Edit policy.yaml to require attestations
gcloud container binauthz policy import policy.yaml
```

**Artifact Registry:**
- Managed repository for container images, language packages (npm, Maven, Python, Go)
- Vulnerability scanning (automatic and on-demand)
- Remote repositories (proxy for Docker Hub, Maven Central, etc.)
- Virtual repositories (single endpoint for multiple backing repos)

```bash
# Create a Docker repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1

# Enable vulnerability scanning
gcloud artifacts repositories update my-repo \
  --location=us-central1 \
  --enable-vulnerability-scanning

# Scan an image
gcloud artifacts docker images scan us-central1-docker.pkg.dev/my-project/my-repo/my-image:latest
```

**SLSA Framework (Supply chain Levels for Software Artifacts):**
- Level 1: Build process documented
- Level 2: Automated build, version control
- Level 3: Hardened build platform, provenance verification
- Level 4: Two-person review, hermetic/reproducible build

**Cloud Build integration:**
- Cloud Build can generate SLSA provenance metadata
- Build attestations can be verified by Binary Authorization
- Private pools provide network isolation for builds
- Custom workers for specific build requirements

**Software Delivery Shield:**
- End-to-end software supply chain security
- Spans development, build, deploy, and runtime
- Includes Cloud Workstations, Cloud Code, Cloud Build, Artifact Registry, Binary Authorization, GKE

**Exam tips:**
- "Only allow verified images in production GKE" → Binary Authorization
- "Scan container images for vulnerabilities" → Artifact Registry scanning
- "Secure the entire CI/CD pipeline" → Software Delivery Shield
- Binary Authorization works at the cluster level -- it's enforced on GKE admission
- Artifact Registry replaces Container Registry (GCR) -- GCR is legacy

**Docs:** [Binary Authorization](https://cloud.google.com/binary-authorization/docs), [Artifact Registry](https://cloud.google.com/artifact-registry/docs), [Software Delivery Shield](https://cloud.google.com/software-supply-chain-security/docs/overview)

---

### Securing AI

**Model Armor:**
- Content safety filters for AI/ML models
- Screens prompts and responses for harmful content
- Supports Vertex AI models and third-party models
- Categories: hate speech, harassment, dangerous content, sexually explicit, self-harm
- Configurable sensitivity levels

**Sensitive Data Protection for AI:**
- Inspect training data for PII before model training
- De-identify sensitive data in model inputs/outputs
- Redact PII from prompts and responses
- Integration with Vertex AI pipelines

**Secure Model Deployment:**
- Use VPC Service Controls around Vertex AI endpoints
- Private endpoints (no public internet access)
- CMEK for model artifacts and predictions
- IAM for endpoint access control

**Data Governance for Training Data:**
- Data lineage tracking in Dataplex
- Dataset access controls in Vertex AI
- Audit logging for model access and predictions

**Exam tips:**
- "Filter harmful content from AI responses" → Model Armor
- "Protect PII in training data" → Sensitive Data Protection
- "Prevent data exfiltration from Vertex AI" → VPC Service Controls
- AI security is new on the PCA exam -- expect 1-2 questions

**Docs:** [Model Armor](https://cloud.google.com/vertex-ai/docs/generative-ai/model-armor), [Responsible AI](https://cloud.google.com/responsible-ai)

---

## 3.2 Designing for Compliance

### Legislation and Regulation

**HIPAA (Health Insurance Portability and Accountability Act):**
- Applies to Protected Health Information (PHI)
- Google Cloud has a Business Associate Agreement (BAA)
- BAA covers specific services (Compute Engine, GKE, Cloud SQL, Cloud Storage, BigQuery, Pub/Sub, Dataflow, etc.)
- NOT all Google Cloud services are covered -- check the BAA services list
- Key requirements: encryption (at rest + in transit), access controls, audit logging, breach notification

**Implementation on GCP:**
- Use Assured Workloads to create HIPAA-compliant environments
- CMEK for encryption key control
- VPC Service Controls for data boundary
- Cloud Audit Logs for access auditing
- Access Transparency + Access Approval for Google admin visibility

**COPPA (Children's Online Privacy Protection Act):**
- Applies to collecting data from children under 13
- Requires verifiable parental consent
- Limits data collection and retention
- GCP implementation: Sensitive Data Protection to detect child data, retention policies, strict access controls

**GDPR (General Data Protection Regulation):**
- EU data privacy law -- applies to any organization processing EU resident data
- Key rights: right to access, right to erasure ("right to be forgotten"), right to portability, right to restrict processing
- Data Processing Agreement (DPA) available from Google
- Data residency: may require EU-only storage

**Implementation on GCP:**
- EU regional/multi-regional storage (europe-west regions)
- Assured Workloads for EU data sovereignty
- CMEK with Cloud EKM (keys held in EU)
- Crypto-shredding: destroy the encryption key to make data unrecoverable (implements right to erasure)
- Sensitive Data Protection for PII detection and de-identification
- Organization policy `constraints/gcp.resourceLocations` to restrict to EU regions

```bash
# Restrict resource locations to EU
gcloud resource-manager org-policies set-policy location-policy.yaml --organization=ORG_ID

# location-policy.yaml example:
# constraint: constraints/gcp.resourceLocations
# listPolicy:
#   allowedValues:
#     - in:europe-west1-locations
#     - in:europe-west4-locations
```

**Exam tips:**
- "HIPAA compliance for healthcare data" → Assured Workloads + BAA + CMEK + VPC-SC + Audit Logs
- "GDPR right to be forgotten" → Crypto-shredding (destroy CMEK key)
- "EU data residency" → org policy for resource locations + EU regional storage + Assured Workloads
- "Data sovereignty" → Cloud EKM (keys held outside Google, in country of choice)
- Not all services are covered by BAA -- always check the list

**Docs:** [HIPAA on GCP](https://cloud.google.com/security/compliance/hipaa), [GDPR](https://cloud.google.com/privacy/gdpr), [Data Residency](https://cloud.google.com/architecture/framework/security/data-residency)

---

### Commercial Compliance

**PCI DSS (Payment Card Industry Data Security Standard):**
- Applies to handling credit card data (cardholder data environment)
- Google Cloud is PCI DSS Level 1 certified (highest level)
- Shared responsibility: Google secures infrastructure, customer secures workloads
- Key controls: network segmentation, encryption, access control, logging, vulnerability management

**Implementation on GCP:**
- Dedicated project for cardholder data environment
- VPC Service Controls for network segmentation
- CMEK for encryption
- Cloud Armor for web application firewall
- Sensitive Data Protection to detect/mask credit card numbers
- Cloud Audit Logs for all access

**PII Handling Best Practices:**
- Classify data (public, internal, confidential, restricted)
- Use Sensitive Data Protection to discover and classify PII
- De-identify PII before analytics (tokenization, masking, generalization)
- Encrypt PII at rest and in transit
- Apply column-level security in BigQuery for PII columns

```bash
# Inspect content for PII
gcloud dlp inspect-content --content="My SSN is 123-45-6789" \
  --info-types=US_SOCIAL_SECURITY_NUMBER

# De-identify (mask SSN)
gcloud dlp deidentify-content \
  --content="My SSN is 123-45-6789" \
  --info-types=US_SOCIAL_SECURITY_NUMBER \
  --deidentify-config=mask-config.json
```

**Data Retention Requirements:**
- Different regulations mandate different retention periods
- Use GCS retention policies (Bucket Lock for WORM compliance)
- Use BigQuery dataset default table expiration
- Use Cloud Logging custom retention on log buckets

**Exam tips:**
- "Process credit card payments securely" → PCI DSS + VPC-SC + CMEK + Cloud Armor
- "Detect and mask PII" → Sensitive Data Protection (DLP)
- "Immutable data retention for compliance" → GCS Bucket Lock (WORM)
- PCI DSS compliance is shared responsibility -- Google provides the platform, you secure the app

**Docs:** [PCI DSS](https://cloud.google.com/security/compliance/pci-dss), [Sensitive Data Protection](https://cloud.google.com/sensitive-data-protection/docs)

---

### Industry Certifications

**Google Cloud compliance certifications:**

| Certification | Scope | Relevance |
|--------------|-------|-----------|
| SOC 1/2/3 | Financial reporting, security controls | General enterprise compliance |
| ISO 27001 | Information security management | International standard |
| ISO 27017 | Cloud security controls | Cloud-specific security |
| ISO 27018 | PII protection in cloud | Privacy |
| FedRAMP (High, Moderate) | US federal government | Government workloads |
| HITRUST | Healthcare information trust | Healthcare (supplement to HIPAA) |
| CSA STAR | Cloud Security Alliance | Cloud security assessment |

**Accessing compliance reports:**
- Compliance Reports Manager in Cloud Console
- Artifact Hub (download SOC reports, ISO certificates)
- Google Cloud trust center: cloud.google.com/security/compliance

**Exam tips:**
- "US federal government workload" → FedRAMP certified services + Assured Workloads
- "Financial audit requirements" → SOC 2 Type II report
- Google Cloud compliance does NOT automatically make YOUR application compliant -- shared responsibility

**Docs:** [Compliance Offerings](https://cloud.google.com/security/compliance/offerings)

---

### Audits and Logs

**Cloud Audit Logs -- 4 types:**

| Log Type | Content | Default State | Retention |
|----------|---------|--------------|-----------|
| **Admin Activity** | Resource config changes (create, delete, update) | Always on, cannot disable | 400 days (free) |
| **Data Access** | Read resource data, read/write user data | Off by default (except BigQuery) | 30 days default |
| **System Event** | Google-initiated system actions | Always on, cannot disable | 400 days (free) |
| **Policy Denied** | Denied access due to security policy | Always on, cannot disable | 30 days default |

```bash
# Read admin activity logs
gcloud logging read 'logName="projects/PROJECT/logs/cloudaudit.googleapis.com%2Factivity"' \
  --limit=10

# Enable Data Access logs for a service
gcloud projects get-iam-policy PROJECT --format=json > policy.json
# Edit policy.json to add auditConfigs section
gcloud projects set-iam-policy PROJECT policy.json

# Export audit logs to BigQuery for long-term analysis
gcloud logging sinks create audit-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com"'
```

**Access Transparency:**
- Shows logs of Google personnel accessing your data (customer support, engineering)
- Available with Premium or Enterprise support
- Read-only -- you can see but not block

**Access Approval:**
- Allows you to approve or deny Google personnel access to your data
- Builds on Access Transparency
- Can set auto-approve or manual approval per service

```bash
# List Access Approval requests
gcloud access-approval requests list --project=PROJECT_ID

# Approve a request
gcloud access-approval requests approve REQUEST_NAME
```

**Long-term log retention architecture:**
- Default: _Required bucket (400 days, immutable), _Default bucket (30 days)
- For compliance: Create custom log buckets with longer retention
- Export to BigQuery for SQL analysis (log sinks)
- Export to GCS for archival (lifecycle → Coldline → Archive)

**Exam tips:**
- Admin Activity logs are always on and free for 400 days -- never disable them
- Data Access logs must be explicitly enabled (except BigQuery)
- "Audit all access to patient data" → Enable Data Access logs + export to BigQuery
- "See when Google accesses your data" → Access Transparency
- "Approve Google access to your data" → Access Approval
- Log sinks can export to BigQuery (analysis), GCS (archival), Pub/Sub (real-time processing)
- Policy Denied logs help identify blocked access attempts (useful for VPC-SC troubleshooting)

**Docs:** [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit), [Access Transparency](https://cloud.google.com/assured-workloads/access-transparency/docs), [Access Approval](https://cloud.google.com/assured-workloads/access-approval/docs)

---

### Assured Workloads

**What it is:** Compliance-bound folders that enforce data residency, personnel controls, and encryption requirements.

**Supported compliance frameworks:**

| Framework | Region | Key Restrictions |
|-----------|--------|-----------------|
| FedRAMP High | US | US data residency, US personnel, CMEK |
| FedRAMP Moderate | US | US data residency |
| IL4 (Impact Level 4) | US | US data residency, US personnel, restricted services |
| CJIS | US | US data residency, background-checked personnel |
| ITAR | US | US data residency, US person access only |
| EU Regions and Support | EU | EU data residency, EU support personnel |
| Canada Regions | Canada | Canada data residency |
| Australia Regions | Australia | Australia data residency |

**How it works:**
1. Create an Assured Workloads folder with a compliance framework
2. Google automatically applies org policy constraints (resource locations, service restrictions)
3. Deploy compliant resources within the folder
4. Continuous monitoring for compliance violations

```bash
# Create an Assured Workloads folder
gcloud assured workloads create \
  --organization=ORG_ID \
  --display-name="HIPAA Production" \
  --compliance-regime=HIPAA \
  --location=us-central1 \
  --billing-account=BILLING_ACCOUNT
```

**What Assured Workloads enforces:**
- Resource location restrictions (only in allowed regions)
- Service restrictions (only compliance-qualified services)
- Encryption requirements (CMEK enforcement)
- Personnel access controls (only cleared personnel for support)
- Key management requirements (Cloud KMS or EKM)

**Exam tips:**
- "Create a HIPAA-compliant environment" → Assured Workloads folder with HIPAA regime
- "Ensure US-only data residency for government" → Assured Workloads with FedRAMP/IL4
- "EU data sovereignty" → Assured Workloads with EU Regions and Support
- Assured Workloads is the one-stop solution for compliance -- it configures org policies, service restrictions, and key management automatically
- Not all services are available in Assured Workloads -- restricted to compliant services
- Assured Workloads creates a folder with pre-configured org policies -- you deploy resources inside it

**Docs:** [Assured Workloads](https://cloud.google.com/assured-workloads/docs), [Supported Services](https://cloud.google.com/assured-workloads/docs/supported-products)
