# Domain 3: Deploying and Implementing a Cloud Solution (~25%)

> **This is the heaviest domain on the exam.** Expect 5-7 questions out of ~25. Most questions are scenario-based: "You need to deploy X with Y requirements. What do you do?"

---

## 3.1 Deploying and Implementing Compute Engine Resources

### 3.1.1 Launching a Compute Instance

A Compute Engine VM is the most fundamental IaaS resource in GCP. You need to know how to configure disks, availability policies, and SSH keys via both the Console and `gcloud`.

**Key flags to remember:**

```bash
# Create a basic instance
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud

# With a custom boot disk (size + type)
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd \
  --image-family=debian-12 \
  --image-project=debian-cloud

# Attach an additional data disk
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --create-disk=name=data-disk,size=200GB,type=pd-balanced,auto-delete=no

# Attach an existing disk to a running instance
gcloud compute instances attach-disk my-vm \
  --disk=my-existing-disk \
  --zone=us-central1-a

# Set availability policy (maintenance behavior + restart)
gcloud compute instances create my-vm \
  --maintenance-policy=MIGRATE \
  --restart-on-failure

# Set availability policy for preemptible/spot
gcloud compute instances create my-vm \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Add project-wide SSH keys (metadata)
gcloud compute project-info add-metadata \
  --metadata=ssh-keys="username:ssh-rsa AAAA...key username"

# Add instance-level SSH keys
gcloud compute instances add-metadata my-vm \
  --metadata=ssh-keys="username:ssh-rsa AAAA...key username"

# Block project-wide SSH keys on a specific instance
gcloud compute instances add-metadata my-vm \
  --metadata=block-project-ssh-keys=TRUE

# SSH into an instance
gcloud compute ssh my-vm --zone=us-central1-a
```

**Exam tips:**
- `--maintenance-policy=MIGRATE` (default) = live migration during host maintenance. Use `TERMINATE` for GPUs or sole-tenant nodes (they cannot live migrate).
- `--restart-on-failure` is enabled by default. Preemptible/Spot VMs can be reclaimed at any time.
- Boot disk is deleted by default when the VM is deleted. Additional disks are NOT deleted by default.
- `pd-standard` = HDD, `pd-balanced` = balanced SSD, `pd-ssd` = high-perf SSD, `pd-extreme` = highest IOPS.

