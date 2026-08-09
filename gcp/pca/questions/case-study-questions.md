# Case Study Questions

> **Rebuilt from the official case study PDFs on 2026-08-09.** The previous set was written against a fabricated version of `docs/08-case-studies.md` and tested facts that do not exist: Oracle PL/SQL for a company with no Oracle, a petabyte Transfer Appliance move for a library already in Cloud Storage, IBM z/OS COBOL volumes for a case that names no mainframe vendor.
>
> Every stem below quotes or closely paraphrases a **real** requirement. Where a number appears, it is either from the source or clearly labelled as an assumption the question is making.

**Total questions: 20** (5 multi-select, 25%)

> **How the real exam works.** Case study questions are 20-30% of the exam. Two of the four cases appear, viewable in a split screen. The official case studies contain **almost no numbers** (across all four there is one: EHR's 99.9% availability), so the exam tests requirement-to-service matching and elimination against stated constraints, not arithmetic.
>
> These questions are built the same way. For most of them, two options stay plausible until you apply one specific stated requirement. Find that requirement.

See [docs/08-case-studies.md](../docs/08-case-studies.md) for the source material.

---

## EHR Healthcare (Questions 1-5)

> **Case context:** SaaS electronic health record software for multi-national medical offices, hospitals and insurance providers. Moving off colocation because a data center lease is expiring. Customer-facing apps are web-based and many are already containerized on Kubernetes. Data is in MySQL, MS SQL Server, Redis and MongoDB. Legacy file- and API-based integrations with insurance providers are hosted on-premises with **no plan to move them**. Users are managed in Microsoft Active Directory. Monitoring is open-source, and **alerts are sent by email and are often ignored**.

### Q1.

EHR Healthcare states a technical requirement to "maintain legacy interfaces to insurance providers with connectivity to both on-premises systems and cloud providers". The case also states these legacy integrations are scheduled for replacement over several years, with no plan to upgrade or move them at this time. What should the architect propose?

A) Refactor the legacy file- and API-based integrations into Cloud Run services so they can be decommissioned from the data center along with everything else.

B) Establish Dedicated Interconnect to the remaining on-premises environment, and use Cross-Cloud Interconnect or Network Connectivity Center to reach the other cloud providers, leaving the integrations in place.

C) Re-host the integrations on Compute Engine using Migrate to Virtual Machines, then expose them through Apigee.

D) Replicate the integration data into BigQuery nightly so cloud workloads can read it without connecting to the on-premises systems.

<details>
<summary>Answer</summary>

**Correct: B)**

The case is unusually explicit that these systems stay where they are. The requirement is *connectivity to* them, not migration of them, so the whole question is a network design problem: a secure high-performance path to on-premises, plus reachability to other cloud providers.

- **A is wrong** -- it directly contradicts "no plan to upgrade or move these systems at the current time". A refactor is the largest possible violation of that constraint.
- **C is wrong** -- less drastic than A, but still a migration. Re-hosting moves the systems, which the case rules out.
- **D is wrong** -- nightly replication changes an integration into a stale copy. The insurance providers' interfaces are live file and API integrations, and a batch mirror does not maintain them. It also silently introduces a data freshness problem the case never asked for.

**Exam tip:** when a case says a system is staying put, every option that moves, rewrites or replaces it is eliminated regardless of technical merit. Read for "no plan to", "cannot be", "must remain" -- these phrases are put there to do exactly this work.

Docs: https://cloud.google.com/network-connectivity/docs/interconnect and https://cloud.google.com/network-connectivity/docs/network-connectivity-center

</details>

---

### Q2.

EHR Healthcare requires "a minimum 99.9% availability for all customer-facing systems" and wants to decrease infrastructure administration costs. Which deployment approach meets the requirement at the lowest cost and operational burden?

A) Multi-region active-active GKE clusters behind a global external Application Load Balancer, with Spanner multi-region for state.

B) A regional GKE cluster with nodes across three zones, behind a global external Application Load Balancer, with Cloud SQL configured for regional high availability.

C) A zonal GKE cluster with a regional managed instance group as a warm standby in a second region.

D) Two single-zone GKE clusters in the same region with DNS round-robin between them.

<details>
<summary>Answer</summary>

**Correct: B)**

99.9% is three nines, which is roughly 43 minutes of downtime a month. A regional deployment across three zones achieves that: a zone failure is survived, and the regional control plane and regional Cloud SQL HA remove the single points of failure. This is the cheapest configuration that clears the stated bar.

- **A is wrong** -- it works, but it is built for four or five nines. The case asks for 99.9% and separately asks to decrease infrastructure administration costs, so multi-region active-active with Spanner is over-engineering that fails the cost requirement. This is the most commonly chosen wrong answer on this style of question.
- **C is wrong** -- a zonal cluster has a single-zone control plane, so a zone failure takes the cluster out. A cross-region warm standby does not compensate for a fragile primary, and it adds cost.
- **D is wrong** -- DNS round-robin is not failover. Clients cache DNS, and unhealthy endpoints keep receiving traffic. Two single-zone clusters also means two single points of failure rather than one resilient deployment.

