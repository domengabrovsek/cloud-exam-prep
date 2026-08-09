# PCA content audit - 2026-08-09

Full fact-check of `gcp/pca/` against `cloud.google.com`, run by a panel of GCP experts.
Scope: all 10 doc files, all 260 practice questions, and `gcp/pca/README.md`.

This file is the errata backlog. Items are removed as they are fixed, not ticked off.

Status legend: `[ ]` open, `[x]` fixed, `[?]` needs a decision before fixing.

---

## P0 - CONFIRMED AND FIXED: the case study write-ups were largely fabricated

**Verified by OCR on 2026-08-09, then rewritten.** `docs/08-case-studies.md` went from 1514 to 293 lines because most of the removed content was invented.

**How it was verified.** The claim originally arrived unsupported: the auditor said it had text-extracted the PDFs, which is impossible (they carry 25 image objects, zero text-showing operators, and only zero-width joiners in `/ActualText`), and a WebFetch of the Cymbal PDF then asserted the opposite of the claim while admitting it could not read the file. The claim was therefore held as unverified rather than actioned.

It was settled by rendering each PDF page through PDFKit at 3x and running Apple's Vision framework OCR over the images, via a small Swift program. That produced clean text for all four case studies. The claim was correct.

**What was wrong**, with the official text as the check:

- **Cymbal Retail.** The repo described "a large omnichannel retail company with both physical stores", a "monolithic legacy Java application (single deployable WAR file)", "Oracle Database 19c", Solr/Elasticsearch search, a separate Node.js mobile backend, PCI DSS, and "10x spikes during Black Friday". The source says "Cymbal is an **online retailer**", running "**Kubernetes clusters to run containerized applications**", on "**MySQL, Microsoft SQL Server, Redis, and MongoDB**". No Oracle, no monolith, no physical stores, no PCI DSS, no Black Friday. The repo also missed two stated technical requirements outright: **human-in-the-loop review** of gen AI content before catalog write, and **image generation and enhancement** (color variants, background changes, text overlays). It also missed that the source names **Discovery AI** directly.
- **Altostrat Media.** The repo described petabytes on NAS/SAN, a bare-metal FFmpeg transcoding farm, 50+ manual taggers, and a Transfer Appliance bulk migration. The source says Altostrat is **already running on Google Cloud**: GKE, Cloud Storage, BigQuery, Cloud Run functions, with on-premises remaining only for ingestion and archival. Two stated requirements were missing entirely: **Kubernetes both on-premises and in the cloud**, and **AI systems must be auditable and their decisions explainable**.
- **EHR Healthcare.** Closest to correct, but the repo added Windows Server and .NET workloads, HL7v2/FHIR and the Cloud Healthcare API, an explicit RPO/RTO mandate, and made HIPAA the central constraint. The source never names HIPAA; the requirement is "maintain regulatory compliance". It also omitted Redis and MongoDB from the four named databases, and missed that customers are **multi-national** and include **insurance providers**, and that the legacy integrations **stay on-premises with no plan to move**.
- **KnightMotives Automotive.** The repo invented SAP and S/4HANA on Bare Metal Solution, an IBM Z mainframe, Detroit and Stuttgart data centres, CCPA and ISO 27001, and billions of vehicle events per day. The source names **no specific legacy products** and gives **no telemetry volumes**. It does state constraints the repo missed: **dealers have no budget for new equipment**, an **unreliable online build-to-order system straining dealer relationships**, and explicit **employee upskilling, talent attraction and business-to-technical communication** requirements, which map to objective 4.2.

**The calibration point, also confirmed.** Across all four case studies there is exactly **one hard number**: EHR's "minimum 99.9% availability". No throughput targets, no data volumes, no RPO/RTO values. The exam therefore tests qualitative requirement-to-service matching, not the transfer-time and throughput arithmetic the repo drills.

**What was done.** `docs/08` rewritten from the OCR'd sources, with a hard structural split per case: a **Source** section carrying quoted requirements, and a **Working the case** section explicitly labelled as inference. That split is what prevents the failure recurring.

**Still open:** the 20 questions in `questions/case-study-questions.md` were written against the fabricated version and several test details that are not in the official case studies. See below.

## P0 - verified and fixed

- [x] `docs/08-case-studies.md` - three of the four case study PDF links returned 404 (`:283` Cymbal, `:570` Altostrat, `:897` KnightMotives; only EHR resolved). Confirmed by HTTP check. Replaced all four with the `v6.1_pca_*_case_study_english.pdf` URLs, each verified 200.
- [x] `questions/case-study-questions.md:79` - option D claimed a 99.99% Interconnect SLA from two connections across two metros. The topology is four connections, two per metro, in separate edge availability domains. Confirmed as an internal contradiction against the repo's own correct statement at `docs/09:310`.

## P0 - wrong answer keys (teaches the wrong answer)

All fixed on branch `fix/pca-answer-key-errata` (2026-08-09). Kept here as the record of what changed.

