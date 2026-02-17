# Key Commands and Terraform Reference

> **Architect-level CLI and IaC reference.** Unlike ACE which tests basic commands, PCA tests understanding of advanced CLI usage, Terraform best practices, and infrastructure orchestration.

---

## 1. gcloud Commands at Architect Level

### Organization and Resource Hierarchy

```bash
# List organizations
gcloud organizations list

# List org policies
gcloud resource-manager org-policies list --organization=ORG_ID

# Set an org policy (boolean constraint)
gcloud resource-manager org-policies enable-enforce \
  constraints/compute.skipDefaultNetworkCreation \
  --organization=ORG_ID

# Set an org policy (list constraint)
gcloud resource-manager org-policies set-policy policy.yaml \
  --project=PROJECT_ID

# Create folders
gcloud resource-manager folders create --display-name="Production" --organization=ORG_ID
gcloud resource-manager folders create --display-name="Dev" --folder=PARENT_FOLDER_ID
```

### IAM at Scale

```bash
# Add IAM binding with condition
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:devs@company.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=resource.matchTag("env","dev"),title=dev-only'

# Create custom role
gcloud iam roles create customViewer \
  --project=PROJECT_ID \
  --title="Custom Viewer" \
  --permissions=compute.instances.get,compute.instances.list

# Create deny policy
gcloud iam policies create deny-sa-keys \
  --attachment-point=cloudresourcemanager.googleapis.com/organizations/ORG_ID \
  --kind=denypolicies \
  --policy-file=deny-sa-keys.json

# Service account impersonation
gcloud auth print-access-token --impersonate-service-account=SA@PROJECT.iam.gserviceaccount.com

# Workload Identity Federation - create pool
gcloud iam workload-identity-pools create github-pool --location=global
gcloud iam workload-identity-pools providers create-oidc github \
  --workload-identity-pool=github-pool --location=global \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub"

# IAM Recommender
gcloud recommender recommendations list \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID --location=global
```

### VPC Service Controls

```bash
# Create access policy
gcloud access-context-manager policies create --organization=ORG_ID --title="Corp Policy"

# Create access level
gcloud access-context-manager levels create corp-network \
  --policy=POLICY_ID --title="Corp Network" \
  --basic-level-spec=spec.yaml

# Create service perimeter
gcloud access-context-manager perimeters create prod-perimeter \
  --policy=POLICY_ID --title="Production" \
  --resources=projects/111,projects/222 \
  --restricted-services=bigquery.googleapis.com,storage.googleapis.com

# Dry-run mode
gcloud access-context-manager perimeters dry-run create test-perimeter \
  --policy=POLICY_ID --resources=projects/111 \
  --restricted-services=storage.googleapis.com

# Update with ingress/egress rules
gcloud access-context-manager perimeters update prod-perimeter \
  --policy=POLICY_ID --set-ingress-policies=ingress.yaml
```

### Networking (Architect Level)

```bash
# Create Shared VPC host project
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Attach service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# Create HA VPN gateway
gcloud compute vpn-gateways create my-vpn-gw --network=my-vpc --region=us-central1

# Create Cloud Router
gcloud compute routers create my-router --network=my-vpc --region=us-central1 --asn=65001

# Create VPN tunnels (HA -- need two tunnels)
gcloud compute vpn-tunnels create tunnel-0 \
  --vpn-gateway=my-vpn-gw --peer-gcp-gateway=peer-gw \
  --region=us-central1 --ike-version=2 --shared-secret=SECRET \
  --router=my-router --vpn-gateway-interface=0

# Create hierarchical firewall policy
gcloud compute firewall-policies create --organization=ORG_ID --short-name=org-fw
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=org-fw --direction=INGRESS --action=deny \
  --src-ip-ranges=0.0.0.0/0 --layer4-configs=tcp:22

# Private Service Connect endpoint
gcloud compute forwarding-rules create psc-endpoint \
  --region=us-central1 --network=my-vpc \
  --subnet=psc-subnet --address=psc-ip \
  --target-service-attachment=SERVICE_ATTACHMENT

# Create Cloud Interconnect (Dedicated)
gcloud compute interconnects create my-interconnect \
  --interconnect-type=DEDICATED --link-type=LINK_TYPE_ETHERNET_10G_LR \
  --location=FACILITY --requested-link-count=1

# Load balancer (global external Application LB)
gcloud compute backend-services create my-backend --global --protocol=HTTP
gcloud compute url-maps create my-url-map --default-service=my-backend
gcloud compute target-https-proxies create my-proxy --url-map=my-url-map --ssl-certificates=my-cert
gcloud compute forwarding-rules create my-lb --global --target-https-proxy=my-proxy --ports=443
```