**Exam tip:** match the architecture to the stated number, not to the most robust option available. 99.9% is regional multi-zone. 99.99% starts to need multi-region. When a case pairs an availability target with a cost-reduction goal, the answer is the cheapest configuration that clears the target.

Docs: https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters and https://cloud.google.com/sql/docs/mysql/high-availability

</details>

---

### Q3.

EHR Healthcare's executive statement attributes past outages to "misconfigured systems, inadequate capacity to manage spikes in traffic, and inconsistent monitoring practices". The case separately notes that alerts are currently sent via email and are often ignored, and requires "centralized visibility and proactive action on system performance and usage". Which two actions best address this? (Choose TWO.)

A) Define SLOs for customer-facing services and alert on error budget burn rate through a paging channel, rather than alerting on every threshold breach.

B) Increase the frequency of the existing email alerts so issues are noticed sooner.

C) Adopt infrastructure as code with policy enforcement so environments are provisioned consistently rather than configured by hand.

D) Route all logs to a single Cloud Logging bucket in one project and grant every engineer access to query it.

E) Replace the open-source monitoring stack with Cloud Monitoring, keeping the existing alert definitions unchanged.

<details>
<summary>Answer</summary>

**Correct: A) and C)**

Two distinct causes are named in the stem, and each answer addresses one. Ignored alerts are a signal-to-noise problem, and the fix is fewer, more meaningful alerts tied to user-visible impact: SLO burn-rate alerting, delivered somewhere that demands acknowledgement. Misconfigured systems are a provisioning problem, and the fix is removing manual configuration from the path.

- **B is wrong** -- more of a signal that is already being ignored makes the problem worse. Alert fatigue is caused by volume, so increasing volume deepens it.
- **D is wrong** -- centralizing logs is useful and is a separate stated requirement, but access to logs is passive. The case asks for *proactive* action, and a queryable log bucket does not notify anyone.
- **E is wrong** -- this is the trap. Changing the monitoring product while carrying the same alert definitions across reproduces the ignored alerts on a new platform. The tool is not the problem; the alerting philosophy is.

**Exam tip:** "alerts are ignored" is never solved by more alerts or by a different alerting product. It is solved by alerting on symptoms users feel, which means SLOs and burn rates. Watch for options that swap the tool while preserving the behaviour.

Docs: https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring and https://sre.google/workbook/alerting-on-slos/

</details>

---

### Q4.

EHR Healthcare stores data in MySQL, MS SQL Server, Redis and MongoDB, and wants to decrease infrastructure administration costs. Which managed service mapping is most appropriate?

A) Cloud SQL for MySQL and SQL Server; Memorystore for Redis Cluster; Firestore in Enterprise edition for MongoDB.

B) Cloud SQL for MySQL and SQL Server; Memorystore for Redis Cluster; Bigtable for MongoDB.

C) AlloyDB for MySQL and SQL Server; Memorystore for Redis Cluster; Firestore in Enterprise edition for MongoDB.

D) Compute Engine instances running all four database engines, so the existing operational tooling is preserved.

<details>
<summary>Answer</summary>

**Correct: A)**

Each engine maps to the managed service that speaks its protocol. Cloud SQL supports both MySQL and SQL Server. Memorystore for Redis Cluster is the managed Redis. Firestore Enterprise edition provides MongoDB compatibility, which makes it the managed target for a MongoDB workload.

- **B is wrong** -- Bigtable is a wide-column store with no MongoDB compatibility and a completely different data model and API. Migrating MongoDB to Bigtable means rewriting the data access layer, which contradicts the cost and effort goal.
- **C is wrong** -- AlloyDB is PostgreSQL-compatible. It is not a MySQL target and definitely not a SQL Server target, so this option fails on two of the four engines.
- **D is wrong** -- self-managing four database engines on Compute Engine preserves exactly the administration burden the case wants to decrease.

**Exam tip:** the same four databases (MySQL, MS SQL Server, Redis, MongoDB) appear in both EHR Healthcare and Cymbal Retail. Learn the managed mapping once and it covers two cases. The MongoDB answer is the one people miss: Firestore Enterprise edition, not Bigtable and not Cloud SQL.

Docs: https://cloud.google.com/sql/docs and https://cloud.google.com/firestore/docs/enterprise/overview

</details>

---

### Q5.

EHR Healthcare wants to "increase ability to provide insights into healthcare trends" and to "make predictions and generate reports on industry trends based on provider data". Users are managed via Microsoft Active Directory, and the company operates multi-nationally. Which approach best fits?

A) Export provider data to Cloud Storage and give analysts direct object access, running analysis locally in notebooks.

B) Consolidate provider data in BigQuery, use BigQuery ML for trend prediction, and federate Active Directory into Cloud Identity so existing groups drive dataset access.

C) Build a data lake on Bigtable and run Dataproc jobs for trend analysis, managing analyst access with individual IAM bindings.

D) Keep provider data in the operational databases and run reporting queries against read replicas, granting analysts database logins.

<details>
<summary>Answer</summary>

**Correct: B)**

