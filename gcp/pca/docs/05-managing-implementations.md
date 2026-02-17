# Section 5: Managing Implementation (~12.5% of Exam)

> **Quick context:** This section tests your ability to advise teams on implementation, use GCP programmatically, and manage infrastructure as code. Expect 6-8 questions. Heavy emphasis on Terraform, API management choices, and migration tooling.

---

## 5.1 Advising Development and Operation Teams

### Application and Infrastructure Deployment

#### Immutable Infrastructure Patterns

Immutable infrastructure means you never patch running instances -- you replace them with new ones built from a known-good image. This is the preferred pattern on GCP for production workloads.

**Building golden images with Packer:**

```bash
# Packer builds a Compute Engine image from a source image + provisioners
# Example: Build a custom image with your app pre-installed
packer build gce-image.pkr.hcl

# List custom images in the project
gcloud compute images list --no-standard-images

# Create an image from a running disk (manual approach)
gcloud compute images create my-golden-image \
  --source-disk=my-instance-disk \
  --source-disk-zone=us-central1-a \
  --family=my-app-family

# Use an image family (always resolves to latest non-deprecated image)
gcloud compute instances create my-vm \
  --image-family=my-app-family \
  --image-project=my-project
```

**Deployment patterns on GCP:**

| Pattern | How It Works | GCP Service |
|---------|-------------|-------------|
| **Rolling update** | Replace instances in batches | MIG `--update-policy=proactive` |
| **Blue/green** | Two identical environments, switch traffic | Cloud Load Balancing + MIGs or GKE |
| **Canary** | Route small % of traffic to new version | GKE traffic splitting, Cloud Run revisions |
| **Recreate** | Tear down all, deploy new | MIG with `maxUnavailable=100%` |

```bash
# Rolling update on a MIG
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=new-template \
  --max-surge=3 \
  --max-unavailable=0 \
  --zone=us-central1-a

# Canary update (20% of instances get new template)
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=stable-template \
  --canary-version=template=canary-template,target-size=20% \
  --zone=us-central1-a
```

#### Progressive Delivery on GKE and Cloud Run

```bash
# Cloud Run: Split traffic between revisions
gcloud run services update-traffic my-service \
  --to-revisions=my-service-00002=10,my-service-00001=90 \
  --region=us-central1

# Cloud Run: Route all traffic to latest
gcloud run services update-traffic my-service \
  --to-latest --region=us-central1

# GKE: Use Gateway API or Istio for traffic splitting
# With Istio VirtualService, you can do weighted routing:
# - route 90% to v1, 10% to v2
# - shift traffic gradually
```

#### Configuration Management

**Config Controller** -- managed Kubernetes cluster that runs Config Connector and Policy Controller. You declare GCP resources as Kubernetes manifests, and Config Controller creates/manages them.

```yaml
# Example: Declare a Cloud Storage bucket via Config Connector
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-config-managed-bucket
  namespace: config-control
spec:
  location: US
  storageClass: STANDARD
  uniformBucketLevelAccess: true
```

**Config Sync** -- GitOps tool for GKE. It syncs Kubernetes manifests from a Git repo to your clusters. Pairs with Policy Controller for guardrails.

```bash
# Enable Config Sync on a GKE fleet membership
gcloud container fleet config-management apply \
  --membership=my-cluster \
  --config=config-sync-config.yaml

# Check sync status
gcloud container fleet config-management status
```

> **Exam tips:**
> - Immutable infrastructure = build new, don't patch. Prefer image families for auto-resolution to latest.
> - Config Controller = managed control plane for declarative GCP resource management (Kubernetes-style).
> - Config Sync = GitOps for GKE clusters. It pulls from Git, not pushes.
> - Rolling updates with `maxUnavailable=0` ensure zero downtime.
> - Cloud Run traffic splitting is revision-based and instant (no new deployment needed).