### Cloud KMS

```bash
# Create key ring and key
gcloud kms keyrings create my-ring --location=us-central1
gcloud kms keys create my-key --keyring=my-ring --location=us-central1 \
  --purpose=encryption --rotation-period=90d

# HSM key
gcloud kms keys create my-hsm-key --keyring=my-ring --location=us-central1 \
  --purpose=encryption --protection-level=hsm

# Encrypt/decrypt
gcloud kms encrypt --key=my-key --keyring=my-ring --location=us-central1 \
  --plaintext-file=data.txt --ciphertext-file=data.enc
gcloud kms decrypt --key=my-key --keyring=my-ring --location=us-central1 \
  --ciphertext-file=data.enc --plaintext-file=data.txt

# Key version management
gcloud kms keys versions disable VERSION --key=my-key --keyring=my-ring --location=us-central1
gcloud kms keys versions destroy VERSION --key=my-key --keyring=my-ring --location=us-central1
```

### Vertex AI

```bash
# Create a custom training job
gcloud ai custom-jobs create \
  --region=us-central1 \
  --display-name=my-training \
  --worker-pool-spec=machine-type=n1-standard-8,replica-count=1,container-image-uri=gcr.io/my-project/trainer:latest

# Deploy a model to an endpoint
gcloud ai endpoints create --region=us-central1 --display-name=my-endpoint
gcloud ai endpoints deploy-model ENDPOINT_ID \
  --region=us-central1 --model=MODEL_ID \
  --display-name=my-deployment --traffic-split=0=100

# List models
gcloud ai models list --region=us-central1

# Create a pipeline run
gcloud ai pipelines runs create \
  --region=us-central1 --display-name=my-pipeline \
  --template-uri=gs://my-bucket/pipeline.yaml
```

### Cloud Deploy

```bash
# Create a delivery pipeline
gcloud deploy apply --file=pipeline.yaml --region=us-central1

# Create a release
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-pipeline --region=us-central1 \
  --source=. --images=my-app=gcr.io/my-project/my-app:v1

# Promote to next stage
gcloud deploy releases promote --release=release-001 \
  --delivery-pipeline=my-pipeline --region=us-central1

# Rollback
gcloud deploy targets rollback my-target \
  --delivery-pipeline=my-pipeline --region=us-central1
```

### Monitoring and Logging

```bash
# Create alerting policy
gcloud monitoring policies create --policy-from-file=alert-policy.json

# Create log sink to BigQuery
gcloud logging sinks create my-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/audit_logs \
  --log-filter='logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity"'

# Create custom log bucket
gcloud logging buckets create my-bucket \
  --location=us-central1 --retention-days=365

# Read logs
gcloud logging read 'severity>=ERROR AND timestamp>="2026-02-01T00:00:00Z"' \
  --project=PROJECT_ID --limit=100

# Create log-based metric
gcloud logging metrics create error-count \
  --description="Count of error logs" \
  --log-filter='severity>=ERROR'
```

### GKE (Architect Level)

```bash
# Create private regional cluster with Workload Identity
gcloud container clusters create my-cluster \
  --region=us-central1 --num-nodes=3 \
  --enable-private-nodes --master-ipv4-cidr=172.16.0.0/28 \
  --enable-ip-alias --enable-master-authorized-networks \
  --master-authorized-networks=10.0.0.0/8 \
  --workload-pool=PROJECT_ID.svc.id.goog \
  --release-channel=regular

# GKE Autopilot
gcloud container clusters create-auto my-autopilot \
  --region=us-central1 --release-channel=regular

# Enable Binary Authorization
gcloud container clusters update my-cluster --region=us-central1 \
  --binauthz-evaluation-mode=PROJECT_SINGLETON_POLICY_ENFORCE

# Fleet management (GKE Enterprise)
gcloud container fleet memberships register my-cluster \
  --gke-cluster=us-central1/my-cluster --enable-workload-identity

# Config Sync
gcloud beta container fleet config-management apply \
  --membership=my-cluster --config=config-sync.yaml

# Backup for GKE
gcloud beta container backup-restore backup-plans create my-plan \
  --cluster=my-cluster --location=us-central1 \
  --all-namespaces --cron-schedule="0 2 * * *"
```