BigQuery is the warehouse for cross-provider analysis, BigQuery ML lets the prediction work happen without moving data or standing up a separate ML platform, and federating Active Directory means access is granted to groups the company already maintains rather than to individuals. That last point matters at multi-national scale.

- **A is wrong** -- raw object access plus local notebooks scatters regulated data onto analyst machines, which sits badly with "maintain regulatory compliance", and it provides no query layer for trend analysis.
- **C is wrong** -- Bigtable is designed for high-throughput key-based access, not ad-hoc analytical queries across providers. Per-user IAM bindings also do not scale and contradict the existing Active Directory group model.
- **D is wrong** -- running analytics against replicas of operational databases couples reporting to the transactional schema and leaves the data siloed per database engine, which is the opposite of the cross-provider view the requirement asks for.

**Exam tip:** when a case mentions an existing directory (Active Directory here), the identity answer is federation into Cloud Identity with group-based IAM, not recreating users or binding individuals. Look for the option that reuses the identity source the company already runs.

Docs: https://cloud.google.com/bigquery/docs/bqml-introduction and https://cloud.google.com/architecture/identity/federating-gcp-with-active-directory-introduction

</details>

---

## Cymbal Retail (Questions 6-10)

> **Case context:** An **online** retailer with a large product catalog. Existing environment is a mix of on-premises and cloud, with Kubernetes clusters running containerized apps, databases in MySQL, MS SQL Server, Redis and MongoDB, legacy **SFTP and ETL batch** integrations, a custom web app doing **keyword queries against relational tables** for product browsing, an **IVR** system, and **call center agents who manually enter orders**. Monitoring is Grafana, Nagios and Elastic. Stated cost goal is to reduce **call center staffing** and **data-center hosting** costs.

### Q6.

Cymbal Retail's technical requirements include a "Human-in-the-Loop (HITL) Review" capability: a user interface for associates to review and manage gen AI-generated content, allowing them to approve, reject, or modify suggestions before updating the product catalog. Which design satisfies this?

A) Generate attributes and descriptions with Gemini on Vertex AI and write them directly to the catalog, with a nightly report of what changed so associates can correct errors afterwards.

B) Generate attributes and descriptions with Gemini on Vertex AI into a staging store, surface them in a review application where associates approve, reject or edit each suggestion, and write only approved content to the catalog.

C) Generate attributes and descriptions with Gemini on Vertex AI and use a confidence threshold to auto-publish high-confidence results, routing only low-confidence results to associates.

D) Have associates write prompts individually for each product so that generation is already human-directed, then publish the output automatically.

<details>
<summary>Answer</summary>

**Correct: B)**

The requirement names three actions the associate must be able to take before the catalog is updated: approve, reject, modify. Only a staging store plus a review UI, with publication gated on approval, provides all three at the right point in the flow.

- **A is wrong** -- review after publication is not review before publication. The requirement explicitly says "before updating the product catalog", and a nightly correction report means wrong content was live in the meantime.
- **C is wrong** -- this is the most attractive distractor because it sounds efficient and is a reasonable production pattern. But it means high-confidence content reaches the catalog with no human approval, which fails the stated requirement. A stated HITL requirement is not satisfied by partial automation.
- **D is wrong** -- directing the generation is not reviewing the output. The associate still never approves, rejects or modifies the result, and prompting per product also defeats the automation goal the case is built on.

**Exam tip:** a stated human-review requirement eliminates every fully or partially automatic publishing path, however well engineered. When you see "approve, reject, or modify" in a requirement, look for the option with an explicit gate, and treat confidence-threshold auto-publishing as a trap.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/learn/overview

</details>

---

### Q7.

Cymbal Retail's current product browsing is "a custom-built web application which allows customers to browse the product catalog by querying the relational databases for names and categories of products". They require automated product discovery that processes customer requests expressed in natural language and returns highly relevant results. What should the architect propose?

A) Add full-text indexes to the relational databases and extend the existing web application to parse natural language into SQL predicates.

B) Adopt Vertex AI Search for commerce (the retail discovery product the case refers to as Discovery AI), and integrate it with conversational agents in the website and mobile app.

C) Move the product catalog into BigQuery and expose a natural language interface over it for customers.

D) Deploy an open-source search engine on GKE and tune keyword relevance ranking with synonym dictionaries.

<details>
<summary>Answer</summary>

**Correct: B)**

The case names the product discovery capability directly, and the requirement is semantic retrieval driven by natural language, not better keyword matching. A retail-specific discovery product also brings relevance tuning, merchandising controls and personalization that a general search index does not.

- **A is wrong** -- full-text indexing improves keyword matching against the same relational structure. The stated problem is that keyword queries over names and categories do not surface relevant products, so making those queries faster does not address relevance.
- **C is wrong** -- BigQuery is an analytical warehouse. Putting a customer-facing, low-latency product lookup in front of it is a poor fit, and it does not provide retail relevance ranking.
- **D is wrong** -- this is a defensible engineering answer and the closest wrong option, but it rebuilds a managed capability, adds operational burden, and synonym dictionaries are still keyword matching. It also works against the stated goal of reducing data-center and operational cost.

