# Domain 4: Ensuring Successful Operation of a Cloud Solution (~20%)

Quick-reference study guide for the GCP ACE exam. Assumes daily GCP experience -- focuses on what the exam tests, key commands, and gotchas.

---

## 4.1 Managing Compute Engine Resources

### Remotely Connecting to Instances

**SSH (Linux)**

```bash
# Standard SSH via gcloud (handles key management + OS Login automatically)
gcloud compute ssh INSTANCE_NAME --zone=ZONE

# SSH with a specific user
gcloud compute ssh USER@INSTANCE_NAME --zone=ZONE

# SSH through IAP tunnel (no external IP needed)
gcloud compute ssh INSTANCE_NAME --zone=ZONE --tunnel-through-iap

# SSH with a custom private key
gcloud compute ssh INSTANCE_NAME --zone=ZONE --ssh-key-file=~/.ssh/my_key

# SCP a file to an instance
gcloud compute scp LOCAL_FILE INSTANCE_NAME:~/REMOTE_PATH --zone=ZONE
```

**RDP (Windows)**

```bash
# Reset Windows password (generates a new one)
gcloud compute reset-windows-password INSTANCE_NAME --zone=ZONE

# Then connect via any RDP client using the returned IP + credentials
# Or use IAP for TCP forwarding on port 3389:
gcloud compute start-iap-tunnel INSTANCE_NAME 3389 --local-host-port=localhost:3389 --zone=ZONE
```

**Serial Console**

```bash
# Connect to serial console (useful when SSH is broken, boot issues, etc.)
gcloud compute connect-to-serial-port INSTANCE_NAME --zone=ZONE

# Must enable serial port access on the VM or project:
gcloud compute instances add-metadata INSTANCE_NAME \
  --metadata=serial-port-enable=TRUE --zone=ZONE
```

> **Exam tip:** IAP (Identity-Aware Proxy) TCP tunneling lets you SSH/RDP to VMs with no external IP and no VPN. It uses IAM permissions (`roles/iap.tunnelResourceAccessor`), not firewall rules on port 22/3389 from the internet. Serial console is the last resort when the OS is unresponsive.