**Docs:** [MIG rolling updates](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups) | [Cloud Run traffic management](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration) | [Config Controller](https://cloud.google.com/anthos-config-management/docs/concepts/config-controller-overview) | [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)

---

### API Management Best Practices

GCP offers three API management products. The exam tests your ability to choose the right one.

#### Apigee (Full API Management Platform)

Apigee is Google Cloud's enterprise API management platform. It sits in front of your backend services and provides:

- **API gateway** (reverse proxy) with traffic management
- **Rate limiting and quotas** (spike arrest, quota policies)
- **Analytics dashboard** (latency, error rates, developer adoption)
- **Developer portal** (self-service API key registration, documentation)
- **API monetization** (charge developers for API usage)
- **Security** (OAuth 2.0, API keys, JWT validation, threat protection)
- **Mediation** (transform requests/responses, JSON-to-XML, etc.)

```bash
# Create an Apigee organization (requires an existing GCP project)
gcloud apigee organizations provision \
  --authorized-network=default \
  --runtime-location=us-central1 \
  --analytics-region=us-central1 \
  --project=my-project

# List Apigee environments
gcloud apigee environments list --organization=my-org

# Deploy an API proxy
gcloud apigee apis deploy \
  --api=my-api \
  --environment=prod \
  --organization=my-org \
  --revision=3

# List deployed API proxies
gcloud apigee deployments list \
  --environment=prod \
  --organization=my-org
```

**Apigee tiers:**

| Tier | Use Case |
|------|----------|
| **Apigee X** | Full enterprise, VPC-peered, private networking |
| **Apigee hybrid** | Control plane on GCP, runtime on-prem or other cloud |
| **Apigee Integrated** | Pay-as-you-go, simpler setup, good for getting started |

#### API Gateway (Serverless, Simpler)

API Gateway is a fully managed, serverless gateway for Cloud Functions, Cloud Run, and App Engine backends. Simpler than Apigee, no developer portal or monetization.

```bash
# Create an API config from an OpenAPI spec
gcloud api-gateway api-configs create my-config \
  --api=my-api \
  --openapi-spec=openapi.yaml \
  --project=my-project

# Create a gateway
gcloud api-gateway gateways create my-gateway \
  --api=my-api \
  --api-config=my-config \
  --location=us-central1 \
  --project=my-project

# Describe a gateway (shows the URL)
gcloud api-gateway gateways describe my-gateway \
  --location=us-central1
```

#### Cloud Endpoints

Cloud Endpoints uses ESP (Extensible Service Proxy) or ESPv2 as a sidecar container. Best for gRPC APIs and microservices that need OpenAPI/gRPC specification validation.

```bash
# Deploy an Endpoints service configuration
gcloud endpoints services deploy openapi-spec.yaml

# Deploy with a gRPC service definition
gcloud endpoints services deploy api_descriptor.pb service-config.yaml

# Check service status
gcloud endpoints services describe my-api.endpoints.my-project.cloud.goog
```

#### Comparison Table: Apigee vs API Gateway vs Cloud Endpoints

| Feature | Apigee | API Gateway | Cloud Endpoints |
|---------|--------|-------------|-----------------|
| **Type** | Full API management platform | Managed serverless gateway | Proxy (sidecar/ESP) |
| **Best for** | Enterprise APIs, external developer ecosystems | Simple serverless backends | gRPC/OpenAPI microservices |
| **Developer portal** | Yes (built-in) | No | No |
| **Monetization** | Yes | No | No |
| **Rate limiting** | Advanced (spike arrest, quotas, per-developer) | Basic (per-gateway) | Basic |
| **Analytics** | Rich dashboard, custom reports | Cloud Logging/Monitoring | Cloud Logging/Monitoring |
| **Protocol** | REST, SOAP, GraphQL | REST (OpenAPI) | REST (OpenAPI), gRPC |
| **Backend** | Any HTTP backend | Cloud Functions, Cloud Run, App Engine | Any (deployed as sidecar) |
| **Pricing** | Most expensive (subscription) | Pay-per-call (cheap) | Free (you pay for ESP compute) |
| **Hybrid/multi-cloud** | Yes (Apigee hybrid) | No | No |
| **API versioning** | Built-in revision management | Via API configs | Via service configs |
| **Transformation** | JSON/XML mediation, payload manipulation | No | No |

**Decision shortcut:**
- Need developer portal, monetization, or advanced policies? --> **Apigee**
- Simple serverless API with OpenAPI spec? --> **API Gateway**
- gRPC or need sidecar proxy alongside your service? --> **Cloud Endpoints**
- External-facing partner/third-party API program? --> **Apigee** (always)

#### API Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URI versioning** | `/v1/users`, `/v2/users` | Simple, explicit | URL changes break clients |
| **Header versioning** | `Accept: application/vnd.myapi.v2+json` | Clean URLs | Hidden, harder to test |
| **Query param** | `/users?version=2` | Easy to implement | Clutters query string |

> **Exam tips:**
> - **Apigee** = enterprise, external developers, monetization, developer portal. If the question mentions "third-party developers" or "partner API program," pick Apigee.
> - **API Gateway** = simplest option for serverless (Cloud Functions / Cloud Run) backends. Pay-per-call.
> - **Cloud Endpoints** = gRPC support, sidecar proxy (ESP/ESPv2). Free service -- you pay for the compute running the proxy.
> - If the question mentions "SOAP" or "XML transformation," Apigee is the only option with mediation capabilities.
> - Apigee hybrid = runtime on-prem, control plane on GCP. Used for regulated industries or multi-cloud.

**Docs:** [Apigee](https://cloud.google.com/apigee/docs) | [API Gateway](https://cloud.google.com/api-gateway/docs) | [Cloud Endpoints](https://cloud.google.com/endpoints/docs) | [Choosing an API management product](https://cloud.google.com/api-gateway/docs/choose-api-management)

---

### Testing Frameworks

#### Load Testing Approaches and Tools

```bash
# Cloud-native load testing with Locust on GKE
# Deploy a Locust master + workers on GKE for distributed load testing
kubectl apply -f locust-master.yaml
kubectl apply -f locust-worker.yaml

# Or use Cloud Build to run load tests as part of CI/CD
# cloudbuild.yaml step:
# - name: 'gcr.io/cloud-builders/docker'
#   args: ['run', 'locustio/locust', '-f', 'locustfile.py', '--headless', '-u', '1000', '-r', '100']
```

**Common load testing tools on GCP:**

| Tool | Type | Where to Run |
|------|------|-------------|
| **Locust** | Python-based, scriptable | GKE, Compute Engine |
| **JMeter** | Java-based, GUI + CLI | Compute Engine, Cloud Build |
| **k6** | JavaScript-based, modern | Cloud Build, Compute Engine |
| **Cloud Load Testing** (deprecated) | Was GCP-native | Use Locust or k6 instead |
| **Fortio** | Istio's load testing tool | GKE (especially with Istio) |

#### Testing in CI/CD Pipelines (Cloud Build)

```yaml
# cloudbuild.yaml -- Example multi-stage test pipeline
steps:
  # Unit tests
  - name: 'python:3.11'
    entrypoint: 'bash'
    args:
      - '-c'
      - 'pip install -r requirements.txt && pytest tests/unit/ -v'

  # Integration tests with emulators
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        gcloud beta emulators pubsub start --host-port=0.0.0.0:8085 &
        sleep 5
        PUBSUB_EMULATOR_HOST=localhost:8085 pytest tests/integration/ -v

  # Build container
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA', '.']

  # Push container
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA']

  # Deploy to staging
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'deploy', 'my-app-staging', '--image', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA', '--region', 'us-central1']

  # Smoke tests against staging
  - name: 'curlimages/curl'
    args: ['-f', 'https://my-app-staging-xyz.a.run.app/health']
```

#### Contract Testing for APIs

Contract testing validates that an API provider meets the expectations of its consumers without full end-to-end tests. Tools:

- **Pact** -- consumer-driven contract testing. Consumer defines expectations, provider verifies.
- **Protovalidate** -- for gRPC, validates protobuf messages against constraints.
- **OpenAPI schema validation** -- validate requests/responses against your OpenAPI spec.

> **Exam tips:**
> - Load testing should run against a staging environment, not production (unless you're doing controlled chaos engineering).
> - Cloud Build can run any container -- use it to run tests with emulators before deploying.
> - Contract testing is the exam-relevant answer when asked "how to test microservice APIs independently."
> - Canary deployments + automated rollback = the GCP-recommended progressive delivery approach.

**Docs:** [Cloud Build](https://cloud.google.com/build/docs) | [Cloud Build triggers](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers) | [Testing overview](https://cloud.google.com/architecture/devops/devops-tech-test-automation)

---

### Data and System Migration and Management Tooling

This is a high-frequency exam topic. Know which tool to use for each migration scenario.

#### Database Migration Service (DMS)

DMS provides continuous data replication from source databases to Cloud SQL, AlloyDB, or Cloud Spanner. Supports:

- **Sources:** MySQL, PostgreSQL, SQL Server, Oracle (on-prem or other cloud)
- **Destinations:** Cloud SQL, AlloyDB, Cloud Spanner
- **Method:** Continuous replication (CDC) for minimal downtime

```bash
# Create a connection profile for the source
gcloud database-migration connection-profiles create mysql my-source-profile \
  --region=us-central1 \
  --host=10.0.0.5 \
  --port=3306 \
  --username=replication_user \
  --password=my-password

# Create a connection profile for the destination (Cloud SQL)
gcloud database-migration connection-profiles create cloudsql my-dest-profile \
  --region=us-central1 \
  --cloud-sql-id=my-project:us-central1:my-cloudsql-instance

# Create a migration job
gcloud database-migration migration-jobs create my-migration \
  --region=us-central1 \
  --type=CONTINUOUS \
  --source=my-source-profile \
  --destination=my-dest-profile

# Start the migration
gcloud database-migration migration-jobs start my-migration \
  --region=us-central1

# Promote the destination (cutover -- makes destination primary)
gcloud database-migration migration-jobs promote my-migration \
  --region=us-central1
```

#### Migrate to Containers (Migrate for Anthos)

Converts VM-based workloads into containers running on GKE. Useful for modernizing legacy apps without rewriting.

```bash
# Install Migrate to Containers on a processing cluster
gcloud container clusters get-credentials my-processing-cluster --zone=us-central1-a
migctl setup install

# Add a migration source (e.g., vSphere, AWS, Azure, or Compute Engine)
migctl source create my-ce-source --type=compute-engine

# Generate a migration plan
migctl migration create my-migration --source=my-ce-source --vm-id=my-vm

# Review and customize the migration plan
migctl migration get-plan my-migration > plan.yaml
# Edit plan.yaml (adjust container config, data volumes, etc.)
migctl migration update my-migration --plan-file=plan.yaml

# Execute the migration (generates container artifacts)
migctl migration generate-artifacts my-migration

# Deploy generated artifacts to target GKE cluster
kubectl apply -f deployment_spec.yaml
```

#### Migrate to VMs (Migrate for Compute Engine)

Migrate VMs from on-premises (VMware, AWS, Azure) to Compute Engine with minimal downtime. Uses continuous replication.

```bash
# Migrate to VMs uses the Google Cloud Console or the Migrate connector
# Key concepts:
# - Source: VMware vSphere, AWS EC2, Azure VMs, physical servers
# - Replication: Continuous block-level replication
# - Test clone: Create a test VM without disrupting the source
# - Cutover: Final migration, source VM is shut down
```

#### Migration Center (Assessment and Planning)

Migration Center is the assessment and planning hub. It discovers your on-prem environment, assesses workloads, and recommends target GCP services.

```bash
# Migration Center key components:
# - Discovery Client: Agent-based or agentless discovery of on-prem assets
# - StratoZone: Performance data collection for right-sizing
# - Fit Assessment: Maps workloads to GCP targets (Compute Engine, GKE, Cloud Run)
# - TCO Analysis: Compares on-prem costs to GCP costs
# - Migration waves: Group workloads for phased migration

# Import data into Migration Center
gcloud migration-center sources create my-source \
  --location=us-central1 \
  --type=UPLOAD

# Create an import job
gcloud migration-center import-jobs create my-import \
  --location=us-central1 \
  --source=my-source
```

#### Storage Transfer Service

Transfers data into Cloud Storage from:

- **Other cloud providers** (AWS S3, Azure Blob Storage)
- **On-premises** storage (via Storage Transfer Agent)
- **HTTP/HTTPS sources** (publicly accessible URLs)
- **Between GCS buckets** (cross-region, cross-project)

```bash
# Create a one-time transfer from S3 to GCS
gcloud transfer jobs create s3://my-aws-bucket gs://my-gcs-bucket \
  --source-creds-file=aws-creds.json

# Create a scheduled transfer (daily at midnight)
gcloud transfer jobs create s3://my-aws-bucket gs://my-gcs-bucket \
  --source-creds-file=aws-creds.json \
  --schedule-starts=2026-03-01 \
  --schedule-repeats-every=1d

# On-premises transfer (requires agent)
gcloud transfer agents install --pool=my-agent-pool

# Transfer between GCS buckets
gcloud transfer jobs create gs://source-bucket gs://dest-bucket

# Check transfer job status
gcloud transfer jobs describe JOB_ID
gcloud transfer operations list --job-names=JOB_ID
```

#### BigQuery Data Transfer Service

Automates data loading into BigQuery from:

- **SaaS sources:** Google Ads, Campaign Manager, YouTube, etc.
- **Cloud Storage** (scheduled loads)
- **Amazon S3** (cross-cloud)
- **Teradata, Amazon Redshift** (data warehouse migrations)

```bash
# Create a scheduled transfer from GCS to BigQuery
bq mk --transfer_config \
  --target_dataset=my_dataset \
  --display_name='GCS Daily Load' \
  --data_source=google_cloud_storage \
  --params='{"data_path_template":"gs://my-bucket/data/*.csv","destination_table_name_template":"my_table","file_format":"CSV"}'

# Create a transfer from Amazon S3
bq mk --transfer_config \
  --target_dataset=my_dataset \
  --display_name='S3 Import' \
  --data_source=amazon_s3 \
  --params='{"data_path":"s3://my-s3-bucket/data/*","destination_table_name_template":"s3_data","file_format":"JSON","access_key_id":"AKIAIOSFODNN7EXAMPLE","secret_access_key":"..."}'

# List transfer configs
bq ls --transfer_config --transfer_location=us

# Run a transfer manually
bq mk --transfer_run --run_time='2026-02-15T00:00:00Z' projects/my-project/locations/us/transferConfigs/CONFIG_ID
```

#### Datastream (Change Data Capture)

Datastream provides serverless change data capture (CDC) and replication. Streams changes from:

- **Sources:** MySQL, PostgreSQL, Oracle, SQL Server
- **Destinations:** BigQuery, Cloud Storage, Cloud SQL (PostgreSQL)

Use case: real-time replication to BigQuery for analytics.

```bash
# Create a Datastream connection profile
gcloud datastream connection-profiles create my-mysql-source \
  --location=us-central1 \
  --type=MYSQL \
  --mysql-hostname=10.0.0.5 \
  --mysql-port=3306 \
  --mysql-username=replication_user \
  --mysql-password=my-password

# Create a BigQuery destination profile
gcloud datastream connection-profiles create my-bq-dest \
  --location=us-central1 \
  --type=BIGQUERY

# Create a stream
gcloud datastream streams create my-stream \
  --location=us-central1 \
  --source=my-mysql-source \
  --destination=my-bq-dest \
  --source-config=source-config.json \
  --destination-config=dest-config.json
```

#### Migration Tooling Decision Matrix

| Scenario | Tool |
|----------|------|
| Migrate MySQL/PostgreSQL/SQL Server to Cloud SQL | **Database Migration Service (DMS)** |
| Migrate Oracle to Cloud SQL or AlloyDB | **DMS** (with conversion) |
| Migrate VMs to Compute Engine | **Migrate to VMs** |
| Containerize VMs to run on GKE | **Migrate to Containers** |
| Assess on-prem environment before migration | **Migration Center** |
| Move data from S3 to Cloud Storage | **Storage Transfer Service** |
| Move data from on-prem NAS to Cloud Storage | **Storage Transfer Service** (with agent) |
| Load data into BigQuery from SaaS/S3 | **BigQuery Data Transfer Service** |
| Real-time CDC from MySQL to BigQuery | **Datastream** |
| Migrate Teradata/Redshift to BigQuery | **BigQuery Data Transfer Service** |
| Move large datasets (petabytes) offline | **Transfer Appliance** |

> **Exam tips:**
> - **DMS** = database-to-database migration with continuous replication. Always pick DMS for relational DB migrations to Cloud SQL/AlloyDB.
> - **Datastream** = real-time CDC. If the question mentions "real-time replication to BigQuery" or "change data capture," pick Datastream.
> - **Storage Transfer Service** = cloud-to-cloud or on-prem-to-GCS file transfers. NOT for database migration.
> - **BigQuery Data Transfer Service** = scheduled data loading INTO BigQuery from external sources. Not for real-time.
> - **Migration Center** = assessment and planning ONLY. It does not perform the actual migration.
> - **Migrate to Containers** generates Docker artifacts from VMs -- it does NOT require code changes.
> - **Transfer Appliance** = physical device for petabyte-scale offline transfers when network bandwidth is insufficient.

**Docs:** [Database Migration Service](https://cloud.google.com/database-migration/docs) | [Migrate to Containers](https://cloud.google.com/migrate/containers/docs) | [Migrate to VMs](https://cloud.google.com/migrate/compute-engine/docs) | [Migration Center](https://cloud.google.com/migration-center/docs) | [Storage Transfer Service](https://cloud.google.com/storage-transfer-service/docs) | [BigQuery Data Transfer Service](https://cloud.google.com/bigquery/docs/transfer-service-overview) | [Datastream](https://cloud.google.com/datastream/docs)

---

### Gemini Cloud Assist

Gemini Cloud Assist is Google's AI assistant integrated across GCP services. The exam tests awareness of its capabilities, not deep usage.

#### Gemini in Cloud Console

- **Natural language queries:** Ask questions like "Show me all VMs in us-central1" or "How do I set up a VPN?"
- **Resource explanation:** Select a resource and ask Gemini to explain its configuration
- **Troubleshooting:** Describe an issue and get suggested fixes
- **SQL generation:** Write BigQuery SQL from natural language descriptions
- **IAM analysis:** "Who has access to this bucket?" style queries

#### Gemini Code Assist in IDEs

- **Code completion** in Cloud Code-supported IDEs (VS Code, IntelliJ)
- **Code generation** from natural language prompts
- **Code explanation** -- highlight code and ask "what does this do?"
- **Code transformation** -- refactor, translate between languages
- **Architecture guidance** for GCP services

```bash
# Gemini Code Assist is enabled via Cloud Code plugin
# In VS Code: Install "Cloud Code" extension, sign in with your GCP account
# Gemini chat panel appears in the sidebar

# Gemini in Cloud Shell is also available
# Simply type natural language in the Cloud Shell terminal
```

#### Gemini in Databases

- **Cloud SQL Studio** and **AlloyDB Studio** -- natural language to SQL
- **Spanner Studio** -- query generation and optimization suggestions
- **BigQuery Studio** -- SQL generation, query explanation, optimization tips

#### Gemini for Operations

- **Cloud Monitoring:** Explain alert conditions, suggest SLO configurations
- **Cloud Logging:** Natural language log queries, anomaly detection
- **Error Reporting:** Root cause analysis suggestions
- **Network Intelligence:** Troubleshoot connectivity issues

> **Exam tips:**
> - Gemini Cloud Assist is advisory -- it suggests, but you approve. It does not make autonomous changes.
> - Gemini Code Assist = IDE integration for code. Gemini Cloud Assist = Cloud Console integration for operations.
> - Data processed by Gemini Cloud Assist stays within your project's data residency boundaries.
> - The exam tests WHEN to use Gemini (natural language queries, troubleshooting) not HOW to prompt it.

**Docs:** [Gemini in Google Cloud](https://cloud.google.com/gemini/docs/overview) | [Gemini Code Assist](https://cloud.google.com/gemini/docs/codeassist/overview) | [Gemini Cloud Assist](https://cloud.google.com/gemini/docs/cloud-assist/overview)

---

## 5.2 Interacting with Google Cloud Programmatically

### Cloud Shell and Cloud Code

#### Cloud Shell

Cloud Shell is a free, browser-based terminal with a persistent 5 GB home directory. Comes pre-installed with:

- `gcloud`, `gsutil`, `bq`, `kubectl`, `terraform`, `docker`, `git`, `python`, `java`, `go`, `node`
- Authenticated automatically with your logged-in account
- Ephemeral VM (Debian-based, `e2-small`), resets after 120 minutes of inactivity
- Home directory persists across sessions (5 GB limit)

```bash
# Cloud Shell is accessed via the console (top-right terminal icon)
# Or via direct URL: https://shell.cloud.google.com

# Upload/download files
# Upload: Click the 3-dot menu > Upload
# Download:
cloudshell dl ~/my-file.txt

# Open the Cloud Shell Editor (VS Code-based web IDE)
cloudshell edit ~/my-file.py

# Open Cloud Shell in a specific project
# https://shell.cloud.google.com/?project=my-project-id

# Customize your Cloud Shell environment (persisted in $HOME)
# .bashrc, .profile, installed pip/npm packages in $HOME
```

**Cloud Shell limitations:**

- 50 hours/week usage limit
- No GPU, limited CPU/RAM
- Sessions timeout after 20 minutes of idle (terminal) or 120 minutes (VM)
- Not for production workloads -- development and admin tasks only
- 5 GB home directory limit

#### Cloud Shell Editor

Cloud Shell Editor is a browser-based VS Code (Theia-based) IDE. Supports:

- File editing with syntax highlighting
- Integrated terminal (Cloud Shell)
- Cloud Code extension (pre-installed)
- Source control (Git)
- Kubernetes and Cloud Run development/debugging

#### Cloud Code

Cloud Code is a plugin for VS Code and IntelliJ/JetBrains IDEs that provides:

- **Kubernetes development:** Create, run, debug apps on GKE or minikube
- **Cloud Run development:** Local emulation, deploy with one click
- **Secret Manager integration:** Browse and manage secrets
- **Cloud APIs explorer:** Browse available APIs and enable them
- **Skaffold integration:** Continuous development for Kubernetes
- **YAML authoring:** Schema validation for Kubernetes manifests, Cloud Build configs

```bash
# Install Cloud Code for VS Code
# Extensions panel > Search "Cloud Code" > Install

# Cloud Code features:
# - Kubernetes Explorer: View clusters, pods, services, deployments
# - Cloud Run Explorer: View services, revisions
# - API Explorer: Enable/disable APIs
# - Secret Manager: View/create secrets
# - Container image registry browser
```

#### Cloud Workstations

Cloud Workstations provides managed, secure development environments running on Compute Engine. Unlike Cloud Shell, these are full persistent VMs with custom configurations.

```bash
# Create a workstation cluster
gcloud workstations clusters create my-cluster \
  --region=us-central1

# Create a workstation config (defines the machine type, image, etc.)
gcloud workstations configs create my-config \
  --cluster=my-cluster \
  --region=us-central1 \
  --machine-type=e2-standard-4 \
  --pd-disk-size=200 \
  --idle-timeout=7200s

# Create a workstation
gcloud workstations create my-workstation \
  --config=my-config \
  --cluster=my-cluster \
  --region=us-central1

# Start a workstation
gcloud workstations start my-workstation \
  --config=my-config \
  --cluster=my-cluster \
  --region=us-central1

# Access via browser-based IDE or SSH
gcloud workstations ssh my-workstation \
  --config=my-config \
  --cluster=my-cluster \
  --region=us-central1
```

**Cloud Shell vs Cloud Workstations:**

| Feature | Cloud Shell | Cloud Workstations |
|---------|------------|-------------------|
| **Cost** | Free | Paid (Compute Engine pricing) |
| **Persistence** | 5 GB home dir only | Full VM disk persists |
| **Machine type** | Fixed (e2-small) | Configurable (any machine type) |
| **Idle timeout** | 20 min (terminal), 120 min (VM) | Configurable |
| **Custom images** | No | Yes (bring your own container image) |
| **GPU** | No | Yes |
| **Use case** | Quick admin tasks, tutorials | Full development environment, teams |
| **Private networking** | No (public internet) | Yes (VPC-connected) |

> **Exam tips:**
> - Cloud Shell = free, quick, ephemeral. Cloud Workstations = paid, persistent, customizable.
> - Cloud Shell's 5 GB home directory persists, but the VM environment resets. Anything installed outside `$HOME` is lost.
> - Cloud Code is the IDE plugin (VS Code/IntelliJ). Cloud Shell Editor is the browser-based IDE.
> - Cloud Workstations can connect to private VPC networks -- Cloud Shell cannot.
> - If the question mentions "team development environment" or "custom IDE configuration," the answer is Cloud Workstations.

**Docs:** [Cloud Shell](https://cloud.google.com/shell/docs) | [Cloud Code](https://cloud.google.com/code/docs) | [Cloud Workstations](https://cloud.google.com/workstations/docs)

---

### Google Cloud SDKs

#### gcloud CLI

The primary CLI for interacting with GCP. Organized into component groups.

```bash
# Install and initialize
gcloud init  # Interactive setup (project, region, zone)
gcloud auth login  # Browser-based authentication
gcloud auth application-default login  # Set up Application Default Credentials

# Configurations (manage multiple project/account profiles)
gcloud config configurations create my-dev-config
gcloud config configurations activate my-dev-config
gcloud config set project my-dev-project
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
gcloud config set account my-dev@company.com

# Switch between configurations
gcloud config configurations activate my-prod-config

# List all configurations
gcloud config configurations list

# View current configuration
gcloud config list

# Override config for a single command with --project, --region, --zone flags
gcloud compute instances list --project=other-project

# Components management
gcloud components list
gcloud components install kubectl
gcloud components install beta
gcloud components update

# Useful global flags
gcloud ... --format=json   # Output as JSON
gcloud ... --format=yaml   # Output as YAML
gcloud ... --format="table(name,zone,status)"  # Custom table
gcloud ... --filter="name~'^prod'"  # Server-side filtering
gcloud ... --quiet  # Suppress prompts (for scripts)
gcloud ... --verbosity=debug  # Debug output
gcloud ... --impersonate-service-account=SA@PROJECT.iam.gserviceaccount.com
```

#### gsutil (Cloud Storage)

```bash
# Copy files
gsutil cp local-file.txt gs://my-bucket/
gsutil cp gs://my-bucket/file.txt .
gsutil cp -r local-dir/ gs://my-bucket/dir/  # Recursive

# Parallel composite upload (large files)
gsutil -o GSUtil:parallel_composite_upload_threshold=150M cp large-file.tar.gz gs://my-bucket/

# Sync directories
gsutil rsync -r local-dir/ gs://my-bucket/dir/
gsutil rsync -d -r gs://source-bucket/ gs://dest-bucket/  # -d deletes extra files in dest

# List and manage
gsutil ls gs://my-bucket/
gsutil ls -l gs://my-bucket/  # Long listing with sizes
gsutil du -s gs://my-bucket/  # Total bucket size

# ACLs and IAM
gsutil iam get gs://my-bucket/
gsutil iam ch user:dev@example.com:objectViewer gs://my-bucket/

# Lifecycle and versioning
gsutil versioning set on gs://my-bucket/
gsutil lifecycle set lifecycle.json gs://my-bucket/

# Multi-threaded operations
gsutil -m cp -r large-dir/ gs://my-bucket/  # -m enables multi-threading
gsutil -m rsync -r source/ gs://my-bucket/dest/
```

> **Note:** `gsutil` is gradually being replaced by `gcloud storage` commands, but `gsutil` is still widely used and exam-relevant.

```bash
# gcloud storage equivalents (newer, faster)
gcloud storage cp local-file.txt gs://my-bucket/
gcloud storage ls gs://my-bucket/
gcloud storage rsync local-dir/ gs://my-bucket/dir/
```

#### bq CLI (BigQuery)

```bash
# Run a query
bq query --use_legacy_sql=false 'SELECT * FROM `my-project.my_dataset.my_table` LIMIT 10'

# Create a dataset
bq mk --dataset my-project:my_dataset

# Create a table
bq mk --table my-project:my_dataset.my_table schema.json

# Load data
bq load --source_format=CSV my_dataset.my_table gs://my-bucket/data.csv schema.json
bq load --autodetect --source_format=NEWLINE_DELIMITED_JSON my_dataset.my_table gs://my-bucket/data.jsonl

# Export data
bq extract my_dataset.my_table gs://my-bucket/export/data-*.csv

# List datasets, tables, jobs
bq ls
bq ls my_dataset
bq ls --jobs --all

# Show table details
bq show my_dataset.my_table
bq show --schema my_dataset.my_table
```

#### cbt CLI (Bigtable)

```bash
# Install cbt
gcloud components install cbt

# Configure cbt
echo project=my-project > ~/.cbtrc
echo instance=my-bigtable-instance >> ~/.cbtrc

# List tables
cbt ls

# Create a table and column family
cbt createtable my-table
cbt createfamily my-table cf1

# Read/write data
cbt set my-table row1 cf1:col1=value1
cbt read my-table

# Count rows
cbt count my-table

# Delete
cbt deleterow my-table row1
cbt deletetable my-table
```

#### kubectl (GKE)

```bash
# Get credentials for a GKE cluster
gcloud container clusters get-credentials my-cluster --zone=us-central1-a

# Common kubectl commands
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get nodes
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl exec -it POD_NAME -- /bin/bash
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
kubectl scale deployment my-app --replicas=5
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
kubectl top pods  # Resource usage (requires metrics-server)
kubectl top nodes
```

#### SDK Client Libraries

Google provides client libraries in Python, Java, Go, Node.js, C#, Ruby, and PHP. Two types:

| Library Type | Description | When to Use |
|-------------|-------------|-------------|
| **Cloud Client Libraries** | Idiomatic, higher-level, recommended | New development (always prefer these) |
| **Google API Client Libraries** | Auto-generated, lower-level | When Cloud Client Library doesn't exist for a service |

```python
# Python example: Cloud Client Library for Cloud Storage
from google.cloud import storage

client = storage.Client()  # Uses Application Default Credentials
bucket = client.get_bucket('my-bucket')
blob = bucket.blob('my-file.txt')
blob.upload_from_filename('/local/path/file.txt')

# Download
blob.download_to_filename('/local/path/downloaded.txt')

# List blobs
blobs = client.list_blobs('my-bucket', prefix='data/')
for blob in blobs:
    print(blob.name)
```

```python
# Python example: Cloud Client Library for BigQuery
from google.cloud import bigquery

client = bigquery.Client()
query = "SELECT name, count FROM `my_dataset.my_table` WHERE count > 10"
results = client.query(query)
for row in results:
    print(f"{row.name}: {row.count}")
```

> **Exam tips:**
> - `gcloud config configurations` lets you switch between projects/accounts quickly. Know this for multi-project scenarios.
> - `gsutil -m` enables multi-threading for parallel operations. Use for large transfers.
> - `bq --use_legacy_sql=false` is required for standard SQL syntax. Legacy SQL is the default (unfortunately).
> - Cloud Client Libraries use Application Default Credentials (ADC) by default. ADC checks: (1) `GOOGLE_APPLICATION_CREDENTIALS` env var, (2) `gcloud auth application-default` credentials, (3) attached service account (on GCE/GKE/Cloud Run).
> - `gcloud storage` is the modern replacement for `gsutil` -- faster, more consistent syntax.
> - Always prefer Cloud Client Libraries over Google API Client Libraries for new development.

**Docs:** [gcloud CLI reference](https://cloud.google.com/sdk/gcloud/reference) | [gsutil](https://cloud.google.com/storage/docs/gsutil) | [bq CLI](https://cloud.google.com/bigquery/docs/bq-command-line-tool) | [cbt CLI](https://cloud.google.com/bigtable/docs/cbt-reference) | [Cloud Client Libraries](https://cloud.google.com/apis/docs/cloud-client-libraries)

---

### Cloud Emulators

Cloud emulators let you develop and test locally without incurring costs or needing network access. They simulate GCP services on your local machine.

#### Available Emulators

| Service | Emulator | Start Command |
|---------|----------|--------------|
| **Pub/Sub** | Yes | `gcloud beta emulators pubsub start` |
| **Bigtable** | Yes | `gcloud beta emulators bigtable start` |
| **Spanner** | Yes | `gcloud beta emulators spanner start` |
| **Firestore** | Yes | `gcloud beta emulators firestore start` |
| **Datastore** | Yes | `gcloud beta emulators datastore start` |

#### Pub/Sub Emulator

```bash
# Start the Pub/Sub emulator
gcloud beta emulators pubsub start --project=my-test-project

# In a separate terminal, set environment variables
$(gcloud beta emulators pubsub env-init)
# This sets PUBSUB_EMULATOR_HOST=localhost:8085

# Verify the env var
echo $PUBSUB_EMULATOR_HOST
# Output: localhost:8085

# Your client libraries automatically detect PUBSUB_EMULATOR_HOST
# and route requests to the emulator instead of the real service
```

```python
# Python: Client library auto-detects emulator
import os
os.environ["PUBSUB_EMULATOR_HOST"] = "localhost:8085"

from google.cloud import pubsub_v1

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path("my-test-project", "my-topic")

# Create a topic (on the emulator)
publisher.create_topic(request={"name": topic_path})

# Publish a message
future = publisher.publish(topic_path, b"Hello, emulator!")
print(f"Published message ID: {future.result()}")
```

#### Bigtable Emulator

```bash
# Start the Bigtable emulator
gcloud beta emulators bigtable start

# Set environment variables
$(gcloud beta emulators bigtable env-init)
# Sets BIGTABLE_EMULATOR_HOST=localhost:8086

# Use cbt with the emulator
cbt -project=my-test-project -instance=my-test-instance createtable test-table
cbt -project=my-test-project -instance=my-test-instance createfamily test-table cf1
cbt -project=my-test-project -instance=my-test-instance set test-table row1 cf1:col1=value1
cbt -project=my-test-project -instance=my-test-instance read test-table
```

#### Spanner Emulator

```bash
# Start the Spanner emulator
gcloud beta emulators spanner start

# Set environment variables
$(gcloud beta emulators spanner env-init)
# Sets SPANNER_EMULATOR_HOST=localhost:9010

# Create an instance on the emulator
gcloud spanner instances create test-instance \
  --config=emulator-config \
  --description="Test Instance" \
  --nodes=1

# Create a database
gcloud spanner databases create test-db --instance=test-instance \
  --ddl='CREATE TABLE Users (UserId INT64 NOT NULL, Name STRING(100)) PRIMARY KEY (UserId)'
```

#### Firestore Emulator

```bash
# Start the Firestore emulator
gcloud beta emulators firestore start --host-port=localhost:8080

# Set environment variables
export FIRESTORE_EMULATOR_HOST=localhost:8080

# Your Firestore client library will automatically use the emulator
```

```python
# Python: Firestore with emulator
import os
os.environ["FIRESTORE_EMULATOR_HOST"] = "localhost:8080"

from google.cloud import firestore

db = firestore.Client(project="my-test-project")
doc_ref = db.collection("users").document("user1")
doc_ref.set({"name": "Alice", "age": 30})

doc = doc_ref.get()
print(f"Document data: {doc.to_dict()}")
```

#### Emulator Limitations

| Limitation | Details |
|-----------|---------|
| **Not feature-complete** | Emulators may not support all features of the real service |
| **No IAM** | Emulators do not enforce IAM permissions |
| **No quotas/limits** | Rate limits and quotas are not enforced |
| **Data is ephemeral** | Data is lost when the emulator stops (unless explicitly persisted) |
| **No cross-service integration** | Emulators don't talk to each other or to real GCP services |
| **Performance differs** | Emulator performance does not reflect production performance |
| **Spanner emulator** | Does not support advanced features like change streams, foreign keys |
| **Pub/Sub emulator** | Does not support schemas, BigQuery subscriptions, or exactly-once delivery |

**Key pattern:** All emulators use an environment variable (`*_EMULATOR_HOST`) that client libraries detect automatically. You do NOT change your application code -- just set the env var.

> **Exam tips:**
> - Emulators are for **development and testing only**. Never use emulators in production.
> - The exam tests whether you know emulators exist and when to use them (local development, CI/CD pipelines, offline testing).
> - All emulators are accessed via the `gcloud beta emulators` command group.
> - Client libraries auto-detect emulator environment variables -- no code changes needed to switch between emulator and production.
> - Not all services have emulators. Cloud SQL, BigQuery, Cloud Storage do NOT have emulators (use real services or mock libraries).
> - Spanner emulator is great for schema testing but does not validate performance characteristics.

**Docs:** [Pub/Sub emulator](https://cloud.google.com/pubsub/docs/emulator) | [Bigtable emulator](https://cloud.google.com/bigtable/docs/emulator) | [Spanner emulator](https://cloud.google.com/spanner/docs/emulator) | [Firestore emulator](https://cloud.google.com/firestore/docs/emulator) | [Datastore emulator](https://cloud.google.com/datastore/docs/tools/datastore-emulator)

---

### Infrastructure as Code (Terraform)

This is the most heavily tested IaC topic on the PCA exam. Know Terraform deeply.

#### Terraform Fundamentals

Terraform uses HashiCorp Configuration Language (HCL) to define infrastructure declaratively. The Google Cloud provider translates HCL into API calls.

```hcl
# main.tf -- Basic Terraform configuration

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

# Variables
variable "project_id" {
  description = "GCP project ID"
  type        = string
}

variable "region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}
```

#### Core Terraform Commands

```bash
# Initialize -- downloads providers, initializes backend
terraform init

# Format -- auto-formats HCL files
terraform fmt

# Validate -- checks syntax without accessing APIs
terraform validate

# Plan -- preview changes (dry run)
terraform plan
terraform plan -out=plan.tfplan  # Save plan to a file (recommended)

# Apply -- execute the plan
terraform apply
terraform apply plan.tfplan  # Apply a saved plan (skips confirmation)
terraform apply -auto-approve  # Skip confirmation (CI/CD)

# Destroy -- delete all managed resources
terraform destroy
terraform destroy -target=google_compute_instance.my_vm  # Destroy specific resource

# Import -- bring existing resources under Terraform management
terraform import google_compute_instance.my_vm projects/my-project/zones/us-central1-a/instances/my-vm

# State commands
terraform state list  # List all resources in state
terraform state show google_compute_instance.my_vm  # Show resource details
terraform state mv google_compute_instance.old google_compute_instance.new  # Rename
terraform state rm google_compute_instance.my_vm  # Remove from state (doesn't delete resource)
terraform state pull  # Download remote state to stdout
terraform state push  # Upload local state to remote backend

# Output -- display output values
terraform output
terraform output -json

# Taint/untaint (deprecated in favor of -replace)
terraform apply -replace=google_compute_instance.my_vm  # Force recreate
```

#### State Management

Terraform state tracks the mapping between your config and real-world resources. State management is critical for team workflows.

**Local state (default -- NOT for teams):**

```hcl
# By default, state is stored in terraform.tfstate (local file)
# Fine for individual use, NOT for teams
```

**Remote state with GCS backend (recommended for GCP):**

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state-bucket"
    prefix = "terraform/state"
  }
}
```

```bash
# Create the state bucket with versioning (important for state recovery)
gsutil mb -l us-central1 gs://my-terraform-state-bucket
gsutil versioning set on gs://my-terraform-state-bucket

# Enable uniform bucket-level access
gsutil uniformbucketlevelaccess set on gs://my-terraform-state-bucket
```

**State locking:**

- GCS backend supports state locking natively (prevents concurrent modifications)
- Lock info is stored as a `.tflock` object in the bucket
- If a lock is stuck: `terraform force-unlock LOCK_ID`

**Remote state data source (read another project's state):**

```hcl
# Read state from another Terraform configuration (e.g., networking project)
data "terraform_remote_state" "network" {
  backend = "gcs"
  config = {
    bucket = "network-team-terraform-state"
    prefix = "terraform/network"
  }
}

# Use outputs from the remote state
resource "google_compute_instance" "my_vm" {
  name         = "my-vm"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  network_interface {
    subnetwork = data.terraform_remote_state.network.outputs.subnet_self_link
  }

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }
}
```

**Terraform Cloud / Terraform Enterprise:**

- Remote execution (runs Terraform in the cloud, not locally)
- State management with built-in locking and versioning
- Policy as Code (Sentinel)
- Private module registry
- Team access controls and workspace permissions

#### Modules

Modules are reusable Terraform configurations. They promote DRY principles and standardization.

```hcl
# Using a public registry module (Cloud Foundation Toolkit)
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 9.0"

  project_id   = var.project_id
  network_name = "my-vpc"
  routing_mode = "GLOBAL"

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "us-central1"
    },
    {
      subnet_name           = "subnet-02"
      subnet_ip             = "10.10.20.0/24"
      subnet_region         = "us-east1"
      subnet_private_access = true
    }
  ]

  secondary_ranges = {
    subnet-01 = [
      {
        range_name    = "pods"
        ip_cidr_range = "10.20.0.0/16"
      },
      {
        range_name    = "services"
        ip_cidr_range = "10.30.0.0/16"
      }
    ]
  }
}
```

```hcl
# Using a local module
module "web_server" {
  source = "./modules/web-server"

  instance_name = "web-01"
  machine_type  = "e2-standard-2"
  zone          = "us-central1-a"
  network       = module.vpc.network_self_link
  subnet        = module.vpc.subnets["us-central1/subnet-01"].self_link
}
```

```hcl
# modules/web-server/main.tf -- Example custom module
variable "instance_name" {
  type = string
}

variable "machine_type" {
  type    = string
  default = "e2-medium"
}

variable "zone" {
  type = string
}

variable "network" {
  type = string
}

variable "subnet" {
  type = string
}

resource "google_compute_instance" "web" {
  name         = var.instance_name
  machine_type = var.machine_type
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    network    = var.network
    subnetwork = var.subnet

    access_config {
      // Ephemeral public IP
    }
  }

  metadata_startup_script = file("${path.module}/startup.sh")

  tags = ["http-server"]
}

resource "google_compute_firewall" "allow_http" {
  name    = "${var.instance_name}-allow-http"
  network = var.network

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["http-server"]
}

output "instance_ip" {
  value = google_compute_instance.web.network_interface[0].access_config[0].nat_ip
}
```

#### Cloud Foundation Toolkit (CFT)

CFT is a collection of best-practice Terraform modules maintained by Google. They encode Google's recommended architectures and security patterns.

**Key CFT modules:**

| Module | Purpose |
|--------|---------|
| `terraform-google-modules/project-factory` | Create and configure GCP projects |
| `terraform-google-modules/network` | VPC networks, subnets, routes, NAT |
| `terraform-google-modules/kubernetes-engine` | GKE clusters with best practices |
| `terraform-google-modules/cloud-storage` | Storage buckets with policies |
| `terraform-google-modules/iam` | IAM bindings and custom roles |
| `terraform-google-modules/sql-db` | Cloud SQL instances |
| `terraform-google-modules/log-export` | Centralized logging sinks |
| `terraform-google-modules/org-policy` | Organization policy constraints |
| `terraform-google-modules/vpc-service-controls` | VPC Service Controls perimeters |

```hcl
# Example: CFT Project Factory
module "project" {
  source  = "terraform-google-modules/project-factory/google"
  version = "~> 15.0"

  name              = "my-new-project"
  org_id            = var.org_id
  billing_account   = var.billing_account
  folder_id         = var.folder_id
  random_project_id = true

  activate_apis = [
    "compute.googleapis.com",
    "container.googleapis.com",
    "iam.googleapis.com",
  ]

  labels = {
    environment = "production"
    team        = "platform"
  }
}
```

```hcl
# Example: CFT GKE module
module "gke" {
  source  = "terraform-google-modules/kubernetes-engine/google//modules/private-cluster"
  version = "~> 33.0"

  project_id         = var.project_id
  name               = "my-gke-cluster"
  region             = "us-central1"
  network            = module.vpc.network_name
  subnetwork         = "subnet-01"
  ip_range_pods      = "pods"
  ip_range_services  = "services"

  enable_private_nodes    = true
  enable_private_endpoint = false
  master_ipv4_cidr_block  = "172.16.0.0/28"

  node_pools = [
    {
      name         = "default-pool"
      machine_type = "e2-standard-4"
      min_count    = 1
      max_count    = 10
      auto_repair  = true
      auto_upgrade = true
      disk_size_gb = 100
      disk_type    = "pd-standard"
    }
  ]
}
```

#### Terraform Best Practices for GCP

**Directory structure:**

```
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       └── backend.tf
├── modules/
│   ├── networking/
│   ├── compute/
│   ├── database/
│   └── security/
└── shared/
    └── terraform.tfvars  # Shared variables
```

**Best practices:**

1. **State per environment:** Separate state files for dev, staging, prod. Never share state across environments.
2. **State locking:** Always use a backend that supports locking (GCS does).
3. **State encryption:** GCS buckets are encrypted by default. Add CMEK for additional control.
4. **Version pinning:** Pin provider and module versions (`~>` for minor version flexibility).
5. **Plan before apply:** Always run `terraform plan` and review changes before `apply`.
6. **Use variables and tfvars:** Never hardcode values. Use `terraform.tfvars` for environment-specific values.
7. **Use outputs:** Export resource attributes for cross-module references.
8. **Sensitive values:** Mark sensitive variables with `sensitive = true`. Use Secret Manager for secrets.
9. **Import existing resources:** Use `terraform import` before managing existing infrastructure.
10. **Use workspaces sparingly:** Workspaces are fine for simple multi-env setups, but separate directories are clearer for complex ones.

```hcl
# Workspaces (alternative to separate directories for environments)
# terraform workspace new dev
# terraform workspace new staging
# terraform workspace new prod
# terraform workspace select dev

resource "google_compute_instance" "vm" {
  name         = "vm-${terraform.workspace}"
  machine_type = terraform.workspace == "prod" ? "e2-standard-4" : "e2-medium"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    network = "default"
  }
}
```

#### Common Terraform Patterns for GCP

**GKE cluster with node pool:**

```hcl
resource "google_container_cluster" "primary" {
  name     = "my-gke-cluster"
  location = "us-central1"

  # Separately managed node pool (recommended)
  remove_default_node_pool = true
  initial_node_count       = 1

  networking_mode = "VPC_NATIVE"
  network         = google_compute_network.vpc.name
  subnetwork      = google_compute_subnetwork.subnet.name

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }
}

