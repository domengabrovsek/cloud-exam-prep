# Section 6: Ensuring Solution and Operations Excellence (~12.5% of Exam)

> **Quick context:** This section tests operations architecture -- observability, reliability, deployment strategies, and quality measures. Expect 6-8 questions. The emphasis is on knowing *which* tool to use for a given operational scenario (monitoring vs logging vs tracing), understanding SLO/SLI/SLA relationships, and selecting appropriate deployment strategies for reliability requirements. This overlaps heavily with SRE principles.

---

## 6.1 Operational Excellence Pillar of the Google Cloud Well-Architected Framework

> Cross-reference: `docs/07-well-architected-framework.md` for full WAF coverage.

The **Operational Excellence** pillar is one of six pillars in the Google Cloud Well-Architected Framework (along with Reliability, Security, Performance Optimization, Cost Optimization, and Sustainability). It focuses on running workloads effectively, monitoring them proactively, and continuously improving processes.

### Key Principles

| Principle | What It Means | Exam Relevance |
|-----------|---------------|----------------|
| **Automate everything** | Eliminate manual, repetitive tasks (toil) through IaC, CI/CD, auto-remediation | Questions about reducing operational burden |
| **Monitor before you act** | Observability must be in place before making changes; data-driven decisions | Questions about what to set up first |
| **Reduce toil** | Toil = manual, repetitive, automatable, tactical, devoid of long-term value | Questions referencing SRE toil concepts |
| **Learn from failure** | Blameless post-mortems, incident reviews, chaos engineering | Questions about incident response |
| **Design for operability** | Systems should be easy to deploy, update, and debug | Questions about choosing architectures |
| **Gradual rollouts** | Progressive delivery reduces blast radius | Questions about deployment strategies |

### How Operational Excellence Appears in Exam Questions

The exam tests operational excellence through scenarios like:

- "Your team spends 60% of time on manual deployments. How do you reduce this?" --> Automate with Cloud Deploy / Cloud Build
- "An outage occurred but nobody was notified for 30 minutes." --> Alerting policies, notification channels, uptime checks
- "After a production incident, the team blamed one engineer." --> Blameless post-mortems, incident culture
- "You need to ensure a new service meets reliability targets before launch." --> SLOs, error budgets, load testing

### Operational Maturity Model

```
Level 0: Manual       --> Ad-hoc operations, no monitoring, firefighting
Level 1: Reactive     --> Basic monitoring, manual incident response
Level 2: Proactive    --> Alerting on SLOs, runbooks, automated deploys
Level 3: Predictive   --> ML-based anomaly detection, chaos engineering, self-healing
```

The exam expects Level 2-3 thinking. If a question describes manual or ad-hoc operations, the answer almost always involves automation and monitoring.

**Exam tips:**
- If a question mentions "toil" or "manual repetitive tasks," the answer is automation (Cloud Build, Cloud Deploy, Terraform, etc.).
- Operational excellence questions often have a "cultural" best-answer (blameless post-mortems) over a purely technical one.
- "Monitor first, then act" is a recurring theme -- you cannot optimize what you cannot measure.