- [x] `questions/section-1-designing-planning.md:310` Q18 - WebSocket LB answer is backwards. Marked D (global external proxy NLB); correct is A. External Application LB supports WebSocket with no configuration. The cookie-affinity rationale at :322 is invented. https://cloud.google.com/load-balancing/docs/https#websocket_proxy_support
- [x] `questions/section-1-designing-planning.md:395` Q23 - marked answer cites "Cloud Load Test", which is not a Google Cloud product. Replace with distributed load testing on GKE (Locust/k6), as `section-6-operations-excellence.md:641` already does correctly.
- [x] `questions/section-1-designing-planning.md:847` Q49 - option E wrongly marked incorrect. The stem specifies Software Assurance, so License Mobility applies and BYOL on standard multi-tenant Compute Engine is valid. The claim at :860 that Microsoft licensing requires sole-tenant is false for this case. https://cloud.google.com/compute/docs/instances/windows/ms-licensing
- [x] `questions/section-2-provisioning-infrastructure.md:45` Q3 - marked answer names "Global external Application Load Balancer (Classic)". Classic is legacy; drop the qualifier.
- [x] `questions/section-2-provisioning-infrastructure.md:284` Q17 - no option meets the stated deadline. 200 TB at 1 Gbps is ~18.5 days and Transfer Appliance round trip also exceeds 2 weeks. Change the stem to a 30-day deadline.
- [x] `questions/section-2-provisioning-infrastructure.md:370` Q22 - marked answer labels C3D "compute-optimized". C3/C3D are general-purpose; compute-optimized is H4D/H3/C2/C2D. Swap to C2D or relabel.
- [x] `questions/section-2-provisioning-infrastructure.md:438` Q26 - distractor A is no longer wrong. Autopilot supports GKE Sandbox, Dataplane V2 network policy and Binary Authorization, so A also satisfies the PCI-DSS requirements. Two defensible answers.
- [x] `questions/section-2-provisioning-infrastructure.md:506` Q30 - internally contradictory. Stem requires 96 vCPU / 360 GB / 4x T4; `a2-highgpu-4g` is 48 vCPU / 340 GB / 4x A100. Make option B an N1 custom machine type with 4x T4 as Spot VMs.
- [x] `questions/section-2-provisioning-infrastructure.md:540` Q32 - unachievable. Stem demands 4 TB RAM and 224 vCPUs; M3 tops out at 128 vCPUs. Explanation at :550 also claims M3 offers 30 TB memory (wrong by ~7x). Rewrite the stem to 3 TB / 128 vCPU, or make option B M4.
- [x] `questions/section-3-security-compliance.md:756` Q40 - answer block is corrupted. Lines 767-773 contain three different verdicts plus visible model reasoning ("Actually, let me clarify"). Also unanswerable as a two-pick because the stem describes both user classes. Split into two questions or restrict the stem to root access.
- [x] `questions/section-5-managing-implementations.md:200` Q8 - the reason B is rejected is false. Storage Transfer Service does support on-premises and private file systems via agent-based transfers. https://cloud.google.com/storage-transfer/docs/overview

## P1 - explanations that state false facts

Marked answer survives; the reasoning does not.

- [ ] `questions/section-1-designing-planning.md:182` Q10 - "no Spot VM option for Dataflow workers". Dataflow FlexRS uses preemptible VMs for most of the batch pool.
- [x] `questions/section-1-designing-planning.md:390` Q22 and `:770` Q44 - both claim Firestore multi-region is eventually consistent. Firestore reads are strongly consistent by default, including multi-region.
- [ ] `questions/section-1-designing-planning.md:719` Q41 - "Vertex AI Endpoints do not support scaling to zero with GPUs". Scale-to-zero via `min_replica_count=0` now supports GPU deployments.
- [ ] `questions/section-2-provisioning-infrastructure.md:177` Q10 - "Standard Tier does not support Cloud Armor". Regional Cloud Armor policies attach to the regional external ALB, which runs on Standard Tier.
- [ ] `questions/section-2-provisioning-infrastructure.md:192` Q11 - justifies Nearline for quarterly access. Google maps monthly to Nearline, quarterly to Coldline. Marked answer stays; reasoning contradicts the class definitions.
- [x] `questions/section-2-provisioning-infrastructure.md:535` Q31 - "Autopilot restricts DaemonSets to system-level add-ons". User DaemonSets are supported.
- [ ] `questions/section-4-optimizing-processes.md:433` Q16 - built on `terraform refresh`, deprecated since 0.15.4 in favour of `apply -refresh-only`.
- [ ] `questions/section-6-operations-excellence.md:87` Q3 - streaming inserts priced at "$0.01/200KB". Actual legacy rate is $0.010 per 200 MB. 1000x error.
- [ ] `questions/section-6-operations-excellence.md:354` Q13 - "escalation policy in Cloud Monitoring". No native escalation or acknowledgement tier exists; escalation comes from a PagerDuty-class notification channel.
- [ ] `questions/section-6-operations-excellence.md:493` - typo "addatively" in the composite SLA explanation.
- [ ] `questions/case-study-questions.md:384` Q14 - "Autopilot doesn't support Spot VMs". Autopilot supports Spot Pods.
- [ ] `questions/section-4-optimizing-processes.md:174` Q7 and `section-6-operations-excellence.md:258` Q10 - conflate Skaffold verify with Cloud Deploy deploy analysis. Right answer, wrong mechanism named. https://cloud.google.com/deploy/docs/analysis

## P1 - non-existent commands in the docs

A learner who types these gets "unrecognized". Highest-confusion class of error.

