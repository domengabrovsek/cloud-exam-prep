# GCP Associate Cloud Engineer -- gcloud CLI Cheat Sheet

> Quick-reference organized by service. Every command is something that could show up on the exam or that you will use daily when working with GCP.

---

## Useful Global Flags

```bash
--project=PROJECT_ID          # Override the active project for this command
--zone=ZONE                   # Override the default zone (e.g. us-central1-a)
--region=REGION               # Override the default region (e.g. us-central1)
--format=FORMAT               # Output format: json | yaml | csv | table | value | flattened
--filter="EXPRESSION"         # Server-side filter (e.g. --filter="status=RUNNING")
--quiet (-q)                  # Suppress interactive prompts (auto-confirm)
--impersonate-service-account=SA_EMAIL   # Run command as a service account
--verbosity=LEVEL             # debug | info | warning | error | critical | none
--log-http                    # Log raw HTTP request/response (useful for debugging)
--flatten=KEY                 # Flatten nested lists in output
--limit=N                     # Maximum number of results to return
--sort-by=FIELD               # Sort output by a field (prefix with ~ for descending)
```

---

## 1. gcloud config -- Configurations & Properties

```bash
# --- Configurations (named profiles) ---
gcloud config configurations create my-config
gcloud config configurations activate my-config
gcloud config configurations list
gcloud config configurations delete my-config
gcloud config configurations describe my-config

# --- Set / Unset Properties ---
gcloud config set project my-project-id
gcloud config set compute/zone us-central1-a
gcloud config set compute/region us-central1
gcloud config set account user@example.com
gcloud config unset compute/zone

# --- View current config ---
gcloud config list                              # Show all properties in active config
gcloud config list --configuration=my-config    # Show properties of a named config
gcloud config get-value project                 # Get a single property value
gcloud config get-value compute/zone

# --- Auth ---
gcloud auth login                               # Browser-based login
gcloud auth activate-service-account --key-file=key.json   # Authenticate as SA
gcloud auth application-default login           # Set Application Default Credentials (ADC)
gcloud auth list                                # List credentialed accounts
gcloud auth revoke user@example.com             # Revoke credentials
gcloud auth print-access-token                  # Print current OAuth2 access token
gcloud auth print-identity-token                # Print OIDC identity token

# --- General info ---
gcloud info                                     # SDK installation diagnostics
gcloud components list                          # List installed SDK components
gcloud components install COMPONENT             # Install a component (e.g. kubectl, bq)
gcloud components update                        # Update all SDK components
gcloud init                                     # Interactive setup wizard
```

---

## 2. gcloud compute -- Compute Engine

### Instances

```bash
# Create a VM
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd \
  --tags=http-server,https-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx'

# Create a preemptible / spot VM
gcloud compute instances create spot-vm \
  --zone=us-central1-a \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# List / describe / delete
gcloud compute instances list
gcloud compute instances list --filter="zone:us-central1-a AND status=RUNNING"
gcloud compute instances describe my-vm --zone=us-central1-a
gcloud compute instances delete my-vm --zone=us-central1-a

# Start / stop / reset / suspend / resume
gcloud compute instances start my-vm --zone=us-central1-a
gcloud compute instances stop my-vm --zone=us-central1-a
gcloud compute instances reset my-vm --zone=us-central1-a
gcloud compute instances suspend my-vm --zone=us-central1-a
gcloud compute instances resume my-vm --zone=us-central1-a

# SSH into a VM
gcloud compute ssh my-vm --zone=us-central1-a
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap   # SSH via IAP

# SCP files to/from a VM
gcloud compute scp local-file.txt my-vm:~/remote-file.txt --zone=us-central1-a
gcloud compute scp my-vm:~/remote-file.txt local-file.txt --zone=us-central1-a

# Move a VM to a different zone (within same region)
gcloud compute instances move my-vm --zone=us-central1-a --destination-zone=us-central1-b

# Update machine type (VM must be stopped)
gcloud compute instances set-machine-type my-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4

# Add/remove tags
gcloud compute instances add-tags my-vm --zone=us-central1-a --tags=allow-ssh
gcloud compute instances remove-tags my-vm --zone=us-central1-a --tags=allow-ssh

# Add/remove metadata
gcloud compute instances add-metadata my-vm --zone=us-central1-a \
  --metadata=key1=value1,key2=value2
gcloud compute instances remove-metadata my-vm --zone=us-central1-a --keys=key1

# Set labels
gcloud compute instances update my-vm --zone=us-central1-a \
  --update-labels=env=prod,team=backend

# Attach a service account to an existing VM (must be stopped)
gcloud compute instances set-service-account my-vm \
  --zone=us-central1-a \
  --service-account=my-sa@my-project.iam.gserviceaccount.com \
  --scopes=cloud-platform

# Serial port output (debugging boot issues)
gcloud compute instances get-serial-port-output my-vm --zone=us-central1-a
```

### Disks

```bash
# Create a persistent disk
gcloud compute disks create my-disk \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-ssd

# List / describe / delete
gcloud compute disks list
gcloud compute disks describe my-disk --zone=us-central1-a
gcloud compute disks delete my-disk --zone=us-central1-a

# Resize a disk (can only increase, not decrease)
gcloud compute disks resize my-disk --zone=us-central1-a --size=200GB

# Attach / detach a disk to/from a VM
gcloud compute instances attach-disk my-vm --disk=my-disk --zone=us-central1-a
gcloud compute instances detach-disk my-vm --disk=my-disk --zone=us-central1-a
```

### Snapshots

```bash
# Create a snapshot from a disk
gcloud compute snapshots create my-snapshot --source-disk=my-disk --source-disk-zone=us-central1-a

# List / describe / delete
gcloud compute snapshots list
gcloud compute snapshots describe my-snapshot
gcloud compute snapshots delete my-snapshot

# Create a disk from a snapshot
gcloud compute disks create restored-disk --source-snapshot=my-snapshot --zone=us-central1-a

# Scheduled snapshots (snapshot schedule)
gcloud compute resource-policies create snapshot-schedule my-schedule \
  --region=us-central1 \
  --max-retention-days=14 \
  --start-time=04:00 \
  --daily-schedule

gcloud compute disks add-resource-policies my-disk \
  --zone=us-central1-a \
  --resource-policies=my-schedule
```

### Images

```bash
# Create a custom image from a disk
gcloud compute images create my-image --source-disk=my-disk --source-disk-zone=us-central1-a

# Create an image from a snapshot
gcloud compute images create my-image --source-snapshot=my-snapshot

# List / describe / delete
gcloud compute images list                                  # All public + custom images
gcloud compute images list --project=debian-cloud           # Images from a specific project
gcloud compute images describe my-image
gcloud compute images delete my-image

# Deprecate an image
gcloud compute images deprecate my-image --state=DEPRECATED --replacement=my-new-image

# Export an image to Cloud Storage
gcloud compute images export --destination-uri=gs://my-bucket/my-image.tar.gz --image=my-image
```

### Firewall Rules

```bash
# Create a firewall rule
gcloud compute firewall-rules create allow-http \
  --network=default \
  --allow=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server \
  --priority=1000 \
  --direction=INGRESS \
  --description="Allow HTTP traffic"

# Allow internal traffic
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --allow=tcp,udp,icmp \
  --source-ranges=10.128.0.0/9

# Create an egress deny rule
gcloud compute firewall-rules create deny-all-egress \
  --network=my-vpc \
  --action=DENY \
  --rules=all \
  --direction=EGRESS \
  --priority=65534

# List / describe / update / delete
gcloud compute firewall-rules list
gcloud compute firewall-rules list --filter="network:my-vpc"
gcloud compute firewall-rules describe allow-http
gcloud compute firewall-rules update allow-http --source-ranges=10.0.0.0/8
gcloud compute firewall-rules delete allow-http
```