Docs: [Connecting to instances](https://cloud.google.com/compute/docs/instances/connecting-to-instance) | [Serial console](https://cloud.google.com/compute/docs/troubleshooting/troubleshooting-using-serial-console) | [IAP TCP forwarding](https://cloud.google.com/iap/docs/using-tcp-forwarding)

---

### Viewing Current Running VM Inventory

```bash
# List all instances in the project
gcloud compute instances list

# List instances in a specific zone
gcloud compute instances list --filter="zone:us-central1-a"

# Filter running instances only
gcloud compute instances list --filter="status=RUNNING"

# Describe a specific instance (full details: machine type, disks, network, etc.)
gcloud compute instances describe INSTANCE_NAME --zone=ZONE

# Get instance ID specifically
gcloud compute instances describe INSTANCE_NAME --zone=ZONE --format="value(id)"

# List with custom columns
gcloud compute instances list --format="table(name,zone,machineType,status,networkInterfaces[0].accessConfigs[0].natIP)"

# List instance templates
gcloud compute instance-templates list

# List managed instance groups
gcloud compute instance-groups managed list
```

> **Exam tip:** `--filter` uses field-level expressions and is processed server-side (efficient). `--format` controls output display. Know the difference between `list`, `describe`, and `get-iam-policy`.

Docs: [Viewing VM instances](https://cloud.google.com/compute/docs/instances/view-vm-details) | [gcloud filters](https://cloud.google.com/sdk/gcloud/reference/topic/filters)

---

### Working with Snapshots

Snapshots back up persistent disks. They are **global** resources (not zonal/regional).

```bash
# Create a snapshot from a disk
gcloud compute snapshots create SNAPSHOT_NAME \
  --source-disk=DISK_NAME \
  --source-disk-zone=ZONE

# Create a snapshot with a storage location
gcloud compute snapshots create SNAPSHOT_NAME \
  --source-disk=DISK_NAME \
  --source-disk-zone=ZONE \
  --storage-location=us

# List all snapshots
gcloud compute snapshots list

# Describe a snapshot
gcloud compute snapshots describe SNAPSHOT_NAME

# Delete a snapshot
gcloud compute snapshots delete SNAPSHOT_NAME

# Create a disk from a snapshot
gcloud compute disks create NEW_DISK --source-snapshot=SNAPSHOT_NAME --zone=ZONE
```

**Snapshot Schedules (automated backups)**

```bash
# Create a snapshot schedule
gcloud compute resource-policies create snapshot-schedule SCHEDULE_NAME \
  --region=REGION \
  --max-retention-days=14 \
  --on-source-disk-delete=keep-auto-snapshots \
  --daily-schedule \
  --start-time=02:00

# Attach schedule to a disk
gcloud compute disks add-resource-policies DISK_NAME \
  --resource-policies=SCHEDULE_NAME \
  --zone=ZONE

# Remove schedule from a disk
gcloud compute disks remove-resource-policies DISK_NAME \
  --resource-policies=SCHEDULE_NAME \
  --zone=ZONE

# List snapshot schedules
gcloud compute resource-policies list --filter="region:REGION"
```

> **Exam tip:** Snapshots are **incremental** -- only the first snapshot is a full copy. Subsequent snapshots store only changed blocks. However, deleting an earlier snapshot is safe; GCP automatically moves needed data to the next snapshot. Snapshots can be multi-regional or regional for storage location. Snapshot schedules are resource policies attached to disks.

Docs: [Persistent disk snapshots](https://cloud.google.com/compute/docs/disks/create-snapshots) | [Snapshot schedules](https://cloud.google.com/compute/docs/disks/scheduled-snapshots)

---

### Working with Images

Custom images are used to create boot disks. They can be created from disks, snapshots, other images, or files in Cloud Storage.

```bash
# Create an image from a VM's boot disk (stop the VM first for consistency)
gcloud compute images create IMAGE_NAME \
  --source-disk=DISK_NAME \
  --source-disk-zone=ZONE

# Create an image from a snapshot
gcloud compute images create IMAGE_NAME \
  --source-snapshot=SNAPSHOT_NAME

# Create an image from another image (e.g., from a different project)
gcloud compute images create IMAGE_NAME \
  --source-image=SOURCE_IMAGE \
  --source-image-project=SOURCE_PROJECT

# List custom images
gcloud compute images list --no-standard-images

# List all images (including public ones)
gcloud compute images list

# Describe an image
gcloud compute images describe IMAGE_NAME

# Delete an image
gcloud compute images delete IMAGE_NAME

# Deprecate an image (mark as deprecated but still usable)
gcloud compute images deprecate IMAGE_NAME --state=DEPRECATED

# Create a VM from a custom image
gcloud compute instances create VM_NAME --image=IMAGE_NAME --image-project=PROJECT_ID --zone=ZONE
```

**Image Families**

```bash
# Create image and assign to a family (latest in family is used by default)
gcloud compute images create IMAGE_NAME \
  --source-disk=DISK_NAME \
  --source-disk-zone=ZONE \
  --family=my-app-images

# Create a VM using the latest image from a family
gcloud compute instances create VM_NAME \
  --image-family=my-app-images \
  --image-project=PROJECT_ID \
  --zone=ZONE
```

> **Exam tip:** Images are **global** resources. Image families always point to the most recent non-deprecated image. Best practice: stop the VM before creating an image from its disk for file system consistency. Images vs. snapshots: images are for creating new VMs, snapshots are for backup/restore.

Docs: [Custom images](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images) | [Image families](https://cloud.google.com/compute/docs/images/image-families-best-practices)

---

## 4.2 Managing GKE Resources

### Viewing Current Running Cluster Inventory

```bash
# Get credentials for a cluster (sets kubectl context)
gcloud container clusters get-credentials CLUSTER_NAME --zone=ZONE

# List clusters
gcloud container clusters list

# Describe a cluster
gcloud container clusters describe CLUSTER_NAME --zone=ZONE

# List nodes
kubectl get nodes
kubectl get nodes -o wide

# List pods (all namespaces)
kubectl get pods --all-namespaces
kubectl get pods -A

# List pods in a specific namespace
kubectl get pods -n NAMESPACE

# List services
kubectl get services
kubectl get svc --all-namespaces

# Describe a specific resource for details
kubectl describe pod POD_NAME
kubectl describe node NODE_NAME
kubectl describe service SERVICE_NAME

# View cluster info
kubectl cluster-info

# Get all resources in all namespaces
kubectl get all -A
```

> **Exam tip:** Always run `gcloud container clusters get-credentials` first to configure `kubectl`. Know the difference between zonal and regional clusters (regional has control plane replicas in 3 zones). `kubectl get` lists, `kubectl describe` gives details, `kubectl logs` shows container logs.

Docs: [Viewing clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-admin-overview) | [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

### Configuring GKE to Access Artifact Registry

```bash
# GKE nodes use the node service account to pull images.
# Grant the service account Artifact Registry Reader role:
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_EMAIL" \
  --role="roles/artifactregistry.reader"

# Or when creating a cluster, set the right access scope:
gcloud container clusters create CLUSTER_NAME \
  --zone=ZONE \
  --scopes="https://www.googleapis.com/auth/cloud-platform"

# Image reference format for Artifact Registry:
# REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/IMAGE:TAG

# Create an Artifact Registry repository
gcloud artifacts repositories create REPO_NAME \
  --repository-format=docker \
  --location=REGION \
  --description="Docker images"

# Configure Docker authentication to Artifact Registry
gcloud auth configure-docker REGION-docker.pkg.dev

# Push an image
docker tag IMAGE REGION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE:TAG
docker push REGION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE:TAG
```

> **Exam tip:** GKE clusters created with Workload Identity use Kubernetes service accounts mapped to Google service accounts (preferred over node-level scopes). The default compute service account needs at minimum `roles/artifactregistry.reader` to pull images. Artifact Registry replaces Container Registry (gcr.io).

Docs: [Artifact Registry with GKE](https://cloud.google.com/artifact-registry/docs/integrate-gke) | [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)

---

### Working with Node Pools

A node pool is a group of nodes within a cluster that share the same configuration.

```bash
# List node pools in a cluster
gcloud container node-pools list --cluster=CLUSTER_NAME --zone=ZONE

# Add a new node pool
gcloud container node-pools create POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=ZONE \
  --machine-type=e2-standard-4 \
  --num-nodes=3 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5

# Describe a node pool
gcloud container node-pools describe POOL_NAME --cluster=CLUSTER_NAME --zone=ZONE

# Resize a node pool (manual scaling)
gcloud container clusters resize CLUSTER_NAME \
  --node-pool=POOL_NAME \
  --num-nodes=5 \
  --zone=ZONE

# Update a node pool (e.g., enable autoscaling)
gcloud container node-pools update POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=ZONE \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Delete a node pool
gcloud container node-pools delete POOL_NAME --cluster=CLUSTER_NAME --zone=ZONE

# Upgrade node pool to a specific version
gcloud container node-pools update POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=ZONE \
  --node-version=LATEST_VERSION
```

> **Exam tip:** Every cluster has at least one node pool (the default pool). You can have multiple node pools with different machine types (e.g., GPU pool for ML workloads). You cannot change a node pool's machine type -- you must create a new pool and migrate. Node auto-upgrade is enabled by default in GKE.

Docs: [Node pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools) | [Adding node pools](https://cloud.google.com/kubernetes-engine/docs/how-to/node-pools)

---

### Working with Kubernetes Resources

**Pods**

```bash
# Run a pod
kubectl run POD_NAME --image=IMAGE:TAG

# Get pod details
kubectl get pods -o wide
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl logs POD_NAME -c CONTAINER_NAME  # multi-container pod

# Execute a command in a running pod
kubectl exec -it POD_NAME -- /bin/bash

# Delete a pod
kubectl delete pod POD_NAME
```

**Deployments**

```bash
# Create a deployment
kubectl create deployment DEPLOY_NAME --image=IMAGE:TAG --replicas=3

# Update a deployment image (rolling update)
kubectl set image deployment/DEPLOY_NAME CONTAINER_NAME=NEW_IMAGE:TAG

# Check rollout status
kubectl rollout status deployment/DEPLOY_NAME

# Rollback a deployment
kubectl rollout undo deployment/DEPLOY_NAME

# Scale a deployment
kubectl scale deployment/DEPLOY_NAME --replicas=5
```

**Services**

```bash
# Expose a deployment as a service
kubectl expose deployment DEPLOY_NAME --type=LoadBalancer --port=80 --target-port=8080

# Service types: ClusterIP (internal), NodePort, LoadBalancer (external), ExternalName
kubectl get svc

# Delete a service
kubectl delete svc SERVICE_NAME
```

**StatefulSets**

```bash
# StatefulSets provide stable network identities and persistent storage
# Pods are named: STATEFULSET_NAME-0, STATEFULSET_NAME-1, etc.

kubectl get statefulsets
kubectl describe statefulset STATEFULSET_NAME
kubectl scale statefulset STATEFULSET_NAME --replicas=5
kubectl delete statefulset STATEFULSET_NAME
```

**Apply from YAML (common pattern)**

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl delete -f deployment.yaml
```

> **Exam tip:** Deployments manage stateless apps (rolling updates, scaling). StatefulSets are for stateful apps (databases) -- they guarantee ordered, graceful deployment and provide stable persistent storage with PVCs. `kubectl apply -f` is declarative (preferred for production). ClusterIP is the default service type.

Docs: [Deploying workloads](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-workloads-overview) | [Services](https://cloud.google.com/kubernetes-engine/docs/concepts/service)

---

### Managing Horizontal and Vertical Autoscaling

**Horizontal Pod Autoscaler (HPA)** -- scales the number of pod replicas.

```bash
# Create an HPA (imperative)
kubectl autoscale deployment DEPLOY_NAME --min=2 --max=10 --cpu-percent=80

# View HPAs
kubectl get hpa

# Describe HPA
kubectl describe hpa DEPLOY_NAME

# Delete HPA
kubectl delete hpa DEPLOY_NAME
```

HPA YAML example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

**Vertical Pod Autoscaler (VPA)** -- adjusts CPU/memory requests for pods.

```bash
# VPA must be enabled on the cluster
gcloud container clusters update CLUSTER_NAME \
  --enable-vertical-pod-autoscaling \
  --zone=ZONE

# View VPAs
kubectl get vpa
```

VPA YAML example:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Recreate, Auto
```

**Cluster Autoscaler** -- scales the number of nodes.

```bash
# Enable cluster autoscaler on a node pool
gcloud container clusters update CLUSTER_NAME \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --node-pool=POOL_NAME \
  --zone=ZONE
```

> **Exam tip:** HPA scales pods horizontally (more replicas). VPA scales pods vertically (bigger resource requests). Cluster autoscaler scales nodes. Do NOT use HPA and VPA on the same metric (e.g., CPU) for the same deployment -- they will conflict. VPA `updateMode: "Off"` just provides recommendations without applying changes.

Docs: [HPA](https://cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler) | [VPA](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler) | [Cluster autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)

---

## 4.3 Managing Cloud Run Resources

### Deploying New Versions

```bash
# Deploy from a container image
gcloud run deploy SERVICE_NAME \
  --image=REGION-docker.pkg.dev/PROJECT_ID/REPO/IMAGE:TAG \
  --region=REGION \
  --platform=managed

# Deploy from source code (Cloud Build builds it)
gcloud run deploy SERVICE_NAME \
  --source=. \
  --region=REGION

# Deploy with environment variables
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --set-env-vars="KEY1=val1,KEY2=val2" \
  --region=REGION

# Deploy with a specific service account
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --service-account=SA_EMAIL \
  --region=REGION

# Deploy and allow unauthenticated access
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --allow-unauthenticated \
  --region=REGION

# List services
gcloud run services list --region=REGION

# List revisions of a service
gcloud run revisions list --service=SERVICE_NAME --region=REGION

# Describe a service
gcloud run services describe SERVICE_NAME --region=REGION
```

> **Exam tip:** Each deployment creates a new **revision**. By default, 100% of traffic goes to the latest revision. Cloud Run is fully managed serverless -- you only pay for actual request processing time. `--source` triggers Cloud Build to build the container from a Dockerfile or Buildpack.

Docs: [Deploying to Cloud Run](https://cloud.google.com/run/docs/deploying) | [Cloud Run overview](https://cloud.google.com/run/docs/overview/what-is-cloud-run)

---

### Adjusting Application Traffic Splitting

```bash
# Send 100% of traffic to a specific revision
gcloud run services update-traffic SERVICE_NAME \
  --to-revisions=REVISION_NAME=100 \
  --region=REGION

# Split traffic between revisions (canary deployment)
gcloud run services update-traffic SERVICE_NAME \
  --to-revisions=REVISION_V1=90,REVISION_V2=10 \
  --region=REGION

# Send all traffic to the latest revision
gcloud run services update-traffic SERVICE_NAME \
  --to-latest \
  --region=REGION

# Tag a revision (creates a unique URL for testing, receives 0% traffic)
gcloud run services update-traffic SERVICE_NAME \
  --set-tags=canary=REVISION_NAME \
  --region=REGION
# Access via: https://canary---SERVICE_NAME-HASH.a.run.app
```

> **Exam tip:** Traffic splitting enables canary deployments and rollbacks. Revisions with a **tag** get a dedicated URL for testing without receiving production traffic. Percentages must add up to 100. You can split across more than two revisions.

Docs: [Traffic splitting](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration) | [Revision tags](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration#tags)

---

### Setting Scaling Parameters

```bash
# Set min and max instances
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --min-instances=1 \
  --max-instances=100 \
  --region=REGION

# Update scaling on an existing service
gcloud run services update SERVICE_NAME \
  --min-instances=2 \
  --max-instances=50 \
  --region=REGION

# Set concurrency (max requests per container instance)
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --concurrency=80 \
  --region=REGION

# Set CPU and memory
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --cpu=2 \
  --memory=1Gi \
  --region=REGION

# Set request timeout
gcloud run deploy SERVICE_NAME \
  --image=IMAGE \
  --timeout=300 \
  --region=REGION
```

> **Exam tip:** `--min-instances=0` means scale to zero (cold starts, but saves cost). `--min-instances=1` keeps at least one instance warm (reduces latency, costs more). `--concurrency` controls how many requests a single container handles simultaneously (default is 80). Setting `--max-instances` limits cost exposure.

Docs: [Configuring scaling](https://cloud.google.com/run/docs/configuring/min-instances) | [Concurrency](https://cloud.google.com/run/docs/about-concurrency)

---

## 4.4 Managing Storage and Database Solutions

### Managing and Securing Objects in Cloud Storage

```bash
# List buckets
gcloud storage ls

# List objects in a bucket
gcloud storage ls gs://BUCKET_NAME/

# Copy objects
gcloud storage cp LOCAL_FILE gs://BUCKET_NAME/
gcloud storage cp gs://BUCKET_NAME/OBJECT ./LOCAL_PATH
gcloud storage cp -r LOCAL_DIR gs://BUCKET_NAME/   # recursive

# Move/rename objects
gcloud storage mv gs://BUCKET/OLD_NAME gs://BUCKET/NEW_NAME

# Delete objects
gcloud storage rm gs://BUCKET_NAME/OBJECT
gcloud storage rm -r gs://BUCKET_NAME/FOLDER/  # recursive

# Make an object public (uniform bucket-level access OFF)
gcloud storage objects update gs://BUCKET_NAME/OBJECT --add-acl-grant=entity=allUsers,role=READER

# Set uniform bucket-level access (recommended -- disables ACLs)
gcloud storage buckets update gs://BUCKET_NAME --uniform-bucket-level-access

# Set bucket IAM policy
gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member=user:email@example.com \
  --role=roles/storage.objectViewer

# Generate a signed URL (temporary access)
gcloud storage sign-url gs://BUCKET_NAME/OBJECT --duration=1h
```

> **Exam tip:** Uniform bucket-level access is the recommended approach (uses IAM only, disables ACLs). Signed URLs provide time-limited access without requiring Google accounts. `gcloud storage` is the modern replacement for `gsutil`. The legacy `gsutil` commands still work but `gcloud storage` is preferred.

Docs: [Cloud Storage access control](https://cloud.google.com/storage/docs/access-control) | [gcloud storage](https://cloud.google.com/sdk/gcloud/reference/storage)

---

### Setting Object Lifecycle Management Policies

Lifecycle rules automate object transitions and deletions.

```bash
# Set lifecycle rules via a JSON file
gcloud storage buckets update gs://BUCKET_NAME --lifecycle-file=lifecycle.json

# View current lifecycle rules
gcloud storage buckets describe gs://BUCKET_NAME --format="yaml(lifecycle)"

# Remove lifecycle rules
gcloud storage buckets update gs://BUCKET_NAME --clear-lifecycle
```

Example `lifecycle.json`:

```json
{
  "rule": [
    {
      "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
      "condition": {"age": 30, "matchesStorageClass": ["STANDARD"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
      "condition": {"age": 90, "matchesStorageClass": ["NEARLINE"]}
    },
    {
      "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
      "condition": {"age": 365, "matchesStorageClass": ["COLDLINE"]}
    },
    {
      "action": {"type": "Delete"},
      "condition": {"age": 730}
    },
    {
      "action": {"type": "Delete"},
      "condition": {"isLive": false, "numNewerVersions": 3}
    }
  ]
}
```

Storage classes and minimum storage durations:

| Class | Min duration | Use case |
|-------|-------------|----------|
| **Standard** | None | Frequently accessed |
| **Nearline** | 30 days | Once per month |
| **Coldline** | 90 days | Once per quarter |
| **Archive** | 365 days | Once per year or less |

> **Exam tip:** Early deletion fees apply if you delete objects before the minimum storage duration. Lifecycle rules run asynchronously (not instant). Object versioning + lifecycle rules = powerful data retention strategy. `age` condition counts from object creation date. Lifecycle transitions only go "colder" (Standard -> Nearline -> Coldline -> Archive).

Docs: [Object lifecycle management](https://cloud.google.com/storage/docs/lifecycle) | [Storage classes](https://cloud.google.com/storage/docs/storage-classes)

---

### Executing Queries

**Cloud SQL**

```bash
# Connect to a Cloud SQL instance
gcloud sql connect INSTANCE_NAME --user=root

# Then run standard SQL
SELECT * FROM my_table LIMIT 10;

# Use Cloud SQL Proxy for persistent connections
cloud-sql-proxy PROJECT_ID:REGION:INSTANCE_NAME
```

**BigQuery**

```bash
# Run a query from the command line
bq query --use_legacy_sql=false 'SELECT * FROM `project.dataset.table` LIMIT 10'

# Run a query from a file
bq query --use_legacy_sql=false < query.sql

# List datasets
bq ls

# List tables in a dataset
bq ls dataset_name

# Show table schema
bq show project:dataset.table

# Create a dataset
bq mk --dataset PROJECT_ID:DATASET_NAME

# Load data into a table
bq load --source_format=CSV dataset.table gs://bucket/file.csv schema.json
```

**Cloud Spanner**

```bash
# Execute a query
gcloud spanner databases execute-sql DATABASE_NAME \
  --instance=INSTANCE_NAME \
  --sql="SELECT * FROM Users LIMIT 10"

# List databases
gcloud spanner databases list --instance=INSTANCE_NAME
```

**Firestore**

```bash
# Firestore queries are typically done via client libraries or the Console.
# gcloud has limited Firestore support:

# Export data
gcloud firestore export gs://BUCKET_NAME

# Import data
gcloud firestore import gs://BUCKET_NAME/EXPORT_PREFIX

# List indexes
gcloud firestore indexes composite list
```

**AlloyDB**

```bash
# List AlloyDB clusters
gcloud alloydb clusters list --region=REGION

# List AlloyDB instances
gcloud alloydb instances list --cluster=CLUSTER_NAME --region=REGION

# Connect via AlloyDB Auth Proxy or private IP with psql
# AlloyDB uses private IP only (no public IP by default)
psql -h ALLOYDB_PRIVATE_IP -U postgres
```

> **Exam tip:** BigQuery uses standard SQL by default (use `--use_legacy_sql=false` to be explicit). Cloud SQL supports MySQL, PostgreSQL, and SQL Server. Spanner is globally distributed relational DB. AlloyDB is PostgreSQL-compatible, high-performance -- always uses VPC private IP. Firestore has two modes: Native (documents) and Datastore mode (entities).

Docs: [Cloud SQL connections](https://cloud.google.com/sql/docs/mysql/connect-overview) | [BigQuery CLI](https://cloud.google.com/bigquery/docs/bq-command-line-tool) | [Spanner queries](https://cloud.google.com/spanner/docs/query-syntax) | [AlloyDB overview](https://cloud.google.com/alloydb/docs/overview)

---

### Estimating Costs of Data Storage Resources

Key pricing dimensions:

| Service | Key cost factors |
|---------|-----------------|
| **Cloud Storage** | Storage per GB/month + retrieval fees (Nearline/Coldline/Archive) + operations + egress |
| **Cloud SQL** | Instance type (vCPU + RAM) + storage (SSD/HDD) + backups + egress |
| **BigQuery** | Storage ($0.02/GB/month active, $0.01/GB/month long-term) + queries ($6.25/TB scanned on-demand) |
| **Spanner** | Nodes ($0.90/node/hr) + storage ($0.30/GB/month) |
| **Firestore** | Document reads/writes/deletes + storage + egress |
| **AlloyDB** | vCPU + memory + storage + backups |

```bash
# BigQuery: estimate query cost before running (dry run)
bq query --dry_run --use_legacy_sql=false 'SELECT * FROM `project.dataset.table`'
# Output shows bytes processed -- multiply by $6.25/TB for on-demand pricing

# Use the Google Cloud Pricing Calculator:
# https://cloud.google.com/products/calculator
```

> **Exam tip:** BigQuery `--dry_run` shows bytes scanned without executing or costing anything. BigQuery long-term storage price drops automatically after 90 days of no edits. `SELECT *` in BigQuery is expensive -- use column selection. Spanner pricing is per node-hour, not per query. Cloud Storage egress (data leaving GCP) is a common hidden cost.

Docs: [GCP Pricing Calculator](https://cloud.google.com/products/calculator) | [BigQuery pricing](https://cloud.google.com/bigquery/pricing) | [Cloud Storage pricing](https://cloud.google.com/storage/pricing)

---

### Backing Up and Restoring Database Instances

**Cloud SQL**

```bash
# Create an on-demand backup
gcloud sql backups create --instance=INSTANCE_NAME

# List backups
gcloud sql backups list --instance=INSTANCE_NAME

# Describe a backup
gcloud sql backups describe BACKUP_ID --instance=INSTANCE_NAME

# Restore from a backup (restores to the SAME instance -- overwrites!)
gcloud sql backups restore BACKUP_ID --restore-instance=INSTANCE_NAME

# Restore to a different instance (clone-based approach)
gcloud sql instances clone SOURCE_INSTANCE CLONE_INSTANCE

# Point-in-time recovery (requires binary logging / WAL enabled)
gcloud sql instances clone SOURCE_INSTANCE CLONE_INSTANCE \
  --point-in-time="2024-01-15T10:00:00Z"

# Enable automated backups
gcloud sql instances patch INSTANCE_NAME --backup-start-time=02:00

# Export to Cloud Storage (another form of backup)
gcloud sql export sql INSTANCE_NAME gs://BUCKET/backup.sql --database=DB_NAME
```

**Firestore**

```bash
# Export Firestore data (backup)
gcloud firestore export gs://BUCKET_NAME --collection-ids=COLLECTION1,COLLECTION2

# Export all collections
gcloud firestore export gs://BUCKET_NAME

# Import Firestore data (restore)
gcloud firestore import gs://BUCKET_NAME/EXPORT_PREFIX

# Schedule exports with Cloud Scheduler + Cloud Functions for automation
```

**Bigtable**

```bash
# Create a backup
cbt createbackup BACKUP_ID -instance=INSTANCE_ID -table=TABLE_NAME -ttl=7d

# Or using gcloud:
gcloud bigtable backups create BACKUP_ID \
  --instance=INSTANCE_NAME \
  --cluster=CLUSTER_NAME \
  --table=TABLE_NAME \
  --retention-period=7d

# List backups
gcloud bigtable backups list --instance=INSTANCE_NAME --cluster=CLUSTER_NAME

# Restore a table from a backup
cbt restorebackup BACKUP_ID -instance=INSTANCE_ID -table=NEW_TABLE_NAME

# Or using gcloud:
gcloud bigtable instances tables restore \
  --source-backup=BACKUP_ID \
  --source-instance=INSTANCE_NAME \
  --source-cluster=CLUSTER_NAME \
  --destination=NEW_TABLE_NAME \
  --destination-instance=INSTANCE_NAME

# Delete a backup
gcloud bigtable backups delete BACKUP_ID \
  --instance=INSTANCE_NAME \
  --cluster=CLUSTER_NAME
```

**Spanner**

```bash
# Create a backup
gcloud spanner backups create BACKUP_NAME \
  --instance=INSTANCE_NAME \
  --database=DATABASE_NAME \
  --retention-period=7d \
  --version-time="2024-01-15T10:00:00Z"  # Optional: backup as of specific time

# List backups
gcloud spanner backups list --instance=INSTANCE_NAME

# Describe a backup
gcloud spanner backups describe BACKUP_NAME --instance=INSTANCE_NAME

# Restore from a backup (creates a new database)
gcloud spanner databases restore \
  --destination-database=NEW_DATABASE \
  --destination-instance=INSTANCE_NAME \
  --source-backup=BACKUP_NAME \
  --source-instance=INSTANCE_NAME

# Point-in-time recovery (PITR)
gcloud spanner databases restore \
  --destination-database=NEW_DATABASE \
  --destination-instance=INSTANCE_NAME \
  --source-database=SOURCE_DATABASE \
  --source-instance=INSTANCE_NAME \
  --version-time="2024-01-15T12:00:00Z"

# Update backup expiration time
gcloud spanner backups update BACKUP_NAME \
  --instance=INSTANCE_NAME \
  --retention-period=14d

# Delete a backup
gcloud spanner backups delete BACKUP_NAME --instance=INSTANCE_NAME
```

**AlloyDB**

```bash
# Create an on-demand backup
gcloud alloydb backups create BACKUP_NAME \
  --cluster=CLUSTER_NAME \
  --region=REGION

# List backups
gcloud alloydb backups list --region=REGION

# Describe a backup
gcloud alloydb backups describe BACKUP_NAME --region=REGION

# Restore from a backup (creates a new cluster)
gcloud alloydb clusters restore CLUSTER_NAME \
  --backup=BACKUP_NAME \
  --region=REGION

# Point-in-time recovery (restore to specific timestamp)
gcloud alloydb clusters restore CLUSTER_NAME \
  --region=REGION \
  --point-in-time="2024-01-15T12:00:00Z"

# Delete a backup
gcloud alloydb backups delete BACKUP_NAME --region=REGION

# Configure automated backup policy (when creating/updating a cluster)
gcloud alloydb clusters update CLUSTER_NAME \
  --region=REGION \
  --automated-backup-days-of-week=MONDAY,WEDNESDAY,FRIDAY \
  --automated-backup-start-times=01:00:00 \
  --automated-backup-window=4h
```

> **Exam tip:** Cloud SQL automated backups are retained for 7 days by default (configurable up to 365). Point-in-time recovery (PITR) allows restoring to any second within the retention window -- requires transaction logs (binary logging for MySQL, WAL for PostgreSQL). `gcloud sql backups restore` restores to the **same** instance and overwrites data. Use `clone` to restore to a new instance. Firestore exports go to Cloud Storage in a proprietary format. **Bigtable backups** are table-level (not instance-level) with configurable retention. **Spanner backups** support PITR for up to 7 days using version retention (configure with `ALTER DATABASE SET OPTIONS (version_retention_period='7d')`). Spanner restores create a NEW database. **AlloyDB** has automated backups enabled by default with 14-day retention, supports continuous backups and PITR similar to Cloud SQL.

Docs: [Cloud SQL backups](https://cloud.google.com/sql/docs/mysql/backup-recovery/backups) | [Cloud SQL PITR](https://cloud.google.com/sql/docs/mysql/backup-recovery/pitr) | [Firestore export/import](https://cloud.google.com/firestore/docs/manage-data/export-import) | [Bigtable backups](https://cloud.google.com/bigtable/docs/backups) | [Spanner backups](https://cloud.google.com/spanner/docs/backup) | [Spanner PITR](https://cloud.google.com/spanner/docs/pitr) | [AlloyDB backups](https://cloud.google.com/alloydb/docs/backup-restore/backup-overview)

---

### Database Center

Database Center provides a centralized view to inventory, monitor, and manage all database instances across projects in your organization.

```bash
# Access Database Center via Cloud Console:
# Cloud Console > Database Center

# Or list database instances programmatically across services:
# Cloud SQL
gcloud sql instances list

# Spanner
gcloud spanner instances list

# AlloyDB
gcloud alloydb clusters list --region=REGION

# Bigtable
gcloud bigtable instances list

# Firestore
gcloud firestore databases list

# Memorystore (Redis)
gcloud redis instances list --region=REGION
```

**Database Center features:**

- **Unified inventory**: See all databases (Cloud SQL, Spanner, AlloyDB, Bigtable, Firestore, Memorystore) in one dashboard
- **Multi-project support**: View databases across all projects in your organization
- **Operational insights**: Performance metrics, health status, and compliance posture
- **Fleet management**: Compare configurations, identify non-compliant instances, track recommendations
- **Cost analysis**: Aggregated cost view across all database resources
- **Security & compliance**: Identify unencrypted instances, public IP exposure, audit log configurations

**Use cases:**

- **Compliance auditing**: Ensure all databases meet encryption, backup, and security standards
- **Cost optimization**: Identify underutilized or oversized database instances
- **Incident response**: Quickly locate databases experiencing performance issues
- **Inventory management**: Track what databases exist across the organization

> **Exam tip:** Database Center is a **centralized dashboard** in the Cloud Console for multi-project, multi-service database visibility. It does NOT replace individual service consoles but provides a unified view for governance, compliance, and fleet management. Particularly useful in organizations with many projects and mixed database types. Access is controlled by IAM -- users need appropriate roles for each database service to see details.

Docs: [Database Center overview](https://cloud.google.com/database-center/docs/overview)

---

### Reviewing Job Status

**BigQuery**

```bash
# List recent jobs
bq ls -j -a PROJECT_ID

# Show job details
bq show -j JOB_ID

# List running jobs only
bq ls -j --filter="RUNNING" PROJECT_ID

# Cancel a job
bq cancel JOB_ID
```

**Dataflow**

```bash
# List Dataflow jobs
gcloud dataflow jobs list --region=REGION

# Describe a specific job
gcloud dataflow jobs describe JOB_ID --region=REGION

# Cancel a job (drain lets in-flight data finish)
gcloud dataflow jobs cancel JOB_ID --region=REGION

# Drain a job (graceful stop -- finishes processing in-flight data)
gcloud dataflow jobs drain JOB_ID --region=REGION
```

> **Exam tip:** BigQuery jobs include queries, loads, exports, and copies. Dataflow `drain` is graceful (finishes current windows); `cancel` is immediate. Dataflow jobs can be streaming (infinite) or batch (finite). Use the Cloud Console or CLI to monitor job progress and errors.

Docs: [BigQuery jobs](https://cloud.google.com/bigquery/docs/managing-jobs) | [Dataflow monitoring](https://cloud.google.com/dataflow/docs/guides/monitoring-overview)

---

## 4.5 Managing Networking Resources

### Adding a Subnet to an Existing VPC

```bash
# Add a subnet to a custom-mode VPC
gcloud compute networks subnets create SUBNET_NAME \
  --network=VPC_NAME \
  --region=REGION \
  --range=10.0.1.0/24

# Add a subnet with Private Google Access enabled
gcloud compute networks subnets create SUBNET_NAME \
  --network=VPC_NAME \
  --region=REGION \
  --range=10.0.2.0/24 \
  --enable-private-ip-google-access

# Add a subnet with a secondary range (for GKE pods/services)
gcloud compute networks subnets create SUBNET_NAME \
  --network=VPC_NAME \
  --region=REGION \
  --range=10.0.3.0/24 \
  --secondary-range=pods=10.4.0.0/14,services=10.8.0.0/20

# List subnets
gcloud compute networks subnets list

# List subnets in a specific network
gcloud compute networks subnets list --filter="network:VPC_NAME"
```

> **Exam tip:** Auto-mode VPCs automatically create one subnet per region. Custom-mode VPCs have no subnets until you create them. Subnets are **regional** resources. You cannot overlap IP ranges within the same VPC. Private Google Access lets VMs with no external IP reach Google APIs.

Docs: [VPC subnets](https://cloud.google.com/vpc/docs/subnets) | [Private Google Access](https://cloud.google.com/vpc/docs/private-google-access)

---

### Expanding a Subnet

```bash
# Expand a subnet CIDR range (e.g., from /24 to /20)
gcloud compute networks subnets expand-ip-range SUBNET_NAME \
  --region=REGION \
  --prefix-length=20
```

> **Exam tip:** You can **expand** a subnet CIDR range but **never shrink** it. The new prefix length must be smaller than the current one (e.g., /24 -> /20 adds more IPs). Expansion does not disrupt existing connections. Plan your IP ranges carefully -- there is no undo.

Docs: [Expanding subnets](https://cloud.google.com/vpc/docs/subnets#expand-subnet)

---

### Reserving Static IP Addresses

**External static IP**

```bash
# Reserve a regional static external IP
gcloud compute addresses create ADDRESS_NAME --region=REGION

# Reserve a global static external IP (for global load balancers)
gcloud compute addresses create ADDRESS_NAME --global

# List reserved addresses
gcloud compute addresses list

# Assign to a VM at creation
gcloud compute instances create VM_NAME \
  --zone=ZONE \
  --address=ADDRESS_NAME

# Assign to an existing VM (must stop the VM first, or use access config)
gcloud compute instances delete-access-config VM_NAME \
  --access-config-name="external-nat" --zone=ZONE
gcloud compute instances add-access-config VM_NAME \
  --access-config-name="external-nat" \
  --address=STATIC_IP \
  --zone=ZONE
```

**Internal static IP**

```bash
# Reserve an internal static IP in a subnet
gcloud compute addresses create ADDRESS_NAME \
  --region=REGION \
  --subnet=SUBNET_NAME \
  --addresses=10.0.1.50

# Use with a VM at creation
gcloud compute instances create VM_NAME \
  --zone=ZONE \
  --private-network-ip=10.0.1.50 \
  --subnet=SUBNET_NAME
```

> **Exam tip:** Static external IPs persist across VM stop/start (ephemeral IPs change). You are billed for reserved static IPs that are **not in use** -- always release unused ones. Regional IPs are for VMs and regional load balancers. Global IPs are for global load balancers (HTTP(S), SSL Proxy, TCP Proxy). Internal IPs are always regional.

Docs: [Reserving static IPs](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address) | [Internal static IPs](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-internal-ip-address)

---

### Custom Static Routes

Custom static routes let you control how traffic is routed within your VPC, often used to route traffic through network appliances (firewalls, proxies, NAT gateways).

```bash
# Create a route with next hop as an instance (e.g., firewall VM)
gcloud compute routes create ROUTE_NAME \
  --network=VPC_NAME \
  --destination-range=10.20.0.0/16 \
  --next-hop-instance=FIREWALL_VM \
  --next-hop-instance-zone=ZONE \
  --priority=1000

# Create a route with next hop as an internal IP
gcloud compute routes create ROUTE_NAME \
  --network=VPC_NAME \
  --destination-range=10.30.0.0/16 \
  --next-hop-address=10.0.1.99 \
  --priority=1000

# Create a route with next hop as a VPN tunnel
gcloud compute routes create ROUTE_NAME \
  --network=VPC_NAME \
  --destination-range=192.168.0.0/16 \
  --next-hop-vpn-tunnel=VPN_TUNNEL_NAME \
  --next-hop-vpn-tunnel-region=REGION \
  --priority=1000

# Create a route with next hop as an internal load balancer (forwarding rule)
gcloud compute routes create ROUTE_NAME \
  --network=VPC_NAME \
  --destination-range=10.40.0.0/16 \
  --next-hop-ilb=ILB_FORWARDING_RULE \
  --next-hop-ilb-region=REGION \
  --priority=1000

# Create a route to the default internet gateway (0.0.0.0/0)
gcloud compute routes create ROUTE_NAME \
  --network=VPC_NAME \
  --destination-range=0.0.0.0/0 \
  --next-hop-gateway=default-internet-gateway \
  --priority=1000

# List all routes
gcloud compute routes list

# List routes for a specific network
gcloud compute routes list --filter="network:VPC_NAME"

# Describe a route
gcloud compute routes describe ROUTE_NAME

# Delete a route
gcloud compute routes delete ROUTE_NAME
```

**Next hop types:**

| Next hop type | Use case |
|--------------|----------|
| `--next-hop-instance` | Route through a VM (firewall, proxy, NAT) |
| `--next-hop-address` | Route to an internal IP (VM, forwarding rule) |
| `--next-hop-vpn-tunnel` | Route to on-premises via VPN |
| `--next-hop-ilb` | Route to internal load balancer for HA |
| `--next-hop-gateway` | Route to internet gateway |

**Route priority:**

- Lower number = higher priority (default is 1000)
- Range: 0 to 65535
- If multiple routes match, the most specific (longest prefix match) wins
- If specificity is equal, priority breaks the tie

> **Exam tip:** Custom static routes are needed when you want to route traffic through a **network appliance VM** (firewall, IDS/IPS) instead of directly to the destination. Common use case: force all internet-bound traffic through a firewall VM by creating a route for `0.0.0.0/0` with next hop as the firewall instance, with higher priority than the default internet gateway route. The next-hop VM must have **IP forwarding enabled** (`--can-ip-forward` flag). VPC routes are global resources but next hops are regional/zonal.

Docs: [VPC routes](https://cloud.google.com/vpc/docs/routes) | [Custom static routes](https://cloud.google.com/vpc/docs/using-routes#static-route-next-hops) | [Route priority](https://cloud.google.com/vpc/docs/routes#routeselection)

---

### Working with Cloud DNS

```bash
# Create a managed DNS zone (public)
gcloud dns managed-zones create ZONE_NAME \
  --dns-name="example.com." \
  --description="My DNS zone" \
  --visibility=public

# Create a private DNS zone (visible only within specified VPC networks)
gcloud dns managed-zones create ZONE_NAME \
  --dns-name="internal.example.com." \
  --description="Private zone" \
  --visibility=private \
  --networks=VPC_NAME

# List DNS zones
gcloud dns managed-zones list

# Start a transaction (needed for record changes)
gcloud dns record-sets transaction start --zone=ZONE_NAME

# Add an A record
gcloud dns record-sets transaction add "1.2.3.4" \
  --name="app.example.com." \
  --type=A \
  --ttl=300 \
  --zone=ZONE_NAME

# Execute the transaction
gcloud dns record-sets transaction execute --zone=ZONE_NAME

# List records in a zone
gcloud dns record-sets list --zone=ZONE_NAME

# Alternatively, use direct create (no transaction needed)
gcloud dns record-sets create "app.example.com." \
  --type=A \
  --rrdatas="1.2.3.4" \
  --ttl=300 \
  --zone=ZONE_NAME
```

> **Exam tip:** Cloud DNS is a managed authoritative DNS service. Private zones resolve only within specified VPC networks. DNS changes use a transaction model (`start`, `add`/`remove`, `execute`). The DNS name must end with a trailing dot. DNSSEC can be enabled for public zones for security.

Docs: [Cloud DNS overview](https://cloud.google.com/dns/docs/overview) | [Managing records](https://cloud.google.com/dns/docs/records)

---

### Working with Cloud NAT

Cloud NAT provides outbound internet access for VMs **without external IP addresses**.

```bash
# Create a Cloud Router (required by Cloud NAT)
gcloud compute routers create ROUTER_NAME \
  --network=VPC_NAME \
  --region=REGION

# Create a Cloud NAT gateway
gcloud compute routers nats create NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges

# Or NAT only specific subnets
gcloud compute routers nats create NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION \
  --auto-allocate-nat-external-ips \
  --nat-custom-subnet-ip-ranges=SUBNET_NAME

# Describe the NAT config
gcloud compute routers nats describe NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION

# Update NAT (e.g., change min ports per VM)
gcloud compute routers nats update NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION \
  --min-ports-per-vm=128

# Delete NAT
gcloud compute routers nats delete NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION
```

> **Exam tip:** Cloud NAT is for **private VMs to reach the internet** without external IPs (outbound only). It does NOT allow inbound connections from the internet. Cloud NAT requires a Cloud Router (but does not use BGP for NAT). Cloud NAT is regional. It supports GKE nodes and VM instances. Use it with Private Google Access for a fully private setup.

Docs: [Cloud NAT overview](https://cloud.google.com/nat/docs/overview) | [Setting up Cloud NAT](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation)

---

## 4.6 Monitoring and Logging

### Creating Cloud Monitoring Alerts

```bash
# Create an alerting policy from a JSON/YAML file
gcloud monitoring policies create --policy-from-file=alert-policy.json

# List alerting policies
gcloud monitoring policies list

# Describe an alerting policy
gcloud monitoring policies describe POLICY_ID

# Delete an alerting policy
gcloud monitoring policies delete POLICY_ID

# List notification channels (email, Slack, PagerDuty, etc.)
gcloud monitoring channels list
```

Example alerting policy JSON (CPU > 80% for 5 minutes):

```json
{
  "displayName": "High CPU Alert",
  "combiner": "OR",
  "conditions": [
    {
      "displayName": "CPU > 80%",
      "conditionThreshold": {
        "filter": "resource.type=\"gce_instance\" AND metric.type=\"compute.googleapis.com/instance/cpu/utilization\"",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 0.8,
        "duration": "300s",
        "aggregations": [
          {
            "alignmentPeriod": "60s",
            "perSeriesAligner": "ALIGN_MEAN"
          }
        ]
      }
    }
  ],
  "notificationChannels": ["projects/PROJECT_ID/notificationChannels/CHANNEL_ID"]
}
```

> **Exam tip:** Alerting policies need: a condition (metric + threshold + duration), notification channels, and documentation (optional). MQL (Monitoring Query Language) is an alternative to the filter syntax. You can alert on metrics, uptime checks, and log-based metrics. Cloud Monitoring is free for GCP metrics; custom metrics cost extra.

Docs: [Creating alerting policies](https://cloud.google.com/monitoring/alerts) | [Alerting policies API](https://cloud.google.com/monitoring/api/v3/)

---

### Creating and Ingesting Custom Metrics

```bash
# Custom metrics can be created via:
# 1. Cloud Monitoring API (writing time series data)
# 2. OpenTelemetry / Prometheus
# 3. Log-based metrics

# Create a log-based metric (counts matching log entries)
gcloud logging metrics create METRIC_NAME \
  --description="Count of 500 errors" \
  --log-filter='resource.type="gae_app" AND severity="ERROR" AND textPayload=~"500"'

# List log-based metrics
gcloud logging metrics list

# Describe a log-based metric
gcloud logging metrics describe METRIC_NAME

# Delete a log-based metric
gcloud logging metrics delete METRIC_NAME
```

Writing custom metrics via API (Python example):

```python
from google.cloud import monitoring_v3

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/{project_id}"

series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/my_metric"
series.resource.type = "global"

point = monitoring_v3.Point()
point.value.double_value = 42.0
now = time.time()
point.interval.end_time.seconds = int(now)
series.points = [point]

client.create_time_series(name=project_name, time_series=[series])
```

> **Exam tip:** Custom metric types start with `custom.googleapis.com/`. Log-based metrics turn log entries into metrics you can alert on. Two types: counter (counts entries) and distribution (extracts numeric values). Custom metrics have a cost -- log-based counter metrics are free up to a limit.

Docs: [Custom metrics](https://cloud.google.com/monitoring/custom-metrics) | [Log-based metrics](https://cloud.google.com/logging/docs/logs-based-metrics)

---

### Exporting Logs to External Systems

**Log Sinks (Log Routers)**

```bash
# Create a sink to BigQuery
gcloud logging sinks create SINK_NAME \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/DATASET_NAME \
  --log-filter='resource.type="gce_instance"'

# Create a sink to Cloud Storage
gcloud logging sinks create SINK_NAME \
  storage.googleapis.com/BUCKET_NAME \
  --log-filter='severity>=ERROR'

# Create a sink to Pub/Sub (for streaming to external/on-premises systems)
gcloud logging sinks create SINK_NAME \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/TOPIC_NAME \
  --log-filter='logName="projects/PROJECT_ID/logs/syslog"'

# Create a sink to another Cloud Logging bucket
gcloud logging sinks create SINK_NAME \
  logging.googleapis.com/projects/PROJECT_ID/locations/LOCATION/buckets/BUCKET_NAME \
  --log-filter='resource.type="gce_instance"'

# List sinks
gcloud logging sinks list

# Describe a sink
gcloud logging sinks describe SINK_NAME

# Update a sink
gcloud logging sinks update SINK_NAME \
  --log-filter='severity>=WARNING'

# IMPORTANT: Grant the sink's service account write access to the destination
# The service account is shown in the sink's writerIdentity field
gcloud logging sinks describe SINK_NAME --format="value(writerIdentity)"
```

> **Exam tip:** To export logs to an **on-premises** system, use a Pub/Sub sink + a Pub/Sub subscription that your on-prem system pulls from. Log sinks export a copy -- original logs stay in Cloud Logging. The sink's service account needs permissions on the destination (BigQuery Data Editor, Storage Object Creator, Pub/Sub Publisher). Sinks only export logs that arrive **after** the sink is created (not retroactive). Aggregated sinks at the org/folder level can export logs from all projects.

Docs: [Log sinks overview](https://cloud.google.com/logging/docs/export) | [Configuring sinks](https://cloud.google.com/logging/docs/export/configure_export_v2)

---

### Configuring Log Buckets, Log Analytics, and Log Routers

**Log Buckets**

```bash
# List log buckets
gcloud logging buckets list --location=global

# Create a custom log bucket
gcloud logging buckets create BUCKET_NAME \
  --location=LOCATION \
  --retention-days=30 \
  --description="Custom log bucket" \
  --enable-analytics

# Update retention period
gcloud logging buckets update BUCKET_NAME \
  --location=LOCATION \
  --retention-days=90

# Default buckets:
# _Required: 400-day retention, Admin Activity + System Event audit logs, cannot be deleted or modified
# _Default: 30-day retention (configurable), all other logs, can be disabled
```

**Log Analytics**

```bash
# Enable Log Analytics on a bucket (allows SQL queries on logs)
gcloud logging buckets update BUCKET_NAME \
  --location=LOCATION \
  --enable-analytics

# Once enabled, query logs using SQL in the Cloud Console Log Analytics page
# or create a linked BigQuery dataset:
gcloud logging links create LINK_NAME \
  --bucket=BUCKET_NAME \
  --location=LOCATION
```

**Log Router**

The log router processes every log entry and routes them based on inclusion/exclusion filters:

```bash
# View all sinks (these are the routing rules)
gcloud logging sinks list

# Create an exclusion filter (prevent logs from being stored)
gcloud logging sinks update _Default \
  --add-exclusion="name=exclude-debug,filter=severity=DEBUG"

# Remove an exclusion filter
gcloud logging sinks update _Default \
  --remove-exclusions=exclude-debug

# Create an exclusion at the project level
gcloud logging exclusions create EXCLUSION_NAME \
  --log-filter='resource.type="gce_instance" AND severity="DEBUG"' \
  --description="Exclude debug logs from VMs"
```

> **Exam tip:** Every project has `_Required` and `_Default` buckets. The `_Required` bucket cannot be modified or deleted (Admin Activity audit logs, System Event audit logs -- always free). `_Default` can have retention changed and exclusions added. Log Analytics enables BigQuery-like SQL queries on log data. The Log Router evaluates every log entry against all sinks -- exclusions reduce cost by preventing storage.

Docs: [Log buckets](https://cloud.google.com/logging/docs/buckets) | [Log Router](https://cloud.google.com/logging/docs/routing/overview) | [Log Analytics](https://cloud.google.com/logging/docs/log-analytics)

---

### Viewing and Filtering Logs in Cloud Logging

```bash
# Read recent logs
gcloud logging read "resource.type=gce_instance" --limit=10

# Filter by severity
gcloud logging read "severity>=ERROR" --limit=20

# Filter by time range
gcloud logging read 'timestamp>="2024-01-15T00:00:00Z" AND timestamp<="2024-01-15T23:59:59Z"'

# Filter by specific log name
gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity"' --limit=5

# Filter by a text payload
gcloud logging read 'textPayload=~"error|timeout"' --limit=10

# Filter by JSON payload field
gcloud logging read 'jsonPayload.status="500"' --limit=10

# Filter by resource labels
gcloud logging read 'resource.type="gce_instance" AND resource.labels.instance_id="123456"'

# Output as JSON
gcloud logging read "severity=ERROR" --format=json --limit=5

# Freshness flag (query recently ingested logs faster)
gcloud logging read "severity>=WARNING" --freshness=1h
```

**Common Logging Query Language operators:**

| Operator | Example |
|----------|---------|
| `=` | `severity=ERROR` |
| `!=` | `severity!=DEBUG` |
| `>=` | `severity>=WARNING` |
| `=~` | `textPayload=~"regex.*pattern"` |
| `!~` | `textPayload!~"noise"` |
| `AND` | `severity=ERROR AND resource.type="gce_instance"` |
| `OR` | `severity=ERROR OR severity=CRITICAL` |

> **Exam tip:** The Logs Explorer in the Console is the primary UI for viewing logs. Use the query builder or write queries manually. Severity levels: DEFAULT, DEBUG, INFO, NOTICE, WARNING, ERROR, CRITICAL, ALERT, EMERGENCY. The `resource.type` field is critical for filtering (e.g., `gce_instance`, `gke_container`, `cloud_run_revision`).

Docs: [Logging query language](https://cloud.google.com/logging/docs/view/logging-query-language) | [Logs Explorer](https://cloud.google.com/logging/docs/view/logs-explorer-interface)

---

### Viewing Specific Log Message Details

```bash
# Get full details of a log entry (JSON format has all fields)
gcloud logging read "severity=ERROR" --format=json --limit=1

# Key fields in a log entry:
# - logName: which log stream
# - resource: the source (type + labels)
# - timestamp: when it occurred
# - severity: log level
# - textPayload / jsonPayload / protoPayload: the actual message
# - insertId: unique ID for the entry
# - httpRequest: HTTP details (for request logs)
# - labels: custom labels
# - operation: for long-running operations
# - trace / spanId: for distributed tracing
```

In the Console Logs Explorer:
- Click any log entry to expand it
- "Expand nested fields" to see the full JSON payload
- Use "Show matching entries" to filter by a field value
- Copy the resource link to share a specific log entry
- Use "View in Log Analytics" to run SQL queries

> **Exam tip:** `protoPayload` is used for audit logs (contains `methodName`, `serviceName`, `authenticationInfo`). `jsonPayload` is for structured logs. `textPayload` is for unstructured text logs. The `insertId` + `timestamp` combination uniquely identifies a log entry.

Docs: [Log entry structure](https://cloud.google.com/logging/docs/reference/v2/rest/v2/LogEntry) | [Understanding logs](https://cloud.google.com/logging/docs/view/overview)

---

### Using Cloud Diagnostics to Research Application Issues

**Error Reporting**

```bash
# List error groups
gcloud beta error-reporting events list --service=SERVICE_NAME

# Error Reporting auto-groups exceptions by stack trace
# Works with: App Engine, Cloud Functions, Cloud Run, GKE, Compute Engine
# Languages: Go, Java, .NET, Node.js, PHP, Python, Ruby
```

**Cloud Trace**

```bash
# List traces
gcloud trace traces list --project=PROJECT_ID

# Cloud Trace collects latency data from distributed applications
# Auto-instrumented for App Engine, Cloud Run, Cloud Functions
# Use OpenTelemetry client libraries for custom instrumentation
```

**Cloud Profiler**

- Continuously profiles CPU and memory usage in production
- Low overhead (~0.5%)
- Requires adding the profiler agent/library to your application
- Available for Go, Java, Node.js, Python

**Query Insights and Index Advisor (Cloud SQL)**

```bash
# Query Insights is enabled through the Cloud Console or API:
# Console: Cloud SQL instance > Query Insights tab

# Enable Query Insights on an instance
gcloud sql instances patch INSTANCE_NAME \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=1024 \
  --insights-config-record-application-tags \
  --insights-config-record-client-address

# Disable Query Insights
gcloud sql instances patch INSTANCE_NAME \
  --no-insights-config-query-insights-enabled

# View query insights via Console:
# Cloud SQL instance > Query Insights > Top queries by latency/CPU/IO/rows
```

**Query Insights features:**

- **Top queries**: Identifies queries by latency, execution frequency, CPU time, IO, rows scanned
- **Query performance trends**: Historical performance data
- **Query tags**: Tag queries with application context for easier troubleshooting
- **Index recommendations**: Suggests missing indexes to improve performance
- **Execution plans**: Shows query execution plans

**Index Advisor:**

- Automatically analyzes query patterns and workload
- Recommends indexes to create for better performance
- Shows estimated performance improvement
- Available in Query Insights dashboard
- Supports PostgreSQL and MySQL

> **Exam tip:** Error Reporting groups errors by stack trace and tracks error frequency. Cloud Trace shows request latency across microservices (like distributed tracing). Cloud Profiler identifies CPU/memory hotspots in code. **Query Insights** is a Cloud SQL feature that identifies slow queries, shows query performance over time, and recommends indexes (Index Advisor). It's particularly useful for database performance troubleshooting. Enable Query Insights before performance issues occur for historical data. All these work together: find errors -> trace the request -> profile the bottleneck -> optimize slow queries.

Docs: [Error Reporting](https://cloud.google.com/error-reporting/docs) | [Cloud Trace](https://cloud.google.com/trace/docs) | [Cloud Profiler](https://cloud.google.com/profiler/docs) | [Query Insights](https://cloud.google.com/sql/docs/postgres/using-query-insights) | [Index Advisor](https://cloud.google.com/sql/docs/postgres/using-query-insights#index-advisor)

---

### Viewing Google Cloud Status

**Google Cloud Status Dashboard** (public):

- **URL**: [https://status.cloud.google.com/](https://status.cloud.google.com/)
- Shows real-time and historical status of **all** GCP services (global view)
- Subscribe to RSS feeds or JSON endpoints for specific products
- Not personalized -- shows all incidents regardless of your resource footprint

**Personalized Service Health** (console dashboard):

```bash
# Access Personalized Service Health:
# Cloud Console > Navigation menu > Service Health

# Or use the gcloud command:
gcloud services health

# View incidents affecting your specific resources
gcloud services health list --project=PROJECT_ID
```

**Personalized Service Health features:**

- **Resource-aware**: Shows only incidents affecting YOUR services and regions
- **Proactive notifications**: Email alerts for disruptions impacting your resources
- **Historical view**: Past incidents that affected your infrastructure
- **Impact assessment**: See which of your resources were/are affected
- **Root cause analysis**: Post-incident reports when available
- **Maintenance notifications**: Planned maintenance affecting your resources

**Differences between Status Dashboard and Personalized Service Health:**

| Feature | Status Dashboard (Public) | Personalized Service Health |
|---------|-------------------------|----------------------------|
| **Audience** | Everyone (public) | Authenticated GCP users |
| **Scope** | All GCP services globally | Only services YOU use |
| **Regions** | All regions | Only regions YOU use |
| **Personalization** | No | Yes (based on your footprint) |
| **Authentication** | Not required | Requires GCP login |
| **Notifications** | RSS/JSON only | Email, console alerts |

> **Exam tip:** If you suspect a GCP outage, check the **public Status Dashboard first** for global incidents. Then check **Personalized Service Health** to see if YOUR specific resources are impacted. The Personalized Service Health dashboard is MUCH more useful operationally because it filters out noise -- you only see incidents that actually affect your workloads. For example, if you don't use us-west1, you won't see incidents there. Service Health data is also available via the Service Health API for automation and integration with incident management tools.

Docs: [Google Cloud Status Dashboard](https://cloud.google.com/support/docs/dashboard) | [Personalized Service Health](https://cloud.google.com/service-health/docs/overview) | [Service Health notifications](https://cloud.google.com/service-health/docs/configuring-notifications)

---

### Gemini Cloud Assist for Cloud Monitoring

Gemini Cloud Assist provides AI-powered assistance for Cloud Monitoring tasks, using natural language to help create queries, explain metrics, and troubleshoot performance issues.

**Features:**

- **Natural language queries**: Ask questions about your metrics in plain English
  - Example: "Show me CPU usage for my VMs in us-central1 for the last hour"
  - Example: "What caused the spike in Cloud SQL latency at 2pm?"
- **Query generation**: Generate MQL (Monitoring Query Language) and PromQL queries from descriptions
- **Metric explanation**: Understand what a metric measures and how to interpret it
- **Alert policy creation**: Help create alerting policies by describing conditions
- **Troubleshooting assistance**: Identify anomalies and suggest investigation steps
- **Dashboard creation**: Generate custom dashboards from natural language requests

**Access:**

```bash
# Access Gemini in Cloud Monitoring:
# Cloud Console > Monitoring > Metrics Explorer
# Click the "Gemini" icon in the query builder

# Or in other Monitoring pages:
# Console > Monitoring > Alerting (Gemini icon)
# Console > Monitoring > Dashboards (Gemini icon)
```

**Example use cases:**

1. **Writing complex queries**: "Show me the 95th percentile latency for Cloud Run services grouped by region"
2. **Understanding metrics**: "What does `compute.googleapis.com/instance/cpu/utilization` measure?"
3. **Creating alerts**: "Alert me when Cloud SQL CPU exceeds 80% for 5 minutes"
4. **Investigating incidents**: "Why did my GKE pod restarts increase yesterday?"
5. **Learning PromQL**: "Convert this MQL query to PromQL"

> **Exam tip:** Gemini Cloud Assist integrates AI into Cloud Monitoring to make it easier to query metrics and create alerts without deep knowledge of MQL or PromQL. It's particularly useful for operators who don't use monitoring queries daily. Gemini is available in the Cloud Console for Monitoring, Logging, and other services. It requires appropriate IAM permissions and is subject to regional availability. Think of it as a "monitoring copilot" that helps translate intent into working queries and configurations.

Docs: [Gemini Cloud Assist overview](https://cloud.google.com/gemini/docs/overview) | [Gemini for Cloud Monitoring](https://cloud.google.com/stackdriver/docs/gemini)

---

### Active Assist for Resource Optimization

Active Assist is a portfolio of intelligent tools that provide proactive recommendations for cost optimization, security, performance, and manageability across Google Cloud.

**Recommender API** (the core engine):

```bash
# List all recommenders available
gcloud recommender recommenders list

# Common recommenders:
# - google.compute.instance.MachineTypeRecommender (VM rightsizing)
# - google.compute.instance.IdleResourceRecommender (idle VMs)
# - google.compute.disk.IdleResourceRecommender (idle disks)
# - google.iam.policy.Recommender (IAM policy recommendations)
# - google.compute.commitment.UsageCommitmentRecommender (committed use discounts)
# - google.logging.productSuggestion.ContainerRecommender (suggest better products)

# List recommendations for a specific recommender
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=PROJECT_ID \
  --location=us-central1

# Describe a specific recommendation
gcloud recommender recommendations describe RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=PROJECT_ID \
  --location=us-central1

# Mark a recommendation as claimed (you're implementing it)
gcloud recommender recommendations mark-claimed RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=PROJECT_ID \
  --location=us-central1

# Mark a recommendation as succeeded (implemented and working)
gcloud recommender recommendations mark-succeeded RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=PROJECT_ID \
  --location=us-central1

# Mark a recommendation as failed or dismissed
gcloud recommender recommendations mark-failed RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=PROJECT_ID \
  --location=us-central1
```

**Recommendation categories:**

| Category | Recommenders | Example recommendations |
|----------|-------------|------------------------|
| **Cost optimization** | VM rightsizing, idle resources, committed use discounts, unattached disks | Reduce e2-standard-4 to e2-standard-2 (saves $50/mo) |
| **Security** | IAM recommender, firewall rules, public IP exposure | Remove unused roles from service account |
| **Performance** | Machine type, disk type, network tier | Upgrade to SSD for database disk |
| **Manageability** | Product suggestions, quota management | Migrate from Compute Engine to Cloud Run for this workload |

**IAM Recommender (important for security):**

```bash
# List IAM policy recommendations
gcloud recommender recommendations list \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID \
  --location=global

# Example output: "Remove roles/editor from user@example.com (unused for 90 days)"

# Apply an IAM recommendation automatically
gcloud recommender recommendations apply RECOMMENDATION_ID \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID \
  --location=global
```

**Viewing recommendations in Console:**

- Recommendations appear on resource pages (e.g., VM instance page shows rightsizing suggestion)
- **Recommendation Hub**: Console > Recommendations (centralized view of all recommendations)
- Filter by category, service, project, impact (cost/security/performance)
- Each recommendation shows estimated savings, implementation complexity, and impact

**Commitment recommenders (CUD/SUDs):**

```bash
# Get committed use discount recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.commitment.UsageCommitmentRecommender \
  --project=PROJECT_ID \
  --location=us-central1

# Shows potential savings from 1-year or 3-year committed use discounts
```

> **Exam tip:** Active Assist / Recommender API is Google's **proactive recommendation system**. Key exam points: (1) **IAM Recommender** removes unused/excessive permissions (principle of least privilege). (2) **Idle Resource Recommender** identifies VMs and disks not in use (cost savings). (3) **Machine Type Recommender** suggests rightsizing based on actual usage (saves money). (4) Recommendations are **free** and appear in the Console automatically. (5) Use `gcloud recommender` commands to list and act on recommendations programmatically. (6) Recommendations integrate with Cloud Asset Inventory for org-wide visibility. Active Assist = "smart autopilot suggestions" for your cloud.

Docs: [Active Assist overview](https://cloud.google.com/recommender/docs/overview) | [Recommender API](https://cloud.google.com/recommender/docs/reference/rest) | [IAM Recommender](https://cloud.google.com/iam/docs/recommender-overview) | [VM rightsizing](https://cloud.google.com/compute/docs/instances/apply-machine-type-recommendations)

---

### Configuring and Deploying Ops Agent

The Ops Agent is the recommended agent for collecting logs and metrics from Compute Engine VMs. It replaces the legacy Monitoring and Logging agents.

```bash
# Install Ops Agent on a single VM
# Option 1: Using the gcloud command
gcloud compute ssh INSTANCE_NAME --zone=ZONE --command='
  curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
  sudo bash add-google-cloud-ops-agent-repo.sh --also-install
'

# Option 2: Using an OS policy assignment (fleet management)
gcloud compute os-config os-policy-assignments create OPS_AGENT_POLICY \
  --location=ZONE \
  --file=ops-agent-policy.yaml

# Check agent status on a VM
sudo systemctl status google-cloud-ops-agent

# Agent configuration file
# /etc/google-cloud-ops-agent/config.yaml
```

Example `config.yaml` (collect nginx logs + metrics):

```yaml
logging:
  receivers:
    nginx_access:
      type: nginx_access
    nginx_error:
      type: nginx_error
  service:
    pipelines:
      nginx:
        receivers:
          - nginx_access
          - nginx_error
metrics:
  receivers:
    nginx:
      type: nginx
      collection_interval: 60s
  service:
    pipelines:
      nginx:
        receivers:
          - nginx
```

> **Exam tip:** The Ops Agent combines metrics (Fluent Bit based) and logging (OpenTelemetry based) collection. It is the **recommended** agent for Compute Engine. Use OS Config agent policies for fleet-wide deployment. The agent is NOT needed for managed services like App Engine, Cloud Run, or GKE (they have built-in logging/monitoring). The VM needs the `logging.logWriter` and `monitoring.metricWriter` roles (or `roles/logging.logWriter` + `roles/monitoring.metricWriter`).

Docs: [Ops Agent overview](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent) | [Installing Ops Agent](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent/installation) | [Agent configuration](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent/configuration)

---

### Deploying Managed Service for Prometheus

Google Cloud Managed Service for Prometheus (GMP) provides fully managed, multi-cloud Prometheus monitoring.

```bash
# GMP is enabled by default on GKE Autopilot clusters
# For Standard clusters, enable managed collection:
gcloud container clusters update CLUSTER_NAME \
  --zone=ZONE \
  --enable-managed-prometheus

# Verify managed collection is enabled
kubectl get pods -n gmp-system

# Deploy a PodMonitoring resource to scrape your app:
```

Example `PodMonitoring` resource:

```yaml
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: my-app-monitoring
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

```bash
# Apply the PodMonitoring resource
kubectl apply -f pod-monitoring.yaml

# Query metrics in Cloud Monitoring using PromQL
# Console > Monitoring > Metrics Explorer > PromQL tab

# Or use the Prometheus UI frontend
kubectl port-forward svc/frontend -n gmp-system 9090
```

> **Exam tip:** GMP lets you use Prometheus-style monitoring with Cloud Monitoring as the backend (no self-managed Prometheus server needed). It uses `PodMonitoring` and `ClusterPodMonitoring` custom resources (not the Prometheus `ServiceMonitor` CRD). PromQL queries work in Cloud Monitoring. GMP is included in GKE cost at no extra charge for GCP metrics.

Docs: [Managed Service for Prometheus](https://cloud.google.com/stackdriver/docs/managed-prometheus) | [Getting started with GMP](https://cloud.google.com/stackdriver/docs/managed-prometheus/setup-managed)

---

### Configuring Audit Logs

GCP audit logs record **who did what, where, and when**.

Four types of audit logs:

| Type | Always on? | Free? | Description |
|------|-----------|-------|-------------|
| **Admin Activity** | Yes | Yes | Config changes (create/delete/update resources) |
| **System Event** | Yes | Yes | Google-driven system events (live migration, auto-scaling) |
| **Data Access** | No (must enable) | No | Data reads/writes (e.g., reading a Cloud Storage object) |
| **Policy Denied** | Yes | Yes | Requests denied by VPC Service Controls |

```bash
# View the current audit log configuration
gcloud projects get-iam-policy PROJECT_ID --format=yaml | grep -A 20 auditConfigs

# Enable Data Access audit logs for a specific service
# Edit the IAM policy YAML, then set it:

# Get the policy
gcloud projects get-iam-policy PROJECT_ID --format=yaml > policy.yaml

# Add auditConfigs section (or edit existing):
```

Example `auditConfigs` section in IAM policy:

```yaml
auditConfigs:
- auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
  service: storage.googleapis.com
- auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
  service: bigquery.googleapis.com
- auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
  service: allServices  # enables for ALL services (can be expensive!)
```

```bash
# Apply the updated policy
gcloud projects set-iam-policy PROJECT_ID policy.yaml

# View audit logs
gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity"' --limit=10

# View Data Access audit logs
gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Fdata_access"' --limit=10
```

> **Exam tip:** **Admin Activity audit logs are always on and free** -- you cannot disable them. **Data Access audit logs must be explicitly enabled** and generate high volume (costs money). Data Access logs for BigQuery are enabled by default (exception to the rule). Audit logs are stored in the `_Required` log bucket (400-day retention) for Admin Activity and System Events. Data Access logs go to `_Default` bucket (30-day retention by default). Use log sinks to export audit logs to BigQuery for long-term analysis and compliance.

Docs: [Audit logs overview](https://cloud.google.com/logging/docs/audit) | [Configuring Data Access audit logs](https://cloud.google.com/logging/docs/audit/configure-data-access)

---

## Quick Reference: Exam Tips Summary

| Topic | Key fact |
|-------|----------|
| **Snapshots** | Incremental -- only first snapshot is full. Deleting earlier snapshots is safe. |
| **Images** | Global resources. Stop VM for consistency before creating. Image families point to latest. |
| **IAP SSH** | No external IP needed. Uses IAM, not firewall rules. `roles/iap.tunnelResourceAccessor` |
| **GKE autoscaling** | HPA = more pods. VPA = bigger pods. Cluster autoscaler = more nodes. Don't mix HPA+VPA on same metric. |
| **Cloud Run** | Revisions are immutable. Traffic split for canary. min-instances=0 enables scale-to-zero. |
| **Storage lifecycle** | Only transitions "colder." Early deletion fees apply. Age counts from creation. |
| **BigQuery** | `--dry_run` for cost estimates. Long-term storage auto-discounts after 90 days. |
| **Cloud SQL backups** | PITR needs binary log/WAL. `restore` overwrites same instance. Use `clone` for new instance. |
| **Bigtable backups** | Table-level, not instance-level. Configure retention period. Use cbt or gcloud commands. |
| **Spanner backups** | Restore creates NEW database. PITR supported (up to 7 days with version retention). |
| **AlloyDB backups** | Automated backups on by default (14-day retention). Supports continuous backup and PITR. |
| **Database Center** | Centralized dashboard for all databases across projects. Supports Cloud SQL, Spanner, AlloyDB, Bigtable, Firestore, Memorystore. |
| **Custom static routes** | Route traffic through network appliance VMs. Next hop VM needs IP forwarding enabled. Lower priority number = higher priority. |
| **Query Insights** | Cloud SQL feature. Shows top queries by latency/CPU/IO. Index Advisor recommends missing indexes. |
| **Subnet expansion** | Can expand (smaller prefix), NEVER shrink. No disruption. No undo. |
| **Static IPs** | Billed when NOT in use. Regional for VMs, global for global LBs. |
| **Cloud NAT** | Outbound only for private VMs. Requires Cloud Router. Regional. |
| **Cloud DNS** | Transaction model for record changes. Trailing dot on DNS names. |
| **Audit logs** | Admin Activity = always on, free. Data Access = must enable, costs money. BigQuery Data Access is on by default. |
| **Log buckets** | `_Required` = 400 days, immutable. `_Default` = 30 days, configurable. |
| **Log sinks** | Not retroactive. Sink service account needs destination permissions. Pub/Sub for on-prem export. |
| **Service Health** | Public Status Dashboard shows all services. Personalized Service Health shows only YOUR affected resources. |
| **Gemini Cloud Assist** | AI-powered help for Cloud Monitoring. Natural language queries. Generates MQL/PromQL. Explains metrics. |
| **Active Assist** | Proactive recommendations: cost (rightsizing, idle VMs), security (IAM recommender), performance. Use Recommender API. |
| **Ops Agent** | Recommended for Compute Engine. Not needed for managed services. Replaces legacy agents. |
| **Managed Prometheus** | Enabled by default on Autopilot. Uses PodMonitoring CRD. PromQL in Cloud Monitoring. |

---

*Last updated: 2026-02-07 | Official exam guide: [cloud.google.com/learn/certification/guides/cloud-engineer](https://cloud.google.com/learn/certification/guides/cloud-engineer/)*