**Exam tips for gcloud:**
- `--impersonate-service-account` works with any gcloud command
- `--format=json|yaml|csv|table` for output formatting
- `--filter` and `--format` are powerful for scripting
- `gcloud config configurations` for managing multiple environments
- Most architect-level commands use `gcloud beta` or `gcloud alpha`

---

## 2. Terraform Deep Dive

### Provider Configuration

```hcl
# Required provider configuration
terraform {
  required_version = ">= 1.5"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Beta provider for preview features
provider "google-beta" {
  project = var.project_id
  region  = var.region
}
```

### State Management

**GCS Backend (recommended for teams):**

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "environments/production"
  }
}
```

**State best practices:**
- Always use remote state (GCS) for team environments
- Enable versioning on the state bucket
- Enable encryption (default or CMEK)
- Use state locking (automatic with GCS backend)
- Separate state per environment (different prefixes or buckets)

```bash
# State management commands
terraform state list                     # List all resources
terraform state show google_compute_instance.my_vm  # Show resource details
terraform state mv google_compute_instance.old google_compute_instance.new  # Rename
terraform state rm google_compute_instance.legacy  # Remove from state (doesn't destroy)
terraform import google_compute_instance.existing projects/P/zones/Z/instances/NAME  # Import
```

**Remote state data source (cross-project references):**

```hcl
data "terraform_remote_state" "network" {
  backend = "gcs"
  config = {
    bucket = "network-terraform-state"
    prefix = "vpc"
  }
}

# Reference outputs from another state
resource "google_compute_instance" "my_vm" {
  network_interface {
    subnetwork = data.terraform_remote_state.network.outputs.subnet_id
  }
}
```

### Modules

**Module structure:**
```
modules/
├── gke-cluster/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── versions.tf
├── cloud-sql/
│   └── ...
└── vpc/
    └── ...
```

**Using modules:**

```hcl
# Public registry module
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 9.0"

  project_id   = var.project_id
  network_name = "my-vpc"
  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "us-central1"
    }
  ]
}

# Local module
module "gke" {
  source = "./modules/gke-cluster"

  project_id  = var.project_id
  cluster_name = "prod-cluster"
  region      = "us-central1"
}
```

**Cloud Foundation Toolkit (CFT) modules:**
- Google-maintained Terraform modules following best practices
- Cover: project-factory, VPC, GKE, IAM, Cloud SQL, log-export, and more
- Source: `terraform-google-modules` GitHub org

```hcl
# CFT Project Factory
module "project" {
  source  = "terraform-google-modules/project-factory/google"
  version = "~> 15.0"

  name                 = "my-project"
  org_id              = var.org_id
  billing_account     = var.billing_account
  folder_id           = var.folder_id
  default_service_account = "disable"
  activate_apis = [
    "compute.googleapis.com",
    "container.googleapis.com",
  ]
}

# CFT GKE
module "gke" {
  source  = "terraform-google-modules/kubernetes-engine/google//modules/private-cluster"
  version = "~> 31.0"

  project_id        = module.project.project_id
  name              = "prod-cluster"
  region            = "us-central1"
  network           = module.vpc.network_name
  subnetwork        = module.vpc.subnets_names[0]
  ip_range_pods     = "pods"
  ip_range_services = "services"
  enable_private_nodes = true
  master_ipv4_cidr_block = "172.16.0.0/28"
}
```

### Key Resource Examples

```hcl
# GKE Autopilot cluster
resource "google_container_cluster" "autopilot" {
  provider = google-beta

  name     = "autopilot-cluster"
  location = "us-central1"

  enable_autopilot = true

  release_channel {
    channel = "REGULAR"
  }

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block = "172.16.0.0/28"
  }

  master_authorized_networks_config {
    cidr_blocks {
      cidr_block   = "10.0.0.0/8"
      display_name = "internal"
    }
  }
}

# Cloud Run service
resource "google_cloud_run_v2_service" "api" {
  name     = "my-api"
  location = "us-central1"

  template {
    containers {
      image = "us-central1-docker.pkg.dev/my-project/my-repo/my-api:latest"
      resources {
        limits = {
          cpu    = "2"
          memory = "1Gi"
        }
      }
      env {
        name = "DB_SECRET"
        value_source {
          secret_key_ref {
            secret  = google_secret_manager_secret.db_password.secret_id
            version = "latest"
          }
        }
      }
    }
    scaling {
      min_instance_count = 1
      max_instance_count = 100
    }
  }
}

