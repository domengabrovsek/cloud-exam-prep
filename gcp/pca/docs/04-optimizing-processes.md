# Section 4: Analyzing and Optimizing Technical and Business Processes (~15% of Exam)

> **Quick context:** This section tests your ability to analyze both technical (CI/CD, testing, DR) and business processes (stakeholder management, cost optimization, change management). Expect 7-9 questions. The split is roughly 60% technical / 40% business. CI/CD pipelines, disaster recovery patterns, and cost optimization are the highest-yield topics.

---

## 4.1 Analyzing and Defining Technical Processes

### Software Development Lifecycle (SDLC)

The PCA exam expects you to understand how cloud-native development workflows operate on GCP, not just theory but which services map to each SDLC phase.

**Development Methodologies on GCP**

| Methodology | GCP Alignment | Key Services |
|-------------|---------------|--------------|
| **Agile/Scrum** | Short iterations, frequent deploys | Cloud Build triggers on push, Cloud Deploy for progressive rollouts |
| **DevOps** | Break silos between dev and ops | Cloud Build, Cloud Deploy, Cloud Monitoring, Error Reporting |
| **SRE** | Reliability engineering with error budgets | Cloud Monitoring SLOs, Alerting, Error Reporting, Cloud Trace |
| **GitOps** | Git as single source of truth for infra + app | Config Sync, Cloud Build triggers, Terraform in repos |

**Cloud-Native Development Workflow**

```
Local Dev
  → Cloud Workstations (managed dev environment)
    → Push to Source Repository (GitHub / GitLab / Secure Source Manager)
      → Cloud Build (CI: build, test, scan)
        → Artifact Registry (store images, packages)
          → Cloud Deploy (CD: promote through environments)
            → Target (GKE / Cloud Run / GCE)
```

> **Naming trap:** **Cloud Source Repositories** has been closed to new customers since 2024-06-17. An organization that never used it cannot enable the API. Google's managed source host for new work is [Secure Source Manager](https://cloud.google.com/secure-source-manager/docs/overview). Treat "Cloud Source Repositories" in an option as a signal the option was written against an older exam guide, unless the scenario explicitly describes an existing deployment.

**Cloud Workstations**

Cloud Workstations provides fully managed development environments running on Compute Engine, accessed through a browser-based IDE or local IDE via SSH.

```bash
# Create a workstation cluster
gcloud workstations clusters create my-cluster \
  --region=us-central1 \
  --network=projects/my-project/global/networks/my-vpc \
  --subnetwork=projects/my-project/regions/us-central1/subnetworks/my-subnet

# Create a workstation configuration
gcloud workstations configs create my-config \
  --cluster=my-cluster \
  --region=us-central1 \
  --machine-type=e2-standard-4 \
  --pd-disk-size=200 \
  --idle-timeout=7200s

# Create and start a workstation
gcloud workstations create my-workstation \
  --config=my-config \
  --cluster=my-cluster \
  --region=us-central1

gcloud workstations start my-workstation \
  --config=my-config \
  --cluster=my-cluster \
  --region=us-central1
```

**SRE Practices on GCP**

The exam tests whether you understand SRE principles and can map them to GCP services:

- **SLIs (Service Level Indicators):** Measurable metrics -- latency, error rate, throughput. Configured in Cloud Monitoring.
- **SLOs (Service Level Objectives):** Target values for SLIs (e.g., 99.9% availability). Set in Cloud Monitoring SLO monitoring.
- **SLAs (Service Level Agreements):** Business contracts with consequences. SLOs inform SLAs but are stricter internally.
- **Error Budgets:** The allowed amount of unreliability (100% - SLO). If error budget is exhausted, freeze feature releases and focus on reliability.
- **Toil Reduction:** Automate repetitive operational work. Use Cloud Functions, Cloud Scheduler, Workflows for automation.

```bash
# SLOs have no gcloud surface. They are created via the Monitoring API v3,
# Terraform, or the console. Terraform is the form worth knowing.
#
#   resource "google_monitoring_slo" "availability" {
#     service             = google_monitoring_custom_service.api.service_id
#     goal                = 0.999
#     rolling_period_days = 30
#     request_based_sli { good_total_ratio { ... } }
#   }
```

**Exam tips:**
- The exam loves questions that map SRE concepts to GCP services. Know that SLO monitoring lives in Cloud Monitoring, not a separate product.
- Error budgets are a PCA favorite: if a team has consumed their error budget, the correct answer is to halt feature releases and invest in reliability -- not to relax the SLO.
- Cloud Workstations is relevant for "secure development environment" scenarios where you need to keep source code off developer laptops.