resource "google_container_node_pool" "primary_nodes" {
  name       = "primary-pool"
  location   = "us-central1"
  cluster    = google_container_cluster.primary.name
  node_count = 3

  node_config {
    machine_type    = "e2-standard-4"
    service_account = google_service_account.gke_sa.email

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      env = var.environment
    }
  }

  autoscaling {
    min_node_count = 1
    max_node_count = 10
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}
```

**Cloud SQL with private IP:**

```hcl
resource "google_sql_database_instance" "main" {
  name             = "my-sql-instance"
  database_version = "POSTGRES_15"
  region           = "us-central1"

  depends_on = [google_service_networking_connection.private_vpc_connection]

  settings {
    tier              = "db-custom-4-16384"
    availability_type = "REGIONAL"  # High availability

    ip_configuration {
      ipv4_enabled                                  = false
      private_network                               = google_compute_network.vpc.id
      enable_private_path_for_google_cloud_services = true
    }

    backup_configuration {
      enabled                        = true
      point_in_time_recovery_enabled = true
      start_time                     = "03:00"
      transaction_log_retention_days = 7

      backup_retention_settings {
        retained_backups = 30
      }
    }

    maintenance_window {
      day          = 7  # Sunday
      hour         = 3  # 3 AM
      update_track = "stable"
    }

    database_flags {
      name  = "log_checkpoints"
      value = "on"
    }
  }

  deletion_protection = true
}