### VPC Networks & Subnets

```bash
# Create a VPC (custom mode)
gcloud compute networks create my-vpc --subnet-mode=custom

# Create a VPC (auto mode -- auto-creates subnets in each region)
gcloud compute networks create my-vpc --subnet-mode=auto

# Create a subnet
gcloud compute networks subnets create my-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.0.0/24 \
  --enable-private-ip-google-access

# Expand a subnet range (can only increase, not decrease)
gcloud compute networks subnets expand-ip-range my-subnet \
  --region=us-central1 \
  --prefix-length=20

# List / describe / delete
gcloud compute networks list
gcloud compute networks describe my-vpc
gcloud compute networks subnets list
gcloud compute networks subnets list --network=my-vpc
gcloud compute networks subnets describe my-subnet --region=us-central1
gcloud compute networks delete my-vpc

# Enable Private Google Access on existing subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access

# Enable VPC Flow Logs on a subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-flow-logs

# VPC Peering
gcloud compute networks peerings create peer-ab \
  --network=vpc-a \
  --peer-network=vpc-b \
  --peer-project=other-project-id

gcloud compute networks peerings list --network=vpc-a
gcloud compute networks peerings delete peer-ab --network=vpc-a

# Shared VPC
gcloud compute shared-vpc enable HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects list HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects remove SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID
```

### Addresses (Static IPs)

```bash
# Reserve an external static IP
gcloud compute addresses create my-static-ip --region=us-central1

# Reserve a global static IP (for global load balancers)
gcloud compute addresses create my-global-ip --global

# Reserve an internal static IP
gcloud compute addresses create my-internal-ip \
  --region=us-central1 \
  --subnet=my-subnet \
  --addresses=10.0.0.50

# List / describe / delete
gcloud compute addresses list
gcloud compute addresses describe my-static-ip --region=us-central1
gcloud compute addresses delete my-static-ip --region=us-central1
```

### Instance Templates

```bash
# Create an instance template
gcloud compute instance-templates create my-template \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx'

# Create a template from an existing instance
gcloud compute instance-templates create my-template \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a

# List / describe / delete
gcloud compute instance-templates list
gcloud compute instance-templates describe my-template
gcloud compute instance-templates delete my-template
```

### Managed Instance Groups (MIGs)

```bash
# Create a managed instance group
gcloud compute instance-groups managed create my-mig \
  --zone=us-central1-a \
  --template=my-template \
  --size=3

# Create a regional (multi-zone) MIG
gcloud compute instance-groups managed create my-regional-mig \
  --region=us-central1 \
  --template=my-template \
  --size=3

# Set autoscaling
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --min-num-replicas=1 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=90

# Stop autoscaling
gcloud compute instance-groups managed stop-autoscaling my-mig --zone=us-central1-a

# Resize manually
gcloud compute instance-groups managed resize my-mig --zone=us-central1-a --size=5

# Rolling update to a new template
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --zone=us-central1-a \
  --version=template=my-new-template \
  --max-surge=1 \
  --max-unavailable=0

# Restart / replace all instances
gcloud compute instance-groups managed rolling-action restart my-mig --zone=us-central1-a
gcloud compute instance-groups managed rolling-action replace my-mig --zone=us-central1-a

# Set named ports (for load balancers)
gcloud compute instance-groups managed set-named-ports my-mig \
  --zone=us-central1-a \
  --named-ports=http:80,https:443

# List / describe / delete
gcloud compute instance-groups managed list
gcloud compute instance-groups managed describe my-mig --zone=us-central1-a
gcloud compute instance-groups managed delete my-mig --zone=us-central1-a
gcloud compute instance-groups managed list-instances my-mig --zone=us-central1-a
```

---

## 3. gcloud container -- Google Kubernetes Engine (GKE)

### Clusters

```bash
# Create an Autopilot cluster (Google manages nodes)
gcloud container clusters create-auto my-autopilot-cluster \
  --region=us-central1

# Create a Standard cluster
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-medium \
  --disk-size=50GB \
  --enable-autoscaling --min-nodes=1 --max-nodes=10 \
  --enable-autorepair \
  --enable-autoupgrade

# Create a private cluster (nodes have no external IPs)
gcloud container clusters create my-private-cluster \
  --zone=us-central1-a \
  --enable-private-nodes \
  --master-ipv4-cidr=172.16.0.0/28 \
  --enable-ip-alias

# Get credentials (configure kubectl)
gcloud container clusters get-credentials my-cluster --zone=us-central1-a

# List / describe / delete
gcloud container clusters list
gcloud container clusters describe my-cluster --zone=us-central1-a
gcloud container clusters delete my-cluster --zone=us-central1-a

# Resize a cluster (set node count per zone)
gcloud container clusters resize my-cluster --zone=us-central1-a --num-nodes=5

# Update a cluster
gcloud container clusters update my-cluster --zone=us-central1-a \
  --enable-autoscaling --min-nodes=2 --max-nodes=8

# Upgrade the control plane
gcloud container clusters upgrade my-cluster --zone=us-central1-a --master

# Upgrade nodes
gcloud container clusters upgrade my-cluster --zone=us-central1-a
```

### Node Pools

```bash
# Create a node pool
gcloud container node-pools create my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=e2-standard-4 \
  --enable-autoscaling --min-nodes=1 --max-nodes=5

# List / describe / delete
gcloud container node-pools list --cluster=my-cluster --zone=us-central1-a
gcloud container node-pools describe my-pool --cluster=my-cluster --zone=us-central1-a
gcloud container node-pools delete my-pool --cluster=my-cluster --zone=us-central1-a

# Resize a node pool
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-autoscaling --min-nodes=2 --max-nodes=10

# Upgrade a node pool
gcloud container node-pools upgrade my-pool --cluster=my-cluster --zone=us-central1-a
```

### Container Images (Artifact Registry / Container Registry)

```bash
# Configure Docker to authenticate with Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev

# Build and push (using Cloud Build)
gcloud builds submit --tag us-central1-docker.pkg.dev/my-project/my-repo/my-image:v1

# List images in Artifact Registry
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo

# Create an Artifact Registry repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="My Docker repo"

# List repositories
gcloud artifacts repositories list --location=us-central1
```

---

## 4. gcloud run -- Cloud Run

```bash
# Deploy a container image
gcloud run deploy my-service \
  --image=us-central1-docker.pkg.dev/my-project/my-repo/my-image:v1 \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --port=8080 \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=10 \
  --set-env-vars=KEY1=VAL1,KEY2=VAL2

# Deploy from source (builds automatically with Cloud Build)
gcloud run deploy my-service --source=. --region=us-central1

# List services / revisions
gcloud run services list --region=us-central1
gcloud run services describe my-service --region=us-central1
gcloud run revisions list --service=my-service --region=us-central1

# Update environment variables
gcloud run services update my-service --region=us-central1 \
  --update-env-vars=KEY1=NEW_VAL1

# Set traffic split between revisions
gcloud run services update-traffic my-service --region=us-central1 \
  --to-revisions=my-service-00001-abc=50,my-service-00002-def=50

# Route all traffic to latest
gcloud run services update-traffic my-service --region=us-central1 --to-latest

# Delete a service
gcloud run services delete my-service --region=us-central1

# View logs
gcloud run services logs read my-service --region=us-central1

# Set IAM policy (make public / restrict)
gcloud run services add-iam-policy-binding my-service \
  --region=us-central1 \
  --member="allUsers" \
  --role="roles/run.invoker"

gcloud run services remove-iam-policy-binding my-service \
  --region=us-central1 \
  --member="allUsers" \
  --role="roles/run.invoker"
```

---

## 5. gcloud functions -- Cloud Functions