**Docs:**
- [Cloud Workstations](https://cloud.google.com/workstations/docs)
- [SRE Workbook - Google](https://sre.google/workbook/table-of-contents/)
- [Cloud Monitoring SLOs](https://cloud.google.com/monitoring/slo)

---

### Continuous Integration and Continuous Delivery (CI/CD)

This is one of the most heavily tested topics in Section 4. You need to know Cloud Build, Artifact Registry, and Cloud Deploy in detail, including YAML configurations and deployment strategies.

#### Cloud Build

Cloud Build is a serverless CI/CD platform that executes build steps as containers. Every step runs in its own container, and steps can share data through the `/workspace` volume.

**Core Concepts:**
- **Build Config (cloudbuild.yaml):** Defines the build steps, images, and artifacts.
- **Triggers:** Automatically start builds on source code events (push, PR, tag).
- **Builder Images:** Pre-built containers for common tools (`gcr.io/cloud-builders/docker`, `gcr.io/cloud-builders/gcloud`, etc.).
- **Worker Pools:** Default (Google-managed) or private (customer-managed, runs in your VPC).
- **Substitutions:** Variables passed into builds (`$PROJECT_ID`, `$COMMIT_SHA`, `$BRANCH_NAME`, custom `$_MY_VAR`).

**Cloud Build YAML -- Basic Example**

```yaml
# cloudbuild.yaml
steps:
  # Step 1: Run unit tests
  - name: 'python:3.11'
    entrypoint: 'pip'
    args: ['install', '-r', 'requirements.txt']

  - name: 'python:3.11'
    entrypoint: 'python'
    args: ['-m', 'pytest', 'tests/', '-v']

  # Step 2: Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', '.']

  # Step 3: Push to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA']

  # Step 4: Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'my-service'
      - '--image=us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
      - '--region=us-central1'
      - '--platform=managed'

images:
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'

options:
  logging: CLOUD_LOGGING_ONLY
  machineType: 'E2_HIGHCPU_8'

timeout: '1200s'
```

**Cloud Build YAML -- Multi-Environment with Substitutions**

```yaml
# cloudbuild.yaml for promotion pattern
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'build'
      - '-t'
      - '${_REGION}-docker.pkg.dev/$PROJECT_ID/${_REPO}/${_IMAGE}:$COMMIT_SHA'
      - '-t'
      - '${_REGION}-docker.pkg.dev/$PROJECT_ID/${_REPO}/${_IMAGE}:latest'
      - '.'

  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '--all-tags', '${_REGION}-docker.pkg.dev/$PROJECT_ID/${_REPO}/${_IMAGE}']

  # Trigger Cloud Deploy release
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'deploy'
      - 'releases'
      - 'create'
      - 'release-$SHORT_SHA'
      - '--delivery-pipeline=${_PIPELINE}'
      - '--region=${_REGION}'
      - '--images=${_IMAGE}=${_REGION}-docker.pkg.dev/$PROJECT_ID/${_REPO}/${_IMAGE}:$COMMIT_SHA'

substitutions:
  _REGION: 'us-central1'
  _REPO: 'my-repo'
  _IMAGE: 'my-app'
  _PIPELINE: 'my-pipeline'

options:
  logging: CLOUD_LOGGING_ONLY
```

**Cloud Build YAML -- GKE Deployment**

```yaml
# cloudbuild.yaml for GKE
steps:
  # Build
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', '.']

  # Push
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA']

  # Deploy to GKE
  - name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - 'run'
      - '--filename=k8s/'
      - '--image=us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
      - '--location=us-central1'
      - '--cluster=my-cluster'
```

**Build Triggers**

```bash
# Create a trigger for GitHub pushes to main
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --name=deploy-to-prod

# Create a trigger for pull requests (runs tests only)
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --pull-request-pattern="^main$" \
  --build-config=cloudbuild-test.yaml \
  --name=pr-tests \
  --comment-control=COMMENTS_ENABLED

# Create a trigger for tags (release builds)
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --tag-pattern="^v[0-9]+\.[0-9]+\.[0-9]+$" \
  --build-config=cloudbuild-release.yaml \
  --name=release-build

# Create a trigger with substitutions
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --substitutions=_DEPLOY_ENV=production,_REGION=us-central1

# List triggers
gcloud builds triggers list

# Run a trigger manually
gcloud builds triggers run my-trigger --branch=main

# Submit a build manually (without a trigger)
gcloud builds submit --config=cloudbuild.yaml .

# Submit a build with a specific tag
gcloud builds submit --tag=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1.0 .
```

**Private Worker Pools**

Private pools run builds inside your VPC, enabling access to private resources (private Artifact Registry, internal APIs, databases).

```bash
# Create a private worker pool
gcloud builds worker-pools create my-pool \
  --region=us-central1 \
  --peered-network=projects/my-project/global/networks/my-vpc \
  --peered-network-ip-range=192.168.0.0/24 \
  --worker-machine-type=e2-standard-4 \
  --worker-disk-size=100

# List worker pools
gcloud builds worker-pools list --region=us-central1

# Use a private pool in cloudbuild.yaml
# Add to options:
#   pool:
#     name: 'projects/my-project/locations/us-central1/workerPools/my-pool'
```

**Using a private pool in cloudbuild.yaml:**

```yaml
options:
  pool:
    name: 'projects/my-project/locations/us-central1/workerPools/my-pool'

steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', '.']
```

**Exam tips:**
- Cloud Build steps run sequentially by default. Use `waitFor: ['-']` to run a step in parallel (it waits for nothing). Use `waitFor: ['step-id']` to create custom dependency chains.
- Which service account Cloud Build defaults to depends on when the project first built. Projects created after the mid-2024 change use the **Compute Engine default service account** (`PROJECT_NUMBER-compute@developer.gserviceaccount.com`); older projects keep the legacy Cloud Build service account (`PROJECT_NUMBER@cloudbuild.gserviceaccount.com`). Neither is a good answer -- both are over-privileged. For least privilege, create a dedicated service account and assign it to the trigger.
- `$PROJECT_ID`, `$COMMIT_SHA`, `$SHORT_SHA`, `$BRANCH_NAME`, `$TAG_NAME`, `$REPO_NAME`, and `$BUILD_ID` are built-in substitutions. Custom substitutions start with `_` (underscore).
- Private pools are the answer when a question says "build needs to access resources in a private VPC" or "build must not use public internet."
- `gcloud builds submit` is for ad-hoc builds. Triggers automate builds on source events.
- The `images` field at the top level of cloudbuild.yaml **is** the push: Cloud Build pushes the listed images to Artifact Registry after all steps succeed. An explicit `docker push` step is one way to do it, `images` is the other, and using both is merely redundant. Two consequences the exam can turn into a question: an image built during the run but not listed in `images` is **discarded** when the build finishes, and listing an image the build never produced **fails** the build.

**Docs:**
- [Cloud Build Overview](https://cloud.google.com/build/docs/overview)
- [Build Configuration File](https://cloud.google.com/build/docs/build-config-file-schema)
- [Build Triggers](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers)
- [Private Worker Pools](https://cloud.google.com/build/docs/private-pools/private-pools-overview)

---

#### Artifact Registry

Artifact Registry is the universal package manager for GCP. It replaces Container Registry (gcr.io) and supports multiple formats.

**Supported Formats:**
- Docker container images
- Language packages: Maven (Java), npm (Node.js), Python (pip), Go, Ruby (gems), PHP (Composer)
- OS packages: Apt (Debian), Yum (RPM)
- Helm charts
- KubeFlow pipelines

**Key Concepts:**
- **Repository:** A named collection of artifacts in a specific format, in a specific region.
- **Remote Repository:** A proxy/cache for upstream registries (Docker Hub, npm registry, Maven Central). Reduces external dependencies and improves reliability.
- **Virtual Repository:** A single endpoint that aggregates multiple repositories (local + remote). Clients point to one URL.

```bash
# Create a Docker repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Production container images"

# Create a Python repository
gcloud artifacts repositories create python-packages \
  --repository-format=python \
  --location=us-central1

# Create a remote repository (caches from Docker Hub)
gcloud artifacts repositories create dockerhub-cache \
  --repository-format=docker \
  --location=us-central1 \
  --mode=remote-repository \
  --remote-repo-config-desc="Docker Hub proxy" \
  --remote-docker-repo=DOCKER-HUB

# Create a virtual repository
gcloud artifacts repositories create virtual-docker \
  --repository-format=docker \
  --location=us-central1 \
  --mode=virtual-repository

# List repositories
gcloud artifacts repositories list --location=us-central1

# List images in a repository
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo

# List tags for a specific image
gcloud artifacts docker tags list us-central1-docker.pkg.dev/my-project/my-repo/my-app

# Delete an image
gcloud artifacts docker images delete \
  us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1.0

# Configure Docker authentication
gcloud auth configure-docker us-central1-docker.pkg.dev

# Set up cleanup policies (auto-delete old images)
gcloud artifacts repositories set-cleanup-policies my-repo \
  --location=us-central1 \
  --policy=cleanup-policy.json
```

**Vulnerability Scanning:**

Artifact Registry integrates with Artifact Analysis (formerly Container Analysis) for automated vulnerability scanning.

```bash
# Enable on-push scanning
gcloud artifacts repositories update my-repo \
  --location=us-central1 \
  --allow-vulnerability-scanning

# View scan results
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo \
  --show-occurrences \
  --occurrence-filter='kind="VULNERABILITY"'
```

**Binary Authorization Integration:**

Binary Authorization enforces deploy-time security policies, ensuring only trusted container images are deployed to GKE or Cloud Run.

```bash
# Create an attestor
gcloud container binauthz attestors create my-attestor \
  --attestation-authority-note=projects/my-project/notes/my-note \
  --attestation-authority-note-project=my-project

# Create an attestation (sign an image)
gcloud container binauthz attestations sign-and-create \
  --artifact-url=us-central1-docker.pkg.dev/my-project/my-repo/my-app@sha256:abc123 \
  --attestor=my-attestor \
  --keyversion=projects/my-project/locations/global/keyRings/my-ring/cryptoKeys/my-key/cryptoKeyVersions/1
```

**Exam tips:**
- Artifact Registry replaces Container Registry. Container Registry is shut down, not deprecated: writes stopped 2025-03-18, reads 2025-06-03. Your own images belong at `REGION-docker.pkg.dev/PROJECT/REPO/IMAGE`.
- **Not every `gcr.io` URL is stale.** Google-owned images such as the Cloud Build builders (`gcr.io/cloud-builders/docker`, `gcr.io/cloud-builders/gcloud`) keep their `gcr.io` addresses and are served from Artifact Registry. The shutdown applies to customer registries, so leave builder-image references alone.
- Remote repositories are the answer for "reduce dependency on external registries" or "cache public packages inside your organization."
- Virtual repositories are the answer for "developers need a single endpoint to pull from multiple repositories."
- Binary Authorization + Artifact Registry is the answer for "ensure only scanned and signed images are deployed."
- Cleanup policies prevent storage cost creep -- the exam may test this in cost optimization scenarios.

**Docs:**
- [Artifact Registry Overview](https://cloud.google.com/artifact-registry/docs/overview)
- [Remote Repositories](https://cloud.google.com/artifact-registry/docs/repositories/remote-repo)
- [Virtual Repositories](https://cloud.google.com/artifact-registry/docs/repositories/virtual-repo)
- [Binary Authorization](https://cloud.google.com/binary-authorization/docs)

---

#### Cloud Deploy

Cloud Deploy is a managed continuous delivery service that handles progressive delivery to GKE, Cloud Run, and GKE Enterprise (the product formerly marketed as Anthos).

**Core Concepts:**
- **Delivery Pipeline:** Defines the sequence of targets (environments) a release passes through.
- **Target:** A specific environment (e.g., dev, staging, prod). Maps to a GKE cluster, Cloud Run service, or GKE Enterprise cluster.
- **Release:** A specific version of your application. Immutable once created.
- **Rollout:** The act of deploying a release to a target. Can require approval.
- **Promotion:** Moving a release from one target to the next in the pipeline.

**Delivery Pipeline Configuration (delivery-pipeline.yaml):**

```yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-app-pipeline
description: "Production delivery pipeline"
serialPipeline:
  stages:
    - targetId: dev
      profiles: [dev]
      strategy:
        standard:
          verify: true
    - targetId: staging
      profiles: [staging]
      strategy:
        standard:
          verify: true
    - targetId: prod
      profiles: [prod]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              serviceNetworking:
                service: "my-app-service"
                deployment: "my-app"
          canaryDeployment:
            percentages: [10, 25, 50]
            verify: true
```

**Target Configuration (targets.yaml):**

```yaml
# Dev target
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: dev
description: "Development environment"
gke:
  cluster: projects/my-project/locations/us-central1/clusters/dev-cluster
  internalIp: true
---
# Staging target
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: staging
description: "Staging environment"
gke:
  cluster: projects/my-project/locations/us-central1/clusters/staging-cluster
requireApproval: true
---
# Production target (Cloud Run)
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
description: "Production environment"
run:
  location: projects/my-project/locations/us-central1
requireApproval: true
```

**Cloud Deploy gcloud Commands:**

```bash
# Register a delivery pipeline
gcloud deploy apply --file=delivery-pipeline.yaml --region=us-central1

# Register targets
gcloud deploy apply --file=targets.yaml --region=us-central1

# Create a release (kicks off the pipeline)
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1 \
  --images=my-app=us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1.0

# Promote a release to the next stage
gcloud deploy releases promote \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Approve a rollout (when requireApproval is true)
gcloud deploy rollouts approve rollout-001 \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --to-target=prod \
  --region=us-central1

# List releases
gcloud deploy releases list \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Roll a target back. Cloud Deploy creates a new rollout from the last known-good
# release on that target; --release and --rollout-id pick a different one.
gcloud deploy targets rollback prod \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Check rollout status
gcloud deploy rollouts list \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1
```

**Deployment Strategies in Cloud Deploy:**

| Strategy | How It Works | Risk | Rollback Speed | Use Case |
|----------|-------------|------|----------------|----------|
| **Standard (Rolling)** | Replace instances gradually | Medium | Medium | General workloads |
| **Blue-Green** | Deploy new version alongside old, switch traffic all at once | Low | Fast (switch back) | Need instant rollback |
| **Canary** | Route a percentage of traffic to new version, increase gradually | Low | Fast (route back) | Validate with real traffic |
| **Traffic Splitting** | Similar to canary but with explicit traffic percentages (Cloud Run) | Low | Fast | Cloud Run services |

**Canary Deployment Example for GKE:**

```yaml
strategy:
  canary:
    runtimeConfig:
      kubernetes:
        serviceNetworking:
          service: "my-app-service"
          deployment: "my-app"
    canaryDeployment:
      percentages: [10, 25, 50]
      verify: true
```

**Cloud Build + Cloud Deploy Integration Pattern:**

The typical end-to-end pipeline:

1. Developer pushes code to GitHub, GitLab or Secure Source Manager
2. Cloud Build trigger fires, runs tests, builds container image
3. Cloud Build pushes image to Artifact Registry
4. Cloud Build creates a Cloud Deploy release
5. Cloud Deploy rolls out to dev automatically
6. Manual promotion (with approval) to staging, then prod
7. Cloud Deploy executes canary or blue-green strategy in prod

```yaml
# cloudbuild.yaml -- full CI/CD integration
steps:
  # Test
  - name: 'golang:1.21'
    args: ['go', 'test', './...']

  # Build and push
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA']

  # Create Cloud Deploy release
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'deploy'
      - 'releases'
      - 'create'
      - 'rel-$SHORT_SHA'
      - '--delivery-pipeline=my-pipeline'
      - '--region=us-central1'
      - '--images=my-app=us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'

images:
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
```

**Jenkins / GitHub Actions Integration with GCP:**

For organizations using external CI/CD tools:

- **Jenkins on GCP:** Run Jenkins on GKE or Compute Engine. Use the Google OAuth plugin for authentication. Use the Kubernetes plugin for dynamic build agents on GKE. Jenkins triggers Cloud Deploy for the CD portion.
- **GitHub Actions:** Use `google-github-actions/auth` for Workload Identity Federation (preferred over service account keys). Use `google-github-actions/setup-gcloud` to configure gcloud CLI. Call `gcloud deploy releases create` in a workflow step.

**GitHub Actions Example:**

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write  # Required for Workload Identity Federation

    steps:
      - uses: actions/checkout@v4

      - id: auth
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123/locations/global/workloadIdentityPools/my-pool/providers/github'
          service_account: 'ci-cd@my-project.iam.gserviceaccount.com'

      - uses: google-github-actions/setup-gcloud@v2

      - name: Build and Push
        run: |
          gcloud auth configure-docker us-central1-docker.pkg.dev
          docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$GITHUB_SHA .
          docker push us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$GITHUB_SHA

      - name: Create Release
        run: |
          gcloud deploy releases create rel-${GITHUB_SHA::7} \
            --delivery-pipeline=my-pipeline \
            --region=us-central1 \
            --images=my-app=us-central1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$GITHUB_SHA
```

**Exam tips:**
- Cloud Deploy is for CD only -- it does not build or test. Cloud Build handles CI. This is a common distractor.
- `requireApproval: true` on a target means a human must approve before rollout proceeds. This is tested in "separation of duties" scenarios.
- Cloud Deploy has **native rollback**: `gcloud deploy targets rollback TARGET --delivery-pipeline=PIPELINE`. It creates a fresh rollout from the last known-good release on that target. Hand-rolling a new release from the old image is the manual fallback, not the mechanism.
- Rollback can also be **automated** via a `repairRolloutRule` automation: it retries the failed rollout, and if the retries are exhausted or none are configured it rolls back to the most recent successful release. The other automation rules are `promoteReleaseRule`, `timedPromoteRule` and `advanceRolloutRule`.
- GKE canary in Cloud Deploy does **not** require a service mesh. `serviceNetworking` uses plain Kubernetes Service selectors and splits by pod count; `gatewayServiceMesh` via the Gateway API is what you pick when you need true traffic-percentage splitting. For Cloud Run, traffic splitting is native.
- When the exam says "managed progressive delivery service," the answer is Cloud Deploy, not Spinnaker or Jenkins.
- Workload Identity Federation is the recommended way to authenticate from external CI/CD (GitHub Actions, GitLab CI). Never use exported service account keys.

**Docs:**
- [Cloud Deploy Overview](https://cloud.google.com/deploy/docs/overview)
- [Delivery Pipeline Configuration](https://cloud.google.com/deploy/docs/config-files)
- [Deployment Strategies](https://cloud.google.com/deploy/docs/deployment-strategies)
- [Roll back a target](https://cloud.google.com/deploy/docs/roll-back)
- [Automate your deployment](https://cloud.google.com/deploy/docs/automation)
- [Canary on GKE with service networking](https://cloud.google.com/deploy/docs/deployment-strategies/canary/gke/service-networking)
- [Cloud Build + Cloud Deploy](https://cloud.google.com/deploy/docs/integrating-ci)

---

### Troubleshooting / Root Cause Analysis

The exam tests your ability to choose the right observability tool for a given debugging scenario.

**Google Cloud Operations Suite (formerly Stackdriver):**

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **Cloud Monitoring** | Metrics, dashboards, alerting, uptime checks, SLOs | Performance monitoring, capacity planning, alerting on thresholds |
| **Cloud Logging** | Centralized log management, log-based metrics, log sinks | Debugging application errors, audit trails, compliance |
| **Cloud Trace** | Distributed tracing for request latency | Diagnosing slow requests across microservices |
| **Error Reporting** | Aggregates and deduplicates errors | Finding top errors, tracking error rates over time |
| **Cloud Profiler** | Continuous CPU/memory profiling in production | Identifying performance bottlenecks, memory leaks |

> **Retired, and therefore a distractor:** Cloud Debugger was shut down on 2023-05-31, and its open-source successor Snapshot Debugger was archived on 2023-09-07. Neither is a live product. Production state inspection is now a logging and tracing problem: structured logs, Cloud Trace spans, and Cloud Profiler.

**Systematic Debugging Approach:**

1. **Check Monitoring dashboards** -- Is there a resource bottleneck (CPU, memory, disk, network)?
2. **Review Error Reporting** -- Are there new or spiking errors?
3. **Search Cloud Logging** -- Filter by severity, resource, time range. Use log-based metrics for trends.
4. **Trace slow requests** -- Use Cloud Trace to find latency hotspots in distributed systems.
5. **Profile the application** -- Use Cloud Profiler for CPU/memory analysis.
6. **Check audit logs** -- Admin Activity and Data Access logs show who changed what.

**Log-Based Metrics:**

```bash
# Create a log-based metric (counts matching log entries)
gcloud logging metrics create error-count \
  --description="Count of application errors" \
  --log-filter='severity>=ERROR AND resource.type="k8s_container"'

# Distribution metrics cannot be defined with flags. The flag form only accepts
# --description and --log-filter; anything with a value extractor, bucket options
# or user-defined labels goes through a config file.
gcloud logging metrics create response-latency \
  --config-from-file=response-latency.yaml
```

**Log Sinks (Routing):**

```bash
# Route logs to Cloud Storage (long-term archival)
gcloud logging sinks create archive-sink \
  storage.googleapis.com/my-log-bucket \
  --log-filter='resource.type="gce_instance"'

# Route logs to BigQuery (analysis)
gcloud logging sinks create bq-sink \
  bigquery.googleapis.com/projects/my-project/datasets/logs \
  --log-filter='severity>=WARNING'

# Route logs to Pub/Sub (real-time processing)
gcloud logging sinks create pubsub-sink \
  pubsub.googleapis.com/projects/my-project/topics/log-events \
  --log-filter='protoPayload.methodName="SetIamPolicy"'
```

**Distributed Tracing for Microservices:**

Cloud Trace automatically captures traces for many Google Cloud services. For custom applications, instrument with OpenTelemetry.

Key concepts:
- **Trace:** The entire journey of a request through the system.
- **Span:** A single operation within a trace (e.g., a database query, an HTTP call).
- **Trace context propagation:** Headers (`traceparent` for W3C, `X-Cloud-Trace-Context` for Google) must be forwarded between services.

For GKE microservices, enable tracing via:
- Managed service mesh (Cloud Service Mesh, which absorbed Anthos Service Mesh and Traffic Director) for automatic tracing
- OpenTelemetry SDK for application-level instrumentation
- Cloud Trace API for direct integration

**Exam tips:**
- Cloud Trace is the answer for "identify which microservice is causing latency."
- Error Reporting is the answer for "aggregate and group application errors."
- Cloud Profiler is the answer for "identify CPU or memory bottlenecks in production without impacting performance."
- Log sinks are the answer for "export logs to BigQuery for analysis" or "archive logs to Cloud Storage."
- Log-based metrics + alerting policies = "alert when a specific error pattern occurs in logs."
- Audit logs are always on for Admin Activity (free). Data Access logs must be enabled and can be expensive.

**Docs:**
- [Cloud Monitoring](https://cloud.google.com/monitoring/docs)
- [Cloud Logging](https://cloud.google.com/logging/docs)
- [Cloud Trace](https://cloud.google.com/trace/docs)
- [Error Reporting](https://cloud.google.com/error-reporting/docs)
- [Cloud Profiler](https://cloud.google.com/profiler/docs)

---

### Testing and Validation

The PCA exam tests whether you know the types of testing and which GCP services support them.

**Testing Pyramid on GCP:**

```
         ╱  E2E Tests  ╲         ← Fewest, slowest, most expensive
        ╱  Integration   ╲       ← Test service interactions
       ╱   Unit Tests     ╲      ← Most numerous, fastest, cheapest
```

**Types of Testing:**

| Test Type | What It Validates | GCP Tools |
|-----------|-------------------|-----------|
| **Unit Tests** | Individual functions/methods | Run in Cloud Build steps |
| **Integration Tests** | Service-to-service interactions | Cloud Build + test environment |
| **End-to-End (E2E) Tests** | Full user workflows | Cloud Build + test deployment |
| **Load/Performance Tests** | System behavior under load | Distributed load testing on GKE (Locust, k6, JMeter) |
| **Security Scanning** | Vulnerabilities in images and code | Artifact Analysis, Web Security Scanner |
| **Chaos Engineering** | System resilience to failures | Fault injection in Cloud Service Mesh, Gremlin |
| **Infrastructure Validation** | IaC correctness | `terraform validate`, `terraform plan`, Sentinel/OPA |

**Load Testing:**

> **There is no Google Cloud product called "Cloud Load Testing."** Google's answer to load testing is a reference architecture, not a service: [Distributed load testing using GKE](https://cloud.google.com/architecture/distributed-load-testing-using-gke), which runs an open-source generator (Locust in the guide; k6 and JMeter follow the same shape) as a master-and-workers deployment on a GKE cluster you size to the load you need. An option naming a managed Google load-testing service is fabricated.

```bash
# Run the load generator from a Cloud Build step against a deployed service
# cloudbuild.yaml step:
steps:
  - name: 'python:3.11'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        pip install locust
        locust -f locustfile.py --headless -u 1000 -r 100 \
          --host=https://my-app-xyz.run.app \
          --run-time=5m \
          --csv=results
```

**Chaos Engineering:**

Chaos engineering on GCP involves deliberately introducing failures to test resilience:

- **Fault injection with Cloud Service Mesh:** Inject delays and aborts at the mesh level.
- **Network disruption:** Use VPC firewall rules to simulate network partitions.
- **Instance termination:** Delete instances to test auto-healing in MIGs.
- **Zone failure simulation:** Drain a zone's instances to test multi-zone resilience.
- **Game Days:** Scheduled team exercises where failure scenarios are executed and the team practices incident response.

**Infrastructure Validation:**

```bash
# Terraform validation workflow
terraform init
terraform validate                    # Syntax and configuration check
terraform plan -out=plan.tfplan       # Preview changes
terraform apply plan.tfplan           # Apply only the reviewed plan

# Policy-as-code with OPA/Gatekeeper on GKE
# Define constraints that prevent non-compliant resources
# Example: Require all containers to have resource limits
```

**Policy-as-Code with GKE:**

GKE Policy Controller (based on Open Policy Agent/Gatekeeper) enforces constraints on Kubernetes resources:

```yaml
# Example: Require all pods to have resource limits
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    limits:
      - cpu
      - memory
```

**Exam tips:**
- The testing pyramid is a conceptual favorite: unit tests are fast/cheap, E2E tests are slow/expensive.
- "How to ensure only compliant infrastructure is deployed?" -- Policy-as-code (OPA/Gatekeeper, Sentinel, or organization policies).
- "How to test if application handles zone failures?" -- Chaos engineering / Game Days.
- Cloud Build should run tests before building images. A failed test step should fail the entire build.
- `terraform plan` output should be reviewed before `terraform apply` -- this is a governance control.

**Docs:**
- [Distributed load testing using GKE](https://cloud.google.com/architecture/distributed-load-testing-using-gke)
- [GKE Policy Controller](https://cloud.google.com/kubernetes-engine/enterprise/policy-controller/docs/overview)
- [Web Security Scanner](https://cloud.google.com/security-command-center/docs/concepts-web-security-scanner-overview)

---

### Service Catalog and Provisioning

Service Catalog enables organizations to create curated, pre-approved solutions that teams can self-service deploy.

**Service Catalog (renamed from Private Catalog in 2022)**

Service Catalog lets platform teams publish curated solutions that other teams in the organization can discover and deploy through a controlled interface.

**Key Concepts:**
- **Catalog:** A collection of solutions created under the organization and shared with specific projects or folders.
- **Solution:** A deployable package. The supported types are **Terraform configurations** and **reference links** to verified external content. Deployment Manager templates were the original type, but Deployment Manager reached end of support on 2026-03-31 and new users have been blocked since 2026-06-30, so **Terraform is the only live authoring path**. [Infrastructure Manager](https://cloud.google.com/infrastructure-manager/docs/overview) is Google's named replacement for Deployment Manager.
- **Version:** Each solution can have multiple versions for controlled rollouts.

```bash
# The gcloud `privatecatalog` surface is search-only. Catalogs and solutions are
# created in the console or through the Service Catalog API. The product was
# renamed from Private Catalog to Service Catalog in 2022.

# Search the catalogs available to you
gcloud privatecatalog products search --query="web application"
```

**Terraform Configurations as Service Catalog Entries:**

Platform teams create standardized Terraform modules that enforce organizational standards:

```hcl
# Example: Standard GKE cluster module
module "standard_gke" {
  source = "terraform-google-modules/kubernetes-engine/google"

  project_id         = var.project_id
  name               = var.cluster_name
  region             = var.region
  network            = var.network
  subnetwork         = var.subnetwork

  # Enforced standards
  release_channel    = "REGULAR"
  enable_shielded_nodes = true
  enable_private_nodes  = true
  master_authorized_networks = var.authorized_networks

  node_pools = [
    {
      name           = "default-pool"
      machine_type   = "e2-standard-4"
      min_count      = 1
      max_count      = 10
      auto_upgrade   = true
      auto_repair    = true
    }
  ]
}
```

**Exam tips:**
- Service Catalog is the answer when the scenario asks for "self-service deployment of standardized solutions" or "curated catalog of approved architectures." The exam guide and older material may still call it Private Catalog -- same product.
- The exam tests that you understand the difference between Service Catalog (internal, curated, approved by your own platform team) and Marketplace (third-party or Google solutions).
- Terraform modules in a centralized repository are a common pattern for standardization without Service Catalog.
- Any option that proposes publishing **Deployment Manager** templates is dated. Deployment Manager is past end of support; Infrastructure Manager (managed Terraform) is the successor.

**Docs:**
- [Service Catalog](https://cloud.google.com/service-catalog/docs)
- [Infrastructure Manager](https://cloud.google.com/infrastructure-manager/docs/overview)
- [Deployment Manager deprecation](https://cloud.google.com/deployment-manager/docs/deprecations)
- [Terraform Google Modules](https://registry.terraform.io/namespaces/terraform-google-modules)

---

### Disaster Recovery

DR is one of the most frequently tested topics in this section. You must be able to match DR patterns to business requirements (RTO/RPO) and cost constraints.

**Key Definitions:**
- **RTO (Recovery Time Objective):** Maximum acceptable downtime. "How long can we be down?"
- **RPO (Recovery Point Objective):** Maximum acceptable data loss. "How much data can we lose?"
- **RTO and RPO are inversely related to cost:** Lower RTO/RPO = higher cost.

**DR Patterns Comparison:**

| Pattern | Description | RPO | RTO | Relative Cost | GCP Implementation |
|---------|-------------|-----|-----|---------------|-------------------|
| **Backup & Restore (Cold)** | Data backed up to another region; infrastructure provisioned on demand | Hours | Hours to days | Lowest ($) | Cloud Storage cross-region, scheduled backups, Terraform for infra |
| **Cold Standby** | Data replicated; minimal infrastructure pre-provisioned but not running | Hours | Hours | Low ($$) | Cloud Storage replication, stopped VMs with boot disks, database backups |
| **Warm Standby** | Scaled-down copy of production running in DR region | Minutes | Minutes to hours | Medium ($$$) | Smaller GKE cluster, read replicas promoted, reduced instance counts |
| **Hot Standby** | Fully scaled duplicate in DR region, ready to take traffic | Near-zero | Minutes | High ($$$$) | Multi-region database, duplicate GKE clusters, Global LB failover |
| **Multi-Region Active-Active** | Both regions actively serving traffic simultaneously | Zero (sync replication) | Near-zero (automatic) | Highest ($$$$$) | Spanner, Global HTTP(S) LB, multi-cluster GKE, Cloud CDN |

**DR for Each Service Type:**

**Compute (GCE, GKE, Cloud Run):**

| Service | DR Strategy | Implementation |
|---------|-------------|----------------|
| **Compute Engine** | Machine images, snapshots | Schedule snapshots cross-region; use instance templates + MIG for fast recovery |
| **GKE** | Multi-cluster, backup/restore | Backup for GKE for cluster state; GKE Enterprise fleets for multi-cluster management |
| **Cloud Run** | Multi-region deployment | Deploy to multiple regions behind Global LB; traffic auto-failover |

```bash
# Create a snapshot schedule for DR
gcloud compute resource-policies create snapshot-schedule dr-schedule \
  --region=us-central1 \
  --max-retention-days=14 \
  --start-time=02:00 \
  --hourly-schedule=4 \
  --storage-location=us-east1

# Attach schedule to a disk
gcloud compute disks add-resource-policies my-disk \
  --resource-policies=dr-schedule \
  --zone=us-central1-a

# Create a machine image (full VM capture including config)
gcloud compute machine-images create my-vm-backup \
  --source-instance=my-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us-east1

# GKE Backup -- create a backup plan
gcloud container backup-restore backup-plans create my-backup-plan \
  --project=my-project \
  --location=us-central1 \
  --cluster=projects/my-project/locations/us-central1/clusters/my-cluster \
  --all-namespaces \
  --include-volume-data \
  --cron-schedule="0 2 * * *" \
  --backup-retain-days=30
```

**Databases:**

| Database | DR Strategy | RPO | Notes |
|----------|-------------|-----|-------|
| **Cloud SQL** | Cross-region read replicas, PITR | Minutes (async replication) | Promote replica to primary; PITR for point-in-time recovery |
| **Cloud Spanner** | Multi-region config (built-in) | Zero (sync replication) | `nam-eur-asia1`, `nam6`, `eur6` configs are inherently DR-ready |
| **Firestore** | Multi-region location | Zero (sync replication within location) | Choose `nam5` or `eur3` for multi-region |
| **Bigtable** | Cross-region replication | Minutes (async) | Add a cluster in DR region; automatic failover with app profiles |
| **AlloyDB** | Cross-region replication | Minutes (async) | Up to five secondary clusters in other regions; promote for DR, or switchover with zero data loss for a planned move |
| **Memorystore for Redis** (non-cluster) | Cross-**zone** replication only | Seconds within the region | Standard tier places replicas in other zones of the same region. It is HA, not DR |
| **Memorystore for Redis Cluster / Valkey** | Cross-region replication | Minutes (async) | Secondary clusters in another region, with switchover. This is the cross-region answer |

```bash
# Cloud SQL: Create a cross-region read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-primary \
  --region=us-east1 \
  --tier=db-custom-4-16384 \
  --availability-type=REGIONAL

# Cloud SQL: Promote replica to primary (DR failover)
gcloud sql instances promote-replica my-replica

# Cloud SQL: Enable point-in-time recovery (PITR)
gcloud sql instances patch my-instance \
  --enable-point-in-time-recovery \
  --retained-transaction-log-days=7

# Bigtable: Add a cluster in another region (replication)
gcloud bigtable clusters create dr-cluster \
  --instance=my-instance \
  --zone=us-east1-b \
  --num-nodes=3
```

**Storage:**

| Storage | DR Strategy | Implementation |
|---------|-------------|----------------|
| **Cloud Storage** | Dual-region or multi-region bucket | Use `nam4`, `eur4`, or `us`/`eu`/`asia` locations |
| **Cloud Storage** | Cross-region replication (single-region) | Transfer Service or Storage Transfer Service |
| **Filestore** | Backup to another region | Filestore backups, cross-region restore |
| **Persistent Disk** | Snapshot to another region | Scheduled snapshots with cross-region storage |

```bash
# Create a dual-region bucket
gcloud storage buckets create gs://my-dr-bucket \
  --location=nam4 \
  --default-storage-class=STANDARD

# Create a multi-region bucket
gcloud storage buckets create gs://my-global-bucket \
  --location=us \
  --default-storage-class=STANDARD
```

**DR Testing Methodology:**

1. **Document runbooks** -- Step-by-step failover and failback procedures for each service.
2. **Tabletop exercises** -- Walk through DR scenarios as a team without actually executing.
3. **Simulation testing** -- Execute failover in a non-production environment.
4. **Full DR test** -- Perform actual failover in production during a maintenance window.
5. **Game Days** -- Unannounced (or semi-announced) failure injection to test team readiness.

**DR Test Schedule (Recommended):**
- Tabletop: Quarterly
- Simulation: Semi-annually
- Full DR test: Annually

**Google Cloud Backup and DR Service:**

A managed backup and disaster recovery service that provides:
- Application-consistent backups for Compute Engine, GKE, Cloud SQL, and VMware
- Centralized backup management
- Backup vault for immutable, tamper-proof backup storage
- Cross-region and cross-project backup copies

```bash
# Backup and DR is primarily managed through the Console
# Key components:
# - Backup vault: Immutable storage for backups
# - Backup plan: Schedule and retention policy
# - Management server: Coordinates backup operations

# The service integrates with:
# - Compute Engine (disk-level and application-consistent backups)
# - GKE (namespace and PV backups)
# - Cloud SQL (automated backups with cross-region copy)
```

**Exam tips:**
- **RTO/RPO drives architecture, cost drives constraints.** The exam gives you both and expects you to pick the cheapest pattern that meets the requirements.
- "Near-zero RPO" for databases = multi-region synchronous replication (Spanner, Firestore multi-region). Async replication (Cloud SQL replicas, Bigtable) has minutes of RPO.
- "Minimize RTO for Compute Engine" = Machine images + instance templates + MIG in DR region, pre-created but scaled to zero or minimum.
- "DR for a stateless web app" = Multi-region Cloud Run or GKE behind Global HTTP(S) LB. The simplest and cheapest approach for stateless workloads.
- Cloud SQL failover to a replica is NOT automatic (you must promote). Cloud SQL HA (regional) with automatic failover is within a region, not cross-region.
- Spanner multi-region is the most expensive DR solution but provides the strongest guarantees (zero RPO, automatic failover).
- **Trap:** "Cold standby" does NOT mean infrastructure is running. It means data is replicated and infrastructure is defined (e.g., in Terraform) but not provisioned.
- Machine images capture everything (boot disk, additional disks, machine config, metadata). Snapshots capture only disk data. For DR, machine images are more complete.
- Backup for GKE is the managed solution for backing up Kubernetes workloads (namespaces, PVs, cluster configuration).
- **Memorystore trap:** Standard tier on Memorystore for Redis replicates across **zones**, not regions. If a scenario needs a cache available after a regional outage, the answer is Memorystore for Redis Cluster or Valkey with a secondary cluster, or rebuilding the cache from the source of truth.
- AlloyDB is not single-region. It supports cross-region secondary clusters, promotion for DR and zero-data-loss switchover -- do not eliminate it on a cross-region requirement.

**Docs:**
- [DR Planning Guide](https://cloud.google.com/architecture/dr-scenarios-planning-guide)
- [Memorystore cross-region replication](https://cloud.google.com/memorystore/docs/cluster/about-cross-region-replication)
- [AlloyDB cross-region replication](https://cloud.google.com/alloydb/docs/cross-region-replication/about-cross-region-replication)
- [DR for Data](https://cloud.google.com/architecture/dr-scenarios-for-data)
- [DR for Applications](https://cloud.google.com/architecture/dr-scenarios-for-applications)
- [Cloud SQL HA and DR](https://cloud.google.com/sql/docs/mysql/high-availability)
- [GKE Backup](https://cloud.google.com/kubernetes-engine/docs/add-on/backup-for-gke/concepts/backup-for-gke)
- [Backup and DR Service](https://cloud.google.com/backup-disaster-recovery/docs)

---

## 4.2 Analyzing and Defining Business Processes

This section focuses on the non-technical aspects of cloud architecture: how to manage stakeholders, drive adoption, optimize costs, and ensure business continuity. The exam tests practical knowledge, not abstract management theory.

---

### Stakeholder Management

Architects must translate technical decisions into business value and navigate organizational dynamics.

**Key Skills Tested:**

| Skill | What the Exam Tests |
|-------|---------------------|
| **Influencing** | Recommending architecture decisions with business justification, not just technical merit |
| **Facilitation** | Leading design reviews, architecture meetings, trade-off discussions |
| **Translation** | Converting technical constraints (latency, throughput) into business terms (user experience, revenue impact) |

**Working with Different Stakeholders:**

| Stakeholder | What They Care About | How to Communicate |
|-------------|---------------------|-------------------|
| **Executives (CTO, CIO)** | Cost, risk, time-to-market, strategic alignment | Business outcomes, ROI, competitive advantage |
| **Developers** | Productivity, tooling, autonomy, deployment speed | Technical benefits, developer experience improvements |
| **Operations/SRE** | Reliability, observability, automation, on-call burden | SLOs, reduced toil, automated remediation |
| **Security/Compliance** | Risk, audit trails, data protection, regulatory compliance | Security controls, compliance certifications, encryption |
| **Finance** | Cost predictability, budget alignment, CapEx vs OpEx | TCO analysis, billing forecasts, committed use savings |

**Cloud Center of Excellence (CCoE):**

A CCoE is a cross-functional team that establishes cloud standards, best practices, and governance. It typically includes:
- Cloud architects (technical leadership)
- Security and compliance representatives
- Finance/FinOps specialists
- Developer advocates (drive adoption)

CCoE responsibilities:
- Define reference architectures and landing zones
- Establish governance policies (org policies, guardrails)
- Create shared services (networking, security baselines)
- Provide training and enablement
- Review and approve non-standard architecture requests

**Exam tips:**
- The exam may present a scenario where teams resist cloud adoption. The answer usually involves a CCoE or phased migration approach, not mandating immediate migration.
- When a question asks "how should the architect present the recommendation," the answer that includes business justification (cost savings, improved reliability) alongside the technical solution is usually correct.
- "Architect disagrees with the security team's requirement" -- the answer is never to override them. It is to present alternatives that meet both security and business requirements.

**Docs:**
- [Cloud Adoption Framework](https://cloud.google.com/adoption-framework)
- [Cloud Center of Excellence](https://cloud.google.com/architecture/cloud-center-of-excellence)

---

### Change Management

Cloud adoption is an organizational transformation, not just a technology migration. The exam tests whether you understand this.

**Organizational Change for Cloud Adoption:**

| Phase | Activities | GCP Relevance |
|-------|-----------|---------------|
| **Awareness** | Communicate why cloud, executive sponsorship | Business case, TCO comparison |
| **Education** | Training programs, hands-on labs | Google Cloud Skills Boost, partner training |
| **Pilot** | Small, low-risk workloads in cloud | Proof of concept projects, sandbox environments |
| **Scale** | Migrate more workloads, expand teams | Landing zones, automated provisioning |
| **Optimize** | Cost optimization, advanced services | FinOps, ML/AI services, managed databases |

**Phased Migration Approach:**

```
Phase 1: Foundation (Months 1-3)
  ├── Set up organization, folders, projects
  ├── Establish networking (Shared VPC, Interconnect)
  ├── Define IAM structure and policies
  └── Create CI/CD pipelines

Phase 2: Migrate (Months 3-12)
  ├── Start with non-critical workloads
  ├── Lift-and-shift first, modernize later
  ├── Run in parallel (hybrid) during transition
  └── Validate performance and compliance

Phase 3: Optimize (Ongoing)
  ├── Modernize: containers, serverless, managed services
  ├── Cost optimization: right-sizing, CUDs, autoscaling
  ├── Advanced services: ML, BigQuery, Pub/Sub
  └── Continuous improvement cycle
```

**Exam tips:**
- "Organization is new to cloud. What is the first step?" -- Establish foundations (org hierarchy, networking, IAM), not migrate workloads.
- "Team is resisting cloud migration." -- Training and upskilling, pilot projects for quick wins, CCoE for governance. Never force immediate migration.
- Phased migration (lift-and-shift first, modernize later) is almost always preferred over big-bang modernization.

**Docs:**
- [Cloud Adoption Framework](https://cloud.google.com/adoption-framework)
- [Migration to Google Cloud](https://cloud.google.com/architecture/migration-to-gcp-getting-started)

---

### Team Assessment / Skills Readiness

The exam may ask how to prepare an organization's workforce for cloud adoption.

**Identifying Skill Gaps:**

| Current Skill | Cloud Equivalent | Training Path |
|--------------|-----------------|--------------|
| On-prem networking | VPC, Cloud Interconnect, Cloud DNS | Network Engineer learning path |
| VMware administration | Compute Engine, GKE | Cloud Engineer learning path |
| Oracle/SQL Server DBA | Cloud SQL, AlloyDB, Spanner | Database Engineer learning path |
| Traditional ITSM | SRE practices, Cloud Monitoring | SRE learning path |
| On-prem security | IAM, VPC SC, Security Command Center | Security Engineer learning path |

**Training Resources:**

| Resource | Description |
|----------|-------------|
| **Google Cloud Skills Boost** | Hands-on labs, learning paths, skill badges. Official Google training platform. |
| **Google Cloud Certifications** | Structured validation: ACE, PCA, PDE, PCSE, etc. |
| **Qwiklabs / CloudSkillsBoost** | Hands-on labs in real GCP environments. |
| **Partner Training** | Google Cloud Partners deliver instructor-led training. |
| **Architecture Center** | Reference architectures, best practices, design patterns. |

**Exam tips:**
- The correct answer for "how to upskill a team" is Google Cloud Skills Boost (hands-on labs), not just documentation.
- Certification paths should align with job roles: operators get ACE, architects get PCA, data engineers get PDE.
- When the question mentions "team lacks Kubernetes experience," the answer may involve managed services (Cloud Run, App Engine) instead of GKE to reduce the skill gap.

**Docs:**
- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [Google Cloud Certifications](https://cloud.google.com/learn/certification)
- [Architecture Center](https://cloud.google.com/architecture)

---

### Decision-Making Processes

Architects must make and document decisions in a structured, defensible way.

**Architecture Decision Records (ADRs):**

ADRs document the context, decision, and consequences of significant architecture choices. They create a history of why decisions were made.

**ADR structure:**

| Section | What it holds |
|---------|--------------|
| **Title** | The decision as a statement, numbered for reference |
| **Status** | Proposed, Accepted, or Superseded by a later ADR. A reversed decision creates a new ADR rather than editing the old one |
| **Context** | The forces in play: requirements, constraints, what the current design cannot do |
| **Decision** | The choice made, stated in one or two sentences |
| **Consequences** | What this buys, and what it costs. The negative half is the part that earns the document its keep |
| **Alternatives considered** | Each option and the specific reason it lost |

**Worked example, ADR-001 "Use Cloud Spanner for global transaction processing":**

- **Status:** Accepted
- **Context:** Globally consistent transactions with sub-10ms reads. The current PostgreSQL deployment cannot scale past one region. Multi-region active-active with zero RPO is required.
- **Decision:** Cloud Spanner in a multi-region configuration.
- **Consequences, positive:** zero RPO with automatic failover; global consistency without application-level conflict resolution; horizontal scale without manual sharding.
- **Consequences, negative:** materially higher cost than Cloud SQL; schema design changes (interleaved tables, key choice to avoid hotspotting); the team needs Spanner-specific training.
- **Alternatives considered:** CockroachDB on GKE, rejected for operational burden. Cloud SQL with cross-region read replicas, rejected because failover is a manual promotion with an async RPO. AlloyDB, rejected because the requirement is synchronous multi-region writes, which its cross-region secondary clusters do not provide -- note that AlloyDB **is** cross-region capable, so "single-region only" would be the wrong reason to reject it.

**Build vs Buy Analysis:**

| Factor | Build (Custom) | Buy (Managed Service) |
|--------|---------------|----------------------|
| **Time to market** | Slow | Fast |
| **Operational burden** | High (you manage) | Low (Google manages) |
| **Customization** | Full control | Limited to service features |
| **Cost at scale** | May be cheaper | May be cheaper at small scale |
| **Security patching** | Your responsibility | Provider responsibility |
| **Compliance** | You must ensure | Shared responsibility |

**Exam pattern:** When a question presents a choice between running open-source software on GKE vs. using a managed Google service, the managed service is usually correct unless the question explicitly mentions a need for customization or features not available in the managed service.

**Risk Assessment Frameworks:**

For the exam, understand that architecture decisions involve trade-offs:
- **Cost vs. Reliability:** Multi-region is more reliable but more expensive.
- **Speed vs. Security:** Fewer approval gates is faster but riskier.
- **Flexibility vs. Complexity:** Microservices are flexible but complex.

The architect's role is to present these trade-offs clearly to stakeholders, not to make business decisions unilaterally.

**Exam tips:**
- ADRs are the answer for "how to document and communicate architecture decisions."
- "Build vs buy" questions almost always favor the managed GCP service unless there is a specific constraint (regulatory, feature gap, data residency).
- When asked how to evaluate risk, the answer involves structured analysis (probability x impact), not gut feeling.

**Docs:**
- [Architecture Framework](https://cloud.google.com/architecture/framework)
- [Architecture Decision Records](https://cloud.google.com/architecture/architecture-decision-records)

---

### Customer Success Management

Architects ensure that deployed solutions continue to meet customer and business needs. Two levers show up in exam scenarios: the SLOs you define for your own users, and the Google support relationship you buy underneath them.

**Cloud Customer Care tiers:**

A surprising number of "customer success" and "operational readiness" items resolve to picking a support tier, so know the four and what separates them.

| Tier | Technical support cases | Response target | Signature feature |
|------|------------------------|-----------------|-------------------|
| **Basic** | None. Docs, community, billing support and Active Assist only | n/a | Included free with every account |
| **Standard** | Unlimited 1:1 technical support | P2 first meaningful response within 4 hours | Cloud Support API. Aimed at workloads still in development |
| **Enhanced** | Unlimited 1:1 technical support | P1 first meaningful response within 1 hour | Third-party technology support; Technical Account Advisor available as an add-on. The usual floor for production |
| **Premium** | Unlimited 1:1 technical support | P1 first meaningful response within 15 minutes | Assigned **Technical Account Manager** and Customer Aware Support; Mission Critical Services available as an add-on |

Reading the tiers on the exam:
- "Production workload, needs a committed response time on outages" → Enhanced at minimum. Basic is disqualified because it carries no technical support cases at all.
- "Named Google contact who knows our architecture", "TAM", "quarterly business reviews" → Premium. The TAM is the discriminator, not the response time.
- "Fastest possible response on a P1" → Premium, 15 minutes.
- Support tier is also a **feature gate**, not only a response-time purchase: Access Transparency requires Standard, Enhanced or Premium (see `docs/03`, Audits and Logs). "Enterprise" is not a Care tier name.

**Monitoring Customer-Facing SLOs:**

The architect defines SLOs that reflect actual user experience, not just infrastructure health.

| SLO Type | Metric | Example Target |
|----------|--------|---------------|
| **Availability** | Successful requests / total requests | 99.9% over 30 days |
| **Latency** | Percentage of requests under threshold | 95% of requests < 200ms |
| **Throughput** | Requests per second at peak | Handle 10,000 RPS |
| **Correctness** | Correct responses / total responses | 99.99% correct data |

```bash
# Create an uptime check (availability monitoring)
gcloud monitoring uptime create my-check \
  --display-name="API Availability" \
  --resource-type=uptime-url \
  --monitored-resource='{"type":"uptime_url","labels":{"host":"api.example.com","project_id":"my-project"}}' \
  --http-check='{"path":"/health","port":443,"use_ssl":true}' \
  --period=60s

# Create an alerting policy. The policy body comes from a file (or an inline
# --policy string); there is no --condition-threshold-value flag. Thresholds are
# expressed with --if, e.g. --if='> 0.01'.
gcloud monitoring policies create \
  --policy-from-file=high-error-rate.yaml
```

**Feedback Loops:**

- **Proactive monitoring:** SLO-based alerts catch issues before customers report them.
- **Incident communication:** Use Cloud Monitoring + PagerDuty/Opsgenie integration for on-call escalation. Status pages for external communication.
- **Post-incident review (PIR):** Blameless postmortems. Document what happened, timeline, root cause, action items. Focus on systemic improvements, not individual blame.
- **User feedback integration:** Application-level metrics (Cloud Monitoring custom metrics) combined with user feedback for continuous improvement.

**Exam tips:**
- SLOs should measure user experience, not just infrastructure metrics. "Server CPU < 80%" is not an SLO; "99.9% of requests complete in < 200ms" is.
- Error budgets link reliability to feature velocity: if the error budget is healthy, ship features faster; if exhausted, focus on reliability.
- Blameless postmortems are a Google SRE principle frequently tested on the PCA exam.
- Read "customer success" scenarios twice: some are SLO questions, some are support-tier questions. A stem about response commitments, a named Google contact, or third-party technology support is asking you to pick a Customer Care tier, not to design an alert.

**Docs:**
- [Cloud Customer Care](https://cloud.google.com/support)
- [Standard Support](https://cloud.google.com/support/docs/standard), [Enhanced Support](https://cloud.google.com/support/docs/enhanced), [Premium Support](https://cloud.google.com/support/docs/premium)
- [SLO Monitoring](https://cloud.google.com/monitoring/slo)
- [Alerting Policies](https://cloud.google.com/monitoring/alerts)
- [Incident Response](https://sre.google/sre-book/managing-incidents/)
- [Postmortem Culture](https://sre.google/sre-book/postmortem-culture/)

---

### Cost Optimization / Resource Optimization

This is the most heavily tested business process topic. You must know every discount type, right-sizing tool, and FinOps practice on GCP.

**FinOps on GCP:**

FinOps (Financial Operations) is the practice of bringing financial accountability to cloud spending. The three phases:

1. **Inform:** Visibility into who is spending what (billing exports, labels, dashboards).
2. **Optimize:** Reduce waste (right-sizing, CUDs, deleting unused resources).
3. **Operate:** Continuous governance (budgets, alerts, automation).

**CapEx vs OpEx Transition:**

| Aspect | CapEx (On-Prem) | OpEx (Cloud) |
|--------|----------------|--------------|
| **Payment** | Large upfront investment | Pay-as-you-go |
| **Depreciation** | Assets depreciate over 3-5 years | No depreciation, current expense |
| **Scaling** | Buy for peak, waste during off-peak | Scale to demand, pay for what you use |
| **Risk** | Stranded assets if requirements change | Flexibility to change services |
| **Budget** | Predictable (fixed) | Variable (but can be controlled with CUDs) |

> **Exam trap:** Cloud is OpEx by default, but **Committed Use Discounts** make it more predictable (closer to CapEx-like commitment). This is how you address finance teams wanting cost predictability.

---

#### Committed Use Discounts (CUDs)

CUDs give a discount in exchange for a 1-year or 3-year commitment. There are **three** types, and the exam tests which one fits a described workload.

| Type | What You Commit To | Flexibility | 1-year | 3-year |
|------|-------------------|-------------|--------|--------|
| **Resource-based CUDs** | Specific vCPU count and memory in one region | Locked to that region and machine family. Covers Compute Engine and GKE nodes | ~37% | up to 55% (up to 70% memory-optimized) |
| **Flexible CUDs** (spend-based compute) | Dollar amount per hour of compute spend | Portable across regions and machine families. Covers Compute Engine, GKE Autopilot and Cloud Run | ~28% | ~46% |
| **Spend-based CUDs** (per service) | Dollar amount per hour on one service | Cloud SQL, AlloyDB, VMware Engine, BigQuery editions and other eligible services | ~25% | ~52% |

**Flexible CUDs are what Google leads with now.** You accept a lower rate in return for not being locked to a machine family or region, which suits any workload that might be re-shaped or moved. Reach for resource-based only when the workload's shape and location are genuinely stable, and take the higher rate for that certainty.

```bash
# Create a resource-based CUD (Compute Engine)
# Commitment types are lowercase-hyphenated: general-purpose, general-purpose-n2,
# compute-optimized, memory-optimized, and so on.
gcloud compute commitments create my-commitment \
  --region=us-central1 \
  --plan=36-month \
  --resources=vcpu=100,memory=400GB \
  --type=general-purpose

# Create a spend-based CUD
# (Done through Cloud Console or API, not gcloud CLI for spend-based)

# List commitments
gcloud compute commitments list --region=us-central1

# Describe a commitment
gcloud compute commitments describe my-commitment --region=us-central1
```

**Key CUD facts:**
- Resource-based CUDs are regional. A commitment in `us-central1` does not apply to VMs in `us-east1`. Flexible CUDs are not regional, which is their point.
- Resource-based CUDs apply across VM families within the commitment type (a general-purpose CUD covers N2, N2D, E2 and so on).
- **Resource-based CUDs do not cover Dataflow.** Dataflow bills under its own SKUs. This is a common wrong assumption because Dataflow runs on VMs.
- CUDs are non-cancellable. If you overcommit, you still pay for the committed amount.
- CUDs can be shared across projects within the same billing account, with CUD sharing enabled.
- Resource-based: ~37% for 1-year, up to 55% for 3-year, up to 70% for memory-optimized. Flexible: ~28% and ~46%.

---

#### Sustained Use Discounts (SUDs)

SUDs are automatic discounts for running Compute Engine resources for a significant portion of the month.

**How SUDs Work:**
- Computed monthly per region per resource type.
- Earned in quarters of the month. The more of the month a resource runs, the higher the discount.
- Applied automatically. No commitment or action required.
- Applies to Compute Engine VMs, GKE nodes, and **sole-tenant nodes, including the sole-tenancy premium**.
- **Does NOT apply to:** E2 machines, Spot VMs, or resources already covered by CUDs.

**SUD Discount Tiers:**

| Usage (% of month) | Discount earned on that portion |
|--------------------|--------------------|
| 0-25% | 0% (full price) |
| 25-50% | 10% |
| 50-75% | 20% |
| 75-100% | 30% |

**The maximum depends on the machine family**, which is the part most people miss:

| Family | Maximum SUD |
|--------|-------------|
| N1, M1, M2 | 30% |
| N2, N2D, C2 | 20% |
| E2 | none |

> **Exam trap:** SUDs and CUDs do not stack. If a resource is covered by a CUD it does not also get a SUD. CUDs give the better rate, so they take priority.

> **Exam trap:** E2 machine types receive no SUDs at all. N1, N2 and C2 do, but at different maximums, so "up to 30%" is only true for N1, M1 and M2. C2 is eligible despite being commonly listed as excluded.

> **Exam trap:** sole-tenant nodes **do** earn SUDs. The sole-tenancy premium is discounted too, which is counter-intuitive because sole-tenancy is usually framed as the expensive option.

---

#### Spot VMs (formerly Preemptible VMs)

Spot VMs offer up to 60-91% discount but can be reclaimed by Google at any time with a 30-second warning.

**When to Use Spot VMs:**
- Batch processing jobs
- CI/CD build workers
- Data processing pipelines (Dataproc, Dataflow)
- Fault-tolerant distributed workloads
- Development and testing environments

**When NOT to Use Spot VMs:**
- Production web servers
- Databases (data loss risk)
- Any workload that cannot tolerate interruption

```bash
# Create a Spot VM
gcloud compute instances create my-spot-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Create a MIG with Spot VMs
gcloud compute instance-templates create spot-template \
  --machine-type=n2-standard-4 \
  --provisioning-model=SPOT \
  --instance-termination-action=DELETE

gcloud compute instance-groups managed create spot-mig \
  --template=spot-template \
  --size=10 \
  --zone=us-central1-a
```

**Key Spot VM facts:**
- `--instance-termination-action=STOP` keeps the VM (disk preserved); `DELETE` removes it entirely.
- Spot VMs in MIGs with autoscaling: if instances are reclaimed, the MIG automatically tries to recreate them.
- No minimum runtime guarantee (unlike old preemptible VMs which had a 24-hour max).
- Availability varies by zone and machine type. Not guaranteed to be available.

---

#### Right-Sizing with Recommender

Google Cloud Recommender analyzes resource utilization and provides recommendations to reduce waste.

**Types of Recommendations:**

| Recommendation | What It Detects | Action |
|----------------|----------------|--------|
| **VM Right-Sizing** | Oversized VMs (low CPU/memory utilization) | Resize to smaller machine type |
| **Idle Resources** | VMs, disks, IPs not being used | Delete or stop the resource |
| **Idle Projects** | Projects with no resource activity | Review and delete if unused |
| **Commitment Recommendations** | Usage patterns suitable for CUDs | Purchase commitments |

```bash
# List VM sizing recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=my-project \
  --location=us-central1-a

# List idle VM recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.instance.IdleResourceRecommender \
  --project=my-project \
  --location=us-central1-a

# List idle disk recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.disk.IdleResourceRecommender \
  --project=my-project \
  --location=us-central1-a

# Apply a recommendation
gcloud recommender recommendations mark-claimed \
  RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --project=my-project \
  --location=us-central1-a \
  --etag=ETAG
```

**Active Assist:**

Active Assist is the umbrella for Google Cloud's intelligent recommendation services:
- **Recommender:** Resource optimization (sizing, idle, commitments)
- **Policy Intelligence:** IAM recommendations (remove unused roles, identify overly permissive roles)
- **Security Recommendations:** Firewall, encryption, vulnerability fixes
- **Cost Recommendations:** Unified cost insights in the Console

---

#### BigQuery Pricing Optimization

BigQuery has two pricing models:

| Model | How It Works | Best For |
|-------|-------------|----------|
| **On-Demand** | $6.25/TB scanned (first 1 TB/month free) | Unpredictable, ad-hoc queries |
| **Capacity (Slots)** | Pay for compute capacity (slots), not data scanned | Predictable, high-volume workloads |

**Slot Reservations:**

```bash
# Purchase slot commitments
bq mk --capacity_commitment \
  --project_id=my-project \
  --location=us \
  --plan=ANNUAL \
  --slots=500

# Create a reservation
bq mk --reservation \
  --project_id=my-project \
  --location=us \
  --slots=500 \
  --reservation_id=my-reservation

# Assign a project to a reservation
bq mk --reservation_assignment \
  --project_id=my-project \
  --location=us \
  --reservation_id=my-reservation \
  --assignee_type=PROJECT \
  --assignee_id=analytics-project
```

**BigQuery Cost Reduction Strategies:**
- **Partitioning:** Reduces data scanned per query. Partition by date, integer range, or ingestion time.
- **Clustering:** Co-locates related data. Reduces data scanned for filtered queries.
- **Materialized views:** Pre-computed query results, auto-refreshed. Reduces repeated expensive queries.
- **BI Engine:** In-memory analysis for dashboards. Reduces query costs for Looker/Data Studio.
- **Storage pricing:** Active storage ($0.02/GB/month) vs. long-term ($0.01/GB/month, auto after 90 days unchanged).

---

#### Billing Exports and Cost Analysis

```bash
# Enable billing export to BigQuery
# (Done via Console: Billing > Billing export > BigQuery export)
# Exports detailed billing data, including: SKU, usage, cost, credits, labels

# Query billing data in BigQuery
# SELECT
#   project.id AS project,
#   service.description AS service,
#   SUM(cost) AS total_cost,
#   SUM(IFNULL((SELECT SUM(c.amount) FROM UNNEST(credits) c), 0)) AS total_credits
# FROM `my-project.billing_export.gcp_billing_export_v1_XXXXXX`
# WHERE invoice.month = '202602'
# GROUP BY 1, 2
# ORDER BY total_cost DESC
```

---

#### Labels and Tags for Cost Allocation

| Feature | Labels | Tags |
|---------|--------|------|
| **Purpose** | Organize resources, cost allocation, filtering | IAM conditional access, org policy conditions, firewall rules |
| **Format** | Key-value pairs on resources | Key-value pairs, centrally defined in Tag Manager |
| **Cost tracking** | Yes (appear in billing exports) | Yes (appear in billing exports) |
| **IAM integration** | No | Yes (IAM conditions can reference tags) |
| **Inheritance** | No | Yes (tags inherit down resource hierarchy) |

```bash
# Add labels to a VM
gcloud compute instances update my-vm \
  --update-labels=env=production,team=backend,cost-center=eng-123 \
  --zone=us-central1-a

# Add labels to a GKE cluster
gcloud container clusters update my-cluster \
  --update-labels=env=production \
  --region=us-central1

# Filter resources by label
gcloud compute instances list --filter="labels.env=production"

# Create a tag key
gcloud resource-manager tags keys create environment \
  --parent=organizations/123456789 \
  --description="Environment tag"

# Create a tag value
gcloud resource-manager tags values create production \
  --parent=organizations/123456789/environment \
  --description="Production environment"

# Bind a tag to a resource
gcloud resource-manager tags bindings create \
  --tag-value=organizations/123456789/environment/production \
  --parent=//compute.googleapis.com/projects/my-project/zones/us-central1-a/instances/my-vm
```

---

#### Cloud Billing Budgets and Alerts

```bash
# Create a budget
gcloud billing budgets create \
  --billing-account=XXXXXX-XXXXXX-XXXXXX \
  --display-name="Monthly Budget" \
  --budget-amount=10000 \
  --threshold-rule=percent=0.5 \
  --threshold-rule=percent=0.9 \
  --threshold-rule=percent=1.0 \
  --notifications-rule-pubsub-topic=projects/my-project/topics/billing-alerts \
  --notifications-rule-monitoring-notification-channels=projects/my-project/notificationChannels/123

# List budgets
gcloud billing budgets list --billing-account=XXXXXX-XXXXXX-XXXXXX
```

**Programmatic Budget Actions:**

Budgets can trigger Pub/Sub messages, which can invoke Cloud Functions to take automated actions:
- Scale down non-critical resources
- Disable billing on a project (nuclear option)
- Send notifications to Slack/Teams
- Create Jira/ServiceNow tickets

> **Exam trap:** **Budgets do NOT stop spending.** They only send alerts. To actually stop spending, you need programmatic actions (Cloud Function triggered by Pub/Sub) to disable billing or shut down resources.

**Cost Optimization Summary Cheat Sheet:**

| Strategy | Savings | Effort | Best For |
|----------|---------|--------|----------|
| **Delete idle resources** | Immediate | Low | Quick wins |
| **Right-size VMs** | 20-50% per VM | Low | Overprovisioned workloads |
| **Sustained Use Discounts** | Up to 30% (N1/M1/M2) or 20% (N2/N2D/C2) | None (automatic) | Steady-state non-E2 VMs |
| **1-Year resource CUDs** | ~37% | Medium (commitment) | Predictable workloads, fixed region and family |
| **3-Year resource CUDs** | Up to 55% (70% memory-optimized) | High (long commitment) | Stable, long-term workloads |
| **Flexible CUDs** | ~28% (1yr), ~46% (3yr) | Medium | Compute spend that may move region or family |
| **Spot VMs** | 60-91% | Medium (fault tolerance) | Batch, CI/CD, dev/test |
| **Autoscaling** | Variable | Medium | Variable traffic patterns |
| **BigQuery partitioning** | 50-90% per query | Low-Medium | Large BigQuery tables |
| **Cloud Storage lifecycle** | Variable | Low | Aging data |
| **Preemptible Dataproc** | 60-80% | Low | Batch analytics |

**Exam tips:**
- **CUDs vs SUDs:** CUDs require commitment and give the bigger discount (~37% for 1 year, up to 55% for 3, up to 70% memory-optimized). SUDs are automatic and cap at 30% for N1/M1/M2 or 20% for N2/N2D/C2. They do NOT stack.
- **Three CUD types:** resource-based (Compute Engine and GKE, locked to region and family), flexible (compute spend, portable, ~28%/~46%), and spend-based per service (Cloud SQL, AlloyDB, Cloud Run, VMware Engine, BigQuery editions).
- **Flexible CUDs are the current default recommendation** when the workload might move. If a question stresses changing requirements or multi-region, the flexible CUD is likely the answer even though its headline rate is lower.
- **E2 machines do NOT get SUDs.** This is a trap.
- **Spot VMs require fault tolerance.** If the question says "mission-critical" or "cannot tolerate interruption," Spot is wrong.
- **Budgets alert but do not stop spending.** Programmatic enforcement requires Pub/Sub + Cloud Functions.
- **Labels are the foundation of cost allocation.** Without labels, you cannot attribute costs to teams, projects, or environments in billing exports.
- **Tags** are different from labels: tags integrate with IAM and org policies, labels are for organization and billing.
- **Right-sizing Recommender** needs at least 8 days of data to generate recommendations.
- For BigQuery, **partitioning + clustering** is the first cost optimization step. Switch to capacity pricing (slots) only when on-demand costs are predictable and high.
- **Billing exports to BigQuery** is the answer for "detailed cost analysis and custom reporting."

**Docs:**
- [Cost Optimization](https://cloud.google.com/architecture/framework/cost-optimization)
- [Committed Use Discounts](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview)
- [Sustained Use Discounts](https://cloud.google.com/compute/docs/sustained-use-discounts)
- [Spot VMs](https://cloud.google.com/compute/docs/instances/spot)
- [Recommender](https://cloud.google.com/recommender/docs)
- [Active Assist](https://cloud.google.com/solutions/active-assist)
- [BigQuery Pricing](https://cloud.google.com/bigquery/pricing)
- [Cloud Billing Budgets](https://cloud.google.com/billing/docs/how-to/budgets)
- [Billing Export to BigQuery](https://cloud.google.com/billing/docs/how-to/export-data-bigquery)
- [Labels](https://cloud.google.com/resource-manager/docs/creating-managing-labels)
- [Tags](https://cloud.google.com/resource-manager/docs/tags/tags-overview)

---

### Business Continuity

Business Continuity Planning (BCP) is broader than Disaster Recovery. It covers the entire organization's ability to maintain operations during and after a disruption.

**BCP vs DR:**

| Aspect | BCP (Business Continuity Plan) | DR (Disaster Recovery) |
|--------|-------------------------------|----------------------|
| **Scope** | Entire business: people, processes, technology, facilities | IT systems and data recovery |
| **Focus** | Maintaining business operations during any disruption | Restoring IT services after a specific incident |
| **Includes** | Communication plans, alternate work sites, supply chain, DR | Technical failover, data restoration, system recovery |
| **Owner** | Business leadership, risk management | IT / operations team |
| **Trigger** | Any business disruption (natural disaster, pandemic, key person departure) | IT system failure, data center outage, data corruption |

**Business Impact Analysis (BIA):**

BIA is the process of identifying critical business functions and quantifying the impact of their disruption.

| Step | Action | Output |
|------|--------|--------|
| 1. **Identify functions** | List all business processes | Process inventory |
| 2. **Assess impact** | Quantify financial, operational, reputational impact of downtime | Impact scores |
| 3. **Determine dependencies** | Map IT systems and data that each function requires | Dependency map |
| 4. **Set recovery priorities** | Rank functions by criticality | Priority tiers (P1, P2, P3) |
| 5. **Define RTO/RPO** | Set recovery objectives per function based on impact | RTO/RPO targets |
| 6. **Cost-justify DR** | Match DR pattern (cold/warm/hot) to each tier | DR architecture |

**Recovery Strategy Aligned with Business Priority:**

| Priority | RTO | RPO | DR Pattern | GCP Example |
|----------|-----|-----|-----------|-------------|
| **P1 (Critical)** | < 15 min | < 5 min | Hot standby or active-active | Spanner multi-region, Global LB, multi-cluster GKE |
| **P2 (Important)** | < 4 hours | < 1 hour | Warm standby | Cloud SQL cross-region replica, smaller GKE in DR region |
| **P3 (Normal)** | < 24 hours | < 24 hours | Cold standby | Cloud Storage backups, Terraform scripts to rebuild |
| **P4 (Low)** | Days | Days | Backup & restore | Scheduled backups to Cloud Storage, manual rebuild |

**Exam tips:**
- BCP is broader than DR. If the question asks "how to ensure the business continues operating during a regional outage," the answer involves BCP (people, processes, communication) not just DR (restore IT systems).
- BIA drives the DR architecture. Questions that provide RTO/RPO requirements expect you to choose the cheapest DR pattern that meets those requirements.
- Not all systems need the same DR level. The architect's job is to align DR investment with business impact.
- "Zero RPO" = synchronous replication = expensive (Spanner, Firestore multi-region). "Near-zero RPO" = asynchronous replication = cheaper (Cloud SQL replicas, Bigtable replication).
- The exam may present a scenario where a company wants "zero downtime for everything." The correct answer is to perform a BIA and tier the systems, not to put everything on the most expensive DR pattern.

**Docs:**
- [Business Continuity Planning](https://cloud.google.com/architecture/framework/reliability/business-continuity-planning)
- [DR Planning Guide](https://cloud.google.com/architecture/dr-scenarios-planning-guide)
- [Reliability Pillar](https://cloud.google.com/architecture/framework/reliability)

---

## Section 4 Quick Reference: Key Services and Their Roles

| Service | Category | Primary Use in This Section |
|---------|----------|---------------------------|
| **Cloud Build** | CI/CD | Build, test, and deploy (CI pipeline) |
| **Cloud Deploy** | CI/CD | Managed progressive delivery (CD pipeline) |
| **Artifact Registry** | CI/CD | Store and manage container images and packages |
| **Cloud Workstations** | Development | Managed development environments |
| **Cloud Monitoring** | Observability | Metrics, SLOs, alerting, dashboards |
| **Cloud Logging** | Observability | Centralized log management |
| **Cloud Trace** | Observability | Distributed request tracing |
| **Error Reporting** | Observability | Error aggregation and deduplication |
| **Cloud Profiler** | Observability | Continuous CPU/memory profiling |
| **Recommender** | Cost Optimization | Right-sizing, idle resource detection |
| **Active Assist** | Cost Optimization | Unified recommendation platform |
| **Cloud Billing** | Cost Optimization | Budgets, alerts, exports |
| **Backup for GKE** | DR | Kubernetes workload backup/restore |
| **Backup and DR** | DR | Managed backup for Compute, GKE, Cloud SQL |
| **Service Catalog** | Governance | Curated, self-service internal solutions (Terraform configurations, reference links) |
| **Cloud Customer Care** | Business process | Support tiers: Basic, Standard, Enhanced, Premium |

---

## Section 4 Exam Strategy

1. **CI/CD questions:** Map the scenario to Cloud Build (CI) + Cloud Deploy (CD) + Artifact Registry (artifacts). Know the YAML config structure.
2. **Troubleshooting questions:** Match the symptom to the right observability tool (Trace for latency, Error Reporting for errors, Logging for investigation, Profiler for performance).
3. **DR questions:** Identify the RTO/RPO requirement, then pick the cheapest pattern that satisfies it. Know which databases support multi-region (Spanner, Firestore).
4. **Cost questions:** Eliminate distractors by knowing which discounts apply where (SUDs not for E2, CUDs are regional, Spot needs fault tolerance).
5. **Business process questions:** Choose answers that include stakeholder communication, phased approaches, and training -- not just technical solutions.
6. **When in doubt:** The PCA exam favors managed services, automation, and business-aligned decisions over manual, custom, or technically pure solutions.