# Private service connection for Cloud SQL private IP
resource "google_compute_global_address" "private_ip_address" {
  name          = "private-ip-address"
  purpose       = "VPC_PEERING"
  address_type  = "INTERNAL"
  prefix_length = 16
  network       = google_compute_network.vpc.id
}

resource "google_service_networking_connection" "private_vpc_connection" {
  network                 = google_compute_network.vpc.id
  service                 = "servicenetworking.googleapis.com"
  reserved_peering_ranges = [google_compute_global_address.private_ip_address.name]
}
```

**Cloud Run service:**

```hcl
resource "google_cloud_run_v2_service" "default" {
  name     = "my-service"
  location = "us-central1"

  template {
    containers {
      image = "gcr.io/${var.project_id}/my-app:latest"

      ports {
        container_port = 8080
      }

      resources {
        limits = {
          cpu    = "2"
          memory = "1Gi"
        }
      }

      env {
        name  = "ENV"
        value = "production"
      }

      env {
        name = "DB_PASSWORD"
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

    service_account = google_service_account.cloudrun_sa.email
  }

  traffic {
    type    = "TRAFFIC_TARGET_ALLOCATION_TYPE_LATEST"
    percent = 100
  }
}

# Allow unauthenticated access (public API)
resource "google_cloud_run_v2_service_iam_member" "public" {
  project  = google_cloud_run_v2_service.default.project
  location = google_cloud_run_v2_service.default.location
  name     = google_cloud_run_v2_service.default.name
  role     = "roles/run.invoker"
  member   = "allUsers"
}
```

#### Policy Validation

**Sentinel (Terraform Cloud/Enterprise):**

```python
# Sentinel policy: Ensure all GCE instances use approved machine types
import "tfplan/v2" as tfplan

allowed_machine_types = ["e2-medium", "e2-standard-2", "e2-standard-4"]

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is "google_compute_instance" and
    rc.change.after.machine_type in allowed_machine_types
  }
}
```

**OPA/Gatekeeper (for GKE and Terraform):**

```rego
# OPA policy: Deny Terraform plans that create public Cloud SQL instances
package terraform.google

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "google_sql_database_instance"
  resource.change.after.settings[_].ip_configuration[_].ipv4_enabled == true
  msg := sprintf("Cloud SQL instance '%s' must not have a public IP", [resource.name])
}
```

```bash
# Validate Terraform plan against OPA policies
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json
opa eval --data policies/ --input plan.json "data.terraform.google.deny"
```

#### IaC Alternatives on GCP

**Config Connector (Kubernetes-native IaC):**

Config Connector is a Kubernetes add-on that lets you manage GCP resources using Kubernetes Custom Resources (CRDs). It runs in a GKE cluster and reconciles GCP resources based on Kubernetes manifests.

```yaml
# Config Connector: Create a Cloud Storage bucket
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-bucket
  annotations:
    cnrm.cloud.google.com/project-id: my-project