Docs: [Creating and starting a VM instance](https://cloud.google.com/compute/docs/instances/create-start-instance)

---

### 3.1.2 Creating an Autoscaled Managed Instance Group (MIG)

MIGs provide identical VMs from a template with autoscaling, autohealing, and rolling updates.

```bash
# Step 1: Create an instance template
gcloud compute instance-templates create my-template \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --tags=http-server

# Step 2: Create a managed instance group from the template
gcloud compute instance-groups managed create my-mig \
  --zone=us-central1-a \
  --template=my-template \
  --size=2

# Step 2 (regional MIG - spread across zones in a region)
gcloud compute instance-groups managed create my-mig \
  --region=us-central1 \
  --template=my-template \
  --size=3

# Step 3: Configure autoscaling
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --min-num-replicas=1 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.60 \
  --cool-down-period=90

# Configure autohealing with a health check
gcloud compute health-checks create http my-health-check \
  --port=80 \
  --check-interval=30s \
  --healthy-threshold=1 \
  --unhealthy-threshold=3

gcloud compute instance-groups managed update my-mig \
  --zone=us-central1-a \
  --health-check=my-health-check \
  --initial-delay=300

# Rolling update to a new template
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=my-template-v2 \
  --zone=us-central1-a \
  --max-surge=1 \
  --max-unavailable=0

# Stop autoscaling
gcloud compute instance-groups managed stop-autoscaling my-mig \
  --zone=us-central1-a

# Manually resize
gcloud compute instance-groups managed resize my-mig \
  --size=5 \
  --zone=us-central1-a
```

**Exam tips:**
- Instance templates are **immutable**. To change config, create a new template and do a rolling update.
- Regional MIGs spread across 3 zones by default -- better for HA.
- Autoscaling signals: CPU utilization, HTTP load balancing utilization, Cloud Monitoring metrics, or schedules.
- Canary deployment = rolling update with two versions (e.g., 80% v1, 20% v2).

Docs: [Instance groups overview](https://cloud.google.com/compute/docs/instance-groups) | [Autoscaling](https://cloud.google.com/compute/docs/autoscaler)

---

### 3.1.3 Configuring OS Login

OS Login links Linux user accounts to Google identities (IAM). It replaces metadata-based SSH key management and integrates with IAM for access control.

```bash
# Enable OS Login at the project level
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE

# Enable OS Login on a specific instance
gcloud compute instances add-metadata my-vm \
  --metadata enable-oslogin=TRUE

# Grant IAM roles for OS Login access
# Basic SSH access:
gcloud projects add-iam-policy-binding my-project \
  --member=user:alice@example.com \
  --role=roles/compute.osLogin

# SSH access with sudo:
gcloud projects add-iam-policy-binding my-project \
  --member=user:alice@example.com \
  --role=roles/compute.osAdminLogin

# For users outside the organization (external users):
gcloud projects add-iam-policy-binding my-project \
  --member=user:external@gmail.com \
  --role=roles/compute.osLoginExternalUser
```

**Exam tips:**
- OS Login is the **recommended** approach over metadata-managed SSH keys.
- `roles/compute.osLogin` = SSH only. `roles/compute.osAdminLogin` = SSH + sudo.
- OS Login supports 2FA (two-factor authentication) via `enable-oslogin-2fa=TRUE`.
- When OS Login is enabled, metadata SSH keys are disabled on that instance.

Docs: [Setting up OS Login](https://cloud.google.com/compute/docs/instances/managing-instance-access)

---

### 3.1.4 Configuring VM Manager

VM Manager is a suite of tools for managing OS patches, configs, and inventory at scale across your VM fleet.

Key components:
- **OS Patch Management** -- automate OS patching across VMs.
- **OS Configuration Management** -- deploy and manage software configurations.
- **OS Inventory Management** -- view OS details and installed packages.

```bash
# Enable the OS Config API (required for VM Manager)
gcloud services enable osconfig.googleapis.com

# Ensure the OS Config agent is running on VMs
# (auto-installed on most public images; verify with):
# sudo systemctl status google-osconfig-agent

# Set metadata to enable OS Config agent at project level
gcloud compute project-info add-metadata \
  --metadata=enable-osconfig=TRUE

# Create a patch job (patch all VMs in a zone)
gcloud compute os-config patch-jobs execute \
  --instance-filter-zones=us-central1-a

# Create a patch deployment (scheduled patching)
gcloud compute os-config patch-deployments create my-patch-schedule \
  --file=patch-deployment.json

# List OS inventory for instances
gcloud compute os-config inventories list \
  --location=us-central1-a

# Describe inventory for a specific instance
gcloud compute os-config inventories describe \
  --location=us-central1-a \
  my-vm
```

**Exam tips:**
- VM Manager requires the **OS Config agent** running on each VM and the **OS Config API** enabled.
- Use patch deployments for recurring patching schedules (e.g., every Tuesday at 2 AM).
- VM Manager works with both Linux and Windows VMs.

Docs: [VM Manager overview](https://cloud.google.com/compute/docs/vm-manager)

---

## 3.2 Deploying and Implementing Google Kubernetes Engine (GKE) Resources

### 3.2.1 Installing and Configuring kubectl

`kubectl` is the CLI for interacting with Kubernetes clusters. GKE uses `gcloud` to fetch credentials, then you use `kubectl` for cluster operations.

```bash
# Install kubectl via gcloud
gcloud components install kubectl

# Verify installation
kubectl version --client

# Get credentials for a GKE cluster (writes to ~/.kube/config)
gcloud container clusters get-credentials my-cluster \
  --zone=us-central1-a \
  --project=my-project

# Get credentials for a regional cluster
gcloud container clusters get-credentials my-cluster \
  --region=us-central1

# View current kubeconfig context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context (if managing multiple clusters)
kubectl config use-context gke_my-project_us-central1-a_my-cluster
```

**Exam tips:**
- `gcloud container clusters get-credentials` is the command that connects kubectl to a GKE cluster. This is frequently tested.
- The kubeconfig file lives at `~/.kube/config` by default.
- You can use the `gke-gcloud-auth-plugin` for kubectl authentication (required for GKE v1.26+).

Docs: [Install kubectl](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)

---

### 3.2.2 Deploying a GKE Cluster

#### Standard Cluster (Zonal)

```bash
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-standard-4 \
  --disk-size=50 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5
```

#### Standard Cluster (Regional -- HA)

```bash
gcloud container clusters create my-cluster \
  --region=us-central1 \
  --num-nodes=1 \
  --machine-type=e2-standard-4 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=3
# Note: --num-nodes is PER ZONE. A regional cluster spans 3 zones,
# so --num-nodes=1 actually creates 3 nodes total.
```

#### Autopilot Cluster

```bash
gcloud container clusters create-auto my-autopilot-cluster \
  --region=us-central1
# That's it. Google manages nodes, scaling, security, and networking.
```

#### Private Cluster

```bash
gcloud container clusters create my-private-cluster \
  --zone=us-central1-a \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --enable-ip-alias \
  --enable-master-authorized-networks \
  --master-authorized-networks=203.0.113.0/24
# --enable-private-nodes = nodes have no public IPs
# --enable-private-endpoint = control plane has no public IP
# --master-authorized-networks = restrict who can reach control plane
```

#### GKE Enterprise (formerly Anthos)

GKE Enterprise extends GKE to multi-cluster, multi-cloud, and on-prem environments.

```bash
# Enable the GKE Enterprise API
gcloud services enable anthos.googleapis.com

# Register a cluster to a fleet
gcloud container fleet memberships register my-cluster \
  --gke-cluster=us-central1-a/my-cluster \
  --enable-workload-identity

# List fleet memberships
gcloud container fleet memberships list
```

**Autopilot vs Standard -- key differences:**

| Feature | Autopilot | Standard |
|---------|-----------|----------|
| Node management | Google-managed | You manage node pools |
| Billing | Per-pod resource requests | Per-node (VM) |
| GPUs | Supported (with constraints) | Full GPU support |
| Node SSH | Not available | Available |
| DaemonSets | Restricted (allowlisted only) | Fully supported |
| Privileged pods | Not allowed | Allowed |
| OS choice | Container-Optimized OS only | COS or Ubuntu |
| Best for | Most workloads, less ops | Custom node configs, GPUs, special workloads |

**Exam tips:**
- **Autopilot** manages node pools for you. Use **Standard** if you need full control over nodes, GPUs with specific configs, or privileged containers.
- Regional clusters have **3 control plane replicas** and nodes across 3 zones -- use for production.
- Private clusters: nodes get **internal IPs only**. Use `--enable-private-endpoint` to also hide the control plane.
- `--num-nodes` in a regional cluster is **per zone**, not total.
- Know the difference between `create` (Standard) and `create-auto` (Autopilot).

Docs: [GKE overview](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview) | [Autopilot overview](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview) | [Private clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/private-clusters)

---

### 3.2.3 Deploying a Containerized Application to GKE

```bash
# Step 1: Build and push container image (to Artifact Registry)
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1

# Configure Docker to authenticate with Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev

# Build and push
docker build -t us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1 .
docker push us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1

# Step 2: Deploy to GKE
kubectl create deployment my-app \
  --image=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1

# Step 3: Expose the deployment via a LoadBalancer service
kubectl expose deployment my-app \
  --type=LoadBalancer \
  --port=80 \
  --target-port=8080

# Scale the deployment
kubectl scale deployment my-app --replicas=5

# Autoscale based on CPU
kubectl autoscale deployment my-app \
  --min=2 --max=10 --cpu-percent=70

# Rolling update to a new image
kubectl set image deployment/my-app \
  my-app=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v2

# Check rollout status
kubectl rollout status deployment/my-app

# Rollback
kubectl rollout undo deployment/my-app

# Apply from a YAML manifest
kubectl apply -f deployment.yaml

# Common debugging commands
kubectl get pods
kubectl get services
kubectl describe pod my-pod-xyz
kubectl logs my-pod-xyz
kubectl exec -it my-pod-xyz -- /bin/sh
```

**Exam tips:**
- Use **Artifact Registry** (not the deprecated Container Registry) to store images.
- Service types: `ClusterIP` (internal), `NodePort` (port on each node), `LoadBalancer` (external L4 LB).
- `kubectl apply -f` is the declarative approach (preferred). `kubectl create` is imperative.
- Know `kubectl rollout undo` for quick rollbacks.

Docs: [Deploying a containerized app](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-workloads-overview) | [Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling)

---

## 3.3 Deploying and Implementing Cloud Run and Cloud Functions

### 3.3.1 Deploying an Application to Cloud Run

Cloud Run is a fully managed serverless platform for running containers. It scales to zero and supports any language/framework/binary.

```bash
# Deploy from source code (Cloud Run builds the container for you)
gcloud run deploy my-service \
  --source=. \
  --region=us-central1

# Deploy from a container image
gcloud run deploy my-service \
  --image=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1 \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --port=8080 \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=100 \
  --concurrency=80 \
  --set-env-vars=KEY=value

# Deploy with a Cloud SQL connection
gcloud run deploy my-service \
  --image=my-image \
  --region=us-central1 \
  --add-cloudsql-instances=my-project:us-central1:my-db \
  --set-env-vars=DB_HOST=/cloudsql/my-project:us-central1:my-db

# Deploy with a VPC connector (to reach internal resources)
gcloud run deploy my-service \
  --image=my-image \
  --region=us-central1 \
  --vpc-connector=my-connector \
  --vpc-egress=all-traffic

# Update traffic splitting (canary / blue-green)
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00005-abc=90,my-service-00006-def=10

# List services
gcloud run services list --region=us-central1

# Describe a service
gcloud run services describe my-service --region=us-central1

# View logs
gcloud run services logs read my-service --region=us-central1
```

**Exam tips:**
- Cloud Run scales to **zero** by default (`--min-instances=0`). Set `--min-instances=1` to avoid cold starts.
- `--allow-unauthenticated` makes the service public. Without it, callers need `roles/run.invoker`.
- Cloud Run supports **revision-based traffic splitting** for canary deployments.
- Max request timeout is 60 minutes. Max instance memory is 32 GiB.

Docs: [Cloud Run overview](https://cloud.google.com/run/docs/overview/what-is-cloud-run) | [Deploying to Cloud Run](https://cloud.google.com/run/docs/deploying)

---

### 3.3.2 Deploying for Receiving Google Cloud Events

Cloud Run and Cloud Functions can be triggered by events using **Eventarc**, **Pub/Sub push subscriptions**, or **direct triggers**.

#### Eventarc (the unified eventing layer)

```bash
# Deploy a Cloud Run service that receives Cloud Storage events via Eventarc
gcloud eventarc triggers create my-storage-trigger \
  --location=us-central1 \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=my-bucket" \
  --service-account=my-sa@my-project.iam.gserviceaccount.com

# Create a trigger for Pub/Sub messages
gcloud eventarc triggers create my-pubsub-trigger \
  --location=us-central1 \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.pubsub.topic.v1.messagePublished" \
  --transport-topic=my-topic \
  --service-account=my-sa@my-project.iam.gserviceaccount.com

# Create a trigger for Audit Log events (e.g., BigQuery job complete)
gcloud eventarc triggers create my-audit-trigger \
  --location=us-central1 \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=bigquery.googleapis.com" \
  --event-filters="methodName=google.cloud.bigquery.v2.JobService.InsertJob" \
  --service-account=my-sa@my-project.iam.gserviceaccount.com

# List triggers
gcloud eventarc triggers list --location=us-central1
```

#### Cloud Functions (2nd gen -- built on Cloud Run)

```bash
# Deploy a Cloud Function triggered by Cloud Storage events
gcloud functions deploy my-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-bucket" \
  --source=. \
  --entry-point=process_file

# Deploy a Cloud Function triggered by Pub/Sub
gcloud functions deploy my-function \
  --gen2 \
  --region=us-central1 \
  --runtime=nodejs20 \
  --trigger-topic=my-topic \
  --source=. \
  --entry-point=handleMessage

# Deploy an HTTP-triggered Cloud Function
gcloud functions deploy my-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --trigger-http \
  --allow-unauthenticated \
  --source=. \
  --entry-point=hello_world

# List functions
gcloud functions list

# View logs
gcloud functions logs read my-function --region=us-central1
```

**Exam tips:**
- **Cloud Functions 2nd gen** is built on Cloud Run under the hood.
- **Eventarc** is the recommended way to route events to Cloud Run and Cloud Functions 2nd gen.
- Eventarc supports 3 event sources: **direct events** (Cloud Storage, etc.), **Pub/Sub**, and **Cloud Audit Logs**.
- Always use `--gen2` for new Cloud Functions deployments (1st gen is legacy).

Docs: [Eventarc overview](https://cloud.google.com/eventarc/docs/overview) | [Cloud Functions overview](https://cloud.google.com/functions/docs/concepts/overview)

---

### 3.3.3 When to Use What: Cloud Run vs Cloud Functions

| Criteria | Cloud Run | Cloud Functions |
|----------|-----------|-----------------|
| Unit of deployment | **Container image** | **Function (source code)** |
| Language flexibility | Any language/binary | Supported runtimes only |
| Request timeout | Up to 60 min | Up to 60 min (2nd gen) |
| Concurrency | Multiple concurrent requests per instance | 1 request/instance (1st gen), configurable (2nd gen) |
| Use case | Microservices, APIs, web apps, any containerized workload | Event-driven glue code, lightweight processing |
| Scales to zero | Yes | Yes |
| VPC connectivity | Via VPC connector or Direct VPC egress | Via VPC connector |

**Decision guide:**
- **Cloud Run** -- You have a containerized app, need multiple concurrent requests, or need a language/framework not in Cloud Functions runtimes.
- **Cloud Functions** -- Simple event-driven functions, quick prototyping, single-purpose triggers. Use 2nd gen for all new work.
- **Cloud Run for Anthos (GKE)** -- You need Cloud Run's developer experience but on your own GKE cluster (on-prem or specific compliance needs). Note: This is being superseded by GKE + Knative.

Docs: [Choosing a compute option](https://cloud.google.com/hosting-options)

---

## 3.4 Deploying and Implementing Data Solutions

### 3.4.1 Cloud SQL

Managed relational database service (MySQL, PostgreSQL, SQL Server).

```bash
# Create a Cloud SQL instance (PostgreSQL)
gcloud sql instances create my-db \
  --database-version=POSTGRES_15 \
  --tier=db-custom-2-8192 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --storage-type=SSD \
  --storage-size=50GB \
  --storage-auto-increase

# Create a database
gcloud sql databases create mydb --instance=my-db

# Create a user
gcloud sql users create myuser \
  --instance=my-db \
  --password=mysecretpassword

# Connect using Cloud SQL Auth Proxy
gcloud sql connect my-db --user=myuser --database=mydb

# Enable backups
gcloud sql instances patch my-db \
  --backup-start-time=02:00 \
  --enable-bin-log

# Create a read replica
gcloud sql instances create my-db-replica \
  --master-instance-name=my-db \
  --region=us-central1

# Import data from Cloud Storage
gcloud sql import sql my-db gs://my-bucket/dump.sql --database=mydb
gcloud sql import csv my-db gs://my-bucket/data.csv --database=mydb --table=mytable

# Export data
gcloud sql export sql my-db gs://my-bucket/backup.sql --database=mydb
```

**Exam tips:**
- `--availability-type=REGIONAL` = HA with automatic failover (synchronous replication to a standby). `ZONAL` = no HA.
- Cloud SQL Auth Proxy is the **recommended** connection method (handles SSL + IAM auth).
- Max storage: 64 TB. Supports automatic storage increases.

Docs: [Cloud SQL overview](https://cloud.google.com/sql/docs/introduction)

---

### 3.4.2 AlloyDB

Fully managed, PostgreSQL-compatible database for demanding enterprise workloads. Combines Google infrastructure with PostgreSQL compatibility.

```bash
# Create an AlloyDB cluster
gcloud alloydb clusters create my-cluster \
  --region=us-central1 \
  --password=mysecretpassword \
  --network=default

# Create a primary instance
gcloud alloydb instances create my-primary \
  --cluster=my-cluster \
  --region=us-central1 \
  --instance-type=PRIMARY \
  --cpu-count=4

# Create a read pool instance
gcloud alloydb instances create my-read-pool \
  --cluster=my-cluster \
  --region=us-central1 \
  --instance-type=READ_POOL \
  --cpu-count=2 \
  --read-pool-node-count=2

# List clusters
gcloud alloydb clusters list --region=us-central1
```

**Exam tips:**
- AlloyDB is **PostgreSQL-compatible** but NOT open-source PostgreSQL -- it uses Google's storage layer.
- Up to **4x faster** than standard PostgreSQL for transactional workloads, **100x faster** for analytical queries.
- AlloyDB requires a **VPC** -- it is a private service (no public IP option by default).
- Use AlloyDB when you need PostgreSQL compatibility with better performance than Cloud SQL.

Docs: [AlloyDB overview](https://cloud.google.com/alloydb/docs/overview)

---

### 3.4.3 Firestore

Serverless, NoSQL document database. Two modes: **Native mode** (real-time sync, offline support) and **Datastore mode** (backward-compatible with legacy Datastore).

```bash
# Create a Firestore database (Native mode)
gcloud firestore databases create \
  --location=us-central1 \
  --type=firestore-native

# Create a Firestore database (Datastore mode)
gcloud firestore databases create \
  --location=us-central1 \
  --type=datastore-mode

# Export data
gcloud firestore export gs://my-bucket/firestore-backup

# Import data
gcloud firestore import gs://my-bucket/firestore-backup

# Create an index
gcloud firestore indexes composite create \
  --collection-group=users \
  --field-config=field-path=name,order=ascending \
  --field-config=field-path=age,order=descending
```

**Exam tips:**
- **Native mode** = mobile/web apps with real-time listeners and offline support.
- **Datastore mode** = server-side apps migrating from legacy Datastore. No real-time sync.
- You **cannot change modes** after creation (must create a new database).
- Firestore is **strongly consistent** in both modes.

Docs: [Firestore overview](https://cloud.google.com/firestore/docs/overview)

---

### 3.4.4 BigQuery

Serverless, petabyte-scale data warehouse for analytics.

```bash
# Create a dataset
bq mk --dataset --location=US my-project:my_dataset

# Create a table with schema
bq mk --table my_dataset.my_table name:STRING,age:INTEGER,email:STRING

# Load data from local file
bq load --source_format=CSV my_dataset.my_table ./data.csv name:STRING,age:INTEGER

# Load data from Cloud Storage
bq load --source_format=NEWLINE_DELIMITED_JSON \
  my_dataset.my_table gs://my-bucket/data.json

# Load Parquet data from Cloud Storage (schema auto-detected)
bq load --source_format=PARQUET \
  --autodetect \
  my_dataset.my_table gs://my-bucket/data.parquet

# Run a query
bq query --use_legacy_sql=false \
  'SELECT name, COUNT(*) as cnt FROM my_dataset.my_table GROUP BY name'

# Export table to Cloud Storage
bq extract my_dataset.my_table gs://my-bucket/export/data-*.csv

# List datasets
bq ls

# Show table info
bq show my_dataset.my_table
```

**Exam tips:**
- BigQuery uses **slots** for query processing. On-demand pricing = per TB scanned. Flat-rate = reserved slots.
- **Partitioning** (by time or integer range) and **clustering** (by column) reduce query cost and improve performance.
- BigQuery is **columnar** and **serverless** -- no infrastructure to manage.
- Data loaded into BigQuery is **free**. You pay for queries and storage.

Docs: [BigQuery overview](https://cloud.google.com/bigquery/docs/introduction)

---

### 3.4.5 Cloud Spanner

Globally distributed, strongly consistent, relational database. Combines the benefits of relational structure with horizontal scalability.

```bash
# Create a Spanner instance
gcloud spanner instances create my-instance \
  --config=regional-us-central1 \
  --description="My Spanner Instance" \
  --nodes=1

# Create a database
gcloud spanner databases create my-db \
  --instance=my-instance \
  --ddl='CREATE TABLE Users (UserId INT64, Name STRING(100)) PRIMARY KEY(UserId)'

# Run a query
gcloud spanner databases execute-sql my-db \
  --instance=my-instance \
  --sql='SELECT * FROM Users'

# Update schema
gcloud spanner databases ddl update my-db \
  --instance=my-instance \
  --ddl='ALTER TABLE Users ADD COLUMN Email STRING(200)'

# List instances
gcloud spanner instances list
```

**Exam tips:**
- Spanner provides **external consistency** (strongest form of consistency) with global distribution.
- Minimum cost is **1 node** (~$0.90/hr) -- it is expensive for small workloads.
- Use Spanner when you need a **globally distributed relational DB** with strong consistency and horizontal scaling.
- Multi-region configs (e.g., `nam-eur-asia1`) for global HA. Regional configs for lower latency/cost.

Docs: [Cloud Spanner overview](https://cloud.google.com/spanner/docs/overview)

---

### 3.4.6 Pub/Sub

Serverless, real-time messaging service for event-driven systems and streaming analytics.

```bash
# Create a topic
gcloud pubsub topics create my-topic

# Create a subscription (pull)
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --ack-deadline=60

# Create a push subscription
gcloud pubsub subscriptions create my-push-sub \
  --topic=my-topic \
  --push-endpoint=https://my-service-xyz.run.app/push

# Publish a message
gcloud pubsub topics publish my-topic --message="Hello World"

# Pull messages
gcloud pubsub subscriptions pull my-sub --limit=10 --auto-ack

# Create a subscription with a dead-letter topic
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --dead-letter-topic=my-dead-letter-topic \
  --max-delivery-attempts=5

# Delete a subscription
gcloud pubsub subscriptions delete my-sub

# List topics / subscriptions
gcloud pubsub topics list
gcloud pubsub subscriptions list
```

**Exam tips:**
- **Pull** = subscriber requests messages. **Push** = Pub/Sub sends messages to an endpoint (HTTP).
- Messages are retained for **7 days** by default (configurable up to 31 days).
- **At-least-once delivery** -- your subscriber must handle duplicates.
- Pub/Sub supports **ordering keys** for ordered delivery within a key.
- Subscriber must **acknowledge** messages or they will be redelivered.

Docs: [Pub/Sub overview](https://cloud.google.com/pubsub/docs/overview)

---

### 3.4.7 Dataflow

Fully managed service for stream and batch data processing, based on Apache Beam.

```bash
# Run a batch Dataflow job from a template (e.g., GCS text to BigQuery)
gcloud dataflow jobs run my-batch-job \
  --gcs-location=gs://dataflow-templates/latest/GCS_Text_to_BigQuery \
  --region=us-central1 \
  --parameters=\
inputFilePattern=gs://my-bucket/input/*.csv,\
JSONPath=gs://my-bucket/schema.json,\
outputTable=my-project:my_dataset.my_table,\
bigQueryLoadingTemporaryDirectory=gs://my-bucket/temp/

# Run a streaming Dataflow job from a template (Pub/Sub to BigQuery)
gcloud dataflow jobs run my-streaming-job \
  --gcs-location=gs://dataflow-templates/latest/PubSub_to_BigQuery \
  --region=us-central1 \
  --parameters=\
inputTopic=projects/my-project/topics/my-topic,\
outputTableSpec=my-project:my_dataset.my_table

# List Dataflow jobs
gcloud dataflow jobs list --region=us-central1

# Cancel a job (drain finishes in-flight; cancel stops immediately)
gcloud dataflow jobs drain JOB_ID --region=us-central1
gcloud dataflow jobs cancel JOB_ID --region=us-central1
```

**Exam tips:**
- Dataflow = managed **Apache Beam**. Write once, run batch or streaming.
- **Drain** gracefully stops a streaming job (processes in-flight data). **Cancel** stops immediately.
- Google provides many **pre-built Dataflow templates** for common patterns.
- Use Dataflow for ETL, real-time analytics, and event processing at scale.

Docs: [Dataflow overview](https://cloud.google.com/dataflow/docs/overview)

---

### 3.4.8 Cloud Storage

Object storage with multiple storage classes for different access patterns.

```bash
# Create a bucket
gcloud storage buckets create gs://my-bucket \
  --location=us-central1 \
  --default-storage-class=STANDARD \
  --uniform-bucket-level-access

# Upload a file
gcloud storage cp local-file.txt gs://my-bucket/

# Upload a directory recursively
gcloud storage cp -r ./my-dir gs://my-bucket/

# Download a file
gcloud storage cp gs://my-bucket/file.txt ./local-file.txt

# Sync a local directory to a bucket
gcloud storage rsync -r ./local-dir gs://my-bucket/remote-dir

# Move/rename an object
gcloud storage mv gs://my-bucket/old-name.txt gs://my-bucket/new-name.txt

# List objects
gcloud storage ls gs://my-bucket/

# Set lifecycle rules (e.g., delete objects older than 365 days)
gcloud storage buckets update gs://my-bucket \
  --lifecycle-file=lifecycle.json

# Change storage class of a bucket
gcloud storage buckets update gs://my-bucket \
  --default-storage-class=NEARLINE

# Enable versioning
gcloud storage buckets update gs://my-bucket --versioning

# Make a bucket public (all objects)
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member=allUsers \
  --role=roles/storage.objectViewer
```

**Storage classes:**

| Class | Min duration | Use case |
|-------|-------------|----------|
| Standard | None | Frequently accessed data |
| Nearline | 30 days | Once a month access |
| Coldline | 90 days | Once a quarter access |
| Archive | 365 days | Once a year access, backups |

**Loading data methods:**
- **CLI upload**: `gcloud storage cp` or `gsutil cp` for direct uploads.
- **From Cloud Storage**: Load into BigQuery, Cloud SQL, Spanner, etc. via their import commands.
- **Storage Transfer Service**: Schedule and automate large-scale transfers from AWS S3, Azure, HTTP/HTTPS sources, or between GCS buckets.
- **Transfer Appliance**: Physical device for petabyte-scale offline transfers.

```bash
# Storage Transfer Service -- create a transfer job (S3 to GCS)
gcloud transfer jobs create \
  s3://source-bucket gs://destination-bucket \
  --source-creds-file=s3-creds.json

# Create a scheduled transfer (daily at midnight)
gcloud transfer jobs create \
  gs://source-bucket gs://dest-bucket \
  --schedule-starts=2024-01-01 \
  --schedule-repeats-every=1d
```

**Exam tips:**
- `gcloud storage` is the modern CLI (replaces `gsutil` for most operations).
- **Uniform bucket-level access** is recommended over ACLs (simplifies IAM).
- **Object Lifecycle Management** automates class transitions and deletions.
- Minimum storage durations have early deletion charges (Nearline 30d, Coldline 90d, Archive 365d).
- **Storage Transfer Service** is for scheduled/recurring transfers from external or cross-bucket sources.

Docs: [Cloud Storage overview](https://cloud.google.com/storage/docs/introduction) | [Storage Transfer Service](https://cloud.google.com/storage-transfer/docs/overview)

---

## 3.5 Deploying and Implementing Networking Resources

### 3.5.1 Creating a VPC with Subnets

A VPC is a global resource. Subnets are regional. GCP offers **auto mode** (auto-creates one subnet per region) and **custom mode** (you define all subnets).

```bash
# Create a custom-mode VPC
gcloud compute networks create my-vpc \
  --subnet-mode=custom

# Create subnets in the VPC
gcloud compute networks subnets create subnet-us \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

gcloud compute networks subnets create subnet-eu \
  --network=my-vpc \
  --region=europe-west1 \
  --range=10.0.2.0/24

# Create a subnet with secondary ranges (for GKE Pod/Service IPs)
gcloud compute networks subnets create subnet-gke \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.3.0/24 \
  --secondary-range=pods=10.4.0.0/14,services=10.8.0.0/20

# Enable Private Google Access on a subnet
gcloud compute networks subnets update subnet-us \
  --region=us-central1 \
  --enable-private-ip-google-access

# Enable Flow Logs
gcloud compute networks subnets update subnet-us \
  --region=us-central1 \
  --enable-flow-logs

# Convert auto-mode VPC to custom-mode (one-way, irreversible)
gcloud compute networks update my-auto-vpc \
  --switch-to-custom-subnet-mode

# Delete a VPC
gcloud compute networks delete my-vpc

# List all VPCs
gcloud compute networks list

# List subnets in a VPC
gcloud compute networks subnets list --network=my-vpc
```

#### Shared VPC

Shared VPC allows a **host project** to share its VPC network with **service projects**. Centralizes network administration.

```bash
# Enable Shared VPC on the host project (requires Org-level permissions)
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Associate a service project with the host project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# List associated service projects
gcloud compute shared-vpc list-associated-resources HOST_PROJECT_ID

# Disable Shared VPC
gcloud compute shared-vpc disable HOST_PROJECT_ID
```

**Exam tips:**
- **Custom mode VPC** is recommended for production (full control over IP ranges).
- **Auto mode to custom mode** conversion is one-way -- you cannot convert back.
- **Shared VPC**: the host project owns the network, service projects use it. Network admins manage the host project; developers deploy resources in service projects.
- **Private Google Access** lets VMs without external IPs reach Google APIs and services.
- VPCs are **global**. Subnets are **regional**. Firewall rules are **global** (applied to VPC).
- Expanding a subnet's IP range is allowed. Shrinking is not.

Docs: [VPC overview](https://cloud.google.com/vpc/docs/vpc) | [Shared VPC overview](https://cloud.google.com/vpc/docs/shared-vpc)

---

### 3.5.2 Creating Firewall Rules

VPC firewall rules control ingress and egress traffic to/from VM instances. They are stateful (return traffic is automatically allowed).

```bash
# Create an ingress firewall rule (allow HTTP from anywhere)
gcloud compute firewall-rules create allow-http \
  --network=my-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server \
  --priority=1000

# Allow HTTPS from specific IP range
gcloud compute firewall-rules create allow-https-office \
  --network=my-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:443 \
  --source-ranges=203.0.113.0/24 \
  --target-tags=https-server \
  --priority=900

# Allow internal communication between VMs
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/8

# Target by service account (more secure than network tags)
gcloud compute firewall-rules create allow-db \
  --network=my-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:5432 \
  --source-service-accounts=web-sa@my-project.iam.gserviceaccount.com \
  --target-service-accounts=db-sa@my-project.iam.gserviceaccount.com \
  --priority=800

# Create an egress deny rule (block outbound to a specific range)
gcloud compute firewall-rules create deny-egress-external \
  --network=my-vpc \
  --direction=EGRESS \
  --action=DENY \
  --rules=all \
  --destination-ranges=0.0.0.0/0 \
  --priority=1000

# List all firewall rules for a VPC
gcloud compute firewall-rules list --filter="network:my-vpc"

# Describe a rule
gcloud compute firewall-rules describe allow-http

# Delete a rule
gcloud compute firewall-rules delete allow-http

# Update a rule
gcloud compute firewall-rules update allow-http \
  --source-ranges=10.0.0.0/8
```

**Exam tips:**
- Firewall rules: **lower priority number = higher priority** (0 is highest, 65535 is lowest).
- Default rules: **deny all ingress** (priority 65535), **allow all egress** (priority 65535).
- **Network tags** target VMs by tag. **Service accounts** target VMs by identity (more secure, survives VM recreation).
- Firewall rules are **stateful** -- if ingress is allowed, return traffic is automatically allowed.
- Implied rules cannot be deleted: deny-all-ingress and allow-all-egress at priority 65535.
- Use `--source-ranges` for ingress, `--destination-ranges` for egress.
- Ingress rules: use `--target-tags` or `--target-service-accounts` to specify which VMs the rule applies to.

Docs: [VPC firewall rules overview](https://cloud.google.com/firewall/docs/firewalls)

---

### 3.5.3 Peering External Networks

#### Cloud VPN

Securely connects your on-prem network to GCP VPC over an encrypted IPsec tunnel.

Two types:
- **Classic VPN** -- single tunnel, 99.9% SLA, supports static and dynamic routing.
- **HA VPN** -- two tunnels for 99.99% SLA, requires dynamic (BGP) routing.

```bash
# --- HA VPN setup (recommended) ---

# Step 1: Create an HA VPN gateway
gcloud compute vpn-gateways create my-ha-vpn \
  --network=my-vpc \
  --region=us-central1

# Step 2: Create a Cloud Router (required for BGP)
gcloud compute routers create my-router \
  --network=my-vpc \
  --region=us-central1 \
  --asn=65001

# Step 3: Create a peer VPN gateway (represents on-prem side)
gcloud compute external-vpn-gateways create my-peer-gateway \
  --interfaces=0=ON_PREM_PUBLIC_IP

# Step 4: Create VPN tunnels (two for HA)
gcloud compute vpn-tunnels create tunnel-0 \
  --vpn-gateway=my-ha-vpn \
  --peer-external-gateway=my-peer-gateway \
  --peer-external-gateway-interface=0 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=MY_SHARED_SECRET \
  --router=my-router \
  --vpn-gateway-interface=0

gcloud compute vpn-tunnels create tunnel-1 \
  --vpn-gateway=my-ha-vpn \
  --peer-external-gateway=my-peer-gateway \
  --peer-external-gateway-interface=0 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=MY_SHARED_SECRET \
  --router=my-router \
  --vpn-gateway-interface=1

# Step 5: Create BGP sessions on the Cloud Router
gcloud compute routers add-interface my-router \
  --interface-name=bgp-if-0 \
  --vpn-tunnel=tunnel-0 \
  --ip-address=169.254.0.1 \
  --mask-length=30 \
  --region=us-central1

gcloud compute routers add-bgp-peer my-router \
  --peer-name=bgp-peer-0 \
  --interface=bgp-if-0 \
  --peer-ip-address=169.254.0.2 \
  --peer-asn=65002 \
  --region=us-central1

# List VPN tunnels
gcloud compute vpn-tunnels list
```

#### VPC Network Peering

Connects two VPC networks (within same or different projects/orgs) using Google's internal network. No VPN or gateway needed.

```bash
# Peer VPC-A with VPC-B (must be done from both sides)

# From project-a: create peering to project-b's VPC
gcloud compute networks peerings create peer-ab \
  --network=vpc-a \
  --peer-project=project-b \
  --peer-network=vpc-b

# From project-b: create peering to project-a's VPC
gcloud compute networks peerings create peer-ba \
  --network=vpc-b \
  --peer-project=project-a \
  --peer-network=vpc-a

# List peerings
gcloud compute networks peerings list

# Delete a peering
gcloud compute networks peerings delete peer-ab --network=vpc-a
```

**Exam tips:**
- **HA VPN** requires **two tunnels** and **BGP** (dynamic routing). It gives 99.99% SLA.
- **Classic VPN** supports static routing and gives 99.9% SLA. Being deprecated in favor of HA VPN.
- **VPC Peering** is NOT transitive. If A peers with B and B peers with C, A cannot reach C through B.
- VPC Peering does not support overlapping IP ranges between peered VPCs.
- **Cloud Interconnect** (Dedicated or Partner) is for high-bandwidth, low-latency connections. Not the same as VPN.
- Cloud Router uses **BGP** to dynamically exchange routes between your on-prem network and GCP.

Docs: [Cloud VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview) | [VPC Network Peering](https://cloud.google.com/vpc/docs/vpc-peering)

---

## 3.6 Implementing Resources Through Infrastructure as Code

### 3.6.1 Terraform

HashiCorp Terraform is the primary IaC tool tested on the exam. It uses declarative HCL (HashiCorp Configuration Language) to define and provision infrastructure.

**Key commands:**

```bash
# Initialize a Terraform working directory (downloads providers)
terraform init

# Preview what changes will be made
terraform plan

# Apply changes (create/update/delete resources)
terraform apply

# Apply with auto-approve (skip confirmation -- use with caution)
terraform apply -auto-approve

# Destroy all managed resources
terraform destroy

# Format Terraform files
terraform fmt

# Validate configuration syntax
terraform validate

# Show current state
terraform show

# Import existing resources into Terraform state
terraform import google_compute_instance.my-vm projects/my-project/zones/us-central1-a/instances/my-vm

# List resources in state
terraform state list

# Remove a resource from state (does not delete the actual resource)
terraform state rm google_compute_instance.my-vm
```

**Example Terraform config (GCE instance):**

```hcl
# main.tf
provider "google" {
  project = "my-project"
  region  = "us-central1"
}

resource "google_compute_instance" "my_vm" {
  name         = "my-vm"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    network = "default"
    access_config {} # Gives external IP
  }

  tags = ["http-server"]
}
```

**Exam tips:**
- **`terraform init`** -- always run first. Downloads provider plugins.
- **`terraform plan`** -- dry run, shows what will change. Does NOT make changes.
- **`terraform apply`** -- actually creates/updates/destroys resources to match config.
- **`terraform destroy`** -- removes all resources managed by the config.
- State file (`terraform.tfstate`) tracks the mapping between config and real resources. **Store it remotely** (e.g., in a GCS bucket) for team use.
- Terraform is **declarative**: you describe the desired end state, and Terraform figures out what to create/update/delete.

Docs: [Terraform on Google Cloud](https://cloud.google.com/docs/terraform)

---

### 3.6.2 Cloud Foundation Toolkit (CFT)

A collection of **opinionated, production-ready Terraform modules** maintained by Google. Implements Google Cloud best practices for common infrastructure patterns.

```bash
# Example: Use the CFT network module
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 9.0"

  project_id   = "my-project"
  network_name = "my-vpc"
  routing_mode = "GLOBAL"

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "us-central1"
    },
  ]
}
```

**Exam tips:**
- CFT = pre-built, Google-maintained Terraform modules.
- Use CFT for landing zones, project factories, network setups, and GKE clusters.
- Found on the [Terraform Registry](https://registry.terraform.io/namespaces/terraform-google-modules) under `terraform-google-modules`.

Docs: [Cloud Foundation Toolkit](https://cloud.google.com/foundation-toolkit)

---

### 3.6.3 Config Connector

Config Connector is a Kubernetes add-on that lets you manage Google Cloud resources through Kubernetes resource definitions (YAML/kubectl). It bridges Kubernetes-native workflows with GCP resource management.

```yaml
# Example: Create a Cloud Storage bucket via Config Connector
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-config-connector-bucket
  namespace: my-namespace
spec:
  location: US
  storageClass: STANDARD
  uniformBucketLevelAccess: true
```

```bash
# Apply the resource via kubectl
kubectl apply -f storage-bucket.yaml

# Check the resource status
kubectl describe storagebucket my-config-connector-bucket

# Delete the GCP resource
kubectl delete storagebucket my-config-connector-bucket
```

**Exam tips:**
- Config Connector = manage GCP resources **using kubectl and Kubernetes manifests**.
- It runs as a controller inside a GKE cluster.
- Best for teams that want a **Kubernetes-native** GitOps workflow for all infrastructure.
- Resources are reconciled continuously -- if someone manually changes a resource, Config Connector will drift-correct it.

Docs: [Config Connector overview](https://cloud.google.com/config-connector/docs/overview)

---

### 3.6.4 Helm

Helm is the **package manager for Kubernetes**. It uses "charts" (pre-packaged Kubernetes manifests) to deploy complex applications.

```bash
# Install Helm (if not already installed)
# https://helm.sh/docs/intro/install/

# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install a chart (creates a "release")
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=LoadBalancer

# Install from a values file
helm install my-release bitnami/nginx -f my-values.yaml

# List installed releases
helm list

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=5

# Rollback a release
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release

# Show chart details
helm show chart bitnami/nginx
helm show values bitnami/nginx
```

**Exam tips:**
- Helm = Kubernetes package manager. Charts = reusable packages of K8s manifests.
- A **release** is an installed instance of a chart.
- Helm supports **rollback** to previous release versions.
- Commonly used to deploy third-party apps (NGINX, Redis, Prometheus) into GKE clusters.

Docs: [Using Helm with GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/helm)

---

## Quick Reference: Service Selection Cheat Sheet

| Need | Service |
|------|---------|
| Run containers serverlessly | **Cloud Run** |
| Run event-driven functions | **Cloud Functions (2nd gen)** |
| Full Kubernetes control | **GKE Standard** |
| Managed Kubernetes, less ops | **GKE Autopilot** |
| Traditional VMs | **Compute Engine** |
| Managed MySQL/PostgreSQL/SQL Server | **Cloud SQL** |
| High-perf PostgreSQL-compatible | **AlloyDB** |
| Global relational DB, strong consistency | **Cloud Spanner** |
| NoSQL document DB | **Firestore** |
| Data warehouse / analytics | **BigQuery** |
| Async messaging | **Pub/Sub** |
| Stream/batch ETL processing | **Dataflow** |
| Object storage | **Cloud Storage** |
| IaC with declarative configs | **Terraform** |
| K8s-native GCP resource management | **Config Connector** |
| K8s package management | **Helm** |

---

## Final Exam Tips for Domain 3

1. **Autopilot vs Standard**: Autopilot manages nodes for you. Use Standard when you need GPU node pools, DaemonSets, privileged containers, or custom node configurations.

2. **Shared VPC**: Host project owns the network, service projects use it. Requires Organization-level setup. Network admins work in the host project.

3. **Firewall rules**: Lower priority number = higher priority. Default implied rules (deny all ingress, allow all egress) are at priority 65535 and cannot be deleted.

4. **VPC Peering is NOT transitive**: A-B peering + B-C peering does NOT mean A can reach C.

5. **Cloud SQL HA**: `--availability-type=REGIONAL` creates a standby in another zone for automatic failover.

6. **Terraform workflow**: `init` -> `plan` -> `apply`. Always `plan` before `apply`.

7. **Cloud Run vs Cloud Functions**: Cloud Run runs containers (any language). Cloud Functions runs source code (supported runtimes). Cloud Functions 2nd gen is built on Cloud Run.

8. **Eventarc**: The unified eventing system. Routes events from 130+ Google Cloud sources to Cloud Run, Cloud Functions, and GKE.

9. **MIG rolling updates**: Instance templates are immutable. Create a new template, then rolling-action start-update.

10. **OS Login > metadata SSH keys**: OS Login integrates with IAM. If the exam asks about centralized SSH key management, the answer is OS Login.

11. **Private Google Access**: Allows VMs without external IPs to reach Google APIs. Must be enabled per subnet.

12. **`gcloud storage` replaces `gsutil`**: The exam may reference either, but `gcloud storage` is the modern CLI.

13. **BigQuery loading is free**: You pay for queries (per TB scanned) and storage, not for data ingestion.

14. **HA VPN = 2 tunnels + BGP + 99.99% SLA. Classic VPN = 1 tunnel + static or BGP + 99.9% SLA.**

15. **Dataflow drain vs cancel**: `drain` finishes processing in-flight data then stops. `cancel` stops immediately.