**Docs:**
- [Well-Architected Framework: Operational Excellence](https://cloud.google.com/architecture/framework/operational-excellence)
- [SRE Book: Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)

---

## 6.2 Google Cloud Observability Solutions

> This is the largest sub-section. The exam heavily tests which observability tool to use for which scenario. Know the boundaries between Monitoring, Logging, Trace, and Profiler.

### Cloud Monitoring

Cloud Monitoring is the central metrics, dashboards, and alerting platform in Google Cloud. It collects metrics from all GCP services automatically and supports custom metrics, Prometheus, and third-party integrations.

#### Metrics Types

| Metric Type | Source | Example | Retention |
|-------------|--------|---------|-----------|
| **Built-in (system)** | Auto-collected from GCP services | `compute.googleapis.com/instance/cpu/utilization` | 5 years (6 weeks full-res, then downsampled) |
| **Custom metrics** | Written via API or client libraries | `custom.googleapis.com/orders/per_minute` | 24 months |
| **Prometheus metrics** | Scraped by GMP (Managed Service for Prometheus) | `http_requests_total` | 24 months in Monarch |
| **External metrics** | From AWS, on-prem via agent | `external.googleapis.com/...` | 24 months |
| **Log-based metrics** | Derived from Cloud Logging | Counter or distribution from log entries | 24 months |

```bash
# List available metric descriptors for a project
gcloud monitoring metrics-descriptors list \
  --filter='metric.type = starts_with("compute.googleapis.com/instance/cpu")'

# Create a custom metric descriptor
gcloud monitoring metrics-descriptors create \
  custom.googleapis.com/orders/per_minute \
  --type=custom.googleapis.com/orders/per_minute \
  --metric-kind=GAUGE \
  --value-type=INT64 \
  --description="Orders processed per minute"

# Write a custom metric data point (typically done via client library, not CLI)
# Python example:
# client = monitoring_v3.MetricServiceClient()
# series = monitoring_v3.TimeSeries()
# series.metric.type = "custom.googleapis.com/orders/per_minute"
# ...
```

#### Monitoring Query Language (MQL)

MQL is a text-based query language for Cloud Monitoring that provides more power than the visual query builder.

```
# Fetch CPU utilization for all VMs, averaged over 5 minutes
fetch gce_instance
| metric 'compute.googleapis.com/instance/cpu/utilization'
| group_by 5m, [value_utilization_mean: mean(value.utilization)]
| every 5m

# Filter by zone and aggregate
fetch gce_instance
| metric 'compute.googleapis.com/instance/cpu/utilization'
| filter (resource.zone == 'us-central1-a')
| group_by [resource.instance_id], [value_utilization_max: max(value.utilization)]
| every 1m

# 95th percentile of HTTP latency
fetch https_lb_rule
| metric 'loadbalancing.googleapis.com/https/total_latencies'
| group_by 1m, [p95: percentile(value.total_latencies, 95)]
| every 1m
```

#### Managed Service for Prometheus (GMP)

GMP lets you use Prometheus query language (PromQL) natively on Google Cloud without managing Prometheus servers. Data is stored in Google's Monarch time-series database.

```yaml
# PodMonitoring resource (GKE) -- auto-discovers and scrapes pods
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: my-app-metrics
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: metrics
      interval: 30s
```

```bash
# Verify GMP is enabled on a GKE cluster
gcloud container clusters describe CLUSTER_NAME \
  --zone=ZONE \
  --format="value(monitoringConfig.managedPrometheusConfig.enabled)"

# Enable managed collection on an existing cluster
gcloud container clusters update CLUSTER_NAME \
  --zone=ZONE \
  --enable-managed-prometheus
```

#### Comparison: Custom Metrics vs Prometheus vs MQL

| Aspect | Custom Metrics API | Managed Prometheus (GMP) | MQL |
|--------|-------------------|--------------------------|-----|
| **Use case** | App-specific business metrics | Kubernetes/container metrics | Advanced queries on any metric |
| **Query language** | Monitoring filters | PromQL | MQL |
| **Setup** | Client library or API calls | PodMonitoring CRD on GKE | Built into Cloud Monitoring |
| **Best for** | Non-K8s workloads, business KPIs | K8s-native teams, existing Prom users | Complex aggregations, dashboards |
| **Retention** | 24 months | 24 months (Monarch) | Queries existing metrics |
| **Cost** | Per time series ingested | Per samples ingested | No extra cost (query layer) |

#### Dashboards and Charts

```bash
# List dashboards
gcloud monitoring dashboards list

# Describe a specific dashboard
gcloud monitoring dashboards describe DASHBOARD_ID

# Create a dashboard from JSON
gcloud monitoring dashboards create --config-from-file=dashboard.json
```

Key dashboard types:
- **Custom dashboards**: User-created, shareable across project
- **Predefined dashboards**: Auto-created for GCP services (GCE, GKE, Cloud SQL, etc.)
- **Logs dashboards**: Visualize log-based metrics

#### Uptime Checks

Uptime checks probe your endpoints from multiple global locations and trigger alerts on failure.

```bash
# Create an HTTP uptime check
gcloud monitoring uptime create my-service-check \
  --display-name="My Service Health Check" \
  --resource-type=uptime-url \
  --hostname=my-service.example.com \
  --path=/healthz \
  --protocol=HTTPS \
  --period=60 \
  --timeout=10 \
  --regions=USA,EUROPE,ASIA_PACIFIC

# List uptime checks
gcloud monitoring uptime list-configs

# Delete an uptime check
gcloud monitoring uptime delete my-service-check
```

Types of uptime checks:
- **HTTP/HTTPS**: Check URL availability and response codes
- **TCP**: Check port connectivity
- **Custom**: Use Cloud Functions for complex health checks

Configuration options:
- Check frequency: 1, 5, 10, or 15 minutes
- Regions: USA, Europe, South America, Asia Pacific (minimum 3 for alerting)
- Content matching: Verify response body contains expected string
- Authentication: Basic auth, custom headers

#### Alerting Policies

Alerting policies define conditions under which notifications are sent.

```bash
# Create an alerting policy from JSON file
gcloud monitoring policies create --policy-from-file=policy.json

# List alerting policies
gcloud monitoring policies list

# Describe a specific policy
gcloud monitoring policies describe POLICY_ID

# Enable/disable a policy
gcloud monitoring policies update POLICY_ID --enabled
gcloud monitoring policies update POLICY_ID --no-enabled

# List notification channels
gcloud monitoring channels list

# Create a notification channel
gcloud monitoring channels create \
  --display-name="Ops Team Email" \
  --type=email \
  --channel-labels=email_address=ops@example.com
```

Alerting policy JSON example:

```json
{
  "displayName": "High CPU Alert",
  "conditions": [
    {
      "displayName": "CPU > 80% for 5 minutes",
      "conditionThreshold": {
        "filter": "metric.type=\"compute.googleapis.com/instance/cpu/utilization\" AND resource.type=\"gce_instance\"",
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
  "notificationChannels": ["projects/PROJECT/notificationChannels/CHANNEL_ID"],
  "alertStrategy": {
    "autoClose": "1800s"
  },
  "documentation": {
    "content": "CPU utilization exceeded 80% for 5 minutes. Check for runaway processes.",
    "mimeType": "text/markdown"
  }
}
```

#### Monitoring Groups

Resource groups let you organize monitored resources for collective dashboards and alerting.

```bash
# Groups are defined by filter criteria, e.g.:
# - Name contains "web-server"
# - Tag: env=production
# - Region: us-central1
# Groups are dynamic -- resources matching the filter are auto-included
```

**Exam tips:**
- Know which metric type to use: built-in for GCP services, custom for business metrics, Prometheus for K8s workloads.
- Uptime checks run from *Google's infrastructure*, not your project -- they check external availability from the internet.
- Alerting policy duration ("for 5 minutes") prevents flapping -- a single spike does not trigger an alert.
- MQL is for advanced queries that the visual builder cannot express (percentiles, ratios, cross-metric calculations).
- GMP does NOT require running your own Prometheus server -- it is fully managed.
- Custom metrics are limited to 10,000 metric descriptors per project by default.

**Docs:**
- [Cloud Monitoring overview](https://cloud.google.com/monitoring/docs)
- [MQL reference](https://cloud.google.com/monitoring/mql/reference)
- [Managed Service for Prometheus](https://cloud.google.com/stackdriver/docs/managed-prometheus)
- [Uptime checks](https://cloud.google.com/monitoring/uptime-checks)
- [Alerting policies](https://cloud.google.com/monitoring/alerts)
- [Custom metrics](https://cloud.google.com/monitoring/custom-metrics)

---

### Cloud Logging

Cloud Logging is the centralized log management service. It ingests, stores, routes, and analyzes logs from all GCP services, VMs (via Ops Agent), containers, and custom applications.

#### Log Types

| Log Type | Description | Examples | Default Destination |
|----------|-------------|----------|---------------------|
| **Platform logs** | Auto-generated by GCP services | GCE instance events, GKE node logs, Cloud SQL logs | `_Default` bucket |
| **User logs** | Written by your applications | App error logs, request logs via client libraries | `_Default` bucket |
| **Security logs (Audit)** | Who did what, where, when | Admin Activity, Data Access, System Event, Policy Denied | See below |

#### Audit Log Types (Critical for Exam)

| Audit Log | What It Records | Enabled By Default? | Storage | Exemptions |
|-----------|----------------|---------------------|---------|------------|
| **Admin Activity** | Resource config changes (create, delete, update) | Yes, always on | `_Required` bucket (400 days, immutable) | Cannot be disabled or exempted |
| **System Event** | Google-initiated system actions | Yes, always on | `_Required` bucket (400 days) | Cannot be disabled |
| **Data Access** | Read resource config, read/write user data | No (except BigQuery) | `_Default` bucket (30 days) | Can exempt specific users/groups |
| **Policy Denied** | Security policy violations | Yes, always on | `_Default` bucket (30 days) | Can exempt specific users/groups |

```bash
# View Admin Activity audit logs
gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity"' \
  --limit=10 --format=json

# View Data Access audit logs
gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Fdata_access"' \
  --limit=10

# Enable Data Access audit logs for all services (via org policy or per-service)
# This is done in IAM & Admin > Audit Logs in Console, or via:
gcloud projects get-iam-policy PROJECT_ID --format=json > policy.json
# Edit auditConfigs section, then:
gcloud projects set-iam-policy PROJECT_ID policy.json
```

#### Log Router Architecture

The Log Router is the central routing engine that processes every log entry and sends it to the appropriate destination based on sink configurations.

```
                        +------------------+
                        |   Log Sources    |
                        | (GCP, Apps, VMs) |
                        +--------+---------+
                                 |
                                 v
                    +------------+------------+
                    |       LOG ROUTER        |
                    |  (processes every entry) |
                    +--+------+------+------+-+
                       |      |      |      |
                       v      v      v      v
              +--------+ +--------+ +------+ +----------+
              | _Req'd | | _Deflt | | Sink | | Sink     |
              | Bucket | | Bucket | |  #1  | |  #2      |
              +--------+ +--------+ +------+ +----------+
              |400 days| |30 days | |      | |          |
              |Audit   | |All logs| |BigQry| |Cloud Stg |
              |Admin,  | |(excl.  | |for   | |for long  |
              |System  | | filters| |analyt| |term arch |
              +--------+ +--------+ +------+ +----------+
                                       |          |
                                       v          v
                              +-----------+ +-----------+
                              | BigQuery  | |   GCS     |
                              | Dataset   | |  Bucket   |
                              +-----------+ +-----------+

  Additional sink destinations:
  - Cloud Pub/Sub (for streaming to external systems / SIEM)
  - Splunk (via Pub/Sub integration)
  - Another Cloud Logging bucket (in same or different project)
  - Log bucket in another project (cross-project routing)
```

#### Log Buckets

| Bucket | Retention | Configurable? | Notes |
|--------|-----------|---------------|-------|
| `_Required` | 400 days | No (fixed) | Admin Activity + System Event audit logs. Cannot delete or modify. |
| `_Default` | 30 days | Yes (1-3650 days) | All other logs routed here by default. Can modify retention. |
| **Custom buckets** | Configurable (1-3650 days) | Yes | Create for specific use cases (compliance, team isolation). |

```bash
# List log buckets
gcloud logging buckets list --location=global

# Update _Default bucket retention to 90 days
gcloud logging buckets update _Default \
  --location=global \
  --retention-days=90

# Create a custom log bucket
gcloud logging buckets create my-compliance-logs \
  --location=us-central1 \
  --retention-days=365 \
  --description="Compliance logs with 1-year retention"

# Create a CMEK-encrypted log bucket
gcloud logging buckets create encrypted-logs \
  --location=us-central1 \
  --retention-days=365 \
  --cmek-kms-key-name=projects/PROJECT/locations/us-central1/keyRings/RING/cryptoKeys/KEY
```

#### Log Router Sinks

Sinks route logs to destinations based on filters.

```bash
# Create a sink to BigQuery
gcloud logging sinks create bq-all-logs \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/all_logs \
  --log-filter='resource.type="gce_instance"'

# Create a sink to Cloud Storage
gcloud logging sinks create gcs-audit-archive \
  storage.googleapis.com/my-audit-archive-bucket \
  --log-filter='logName:"cloudaudit.googleapis.com"'

# Create a sink to Pub/Sub (for external SIEM)
gcloud logging sinks create pubsub-security-logs \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/security-logs \
  --log-filter='logName:"cloudaudit.googleapis.com%2Fdata_access"'

# Create an aggregated sink (org-level, captures logs from all projects)
gcloud logging sinks create org-audit-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/org_audit \
  --organization=ORG_ID \
  --include-children \
  --log-filter='logName:"cloudaudit.googleapis.com"'

# List sinks
gcloud logging sinks list

# Describe a sink (shows the service account that needs destination permissions)
gcloud logging sinks describe bq-all-logs
```

**Important:** After creating a sink, you must grant the sink's service account write permissions on the destination (BigQuery Data Editor, Storage Object Creator, Pub/Sub Publisher). The service account is shown in the sink description.

#### Inclusion and Exclusion Filters

```bash
# Create an exclusion filter on _Default sink (reduce log volume/cost)
gcloud logging sinks update _Default \
  --add-exclusion=name=exclude-debug,filter='severity="DEBUG"'

# Exclude GKE system container logs (high volume, low value)
gcloud logging sinks update _Default \
  --add-exclusion=name=exclude-gke-system,filter='resource.type="k8s_container" AND resource.labels.namespace_name="kube-system"'

# Remove an exclusion
gcloud logging sinks update _Default \
  --remove-exclusions=exclude-debug

# Inclusion filter example (on a custom sink -- only send matching logs)
gcloud logging sinks create errors-only-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/errors \
  --log-filter='severity >= "ERROR"'
```

#### Log Analytics

Log Analytics lets you run SQL queries against log data stored in upgraded log buckets.

```bash
# Upgrade a bucket to use Log Analytics
gcloud logging buckets update _Default \
  --location=global \
  --enable-analytics

# Create a linked BigQuery dataset for the bucket
gcloud logging links create my-link \
  --bucket=_Default \
  --location=global \
  --linked-dataset-id=log_analytics_default
```

Sample Log Analytics SQL query:
```sql
-- Top 10 error messages in the last 24 hours
SELECT
  json_value(json_payload, '$.message') AS error_message,
  COUNT(*) AS count
FROM `PROJECT_ID.global._Default._AllLogs`
WHERE
  timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND severity = 'ERROR'
GROUP BY error_message
ORDER BY count DESC
LIMIT 10
```

#### Log-Based Metrics

Log-based metrics convert log entries into Cloud Monitoring metrics.

| Type | Description | Use Case |
|------|-------------|----------|
| **Counter** | Counts matching log entries | "How many 500 errors per minute?" |
| **Distribution** | Extracts numeric values and creates histograms | "What is the latency distribution from access logs?" |

```bash
# Create a counter metric for 500 errors
gcloud logging metrics create server-errors \
  --description="Count of 500 error responses" \
  --log-filter='resource.type="gae_app" AND httpRequest.status=500'

# Create a distribution metric for latency
gcloud logging metrics create request-latency \
  --description="Request latency distribution" \
  --log-filter='resource.type="gae_app"' \
  --bucket-options=exponential=64,1.5,0.1

# List log-based metrics
gcloud logging metrics list

# Delete a log-based metric
gcloud logging metrics delete server-errors
```

#### Ops Agent

The Ops Agent is the unified agent for collecting logs and metrics from Compute Engine VMs. It replaces the legacy Monitoring agent and Logging agent.

```bash
# Install Ops Agent on a VM (run on the VM)
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# Or install via policy (fleet-wide, recommended)
gcloud compute instances ops-agents policies create ops-agent-policy \
  --agent-rules="type=ops-agent,version=current-major,package-state=installed,enable-autoupgrade=true" \
  --os-types="short-name=centos,version=8" \
  --zone=us-central1-a \
  --instances="zones/us-central1-a/instances/my-vm"

# Check Ops Agent status on a VM
sudo systemctl status google-cloud-ops-agent

# Ops Agent config file location
# /etc/google-cloud-ops-agent/config.yaml
```

**Exam tips:**
- **_Required bucket cannot be deleted or modified** -- 400-day retention, always contains Admin Activity and System Event audit logs.
- **Data Access logs are OFF by default** (except BigQuery). You must explicitly enable them. They can generate enormous volume.
- Sink service accounts need explicit permissions on the destination -- this is a common exam trap.
- **Exclusion filters reduce cost** by preventing logs from being stored in `_Default`. But excluded logs are gone forever.
- **Aggregated sinks** at the org/folder level use `--include-children` to capture logs from all child projects.
- Log Analytics requires upgrading the bucket -- it is not enabled by default.
- Ops Agent replaces both the legacy Monitoring agent AND Logging agent -- use Ops Agent for all new deployments.
- Log-based metrics appear in Cloud Monitoring and can be used in alerting policies and dashboards.

**Docs:**
- [Cloud Logging overview](https://cloud.google.com/logging/docs)
- [Audit logs](https://cloud.google.com/logging/docs/audit)
- [Log Router](https://cloud.google.com/logging/docs/routing/overview)
- [Log buckets](https://cloud.google.com/logging/docs/buckets)
- [Log Analytics](https://cloud.google.com/logging/docs/log-analytics)
- [Log-based metrics](https://cloud.google.com/logging/docs/logs-based-metrics)
- [Ops Agent](https://cloud.google.com/stackdriver/docs/solutions/agents/ops-agent)

---

### Profiling and Benchmarking

#### Cloud Trace

Cloud Trace is a distributed tracing system that collects latency data from applications. It helps identify bottlenecks in microservice architectures.

- Automatically collects traces from App Engine, Cloud Run, and Cloud Functions
- Requires instrumentation (OpenTelemetry) for GKE, GCE, and other platforms
- Shows end-to-end request latency across service boundaries
- Identifies slow RPCs, database calls, and external API calls

```bash
# List traces (via API -- no direct gcloud command for trace data)
# Typically viewed in Console > Trace > Trace list

# Auto-instrumentation with OpenTelemetry for Python:
# pip install opentelemetry-exporter-gcp-trace
# from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
```

Key concepts:
- **Trace**: The complete journey of a request across all services
- **Span**: A single operation within a trace (e.g., one RPC call)
- **Latency analysis report**: Shows latency distribution over time, identifies regressions
- **Analysis report**: Auto-generated daily, compares latency percentiles

#### Cloud Profiler

Cloud Profiler continuously collects CPU, heap, and memory profiling data from production with minimal overhead (~0.5% CPU).

- **CPU profiling**: Where is CPU time spent?
- **Heap profiling**: What objects are consuming memory?
- **Wall-clock profiling**: Where is time spent (including I/O waits)?
- **Thread profiling**: How are threads distributed?

Supports: Go, Java, Node.js, Python

```bash
# Python profiler setup:
# pip install google-cloud-profiler
# import googlecloudprofiler
# googlecloudprofiler.start(service='my-service', service_version='1.0')
```

#### Error Reporting

Error Reporting aggregates and displays errors from cloud services, automatically grouping similar errors.

- Automatically works with App Engine, Cloud Functions, Cloud Run
- Requires Logging integration for GCE and GKE (errors in logs are auto-detected)
- Groups errors by stack trace similarity
- Tracks error count, affected users, first/last occurrence
- Can create alerting policies on new error groups

```bash
# Error Reporting uses Cloud Logging -- errors are detected from log entries
# with severity ERROR or higher, or entries containing exception stack traces

# View error groups (typically done in Console > Error Reporting)
# Or via API:
gcloud beta error-events list --service=my-service --limit=10
```

**Exam tips:**
- **Cloud Trace = latency** (distributed tracing across microservices). Use when a question asks about "slow requests" or "identifying bottlenecks."
- **Cloud Profiler = resource usage** (CPU, memory). Use when a question asks about "high CPU" or "memory leaks" in production.
- **Error Reporting = error grouping** (aggregate and deduplicate). Use when a question asks about "tracking production errors" or "new error types."
- Trace is automatic for App Engine/Cloud Run/Cloud Functions. For GKE/GCE, you need OpenTelemetry.
- Profiler has negligible overhead and is safe for production. This is a key exam differentiator vs traditional profiling.

**Docs:**
- [Cloud Trace overview](https://cloud.google.com/trace/docs)
- [Cloud Profiler overview](https://cloud.google.com/profiler/docs)
- [Error Reporting overview](https://cloud.google.com/error-reporting/docs)
- [OpenTelemetry on Google Cloud](https://cloud.google.com/trace/docs/setup#opentelemetry)

---

### Alerting Strategies

Effective alerting is a core SRE practice. The exam tests your ability to design alerting that is actionable, not noisy.

#### SLO-Based Alerting (Burn Rate Alerts)

The most important alerting concept for the PCA exam. Instead of alerting on raw metrics (CPU > 80%), alert on SLO burn rate.

**Burn rate** = the rate at which your error budget is being consumed.

```
Burn rate = (observed error rate) / (1 - SLO)

Example:
  SLO = 99.9% availability
  Error budget = 0.1% (over 30 days = 43.2 minutes of downtime)
  Current error rate = 1% (10x the budget rate)
  Burn rate = 0.01 / 0.001 = 10x

  At 10x burn rate, you'll exhaust your 30-day budget in 3 days.
```

Google recommends **multi-window, multi-burn-rate alerting**:

| Alert | Burn Rate | Long Window | Short Window | Severity |
|-------|-----------|-------------|--------------|----------|
| Page (wake someone up) | 14.4x | 1 hour | 5 minutes | Critical |
| Page | 6x | 6 hours | 30 minutes | Critical |
| Ticket (next business day) | 3x | 1 day | 2 hours | Warning |
| Ticket | 1x | 3 days | 6 hours | Warning |

The dual-window approach prevents alerting on brief spikes (the short window must also be breaching).

```json
{
  "displayName": "SLO Burn Rate - High (Page)",
  "conditions": [
    {
      "displayName": "Burn rate > 14.4x over 1h",
      "conditionThreshold": {
        "filter": "select_slo_burn_rate(\"projects/PROJECT/services/SERVICE/serviceLevelObjectives/SLO_ID\", \"3600s\")",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 14.4,
        "duration": "0s"
      }
    },
    {
      "displayName": "Burn rate > 14.4x over 5m",
      "conditionThreshold": {
        "filter": "select_slo_burn_rate(\"projects/PROJECT/services/SERVICE/serviceLevelObjectives/SLO_ID\", \"300s\")",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 14.4,
        "duration": "0s"
      }
    }
  ],
  "combiner": "AND"
}
```

#### Multi-Condition Alerts

Alerting policies can combine multiple conditions:

- **AND**: All conditions must be true (e.g., high CPU AND high memory)
- **OR**: Any condition triggers (e.g., disk full OR inode exhaustion)

```bash
# Multi-condition alerts are configured via JSON policy files
# The "combiner" field controls AND/OR logic:
# "AND" = all conditions must be met
# "OR" = any condition triggers the alert
```

#### Notification Channels

| Channel Type | Use Case | Latency |
|-------------|----------|---------|
| **Email** | Non-urgent, tickets | Minutes |
| **SMS** | On-call paging | Seconds |
| **Slack** | Team awareness | Seconds |
| **PagerDuty** | Incident management integration | Seconds |
| **Pub/Sub** | Automated remediation, custom integrations | Seconds |
| **Webhooks** | Custom HTTP endpoints | Seconds |
| **Mobile app** | Google Cloud mobile app push notifications | Seconds |

```bash
# Create a Pub/Sub notification channel (for automated remediation)
gcloud monitoring channels create \
  --display-name="Auto-Remediation" \
  --type=pubsub \
  --channel-labels=topic=projects/PROJECT_ID/topics/alert-notifications

# A Cloud Function can subscribe to this topic and auto-remediate
# e.g., restart a VM, scale up an instance group, etc.
```

#### Alert Fatigue Prevention

| Strategy | Implementation |
|----------|---------------|
| **Proper thresholds** | Alert on SLOs, not arbitrary thresholds |
| **Duration windows** | Require condition to persist (e.g., 5 minutes) |
| **Snooze policies** | Temporarily silence known issues during maintenance |
| **Documentation** | Include runbook links in alert documentation field |
| **Severity levels** | Page only for critical; ticket for non-urgent |
| **Alert grouping** | Group related alerts to reduce notification volume |

```bash
# Create a snooze for a maintenance window
gcloud monitoring snoozes create \
  --display-name="Maintenance Window" \
  --criteria-policies=POLICY_ID \
  --start-time="2026-02-20T02:00:00Z" \
  --end-time="2026-02-20T04:00:00Z"
```

**Exam tips:**
- SLO-based burn rate alerting is the **preferred approach** for the PCA exam. If a question asks "how to alert on reliability," choose burn rate over raw threshold.
- Multi-window alerting (long + short window) prevents false positives from brief spikes.
- Pub/Sub notification channels enable **automated remediation** -- this is the answer when a question asks about "auto-healing" or "self-remediation."
- Snooze policies are for planned maintenance. Do not disable alerting policies entirely.
- Alert documentation should include runbook links so on-call engineers know exactly what to do.

**Docs:**
- [SLO burn rate alerting](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/alerting-on-budget-burn-rate)
- [Alerting best practices](https://cloud.google.com/monitoring/alerts/alerting-best-practices)
- [Notification channels](https://cloud.google.com/monitoring/support/notification-options)
- [Snooze policies](https://cloud.google.com/monitoring/alerts/snooze)

---

## 6.3 Deployment and Release Management

### Cloud Deploy

Cloud Deploy is a managed continuous delivery service for GKE, Cloud Run, and Anthos. It follows a pipeline model with explicit promotion between environments.

#### Core Concepts

| Concept | Description |
|---------|-------------|
| **Delivery pipeline** | Defines the progression of environments (dev -> staging -> prod) |
| **Target** | A deployment destination (GKE cluster, Cloud Run service) |
| **Release** | A specific version of your application (immutable) |
| **Rollout** | The act of deploying a release to a target |
| **Promotion** | Moving a release from one target to the next |
| **Approval** | Optional manual gate before promotion |

#### Pipeline Configuration

```yaml
# clouddeploy.yaml -- Delivery pipeline definition
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-app-pipeline
description: "Pipeline for my-app: dev -> staging -> prod"
serialPipeline:
  stages:
    - targetId: dev
      profiles: [dev]
    - targetId: staging
      profiles: [staging]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              serviceNetworking:
                service: my-app-svc
                deployment: my-app
          canaryDeployment:
            percentages: [25, 50, 75]
            verify: true
    - targetId: prod
      profiles: [prod]
      strategy:
        standard:
          verify: true
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: dev
description: "Development cluster"
gke:
  cluster: projects/PROJECT/locations/us-central1/clusters/dev-cluster
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: staging
description: "Staging cluster"
gke:
  cluster: projects/PROJECT/locations/us-central1/clusters/staging-cluster
requireApproval: true
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
description: "Production cluster"
gke:
  cluster: projects/PROJECT/locations/us-central1/clusters/prod-cluster
requireApproval: true
```

```bash
# Register the pipeline and targets
gcloud deploy apply --file=clouddeploy.yaml --region=us-central1

# Create a release (triggered by CI/CD or manually)
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1 \
  --images=my-app=gcr.io/PROJECT/my-app:v1.2.3

# Promote a release to the next stage
gcloud deploy releases promote \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Approve a pending rollout
gcloud deploy rollouts approve rollout-001 \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --to-target=staging \
  --region=us-central1

# Check rollout status
gcloud deploy rollouts list \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Rollback: create a new release pointing to the previous image
gcloud deploy releases create rollback-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1 \
  --images=my-app=gcr.io/PROJECT/my-app:v1.2.2

# Or rollback a specific rollout
gcloud deploy targets rollback prod \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1
```

### Canary Deployments

Canary deployments gradually shift traffic to a new version, allowing you to detect issues before full rollout.

**With Cloud Deploy (GKE):**
```yaml
# In the pipeline stage (see above):
strategy:
  canary:
    runtimeConfig:
      kubernetes:
        serviceNetworking:
          service: my-app-svc
          deployment: my-app
    canaryDeployment:
      percentages: [10, 25, 50, 75]
      verify: true
```

**With Cloud Run (traffic splitting):**
```bash
# Deploy new revision without sending traffic
gcloud run deploy my-service \
  --image=gcr.io/PROJECT/my-app:v2 \
  --no-traffic \
  --region=us-central1

# Split traffic: 90% old, 10% new (canary)
gcloud run services update-traffic my-service \
  --to-revisions=my-service-v2=10 \
  --region=us-central1

# Gradually increase
gcloud run services update-traffic my-service \
  --to-revisions=my-service-v2=50 \
  --region=us-central1

# Full rollout
gcloud run services update-traffic my-service \
  --to-revisions=my-service-v2=100 \
  --region=us-central1

# Rollback instantly
gcloud run services update-traffic my-service \
  --to-revisions=my-service-v1=100 \
  --region=us-central1
```

### Blue-Green Deployments

Blue-green maintains two identical production environments. Traffic is switched all at once from blue (current) to green (new).

**On GKE:**
```yaml
# Blue deployment (current production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
  labels:
    app: my-app
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
        - name: my-app
          image: gcr.io/PROJECT/my-app:v1
---
# Green deployment (new version, deployed alongside blue)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
  labels:
    app: my-app
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
        - name: my-app
          image: gcr.io/PROJECT/my-app:v2
---
# Service pointing to blue (switch selector to green to cutover)
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
    version: blue   # Change to "green" to switch traffic
  ports:
    - port: 80
      targetPort: 8080
```

Cutover: `kubectl patch svc my-app-svc -p '{"spec":{"selector":{"version":"green"}}}'`
Rollback: `kubectl patch svc my-app-svc -p '{"spec":{"selector":{"version":"blue"}}}'`

### GKE Rolling Updates

GKE rolling updates are the default deployment strategy. They gradually replace old pods with new ones.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # At most 1 pod unavailable during update
      maxSurge: 2          # At most 2 extra pods during update
  template:
    spec:
      containers:
        - name: my-app
          image: gcr.io/PROJECT/my-app:v2
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

| Parameter | Description | Default |
|-----------|-------------|---------|
| `maxUnavailable` | Max pods that can be down during update | 25% |
| `maxSurge` | Max extra pods above desired count | 25% |

- `maxUnavailable=0, maxSurge=1`: Safest -- always have full capacity, add 1 new before removing old
- `maxUnavailable=1, maxSurge=0`: No extra resources needed, but 1 pod always unavailable
- Higher values = faster rollouts but more disruption

```bash
# Check rollout status
kubectl rollout status deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to a specific revision
kubectl rollout undo deployment/my-app --to-revision=3

# View rollout history
kubectl rollout history deployment/my-app

# Pause a rollout (for canary-like manual verification)
kubectl rollout pause deployment/my-app

# Resume a paused rollout
kubectl rollout resume deployment/my-app
```

### Deployment Strategy Comparison

| Strategy | Downtime | Rollback Speed | Resource Cost | Risk | Best For |
|----------|----------|---------------|---------------|------|----------|
| **Rolling update** | None | Slow (reverse rollout) | Low (gradual) | Medium | Default K8s, most workloads |
| **Blue-green** | None | Instant (switch service) | High (2x infra) | Low | Critical services, databases |
| **Canary** | None | Instant (shift traffic back) | Medium | Lowest | High-traffic services, risk-averse |
| **Recreate** | Yes | Slow | Low | High | Dev/test, stateful apps requiring clean slate |

### Feature Flags and Progressive Delivery

Feature flags decouple deployment from release:
- Deploy code with flags disabled
- Gradually enable for % of users
- Instant rollback by disabling the flag
- No redeployment needed

GCP does not have a native feature flag service. Common approaches:
- **Firebase Remote Config** (mobile/web apps)
- Third-party: LaunchDarkly, Split, Unleash
- Custom: Firestore or Cloud Memorystore for flag storage

**Exam tips:**
- Cloud Deploy is the answer when a question asks about "managed continuous delivery" or "promotion between environments."
- Cloud Deploy supports **canary** and **standard** strategies natively.
- Cloud Run traffic splitting is the simplest canary approach -- instant, no extra infrastructure.
- Blue-green costs 2x resources but gives instant rollback -- choose when the question emphasizes "zero-downtime rollback."
- Rolling updates are the default for GKE. Know `maxUnavailable` and `maxSurge` behavior.
- `kubectl rollout undo` is for emergency rollbacks. Cloud Deploy rollback creates a new release pointing to an old image.
- If a question mentions "deploy without sending traffic" + "gradually shift traffic" on Cloud Run, the answer is `--no-traffic` + `update-traffic`.

**Docs:**
- [Cloud Deploy overview](https://cloud.google.com/deploy/docs)
- [Cloud Deploy canary deployments](https://cloud.google.com/deploy/docs/deployment-strategies/canary)
- [Cloud Run traffic management](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration)
- [GKE deployment strategies](https://cloud.google.com/kubernetes-engine/docs/concepts/deployment-strategies)
- [Kubernetes rolling updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)

---

## 6.4 Assisting with the Support of Deployed Solutions

### Incident Response Process

Google follows a structured incident response process based on SRE principles:

```
  DETECT          TRIAGE          MITIGATE        RESOLVE         LEARN
  -------         -------         ---------       --------        -----
  Monitoring      Assess          Stop the        Fix root        Post-mortem
  alerts          severity        bleeding        cause           (blameless)
  Uptime checks   Assign IC       Rollback        Deploy fix      Action items
  User reports    Communicate     Scale up        Verify          Update runbooks
  Error spikes    Escalate        Failover        Monitor         Share findings
```

| Phase | Key Activities | GCP Tools |
|-------|---------------|-----------|
| **Detect** | Alert fires, user report, error spike | Cloud Monitoring alerts, Error Reporting, uptime checks |
| **Triage** | Determine severity, assign Incident Commander (IC) | Alert documentation, runbooks |
| **Mitigate** | Reduce impact immediately (rollback, failover, scale) | Cloud Deploy rollback, traffic splitting, autoscaler |
| **Resolve** | Fix the root cause, deploy permanent fix | Cloud Build, Cloud Deploy, configuration changes |
| **Learn** | Blameless post-mortem, action items, update documentation | Google Docs, internal wiki, updated runbooks |

#### Incident Severity Levels

| Severity | Impact | Response | Example |
|----------|--------|----------|---------|
| **P1 (Critical)** | Service fully down, all users affected | Page immediately, all hands | Production database down |
| **P2 (Major)** | Significant feature broken, many users affected | Page during business hours | Payment processing failing |
| **P3 (Minor)** | Partial impact, workaround exists | Ticket, next business day | Dashboard loading slowly |
| **P4 (Low)** | Cosmetic or minor issue | Backlog | Typo in error message |

### Runbooks and Playbooks

| Document | Purpose | When Used |
|----------|---------|-----------|
| **Runbook** | Step-by-step operational procedures | During incidents, routine maintenance |
| **Playbook** | Decision trees for specific alert types | When a specific alert fires |

A good runbook includes:
1. **Alert description**: What triggered this runbook
2. **Impact assessment**: Who/what is affected
3. **Diagnostic steps**: Commands to run, dashboards to check
4. **Mitigation steps**: How to stop the bleeding
5. **Escalation path**: Who to contact if steps fail
6. **Root cause investigation**: How to identify the underlying issue

Example runbook entry:
```markdown
## Alert: High Error Rate (>1% 5xx responses)

### Diagnostics
1. Check error rate dashboard: [link]
2. Check recent deployments: `gcloud deploy releases list --delivery-pipeline=PIPELINE --region=REGION`
3. Check Cloud Logging for error details:
   `gcloud logging read 'severity="ERROR" AND resource.type="k8s_container"' --limit=50`
4. Check pod status: `kubectl get pods -n production`

### Mitigation
1. If caused by recent deployment: rollback
   `gcloud deploy targets rollback prod --delivery-pipeline=PIPELINE --region=REGION`
2. If caused by load spike: scale up
   `kubectl scale deployment my-app --replicas=20 -n production`
3. If caused by downstream dependency: enable circuit breaker / fallback

### Escalation
- On-call SRE: [PagerDuty rotation link]
- Service owner: @team-lead
- VP escalation (P1 only): @vp-engineering
```

### Post-Mortems / Post-Incident Reviews

Key principles of blameless post-mortems:
- **Blameless**: Focus on systems and processes, not individuals
- **Timeline**: Detailed timeline of events (detection, response, resolution)
- **Root cause analysis**: Identify the actual technical failure
- **Contributing factors**: What made the incident worse or delayed resolution
- **Action items**: Concrete, assigned, time-bound improvements
- **Sharing**: Publish internally so other teams learn

Post-mortem template:
```
Title: [Service] [Brief description] on [Date]
Severity: P1/P2/P3
Duration: X hours Y minutes
Impact: [Number of users affected, revenue impact, SLO impact]

Timeline:
- 14:00 UTC: Deployment of v2.3.1 to production
- 14:05 UTC: Error rate alert fires
- 14:10 UTC: IC assigned, investigation begins
- 14:25 UTC: Root cause identified (database connection pool exhausted)
- 14:30 UTC: Rollback initiated
- 14:35 UTC: Service recovered

Root Cause: Connection pool size was not updated for the new database instance.

Action Items:
- [ ] Add connection pool size to deployment checklist (@alice, 2026-02-28)
- [ ] Create alert for connection pool utilization (@bob, 2026-02-22)
- [ ] Load test with production-scale connection counts (@carol, 2026-03-07)
```

### Google Cloud Support Tiers

| Feature | Standard | Enhanced | Premium |
|---------|----------|----------|---------|
| **Price** | $29/mo min | 3% of monthly spend ($500 min) | Contact sales (high) |
| **Response time (P1)** | 4 hours | 1 hour | 15 minutes |
| **Response time (P2)** | 8 hours | 4 hours | 2 hours |
| **Channels** | Web/chat | Web/chat/phone | Web/chat/phone |
| **TAM** | No | No | Yes (named) |
| **Training** | No | No | Yes |
| **Advisory services** | No | No | Yes |
| **Third-party support** | No | No | Yes |
| **Active Assist** | No | Recommendations | Recommendations + review |

#### Technical Account Managers (TAMs)

- Available only with **Premium Support**
- Named, dedicated technical advisor
- Proactive guidance on architecture, operations, migrations
- Conducts operational health reviews
- Helps with capacity planning and incident management
- Coordinates with Google engineering teams

**Exam tips:**
- If a question asks about "reducing MTTR (Mean Time To Recover)," the answer involves runbooks, automated rollbacks, and clear escalation paths.
- **Blameless post-mortems** are always the right answer for "how to handle post-incident review." Never choose answers that assign blame.
- The incident response order is: Detect -> Triage -> Mitigate -> Resolve -> Learn. Mitigation (stop the bleeding) comes BEFORE root cause resolution.
- Premium Support with TAMs is the answer when the question describes a large enterprise needing dedicated technical guidance.
- Enhanced Support is the most common exam answer for "business-critical support" without enterprise scale.

**Docs:**
- [Incident response process](https://sre.google/sre-book/managing-incidents/)
- [Post-mortem culture](https://sre.google/sre-book/postmortem-culture/)
- [Google Cloud Support](https://cloud.google.com/support/docs/premium-support)
- [Runbooks and playbooks](https://sre.google/workbook/on-call/)

---

## 6.5 Evaluating Quality Control Measures

### SLOs, SLIs, and SLAs

These three concepts form the foundation of reliability measurement in SRE. The exam heavily tests the relationship between them.

#### Service Level Indicators (SLIs)

An SLI is **the metric you measure**. It quantifies a specific aspect of service quality.

Common SLI types:

| SLI Type | What It Measures | Example |
|----------|-----------------|---------|
| **Availability** | % of successful requests | 99.95% of HTTP requests return non-5xx |
| **Latency** | % of requests faster than threshold | 99% of requests complete in < 200ms |
| **Throughput** | Rate of successful operations | 10,000 requests/second processed successfully |
| **Error rate** | % of requests that fail | 0.1% of requests return 5xx |
| **Freshness** | Data staleness | 99% of data updated within 1 minute |
| **Durability** | Data loss probability | 99.999999999% of objects preserved (11 nines) |
| **Correctness** | % of correct responses | 99.99% of calculations return correct results |

Good SLIs are:
- **Measurable**: Can be computed from metrics
- **User-facing**: Reflect what users experience
- **Actionable**: Can be improved through engineering effort

#### Service Level Objectives (SLOs)

An SLO is **the target for the SLI**. It defines the acceptable level of reliability.

```
SLO = SLI target over a time window

Examples:
  - "99.9% of HTTP requests will succeed over a 30-day rolling window"
  - "95% of requests will have latency < 200ms, measured monthly"
  - "99.95% availability per calendar month"
```

SLO considerations:
- **Too high**: Expensive, limits feature velocity, may exceed what users notice
- **Too low**: Users become unhappy, business impact
- **Just right**: Balances reliability investment with feature delivery

#### Service Level Agreements (SLAs)

An SLA is **the business commitment** -- a contractual obligation with penalties for non-compliance.

```
Relationship:  SLI (measures) --> SLO (targets) --> SLA (commits)

Example flow:
  SLI: Availability = (successful requests / total requests) * 100
  SLO: 99.9% availability over 30 days (internal target)
  SLA: 99.5% availability per month (external commitment, with credits)

Note: SLA is ALWAYS less stringent than SLO.
      The buffer between SLO and SLA protects the business.
```

#### Concrete Example: E-Commerce Platform

```
Service: Product Search API

SLI Definition:
  availability_sli = (requests returning 2xx or 3xx) / (total requests) * 100
  latency_sli = (requests completing in < 300ms) / (total requests) * 100

SLO Targets (internal):
  Availability: 99.95% over 30-day rolling window
  Latency: 99% of requests < 300ms over 30-day rolling window

SLA Commitment (to customers):
  Availability: 99.9% per calendar month
  Penalty: 10% credit for <99.9%, 25% credit for <99.0%, 50% credit for <95.0%

Error Budget (from SLO):
  Availability budget: 100% - 99.95% = 0.05%
  In 30 days: 0.05% * 30 * 24 * 60 = 21.6 minutes of allowed downtime
```

#### Setting Up SLOs in Cloud Monitoring

```bash
# Create a service (for SLO attachment)
# Services are auto-detected for App Engine, Cloud Run, GKE, Istio

# Create an availability SLO
gcloud monitoring slos create \
  --service=my-service \
  --display-name="Availability SLO" \
  --goal=0.999 \
  --rolling-period=30d \
  --request-based-sli \
  --good-total-ratio-filter='metric.type="loadbalancing.googleapis.com/https/request_count" AND metric.labels.response_code_class="200"' \
  --total-ratio-filter='metric.type="loadbalancing.googleapis.com/https/request_count"'

# List SLOs for a service
gcloud monitoring slos list --service=my-service

# Describe an SLO (shows current compliance)
gcloud monitoring slos describe SLO_ID --service=my-service
```

**Exam tips:**
- **SLI = metric, SLO = target, SLA = contract.** If the question asks "what to measure," it is asking for the SLI. If it asks "what target to set," it is asking for the SLO.
- SLA is always **less stringent** than SLO. The buffer is intentional -- it gives the team room to detect and fix issues before contractual penalties kick in.
- Not every service needs an SLA. SLOs are internal targets; SLAs are external commitments.
- The exam may ask you to identify the correct SLI for a scenario: availability for uptime, latency for response time, freshness for data pipelines.
- 99.9% ("three nines") = 43.8 minutes of downtime per month. 99.99% ("four nines") = 4.38 minutes. Know these numbers.

**Docs:**
- [SLO monitoring in Cloud Monitoring](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring)
- [SRE Book: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [Defining SLIs](https://cloud.google.com/architecture/defining-SLOs)

---

### Error Budgets

The error budget is the complement of the SLO. It represents the **allowed unreliability** over a time window.

```
Error Budget = 1 - SLO

Examples:
  SLO = 99.9%   --> Error Budget = 0.1%  --> 43.2 min/month downtime allowed
  SLO = 99.95%  --> Error Budget = 0.05% --> 21.6 min/month downtime allowed
  SLO = 99.99%  --> Error Budget = 0.01% --> 4.32 min/month downtime allowed
  SLO = 99.5%   --> Error Budget = 0.5%  --> 3.6 hours/month downtime allowed
```

#### Error Budget Calculation Example

```
Service: Payment Processing API
SLO: 99.95% availability over 30 days
Total minutes in 30 days: 30 * 24 * 60 = 43,200 minutes

Error budget = 0.05% * 43,200 = 21.6 minutes

Current month status:
  - Day 1-15: Two incidents totaling 8 minutes downtime
  - Remaining budget: 21.6 - 8 = 13.6 minutes
  - Budget consumed: 8 / 21.6 = 37%
  - Days remaining: 15
  - Burn rate: 37% consumed in 50% of the window = 0.74x (healthy)
```

#### Error Budget Policy

An error budget policy defines what happens when the budget is exhausted or at risk.

```
Error Budget Policy for Payment Service:

When budget is HEALTHY (>50% remaining):
  - Normal feature development velocity
  - Standard deployment cadence (daily)
  - Standard change review process

When budget is WARNING (25-50% remaining):
  - Reduce deployment frequency to 2x/week
  - Require additional review for risky changes
  - Prioritize reliability improvements

When budget is CRITICAL (<25% remaining):
  - Feature freeze -- only reliability improvements deployed
  - All deployments require SRE approval
  - Focus on error budget recovery

When budget is EXHAUSTED (0% remaining):
  - Hard feature freeze
  - All engineering effort directed at reliability
  - Post-mortem for any further incidents
  - Remains frozen until budget recovers naturally
```

#### Balancing Reliability vs Feature Velocity

```
                    Feature Velocity
                         ^
                         |
         "Move fast"     |     ERROR BUDGET
         zone            |     EXHAUSTED
                         |     (Feature freeze)
                         |
  -----------------------+----------------------->
                         |               Reliability
         ERROR BUDGET    |     "Too reliable"
         HEALTHY         |     (over-investing in
         (Ship features) |      reliability)
                         |
```

Key insight: **Being too reliable is wasteful.** If you never consume your error budget, your SLO is too lenient or you are over-investing in reliability at the expense of features. Error budgets create a data-driven negotiation between development teams (who want to ship features) and SRE teams (who want reliability).

**Exam tips:**
- Error budget = 1 - SLO. Memorize: 99.9% = 43.2 min/month, 99.99% = 4.32 min/month.
- When a question says "error budget exhausted," the answer is **feature freeze + focus on reliability**.
- Error budget policies should be agreed upon by both development and SRE teams before issues arise.
- If a service consistently has unused error budget, the SLO may be too lenient -- consider tightening it.
- Burn rate > 1 means you will exhaust the budget before the window ends. This is the basis for burn rate alerting.

**Docs:**
- [Error budgets](https://sre.google/sre-book/embracing-risk/)
- [Error budget policy](https://sre.google/workbook/error-budget-policy/)
- [Alerting on budget burn rate](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/alerting-on-budget-burn-rate)

---

### Capacity Planning

Capacity planning ensures your infrastructure can handle current and future load.

#### Resource Utilization Monitoring

```bash
# View current utilization for a MIG
gcloud compute instance-groups managed describe my-mig \
  --zone=us-central1-a \
  --format="yaml(status.autoscaler)"

# Set up capacity-related alerts
# Example: Alert when CPU utilization > 70% sustained for 10 minutes
# This gives you headroom before hitting limits
```

MQL query for capacity monitoring:
```
# Average CPU utilization across all instances in a MIG
fetch gce_instance
| metric 'compute.googleapis.com/instance/cpu/utilization'
| filter (metadata.user_labels.managed-by == 'my-mig')
| group_by [], [avg_utilization: mean(value.utilization)]
| every 5m
```

#### Growth Forecasting and Headroom Planning

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| **Autoscaling** | Automatically add/remove resources based on metrics | Variable workloads, cloud-native apps |
| **Pre-provisioning** | Provision capacity ahead of expected demand | Predictable spikes (Black Friday, launches) |
| **Headroom planning** | Maintain N% spare capacity at all times | Latency-sensitive services, databases |
| **Committed use discounts** | 1 or 3 year commitments for predictable baseline | Steady-state baseline workloads |

```bash
# Configure autoscaling for a MIG
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --max-num-replicas=20 \
  --min-num-replicas=3 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=300

# Configure autoscaling based on custom metric
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --max-num-replicas=20 \
  --min-num-replicas=3 \
  --custom-metric-utilization=metric=custom.googleapis.com/queue_depth,utilization-target=100,utilization-target-type=GAUGE

# Configure predictive autoscaling (uses ML to predict demand)
gcloud compute instance-groups managed update-autoscaling my-mig \
  --zone=us-central1-a \
  --cpu-utilization-predictive-method=optimize-availability
```

GKE autoscaling layers:
1. **Horizontal Pod Autoscaler (HPA)**: Scales pod replicas based on CPU/memory/custom metrics
2. **Vertical Pod Autoscaler (VPA)**: Adjusts resource requests/limits per pod
3. **Cluster Autoscaler**: Adds/removes nodes based on pod scheduling needs
4. **Node Auto-Provisioning (NAP)**: Creates new node pools with optimal machine types

```bash
# Enable cluster autoscaler
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=3 --max-nodes=20 \
  --zone=us-central1-a

# Enable node auto-provisioning
gcloud container clusters update my-cluster \
  --enable-autoprovisioning \
  --max-cpu=100 --max-memory=256 \
  --zone=us-central1-a

# Configure HPA
kubectl autoscale deployment my-app --cpu-percent=60 --min=3 --max=20
```

**Exam tips:**
- Autoscaling is the default answer for variable workloads. Pre-provisioning is for known events.
- Predictive autoscaling uses ML to scale up BEFORE demand hits -- great for periodic patterns.
- GKE has four layers of autoscaling (HPA, VPA, Cluster Autoscaler, NAP). Know when to use each.
- Committed use discounts are for baseline capacity that does not change. Autoscaling handles the variable portion on top.
- Capacity planning is not just about scaling up -- it also means right-sizing (not over-provisioning).

**Docs:**
- [Compute Engine autoscaling](https://cloud.google.com/compute/docs/autoscaler)
- [Predictive autoscaling](https://cloud.google.com/compute/docs/autoscaler/predictive-autoscaling)
- [GKE autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
- [Capacity planning](https://cloud.google.com/architecture/capacity-planning-and-management)

---

## 6.6 Ensuring Reliability of Solutions in Production

### Chaos Engineering

Chaos engineering is the practice of intentionally introducing failures to test system resilience. The goal is to find weaknesses before they cause real outages.

#### Principles of Chaos Engineering

1. **Start with a hypothesis**: "If we kill 30% of pods, the service should still serve traffic with <500ms latency"
2. **Minimize blast radius**: Start small (single pod), expand gradually
3. **Run in production**: Staging environments do not capture all production behaviors
4. **Automate experiments**: Repeatable, scheduled chaos tests
5. **Stop immediately if needed**: Have kill switches and monitoring in place

#### Fault Injection Testing on GCP

**With Istio / Anthos Service Mesh:**
```yaml
# VirtualService with fault injection
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
    - my-service
  http:
    - fault:
        delay:
          percentage:
            value: 10
          fixedDelay: 5s
        abort:
          percentage:
            value: 5
          httpStatus: 500
      route:
        - destination:
            host: my-service
```

This configuration:
- Adds 5-second delay to 10% of requests (tests timeout handling)
- Returns HTTP 500 for 5% of requests (tests error handling)

**With GKE Pod Disruption:**
```bash
# Manually kill pods to test resilience
kubectl delete pod -l app=my-service --grace-period=0

# Create a PodDisruptionBudget (PDB) to limit blast radius
kubectl apply -f - <<EOF
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-service-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-service
EOF
```

#### Game Days

Game Days are planned exercises where teams intentionally introduce failures and practice incident response.

| Phase | Activities |
|-------|-----------|
| **Planning** | Define scope, scenarios, success criteria, safety mechanisms |
| **Pre-game** | Notify stakeholders, prepare monitoring dashboards, confirm kill switches |
| **Execution** | Run scenarios, observe system behavior, practice incident response |
| **Post-game** | Review findings, document gaps, create action items |

Common Game Day scenarios:
- Zone failure (drain a zone, verify multi-zone resilience)
- Database failover (trigger Cloud SQL failover, measure RTO)
- Network partition (block traffic between services)
- Dependency failure (external API returns errors)
- Certificate expiration (test auto-renewal)
- Load spike (10x normal traffic)

#### Chaos Engineering Tools for GKE

| Tool | Description |
|------|-------------|
| **Litmus** | CNCF chaos engineering framework for Kubernetes |
| **Chaos Mesh** | Kubernetes-native chaos engineering platform |
| **Gremlin** | SaaS chaos engineering platform (commercial) |
| **Pod fault injection** | Native Istio/ASM fault injection |
| **Compute Engine simulate-maintenance** | Test live migration behavior |

```bash
# Simulate a maintenance event on a Compute Engine VM
gcloud compute instances simulate-maintenance-event INSTANCE_NAME \
  --zone=us-central1-a

# This triggers live migration, useful for testing that your app handles it
```

**Exam tips:**
- Chaos engineering is about **proactive reliability** -- finding failures before users do.
- PodDisruptionBudgets (PDBs) protect against over-aggressive chaos (and normal maintenance). Know `minAvailable` and `maxUnavailable`.
- Istio/ASM fault injection is the GCP-native way to inject faults. No extra tools needed.
- Game Days are the organizational practice; chaos engineering is the technical practice. Both are valid exam answers.
- `simulate-maintenance-event` tests VM live migration behavior -- important for latency-sensitive workloads.

**Docs:**
- [Chaos engineering on Google Cloud](https://cloud.google.com/architecture/framework/reliability/test-recovery)
- [Istio fault injection](https://istio.io/latest/docs/tasks/traffic-management/fault-injection/)
- [PodDisruptionBudgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- [Simulate maintenance events](https://cloud.google.com/compute/docs/instances/simulating-host-maintenance)

---

### Penetration Testing

#### Google Cloud Penetration Testing Policy

Google Cloud **allows** penetration testing on your own resources without prior approval. Key rules:
- You CAN pen-test your own VMs, containers, App Engine apps, Cloud Functions, etc.
- You CANNOT pen-test Google's underlying infrastructure
- You MUST stay within your own project boundaries
- No need to notify Google beforehand (unlike some other cloud providers)
- Follow the [Acceptable Use Policy](https://cloud.google.com/terms/aup)

#### Web Security Scanner

Web Security Scanner is a built-in tool that scans App Engine, GKE, and Compute Engine web applications for common vulnerabilities.

```bash
# Web Security Scanner is configured in Console > Security > Web Security Scanner
# It scans for:
# - XSS (Cross-Site Scripting)
# - Flash injection
# - Mixed content (HTTP in HTTPS pages)
# - Outdated/insecure libraries
# - Clear-text passwords
```

Scan types:
- **Managed scans**: Automatically scan App Engine and Cloud Run apps weekly
- **Custom scans**: Configure target URLs, authentication, schedule

#### Security Command Center (SCC)

SCC provides centralized vulnerability and threat detection. Key findings relevant to operations:

| Finding Type | Source | Example |
|-------------|--------|---------|
| **Vulnerabilities** | Security Health Analytics | Open firewall rules, public buckets |
| **Threats** | Event Threat Detection | Unusual API calls, crypto mining |
| **Errors** | Misconfiguration detectors | Missing audit logs, no org policy |
| **Compliance** | CIS Benchmarks, PCI-DSS | Non-compliant resource configurations |

```bash
# List SCC findings
gcloud scc findings list ORGANIZATION_ID \
  --source="-" \
  --filter='state="ACTIVE" AND severity="HIGH"'

# Get SCC notification configs
gcloud scc notifications list ORGANIZATION_ID
```

SCC tiers:
- **Standard**: Free, basic security findings
- **Premium**: Threat detection, compliance monitoring, vulnerability scanning
- **Enterprise**: Advanced threat intelligence, attack path simulation

**Exam tips:**
- You do NOT need Google's permission to pen-test your own resources. This is a common exam question.
- Web Security Scanner scans for OWASP vulnerabilities in web apps. It does not scan infrastructure.
- Security Command Center is the centralized security dashboard. The exam often asks about it in the context of "security posture management."
- SCC Premium is required for Event Threat Detection and compliance reporting.

**Docs:**
- [Penetration testing on Google Cloud](https://cloud.google.com/terms/aup)
- [Web Security Scanner](https://cloud.google.com/security-command-center/docs/concepts-web-security-scanner-overview)
- [Security Command Center](https://cloud.google.com/security-command-center/docs)

---

### Load Testing

Load testing validates that your architecture can handle expected and unexpected traffic volumes.

#### Load Testing Methodology

```
1. Define objectives
   - Target throughput (requests/second)
   - Acceptable latency (p50, p95, p99)
   - Maximum error rate

2. Baseline measurement
   - Current performance under normal load
   - Resource utilization at baseline

3. Ramp-up test
   - Gradually increase load
   - Identify the breaking point

4. Sustained load test
   - Hold at target throughput for extended period (30-60 min)
   - Watch for memory leaks, connection pool exhaustion

5. Spike test
   - Sudden 10x traffic increase
   - Test autoscaling response time

6. Soak test
   - Moderate load over long period (hours/days)
   - Identify slow resource leaks
```

#### Load Testing Tools

| Tool | Type | Best For |
|------|------|----------|
| **Locust** | Open source (Python) | Teams familiar with Python, custom scenarios |
| **k6** | Open source (JavaScript) | Developer-friendly, CI/CD integration |
| **JMeter** | Open source (Java) | Complex scenarios, GUI-based test creation |
| **Cloud Run Load Testing** | GCP-native pattern | Distributed load gen using Cloud Run jobs |
| **Distributed load testing on GKE** | GCP reference architecture | Large-scale tests (millions of RPS) |

Distributed load testing architecture on GKE:
```
                    +------------------+
                    |  Cloud Build     |
                    |  (orchestrator)  |
                    +--------+---------+
                             |
                    +--------v---------+
                    |    GKE Cluster   |
                    |                  |
            +-------+------+------+-------+
            |       |      |      |       |
         +--v--+ +--v--+ +-v---+ +--v--+
         |Locust| |Locust| |Locust| |Locust|
         |Worker| |Worker| |Worker| |Worker|
         +--+--+ +--+--+ +--+--+ +--+--+
            |       |      |       |
            +-------+------+-------+
                    |
            +-------v-------+
            | Target Service |
            +----------------+
```

```bash
# Example: Run Locust on GKE for distributed load testing
# Deploy Locust master
kubectl apply -f locust-master.yaml

# Deploy Locust workers (scale for more load)
kubectl apply -f locust-workers.yaml
kubectl scale deployment locust-worker --replicas=20

# Example: k6 load test script
# k6 run --vus 100 --duration 30m load-test.js
```

**Exam tips:**
- Load testing should happen in a **production-like environment**, not just staging, for accurate results.
- The GCP reference architecture for distributed load testing uses GKE -- this is the recommended approach for large-scale tests.
- Always monitor your target service's metrics during load tests (CPU, memory, latency, error rate).
- Spike tests validate autoscaling; soak tests find slow leaks. Know the difference.
- Load test results should inform capacity planning and autoscaling configuration.

**Docs:**
- [Distributed load testing on GKE](https://cloud.google.com/architecture/distributed-load-testing-using-gke)
- [Load testing Cloud Run](https://cloud.google.com/run/docs/testing/load-testing)
- [Performance testing best practices](https://cloud.google.com/architecture/framework/performance-optimization/testing)

---

### Production Maintenance

#### GKE Cluster Upgrades

GKE offers three release channels that determine when your cluster receives updates:

| Release Channel | Description | Update Cadence | Best For |
|----------------|-------------|----------------|----------|
| **Rapid** | Newest features and patches first | Weeks after GA | Dev/test, early adopters |
| **Regular** | Balanced stability and features | 2-3 months after Rapid | Most production workloads |
| **Stable** | Most proven, longest lag | 2-3 months after Regular | Risk-averse, regulated industries |
| **Extended** | Long-term support for a specific minor version | Security patches only (up to 24 months) | Workloads requiring version stability |
| **No channel** | Manual control, no auto-upgrades | You decide | Special requirements |

```bash
# Create a cluster on the Regular channel
gcloud container clusters create my-cluster \
  --release-channel=regular \
  --zone=us-central1-a

# Change release channel
gcloud container clusters update my-cluster \
  --release-channel=stable \
  --zone=us-central1-a

# Check available versions for a channel
gcloud container get-server-config --zone=us-central1-a

# View cluster version and update status
gcloud container clusters describe my-cluster \
  --zone=us-central1-a \
  --format="yaml(currentMasterVersion,currentNodeVersion,releaseChannel)"
```

#### GKE Maintenance Windows and Exclusions

Maintenance windows define **when** GKE can perform automatic upgrades.

```bash
# Set a maintenance window (e.g., weekdays 2-6 AM UTC)
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --maintenance-window-start=2026-01-01T02:00:00Z \
  --maintenance-window-end=2026-01-01T06:00:00Z \
  --maintenance-window-recurrence="FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR"

# Add a maintenance exclusion (e.g., holiday freeze)
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --add-maintenance-exclusion-name=holiday-freeze \
  --add-maintenance-exclusion-start=2026-12-20T00:00:00Z \
  --add-maintenance-exclusion-end=2027-01-05T00:00:00Z \
  --add-maintenance-exclusion-scope=no_upgrades

# Remove a maintenance exclusion
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --remove-maintenance-exclusion=holiday-freeze
```

Maintenance exclusion scopes:
- `no_upgrades`: No control plane or node upgrades
- `no_minor_upgrades`: Patch upgrades allowed, minor version upgrades blocked
- `no_minor_or_node_upgrades`: Only control plane patch upgrades allowed

#### GKE Node Upgrade Strategies

```bash
# Surge upgrade (default) -- adds extra nodes during upgrade
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --max-surge-upgrade=1 \
  --max-unavailable-upgrade=0

# Blue-green upgrade -- creates entirely new node pool
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-blue-green-upgrade \
  --node-pool-soak-duration=3600s  # 1 hour soak time in new pool
```

| Upgrade Strategy | How It Works | Downtime Risk | Cost |
|-----------------|-------------|---------------|------|
| **Surge upgrade** | Adds `max-surge` extra nodes, drains and removes old ones | Low (with PDBs) | Temporary extra node cost |
| **Blue-green upgrade** | Creates new node pool, migrates pods, deletes old pool | Very low | 2x node cost during upgrade |

#### Cloud SQL Maintenance Windows

```bash
# Set maintenance window for Cloud SQL
gcloud sql instances patch my-instance \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=2

# Deny maintenance during a period
gcloud sql instances patch my-instance \
  --deny-maintenance-period-start-date=2026-12-20 \
  --deny-maintenance-period-end-date=2027-01-05 \
  --deny-maintenance-period-time=00:00:00

# Check maintenance schedule
gcloud sql instances describe my-instance \
  --format="yaml(settings.maintenanceWindow,scheduledMaintenance)"
```

Cloud SQL maintenance behavior:
- Maintenance updates include patches, minor version upgrades, infrastructure updates
- During maintenance, the instance restarts (brief downtime, typically < 60 seconds)
- High Availability instances perform failover during maintenance (minimal disruption)
- You can set a preferred day and hour, but Google may override in emergencies

#### Rolling Updates for Managed Instance Groups (MIGs)

```bash
# Update a MIG with a rolling update
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=new-template \
  --zone=us-central1-a \
  --max-unavailable=1 \
  --max-surge=1

# Canary update (partial rollout)
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=current-template \
  --canary-version=template=new-template,target-size=20% \
  --zone=us-central1-a

# Check update status
gcloud compute instance-groups managed describe my-mig \
  --zone=us-central1-a \
  --format="yaml(status.versionTarget,status.isStable)"

# Stop (cancel) an in-progress update
gcloud compute instance-groups managed rolling-action stop-proactive-update my-mig \
  --zone=us-central1-a
```

#### OS Patch Management with VM Manager

VM Manager provides automated OS patch management for Compute Engine VMs.

```bash
# Enable VM Manager (OS Config agent)
gcloud compute project-info add-metadata \
  --metadata=enable-os-config=TRUE

# Create a patch deployment (recurring)
gcloud compute os-config patch-deployments create weekly-patches \
  --instance-filter-names="zones/us-central1-a/instances/vm-1,zones/us-central1-a/instances/vm-2" \
  --recurring-schedule-frequency=WEEKLY \
  --recurring-schedule-time-of-day="02:00" \
  --recurring-schedule-time-zone="America/New_York" \
  --recurring-schedule-day-of-week=SUNDAY \
  --patch-config-reboot-config=DEFAULT \
  --rollout-mode=zone-by-zone \
  --rollout-disruption-budget=20%

# Create a one-time patch job
gcloud compute os-config patch-jobs execute \
  --instance-filter-names="zones/us-central1-a/instances/vm-1" \
  --reboot-config=DEFAULT

# List patch jobs
gcloud compute os-config patch-jobs list

# Describe a patch job
gcloud compute os-config patch-jobs describe JOB_ID
```

#### Production Maintenance Decision Matrix

| Scenario | Strategy | Downtime |
|----------|----------|----------|
| GKE node OS update | Surge upgrade (default) | None (with PDBs) |
| GKE major version upgrade | Blue-green node pool upgrade | None |
| Cloud SQL patch | Maintenance window + HA failover | < 60 seconds |
| MIG template update | Rolling update with canary | None |
| VM OS patching | VM Manager patch deployment | Brief restart per VM |
| Stateful workload upgrade | Blue-green with data sync | None |
| Emergency security patch | Immediate forced update | Possible brief disruption |

**Exam tips:**
- **GKE release channels**: Regular is the default and recommended for most production. Stable is for regulated industries. Extended is for version pinning.
- Maintenance windows define *when* upgrades can happen, not *if*. GKE will still auto-upgrade -- just within your window.
- Maintenance exclusions cannot exceed 180 days for `no_upgrades` scope.
- **Blue-green node pool upgrade** is the safest GKE upgrade strategy but costs 2x during the upgrade window.
- Cloud SQL HA instances failover during maintenance, reducing downtime. This is a common exam justification for HA configuration.
- VM Manager / OS Config is the answer for "how to patch VMs at scale." Do not confuse with manual SSH + `apt-get upgrade`.
- MIG rolling updates with `--canary-version` let you test a new template on a subset before full rollout.
- PodDisruptionBudgets ensure pods are not all killed at once during node upgrades. Always set PDBs for production workloads.

**Docs:**
- [GKE release channels](https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels)
- [GKE maintenance windows](https://cloud.google.com/kubernetes-engine/docs/concepts/maintenance-windows-and-exclusions)
- [GKE node upgrade strategies](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pool-upgrade-strategies)
- [Cloud SQL maintenance](https://cloud.google.com/sql/docs/mysql/maintenance)
- [MIG rolling updates](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups)
- [VM Manager OS patch management](https://cloud.google.com/compute/docs/os-patch-management)
- [PodDisruptionBudgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets)

---

## Quick Reference: Section 6 Decision Tree

```
Question about observability?
├── "How to find slow requests?"           --> Cloud Trace
├── "How to find high CPU in production?"  --> Cloud Profiler
├── "How to track production errors?"      --> Error Reporting
├── "How to monitor metrics/dashboards?"   --> Cloud Monitoring
├── "How to analyze logs with SQL?"        --> Log Analytics
├── "How to route logs to BigQuery?"       --> Log Router sink
├── "How to reduce logging costs?"         --> Exclusion filters on _Default sink
├── "How to monitor K8s with Prometheus?"  --> Managed Service for Prometheus (GMP)
└── "How to collect VM logs/metrics?"      --> Ops Agent

Question about deployment?
├── "Managed CD with promotions?"          --> Cloud Deploy
├── "Gradual traffic shift on Cloud Run?"  --> Traffic splitting (--no-traffic + update-traffic)
├── "Zero-downtime rollback?"              --> Blue-green deployment
├── "Reduce blast radius of deploy?"       --> Canary deployment
├── "Default K8s deploy strategy?"         --> Rolling update (maxUnavailable/maxSurge)
└── "Emergency rollback on K8s?"           --> kubectl rollout undo

Question about reliability?
├── "How to measure reliability?"          --> SLIs (define the metric)
├── "How to set reliability targets?"      --> SLOs (set the target)
├── "Contractual reliability commitment?"  --> SLA (business agreement)
├── "When to freeze features?"             --> Error budget exhausted
├── "How to alert on reliability?"         --> SLO burn rate alerting
├── "How to test resilience?"              --> Chaos engineering / Game Days
├── "How to handle post-incident?"         --> Blameless post-mortem
└── "How to upgrade GKE safely?"           --> Release channels + maintenance windows + PDBs

Question about support?
├── "15-min P1 response time?"             --> Premium Support
├── "Dedicated technical advisor?"         --> TAM (Premium only)
├── "Business-critical support?"           --> Enhanced Support
└── "Basic support for small team?"        --> Standard Support
```

---

## Section 6 Exam Trap Summary

| Trap | Correct Understanding |
|------|----------------------|
| "SLA and SLO are the same thing" | SLO is internal target; SLA is external contract with penalties. SLA < SLO always. |
| "Alert on CPU > 80%" | Prefer SLO-based burn rate alerting over raw metric thresholds. |
| "Error budget means you should have errors" | Error budget is the *allowed* unreliability. It enables data-driven tradeoffs. |
| "_Required bucket can be modified" | _Required is immutable: 400 days, Admin Activity + System Event logs. Cannot change. |
| "Data Access logs are on by default" | OFF by default (except BigQuery). Must explicitly enable. High volume = high cost. |
| "Blue-green is always better than canary" | Blue-green costs 2x. Canary is cheaper and catches issues early. Choose based on requirements. |
| "You need Google permission for pen testing" | No prior approval needed for testing your own resources. |
| "Cloud Trace and Cloud Profiler do the same thing" | Trace = distributed request latency across services. Profiler = CPU/memory within a single service. |
| "Maintenance exclusions stop all upgrades forever" | Maximum 180 days for `no_upgrades` scope. Cannot indefinitely block. |
| "Ops Agent is only for logging" | Ops Agent handles BOTH logging AND metrics collection. It replaces both legacy agents. |
| "Uptime checks run from your VMs" | They run from Google's global infrastructure, checking your service externally. |
| "Feature freeze is a punishment" | It is a data-driven response to exhausted error budget, not punitive. |
