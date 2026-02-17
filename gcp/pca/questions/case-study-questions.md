# Case Study Questions

> **Exam context:** The PCA exam includes 4 published case studies. Two appear on each exam. 20-30% of questions reference case studies. Each question provides the case study context, so you don't need to memorize details, but familiarity saves time.
>
> See [docs/08-case-studies.md](../docs/08-case-studies.md) for full case study analysis.

**Total questions: 20**

---

## EHR Healthcare (Questions 1-5)

> **Case context:** EHR Healthcare provides electronic health record software. They're migrating from a colocation facility to Google Cloud. Key themes: HIPAA compliance, containerized + Windows workloads, Active Directory integration, 99.9% SLA, hybrid connectivity, healthcare APIs.

### Q1. EHR Healthcare runs a mix of containerized Linux services and legacy Windows-based applications in their colocation facility. The containerized services use Kubernetes, while the Windows applications depend on .NET Framework 4.x and cannot be refactored in the near term. Which migration approach best meets EHR Healthcare's requirements?

A) Migrate all workloads to GKE using Windows Server node pools for .NET apps and Linux node pools for containerized services. Use Migrate to Containers for the Windows applications.

B) Migrate containerized services to GKE Autopilot. Use Migrate to Virtual Machines to move Windows workloads to Compute Engine instances. Establish a phased refactoring plan for the Windows apps.

C) Rewrite all Windows applications as Linux containers before migration, then deploy everything to GKE Standard with Linux node pools only.

D) Migrate all workloads to Cloud Run. Use Cloud Run's Windows container support for the .NET Framework applications.

<details>
<summary>Answer</summary>

**Correct: B)**

GKE Autopilot is the best fit for the already-containerized Linux services -- it reduces operational overhead and aligns with EHR's goal of managed infrastructure. The Windows .NET Framework 4.x applications cannot run in Linux containers and cannot be easily refactored, so Migrate to Virtual Machines (moving them as-is to Compute Engine) is the pragmatic first step. This gives EHR a clear lift-and-shift path for Windows while modernizing the container workloads.

- **A is wrong:** While GKE does support Windows Server node pools, Migrate to Containers for Windows has significant limitations with .NET Framework 4.x apps that depend on full Windows Server features. Also, GKE Autopilot (not Standard) better meets the managed-infrastructure goal, and Autopilot does not support Windows node pools.
- **C is wrong:** Rewriting all Windows applications before migration contradicts the requirement that they "cannot be refactored in the near term." This would delay the migration significantly.
- **D is wrong:** Cloud Run does not support Windows containers. It only supports Linux-based containers.

**Exam tip:** When a case study mentions both containerized and legacy VM workloads, the answer almost always involves a split strategy -- GKE for containers, Compute Engine for VMs that can't be containerized yet. Look for "cannot be refactored" as the signal.
</details>

---

### Q2. EHR Healthcare must ensure their Google Cloud environment meets HIPAA compliance requirements for storing and processing electronic health records. The security team requires encryption key control, comprehensive audit logging, and guardrails that prevent accidental deployment of resources outside compliant configurations. Which combination of services should the architect implement? (Choose TWO.)

A) Enable Assured Workloads for Healthcare to create a compliant environment with organizational policy guardrails, and configure Cloud Audit Logs with Data Access logs exported to a locked Cloud Storage bucket.

B) Use Google-managed encryption keys (GMEK) for all storage services and rely on Access Transparency logs for compliance auditing.

C) Configure Cloud Key Management Service (CMEK) for all data-at-rest encryption so EHR Healthcare controls key rotation, access policies, and key destruction schedules.

D) Deploy all resources into a single project with VPC Service Controls and use default encryption with no additional key management.

E) Use Customer-Supplied Encryption Keys (CSEK) for all services to maintain maximum control over encryption keys.

<details>
<summary>Answer</summary>

**Correct: A) and C)**

EHR Healthcare's HIPAA requirements demand both organizational guardrails and encryption key control. **Assured Workloads for Healthcare** provides the compliance boundary -- it enforces organizational policies that restrict resource locations, disable non-compliant services, and ensure data residency. Combined with **Cloud Audit Logs** (including Data Access logs), this creates the audit trail HIPAA mandates. **CMEK** via Cloud KMS gives EHR control over encryption keys -- they can manage rotation schedules, set IAM policies on keys, and control key lifecycle, which is a common HIPAA requirement for covered entities.

- **B is wrong:** GMEK (Google-managed keys) means Google controls the keys. HIPAA-regulated organizations typically require customer-managed keys for demonstrable control. Access Transparency logs show when Google accesses data but are not a substitute for Cloud Audit Logs.
- **D is wrong:** A single project with default encryption provides no key control and no compliance guardrails. VPC Service Controls help with data exfiltration prevention but do not replace Assured Workloads for compliance posture.
- **E is wrong:** CSEK requires EHR to supply keys with every API call and manage their own key storage infrastructure. This adds significant operational complexity and risk (lost keys = lost data). CSEK is also not supported by all Google Cloud services, making it impractical as a blanket strategy. CMEK provides the right balance of control and manageability.

**Exam tip:** For healthcare/HIPAA questions, the winning trio is Assured Workloads + CMEK + Cloud Audit Logs (with Data Access enabled). CSEK is a distractor -- it's overkill and operationally fragile for most organizations.
</details>

---

### Q3. EHR Healthcare requires hybrid connectivity between their colocation facility and Google Cloud during the multi-year migration. The connection must support the 99.9% uptime SLA, handle consistent 5 Gbps throughput for health record synchronization, and comply with HIPAA data-in-transit requirements. Which connectivity design should the architect recommend?

A) Configure two Dedicated Interconnect connections in the same metro with ECMP routing. Encrypt traffic at the application layer using TLS for all health record transfers.

B) Set up a single Partner Interconnect connection with 10 Gbps capacity and configure Cloud VPN as a backup. Use MACsec on the Interconnect link.

C) Deploy redundant HA VPN gateways with multiple tunnels across two regions. Use BGP for dynamic routing and rely on IPsec encryption provided by the VPN tunnels.

D) Provision two Dedicated Interconnect connections in different metro areas (edge availability domains) for 99.99% SLA. Add a Cloud HA VPN overlay on top of the Interconnect for encrypted transit and as a failover path.

<details>
<summary>Answer</summary>

**Correct: D)**

This design addresses all three requirements. Two Dedicated Interconnect connections in different edge availability domains provide the highest availability (up to 99.99% SLA, exceeding the 99.9% requirement). At 5 Gbps consistent throughput, Dedicated Interconnect (minimum 10 Gbps per connection) easily handles the load. The HA VPN overlay running over the Interconnect connections provides IPsec encryption for data in transit, meeting HIPAA requirements. The VPN also serves as an encrypted failover path.