- [ ] `docs/02:1724`, `02:1731` - `gcloud ai pipelines run create` / `schedules create`. No `pipelines` group under `gcloud ai`. Pipelines are SDK/REST only.
- [ ] `docs/02:1933` - `gcloud ai batch-prediction-jobs create`. No such group.
- [ ] `docs/02:1793`, `02:1799` - `gcloud ai feature-online-stores` / `feature-views` exist only under `gcloud beta ai`.
- [ ] `docs/03:769-777` - `gcloud dlp inspect-content` / `deidentify-content`. Actual surface is `gcloud alpha dlp text inspect` / `redact`.
- [ ] `docs/03:611-614` and `04:385-387` - `--enable-vulnerability-scanning`. Real flag is `--allow-vulnerability-scanning`.
- [ ] `docs/04:77-84` and `06:1350-1363` and `07:145` - `gcloud monitoring slos create/list/describe`. No `slos` group. SLOs come from the Monitoring API v3, Terraform `google_monitoring_slo`, or the console. Significant because 6.5 is the SLO objective.
- [ ] `docs/04:713-717` - `gcloud logging metrics create --bucket-options=...`. No such flag; distribution metrics need `--config-from-file`.
- [ ] `docs/04:889-898` - `gcloud privatecatalog catalogs create` / `products create`. Not real; the surface is search-only. Product also renamed to Service Catalog in 2022.
- [ ] `docs/04:1352-1357` - `gcloud monitoring policies create --condition-display-name/--condition-filter/--condition-threshold-value`. Invented flags; takes `--policy` / `--policy-from-file`.
- [ ] `docs/06:76`, `06:80-85` - `gcloud monitoring metrics-descriptors list/create`. No such group on any track. GA groups are dashboards, policies, snoozes, uptime.
- [ ] `docs/06:234`, `06:237-240`, `07:139-142` - `gcloud monitoring channels`. Beta only.
- [ ] `docs/06:634` - `gcloud beta error-events list`. Correct group is `gcloud beta error-reporting events list`.
- [ ] `docs/07:1293` - `gcloud beta carbon-footprint get`. No such group; use the BigQuery export.
- [ ] `docs/10:184-186` - `gcloud ai pipelines runs create`. Fabricated.
- [ ] `docs/05:2040` - `gcloud services quota update` is alpha-only.
- [ ] `docs/07:831` - org policy `constraints/compute.requireLabels` does not exist. Label governance uses tags or apply-time policy-as-code.
- [ ] `docs/07:361` - `enable-enforce` used on `compute.vmExternalIpAccess`, which is a list constraint needing `set-policy`. The three neighbouring boolean constraints are correct, which makes this harder to spot.

## P1 - numbers that are wrong

- [ ] `docs/07:1026` - Hyperdisk "up to 2.4 TB/s". Off by 1000x. Hyperdisk Balanced maxes at 2,400 MiB/s per volume; Extreme at 5,000 MiB/s; only Hyperdisk ML reaches ~2 TiB/s.
- [ ] `docs/06:68` - metric retention "5 years". Actual 24 months (6 weeks native, then 10-minute).
- [ ] `docs/06:72` - log-based metrics "24 months". Actual 6 weeks.
- [ ] `docs/06:1226` - Enhanced support minimum "$500/month" (actual $100); Premium "contact sales" (published minimum $15,000/month).
- [ ] `docs/06:2034`, `06:2105` - GKE maintenance exclusion `no_upgrades` "cannot exceed 180 days". Actual 90 days, plus a 48-hours-per-92-day-window rule.
- [ ] `docs/05:621`, `05:647`, `05:727` - Cloud Shell "resets after 120 min inactivity" / "20 min idle". Actual: session ends after 40 minutes idle, 12-hour session cap, `$HOME` deleted after 120 days.
- [ ] `docs/09:118` - Spanner "~$650/mo min". Wrong by 10x. Minimum is 100 processing units; 1000 PU = 1 node.
- [ ] `docs/01:576` - Spanner "1 node ~ 10,000 reads/sec or 2,000 writes/sec". Current: 22,500 peak reads/sec, 3,500 peak writes/sec (22,500 with throughput-optimized writes).
- [ ] `docs/02:873` - Bigtable "10K rows/sec reads and writes". Current SSD node: up to 17,000 reads/sec, 14,000 writes/sec.
- [ ] `docs/07:624` - `nam14` replica list wrong. Actual: read-write in us-east4 (default leader) and northamerica-northeast1, witness in us-east1.
- [ ] `docs/07:749`, `07:760` - CUD "20-57%" / "~57% memory-optimized". Actual up to 55% for most series, up to 70% memory-optimized. Flexible CUDs (17-63%) missing entirely.
- [ ] `docs/04:1457-1462` - SUD tier table (~8.3% / ~16.7% / 30%). Published tiers are 0 / 10 / 20 / 30% across quartiles.
- [ ] `docs/04:1453` - "SUDs do not apply to sole-tenant nodes". They do, including the sole-tenancy premium.
- [ ] `docs/01:126-127` - claims C2 is SUD-ineligible. C2 is eligible (20% max), as are M1 and M2 (30%). The flat "up to 30%" is wrong for N2/N2D/C2.
- [ ] `docs/02:997` - Firestore PITR "enabled by default, 1-hour granularity". Both wrong: disabled by default, 1-minute granularity. Contradicts `01:674` which is correct.
- [ ] `docs/02:900` - BigQuery Standard edition shown with 1yr/3yr commitments. Standard has no capacity commitments.
- [ ] `docs/02:1577` - Cloud Run "max timeout 60 min (default)". Default is 5 minutes; 60 is the max.
- [ ] `docs/02:1628` - Cloud Run functions "up to 16 GiB / 4 vCPU". Actual 32 GiB / 8 vCPU.
- [ ] `docs/02:1349` - Serverless VPC Access "up to 1 Gbps". That is the e2-micro ceiling; e2-standard-4 reaches 3,200-16,000 Mbps.
- [ ] `docs/02:272-277`, `02:323` - firewall precedence order wrong for the default `AFTER_CLASSIC_FIREWALL` enforcement.
- [ ] `docs/02:740`, `02:751`, `01:237` - three different Transfer Appliance capacities, all retired. Current models are TA40 (40 TB) and TA300 (300 TB).
- [ ] `docs/02:1089` - M2 listed as "12-416" vCPUs. M2 starts at 208. M1 and M4 missing entirely.
- [ ] `docs/03:252` - SDP "150+ built-in detectors". Actual 200+.
- [ ] `docs/09:311` - Interconnect 99.9% SLA needs two connections in different edge availability domains, not just one metro.
- [ ] `docs/09:305`, `09:309`, `09:367` - Dedicated Interconnect "10-200 Gbps". Now 10, 100 and 400 Gbps link types.
- [ ] `docs/09:322`, `09:329`, `09:369` and `02:22` - HA VPN "3 Gbps per tunnel". Documented limit is 250,000 packets/sec, which is 1-3 Gbps depending on packet size.
- [ ] `docs/07:858-862` - inter-region egress "$0.01/GB". Within North America it is $0.02/GB.
- [ ] `docs/07:711` and `02:596` conflict on storage class SLAs. `02:596` is correct (99.9% multi/dual-region, 99.0% regional for the cold classes). `07:711` flattens all three to 99.0% and is wrong.
- [ ] `docs/01:727` - Dedicated Interconnect "99.9% (1 link)". A single link carries no SLA.
- [ ] `docs/06:1371` vs `06:668`/`06:1388-1391` - two different downtime bases (43.8 vs 43.2 min). Pick the 30-day basis and make it consistent.