spec:
  location: US
  uniformBucketLevelAccess: true
---
# Config Connector: Create a Cloud SQL instance
apiVersion: sql.cnrm.cloud.google.com/v1beta1
kind: SQLInstance
metadata:
  name: my-sql-instance
spec:
  databaseVersion: POSTGRES_15
  region: us-central1
  settings:
    tier: db-f1-micro
```

```bash
# Install Config Connector on a GKE cluster
gcloud container clusters update my-cluster \
  --update-addons=ConfigConnector=ENABLED \
  --zone=us-central1-a

# Verify Config Connector is running
kubectl get pods -n cnrm-system

# Apply a Config Connector resource
kubectl apply -f storage-bucket.yaml

# Check resource status
kubectl describe storagebucket my-bucket
```

**Deployment Manager (legacy, avoid for new projects):**

Deployment Manager is Google's native IaC tool using YAML/Jinja2/Python templates. It is legacy and not recommended for new projects. Use Terraform instead.

```yaml
# deployment.yaml (Deployment Manager)
resources:
  - name: my-vm
    type: compute.v1.instance
    properties:
      zone: us-central1-a
      machineType: zones/us-central1-a/machineTypes/e2-medium
      disks:
        - deviceName: boot
          type: PERSISTENT
          boot: true
          autoDelete: true
          initializeParams:
            sourceImage: projects/debian-cloud/global/images/family/debian-12
      networkInterfaces:
        - network: global/networks/default