- **A is wrong:** Two Interconnect connections in the same metro only achieves 99.9% SLA (not leveraging different edge availability domains for maximum resilience). More critically, application-layer TLS alone may not satisfy HIPAA data-in-transit requirements at the network level -- a VPN overlay or MACsec provides more comprehensive encryption.
- **B is wrong:** A single Partner Interconnect is a single point of failure and does not meet the 99.9% SLA on its own. MACsec availability depends on the partner/location and is not guaranteed.
- **C is wrong:** HA VPN maxes out at ~3 Gbps per tunnel. While multiple tunnels can aggregate throughput, achieving consistent 5 Gbps with headroom requires many tunnels and is operationally complex. Dedicated Interconnect is the right choice for this throughput level.

**Exam tip:** When you see consistent throughput above 3 Gbps + high availability + encryption in transit, the answer is Dedicated Interconnect + HA VPN overlay. VPN alone can't handle the throughput; Interconnect alone doesn't encrypt. The combo solves both.
</details>

---

### Q4. EHR Healthcare uses on-premises Active Directory (AD) to manage user identities for over 2,000 employees. During the migration to Google Cloud, they need to maintain a single source of identity truth while enabling employees to access both on-premises and Google Cloud resources with their existing credentials. The solution must support group-based access control in Google Cloud IAM. What should the architect recommend?

A) Deploy Google Cloud Directory Sync (GCDS) to synchronize AD users and groups to Cloud Identity. Configure AD Federation Services (AD FS) with SAML 2.0 for single sign-on to Google Cloud. Map AD security groups to Google Cloud IAM roles.

B) Manually create Google Cloud accounts for all users in Cloud Identity. Configure each user with a separate password and use Cloud IAM to replicate the AD group structure.

C) Deploy Managed Microsoft AD in Google Cloud and establish a trust relationship with the on-premises AD forest. Use Managed AD as the sole identity provider for Google Cloud IAM.

D) Migrate all users from Active Directory to Google Workspace accounts. Decommission the on-premises AD and use Google Workspace as the identity provider for all environments.

<details>
<summary>Answer</summary>

**Correct: A)**

GCDS + AD FS is the standard pattern for integrating on-premises Active Directory with Google Cloud while keeping AD as the single source of truth. GCDS performs one-way synchronization of users and groups from AD to Cloud Identity (no passwords are synced). AD FS handles federated authentication via SAML 2.0, so users sign in with their existing AD credentials. Synchronized AD security groups can be referenced in Google Cloud IAM policy bindings for group-based RBAC.

- **B is wrong:** Manually creating 2,000+ accounts is not scalable, introduces identity drift, and requires users to manage separate passwords. This violates the single-source-of-truth requirement.
- **C is wrong:** Managed Microsoft AD is designed for AD-dependent workloads running on Google Cloud (e.g., Windows VMs that need domain join). It does not serve as an identity provider for Google Cloud IAM -- you still need Cloud Identity for IAM. It's a complementary service, not a replacement for GCDS + federation.
- **D is wrong:** Decommissioning on-premises AD during an active multi-year migration would break on-premises workloads that depend on AD. This also violates the requirement to maintain AD as the source of truth.

**Exam tip:** AD integration on the PCA exam almost always follows the GCDS + SAML federation pattern. Managed Microsoft AD is a distractor -- it's for Windows workloads that need AD domain services, not for IAM identity federation.
</details>

---

### Q5. EHR Healthcare requires a disaster recovery strategy for their patient record system. The business mandates an RPO of 1 hour and an RTO of 15 minutes. The primary deployment runs on GKE in us-central1 with Cloud SQL for PostgreSQL as the database. Which DR architecture meets these requirements?

A) Configure Cloud SQL cross-region read replicas in us-east1 with automatic failover promotion. Deploy a standby GKE cluster in us-east1 using Multi Cluster Ingress. Use Cloud DNS routing policies for automated failover.

B) Take daily Cloud SQL exports to a Cloud Storage bucket in a separate region. In the event of a disaster, import the backup to a new Cloud SQL instance and redeploy the GKE workloads from CI/CD.

C) Enable Cloud SQL point-in-time recovery (PITR) and store automated backups in a multi-region Cloud Storage bucket. Rely on GKE Autopilot in the primary region to self-heal from failures.

D) Use Cloud Spanner instead of Cloud SQL for automatic multi-region replication with zero RPO. Deploy GKE clusters in three regions with global load balancing.

<details>
<summary>Answer</summary>

**Correct: A)**

Cloud SQL cross-region read replicas provide continuous asynchronous replication (typically seconds of lag, well within the 1-hour RPO). When the primary fails, the replica can be promoted to a standalone primary. A pre-provisioned standby GKE cluster in us-east1 (warm standby) with Multi Cluster Ingress enables rapid traffic shifting, meeting the 15-minute RTO. Cloud DNS routing policies (failover routing) automate the DNS cutover.

- **B is wrong:** Daily exports give a maximum RPO of 24 hours, far exceeding the 1-hour requirement. Importing a backup and redeploying GKE workloads from CI/CD would take well over 15 minutes, failing the RTO requirement.
- **C is wrong:** PITR and backups in multi-region storage can meet the RPO, but "self-heal in the primary region" does not protect against a regional outage. If us-central1 goes down, there is no standby region to fail over to, so the RTO cannot be met.
- **D is wrong:** While Cloud Spanner would provide excellent multi-region availability, migrating from Cloud SQL for PostgreSQL to Spanner is a significant application rewrite (different data model, different query patterns). The question asks about DR architecture for the existing system, not a database re-platforming project. This option is disproportionate to the stated requirements.

**Exam tip:** Match DR tier to RPO/RTO. Daily backups = RPO in hours/days. Cross-region replicas = RPO in seconds/minutes. For RTO under 30 minutes, you need a warm or hot standby, not a rebuild-from-scratch approach. If the exam offers Spanner as a "just switch databases" answer, check whether the case study supports a major migration effort.
</details>

---

## Cymbal Retail (Questions 6-10)

> **Case context:** Cymbal Retail is implementing gen AI for catalog enrichment, conversational commerce, and product discovery. Key themes: Oracle migration, microservices, AI agents, search, API-first, scaling for peak events.

### Q6. Cymbal Retail wants to use generative AI to automatically enrich product catalog entries with improved descriptions, extracted attributes, and generated tags. The catalog contains 5 million SKUs and is updated daily with 10,000-50,000 new or modified items. The enrichment pipeline must be cost-effective and handle variable throughput. Which architecture should the architect design?

A) Deploy a custom fine-tuned open-source LLM on a GKE cluster with GPU node pools. Build a Pub/Sub-triggered pipeline that processes catalog updates through the model in real time. Store enriched data in Firestore.

B) Use Vertex AI batch prediction with Gemini models. Trigger a daily Cloud Workflow that extracts changed catalog entries from the source database, submits them as a batch prediction job, and writes enriched results back to AlloyDB. Use Pub/Sub to handle intra-day updates with online predictions.

C) Call the Gemini API directly from the catalog management application for each product update synchronously. Cache results in Memorystore to avoid duplicate processing.