```bash
# Deploy a Gen2 function (HTTP trigger)
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point=main \
  --source=. \
  --memory=256Mi \
  --timeout=60s \
  --set-env-vars=KEY=VALUE

# Deploy with Pub/Sub trigger
gcloud functions deploy my-function \
  --gen2 \
  --runtime=nodejs20 \
  --region=us-central1 \
  --trigger-topic=my-topic \
  --entry-point=handler

# Deploy with Cloud Storage trigger
gcloud functions deploy my-function \
  --gen2 \
  --runtime=go122 \
  --region=us-central1 \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-bucket" \
  --entry-point=ProcessFile

# List / describe / delete
gcloud functions list --region=us-central1
gcloud functions describe my-function --region=us-central1
gcloud functions delete my-function --region=us-central1

# View logs
gcloud functions logs read my-function --region=us-central1 --limit=50

# Call a function directly (for testing)
gcloud functions call my-function --region=us-central1 --data='{"key":"value"}'
```

---

## 6. gcloud iam -- Identity and Access Management

### Roles

```bash
# List predefined roles
gcloud iam roles list

# List custom roles in a project
gcloud iam roles list --project=my-project

# Describe a role (see its permissions)
gcloud iam roles describe roles/compute.admin
gcloud iam roles describe roles/editor

# Create a custom role
gcloud iam roles create myCustomRole --project=my-project \
  --title="My Custom Role" \
  --description="A custom role" \
  --permissions=compute.instances.get,compute.instances.list \
  --stage=GA

# Update a custom role
gcloud iam roles update myCustomRole --project=my-project \
  --add-permissions=compute.instances.start,compute.instances.stop

# Disable / delete / undelete a custom role
gcloud iam roles update myCustomRole --project=my-project --stage=DISABLED
gcloud iam roles delete myCustomRole --project=my-project
gcloud iam roles undelete myCustomRole --project=my-project

# List all grantable roles on a resource
gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/my-project
```

### Service Accounts

```bash
# Create a service account
gcloud iam service-accounts create my-sa \
  --display-name="My Service Account" \
  --description="Used for CI/CD pipeline"

# List / describe / delete
gcloud iam service-accounts list
gcloud iam service-accounts describe my-sa@my-project.iam.gserviceaccount.com
gcloud iam service-accounts delete my-sa@my-project.iam.gserviceaccount.com

# Enable / disable
gcloud iam service-accounts enable my-sa@my-project.iam.gserviceaccount.com
gcloud iam service-accounts disable my-sa@my-project.iam.gserviceaccount.com

# Create and download a key
gcloud iam service-accounts keys create ~/key.json \
  --iam-account=my-sa@my-project.iam.gserviceaccount.com

# List keys
gcloud iam service-accounts keys list --iam-account=my-sa@my-project.iam.gserviceaccount.com

# Delete a key
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=my-sa@my-project.iam.gserviceaccount.com
```

### IAM Policy Bindings (Project-Level)

```bash
# View the full IAM policy for a project
gcloud projects get-iam-policy my-project

# Grant a role to a user
gcloud projects add-iam-policy-binding my-project \
  --member="user:alice@example.com" \
  --role="roles/editor"

# Grant a role to a service account
gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:my-sa@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Grant a role to a group
gcloud projects add-iam-policy-binding my-project \
  --member="group:devs@example.com" \
  --role="roles/viewer"

# Remove a role binding
gcloud projects remove-iam-policy-binding my-project \
  --member="user:alice@example.com" \
  --role="roles/editor"

# Grant a role with a condition
gcloud projects add-iam-policy-binding my-project \
  --member="user:alice@example.com" \
  --role="roles/compute.admin" \
  --condition='expression=request.time < timestamp("2026-12-31T00:00:00Z"),title=Temporary Access,description=Expires end of 2026'

# Bind IAM on a service account (who can act as this SA)
gcloud iam service-accounts add-iam-policy-binding \
  my-sa@my-project.iam.gserviceaccount.com \
  --member="user:alice@example.com" \
  --role="roles/iam.serviceAccountUser"
```

---

## 7. gcloud projects -- Project Management

```bash
# Create a project
gcloud projects create my-project-id --name="My Project"
gcloud projects create my-project-id --name="My Project" --folder=123456789
gcloud projects create my-project-id --name="My Project" --organization=987654321

# List / describe
gcloud projects list
gcloud projects describe my-project-id

# Delete / undelete (30-day recovery window)
gcloud projects delete my-project-id
gcloud projects undelete my-project-id

# Update labels
gcloud projects update my-project-id --update-labels=env=prod,team=platform

# Get / set IAM policy
gcloud projects get-iam-policy my-project-id
gcloud projects get-iam-policy my-project-id --format=json > policy.json
gcloud projects set-iam-policy my-project-id policy.json

# Add / remove IAM bindings (same as Section 6)
gcloud projects add-iam-policy-binding my-project-id \
  --member="user:alice@example.com" \
  --role="roles/viewer"

gcloud projects remove-iam-policy-binding my-project-id \
  --member="user:alice@example.com" \
  --role="roles/viewer"
```

---

## 8. gcloud organizations -- Organization Management

```bash
# List organizations (you must have orgViewer or equivalent)
gcloud organizations list

# Describe an organization
gcloud organizations describe ORG_ID

# Get IAM policy on the org
gcloud organizations get-iam-policy ORG_ID

# Add IAM binding at the org level
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="user:admin@example.com" \
  --role="roles/resourcemanager.organizationAdmin"

# Remove IAM binding at the org level
gcloud organizations remove-iam-policy-binding ORG_ID \
  --member="user:admin@example.com" \
  --role="roles/resourcemanager.organizationAdmin"
```

---

## 9. gcloud resource-manager -- Folders

```bash
# Create a folder
gcloud resource-manager folders create --display-name="Engineering" --organization=ORG_ID
gcloud resource-manager folders create --display-name="Team-A" --folder=PARENT_FOLDER_ID

# List folders
gcloud resource-manager folders list --organization=ORG_ID
gcloud resource-manager folders list --folder=PARENT_FOLDER_ID

# Describe / update / delete
gcloud resource-manager folders describe FOLDER_ID
gcloud resource-manager folders update FOLDER_ID --display-name="New Name"
gcloud resource-manager folders delete FOLDER_ID

# Move a folder under a different parent
gcloud resource-manager folders move FOLDER_ID --folder=NEW_PARENT_FOLDER_ID

# Move a project into a folder
gcloud projects move my-project-id --folder=FOLDER_ID

# Get / add / remove IAM on a folder
gcloud resource-manager folders get-iam-policy FOLDER_ID
gcloud resource-manager folders add-iam-policy-binding FOLDER_ID \
  --member="group:devs@example.com" \
  --role="roles/editor"
gcloud resource-manager folders remove-iam-policy-binding FOLDER_ID \
  --member="group:devs@example.com" \
  --role="roles/editor"
```

---

## 10. gcloud services -- API Management

```bash
# Enable an API
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable cloudfunctions.googleapis.com
gcloud services enable run.googleapis.com
gcloud services enable storage.googleapis.com
gcloud services enable bigquery.googleapis.com
gcloud services enable sqladmin.googleapis.com

# Enable multiple APIs at once
gcloud services enable \
  compute.googleapis.com \
  container.googleapis.com \
  cloudbuild.googleapis.com

# Disable an API
gcloud services disable compute.googleapis.com
gcloud services disable compute.googleapis.com --force   # Force-disable even if in use

# List enabled APIs
gcloud services list --enabled

# List available (all) APIs
gcloud services list --available

# Check if a specific API is enabled
gcloud services list --enabled --filter="name:compute.googleapis.com"
```

---

## 11. Cloud Storage -- gcloud storage (modern) & gsutil (legacy)

### gcloud storage (recommended)