**Exam tip:** "natural language" plus "highly relevant results" in a retail context points at the managed retail search product, not at improving an existing keyword index. More generally, when a case describes why the current approach fails, the correct answer usually changes the approach rather than optimising it.

Docs: https://cloud.google.com/solutions/retail-product-discovery

</details>

---

### Q8.

Cymbal Retail wants to reduce call center staffing costs. Today an IVR routes calls to agents, and agents manually enter orders when a customer cannot complete a transaction themselves. Which two actions most directly address the stated cost goal? (Choose TWO.)

A) Deploy conversational agents with natural language understanding that can complete transactions end to end, on both the website and mobile app.

B) Extend the conversational agent to the telephony channel so calls are handled without transferring to an agent for common intents.

C) Add more IVR menu options so calls are routed to the correct department more accurately.

D) Move the call center telephony infrastructure from on-premises to Compute Engine.

E) Train agents to handle calls faster using scripted workflows.

<details>
<summary>Answer</summary>

**Correct: A) and B)**

The cost is agent headcount, so the lever is reducing the number of interactions that need an agent. A handles the root cause on the self-service side: customers currently call because they cannot complete transactions themselves, so an agent that completes transactions removes the call entirely. B handles the calls that still happen, resolving common intents in the telephony channel without a transfer.

- **C is wrong** -- better routing sends the call to the right agent faster. It still consumes an agent, so it does not reduce staffing.
- **D is wrong** -- this reduces data-center hosting cost, which is a real stated goal, but the question asks specifically about call center staffing. Moving telephony to the cloud does not remove a single agent interaction.
- **E is wrong** -- faster handling improves throughput per agent but is a process change, not the AI-driven transformation the case is built around, and the case gives no handling-time problem to solve.

**Exam tip:** when a case states a cost goal, check which cost. Cymbal names two separately: call center staffing and data-center hosting. An option that reduces the other one is a well-designed distractor, not a wrong answer in general.

Docs: https://cloud.google.com/products/conversational-agents

</details>

---

### Q9.

Cymbal Retail requires image generation and enhancement: producing different product image variations from a base image such as various colors, plus background changes, product color adjustments, and the addition of text overlays. Which approach fits?

A) Imagen on Vertex AI for generation and editing, with the outputs routed through the human review workflow before catalog publication.

B) Cloud Vision API to analyse supplier images and extract attributes, with a design team producing variants manually.

C) Gemini on Vertex AI to generate textual descriptions of the desired variants, which suppliers then use to provide new photography.

D) A GKE-hosted image processing pipeline using open-source libraries for colour transforms and text compositing.

<details>
<summary>Answer</summary>

**Correct: A)**

The requirement is generative and editing work on images: variants from a base image, background replacement, colour adjustment, text overlay. Imagen covers generation and editing. Routing through the review workflow is required because the same case mandates human approval before catalog updates.

- **B is wrong** -- Cloud Vision analyses existing images; it does not generate or edit them. Manual variant production is the manual effort the case exists to remove.
- **C is wrong** -- this produces text, not images, and pushes the work back onto suppliers. The requirement is for Cymbal to generate the variations.
- **D is wrong** -- deterministic image processing can do colour transforms and text compositing, but not generative background replacement or plausible new product variants, and it adds infrastructure to run and maintain.

**Exam tip:** Cymbal's gen AI requirements split across two modalities. Text and attributes are Gemini; images are Imagen. People revise the text half and forget images entirely, which makes this an easy question to lose. Also note that the human review gate applies to both.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/image/overview

</details>

---

### Q10.

Cymbal Retail requires that all customer data, including product information and interactions with virtual agents, is handled securely and complies with relevant industry regulations. Which two controls most directly address the risk introduced by the new conversational and generative AI surfaces? (Choose TWO.)

A) Screen prompts and model responses for prompt injection, jailbreak attempts and unsafe content before they reach or leave the model.

B) Inspect and de-identify sensitive customer data in prompts and responses, so personal information is not persisted in logs or sent to the model unnecessarily.

C) Enable uniform bucket-level access on the Cloud Storage buckets holding product images.

D) Apply VPC Service Controls around the analytics project to prevent data exfiltration to external projects.

E) Require multi-factor authentication for the associates using the review interface.

<details>
<summary>Answer</summary>

**Correct: A) and B)**

The question narrows to the risk the AI surfaces introduce, and there are two: the model can be manipulated through its input, and customer data can flow into prompts, responses and logs. Model Armor addresses the first. Sensitive Data Protection addresses the second.

- **C is wrong** -- a sound storage control, and worth having, but it is about bucket ACL consistency and has nothing to do with conversational or generative AI surfaces.
- **D is wrong** -- VPC Service Controls is a strong exfiltration control and a reasonable part of the overall design, but it protects the perimeter around services rather than the content of prompts and responses. The question asks specifically about the new AI surfaces.
- **E is wrong** -- MFA protects the associate accounts, which is good practice, but the review interface is an internal tool. It does not address customer data flowing through the model.

**Exam tip:** for gen AI security questions, separate the two risk classes. Manipulating the model through input is Model Armor. Sensitive data in prompts, responses and logs is Sensitive Data Protection. General cloud security controls in the option list are usually correct statements that do not answer the question asked.