```

```bash
# Deploy with Deployment Manager
gcloud deployment-manager deployments create my-deployment --config=deployment.yaml

# Update
gcloud deployment-manager deployments update my-deployment --config=deployment-v2.yaml

# Delete
gcloud deployment-manager deployments delete my-deployment
```

**Pulumi (third-party, multi-cloud):**

Pulumi uses general-purpose programming languages (Python, Go, TypeScript, Java) instead of HCL. Supports GCP via the Pulumi Google Cloud provider.

#### IaC Comparison Table

| Feature | Terraform | Deployment Manager | Config Connector | Pulumi |
|---------|-----------|-------------------|-----------------|--------|
| **Language** | HCL | YAML/Jinja2/Python | YAML (K8s CRDs) | Python, Go, TS, Java |
| **State** | Managed (local/remote) | GCP-managed | Kubernetes etcd | Managed (Pulumi Cloud) |
| **Multi-cloud** | Yes | No (GCP only) | No (GCP only) | Yes |
| **Status** | Recommended | Legacy | Active (K8s teams) | Active |
| **Community** | Largest | Small | Growing | Growing |
| **GCP support** | Comprehensive | Full | Comprehensive | Comprehensive |
| **GitOps** | Via CI/CD | No | Native (K8s) | Via CI/CD |
| **Best for** | Most teams | (don't use) | K8s-native teams | Dev teams who prefer code |

> **Exam tips:**
> - **Terraform is the default answer** for IaC on the PCA exam. If the question doesn't mention Kubernetes-native, pick Terraform.
> - **State in GCS** with versioning enabled is the recommended backend for GCP. Know how to set up the `backend "gcs"` block.
> - **State locking** prevents concurrent modifications. GCS backend supports it natively.
> - **`terraform import`** brings existing resources under management without recreating them. Critical for brownfield environments.
> - **`terraform state rm`** removes a resource from state WITHOUT deleting the actual resource. Use when you want to stop managing a resource.
> - **CFT modules** are the Google-recommended Terraform modules. If the question says "Google best practices" or "opinionated modules," pick CFT.
> - **Config Connector** = Kubernetes-native IaC. If the question says "manage GCP resources with kubectl" or "Kubernetes-style declarative management," pick Config Connector.
> - **Deployment Manager** is legacy. The exam may include it as a distractor. If the question asks for the "recommended" approach, pick Terraform.
> - **Workspaces vs directories:** Workspaces are simpler but less visible. Separate directories per environment are recommended for production setups.
> - **Sentinel** = policy as code for Terraform Cloud/Enterprise. **OPA/Gatekeeper** = open-source alternative.
> - `terraform plan -out=plan.tfplan` then `terraform apply plan.tfplan` is the safest workflow (ensures you apply exactly what you reviewed).
> - Always enable **versioning on the state bucket** so you can recover from state corruption.

**Docs:** [Terraform Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs) | [Cloud Foundation Toolkit](https://cloud.google.com/docs/terraform/blueprints/terraform-blueprints) | [Config Connector](https://cloud.google.com/config-connector/docs/overview) | [Deployment Manager](https://cloud.google.com/deployment-manager/docs) | [Terraform best practices on GCP](https://cloud.google.com/docs/terraform/best-practices-for-terraform)

---

### Accessing Google API Best Practices

#### REST API Conventions

Google Cloud APIs follow consistent REST conventions:

```
# API endpoint pattern
https://{service}.googleapis.com/{version}/{resource}