```bash
# Create a bucket
gcloud storage buckets create gs://my-bucket --location=us-central1
gcloud storage buckets create gs://my-bucket --location=us --uniform-bucket-level-access

# List buckets
gcloud storage buckets list

# Describe a bucket
gcloud storage buckets describe gs://my-bucket

# Delete a bucket (must be empty, or use --recursive)
gcloud storage rm --recursive gs://my-bucket

# Upload / download files
gcloud storage cp local-file.txt gs://my-bucket/
gcloud storage cp gs://my-bucket/remote-file.txt ./local-file.txt
gcloud storage cp -r ./local-dir gs://my-bucket/          # Recursive upload

# Move / rename
gcloud storage mv gs://my-bucket/old-name.txt gs://my-bucket/new-name.txt
gcloud storage mv gs://my-bucket/file.txt gs://other-bucket/

# List objects
gcloud storage ls gs://my-bucket
gcloud storage ls gs://my-bucket/**                        # Recursive listing

# Delete objects
gcloud storage rm gs://my-bucket/file.txt
gcloud storage rm --recursive gs://my-bucket/folder/

# Sync directories
gcloud storage rsync ./local-dir gs://my-bucket/dir --recursive
gcloud storage rsync gs://my-bucket/dir ./local-dir --recursive
gcloud storage rsync ./local-dir gs://my-bucket/dir --recursive --delete-unmatched-destination-objects

# Set bucket labels
gcloud storage buckets update gs://my-bucket --update-labels=env=prod

# Set lifecycle policy
gcloud storage buckets update gs://my-bucket --lifecycle-file=lifecycle.json

# Set / get IAM on a bucket
gcloud storage buckets get-iam-policy gs://my-bucket
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member="user:alice@example.com" \
  --role="roles/storage.objectViewer"

# Enable versioning
gcloud storage buckets update gs://my-bucket --versioning

# Enable uniform bucket-level access
gcloud storage buckets update gs://my-bucket --uniform-bucket-level-access

# Set default storage class
gcloud storage buckets update gs://my-bucket --default-storage-class=NEARLINE

# Generate signed URL
gcloud storage sign-url gs://my-bucket/file.txt --duration=1h \
  --private-key-file=key.json
```

### gsutil (legacy -- still valid on the exam)

```bash
# Create a bucket
gsutil mb -l us-central1 gs://my-bucket

# Copy / move
gsutil cp local-file.txt gs://my-bucket/
gsutil cp -r ./local-dir gs://my-bucket/
gsutil mv gs://my-bucket/old.txt gs://my-bucket/new.txt

# List / remove
gsutil ls gs://my-bucket
gsutil rm gs://my-bucket/file.txt
gsutil rm -r gs://my-bucket

# Sync
gsutil rsync -r ./local-dir gs://my-bucket/dir
gsutil rsync -r -d ./local-dir gs://my-bucket/dir           # -d deletes extra files at dest

# Set ACL (fine-grained only)
gsutil acl set private gs://my-bucket/file.txt
gsutil acl ch -u alice@example.com:READ gs://my-bucket/file.txt

# Set IAM on a bucket
gsutil iam ch user:alice@example.com:objectViewer gs://my-bucket

# Enable versioning
gsutil versioning set on gs://my-bucket

# View lifecycle config
gsutil lifecycle get gs://my-bucket
gsutil lifecycle set lifecycle.json gs://my-bucket

# Set CORS
gsutil cors set cors.json gs://my-bucket

# Set default storage class
gsutil defstorageclass set NEARLINE gs://my-bucket

# Compose objects (combine multiple objects into one)
gsutil compose gs://my-bucket/part1 gs://my-bucket/part2 gs://my-bucket/combined

# Get / set metadata
gsutil stat gs://my-bucket/file.txt
gsutil setmeta -h "Content-Type:text/plain" gs://my-bucket/file.txt

# Generate signed URL
gsutil signurl -d 1h key.json gs://my-bucket/file.txt
```

---

## 12. bq -- BigQuery

```bash
# Run a query
bq query --use_legacy_sql=false 'SELECT * FROM `project.dataset.table` LIMIT 10'

# Run a query and save to destination table
bq query --use_legacy_sql=false --destination_table=project:dataset.new_table \
  'SELECT * FROM `project.dataset.table` WHERE year = 2025'

# Create a dataset
bq mk --dataset my-project:my_dataset
bq mk --dataset --location=US my-project:my_dataset

# Create a table with schema
bq mk --table my-project:my_dataset.my_table name:STRING,age:INTEGER,active:BOOLEAN

# Create a table from a schema file
bq mk --table my-project:my_dataset.my_table schema.json

# List datasets / tables
bq ls                                          # List datasets in default project
bq ls my-project:                              # List datasets in a specific project
bq ls my-project:my_dataset                    # List tables in a dataset

# Show table info
bq show my-project:my_dataset.my_table
bq show --schema my-project:my_dataset.my_table

# Load data from Cloud Storage
bq load --source_format=CSV --autodetect \
  my-project:my_dataset.my_table gs://my-bucket/data.csv

bq load --source_format=NEWLINE_DELIMITED_JSON \
  my-project:my_dataset.my_table gs://my-bucket/data.json schema.json

bq load --source_format=PARQUET \
  my-project:my_dataset.my_table gs://my-bucket/data.parquet

# Export a table to Cloud Storage
bq extract --destination_format=CSV \
  my-project:my_dataset.my_table gs://my-bucket/export/data-*.csv

bq extract --destination_format=NEWLINE_DELIMITED_JSON \
  my-project:my_dataset.my_table gs://my-bucket/export/data.json

# Copy a table
bq cp my-project:my_dataset.source_table my-project:my_dataset.dest_table

# Delete a table / dataset
bq rm -t my-project:my_dataset.my_table
bq rm -r -d my-project:my_dataset                # -r removes all tables in dataset

# Create a view
bq mk --view 'SELECT name, age FROM `my-project.my_dataset.my_table`' \
  my-project:my_dataset.my_view

# Update table expiration (seconds)
bq update --expiration=86400 my-project:my_dataset.my_table

# Set dataset-level access
bq show --format=prettyjson my-project:my_dataset > dataset.json
# (edit access in dataset.json)
bq update --source=dataset.json my-project:my_dataset

# Create a partitioned table
bq mk --table --time_partitioning_type=DAY --time_partitioning_field=created_at \
  my-project:my_dataset.partitioned_table schema.json

# Create a clustered table
bq mk --table --clustering_fields=country,city \
  my-project:my_dataset.clustered_table schema.json
```

---

## 13. kubectl -- Kubernetes Essentials