Docs: https://cloud.google.com/security-command-center/docs/model-armor-overview and https://cloud.google.com/sensitive-data-protection/docs

</details>

---

## Altostrat Media (Questions 11-15)

> **Case context:** A media company whose content platform is **already running on Google Cloud**: GKE for the platform, Cloud Storage for the media library, BigQuery as the warehouse, Cloud Run functions for event-driven transcoding and metadata extraction. Legacy on-premises systems remain only for **content ingestion and archival**. Identity is Google Identity plus third-party providers. Monitoring is Cloud Monitoring plus Prometheus, with **alerts primarily delivered via email**. The executive statement says "**reliability and cost management are our top priorities**".

### Q11.

Altostrat Media requires "scalable, performant Kubernetes environments both on-premises and in the cloud" and separately requires modernized CI/CD for containerized deployments "with a centralized management platform". Which approach meets both?

A) Run GKE in Google Cloud and a self-managed Kubernetes distribution on-premises, with separate CI/CD pipelines and separate policy definitions for each.

B) Register both the Google Cloud and on-premises clusters into a GKE Enterprise fleet, apply configuration and policy through Config Sync and Policy Controller, and promote releases with Cloud Deploy.

C) Migrate the on-premises ingestion workloads into GKE in Google Cloud so only one Kubernetes environment needs to be managed.

D) Use Cloud Run for all workloads to remove Kubernetes management entirely, keeping the on-premises systems as they are.

<details>
<summary>Answer</summary>

**Correct: B)**

The phrase "both on-premises and in the cloud" plus "centralized management platform" is the signature of a fleet question. GKE Enterprise registers clusters wherever they run, Config Sync applies configuration from a common source of truth, Policy Controller enforces guardrails consistently, and Cloud Deploy handles promotion across targets.

- **A is wrong** -- it delivers Kubernetes in both places but explicitly rejects the centralized management the requirement asks for. Divergent policy across environments is the problem, not the solution.
- **C is wrong** -- it satisfies the CI/CD half by eliminating the on-premises half, which contradicts a stated requirement for Kubernetes to run on-premises as well.
- **D is wrong** -- Cloud Run is a reasonable platform, but the requirement names Kubernetes environments in both locations. Removing Kubernetes does not satisfy a requirement for Kubernetes.

**Exam tip:** "consistent across on-premises and cloud", "centralized management", "fleet" and "single source of truth for configuration" all point at GKE Enterprise with Config Sync. Options that solve the problem by removing one of the two environments are eliminating a requirement, not meeting it.

Docs: https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/overview and https://cloud.google.com/kubernetes-engine/enterprise/config-sync/docs/overview

</details>

---

### Q12.

Altostrat Media requires AI-powered detection of harmful content, and separately requires that "AI systems are auditable and their decisions can be explained". Which two elements should the design include? (Choose TWO.)

A) Feature attributions on model predictions so a reviewer can see which inputs drove a classification decision.

B) Logging of model inputs, outputs and model version for each decision, retained so that a past decision can be reconstructed.

C) A higher confidence threshold on the classifier so that only unambiguous decisions are acted upon.

D) A larger foundation model, on the basis that stronger models make fewer classification errors.

E) Human moderation of every piece of content before publication.

<details>
<summary>Answer</summary>

**Correct: A) and B)**

Explainability and auditability are two different obligations. Explainability means being able to say why a decision was made, which is what feature attributions provide. Auditability means being able to reconstruct a decision after the fact, which requires the inputs, the output and the model version to have been recorded.

- **C is wrong** -- a confidence threshold changes which decisions are acted on. It says nothing about why a decision was made and leaves it just as opaque.
- **D is wrong** -- accuracy and explainability are independent. Larger models are typically harder to explain, not easier, so this arguably moves away from the requirement.
- **E is wrong** -- full human moderation would sidestep the need to explain automated decisions, but the case asks for AI-powered detection at media-library scale, and reviewing everything by hand contradicts that. Note the contrast with Cymbal, where human review *is* an explicit requirement.

**Exam tip:** Altostrat is the only case that demands explainability, which makes Vertex Explainable AI a case-specific answer worth remembering. Distinguish it from accuracy: an option offering a better or more confident model is answering a different question.

Docs: https://cloud.google.com/vertex-ai/docs/explainable-ai/overview

</details>

---

### Q13.

Altostrat Media requires optimizing cloud storage costs for a growing media library, while maintaining high availability and scalability. Access patterns across the library vary and are not known in advance for new content. What should the architect recommend?

A) Enable Autoclass on the media buckets so objects move between storage classes automatically based on observed access patterns.

B) Apply a lifecycle rule moving all objects to Nearline after 30 days, Coldline after 90 days and Archive after 365 days.

C) Move the entire media library to Archive storage, since most content is accessed rarely after publication.

D) Keep everything in Standard storage and reduce cost by enabling object versioning with a short retention window.

<details>
<summary>Answer</summary>

**Correct: A)**