# Cloud SQL with private IP and CMEK
resource "google_sql_database_instance" "main" {
  name             = "prod-db"
  database_version = "POSTGRES_15"
  region           = "us-central1"
  encryption_key_name = google_kms_crypto_key.sql_key.id

  settings {
    tier              = "db-custom-4-16384"
    availability_type = "REGIONAL"

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id
    }

    backup_configuration {
      enabled                        = true
      point_in_time_recovery_enabled = true
      start_time                     = "02:00"
      transaction_log_retention_days = 7
    }
  }

  depends_on = [google_service_networking_connection.private_vpc_connection]
}
```

### Best Practices

**Environment separation:**

```
environments/
├── production/
│   ├── main.tf
│   ├── terraform.tfvars
│   └── backend.tf      # GCS bucket: tf-state/prod
├── staging/
│   ├── main.tf
│   ├── terraform.tfvars
│   └── backend.tf      # GCS bucket: tf-state/staging
└── modules/             # Shared modules
    ├── gke/
    └── vpc/
```

Prefer separate directories over Terraform workspaces for environment isolation.

**CI/CD integration:**

```yaml
# Cloud Build + Terraform
steps:
  - id: 'tf-init'
    name: 'hashicorp/terraform:1.7'
    args: ['init']
    dir: 'environments/production'

  - id: 'tf-plan'
    name: 'hashicorp/terraform:1.7'
    args: ['plan', '-out=tfplan']
    dir: 'environments/production'

  - id: 'tf-apply'
    name: 'hashicorp/terraform:1.7'
    args: ['apply', '-auto-approve', 'tfplan']
    dir: 'environments/production'
```

**Policy validation:**

| Tool | Type | Use Case |
|------|------|----------|
| `terraform validate` | Syntax | Basic HCL validation |
| `terraform plan` | Drift | Preview changes before apply |
| Sentinel (Terraform Cloud) | Policy | Enterprise policy enforcement |
| OPA/Conftest | Policy | Open-source policy-as-code |
| Google Cloud Policy Validator | Compliance | Validate against CIS benchmarks |
| tflint | Linting | Terraform-specific linting |
| checkov | Security | Security scanning for IaC |

**Terraform vs alternatives:**

| Feature | Terraform | Config Connector | Deployment Manager | Pulumi |
|---------|-----------|-------------------|-------------------|--------|
| Language | HCL | Kubernetes YAML | YAML/Jinja/Python | Python/TypeScript/Go |
| State | Explicit (GCS) | Kubernetes API | Google-managed | Explicit |
| Multi-cloud | Yes | No (GCP only) | No (GCP only) | Yes |
| Ecosystem | Largest | Limited | Limited | Growing |
| Status | Recommended | Active | Deprecated | Active |

**Exam tips for Terraform:**
- "Infrastructure as Code for GCP" --> Terraform (default answer)
- "Kubernetes-native resource management" --> Config Connector
- State locking prevents concurrent modifications
- `terraform import` brings existing resources under Terraform management
- CFT modules are the Google-recommended starting point
- Deployment Manager is DEPRECATED -- Terraform is the replacement

**Docs:** [Terraform on Google Cloud](https://cloud.google.com/docs/terraform), [CFT](https://cloud.google.com/foundation-toolkit), [Config Connector](https://cloud.google.com/config-connector/docs)

---

## 3. kubectl for GKE Architects

### Cluster Access

```bash
# Get cluster credentials
gcloud container clusters get-credentials my-cluster --region=us-central1

# Switch context
kubectl config use-context gke_PROJECT_us-central1_my-cluster
kubectl config get-contexts
kubectl config current-context
```

### RBAC Configuration

```yaml
# ClusterRole - cluster-wide permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-global
subjects:
- kind: Group
  name: "gke-developers@company.com"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# Namespace-scoped Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: deployment-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]

---
# RoleBinding (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: production
  name: manage-deployments
subjects:
- kind: Group
  name: "prod-deployers@company.com"
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# Deny all ingress to a namespace (default deny)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Allow specific ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Resource Quotas and Limits

```yaml
# ResourceQuota per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"

---
# LimitRange (defaults for containers)
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-a
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

### Scaling and Rollouts

```bash
# HPA
kubectl autoscale deployment my-app --cpu-percent=70 --min=2 --max=20