## P1 - facts that are wrong but not numeric

- [ ] `docs/04:305` - "the `images` field does NOT push to a registry". It does. Stated as an exam tip, which makes it worse.
- [ ] `docs/04:301` - Cloud Build default SA. New projects default to the Compute Engine default SA, not `{PROJECT_NUMBER}@cloudbuild.gserviceaccount.com`.
- [ ] `docs/04:788`, `04:795`, `04:869` and `05:270` and `07:660` - "Cloud Load Testing" as a Google product, with a doc link that does not resolve. No such product has ever existed.
- [ ] `docs/04:539-543`, `04:667` - "Cloud Deploy rollback means creating a new release". Native rollback exists (`gcloud deploy targets rollback`) plus automated rollback rules.
- [ ] `docs/04:668` - "Cloud Deploy canary requires a service mesh". It supports plain Kubernetes service networking.
- [ ] `docs/04:693` - Cloud Debugger "replaced by Snapshot Debugger". Both are gone; delete the row.
- [ ] `docs/04:1015` - "Memorystore Redis cross-region replication, Standard tier". Standard tier is cross-zone within one region. Cross-region is a Redis Cluster / Valkey feature.
- [ ] `docs/04:1414` - resource-based CUDs cover Dataflow. They do not.
- [ ] `docs/04:1291` - "AlloyDB single-region only". AlloyDB supports cross-region replication.
- [ ] `docs/03:226-227` - "Key Access Justifications only available with EKM". KAJ works on software and HSM keys too.
- [ ] `docs/03:583`, `03:641` - "Binary Authorization is for GKE". Also Cloud Run, Cloud Service Mesh, Google Distributed Cloud. Contradicts `04:397`.
- [ ] `docs/03:620-624` - SLSA "Level 1-4". v1.0 Build track has L1-L3 only.
- [ ] `docs/03:812` - "Artifact Hub" for compliance reports. Not a Google product. Correct: Compliance Reports Manager, and Audit Manager for evidence generation.
- [ ] `docs/03:853` - Access Transparency "Premium or Enterprise support". Actual: Standard, Enhanced or Premium. Enterprise is not a Care tier.
- [ ] `docs/06:325` - "Policy Denied logs can exempt specific users". No principal-exemption mechanism; only a Log Router exclusion filter.
- [ ] `docs/06:1712` - Web Security Scanner "App Engine and Cloud Run". Actual: App Engine, GKE, Compute Engine. Cloud Run not supported. Contradicts `06:1699` two lines earlier.
- [ ] `docs/06:1993` - VM Manager metadata key `enable-os-config`. Correct key is `enable-osconfig`.
- [ ] `docs/05:502` - Datastream destinations include "Cloud SQL (PostgreSQL)". Actual: BigQuery, Cloud Storage, Apache Iceberg.
- [ ] `docs/05:165` - "Apigee Integrated" tier does not exist. Tiers are Standard, Enterprise, Enterprise Plus, plus pay-as-you-go.
- [ ] `docs/05:372-397` - the entire Migrate to Containers section is dead. `migctl` and the processing-cluster flow were removed in May 2024.
- [ ] `docs/05:2271` - decision tree offers "service account with key (or WIF)" for CI/CD. Keys should be a distractor, not a branch. Contradicts `05:2055`.
- [ ] `docs/09:15`, `09:98` - "Autopilot has no DaemonSets" and "need DaemonSets or GPUs, use Standard". Both wrong, stated twice. Autopilot supports both.
- [ ] `docs/09:148`, `09:200` - "Firestore cannot change mode after creation". An empty database can switch, and both modes can coexist in one project.
- [ ] `docs/09:141` - "Firestore under 10 TB" as a branch condition. No such documented limit; invented threshold.
- [ ] `docs/09:157` - "Bigtable has no SQL". GoogleSQL for Bigtable is GA.
- [ ] `docs/09:113` - "Spanner 99.999% SLA" as the entry condition. Five nines is multi-region only; regional is 99.99%.
- [ ] `docs/09:120`, `09:196` - AlloyDB "4x Cloud SQL performance". The claim is 4x standard PostgreSQL.
- [ ] `docs/09:556` - conflates CSEK and EKM. The file's own tip at `09:602` gets it right.
- [ ] `docs/09:540-542` - recommends network tags over service accounts for firewall targeting. Google recommends the opposite; secure tags now close the gap.
- [ ] `docs/09:808` - "Cloud SQL cross-region DR requires manual promotion". Enterprise Plus supports advanced DR with replica failover and zero-data-loss switchover.
- [ ] `docs/09:765` - turbo replication placed in "hot DR, RPO seconds to minutes". Its guarantee is 15 minutes, matching `09:281`/`09:809`.
- [ ] `docs/10:765` - "network policy requires Calico, enabled by default on Standard". Not enabled by default; Dataplane V2 is the recommended plugin and the Autopilot default.
- [ ] `docs/10:136-139` - the global external ALB snippet builds a classic ALB. Needs `--load-balancing-scheme=EXTERNAL_MANAGED` on backend service and forwarding rule. Contradicts `09:410-411`.
- [ ] `docs/10:113-116` - HA VPN example uses `--peer-gcp-gateway` (VPC-to-VPC) while the text describes the on-prem case, which needs `--peer-external-gateway`.
- [ ] `docs/10:57-61` - WIF OIDC provider created with no `--attribute-condition`. For the GitHub issuer this lets any repository mint tokens. Must be repository-scoped.
- [ ] `docs/10:815` - `gsutil signurl` requires a downloaded SA key, contradicting `09:506` ("no service account keys, ever"). Use `gcloud storage sign-url --impersonate-service-account`.
- [ ] `docs/01:1082` - "Migration Center (formerly Migrate for Compute Engine)". False lineage; M4CE became Migrate to VMs, Migration Center is a separate product. File contradicts itself at `01:1093`.
- [ ] `docs/01:795` - "Vertex AI Conversation replaces Dialogflow CX for new projects". Dialogflow CX is the underlying engine, rebranded Conversational Agents. Never replaced.
- [ ] `docs/01:949`, `01:676` - dual-region "sub-second failover" / "near-zero RPO". No such guarantee. Default is most objects within 15 minutes; turbo guarantees 15 min for 100%.
- [ ] `docs/01:463` - "Memorystore Redis Standard (single zone)". Standard places the replica in a different zone.
- [ ] `docs/01:1157` - "Azure Hybrid Benefit equivalent". Google has no such thing; Microsoft Listed Provider terms generally require sole-tenant nodes.
- [ ] `docs/02:1557` - "Autopilot billing is per-pod". Workloads selecting hardware via ComputeClasses are billed node-based.
- [ ] `docs/02:1480`, `02:1493` - "Autopilot DaemonSets limited to an allowlist". The allowlist applies to privileged workloads, not DaemonSets.
- [ ] `docs/02:343` - "25 peering limit per VPC". Quota is increasable on request.
- [ ] `docs/02:25-26` - "Classic VPN deprecated for new deployments". Not marked deprecated; de-emphasized, still carries a 99.9% SLA.