D) Export the entire catalog to BigQuery nightly, run a Vertex AI batch prediction across all 5 million SKUs, and overwrite the catalog database with enriched results each morning.

<details>
<summary>Answer</summary>

**Correct: B)**

This architecture balances cost and freshness. Vertex AI batch prediction with Gemini handles the bulk of daily enrichment cost-effectively (batch pricing is significantly cheaper than online prediction). Cloud Workflows orchestrates the pipeline. For intra-day updates (new products that need immediate enrichment), Pub/Sub triggers online predictions -- this hybrid approach ensures new items are enriched quickly without running expensive real-time inference for the entire catalog. AlloyDB is a strong choice as it aligns with the Oracle-to-PostgreSQL migration path.

- **A is wrong:** Hosting and managing a custom LLM on GKE with GPUs introduces significant operational overhead (GPU provisioning, model serving, scaling). For catalog enrichment using standard capabilities (descriptions, attributes, tags), Gemini via Vertex AI is more cost-effective and easier to manage than self-hosted models.
- **C is wrong:** Synchronous API calls for each product update would be slow for bulk operations, expensive at 10,000-50,000 daily updates, and would make the catalog application dependent on API latency and availability. Memorystore caching doesn't help because each product's enrichment is unique.
- **D is wrong:** Reprocessing all 5 million SKUs nightly is extremely wasteful and expensive when only 10,000-50,000 change daily. Overwriting the entire catalog also risks data loss if the batch job fails midway.

**Exam tip:** When you see "variable throughput" + "cost-effective" in an AI pipeline question, the answer usually combines batch processing for bulk work with online/streaming for real-time needs. Pure real-time is expensive; pure batch is stale.
</details>

---

### Q7. Cymbal Retail is migrating from Oracle Database to Google Cloud. The application uses complex PL/SQL stored procedures, Oracle-specific features (partitioning, materialized views), and handles 50,000 transactions per second during peak holiday events. The team wants to minimize application code changes. Which database migration strategy should the architect recommend?

A) Migrate to AlloyDB for PostgreSQL using the Database Migration Service (DMS). Refactor PL/SQL stored procedures to PL/pgSQL. Use AlloyDB's columnar engine for analytical queries and its PostgreSQL compatibility for transactional workloads.

B) Migrate directly to Cloud Spanner for unlimited horizontal scaling. Rewrite all stored procedures as application-level logic in microservices.

C) Use Bare Metal Solution to run Oracle Database on Google Cloud infrastructure. Maintain existing PL/SQL and Oracle features while gaining Google Cloud network proximity.

D) Migrate to Cloud SQL for PostgreSQL Enterprise Plus. Use pgLoader for automated schema and data migration. Deploy read replicas for peak scaling.

<details>
<summary>Answer</summary>

**Correct: A)**

AlloyDB for PostgreSQL is Google's answer for Oracle migrations that need high transactional throughput with minimal code changes. It is PostgreSQL-compatible, so PL/SQL procedures can be converted to PL/pgSQL (many constructs map directly). AlloyDB supports PostgreSQL-native partitioning and materialized views, covering the Oracle features mentioned. Its architecture (disaggregated compute/storage, columnar engine) handles high transaction rates and mixed workloads. DMS provides a managed migration path with minimal downtime.

- **B is wrong:** Spanner requires a complete rewrite of the data model (no stored procedures, different SQL dialect, interleaved tables). While it scales horizontally, the requirement to "minimize application code changes" makes this a poor fit. Rewriting all PL/SQL logic into microservices is a massive effort.
- **C is wrong:** Bare Metal Solution keeps Oracle running but does not achieve the goal of migrating to a cloud-native database. It maintains Oracle licensing costs and the operational model of managing Oracle. It's a valid interim step but not a migration strategy.
- **D is wrong:** Cloud SQL for PostgreSQL Enterprise Plus has a 64 TB storage limit and a throughput ceiling that may not handle 50,000 TPS during peak events. AlloyDB's disaggregated architecture is specifically designed for this performance tier. Cloud SQL read replicas help with read scaling but not write-heavy transactional workloads.

**Exam tip:** Oracle migration questions on the PCA almost always point to AlloyDB when the requirements mention high throughput + stored procedures + minimize changes. Spanner is wrong if they say "minimize code changes." Bare Metal Solution is wrong if the goal is cloud-native migration.
</details>

---

### Q8. Cymbal Retail wants to build a conversational AI shopping assistant that can answer product questions, check inventory, process returns, and make personalized recommendations. The assistant must integrate with existing backend APIs (inventory, orders, CRM) and support both web chat and voice channels. Which architecture should the architect design?

A) Build a custom chatbot using Dialogflow CX with hand-crafted intents and training phrases for each product category. Connect to backend systems using webhook fulfillment. Deploy separate bots for web and voice.

B) Use Vertex AI Agent Builder to create a conversational agent grounded in the product catalog and connected to backend APIs via tools (function calling). Use Vertex AI Search for product discovery. Deploy to web and voice channels using Agent Builder's multi-channel integration.

C) Deploy a fine-tuned Gemini model on Vertex AI Endpoints. Build custom orchestration logic to route queries to backend APIs. Implement conversation management and channel routing in a custom application layer.

D) Use a third-party chatbot platform hosted on GKE. Integrate with Google Cloud APIs for NLU processing and connect to backend services through a custom API gateway.

<details>
<summary>Answer</summary>

**Correct: B)**

Vertex AI Agent Builder is purpose-built for this use case. It provides: (1) conversational agents powered by Gemini with grounding in enterprise data (the product catalog), (2) tool/function-calling capabilities to integrate with backend APIs (inventory, orders, CRM), (3) Vertex AI Search for product discovery and recommendations, and (4) built-in multi-channel deployment. This is the most integrated, least-custom-code approach on Google Cloud.

- **A is wrong:** Dialogflow CX with hand-crafted intents is the previous generation approach. For a product catalog with potentially millions of items, manually crafting intents and training phrases per category is not scalable. Agent Builder with Gemini can understand product queries generatively without exhaustive intent definitions.
- **C is wrong:** Building custom orchestration, conversation management, and channel routing is significant undifferentiated engineering effort. This approach works but violates the principle of using managed services. It also requires managing model endpoints, scaling, and updates manually.
- **D is wrong:** A third-party chatbot platform adds licensing costs, another vendor dependency, and operational complexity. Google Cloud's native Agent Builder provides all the required capabilities without needing external tooling.

**Exam tip:** For 2024+ PCA exam questions about conversational AI or agents, Vertex AI Agent Builder is almost always the answer. Dialogflow CX is still valid but is positioned for structured/deterministic conversations. If the question mentions "generative," "grounded in enterprise data," or "function calling," it's Agent Builder.
</details>

---

### Q9. Cymbal Retail exposes product catalog, inventory, and pricing APIs to over 200 partner integrations (mobile apps, marketplaces, comparison engines). They need API versioning, rate limiting per partner, usage analytics, a developer portal, and monetization capabilities. Which API management approach should the architect implement?