# Rolling update
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2

# Canary deployment (manual)
kubectl set image deployment/my-app my-app=image:v2
kubectl rollout pause deployment/my-app   # After updating some pods
kubectl rollout resume deployment/my-app  # Continue rollout
```

**Exam tips for kubectl:**
- RBAC: ClusterRole/ClusterRoleBinding for cluster-wide, Role/RoleBinding for namespace-scoped
- Network Policies require a CNI that supports them (Calico on GKE -- enabled by default on Standard)
- ResourceQuota is per namespace -- use for multi-tenant clusters
- `kubectl rollout undo` rolls back to previous revision

**Docs:** [GKE RBAC](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control), [Network Policies](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy)

---

## 4. Cloud Emulators Reference

| Emulator | Install | Start | Environment Variable |
|----------|---------|-------|---------------------|
| Pub/Sub | `gcloud components install pubsub-emulator` | `gcloud beta emulators pubsub start --project=test` | `PUBSUB_EMULATOR_HOST` |
| Bigtable | `gcloud components install bigtable` | `gcloud beta emulators bigtable start` | `BIGTABLE_EMULATOR_HOST` |
| Spanner | `gcloud components install cloud-spanner-emulator` | `gcloud emulators spanner start` | `SPANNER_EMULATOR_HOST` |
| Firestore | `gcloud components install cloud-firestore-emulator` | `gcloud beta emulators firestore start` | `FIRESTORE_EMULATOR_HOST` |
| Datastore | Built-in | `gcloud beta emulators datastore start` | `DATASTORE_EMULATOR_HOST` |

```bash
# Set environment (example for Pub/Sub)
$(gcloud beta emulators pubsub env-init)

# Or manually
export PUBSUB_EMULATOR_HOST=localhost:8085
```

**Limitations:**
- Emulators don't support IAM, encryption, or VPC-SC
- Performance characteristics differ from production
- Not all features are emulated (check docs per emulator)
- Use for development and unit testing only

**Exam tips:**
- "Test locally without incurring costs" --> Cloud emulators
- "Emulators don't replicate production security" --> use separate test projects for integration testing
- Spanner emulator is the most feature-complete

**Docs:** [Cloud Emulators](https://cloud.google.com/sdk/gcloud/reference/beta/emulators)

---

## 5. gsutil and bq at Architect Level

### gsutil Advanced

```bash
# Parallel composite upload (for large files)
gsutil -o GSUtil:parallel_composite_upload_threshold=150M cp large-file.tar.gz gs://bucket/

# Generate signed URL (temporary access)
gsutil signurl -d 1h key.json gs://bucket/object

# Set lifecycle policy
gsutil lifecycle set lifecycle.json gs://bucket/

# Cross-project copy with service account
gsutil -i sa@project.iam.gserviceaccount.com cp gs://src-bucket/file gs://dst-bucket/

# Configure notifications (to Pub/Sub)
gsutil notification create -t my-topic -f json gs://my-bucket
```

### bq Advanced

```bash
# Authorized view (cross-dataset access)
bq mk --view 'SELECT col1, col2 FROM `project.dataset.table` WHERE condition' \
  project:authorized_dataset.authorized_view

# Scheduled query
bq mk --transfer_config \
  --data_source=scheduled_query \
  --display_name="Daily aggregation" \
  --schedule="every 24 hours" \
  --params='{"query":"SELECT ...","destination_table_name_template":"agg_{run_date}"}'

# Slot reservations (BigQuery editions)
bq mk --reservation --project_id=admin-project --location=us \
  --reservation_id=prod-reservation --slots=500 --edition=ENTERPRISE

# Query INFORMATION_SCHEMA
bq query --use_legacy_sql=false \
  'SELECT * FROM `project`.INFORMATION_SCHEMA.TABLES WHERE table_schema="my_dataset"'

# Dry run (estimate query cost)
bq query --dry_run --use_legacy_sql=false 'SELECT * FROM `project.dataset.table`'
```

**Exam tips:**
- `gsutil signurl` generates pre-signed URLs for temporary access without IAM
- `bq --dry_run` estimates query cost without running it
- BigQuery slot reservations provide predictable pricing for heavy analytics
- INFORMATION_SCHEMA queries are free (metadata only)

**Docs:** [gsutil](https://cloud.google.com/storage/docs/gsutil), [bq CLI](https://cloud.google.com/bigquery/docs/reference/bq-cli-reference)