## P2 - stale product naming (systematic sweep, not one-offs)

The Vertex AI rebrand is the big one and is a rewrite, not a find-and-replace.

- [?] **Vertex AI to Gemini Enterprise Agent Platform.** Rebranded at Cloud Next 2026; Vertex AI left the Console 2026-05-21. "Vertex AI Agent Builder" appears as a *correct answer* in `section-1-designing-planning.md:659` and `:1094`, `section-2-provisioning-infrastructure.md:661`, `case-study-questions.md:207`. Model tables reference SKUs that now 404 (`docs/02:2098-2100`, `02:2190`, `01:787`). Affects docs 01, 02, 09.
  **Decision needed:** the v6.1 exam guide (Oct 2025) still uses Vertex AI naming. Recommendation is to keep exam-guide names primary and add a "now marketed as" note, rather than rewriting to names the exam does not use.
- [ ] **Anthos to GKE Enterprise**, **Anthos Service Mesh + Traffic Director to Cloud Service Mesh**: `docs/01:279`, `01:641`, `01:742`, `01:1043`, `02:425`, `02:1505`, `04:429`, `04:668`, `04:749`, `04:790`, `04:820`, `04:972`, `06:788`, `06:1586`, `06:1660`, `06:1674`, `07:102`, `07:469`, `07:1005`, `07:1388`, `05:115`.
- [ ] **Container Registry shut down** (writes 2025-03-18, reads 2025-06-03) but `gcr.io` still used in examples: `docs/02:1363`, `02:1370`, `05:297`, `05:301`, `05:305`, `05:1671`, `06:867`, `06:892`, `06:976`, `06:1000`, `07:178`. Also listed as a VPC-SC protectable service at `03:369`.
- [ ] **Cloud Functions to Cloud Run functions**: `docs/01:1018`, `05:169`, `05:216`, `05:238`, `06:622`, `06:641`, `07:562`, `07:977`, `07:803`, `07:1109`, `07:1231`, `questions/section-5-managing-implementations.md:229`, `:531`.
- [ ] **Dataproc to Managed Service for Apache Spark**: `docs/09:62-66`, `09:93`, `09:100`, `09:891`, `09:1002`.
- [ ] **Migrate for Compute Engine to Migrate to Virtual Machines**: `docs/09:835`, `09:889`, `09:903`, `09:1033`, `05:399`, `05:555`. M4CE v4.11 end of support 2024-04-30.
- [ ] **BeyondCorp Enterprise to Chrome Enterprise Premium**: `questions/section-3-security-compliance.md:749`.
- [ ] **Cloud Operations Suite to Google Cloud Observability**: `docs/01:414`, `07:222`, `07:1476`.
- [ ] **Cloud Source Repositories** closed to new customers 2024-06-17, presented as live: `docs/01:180`, `04:28`, `04:580`. Successor is Secure Source Manager.
- [ ] **Deployment Manager** past end of support 2026-03-31, new users blocked from 2026-06-30: `docs/05:1809-1841`, `09:937-941`, `09:967`, `09:974`, `10:592`, `10:600`, `04:881`.
- [ ] **Memorystore for Memcached deprecated** (no new instances after 2027-02-01, shutdown 2029-01-31): `docs/09:174`, `09:191`. Memorystore for Valkey missing entirely from the in-memory branch.
- [ ] **PaLM 2 and Codey retired** April 2025: `docs/02:2156`, `09:634`.
- [ ] **Model Armor is under Security Command Center**, not Vertex AI: `questions/section-3-security-compliance.md:654`, `docs/03:650-654` (also wrong doc URL).
- [ ] **Cloud Armor Managed Protection Plus to Cloud Armor Enterprise**: `docs/02:214`.
- [ ] **Private Catalog to Service Catalog** (renamed March 2022): `docs/04:879`, `04:936-937`.
- [ ] **Terraform Cloud to HCP Terraform**: `docs/05:1262`.
- [ ] **gsutil to gcloud storage.** gsutil leaves the CLI bundle March 2027. `docs/10:806-825` gives gsutil its own section while `gcloud storage` is absent. Invert the emphasis but keep gsutil, since the exam guide still names it.
- [ ] Dated version pins that will rot: TPU v3 (`questions/section-1-designing-planning.md:472`), GKE 1.28/1.29 (`section-6:587`), pd-ssd with no Hyperdisk (`section-2:492`), provider `~> 5.0` (`docs/10:285-294`, `05:1122-1129`) against a current 7.x, `POSTGRES_15` (`10:505`), CFT module pins (`10:412`, `10:427`).
- [ ] `docs/06:1589` - Istio `networking.istio.io/v1alpha3`. Current is `v1` since Istio 1.22.
- [ ] `docs/02:5` claims "Last updated: February 2026", now stale and predating the rebrand. `docs/01` has no last-updated marker.