```bash
# --- Context / Config ---
kubectl config get-contexts                     # List available contexts
kubectl config current-context                  # Show current context
kubectl config use-context my-context           # Switch context
kubectl config set-context --current --namespace=my-ns   # Set default namespace

# --- Get resources ---
kubectl get pods                                # List pods in current namespace
kubectl get pods -n kube-system                 # List pods in a specific namespace
kubectl get pods -o wide                        # Show more details (node, IP)
kubectl get pods --all-namespaces               # Across all namespaces
kubectl get deployments
kubectl get services
kubectl get nodes
kubectl get namespaces
kubectl get configmaps
kubectl get secrets
kubectl get ingress
kubectl get pv                                  # Persistent volumes
kubectl get pvc                                 # Persistent volume claims
kubectl get all                                 # All resource types

# --- Describe (detailed info + events) ---
kubectl describe pod my-pod
kubectl describe deployment my-deployment
kubectl describe service my-service
kubectl describe node my-node

# --- Apply / Create ---
kubectl apply -f deployment.yaml                # Create or update from a file
kubectl apply -f ./manifests/                   # Apply all YAML in a directory
kubectl apply -f https://example.com/manifest.yaml
kubectl create namespace my-ns
kubectl create configmap my-config --from-literal=key1=val1 --from-literal=key2=val2
kubectl create secret generic my-secret --from-literal=password=s3cr3t

# --- Delete ---
kubectl delete pod my-pod
kubectl delete deployment my-deployment
kubectl delete service my-service
kubectl delete -f deployment.yaml               # Delete resources defined in a file
kubectl delete pods --all -n my-ns              # Delete all pods in a namespace

# --- Scale ---
kubectl scale deployment my-deployment --replicas=5

# --- Autoscale ---
kubectl autoscale deployment my-deployment --min=2 --max=10 --cpu-percent=80

# --- Rollouts ---
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment/my-deployment
kubectl rollout undo deployment/my-deployment --to-revision=2
kubectl rollout restart deployment/my-deployment

# --- Logs ---
kubectl logs my-pod
kubectl logs my-pod -c my-container             # Specific container in a multi-container pod
kubectl logs my-pod --previous                  # Logs from the previous (crashed) container
kubectl logs -f my-pod                          # Follow / stream logs
kubectl logs -l app=my-app                      # Logs by label selector

# --- Exec (run commands inside a container) ---
kubectl exec -it my-pod -- /bin/bash
kubectl exec my-pod -- ls /app
kubectl exec -it my-pod -c my-container -- /bin/sh

# --- Port forwarding ---
kubectl port-forward pod/my-pod 8080:80
kubectl port-forward svc/my-service 8080:80

# --- Copy files ---
kubectl cp my-pod:/app/data.txt ./data.txt
kubectl cp ./local-file.txt my-pod:/app/

# --- Labels and annotations ---
kubectl label pods my-pod env=prod
kubectl label pods my-pod env-                  # Remove a label
kubectl annotate pods my-pod description="my pod"

# --- Top (resource usage) ---
kubectl top nodes
kubectl top pods
kubectl top pods --sort-by=memory

# --- Set image (update container image) ---
kubectl set image deployment/my-deployment my-container=my-image:v2

# --- Expose a deployment as a service ---
kubectl expose deployment my-deployment --port=80 --target-port=8080 --type=LoadBalancer

# --- Dry run + output YAML (generate manifests without applying) ---
kubectl create deployment my-dep --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl run my-pod --image=nginx --dry-run=client -o yaml > pod.yaml
```

---

## 14. gcloud logging -- Cloud Logging

```bash
# Read log entries
gcloud logging read "resource.type=gce_instance" --limit=10
gcloud logging read "severity>=ERROR" --limit=20
gcloud logging read 'resource.type="gce_instance" AND severity="ERROR"' \
  --freshness=1d --limit=50
gcloud logging read "logName=projects/my-project/logs/my-log" --limit=10
gcloud logging read 'textPayload:"error"' --limit=10 --format=json

# Write a log entry (useful for testing)
gcloud logging write my-log "Test log entry" --severity=INFO

# --- Log Sinks (export logs) ---
# Create a sink to Cloud Storage
gcloud logging sinks create my-gcs-sink \
  storage.googleapis.com/my-log-bucket \
  --log-filter='resource.type="gce_instance"'

# Create a sink to BigQuery
gcloud logging sinks create my-bq-sink \
  bigquery.googleapis.com/projects/my-project/datasets/my_dataset \
  --log-filter='severity>=WARNING'

# Create a sink to Pub/Sub
gcloud logging sinks create my-pubsub-sink \
  pubsub.googleapis.com/projects/my-project/topics/my-topic \
  --log-filter='resource.type="gae_app"'

# List / describe / update / delete sinks
gcloud logging sinks list
gcloud logging sinks describe my-gcs-sink
gcloud logging sinks update my-gcs-sink --log-filter='severity>=ERROR'
gcloud logging sinks delete my-gcs-sink

# --- Log-Based Metrics ---
gcloud logging metrics create my-metric \
  --description="Count of 5xx errors" \
  --log-filter='resource.type="http_load_balancer" AND httpRequest.status>=500'

gcloud logging metrics list
gcloud logging metrics describe my-metric
gcloud logging metrics update my-metric --log-filter='httpRequest.status>=500'
gcloud logging metrics delete my-metric
```

---

## 15. gcloud monitoring -- Cloud Monitoring

```bash
# --- Alerting Policies ---
gcloud monitoring policies list
gcloud monitoring policies describe POLICY_ID
gcloud monitoring policies delete POLICY_ID

# Create an alerting policy from JSON/YAML
gcloud monitoring policies create --policy-from-file=policy.json

# Update a policy
gcloud monitoring policies update POLICY_ID --policy-from-file=updated-policy.json

# Enable / disable a policy
gcloud monitoring policies update POLICY_ID --enabled
gcloud monitoring policies update POLICY_ID --no-enabled

# --- Notification Channels ---
gcloud monitoring channels list
gcloud monitoring channels describe CHANNEL_ID
gcloud monitoring channels create --channel-content-from-file=channel.json
gcloud monitoring channels update CHANNEL_ID --display-name="New Name"
gcloud monitoring channels delete CHANNEL_ID

# --- Dashboards ---
gcloud monitoring dashboards list
gcloud monitoring dashboards describe DASHBOARD_ID
gcloud monitoring dashboards create --config-from-file=dashboard.json
gcloud monitoring dashboards delete DASHBOARD_ID

# --- Uptime Checks ---
gcloud monitoring uptime create --display-name="My Check" --config-from-file=uptime.json
gcloud monitoring uptime list-configs
gcloud monitoring uptime describe CHECK_ID
gcloud monitoring uptime delete CHECK_ID
```

---

## 16. gcloud pubsub -- Pub/Sub

```bash
# --- Topics ---
gcloud pubsub topics create my-topic
gcloud pubsub topics list
gcloud pubsub topics describe my-topic
gcloud pubsub topics delete my-topic

# Publish a message
gcloud pubsub topics publish my-topic --message="Hello, World!"
gcloud pubsub topics publish my-topic --message="data" --attribute=key1=val1,key2=val2

# Get / set IAM on a topic
gcloud pubsub topics get-iam-policy my-topic
gcloud pubsub topics add-iam-policy-binding my-topic \
  --member="serviceAccount:my-sa@my-project.iam.gserviceaccount.com" \
  --role="roles/pubsub.publisher"

# --- Subscriptions ---
# Create a pull subscription
gcloud pubsub subscriptions create my-sub --topic=my-topic

# Create a push subscription
gcloud pubsub subscriptions create my-push-sub \
  --topic=my-topic \
  --push-endpoint=https://my-service.run.app/push

# Create a subscription with message retention and ack deadline
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --ack-deadline=60 \
  --message-retention-duration=7d

# Create a dead-letter subscription
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --dead-letter-topic=my-dead-letter-topic \
  --max-delivery-attempts=5

# Pull messages
gcloud pubsub subscriptions pull my-sub --limit=10 --auto-ack

# List / describe / delete
gcloud pubsub subscriptions list
gcloud pubsub subscriptions describe my-sub
gcloud pubsub subscriptions delete my-sub

# Seek (replay messages)
gcloud pubsub subscriptions seek my-sub --time="2026-01-01T00:00:00Z"

# Create and seek to a snapshot
gcloud pubsub snapshots create my-snapshot --subscription=my-sub
gcloud pubsub subscriptions seek my-sub --snapshot=my-snapshot
```

---

## 17. gcloud sql -- Cloud SQL