A) Deploy Kong API Gateway on GKE. Build a custom developer portal using Cloud Run. Implement rate limiting with Redis on Memorystore and track usage in BigQuery.

B) Use Apigee API Management. Define API products with quota policies per partner tier. Enable the integrated developer portal for self-service API key management. Use Apigee Analytics for usage tracking and Apigee Monetization for partner billing.

C) Use Cloud Endpoints with ESP (Extensible Service Proxy) deployed alongside each microservice. Implement API keys for partner identification and Cloud Armor for rate limiting.

D) Place a Cloud Load Balancer with Cloud Armor rate-limiting rules in front of the APIs. Use Identity Platform for partner API key management and export access logs to BigQuery for analytics.

<details>
<summary>Answer</summary>

**Correct: B)**

Apigee is Google Cloud's full-lifecycle API management platform and directly addresses every requirement: API product packaging with versioning, quota/rate-limiting policies configurable per partner tier, a built-in developer portal for self-service onboarding and key management, comprehensive analytics dashboards, and a monetization module for usage-based partner billing. With 200+ partner integrations, Apigee's enterprise-grade API management is the right tool.

- **A is wrong:** Kong on GKE would work technically but requires building and maintaining the developer portal, analytics pipeline, and monetization system separately. This is significant operational overhead compared to Apigee's integrated platform.
- **C is wrong:** Cloud Endpoints is a lightweight API gateway suitable for simpler use cases. It lacks a developer portal, monetization, advanced analytics, and the granular per-partner quota management that Apigee provides. Cloud Armor rate limiting is IP-based, not per-API-key or per-partner.
- **D is wrong:** This cobbles together multiple services that don't form a coherent API management platform. Cloud Load Balancer + Cloud Armor can do basic rate limiting but can't handle API versioning, developer portals, per-partner quotas, or monetization.

**Exam tip:** If an exam question mentions developer portal, API monetization, or per-partner rate limiting, the answer is Apigee. Cloud Endpoints is for simpler API proxying. The more "enterprise API program" the requirements sound, the more Apigee is the answer.
</details>

---

### Q10. Cymbal Retail experiences 10x traffic spikes during flash sales and holiday events (Black Friday, Singles Day). The microservices architecture on GKE must scale from a baseline of 500 pods to 5,000 pods within minutes, while minimizing idle infrastructure costs during normal periods. Which scaling strategy should the architect implement? (Choose TWO.)

A) Use GKE Autopilot mode, which automatically provisions nodes as pods are scheduled, and configure Horizontal Pod Autoscaler (HPA) with custom metrics from Cloud Monitoring tied to request latency and queue depth.

B) Pre-provision a fixed pool of 5,000-pod-capacity nodes year-round to ensure immediate availability during traffic spikes.

C) Use GKE Standard with Cluster Autoscaler and configure node pool overprovisioning using priority-based pod scheduling (pause/placeholder pods). Combine with HPA on CPU and custom metrics.

D) Deploy all microservices to Cloud Run instead of GKE to leverage its automatic scaling from zero to thousands of instances.

E) Use Committed Use Discounts (CUDs) for baseline capacity and configure GKE node auto-provisioning (NAP) with Spot VMs for burst capacity above the baseline. Combine with HPA and VPA for pod-level scaling.

<details>
<summary>Answer</summary>

**Correct: A) and E)**

**A)** GKE Autopilot removes node management entirely -- Google provisions and scales the underlying nodes automatically as pods demand compute. HPA with custom metrics (request latency, queue depth) ensures pods scale based on actual business load rather than just CPU, which is critical for retail workloads where traffic patterns differ from CPU patterns.

**E)** CUDs for baseline capacity (the 500-pod steady state) lock in significant cost savings (~50-57% discount). NAP with Spot VMs for burst capacity handles the 10x spikes cost-effectively -- Spot VMs are up to 91% cheaper and are appropriate for stateless microservice replicas where individual pod preemption is tolerable. Combining HPA (horizontal) and VPA (vertical right-sizing) optimizes resource efficiency at the pod level.

- **B is wrong:** Maintaining 10x capacity year-round wastes enormous infrastructure spend. The whole point is to minimize idle costs during normal periods.
- **C is wrong:** While technically valid, overprovisioning with pause pods is a GKE Standard workaround for node startup latency. Autopilot (option A) handles this more elegantly. This answer is not wrong per se, but it's less optimal than A + E.
- **D is wrong:** Migrating an entire microservices architecture from GKE to Cloud Run is a significant re-architecture effort. Cloud Run has concurrency and timeout limitations that may not suit all microservice patterns (e.g., long-running background processing, stateful services, custom networking).

**Exam tip:** For GKE scaling questions, look for the combination of pod-level scaling (HPA/VPA) + node-level scaling (Autopilot or NAP) + cost optimization (CUDs for baseline, Spot for burst). The exam loves questions that test whether you understand scaling at multiple layers.
</details>

---

## Altostrat Media (Questions 11-15)

> **Case context:** Altostrat Media manages a large media library. Key themes: petabyte migration, AI content analysis, automated transcoding, GKE platform, CDN, storage optimization, content moderation.

### Q11. Altostrat Media needs to migrate 2.5 petabytes of video assets from their on-premises data center to Google Cloud. Their internet connection is 10 Gbps, but it's shared with production traffic and only 2 Gbps can be allocated to migration. The migration must complete within 90 days. Which migration strategy should the architect recommend?

A) Use gsutil rsync with parallel composite uploads over the existing 10 Gbps connection, running transfers during off-peak hours to avoid impacting production.

B) Order multiple Transfer Appliance units (TA-480 or TA-300). Ship the first batch while preparing subsequent loads. Use Storage Transfer Service for incremental synchronization of new/changed files during and after appliance transfers.

C) Provision a 10 Gbps Dedicated Interconnect for the migration and use Storage Transfer Service to transfer data at line rate. Decommission the Interconnect after migration.

D) Use Storage Transfer Service over the existing internet connection at 2 Gbps, throttled to avoid production impact. Enable resumable transfers for reliability.

<details>
<summary>Answer</summary>

**Correct: B)**

At 2 Gbps available bandwidth, transferring 2.5 PB would take approximately 116 days (2.5 PB / 2 Gbps = ~11.6M seconds = ~134 days at wire speed, ~116 days accounting for protocol overhead), exceeding the 90-day window. Transfer Appliance is designed for this scale: each TA-480 holds up to 480 TB, so approximately 6 appliances would cover the full dataset. Shipping, loading, and ingestion of appliances can be parallelized (load one while another ships). Storage Transfer Service handles delta sync for files created/modified during the physical transfer period, ensuring the migration lands up-to-date.