Autoclass exists for exactly this situation: variable and unpredictable access patterns, where you do not want to hand-tune policy per content type. It transitions objects on observed access, moves them back to Standard when they are read again, and has no retrieval charges for the transitions it manages.

- **B is wrong** -- a fixed age-based ladder is reasonable when access patterns are predictable, but here they are not. Content that becomes popular after 90 days would sit in Coldline incurring retrieval charges on every read, which can cost more than the storage saved.
- **C is wrong** -- Archive has a 365-day minimum storage duration and the highest retrieval cost. A media library that serves user requests would pay heavily on reads, and early deletion of any object incurs the remaining minimum duration charge.
- **D is wrong** -- versioning increases storage consumed by keeping noncurrent versions. It is a data protection feature, not a cost reduction one.

**Exam tip:** unpredictable or unknown access patterns is the Autoclass signal. Known, predictable ageing is the lifecycle-rule signal. And remember Archive is not slow to read, its problem is retrieval cost and the 365-day minimum duration, which is a distinct trap from latency.

Docs: https://cloud.google.com/storage/docs/autoclass

</details>

---

### Q14.

Altostrat Media wants to automatically generate concise summaries of media content and extract rich metadata from media assets using NLP and computer vision. Their platform already uses Cloud Run functions for event-driven tasks such as metadata extraction. What is the most appropriate way to add these capabilities?

A) Extend the existing event-driven Cloud Run functions to call Gemini on Vertex AI for summarization, and Speech-to-Text and Video Intelligence for transcription and visual metadata, triggered when new content lands in Cloud Storage.

B) Build a scheduled batch job on Dataproc that reprocesses the entire media library nightly to regenerate summaries and metadata.

C) Train custom summarization and object detection models from scratch on the media library using Vertex AI custom training.

D) Move media processing to GKE with GPU node pools and run open-source models for summarization and vision tasks.

<details>
<summary>Answer</summary>

**Correct: A)**

The case already establishes the pattern: Cloud Run functions handling event-driven tasks including metadata extraction, over a library in Cloud Storage. Extending that path with managed AI services fits the existing architecture, scales per object, and processes content once as it arrives.

- **B is wrong** -- reprocessing the whole library nightly is wasteful and directly opposes "cost management is a top priority". It also delays metadata for new content by up to a day.
- **C is wrong** -- custom training is expensive and slow when managed models already cover summarization, transcription and visual metadata. Nothing in the case suggests a domain-specific need that pre-trained models cannot meet.
- **D is wrong** -- self-hosting models on GPU nodes adds infrastructure and cost to a company that has named cost management a top priority and already prefers serverless for this exact workload.

**Exam tip:** when a case describes its existing architecture, the best answer usually extends it rather than introducing a parallel one. Altostrat's event-driven Cloud Run functions over Cloud Storage is stated in the source, so an option that matches that shape is likely correct.

Docs: https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/overview and https://cloud.google.com/video-intelligence/docs

</details>

---

### Q15.

Altostrat Media's executive statement names reliability and cost management as their top priorities, and the case notes alerts are primarily delivered by email. The architect must improve reliability of operational workflows across both Google Cloud and on-premises environments. Which approach best reflects the stated priorities?

A) Define SLOs for the content delivery and ingestion workflows, alert on burn rate, and use the error budget to decide when to prioritise reliability work over feature delivery.

B) Add redundancy at every layer of the platform, including multi-region deployment of all services, to minimise the chance of any outage.

C) Increase monitoring coverage by collecting every available metric from both environments into Cloud Monitoring and alerting on anomalies.

D) Introduce a change freeze process so that deployments happen only during scheduled maintenance windows.

<details>
<summary>Answer</summary>

**Correct: A)**

SLOs with error budgets are the mechanism that reconciles the two stated priorities. They define reliability in terms users experience, and the error budget makes the reliability-versus-velocity trade-off explicit rather than implicit. Burn-rate alerting also replaces the ignored email alerts with a small number of meaningful signals.

- **B is wrong** -- redundancy everywhere maximises reliability at the direct expense of cost, which is the other named priority. When a case names two priorities, an answer that sacrifices one for the other has not understood the question.
- **C is wrong** -- collecting every metric and alerting on anomalies increases cost (ingestion and retention) and increases alert volume, which is the existing problem. More signal is not better signal.
- **D is wrong** -- change freezes reduce deployment risk by reducing deployment frequency, which conflicts with the requirement to accelerate operational workflows and with modern delivery practice. Lower change failure rate comes from smaller, safer, more frequent changes.

**Exam tip:** when a case states two priorities that pull against each other, the answer is usually the mechanism that makes the trade-off explicit, which for reliability versus cost or velocity means SLOs and error budgets. Options that maximise one dimension are wrong by construction.

Docs: https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring and https://sre.google/workbook/implementing-slos/

</details>

---

## KnightMotives Automotive (Questions 16-20)

> **Case context:** A manufacturer of autonomous, BEV, hybrid and ICE vehicles. IT is largely on-premises with some applications on major cloud platforms. The supply chain runs on an **outdated mainframe** and the **ERP is outdated**. **Dealers have no budget for new equipment.** There are **multiple vehicle code bases and significant technical debt**. **Network connectivity to manufacturing plants and vehicle connectivity in rural areas** are challenges. **Security is paramount due to past data breaches**, and **EU data protection** applies to autonomous platforms. The **online build-to-order system is unreliable**, straining dealer relationships. The case gives **no data volumes and names no specific legacy vendors**.