```bash
# --- Instances ---
# Create a MySQL instance
gcloud sql instances create my-mysql \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --root-password=my-root-password \
  --storage-size=100GB \
  --storage-type=SSD \
  --availability-type=REGIONAL \
  --backup-start-time=04:00

# Create a PostgreSQL instance
gcloud sql instances create my-postgres \
  --database-version=POSTGRES_15 \
  --tier=db-custom-2-8192 \
  --region=us-central1 \
  --availability-type=ZONAL

# List / describe / delete
gcloud sql instances list
gcloud sql instances describe my-mysql
gcloud sql instances delete my-mysql

# Restart / patch
gcloud sql instances restart my-mysql
gcloud sql instances patch my-mysql --tier=db-n1-standard-4
gcloud sql instances patch my-mysql --activation-policy=NEVER      # Stop the instance
gcloud sql instances patch my-mysql --activation-policy=ALWAYS     # Start the instance

# Set authorized networks (allow external connections)
gcloud sql instances patch my-mysql \
  --authorized-networks=203.0.113.0/24

# Create a read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-mysql \
  --region=us-central1

# Promote a replica to standalone
gcloud sql instances promote-replica my-replica

# Failover (HA instance)
gcloud sql instances failover my-mysql

# --- Databases ---
gcloud sql databases create my-db --instance=my-mysql
gcloud sql databases list --instance=my-mysql
gcloud sql databases describe my-db --instance=my-mysql
gcloud sql databases delete my-db --instance=my-mysql

# --- Users ---
gcloud sql users create my-user --instance=my-mysql --password=my-password
gcloud sql users list --instance=my-mysql
gcloud sql users set-password my-user --instance=my-mysql --password=new-password
gcloud sql users delete my-user --instance=my-mysql

# --- Backups ---
gcloud sql backups create --instance=my-mysql
gcloud sql backups list --instance=my-mysql
gcloud sql backups describe BACKUP_ID --instance=my-mysql
gcloud sql backups delete BACKUP_ID --instance=my-mysql
gcloud sql backups restore BACKUP_ID --restore-instance=my-mysql

# --- Connect ---
# Connect via Cloud SQL Auth Proxy (interactive)
gcloud sql connect my-mysql --user=root

# Export to Cloud Storage
gcloud sql export sql my-mysql gs://my-bucket/backup.sql --database=my-db
gcloud sql export csv my-mysql gs://my-bucket/data.csv --database=my-db --query="SELECT * FROM users"

# Import from Cloud Storage
gcloud sql import sql my-mysql gs://my-bucket/backup.sql --database=my-db
gcloud sql import csv my-mysql gs://my-bucket/data.csv --database=my-db --table=users
```

---

## 18. gcloud dns -- Cloud DNS

```bash
# --- Managed Zones ---
# Create a public DNS zone
gcloud dns managed-zones create my-zone \
  --dns-name="example.com." \
  --description="My DNS zone" \
  --visibility=public

# Create a private DNS zone
gcloud dns managed-zones create my-private-zone \
  --dns-name="internal.example.com." \
  --description="Private zone" \
  --visibility=private \
  --networks=my-vpc

# List / describe / delete
gcloud dns managed-zones list
gcloud dns managed-zones describe my-zone
gcloud dns managed-zones delete my-zone

# --- Record Sets ---
# Start a transaction (batch changes)
gcloud dns record-sets transaction start --zone=my-zone

# Add an A record
gcloud dns record-sets transaction add "203.0.113.1" \
  --name="www.example.com." \
  --ttl=300 \
  --type=A \
  --zone=my-zone

# Add a CNAME record
gcloud dns record-sets transaction add "example.com." \
  --name="blog.example.com." \
  --ttl=300 \
  --type=CNAME \
  --zone=my-zone

# Remove a record
gcloud dns record-sets transaction remove "203.0.113.1" \
  --name="www.example.com." \
  --ttl=300 \
  --type=A \
  --zone=my-zone

# Execute the transaction
gcloud dns record-sets transaction execute --zone=my-zone

# Abort a transaction
gcloud dns record-sets transaction abort --zone=my-zone

# List record sets
gcloud dns record-sets list --zone=my-zone

# Describe a specific record set
gcloud dns record-sets describe www.example.com. --type=A --zone=my-zone
```

---

## 19. gcloud billing -- Billing Management

```bash
# --- Billing Accounts ---
gcloud billing accounts list
gcloud billing accounts describe BILLING_ACCOUNT_ID
gcloud billing accounts get-iam-policy BILLING_ACCOUNT_ID

gcloud billing accounts add-iam-policy-binding BILLING_ACCOUNT_ID \
  --member="user:finance@example.com" \
  --role="roles/billing.viewer"

# --- Link / unlink projects to billing ---
gcloud billing projects link my-project --billing-account=BILLING_ACCOUNT_ID
gcloud billing projects unlink my-project
gcloud billing projects describe my-project     # Show billing info for a project

# --- Budgets ---
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Monthly Budget" \
  --budget-amount=1000.00USD \
  --threshold-rule=percent=0.5 \
  --threshold-rule=percent=0.9 \
  --threshold-rule=percent=1.0

gcloud billing budgets list --billing-account=BILLING_ACCOUNT_ID
gcloud billing budgets describe BUDGET_ID --billing-account=BILLING_ACCOUNT_ID
gcloud billing budgets update BUDGET_ID --billing-account=BILLING_ACCOUNT_ID \
  --budget-amount=2000.00USD
gcloud billing budgets delete BUDGET_ID --billing-account=BILLING_ACCOUNT_ID
```

---

## 20. Terraform Essentials

```bash
# --- Core Workflow ---
terraform init                                  # Initialize working directory, download providers
terraform plan                                  # Preview changes without applying
terraform plan -out=tfplan                      # Save plan to a file
terraform apply                                 # Apply changes (prompts for confirmation)
terraform apply tfplan                          # Apply a saved plan (no prompt)
terraform apply -auto-approve                   # Apply without confirmation prompt
terraform destroy                               # Destroy all managed resources
terraform destroy -auto-approve                 # Destroy without confirmation prompt

# --- State Management ---
terraform state list                            # List all resources in state
terraform state show google_compute_instance.my_vm   # Show details of a resource
terraform state rm google_compute_instance.my_vm     # Remove a resource from state (does not delete)
terraform state mv google_compute_instance.old google_compute_instance.new   # Rename in state
terraform state pull                            # Download remote state to stdout

# --- Import existing resources ---
terraform import google_compute_instance.my_vm projects/my-project/zones/us-central1-a/instances/my-vm

# --- Workspaces (multiple environments) ---
terraform workspace list
terraform workspace new staging
terraform workspace select production
terraform workspace show                        # Show current workspace
terraform workspace delete staging

# --- Other useful commands ---
terraform fmt                                   # Format .tf files
terraform fmt -check                            # Check formatting (CI-friendly)
terraform validate                              # Validate syntax and configuration
terraform output                                # Show all outputs
terraform output my_output                      # Show a specific output
terraform refresh                               # Sync state with real infrastructure
terraform show                                  # Show current state or a saved plan
terraform graph | dot -Tpng > graph.png         # Generate dependency graph
terraform providers                             # List providers used in config
terraform version                               # Show Terraform version
terraform taint google_compute_instance.my_vm   # Mark resource for recreation (deprecated, use -replace)
terraform apply -replace=google_compute_instance.my_vm   # Recreate a specific resource

# --- GCP Backend Configuration (store state in GCS) ---
# In your main.tf or backend.tf:
# terraform {
#   backend "gcs" {
#     bucket = "my-tf-state-bucket"
#     prefix = "terraform/state"
#   }
# }
```

---

## 21. gcloud asset -- Cloud Asset Inventory