- **A is wrong:** At 2 Gbps effective bandwidth, 2.5 PB cannot be transferred in 90 days even with optimized transfers. gsutil is also not the recommended tool for petabyte-scale migrations -- Storage Transfer Service or Transfer Appliance is preferred.
- **C is wrong:** Even a 10 Gbps Dedicated Interconnect would take ~23 days to transfer 2.5 PB at wire speed (theoretical; actual throughput would be lower). While this could meet the timeline, provisioning a Dedicated Interconnect takes 4-8 weeks for physical cross-connect setup, which could consume most of the 90-day window. The cost of Interconnect for a one-time migration is also not justified.
- **D is wrong:** 2 Gbps cannot transfer 2.5 PB within 90 days (as calculated above). Resumable transfers help with reliability but don't solve the bandwidth limitation.

**Exam tip:** Use this rule of thumb: at 1 Gbps, you can transfer ~10 TB/day. So 2 Gbps moves ~20 TB/day, and 2.5 PB / 20 TB = 125 days -- over the 90-day limit. When network transfer exceeds the deadline, the answer is Transfer Appliance. Always check the math.
</details>

---

### Q12. Altostrat Media wants to build an automated pipeline that analyzes uploaded video content to extract metadata: scene detection, object recognition, celebrity identification, speech-to-text transcription, content moderation (detecting inappropriate content), and generating searchable tags. Which architecture should the architect design?

A) Use Video Intelligence API for scene detection, object tracking, and content moderation. Use Speech-to-Text API for transcription. Build a Cloud Functions pipeline triggered by Cloud Storage uploads that orchestrates these API calls and stores structured metadata in BigQuery. Use Vertex AI for celebrity identification with a custom model.

B) Train a single custom TensorFlow model on Vertex AI that handles all video analysis tasks (scene detection, object recognition, speech-to-text, content moderation). Deploy it on GPU-equipped Compute Engine instances.

C) Use FFmpeg on Compute Engine to extract frames, then send frames to Vision API for image analysis. Use a third-party transcription service for speech-to-text. Store results in Cloud SQL.

D) Upload all videos to YouTube and use the YouTube Content ID and auto-captioning systems. Export the metadata back to Google Cloud via the YouTube Data API.

<details>
<summary>Answer</summary>

**Correct: A)**

This architecture uses purpose-built Google Cloud AI APIs for each analysis task, which is the most efficient and accurate approach. Video Intelligence API natively supports shot/scene detection, object tracking, label detection, and explicit content detection. Speech-to-Text API handles transcription with high accuracy across languages. Cloud Functions provides event-driven orchestration triggered by GCS uploads. BigQuery is ideal for storing and querying structured metadata at scale. For celebrity identification (face recognition of known individuals), a custom Vertex AI model is appropriate since Video Intelligence API's face detection doesn't identify specific people.

- **B is wrong:** Training a single custom model for all these diverse tasks (scene detection, OCR, speech-to-text, content moderation) would be enormously complex, expensive, and would likely underperform compared to Google's pre-trained, specialized APIs. This is reinventing the wheel.
- **C is wrong:** Extracting frames with FFmpeg and using Vision API loses temporal/video context that Video Intelligence API understands (scene transitions, object tracking across frames). A third-party transcription service adds unnecessary vendor dependency when Speech-to-Text API is native. Cloud SQL is not ideal for large-scale metadata querying.
- **D is wrong:** YouTube Content ID is a copyright-detection system, not a general-purpose content analysis pipeline. Using YouTube as a processing intermediary introduces terms-of-service concerns, latency, and a dependency on a consumer platform for enterprise media workflows.

**Exam tip:** For media analysis pipelines on the PCA exam, the answer typically chains pre-built AI APIs (Video Intelligence, Speech-to-Text, Vision) rather than custom models. Custom ML is only the answer when the use case is truly unique (e.g., recognizing specific proprietary objects). Look for Cloud Functions or Workflows as the orchestrator.
</details>

---

### Q13. Altostrat Media stores video content across multiple storage tiers. Their library includes: frequently accessed recent content (last 30 days), occasionally accessed catalog content (30 days - 2 years), and archival master copies that may never be accessed again but must be retained for 10 years. They want to minimize storage costs while maintaining access SLAs. Which storage strategy should the architect implement?

A) Store all content in Standard storage class and use Object Lifecycle Management to transition to Nearline after 30 days, Coldline after 365 days, and Archive after 730 days. Enable Object Versioning for master copies with a 10-year retention policy using Bucket Lock.

B) Store all content in Multi-Regional Standard storage to ensure the fastest access times. Delete content older than 2 years to reduce costs.

C) Use Standard storage for recent content, manually move files to Coldline after 30 days using a scheduled Cloud Function, and use Archive storage with Object Retention Lock for master copies.

D) Store everything in Nearline storage to balance cost and access. Use Autoclass to automatically adjust storage classes based on access patterns.

<details>
<summary>Answer</summary>

**Correct: A)**

This strategy correctly maps each content tier to the appropriate storage class with automated lifecycle transitions. Standard for the active 30-day window, Nearline for 30-day to 1-year content (minimum 30-day storage, fits the "occasionally accessed" pattern), Coldline for 1-2 year content (minimum 90-day storage, lower cost for rare access), and Archive for 2+ year retention (minimum 365-day storage, lowest cost for master copies). Object Lifecycle Management automates these transitions without manual intervention. Bucket Lock with a retention policy ensures master copies cannot be deleted for 10 years, meeting the compliance requirement.

- **B is wrong:** Multi-Regional Standard for all content is the most expensive option. Deleting content older than 2 years violates the 10-year retention requirement for master copies.
- **C is wrong:** Manually moving files with a Cloud Function reimplements Object Lifecycle Management poorly. The 30-day-to-Coldline transition skips Nearline, which would be more cost-effective for content accessed occasionally (Coldline has higher retrieval costs). Also, Object Retention Lock is per-object, which is harder to manage than a bucket-level retention policy.
- **D is wrong:** Nearline has a minimum 30-day storage charge, so using it for frequently accessed recent content (which changes daily) incurs unnecessary early-deletion fees. While Autoclass can optimize storage classes, it works based on observed access patterns, which means it may not transition archival content to Archive class quickly enough if it's never accessed -- and it doesn't enforce retention policies.

**Exam tip:** Storage lifecycle questions follow a predictable pattern: Standard (hot) -> Nearline (30d) -> Coldline (90d) -> Archive (365d). Key details to watch: minimum storage durations, retrieval costs, and whether the question mentions compliance/retention (which needs Bucket Lock). Autoclass is a valid answer when access patterns are unpredictable, but not when the patterns are clearly defined.
</details>

---

### Q14. Altostrat Media runs their video transcoding pipeline on GKE. The pipeline processes uploaded videos into multiple output formats (4K, 1080p, 720p, HLS adaptive streams). Jobs are CPU and memory intensive, with variable load -- 200 jobs/hour during business hours but 2,000+ jobs/hour during content release events. Each job takes 5-30 minutes depending on resolution. Which GKE architecture should the architect design?