### Q16.

KnightMotives needs modern dealer tools for sales, service and inventory management, and an improved online build-to-order system. The case states that dealers have no budget for new equipment. Which approach fits the constraint?

A) Deliver dealer tools as browser-based web applications that run on the hardware dealers already have, hosted on Google Cloud.

B) Provide dealers with pre-configured tablets running a native application, funded through a dealer equipment programme.

C) Deploy Google Distributed Cloud appliances at each dealership so tools run locally with low latency.

D) Require dealers to install an on-premises server to host the inventory management system for their location.

<details>
<summary>Answer</summary>

**Correct: A)**

The dealer budget constraint is stated plainly and eliminates anything requiring hardware at the dealership. Browser-based tools delivered from the cloud run on whatever the dealer already has, and centralise the operational burden with KnightMotives rather than distributing it across a dealer network.

- **B is wrong** -- funding a hardware programme works around the constraint by having KnightMotives pay instead, but the case gives no indication of appetite for that, and it introduces a fleet of managed devices across an independent dealer network. The constraint is there to be respected, not bought out.
- **C is wrong** -- Distributed Cloud appliances are hardware at each site, which is precisely what dealers cannot fund. It also introduces significant operational complexity for what are essentially CRUD applications.
- **D is wrong** -- an on-premises server per dealership is the most expensive and least maintainable option, and squarely violates the stated constraint.

**Exam tip:** business and financial constraints eliminate options just as firmly as technical ones, and are easier to miss because they sit in the narrative rather than the requirements list. "No budget for new equipment", "cannot be refactored", "lease is expiring" all do real work in the answer.

Docs: https://cloud.google.com/run/docs and https://cloud.google.com/architecture/framework/system-design

</details>

---

### Q17.

KnightMotives states that security is paramount due to past data breaches, and requires a comprehensive security framework, an incident response plan, and security awareness training for employees. Which three elements belong in the architect's proposal? (Choose THREE.)

A) Centralised detection and posture management across the environment, with findings triaged into a defined response process.

B) A documented and rehearsed incident response plan, including defined roles, escalation paths and post-incident review.

C) A recurring security awareness training programme for employees, tracked to completion.

D) Encrypting all data at rest with customer-managed keys, which removes the need for the other controls.

E) Restricting all cloud access to a single administrator account to reduce the attack surface.

<details>
<summary>Answer</summary>

**Correct: A), B) and C)**

The case asks for three things and names them explicitly: a security framework, an incident response plan, and security awareness training. The proposal has to cover all three, and the third is a people control rather than a product one, which is what makes this question representative of the case.

- **D is wrong** -- CMEK is a valuable control and appropriate here given EU data protection requirements, but the claim that it removes the need for the others is false. Encryption at rest does not detect intrusions, respond to incidents, or stop an employee falling for a phishing email, and past breaches suggest exactly those paths.
- **E is wrong** -- a single shared administrator account is an anti-pattern. It destroys attribution, prevents least privilege, and creates a catastrophic single point of compromise. It increases risk while appearing to reduce surface.

**Exam tip:** KnightMotives is the case that makes people and process explicit requirements, including security awareness training, employee upskilling and business-to-technical communication. Those are exam objective 4.2 territory, and questions built on them will have a correct answer that is not a product. Do not discard an option because it is not a service.

Docs: https://cloud.google.com/security-command-center/docs/security-command-center-overview and https://cloud.google.com/architecture/framework/security

</details>

---

### Q18.

KnightMotives wants to monetize corporate data to finance new technology investments, but states that corporate data remains siloed. They must also adhere to European Union data protection regulations. Which approach fits both requirements?

A) Consolidate data into BigQuery, publish curated datasets through Analytics Hub for controlled sharing with partners, and enforce data residency with organization policy location constraints plus CMEK.

B) Export datasets to Cloud Storage and share signed URLs with paying partners, with an agreement governing their use.

C) Give partners read access to the operational databases through a VPN, so data is always current.

D) Publish datasets to a public Cloud Storage bucket with requester-pays enabled, so consumers cover the egress cost.

<details>
<summary>Answer</summary>

**Correct: A)**

Two requirements have to be satisfied at once. Analytics Hub is built for sharing curated data without copying it, with the publisher retaining control and visibility over subscribers, which is what monetization needs. Location constraints and customer-managed keys address the EU data protection obligation on where data lives and who can decrypt it.

- **B is wrong** -- signed URLs hand over copies of files. Once distributed there is no revocation, no usage visibility, and no control over onward sharing, which is unworkable for monetized data and weak under a data protection regime.
- **C is wrong** -- exposing production databases to external parties couples partners to the operational schema, creates a serious security exposure for a company that has already suffered breaches, and provides no commercial controls.
- **D is wrong** -- a public bucket means anyone can access the data. Requester-pays shifts egress cost but does not restrict access, so this monetizes nothing and creates an EU compliance problem.