## P2 - README and exam-metadata problems

- [x] `README.md:16-25` - section weights. **Two auditors contradicted each other**: one said Google publishes no per-section weights, the other said it extracted them verbatim and that the table is correct. Neither is reproducible: the exam guide PDF renders text as images, so nothing can be quoted from it. A third-party variant reports 25/18/19/15/11/12 instead. Resolved by keeping the numbers, since the ordering is consistent across all sources and the numbers may well be right, and adding a note that they are approximate and unconfirmed. Revisit if a text-layer version of the guide appears.
- [ ] `README.md:11` - omits the officially published figure: case study questions are **20-30% of the exam**. This is the most actionable planning fact available and it is missing.
- [ ] `README.md:3-14` - does not pin the exam guide version. Current is **v6.1, effective 2025-10-30**, which added the Vertex AI sections 2.4/2.5, made WAF and Terraform/IaC explicit, added "Securing AI", and swapped out Helicopter Racing League / Mountkirk Games / TerramEarth for the current case studies. Pin it so drift is detectable.
- [ ] `README.md:3-14` - the **Renewal exam** is absent: 1 hour, 25 questions, $100, 1 case study aligned to generative AI, and case-study questions are 90-100% of that exam.
- [ ] `README.md:65` - "~20% multi-select" is unsourced. The official page says "multiple choice and multiple select" with no ratio.
- [x] Case study names verified correct against the official page: EHR Healthcare, Cymbal Retail, Altostrat Media, KnightMotives Automotive. Exam facts verified: 2 hours, 50-60 questions, $200, 2-year validity, 4 cases with 2 on the exam.

## P2 - question bank structural issues

- [ ] **Multi-select is 13.8% (36 of 260), against a target of ~20%.** Case-study went from 3 to 5 of 20 in the 2026-08-09 rebuild. The remaining gap is concentrated in the two heaviest sections: section 1 (6 of 65) and section 2 (3 of 45). About 16 more conversions there reaches 20%.
- [ ] `questions/section-4-optimizing-processes.md:257` Q10 - the only 6-option item in the bank (A-F for a "choose THREE").
- [x] **The 20 case study questions have been rebuilt on the real cases** (2026-08-09). Every stem now quotes or closely paraphrases a stated requirement, and each question turns on one constraint that eliminates the alternatives. The arithmetic is gone, matching the sources, which contain one number in total. Multi-select went from 3 to 5 of 20. All 20 now carry an exam tip and official doc links, which the file previously lacked entirely.
  Removed with the rewrite: Q1's .NET Framework 4.x (not in EHR), Q7's Oracle PL/SQL at 50,000 TPS (Cymbal has no Oracle), Q11's 2.5 PB Transfer Appliance move (Altostrat's library is already in Cloud Storage), Q19's IBM z/OS COBOL volumes (KnightMotives names no vendor), and the duplicates of `section-4:288`, `section-2:284` and `section-5:11`.
- [ ] Invented case-study facts have also leaked into the section files: Oracle, Black Friday, Transfer Appliance petabytes, HL7/FHIR, Windows Server, SAP and mainframe references appear in `questions/section-1` and `questions/section-2`. Sweep those once the case-study questions are rebuilt.
- [ ] **~15-20% of questions are answerable by elimination alone.** Worst: `section-2:16`, `:169`, `:511`; `section-4:818`, `:928`, `:289`; `section-5:289`, `:400`; `section-6:346`, `:619`; `case-study:323`. The stakeholder-management questions in section 4 (Q25, Q26, Q30, Q34) are near-uniformly one-plausible-of-four.
- [ ] **Recall vs judgment.** Sections 2 (~35-40%), 3 (~40%) and 5 (~35%) test service-fact recall rather than architectural trade-off. Roughly 55 questions across those three files would fit an ACE bank unchanged. Sections 1 (~10%), 4 (~15%) and case studies (~10%) are well calibrated.
- [ ] Ambiguous or under-determined: `section-1:502` Q29, `section-1:344` Q20, `section-3:163` Q9, `section-2:455` Q27.