A) Use GKE Standard with a dedicated CPU-optimized node pool (c3-highcpu-* machines). Configure Kueue for job queuing and fair scheduling. Set up Horizontal Pod Autoscaler on a custom metric (jobs-in-queue) and Cluster Autoscaler with Spot VM node pools for burst capacity. Use a separate small node pool with on-demand instances for the queue controller and monitoring.

B) Deploy transcoding pods on GKE Autopilot with Burstable pod configurations. Set resource requests to minimum values to pack more pods per node.

C) Run all transcoding on a fixed pool of GPU-enabled nodes (a2-highgpu) for maximum processing speed. Pre-provision enough nodes to handle peak load at all times.

D) Use Cloud Run jobs for all transcoding, triggered by Pub/Sub messages when new videos are uploaded. Configure maximum concurrency to handle peak events.

<details>
<summary>Answer</summary>

**Correct: A)**

This architecture addresses all requirements. CPU-optimized machines (c3-highcpu) match the compute-intensive transcoding workload. Kueue provides Kubernetes-native job queuing with priority scheduling and resource quotas -- essential for managing 2,000+ concurrent jobs during peaks. HPA on a custom queue-depth metric scales pods based on actual work backlog rather than CPU utilization (which may be high even at low job counts). Cluster Autoscaler with Spot VM node pools provides cost-effective burst capacity for the stateless, fault-tolerant transcoding jobs (if a Spot VM is preempted, the job is re-queued). A small on-demand node pool for control plane components ensures reliability.

- **B is wrong:** Autopilot with minimal resource requests would lead to resource contention and job failures. Transcoding requires predictable CPU and memory allocation -- underprovisioning resources causes jobs to be OOMKilled or run extremely slowly. Autopilot also doesn't support Spot VMs for cost optimization.
- **C is wrong:** Video transcoding is CPU-intensive, not GPU-intensive (GPUs are for ML training, rendering, not standard video encoding). A2 GPU instances are extremely expensive and would be wasted on CPU-bound ffmpeg/transcoding workloads. Pre-provisioning for peak load year-round wastes resources during the 200 jobs/hour baseline.
- **D is wrong:** Cloud Run jobs have a maximum timeout of 60 minutes and limited CPU/memory configurations. While they could handle shorter transcoding tasks, 4K transcoding can be extremely resource-intensive. Cloud Run also lacks the fine-grained job scheduling, priority queuing, and resource management that Kueue provides for a high-volume transcoding pipeline.

**Exam tip:** For batch/job-processing workloads on GKE, look for Kueue (job queuing), custom metrics HPA (queue depth, not just CPU), and Spot VMs (for fault-tolerant, stateless jobs). GPU nodes are a distractor unless the workload explicitly requires ML inference or GPU-accelerated rendering.
</details>

---

### Q15. Altostrat Media serves video content to global audiences across 60+ countries. They need to minimize video start time (target: under 2 seconds), reduce buffering for adaptive bitrate streams, protect premium content from unauthorized access, and optimize delivery costs for their highest-traffic regions (North America, Europe, Asia-Pacific). Which content delivery architecture should the architect implement?

A) Deploy Compute Engine instances with Nginx caching proxies in each Google Cloud region closest to the target audiences. Use Cloud DNS geolocation routing to direct users to the nearest proxy.

B) Use Cloud CDN with Media CDN for video-specific delivery. Configure origin shielding to reduce origin load. Use signed URLs with expiring tokens for content protection. Deploy cache-fill regions aligned with traffic patterns (NA, EU, APAC). Enable HTTP/3 (QUIC) for improved streaming performance.

C) Use a third-party CDN (e.g., Akamai, Cloudflare) exclusively, with Cloud Storage as the origin. Manage cache purging and signed URLs through the third-party CDN's API.

D) Serve all content directly from multi-regional Cloud Storage buckets (US, EU, ASIA). Enable Cloud Storage's built-in CDN caching and use signed URLs for access control.

<details>
<summary>Answer</summary>

**Correct: B)**

Media CDN is Google Cloud's purpose-built CDN for large-scale media delivery. It provides: (1) a massive edge network optimized for video streaming with sub-2-second start times, (2) origin shielding that consolidates cache-fill requests to protect the origin (critical for high-traffic content releases), (3) signed URLs and signed cookies for premium content protection with expiring tokens, (4) configurable cache-fill regions to optimize cost and performance for target geographies, and (5) HTTP/3 (QUIC) support that reduces connection setup time and improves streaming over lossy networks. This is the correct Google Cloud-native solution for global media delivery.

- **A is wrong:** Self-managed Nginx caching proxies require significant operational effort (patching, scaling, cache management, TLS termination) and cannot match the edge footprint and optimization of a purpose-built CDN. Google's CDN has thousands of edge points-of-presence; deploying VMs in ~30 regions provides far fewer cache locations.
- **C is wrong:** While third-party CDNs are technically capable, this goes against the case study context of building on Google Cloud. It also adds vendor complexity, separate billing, and lacks the native integration with Cloud Storage origins that Media CDN provides (such as dual-token authentication, cache-fill optimization).
- **D is wrong:** Cloud Storage does not have built-in CDN caching. Multi-regional storage provides redundancy, not edge caching. Serving directly from Cloud Storage would result in high latency for users far from storage regions and high egress costs. There is no CDN layer in this architecture.

**Exam tip:** For media/video delivery on Google Cloud, the answer is Media CDN (not standard Cloud CDN, which is better for web applications). Key differentiators of Media CDN: origin shielding, HTTP/3 support, massive edge capacity for video. If the question mentions "video," "streaming," or "media delivery," look for Media CDN specifically.
</details>

---

## KnightMotives Automotive (Questions 16-20)

> **Case context:** KnightMotives is building connected/autonomous vehicle platform. Key themes: IoT at scale, edge AI, EU data residency (GDPR), mainframe modernization, dealer tools, data monetization.

### Q16. KnightMotives needs to ingest telemetry data from 2 million connected vehicles worldwide. Each vehicle sends sensor data (GPS, engine diagnostics, driving behavior, camera feeds) every 5 seconds, resulting in approximately 400,000 messages per second at peak. The data must be processed in real time for driver safety alerts and stored for batch analytics. Which data ingestion architecture should the architect design?

A) Use Cloud IoT Core to register and authenticate vehicles, then route telemetry to Pub/Sub. Process safety-critical streams with Dataflow (streaming mode) for real-time alerts. Land raw data in Cloud Storage (Parquet format) via Dataflow for batch analytics in BigQuery.

B) Have vehicles send data directly to Pub/Sub using service account key authentication. Use Dataflow for stream processing and write results to Bigtable for time-series queries and BigQuery for analytics.

C) Deploy MQTT brokers on GKE in multiple regions. Vehicles connect via MQTT and data is forwarded to Pub/Sub. Use Dataflow streaming pipelines for real-time alert processing. Store raw telemetry in Bigtable for time-series access and aggregated data in BigQuery for analytics. Authenticate devices using mTLS certificates managed by Certificate Authority Service.

D) Use Apache Kafka on Compute Engine for message ingestion. Process data with Spark Streaming on Dataproc. Store results in HDFS on a Dataproc cluster for batch analytics.