**Exam tip:** data sharing and monetization questions point at Analytics Hub, because sharing in place with subscriber controls is exactly its purpose. Options that copy or export data lose control, which is nearly always the discriminator when a compliance requirement is also present.

Docs: https://cloud.google.com/bigquery/docs/analytics-hub-introduction and https://cloud.google.com/assured-workloads/docs/overview

</details>

---

### Q19.

KnightMotives runs its supply chain on an outdated mainframe and needs to gradually modernize or replace legacy systems while adopting a hybrid cloud strategy. The case does not state a hard cutover deadline. Which migration approach is most appropriate?

A) Rewrite the supply chain application as cloud-native microservices in a single coordinated cutover once the rewrite is complete.

B) Replatform incrementally, running the mainframe and the migrated components in parallel to validate equivalence before shifting traffic, and retire mainframe functions progressively.

C) Keep the mainframe indefinitely and expose its functions through an API layer, treating modernization as complete.

D) Re-host the mainframe workload unchanged onto Compute Engine instances to exit the data center quickly.

<details>
<summary>Answer</summary>

**Correct: B)**

The case asks to "gradually modernize or replace legacy systems", and gradual is the operative word. Running old and new in parallel lets each migrated function be validated against the system of record before it carries traffic, which is the only safe way to move a supply chain that the business depends on daily.

- **A is wrong** -- a full rewrite with a single cutover is the highest-risk option available for a critical system, and it contradicts "gradually". Big-bang cutovers of supply chain systems are where migrations fail.
- **C is wrong** -- an API façade is a genuinely useful step and often part of a strangler approach, but declaring modernization complete leaves the outdated mainframe in place. The case wants the legacy system modernized or replaced, not wrapped.
- **D is wrong** -- mainframe workloads do not re-host onto x86 Compute Engine instances unchanged; the architectures are not compatible. Even where emulation is possible, it carries the technical debt forward without addressing the stated problem.

**Exam tip:** the case names no mainframe vendor, no language and no transaction volumes, so any option that depends on those specifics is inventing them. When a case says "gradually", incremental parallel-run approaches beat both big-bang rewrites and do-nothing façades.

Docs: https://cloud.google.com/mainframe-modernization/docs/overview and https://cloud.google.com/architecture/migration-to-gcp-getting-started

</details>

---

### Q20.

KnightMotives identifies vehicle connectivity in rural areas as a challenge, and requires reliable network connectivity to support real-time AI features and data transmission. Which design best accommodates the constraint?

A) Buffer telemetry and AI inference results on the vehicle, run safety-critical inference locally, and synchronise opportunistically when connectivity is available.

B) Require a persistent connection for all AI features, and disable them when the vehicle is out of coverage.

C) Stream all raw sensor data continuously to the cloud for processing, and rely on cellular failover to maintain the connection.

D) Provision satellite connectivity for the entire vehicle fleet so coverage gaps are eliminated.

<details>
<summary>Answer</summary>

**Correct: A)**

Intermittent connectivity is a property of the environment, not a defect to be engineered away. The design that survives it keeps safety-critical inference local so the vehicle does not depend on a network round trip, buffers what needs to be sent, and syncs when a link is available.

- **B is wrong** -- disabling features in rural areas fails the customers who live there, and the case already says hybrid and ICE drivers view the in-vehicle experience poorly. Degrading it further in exactly the places coverage is worst works against the business goal.
- **C is wrong** -- streaming all raw sensor data continuously is the most bandwidth-hungry option possible, which is the opposite of what a coverage-constrained environment can support. Cellular failover does not help where there is no cellular coverage.
- **D is wrong** -- fitting satellite connectivity to an entire consumer vehicle fleet is disproportionate in cost and hardware, and the case gives no indication of that appetite. It also does not remove the need for local inference, since latency still rules out a round trip for safety-critical decisions.

**Exam tip:** for edge and connectivity questions, the pattern is local processing for anything latency-critical or safety-critical, buffering plus opportunistic sync for everything else. Options that assume reliable connectivity have not accepted the stated constraint, and options that eliminate the constraint with hardware are usually disproportionate.

Docs: https://cloud.google.com/distributed-cloud/edge/latest/docs and https://cloud.google.com/architecture/connected-devices

</details>

---

## Working case study questions

The pattern across all twenty:

1. **Find the stated requirement in the stem.** It is usually quoted or closely paraphrased from the case.
2. **Ask what it rules out.** Most of these are decided by elimination, not by picking the strongest service.
3. **Check the option against the existing environment.** Rebuilding something the case says already exists is nearly always wrong. Altostrat is already on Google Cloud; Cymbal already runs Kubernetes; EHR has already containerized much of its estate.
4. **Watch for constraints in the narrative**, not just the requirements list: dealers have no budget, the integrations are not moving, the lease is expiring.
5. **Use the stated priority as a tie-breaker.** Altostrat says reliability and cost. Cymbal names call center and hosting costs specifically. EHR wants lower administration cost.
6. **Prefer the cheapest option that meets the requirement**, not the most capable one.