# Examples
https://compute.googleapis.com/compute/v1/projects/my-project/zones/us-central1-a/instances
https://storage.googleapis.com/storage/v1/b/my-bucket/o
https://bigquery.googleapis.com/bigquery/v2/projects/my-project/datasets
```

**Common HTTP methods:**

| Method | Action | Example |
|--------|--------|---------|
| `GET` | Read/list | `GET /compute/v1/projects/{project}/zones/{zone}/instances` |
| `POST` | Create | `POST /compute/v1/projects/{project}/zones/{zone}/instances` |
| `PUT/PATCH` | Update | `PATCH /compute/v1/projects/{project}/zones/{zone}/instances/{instance}` |
| `DELETE` | Delete | `DELETE /compute/v1/projects/{project}/zones/{zone}/instances/{instance}` |

```bash
# Use gcloud to discover API endpoints
gcloud services list --available --filter="name:compute"
gcloud services enable compute.googleapis.com

# Direct API call with curl (using gcloud for authentication)
curl -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://compute.googleapis.com/compute/v1/projects/my-project/zones/us-central1-a/instances"

# API discovery document
curl "https://www.googleapis.com/discovery/v1/apis/compute/v1/rest"
```

#### Authentication Patterns

**Application Default Credentials (ADC)** -- the recommended authentication method:

```bash
# ADC resolution order:
# 1. GOOGLE_APPLICATION_CREDENTIALS environment variable (path to service account key JSON)
# 2. gcloud auth application-default credentials (~/.config/gcloud/application_default_credentials.json)
# 3. Attached service account (on GCE, GKE, Cloud Run, Cloud Functions)
# 4. Compute Engine default service account

# Set ADC for local development
gcloud auth application-default login

# Set ADC to use a service account key (NOT recommended for production)
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"

# On GCE/GKE/Cloud Run: ADC automatically uses the attached service account
# No configuration needed
```

```python
# Python: ADC is used automatically by client libraries
from google.cloud import storage

# No credentials needed -- ADC handles it
client = storage.Client()