```bash
# Search all resources in a project
gcloud asset search-all-resources --scope=projects/PROJECT_ID

# Search all resources in an organization
gcloud asset search-all-resources --scope=organizations/ORG_ID

# Search resources by type
gcloud asset search-all-resources --scope=projects/PROJECT_ID \
  --asset-types=compute.googleapis.com/Instance

# Search IAM policies in a project
gcloud asset search-all-iam-policies --scope=projects/PROJECT_ID

# Search IAM policies in an organization
gcloud asset search-all-iam-policies --scope=organizations/ORG_ID

# Search for who has a specific role
gcloud asset search-all-iam-policies --scope=projects/PROJECT_ID \
  --query="policy:roles/editor"

# Export all assets to Cloud Storage
gcloud asset export --project=PROJECT_ID \
  --output-path=gs://BUCKET/FILE.json \
  --content-type=resource

# Export IAM policies to Cloud Storage
gcloud asset export --project=PROJECT_ID \
  --output-path=gs://BUCKET/iam-policies.json \
  --content-type=iam-policy

# Analyze IAM policy (who has access to what)
gcloud asset analyze-iam-policy --organization=ORG_ID \
  --full-resource-name="//cloudresourcemanager.googleapis.com/projects/PROJECT_ID"

# Analyze IAM policy for a specific identity
gcloud asset analyze-iam-policy --organization=ORG_ID \
  --identity="user:alice@example.com"
```

---

## 22. Hyperdisk Commands

```bash
# Create a Hyperdisk Extreme disk with provisioned IOPS and throughput
gcloud compute disks create my-hyperdisk-extreme \
  --zone=us-central1-a \
  --type=hyperdisk-extreme \
  --provisioned-iops=100000 \
  --provisioned-throughput=2400 \
  --size=1000GB

# Create a Hyperdisk Balanced disk
gcloud compute disks create my-hyperdisk-balanced \
  --zone=us-central1-a \
  --type=hyperdisk-balanced \
  --size=500GB

# Update IOPS and throughput on an existing Hyperdisk Extreme
gcloud compute disks update my-hyperdisk-extreme \
  --zone=us-central1-a \
  --provisioned-iops=150000 \
  --provisioned-throughput=3000

# Attach Hyperdisk to a VM
gcloud compute instances attach-disk my-vm \
  --disk=my-hyperdisk-extreme \
  --zone=us-central1-a
```

---

## 23. Cloud NGFW / Network Firewall Policies

```bash
# Create a global network firewall policy
gcloud compute network-firewall-policies create my-policy --global

# Create a regional network firewall policy
gcloud compute network-firewall-policies create my-regional-policy \
  --region=us-central1

# List firewall policies
gcloud compute network-firewall-policies list --global
gcloud compute network-firewall-policies list --region=us-central1

# Describe a firewall policy
gcloud compute network-firewall-policies describe my-policy --global

# Create a firewall rule in the policy
gcloud compute network-firewall-policies rules create 1000 \
  --firewall-policy=my-policy \
  --global \
  --action=allow \
  --direction=INGRESS \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:443 \
  --description="Allow HTTPS from internet"

# Create a deny rule
gcloud compute network-firewall-policies rules create 2000 \
  --firewall-policy=my-policy \
  --global \
  --action=deny \
  --direction=EGRESS \
  --dest-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:22

# List rules in a policy
gcloud compute network-firewall-policies rules describe 1000 \
  --firewall-policy=my-policy \
  --global

# Update a rule
gcloud compute network-firewall-policies rules update 1000 \
  --firewall-policy=my-policy \
  --global \
  --src-ip-ranges=10.0.0.0/8

# Delete a rule
gcloud compute network-firewall-policies rules delete 1000 \
  --firewall-policy=my-policy \
  --global

# Associate a firewall policy with a VPC network
gcloud compute network-firewall-policies associations create \
  --firewall-policy=my-policy \
  --network=my-vpc \
  --global-firewall-policy

# List associations
gcloud compute network-firewall-policies associations list \
  --firewall-policy=my-policy \
  --global

# Delete an association
gcloud compute network-firewall-policies associations delete \
  --name=ASSOCIATION_NAME \
  --firewall-policy=my-policy \
  --global

# Delete a firewall policy
gcloud compute network-firewall-policies delete my-policy --global
```

---

## 24. Resource Manager Tags (Secure Tags)

```bash
# Create a tag key at the organization level
gcloud resource-manager tags keys create my-tag-key \
  --parent=organizations/ORG_ID \
  --purpose=GCE_FIREWALL \
  --purpose-data=network=my-vpc

# Create a tag key without firewall purpose
gcloud resource-manager tags keys create environment \
  --parent=organizations/ORG_ID

# List tag keys
gcloud resource-manager tags keys list --parent=organizations/ORG_ID

# Describe a tag key
gcloud resource-manager tags keys describe TAG_KEY_ID

# Create a tag value under a tag key
gcloud resource-manager tags values create production \
  --parent=TAG_KEY_ID

gcloud resource-manager tags values create staging \
  --parent=TAG_KEY_ID

# List tag values
gcloud resource-manager tags values list --parent=TAG_KEY_ID

# Bind a tag to a resource (e.g., VM instance)
gcloud resource-manager tags bindings create \
  --tag-value=TAG_VALUE_ID \
  --parent=//compute.googleapis.com/projects/PROJECT_ID/zones/ZONE/instances/VM_NAME

# Bind a tag to a project
gcloud resource-manager tags bindings create \
  --tag-value=TAG_VALUE_ID \
  --parent=//cloudresourcemanager.googleapis.com/projects/PROJECT_ID

# List tag bindings on a resource
gcloud resource-manager tags bindings list \
  --parent=//compute.googleapis.com/projects/PROJECT_ID/zones/ZONE/instances/VM_NAME

# Delete a tag binding
gcloud resource-manager tags bindings delete \
  --tag-value=TAG_VALUE_ID \
  --parent=//compute.googleapis.com/projects/PROJECT_ID/zones/ZONE/instances/VM_NAME

# Delete a tag value
gcloud resource-manager tags values delete TAG_VALUE_ID

# Delete a tag key
gcloud resource-manager tags keys delete TAG_KEY_ID
```

---

## 25. Bigtable Backup Commands

```bash
# Create a backup of a Bigtable table
gcloud bigtable backups create my-backup \
  --instance=my-instance \
  --cluster=my-cluster \
  --table=my-table \
  --retention-period=30d

# Create a backup with expiration time
gcloud bigtable backups create my-backup \
  --instance=my-instance \
  --cluster=my-cluster \
  --table=my-table \
  --expiration-date=2026-12-31T00:00:00Z

# List backups in a cluster
gcloud bigtable backups list \
  --instance=my-instance \
  --cluster=my-cluster

# Describe a backup
gcloud bigtable backups describe my-backup \
  --instance=my-instance \
  --cluster=my-cluster

# Update backup expiration
gcloud bigtable backups update my-backup \
  --instance=my-instance \
  --cluster=my-cluster \
  --retention-period=60d

# Restore a table from a backup
gcloud bigtable instances tables restore \
  --source-instance=my-instance \
  --source-cluster=my-cluster \
  --source-backup=my-backup \
  --destination=restored-table \
  --destination-instance=my-instance

# Restore to a different instance
gcloud bigtable instances tables restore \
  --source-instance=source-instance \
  --source-cluster=source-cluster \
  --source-backup=my-backup \
  --destination=restored-table \
  --destination-instance=destination-instance

# Delete a backup
gcloud bigtable backups delete my-backup \
  --instance=my-instance \
  --cluster=my-cluster
```

---

## 26. Spanner Backup Commands

```bash
# Create a backup of a Spanner database
gcloud spanner backups create my-backup \
  --instance=my-instance \
  --database=my-database \
  --retention-period=7d

# Create a backup with expiration time
gcloud spanner backups create my-backup \
  --instance=my-instance \
  --database=my-database \
  --expiration-date=2026-06-01T00:00:00Z

# List backups in an instance
gcloud spanner backups list --instance=my-instance

# Describe a backup
gcloud spanner backups describe my-backup --instance=my-instance

# Update backup expiration
gcloud spanner backups update my-backup \
  --instance=my-instance \
  --retention-period=14d

# Restore a database from a backup
gcloud spanner databases restore \
  --instance=my-instance \
  --source-backup=my-backup \
  --source-instance=my-instance \
  --destination-database=restored-db

# Restore to a different instance
gcloud spanner databases restore \
  --instance=destination-instance \
  --source-backup=my-backup \
  --source-instance=source-instance \
  --destination-database=new-db

# Delete a backup
gcloud spanner backups delete my-backup --instance=my-instance
```