<details>
<summary>Answer</summary>

**Correct: C)**

Cloud IoT Core was retired in August 2023, so option A is no longer valid. The recommended pattern is MQTT brokers (such as HiveMQ or EMQX) on GKE with regional deployments for global coverage. Vehicles connect via standard MQTT protocol with mTLS certificate authentication managed by Certificate Authority Service (CAS) -- this provides device identity at scale. Pub/Sub handles the massive message volume (400K msgs/sec is well within its capacity). Dataflow streaming pipelines process safety-critical events in real time. Bigtable is ideal for high-throughput time-series telemetry storage (fast writes, efficient range scans by vehicle ID + timestamp). BigQuery handles analytical queries on aggregated data.

- **A is wrong:** Cloud IoT Core was deprecated in August 2023 and fully shut down. It is no longer available. Any answer referencing Cloud IoT Core on a current exam is automatically incorrect.
- **B is wrong:** Distributing service account keys to 2 million vehicles is a massive security risk. If a key is compromised, revoking it affects all vehicles using that key. Per-device service accounts at this scale is impractical. mTLS certificate-based authentication (as in option C) is the industry standard for IoT device identity.
- **D is wrong:** Self-managed Kafka and HDFS on Compute Engine/Dataproc introduces significant operational overhead compared to managed Pub/Sub and Cloud Storage/BigQuery. Spark Streaming has higher latency than Dataflow for real-time alerting. HDFS is not a durable, scalable analytics storage tier.

**Exam tip:** Cloud IoT Core was sunset in 2023. If you see it as an answer option on the exam, it is WRONG. The current pattern is MQTT brokers on GKE + Pub/Sub + Dataflow. Also remember: Bigtable for high-throughput time-series writes, BigQuery for analytics.
</details>

---

### Q17. KnightMotives collects vehicle telemetry data from EU-based vehicles and must comply with GDPR requirements. Personal data (driver location, driving behavior, vehicle identification) must remain in the EU, processing must occur in EU regions, and drivers must be able to exercise their right to erasure. KnightMotives also needs SOC 2 compliance for their platform. Which architecture ensures compliance? (Choose TWO.)

A) Configure Assured Workloads for EU Regions to enforce data residency. Create an organization policy constraint (`gcp.resourceLocations`) restricting EU vehicle data projects to `europe-west1`, `europe-west3`, and `europe-west4`. Use CMEK with Cloud KMS keys in EU regions for encryption.

B) Use VPC Service Controls to create a perimeter around EU data projects, preventing data exfiltration to non-EU services or projects. Implement crypto-shredding by encrypting each driver's personal data with a unique CMEK key -- when a driver requests erasure, destroy the key via Cloud KMS, rendering the data unrecoverable.

C) Store EU data in a US multi-region BigQuery dataset with row-level security that restricts EU data access to EU-based employees only.

D) Implement data residency by tagging resources with `location: eu` labels and creating alerting policies that notify when data is created outside EU regions.

E) Use Data Loss Prevention (DLP) API to scan all data for personal information and automatically move it to EU regions if found in non-EU locations.

<details>
<summary>Answer</summary>

**Correct: A) and B)**

**A)** Assured Workloads for EU Regions provides the foundational compliance framework -- it enforces organizational policies that restrict resource deployment to approved EU regions. The `gcp.resourceLocations` constraint is the enforcement mechanism that prevents accidental deployment of resources in non-EU regions. CMEK with EU-based Cloud KMS keys ensures encryption keys never leave the EU.

**B)** VPC Service Controls prevent data from being copied or accessed from outside the defined perimeter, which is critical for GDPR data residency (preventing API-level data exfiltration, not just network-level). Crypto-shredding is the recommended approach for GDPR right-to-erasure at scale: rather than finding and deleting every copy of a driver's data across all storage systems, you encrypt their data with a unique key and destroy the key when erasure is requested. This is cryptographically equivalent to deletion and is explicitly recognized as a valid erasure method under GDPR.

- **C is wrong:** Storing EU data in a US multi-region dataset violates GDPR data residency requirements regardless of access controls. Row-level security controls who can query the data, not where the data physically resides.
- **D is wrong:** Labels are metadata tags and have no enforcement capability. They cannot prevent data from being created in the wrong region. Alerting after the fact is not a compliance control -- it's reactive, not preventive.
- **E is wrong:** DLP API can identify personal information, but automatically moving data between regions after creation is not a reliable compliance strategy. The data would temporarily reside in the wrong region, which itself is a GDPR violation. Compliance must be enforced at creation time, not after the fact.

**Exam tip:** GDPR/data residency questions require preventive controls, not detective ones. The stack is: Assured Workloads (compliance framework) + org policy constraints (enforcement) + VPC Service Controls (exfiltration prevention) + CMEK (encryption control) + crypto-shredding (right to erasure). If an option uses labels, alerting, or after-the-fact remediation, it's wrong.
</details>

---

### Q18. KnightMotives needs to run AI inference models inside vehicles for real-time autonomous driving decisions (object detection, path planning, hazard avoidance). The models must function without network connectivity, update over-the-air (OTA) when connectivity is available, and the edge-to-cloud pipeline must support model versioning and A/B testing. Which architecture should the architect design?

A) Run inference models on the vehicle's onboard computing unit using TensorFlow Lite or ONNX Runtime. Use Vertex AI Model Registry for model versioning. Deploy updated models via an OTA pipeline that stages models in Cloud Storage, validates checksums, and uses a phased rollout controller (canary/A-B) that tracks vehicle cohorts in Spanner. Vehicles report inference telemetry to Pub/Sub for centralized model performance monitoring.

B) Use Distributed Cloud Edge appliances installed in each vehicle for running Kubernetes-based inference workloads. Manage model deployments using GKE fleet management with Anthos Config Management for versioning.

C) Stream all sensor data to the cloud over 5G and run inference in Vertex AI Endpoints with auto-scaling. Cache the last known predictions locally for temporary offline operation.

D) Deploy Coral Edge TPUs in each vehicle connected to a vehicle gateway. Use IoT Core to push model updates and monitor device health remotely.

<details>
<summary>Answer</summary>

**Correct: A)**

Autonomous driving decisions require on-device inference with zero latency tolerance -- the models must run locally on the vehicle's embedded computing hardware. TensorFlow Lite and ONNX Runtime are standard frameworks for edge inference, optimized for the constrained compute available in automotive platforms. Vertex AI Model Registry provides centralized version control for models. The OTA pipeline using Cloud Storage + checksums + phased rollouts is the automotive industry standard pattern for safety-critical software updates. Spanner tracking vehicle cohorts enables global, consistent A/B test management. Telemetry flowing back to Pub/Sub closes the feedback loop for model improvement.