# Explicitly specify credentials (override ADC)
from google.oauth2 import service_account
credentials = service_account.Credentials.from_service_account_file(
    'service-account-key.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)
client = storage.Client(credentials=credentials)
```

**OAuth 2.0 flows:**

| Flow | Use Case |
|------|----------|
| **Service account** | Server-to-server, no user context |
| **Authorized user** | CLI tools, local development |
| **Web server** | Web apps that act on behalf of users |
| **Workload Identity Federation** | External workloads (AWS, Azure, GitHub Actions) |

#### Rate Limiting and Error Handling

```python
# Python: Exponential backoff with google-cloud libraries
# Most client libraries handle retries automatically

# Manual retry with exponential backoff
import time
import random
from google.api_core import exceptions, retry

# Using the built-in retry decorator
@retry.Retry(
    initial=1.0,        # Initial delay in seconds
    maximum=60.0,       # Maximum delay
    multiplier=2.0,     # Delay multiplier
    deadline=300.0,     # Total timeout
    predicate=retry.if_exception_type(
        exceptions.ServiceUnavailable,
        exceptions.TooManyRequests,
    ),
)
def make_api_call():
    # Your API call here
    pass

# Manual exponential backoff pattern
def call_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except exceptions.TooManyRequests:
            if attempt == max_retries - 1:
                raise
            delay = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

**Pagination:**

```python
# Most list operations return paginated results
from google.cloud import storage

client = storage.Client()

# Client libraries handle pagination automatically with iterators
blobs = client.list_blobs('my-bucket', prefix='data/')
for blob in blobs:  # Automatically paginates
    print(blob.name)

# Manual pagination with page tokens (REST API)
import requests

url = "https://storage.googleapis.com/storage/v1/b/my-bucket/o"
params = {"maxResults": 100}
headers = {"Authorization": f"Bearer {access_token}"}

while True:
    response = requests.get(url, params=params, headers=headers)
    data = response.json()
    for item in data.get("items", []):
        print(item["name"])
    page_token = data.get("nextPageToken")
    if not page_token:
        break
    params["pageToken"] = page_token
```

#### API Quotas and Limits

```bash
# View quotas for a project
gcloud compute project-info describe --project=my-project --format="table(quotas.metric,quotas.usage,quotas.limit)"

# Request a quota increase
# Console: IAM & Admin > Quotas > Select quota > Edit Quotas
# Or via gcloud:
gcloud services quota update \
  --service=compute.googleapis.com \
  --consumer=projects/my-project \
  --metric=compute.googleapis.com/cpus \
  --value=100 \
  --unit=1/min/{project}

# Common quota types:
# - Rate quotas: Requests per minute/second (resets periodically)
# - Allocation quotas: Total resource count (e.g., max VMs, max IPs)
```

> **Exam tips:**
> - **ADC is the recommended auth method.** Never hardcode credentials. Client libraries use ADC automatically.
> - ADC resolution order: env var > gcloud default creds > attached service account. Know this for exam questions about "where does the app get credentials?"
> - **Workload Identity Federation** = authenticate external workloads (AWS, Azure, GitHub) without service account keys. If the question mentions cross-cloud auth, pick this.
> - **Exponential backoff** with jitter is required when handling `429 Too Many Requests` or `503 Service Unavailable` errors.
> - Client libraries handle pagination and retries automatically -- prefer them over raw REST calls.
> - **Rate quotas** reset periodically. **Allocation quotas** are hard limits that require increase requests.
> - Use `gcloud auth print-access-token` for quick API testing with curl, but NEVER in production scripts.

**Docs:** [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials) | [Client Libraries](https://cloud.google.com/apis/docs/cloud-client-libraries) | [Quotas](https://cloud.google.com/docs/quotas) | [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) | [API design guide](https://cloud.google.com/apis/design)

---

### Google API Client Libraries

#### Cloud Client Libraries (Recommended)

Cloud Client Libraries are the idiomatic, higher-level libraries. They provide:

- Language-specific conventions (Pythonic, idiomatic Java, etc.)
- Built-in retry logic and pagination
- ADC integration
- Type safety and IDE autocompletion

```bash
# Install Cloud Client Libraries
pip install google-cloud-storage        # Cloud Storage
pip install google-cloud-bigquery       # BigQuery
pip install google-cloud-pubsub         # Pub/Sub
pip install google-cloud-firestore      # Firestore
pip install google-cloud-spanner        # Spanner
pip install google-cloud-logging        # Cloud Logging
pip install google-cloud-monitoring     # Cloud Monitoring
pip install google-cloud-secret-manager # Secret Manager
```

```python
# Pub/Sub: Publish and subscribe
from google.cloud import pubsub_v1
from concurrent.futures import TimeoutError

# Publisher
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path("my-project", "my-topic")

future = publisher.publish(topic_path, b"My message", attribute1="value1")
print(f"Published: {future.result()}")

# Subscriber
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path("my-project", "my-sub")

def callback(message):
    print(f"Received: {message.data}")
    message.ack()

streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
try:
    streaming_pull_future.result(timeout=60)
except TimeoutError:
    streaming_pull_future.cancel()
    streaming_pull_future.result()
```

```python
# Secret Manager: Access secrets
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = f"projects/my-project/secrets/my-secret/versions/latest"
response = client.access_secret_version(request={"name": name})
secret_value = response.payload.data.decode("UTF-8")
```

```go
// Go: Cloud Storage example
package main

import (
    "context"
    "fmt"
    "io"

    "cloud.google.com/go/storage"
)

func readObject(bucket, object string) ([]byte, error) {
    ctx := context.Background()
    client, err := storage.NewClient(ctx)  // Uses ADC
    if err != nil {
        return nil, fmt.Errorf("storage.NewClient: %w", err)
    }
    defer client.Close()

    rc, err := client.Bucket(bucket).Object(object).NewReader(ctx)
    if err != nil {
        return nil, fmt.Errorf("Object.NewReader: %w", err)
    }
    defer rc.Close()

    data, err := io.ReadAll(rc)
    if err != nil {
        return nil, fmt.Errorf("io.ReadAll: %w", err)
    }
    return data, nil
}
```

#### Google API Client Libraries (Lower-Level)

These are auto-generated from Google's API discovery documents. Use them only when a Cloud Client Library doesn't exist for a service.

```python
# Google API Client Library (lower-level)
from googleapiclient.discovery import build
from google.oauth2 import service_account

# Build the service object
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/compute']
)
compute = build('compute', 'v1', credentials=credentials)

# List instances
result = compute.instances().list(
    project='my-project',
    zone='us-central1-a'
).execute()

for instance in result.get('items', []):
    print(instance['name'])
```

#### When to Use Which Library

| Criteria | Cloud Client Libraries | Google API Client Libraries |
|----------|----------------------|---------------------------|
| **New project** | Always use these | Only if Cloud Client Library unavailable |
| **Retry/pagination** | Built-in | Manual |
| **Authentication** | ADC built-in | Requires explicit credential setup |
| **Type safety** | Yes | Limited |
| **API coverage** | Most popular services | All Google APIs |
| **Installation** | `google-cloud-*` packages | `google-api-python-client` |

#### Client Library Authentication Patterns

```python
# Pattern 1: ADC (recommended -- works everywhere)
from google.cloud import storage
client = storage.Client()  # Just works

# Pattern 2: Explicit service account key (avoid in production)
from google.cloud import storage
client = storage.Client.from_service_account_json('key.json')

# Pattern 3: Impersonated credentials
from google.auth import impersonated_credentials
import google.auth

source_credentials, _ = google.auth.default()
target_credentials = impersonated_credentials.Credentials(
    source_credentials=source_credentials,
    target_principal='target-sa@project.iam.gserviceaccount.com',
    target_scopes=['https://www.googleapis.com/auth/cloud-platform']
)
client = storage.Client(credentials=target_credentials)

# Pattern 4: Workload Identity Federation (external workloads)
# Configure a credential config file, then:
# export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credential-config.json"
# Client libraries automatically use the federated identity
```

> **Exam tips:**
> - **Always prefer Cloud Client Libraries** (`google-cloud-*`) over Google API Client Libraries (`google-api-python-client`).
> - Cloud Client Libraries handle retries, pagination, and ADC automatically. Google API Client Libraries require manual handling.
> - For the exam, know the installation pattern: `pip install google-cloud-{service}` for Python.
> - **Service account impersonation** is the recommended pattern when a human user needs to act as a service account. Avoids downloading keys.
> - **Workload Identity Federation** replaces service account keys for external workloads. No keys to rotate or leak.
> - All client libraries support both synchronous and asynchronous operations. Python has `grpc.aio` variants.

**Docs:** [Cloud Client Libraries](https://cloud.google.com/apis/docs/cloud-client-libraries) | [Google API Client Libraries](https://developers.google.com/api-client-library) | [Authentication overview](https://cloud.google.com/docs/authentication) | [Service account impersonation](https://cloud.google.com/iam/docs/create-short-lived-credentials-direct)

---

## Quick Reference: Section 5 Decision Tree

```
Need to manage APIs for external developers?
├── Yes, with monetization/portal → Apigee
├── Yes, simple serverless backend → API Gateway
└── Yes, gRPC microservices → Cloud Endpoints

Need to migrate data?
├── Database to Cloud SQL/AlloyDB → Database Migration Service
├── Database CDC to BigQuery → Datastream
├── Files from S3/Azure/on-prem → Storage Transfer Service
├── SaaS data to BigQuery → BigQuery Data Transfer Service
├── VMs to Compute Engine → Migrate to VMs
├── VMs to containers → Migrate to Containers
└── Need assessment first → Migration Center

Need Infrastructure as Code?
├── General purpose (recommended) → Terraform
├── Kubernetes-native team → Config Connector
├── Existing Deployment Manager → Migrate to Terraform
└── Prefer real programming languages → Pulumi

Need local development/testing?
├── Quick admin task → Cloud Shell
├── Full dev environment → Cloud Workstations
├── Test without real services → Cloud Emulators
└── IDE integration → Cloud Code

Need to authenticate?
├── GCP workload → Attached service account (ADC)
├── Local development → gcloud auth application-default login
├── External workload (AWS/Azure/GitHub) → Workload Identity Federation
└── CI/CD pipeline → Service account with key (or WIF)
```

---

## Section 5 Exam Checklist

- [ ] Can you choose between Apigee, API Gateway, and Cloud Endpoints for a given scenario?
- [ ] Do you know the migration tool for each type of migration (DB, VM, files, CDC)?
- [ ] Can you write a Terraform `backend "gcs"` block from memory?
- [ ] Do you know the `terraform import`, `state rm`, `state mv` commands?
- [ ] Can you explain ADC resolution order?
- [ ] Do you know which services have emulators and how to start them?
- [ ] Can you distinguish Cloud Shell vs Cloud Workstations?
- [ ] Do you know when to use Config Connector vs Terraform?
- [ ] Can you explain Workload Identity Federation vs service account keys?
- [ ] Do you know the difference between Cloud Client Libraries and Google API Client Libraries?