---

## 27. AlloyDB Commands

```bash
# Create an AlloyDB cluster
gcloud alloydb clusters create my-cluster \
  --region=us-central1 \
  --password=MY_PASSWORD \
  --network=projects/PROJECT_ID/global/networks/my-vpc

# List clusters
gcloud alloydb clusters list --region=us-central1

# Describe a cluster
gcloud alloydb clusters describe my-cluster --region=us-central1

# Create a primary instance in the cluster
gcloud alloydb instances create my-primary \
  --cluster=my-cluster \
  --region=us-central1 \
  --instance-type=PRIMARY \
  --cpu-count=2

# Create a read pool instance
gcloud alloydb instances create my-read-pool \
  --cluster=my-cluster \
  --region=us-central1 \
  --instance-type=READ_POOL \
  --read-pool-node-count=2 \
  --cpu-count=2

# List instances in a cluster
gcloud alloydb instances list --cluster=my-cluster --region=us-central1

# Describe an instance
gcloud alloydb instances describe my-primary \
  --cluster=my-cluster \
  --region=us-central1

# Update an instance (e.g., resize)
gcloud alloydb instances update my-primary \
  --cluster=my-cluster \
  --region=us-central1 \
  --cpu-count=4

# Restart an instance
gcloud alloydb instances restart my-primary \
  --cluster=my-cluster \
  --region=us-central1

# Create a backup
gcloud alloydb backups create my-backup \
  --cluster=my-cluster \
  --region=us-central1

# List backups
gcloud alloydb backups list --region=us-central1

# Describe a backup
gcloud alloydb backups describe my-backup --region=us-central1

# Restore from a backup (creates a new cluster)
gcloud alloydb clusters restore my-restored-cluster \
  --region=us-central1 \
  --backup=my-backup \
  --network=projects/PROJECT_ID/global/networks/my-vpc

# Delete an instance
gcloud alloydb instances delete my-primary \
  --cluster=my-cluster \
  --region=us-central1

# Delete a cluster (must delete all instances first)
gcloud alloydb clusters delete my-cluster --region=us-central1

# Delete a backup
gcloud alloydb backups delete my-backup --region=us-central1
```

---

## 28. Database Center

**Note:** Database Center is a **Console-only** feature. There are no gcloud commands for Database Center. Access it via the Cloud Console:

```
Cloud Console > Database Center
```

Database Center provides a unified view of all databases in your project (Cloud SQL, Spanner, AlloyDB, Bigtable, Firestore) with insights, recommendations, and monitoring dashboards.

---

## 29. Custom Static Routes

```bash
# Create a custom static route with next-hop as an instance (network appliance)
gcloud compute routes create my-route \
  --network=my-vpc \
  --destination-range=10.0.0.0/8 \
  --next-hop-instance=appliance-vm \
  --next-hop-instance-zone=us-central1-a \
  --priority=100

# Create a route with next-hop as an IP address
gcloud compute routes create my-route \
  --network=my-vpc \
  --destination-range=192.168.0.0/16 \
  --next-hop-address=10.1.0.10 \
  --priority=100

# Create a route with next-hop as a VPN tunnel
gcloud compute routes create vpn-route \
  --network=my-vpc \
  --destination-range=172.16.0.0/12 \
  --next-hop-vpn-tunnel=my-vpn-tunnel \
  --next-hop-vpn-tunnel-region=us-central1

# Create a route with next-hop as the default internet gateway
gcloud compute routes create internet-route \
  --network=my-vpc \
  --destination-range=0.0.0.0/0 \
  --next-hop-gateway=default-internet-gateway

# Create a route with next-hop as an internal load balancer
gcloud compute routes create ilb-route \
  --network=my-vpc \
  --destination-range=10.20.0.0/16 \
  --next-hop-ilb=my-ilb \
  --next-hop-ilb-region=us-central1

# List all routes
gcloud compute routes list

# List routes in a specific VPC
gcloud compute routes list --filter="network:my-vpc"

# List routes by destination
gcloud compute routes list --filter="destRange:10.0.0.0/8"

# Describe a route
gcloud compute routes describe my-route

# Delete a route
gcloud compute routes delete my-route
```

---

## 30. Active Assist / Recommender Commands

```bash
# List all available recommenders
gcloud recommender recommenders list

# List machine type recommendations for VMs in a zone
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender

# List idle VM recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.IdleResourceRecommender

# List IAM policy recommendations (remove unused permissions)
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.iam.policy.Recommender

# List unattended project recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.resourcemanager.projectUtilization.Recommender

# Describe a specific recommendation
gcloud recommender recommendations describe RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender

# Mark a recommendation as claimed (in progress)
gcloud recommender recommendations mark-claimed RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --etag=ETAG

# Mark a recommendation as succeeded (applied)
gcloud recommender recommendations mark-succeeded RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --etag=ETAG

# Mark a recommendation as failed
gcloud recommender recommendations mark-failed RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --location=us-central1-a \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --etag=ETAG

# List IAM insights (who has access but hasn't used it)
gcloud recommender insights list \
  --project=PROJECT_ID \
  --location=global \
  --insight-type=google.iam.policy.Insight

# Describe an insight
gcloud recommender insights describe INSIGHT_ID \
  --project=PROJECT_ID \
  --location=global \
  --insight-type=google.iam.policy.Insight
```

---

## Quick Reference: Common Patterns

### Format & Filter Examples

```bash
# Output as JSON
gcloud compute instances list --format=json

# Output as table with specific columns
gcloud compute instances list --format="table(name, zone, status, machineType)"

# Output only values (for scripting)
gcloud compute instances list --format="value(name)"

# Filter by field value
gcloud compute instances list --filter="status=RUNNING"
gcloud compute instances list --filter="zone:us-central1-a"
gcloud compute instances list --filter="name~'^web-.*'"       # Regex match
gcloud compute instances list --filter="labels.env=prod"

# Combine filter + format
gcloud compute instances list \
  --filter="status=RUNNING AND zone:us-central1-a" \
  --format="table(name, networkInterfaces[0].accessConfigs[0].natIP)"

# Sort and limit
gcloud compute instances list --sort-by=name --limit=5
gcloud compute instances list --sort-by=~creationTimestamp    # ~ for descending
```

### Impersonation

```bash
# Run any command as a service account (no key file needed)
gcloud compute instances list \
  --impersonate-service-account=my-sa@my-project.iam.gserviceaccount.com

# Set impersonation globally in config
gcloud config set auth/impersonate_service_account my-sa@my-project.iam.gserviceaccount.com
gcloud config unset auth/impersonate_service_account
```

### Export & Scripting Patterns

```bash
# Get project ID into a variable
PROJECT_ID=$(gcloud config get-value project)

# Get current account
ACCOUNT=$(gcloud config get-value account)

# List all zones in a region
gcloud compute zones list --filter="region:us-central1" --format="value(name)"

# Delete all instances with a specific label
gcloud compute instances list --filter="labels.env=test" --format="value(name, zone)" | \
  while read NAME ZONE; do
    gcloud compute instances delete "$NAME" --zone="$ZONE" --quiet
  done

# Get the external IP of an instance
gcloud compute instances describe my-vm --zone=us-central1-a \
  --format="value(networkInterfaces[0].accessConfigs[0].natIP)"

# Get cluster endpoint
gcloud container clusters describe my-cluster --zone=us-central1-a \
  --format="value(endpoint)"
```