## P3 - coverage gaps against the v6.1 exam guide

- [ ] **Google Cloud VMware Engine** - named verbatim in objective 2.3, zero mentions anywhere. Belongs in `docs/02` under 2.3.
- [ ] **Infrastructure Manager** - Google's managed Terraform and the named Deployment Manager replacement. Absent from `docs/05` and `docs/09` section 10. Biggest single gap given v6.1 made IaC explicit.
- [ ] **Confidential Computing** - Confidential VMs, Confidential GKE Nodes, Confidential Space. Absent from `docs/03`.
- [ ] **Workforce Identity Federation** - absent; only Workload Identity Federation is covered (`docs/03:529-560`). The workforce/workload distinction is a classic trap.
- [ ] **Privileged Access Manager** - the canonical answer for just-in-time elevation and break-glass. Missing from separation of duties (`docs/03:269-303`).
- [ ] **Cloud NGFW Enterprise** - objective 2.1 names intrusion protection; `docs/02:247-266` covers only Cloud IDS, which is detection-only.
- [ ] **Network Connectivity Center** - two passing rows (`docs/02:188`, `02:344`), absent from the `docs/09` connectivity tree. Objective 2.1 covers exactly this.
- [ ] **Metrics scopes** - multi-project observability, arguably the architect-level Cloud Monitoring topic. Absent from `docs/06`.
- [ ] **SLO composition across dependent services** - the multiplicative rule is a classic PCA calculation, absent.
- [ ] **DORA metrics** - absent from all files despite Google owning the research and citing it throughout WAF.
- [ ] **Cloud Customer Care tiers** - `docs/04:1327-1377` "customer success" is effectively a second SLO section. Exam items framed as customer success often resolve to picking a support tier, and there is no basis for that here.
- [ ] **Dynamic Workload Scheduler** - objective 2.4's "optimizing for different consumption models" is exactly this. Absent.
- [ ] **Gemini Enterprise** (objective 2.5: AI Agents and NotebookLM) - NotebookLM gets one row; AI Agents absent.
- [ ] Missing decision trees in `docs/09`, ranked by expected yield: resource hierarchy / landing zone, network topology selection, data pipeline and ingestion, multi-tenancy isolation, identity and federation, CI/CD topology, AI/ML serving, caching strategy, observability and logging architecture, cost optimization, compliance and data residency, encryption key strategy.
- [ ] Thin relative to weight: securing AI (`docs/03:648-681`, 33 lines for a named objective), envisioning future improvements (`docs/01:1177-1237`, all of 1.5), success measurements (`docs/01:331-354`), Cloud Code (`docs/05:661-682`).
- [ ] `docs/06:13-22` and `07:53-65` - the operational excellence "key principles" are hand-written SRE principles, not Google's published pillar principles. Objective 6.1 reads verbatim "the principles and recommendations of the operational excellence pillar", so a question quoting Google's wording would not be recognisable.

## P2 - effort is allocated inversely to exam weight (verified)

Measured directly on 2026-08-09. Line counts confirmed with `wc -l`.

| Section | Weight | Doc lines | Lines per weight point |
|---------|--------|-----------|------------------------|
| 1 Designing and planning | 25% | 1237 | **49** |
| 3 Security and compliance | 17.5% | 936 | **53** |
| 2 Provisioning | 17.5% | 2242 | 128 |
| 4 Optimizing processes | 15% | 1820 | 121 |
| 6 Operations excellence | 12.5% | 2108 | 169 |
| 5 Managing implementation | 12.5% | 2287 | **183** |

Sections 1 and 3 are 42.5% of the exam and hold 20% of the section-doc content. Sections 5 and 6 are 25% of the exam and hold 41%. Section 5 gets 3.7x more prose per weight point than section 1.

The surplus sits in the most code-dense files, so it is spent on the material that transfers least to a multiple-choice architecture exam: `docs/10` is roughly 71% inside code fences, `docs/05` roughly 61% including 208 lines of Python and 31 of Go. Question distribution is fine (2.0-2.6 per weight point); it is only the prose that is skewed.

- [ ] Move roughly 800 lines of CLI and client-library code out of `docs/05` and `docs/10`, and spend that budget on sections 1 and 3. See the P3 compression list below for the specific blocks.
- [ ] `docs/10:3` claims "PCA tests understanding of advanced CLI usage". That framing is what produced the imbalance, and several of the fabricated commands above exist because the file reached for CLI depth it did not need. Rewrite the opening.

## P2 - question format inconsistency (verified)

Measured on 2026-08-09 with `grep -c`:

| File | Questions | Exam tips | Doc links |
|------|-----------|-----------|-----------|
| section-1-designing-planning | 65 | 0 | 0 |
| section-2-provisioning-infrastructure | 45 | 0 | 0 |
| section-3-security-compliance | 45 | 45 | 0 |
| section-4-optimizing-processes | 35 | 35 | 35 |
| section-5-managing-implementations | 25 | 25 | 24 |
| section-6-operations-excellence | 25 | 25 | 21 |
| case-study-questions | 20 | 20 | 0 |

(Counts are pre-fix. A handful of tips and links were added to sections 1, 2, 3 and 5 on 2026-08-09 as part of the answer key corrections.)