- **B is wrong:** Distributed Cloud Edge is designed for on-premises datacenter or factory floor deployments, not for installation inside individual vehicles. The hardware form factor, power requirements, and cost per unit make this impractical for 2 million vehicles. Also, running full Kubernetes in a vehicle adds unnecessary complexity and resource overhead for inference-only workloads.
- **C is wrong:** Streaming sensor data to the cloud for inference is fundamentally incompatible with autonomous driving. Network latency (even on 5G) and connectivity gaps make cloud-dependent inference unsafe for real-time driving decisions. Caching "last known predictions" is meaningless for dynamic driving scenarios where conditions change every millisecond.
- **D is wrong:** Cloud IoT Core was retired in 2023. Additionally, Coral Edge TPUs are designed for low-power edge inference (cameras, sensors) but may not have sufficient compute capacity for the full autonomous driving stack (object detection + path planning + hazard avoidance simultaneously).

**Exam tip:** For edge AI / autonomous systems, the answer always involves local inference on the device with cloud-managed model lifecycle (versioning, A/B testing, OTA updates, telemetry). If an option suggests streaming data to the cloud for real-time safety-critical decisions, it's wrong. Also, remember that Cloud IoT Core is retired.
</details>

---

### Q19. KnightMotives operates legacy mainframe systems (IBM z/OS) that run core manufacturing execution, supply chain, and dealer parts inventory applications. The systems process 500,000 COBOL transactions daily. The company wants to modernize these workloads to Google Cloud while minimizing risk and business disruption. Which modernization strategy should the architect recommend?

A) Perform a complete rewrite of all COBOL applications in Java microservices using Vertex AI Code generation. Deploy on GKE and use AlloyDB as the replacement for the mainframe databases. Execute a big-bang cutover during a planned downtime window.

B) Use a phased approach: first, use Dual Run to operate mainframe transactions simultaneously on the mainframe and Google Cloud, validating output parity. Convert COBOL to Java using automated refactoring tools (e.g., Google Cloud Mainframe Modernization). Deploy modernized services on GKE with Cloud SQL for PostgreSQL. Migrate workloads incrementally using the strangler fig pattern with Apigee as the API facade.

C) Use a mainframe emulator running on Compute Engine (bare metal) to lift-and-shift the z/OS environment to Google Cloud. Maintain COBOL code and gradually refactor over time.

D) Migrate mainframe data to BigQuery using batch exports. Rewrite all business logic as BigQuery stored procedures and scheduled queries. Use Looker for transaction processing interfaces.

<details>
<summary>Answer</summary>

**Correct: B)**

This phased approach minimizes risk for mission-critical manufacturing systems. **Dual Run** validates that the modernized application produces identical results to the mainframe -- this is critical for 500,000 daily transactions where errors have real-world manufacturing and supply chain impact. Automated COBOL-to-Java refactoring tools reduce the effort compared to a manual rewrite. The **strangler fig pattern** allows incremental migration (one capability at a time) rather than a risky big-bang cutover. Apigee as an API facade routes traffic between mainframe and cloud services during the transition, enabling gradual traffic shifting.

- **A is wrong:** A complete AI-generated rewrite of 500,000-transaction-per-day COBOL applications is extremely high risk. AI code generation can assist but cannot guarantee functional parity for complex COBOL business logic with decades of embedded rules. A big-bang cutover for manufacturing execution systems risks production line shutdowns if anything goes wrong.
- **C is wrong:** Mainframe emulators on Compute Engine maintain the COBOL dependency and do not achieve modernization. This approach also requires expensive bare-metal instances and specialized mainframe emulation licensing. It's a lateral move, not a modernization.
- **D is wrong:** BigQuery is an analytics data warehouse, not a transaction processing system. Rewriting OLTP business logic as BigQuery stored procedures is architecturally inappropriate -- BigQuery is optimized for analytical queries, not high-frequency transactional workloads. Looker is a BI tool, not a transaction processing interface.

**Exam tip:** Mainframe modernization on the PCA exam follows the low-risk pattern: Dual Run (validate parity) + automated refactoring (COBOL to Java) + strangler fig (incremental migration) + API facade (traffic routing). Big-bang rewrites and mainframe emulators are always wrong for mission-critical systems. If you see "minimize risk" in the question, the answer involves phased migration.
</details>

---

### Q20. KnightMotives wants to monetize anonymized vehicle telemetry data by offering analytics products to urban planners, insurance companies, and road infrastructure agencies. The platform must ensure driver privacy (no re-identification possible), provide self-service analytics for external customers, and support usage-based billing. Which architecture should the architect design?

A) Export raw telemetry data to a shared BigQuery dataset. Grant external customers the `bigquery.dataViewer` role. Track query usage with Cloud Monitoring and send manual invoices.

B) Build a data clean room using Analytics Hub. Apply k-anonymity and differential privacy transformations using the DLP API and BigQuery privacy-preserving functions before publishing datasets. Create Analytics Hub listings with data sharing agreements. Use BigQuery capacity-based pricing with per-subscriber billing. Expose pre-built dashboards via Looker for customers who don't need raw data access.

C) Anonymize data by removing the vehicle VIN from telemetry records. Store in Cloud Storage and share via signed URLs with expiring tokens. Build a custom billing system on Cloud Run.

D) Use Pub/Sub to stream raw telemetry to customer-owned Google Cloud projects in real time. Customers process their own data and pay for their own Pub/Sub and Dataflow usage.

<details>
<summary>Answer</summary>

**Correct: B)**

This architecture addresses all three requirements comprehensively. **Privacy:** K-anonymity ensures that no individual vehicle can be singled out from the dataset (each record is indistinguishable from at least k-1 others). Differential privacy adds mathematical noise guarantees that prevent re-identification even with auxiliary data. DLP API can detect and redact any remaining PII. **Self-service analytics:** Analytics Hub provides a managed data marketplace where external customers can discover and subscribe to datasets. Looker dashboards serve customers who want insights without writing SQL. **Billing:** Analytics Hub supports per-subscriber pricing, and BigQuery capacity-based pricing models allow usage-based billing.

- **A is wrong:** Sharing raw telemetry data violates the anonymization requirement. Granting `bigquery.dataViewer` on raw data gives external customers access to potentially identifiable information. Manual invoicing does not scale and doesn't support self-service.
- **C is wrong:** Simply removing the VIN is not sufficient anonymization -- vehicles can be re-identified through GPS patterns (home location, commute routes), driving behavior signatures, or correlation with other datasets. This is a well-documented privacy failure mode. Signed URLs and a custom billing system add unnecessary operational complexity.
- **D is wrong:** Streaming raw telemetry to customer projects exposes un-anonymized personal data (location, driving behavior) to external parties. This violates GDPR and the anonymization requirement. Having customers manage their own infrastructure also creates a poor product experience.

**Exam tip:** Data monetization questions test whether you understand that anonymization means more than removing obvious identifiers. K-anonymity + differential privacy is the gold standard. Simply removing names/IDs is never sufficient (the exam will test this). Analytics Hub is the Google Cloud data marketplace service -- if a question mentions "data sharing" or "data products," Analytics Hub is likely the answer.
</details>