- [ ] **Sections 1 and 2 carry 110 questions, 42% of the bank, with no exam tips and no doc links at all.** This violates rules 7 and 9 of the repo's own `CLAUDE.md`. These are also the two heaviest-weighted sections.
- [ ] Sections 3 and case-study have tips but no doc links.
- [ ] Structural split: sections 1, 2, 3 and case-study put the stem inline on the `### Qn.` line; sections 4, 5 and 6 put it on the following line. Anything parsing `### Q` behaves differently across files. The 4/5/6 shape is better, with per-option "X is wrong" bullets and a `Docs:` line. Standardise on it.

## P3 - depth calibration (ACE-level filler to compress)

PCA does not test flag recall. These blocks should compress to the decision they encode:

- [ ] `docs/10:620-761` - ~140 lines of kubectl RBAC / NetworkPolicy / ResourceQuota YAML and rollout commands. Compress to a table of "isolation need to K8s primitive".
- [ ] `docs/10:154-207` - KMS encrypt/decrypt, Vertex AI and Cloud Deploy command dumps.
- [ ] `docs/02:36-81` - 45 lines of HA VPN and BGP peer commands. Keep the topology decision, drop the commands.
- [ ] `docs/02:396-427` - six-step global external ALB creation sequence. The architect question is which LB and why, already answered at `02:372-394`.
- [ ] `docs/02:1130-1183`, `02:538-570`, `02:1405-1421`, `02:2051-2066` - instance template/MIG, Cloud DNS, patch deployment, and pre-built AI API command blocks.
- [ ] `docs/05:750-913` - 160 lines of gcloud/gsutil/bq/cbt/kubectl recall. `cbt` and the kubectl block have no PCA surface at all.
- [ ] `docs/05:1964-2029`, `05:2088-2157` - hand-rolled pagination loop, manual backoff implementation, 30-line Go GCS function, full Pub/Sub and Secret Manager client code.
- [ ] `docs/06:860-898`, `06:948-1017`, `06:1057-1075` - Cloud Deploy flags, 70 lines of blue/green YAML, kubectl rollout list.
- [ ] `docs/04:115-297`, `04:331-377`, `04:506-550`, `04:1640-1668` - three near-identical cloudbuild.yaml variants, Artifact Registry CRUD, eleven gcloud deploy commands, label/tag CLI.
- [ ] `docs/03:190-248`, `03:337-408` - KMS/Secret Manager and access-context-manager syntax.
- [ ] Too deep for PCA: `docs/02:1941-1948` and `02:2169-2177` (quantization, distillation, pruning, RLHF, soft-prompt tuning - ML Engineer territory), `02:1977-1983` (parallelism strategies), `02:1834-1838` (TPU slice/ICI/DCN internals).

Keep as the model for rewrites: `docs/07:1392-1441` (pillar trade-off matrix, the best-calibrated section in the repo), `07:1342-1357`, `05:205-226`, `05:530-553`, `05:1082-1091`, `06:1077-1084`, `06:2021-2029`, `02:340-346`, `02:1472-1495`, `01:277-297`, `01:659-679`.

## P3 - structural defects

- [ ] `docs/06:1166-1187` - the sample runbook is fenced but its internal markdown headings leak into the document outline, appearing as siblings of section 6.4.
- [ ] `docs/04:1266-1288` - same problem with the ADR example's headings.
- [ ] `docs/05:1809-1858` - IaC comparison table omits Infrastructure Manager.
- [ ] `gcp/ace/docs/05-access-and-security.md:3` and `:1150` - `[Back to README](./README.md)` links to a file that does not exist; the README is one level up.
- [ ] **ACE exam weights disagree across four files.** `gcp/ace/README.md:25-28` says Domain 1 ~23%, Domain 2 ~30%; the doc H1s say ~20% and ~17.5% on the older 5-domain scheme. Root `CLAUDE.md` agrees with the README. Question file headers restate it a fourth time.
- [ ] **Question counts are maintained in five places**: root `CLAUDE.md`, `.claude/domain-reference.md:130-135` and `:36-46`, `.claude/study-modes.md:15-31`, both exam READMEs, and inside each question file. Single source of truth should be the `Total questions: N` line in each question file.
- [ ] **The Cloud Storage class table appears in 9 doc files** and has already diverged (see `07:711` vs `02:596` above). Same shape for the Spot VM 60-91% figure (4 PCA files) and CMEK snippets (9 files).
- [ ] Example timestamps now in the past: `docs/03:59`, `03:68`, `03:200`.
- [ ] `docs/07:1207`, `07:1215-1221` - CFE scores from the 2021-2022 dataset. Either refresh or keep only the relative ordering.
- [ ] `docs/04:1323` - doc link `cloud.google.com/architecture/architecture-decision-records` could not be confirmed to exist.
- [ ] Note: `cloud.google.com/*` doc URLs now 301-redirect to `docs.cloud.google.com/*`. Existing links all resolve, so this is not a defect, but new links should use the new host.

## Open items needing verification

- [ ] `questions/section-3-security-compliance.md:202` Q11 assumes `cloudsql.instances.delete` is denyable via IAM deny policies. Deny policies cover a published subset; confirm Cloud SQL is on the supported list. https://cloud.google.com/iam/docs/deny-permissions-support
- [ ] `questions/section-1-designing-planning.md:519` Q30 cites AlloyDB's 128 TB ceiling. Correct today, but exactly the kind of published limit that moves.
- [ ] `questions/section-4-optimizing-processes.md:592` Q22 option B pairs "on-demand pricing" with "reservation assignment" in one clause, which reads as contradictory.
