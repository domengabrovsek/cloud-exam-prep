# PCA Case Studies Reference

> **Rebuilt from the official PDFs on 2026-08-09.** The previous version of this file was largely invented: it gave Cymbal Retail an Oracle database and physical stores, gave Altostrat Media a bare-metal transcoding farm, and made HIPAA the centre of EHR Healthcare. None of that is in the source documents. If you studied the old version, re-read the Source sections below carefully, because several things you may have memorised are wrong.

Case study questions are **20-30% of the exam** (officially published). Four case studies exist; two appear on your exam, viewable in a split screen while you answer.

## How this file is organised

Each case study has two parts, and the split is the point:

- **Source** -- taken from the official PDF. Requirements are quoted or closely paraphrased. This is what the exam is written against.
- **Working the case** -- a suggested service mapping and the question patterns that follow from it. **This is inference, not source material.** It is a reasonable reading, not the answer key.

Never let the second part harden into something you believe was in the case study. That confusion is exactly what made the previous version of this file wrong.

## The single most important calibration point

**The official case studies contain almost no numbers.** Across all four, there is one hard figure: EHR's "minimum 99.9% availability". There are no throughput targets, no data volumes, no RPO/RTO values, no user counts.

This means the exam tests **qualitative requirement-to-service matching**, not arithmetic. When a case study question appears, you are being asked "which service satisfies this stated requirement, and which stated constraint rules out the alternatives" -- not "how long does 200 TB take at 1 Gbps". Practise reading requirements, not doing sums.

## Official sources

| Case study | PDF |
|---|---|
| EHR Healthcare | [v6.1_pca_ehr_healthcare_case_study_english.pdf](https://services.google.com/fh/files/misc/v6.1_pca_ehr_healthcare_case_study_english.pdf) |
| Cymbal Retail | [v6.1_pca_cymbal_retail_case_study_english.pdf](https://services.google.com/fh/files/misc/v6.1_pca_cymbal_retail_case_study_english.pdf) |
| Altostrat Media | [v6.1_pca_altostrat_media_case_study_english.pdf](https://services.google.com/fh/files/misc/v6.1_pca_altostrat_media_case_study_english.pdf) |
| KnightMotives Automotive | [v6.1_pca_knightmotives_automotive_case_study_english.pdf](https://services.google.com/fh/files/misc/v6.1_pca_knightmotives_automotive_case_study_english.pdf) |

Read the PDFs themselves at least once. They are three pages each.

---

## Case Study 1: EHR Healthcare

### Source

**Company.** A leading provider of **electronic health record software**, sold as SaaS to **multi-national** medical offices, hospitals, and **insurance providers**.

**Why they are moving.** Business is growing exponentially. They need to scale, adapt their DR plan, and roll out continuous deployment. **Google Cloud has been chosen to replace their current colocation facilities**, and **the lease on one of the data centers is about to expire.**

**Existing environment.**
- Software hosted in multiple colocation facilities
- Customer-facing applications are web-based; **many have recently been containerized to run on a group of Kubernetes clusters**
- Data in a mixture of relational and NoSQL: **MySQL, MS SQL Server, Redis, and MongoDB**
- Several **legacy file- and API-based integrations with insurance providers hosted on-premises**. Scheduled for replacement over several years, with **no plan to upgrade or move them at this time**
- Users managed via **Microsoft Active Directory**
- Monitoring via various open source tools. **Alerts are sent via email and are often ignored**

**Business requirements.**
- On-board new insurance providers as quickly as possible
- Provide a **minimum 99.9% availability** for all customer-facing systems
- Provide centralized visibility and proactive action on system performance and usage
- Increase ability to provide insights into healthcare trends
- Reduce latency to all customers
- **Maintain regulatory compliance**
- Decrease infrastructure administration costs
- Make predictions and generate reports on industry trends based on provider data

**Technical requirements.**
- Maintain legacy interfaces to insurance providers with connectivity to **both on-premises systems and cloud providers**
- Provide a consistent way to manage customer-facing applications that are container-based
- Provide a secure and high-performance connection between on-premises systems and Google Cloud
- Provide consistent logging, log retention, monitoring, and alerting capabilities
- Maintain and manage multiple container-based environments
- **Dynamically scale and provision new environments**
- Create interfaces to ingest and process data from new providers

**Executive statement.** Outages have come from **misconfigured systems, inadequate capacity to handle traffic spikes, and inconsistent monitoring practices**. They want a platform that spans multiple environments seamlessly.

### Working the case

*Inference below, not source material.*

| Stated requirement | Reasonable service answer |
|---|---|
| Consistent management of container-based apps, multiple environments | GKE Enterprise fleets, Config Sync for consistent policy |
| Dynamically scale and provision new environments | Infrastructure as code, GKE cluster autoscaler and node auto-provisioning |
| Secure high-performance on-prem connection | Dedicated Interconnect; HA VPN if lead time or budget rules it out |
| Connectivity to on-prem **and other cloud providers** | Cross-Cloud Interconnect or Network Connectivity Center |
| MySQL and MS SQL Server | Cloud SQL |
| Redis | Memorystore for Redis Cluster or Valkey |
| MongoDB | Firestore Enterprise edition (MongoDB compatible), or MongoDB Atlas |
| Users in Active Directory | Cloud Identity with GCDS or federation; Managed Microsoft AD if domain join is needed |
| 99.9% availability | Regional resources with multi-zone deployment. Note this is only three nines, so multi-region is over-engineering unless something else demands it |
| Consistent logging, retention, monitoring, alerting | Aggregated org-level log sink with `--include-children`, log buckets with retention, SLO-based alerting |
| Alerts ignored today | The fix is fewer, better alerts: SLO burn-rate alerting, not more email |
| Insights and predictions on industry trends | BigQuery, plus BigQuery ML or Vertex AI |
| Maintain regulatory compliance | Assured Workloads if a specific regime is named; CMEK; audit logs; Access Transparency |

**Traps specific to this case:**
- **HIPAA is never mentioned.** The requirement is "maintain regulatory compliance". If an option is attractive only because it says HIPAA, check whether the question actually established that constraint.
- **The legacy insurance integrations stay on-premises.** Any option that migrates or refactors them contradicts the case. The requirement is *connectivity to* them.
- **Customers are multi-national**, which pulls toward multi-region latency reduction and raises data residency, but the availability target is still only 99.9%.
- Redis and MongoDB are easy to forget. A question naming four databases expects four answers.
- The colocation lease expiring is the schedule pressure. It supports lift-and-shift-then-modernise over a long refactor.

---

## Case Study 2: Cymbal Retail

### Source

**Company.** Cymbal is an **online retailer** experiencing significant growth, specializing in a large assortment of products across several retail sub-verticals, which makes managing their extensive product catalog a constant challenge.

There are **no physical stores**. There is **no Oracle database**. There is **no monolith**. There is **no PCI DSS requirement and no Black Friday scenario** in the source document.

**Solution concept**, in three core areas:
- **Catalog and content enrichment** using gen AI to generate product attributes, descriptions, and images from supplier-provided information
- **Conversational commerce with product discovery**: AI-powered virtual agents in the website and mobile app, using **Google Cloud's Discovery AI** to process requests and retrieve relevant products
- **Technical stack modernization**: cloud-based infrastructure, secure and efficient data handling, third-party integrations, proactive monitoring and security

**Existing environment.**
- A mix of on-premises and cloud-based systems
- Databases: **MySQL, Microsoft SQL Server, Redis, and MongoDB**
- **Kubernetes clusters to run containerized applications**
- Legacy file-based integrations with on-premises systems, including **SFTP file transfers and ETL batch processing**
- A custom-built web application that lets customers browse the catalog by **querying the relational databases for names and categories**
- An **IVR system** to handle initial customer calls and route them
- **Call center agents** who receive transferred calls and **manually enter orders** when a customer cannot complete a transaction
- Monitoring via **Grafana, Nagios, and Elastic**

Stated challenges: manual processes are time-consuming and error-prone, data silos limit a unified view of the customer journey, and integrating new technologies is difficult.

**Business requirements.**
- Automate product catalog enrichment
- Improve product discoverability
- Increase customer engagement
- Drive sales conversion
- **Reduce costs: reduce call center staffing costs and data-center hosting costs**

**Technical requirements.**
- **Attribute generation** from supplier data including titles, descriptions and images, aligned to the product category and existing catalog structure
- **Image generation and enhancement**: different product image variations from a base image (for example various colors), background changes, product color adjustments, and text overlays
- **Automate product discovery**: process natural language requests and return highly relevant results
- **Scalability and performance** for an extensive catalog and anticipated growth
- **Human-in-the-loop (HITL) review**: a UI for associates to **approve, reject, or modify** gen AI content before it updates the catalog
- **Data security and compliance** for customer data and virtual agent interactions

### Working the case

*Inference below, not source material.*

| Stated requirement | Reasonable service answer |
|---|---|
| Attribute and description generation from supplier data | Gemini on Vertex AI, batch prediction over supplier feeds |
| Image generation, variants, background and color changes, text overlays | Imagen on Vertex AI |
| Natural language product discovery | Vertex AI Search for commerce (the case names it Discovery AI) |
| Virtual agents in web and mobile | Conversational Agents (Dialogflow CX) |
| Replace IVR and reduce call center staffing | Conversational Agents with telephony integration |
| **HITL review UI** | A web app, an approval queue, and a workflow gate before catalog write. Human review is a stated requirement, so any fully automatic pipeline is wrong |
| Existing Kubernetes | GKE, or GKE Enterprise if fleet management is implied |
| MySQL, SQL Server | Cloud SQL |
| Redis, MongoDB | Memorystore; Firestore Enterprise edition |
| SFTP and ETL batch legacy integrations | Storage Transfer Service, Cloud Storage landing bucket, Dataflow or Cloud Composer |
| Grafana, Nagios, Elastic | Cloud Monitoring, Managed Service for Prometheus, Log Analytics |
| Reduce data-center hosting costs | Migrate remaining on-prem workloads; committed use discounts |
| Data security and compliance for AI interactions | Model Armor, Sensitive Data Protection on prompts and responses |

**Traps specific to this case:**
- **Human-in-the-loop is mandatory.** An option that publishes gen AI output straight to the live catalog fails a stated requirement, however elegant it looks.
- **Image generation is in scope.** People remember the text side and forget Imagen.
- The company is **online only**. Anything about in-store inventory, point of sale, or omnichannel fulfilment is fabricated.
- The existing search is **keyword queries against relational tables**, which is why relevance is poor. The fix is a retrieval product, not a bigger database.
- **Cost reduction is explicitly about call center staffing and data-center hosting**, not about compute right-sizing.

---

## Case Study 3: Altostrat Media

### Source

**Company.** A prominent media company with an extensive collection of audio and video content: podcasts, interviews, news broadcasts, and documentaries.

**Altostrat is already running on Google Cloud.** This is the single most important fact about this case and the previous version of this file got it backwards.

**Existing environment.**
- **GKE** for the content management and delivery platform, for scalability and high availability
- **Cloud Storage** for the extensive media library across document, audio and video formats
- **BigQuery** as the primary data warehouse for user behavior, consumption patterns and audience demographics
- **Cloud Run functions** for serverless event-driven tasks: video transcoding, metadata extraction, personalized recommendations
- Some **legacy on-premises systems** remain, for **content ingestion and archival**, slated for modernization and migration
- Identity via a combination of **Google Identity and third-party identity providers**
- Monitoring via **Cloud Monitoring and Prometheus**, with **alerts primarily delivered via email**

**Business requirements.**
- Accelerate and enhance the reliability of operational workflows **across all environments (Google Cloud and on-premises)**
- Simplify infrastructure management for rapid application deployment
- Optimize cloud storage costs while maintaining high availability and scalability
- Enable natural language interaction with the platform, with 24/7 user support
- Automatically generate concise summaries of media content
- Extract rich metadata from media assets using NLP and computer vision
- Detect and filter inappropriate content
- Analyze media content to identify trends and extract insights
- Inform content strategy and decision-making with data

**Technical requirements.**
- **Modernize CI/CD for containerized deployments with a centralized management platform**
- Secure, high-performance **hybrid cloud connectivity for data ingestion**
- Provide **scalable, performant Kubernetes environments both on-premises and in the cloud**
- Optimize cloud storage costs for growing media volumes
- Design AI-powered detection of harmful content
- **Ensure that AI systems are auditable and their decisions can be explained**
- Leverage LLMs and conversational AI for personalized experiences and content virality
- Develop advanced chatbots with natural language understanding
- Automated summarization for diverse media

**Executive statement.** "**Reliability and cost management are our top priorities.**"

### Working the case

*Inference below, not source material.*

| Stated requirement | Reasonable service answer |
|---|---|
| Kubernetes **both on-premises and in cloud**, centrally managed | GKE Enterprise fleets with GKE attached clusters or Google Distributed Cloud, plus Config Sync and Policy Controller |
| Modernize CI/CD with a centralized platform | Cloud Build plus Cloud Deploy, with Artifact Registry and Binary Authorization |
| Secure high-performance hybrid connectivity for ingestion | Dedicated Interconnect |
| **Optimize storage cost** for growing media | Storage Autoclass, or lifecycle rules to Nearline, Coldline and Archive |
| Harmful content detection | Model Armor for gen AI surfaces; Cloud Vision and Video Intelligence for media safe-search |
| **AI decisions auditable and explainable** | **Vertex Explainable AI**, plus model and prediction logging |
| Summarization, metadata extraction, NLP and computer vision | Gemini on Vertex AI; Speech-to-Text; Video Intelligence; Document AI |
| Chatbots and conversational AI, 24/7 support | Conversational Agents, grounded on their own content |
| Trends and content strategy insights | BigQuery, already in place |
| Alerts by email, reliability a top priority | SLOs with multi-window burn-rate alerting, and a real notification channel |

**Traps specific to this case:**
- **Do not propose a migration to Google Cloud.** They are already there. The on-premises remnant is only ingestion and archival.
- **Explainability is a stated requirement.** This is the one case where Vertex Explainable AI is directly demanded, and a black-box model fails a requirement.
- "Kubernetes both on-premises and in the cloud" is the phrase that makes this a **GKE Enterprise fleet** question rather than a plain GKE question.
- Storage cost optimization plus "growing volumes" and unpredictable access points at **Autoclass** rather than hand-written lifecycle rules.
- Reliability and cost are named as the top two priorities, which is your tie-breaker when two options both work.

---

## Case Study 4: KnightMotives Automotive

### Source

**Company.** A car manufacturer specializing in **autonomous self-driving vehicles**, including **BEVs, hybrids, and traditional ICE vehicles**. The BEV fleet has a modern in-vehicle experience; **hybrid and ICE vehicles do not, and are viewed poorly by critics and drivers**, causing declining sales and customer satisfaction. They want to modernize the consumer experience across all vehicles **within five years**.

**The online ordering system is unreliable.** Systems for customers to build a vehicle online for acquisition through a dealer are not delivering the data or reliability dealers need, **straining the relationship between KnightMotives and dealers**.

**Solution concept.** Shift from manufacturing cars to creating a complete "automotive experience": a consistent experience across models, AI-powered features, **new revenue from data monetization**, a digital focus, and better tools for mechanics and salespeople.

**Existing environment.**
- IT is largely on-premises with some applications on major cloud platforms
- Supply chain runs on an **outdated mainframe**
- **ERP is outdated**, making new promotions and dealer discounts difficult
- **Dealers have no budget for new equipment**
- Fragmentation across vehicles with **multiple code bases**, and significant **technical debt from backwards compatibility**
- **Network connectivity to manufacturing plants** and **vehicle connectivity in rural areas** are challenges

**Business requirements.**
- Personalized relationship with the driver, cohesive experience across all models
- Better build-to-order model, reducing time on the lot with transparency for dealers and customers
- **Monetize corporate data** to finance technology investment; current AI infrastructure is obsolete and corporate data is siloed
- **Security is paramount due to past data breaches**
- **Adherence to EU data protection regulations**, especially for autonomous platforms
- Significant investment in fully autonomous driving, initially in regions with favorable regulation
- **Employee upskilling, attracting top-tier talent, and better communication between business and technical teams**

**Technical requirements.**
- Consistent UX integrating AI features across all models; update in-vehicle hardware and software in legacy models; **reliable network connectivity especially in rural areas**
- **Network upgrades between plants and headquarters**
- Hybrid cloud strategy; gradually modernize or replace legacy systems
- Autonomous vehicle development and testing: cutting-edge AI/ML, **a robust simulation environment**, compliance with evolving regulations
- Data monetization: a robust data management platform, strict data security and privacy, scalable AI/ML infrastructure
- **Security framework, an incident response plan, and security awareness training for employees**
- Improve the online build-to-order system; modern dealer tools for sales, service and inventory; **a comprehensive CRM system**

### Working the case

*Inference below, not source material.*

| Stated requirement | Reasonable service answer |
|---|---|
| Outdated mainframe for supply chain | Mainframe modernization: refactor, or replatform via Dual Run. Note the source never names IBM Z or COBOL |
| Outdated ERP | Migrate or replace; SAP on Google Cloud if the ERP turns out to be SAP, but the source does not say it is |
| Hybrid cloud strategy | Interconnect, GKE Enterprise for consistent platforms across sites |
| Plant-to-headquarters network upgrades | Dedicated Interconnect, Network Connectivity Center |
| Rural vehicle connectivity | Store-and-forward on the vehicle, tolerant sync, edge processing |
| Simulation environment for autonomous driving | Batch or GKE with GPUs, Spot for fault-tolerant simulation runs |
| Scalable AI/ML infrastructure | Vertex AI, GPUs and TPUs, Dynamic Workload Scheduler for burst capacity |
| Data monetization, siloed corporate data | BigQuery as the platform, **Analytics Hub** for sharing and monetizing datasets |
| **EU data protection** | Data residency via org policy resource-location constraints, Assured Workloads EU, CMEK or Cloud EKM, Access Approval |
| Past breaches, security paramount | Security Command Center, VPC Service Controls, a formal incident response plan |
| **Security awareness training, upskilling, talent** | This is a people answer, not a product one. Objective 4.2 territory |
| Dealers have no budget for new equipment | Browser-based dealer tools, no on-site hardware refresh |
| Unreliable build-to-order system | Reliability engineering: SLOs, error budgets, autoscaling, removing single points of failure |
| CRM | A packaged CRM integrated via Apigee, rather than something built from scratch |

**Traps specific to this case:**
- **The source names no specific legacy products.** No SAP, no IBM Z, no COBOL, no Detroit or Stuttgart data centers. Those were invented in the previous version of this file. Do not let an option feel right because it matches a detail you memorised from here.
- **No vehicle telemetry volumes are given.** There is no "billions of events per day" in the source. Questions built on IoT scale arithmetic are not what this case supports.
- **Dealer relationships and dealer budget** are real constraints and are unusual for a technical exam. An option requiring dealers to buy hardware fails.
- **EU data protection** is the compliance anchor, not GDPR-by-name plus CCPA plus ISO 27001. Read what the question actually establishes.
- **Upskilling, talent and business-to-technical communication are stated business requirements.** They map to exam objective 4.2, and this is the only case that makes people and process an explicit requirement.

---

## Cross-case comparison

| | EHR Healthcare | Cymbal Retail | Altostrat Media | KnightMotives |
|---|---|---|---|---|
| Starting point | Colocation, partly containerized | Mixed on-prem and cloud, containerized | **Already on Google Cloud** | Largely on-premises |
| Driving event | Colocation lease expiring | Manual catalog work, call center cost | Gen AI modernization | Declining sales, dealer strain |
| Databases named | MySQL, MS SQL, Redis, MongoDB | MySQL, MS SQL, Redis, MongoDB | Cloud Storage, BigQuery | None named |
| Compliance anchor | "Maintain regulatory compliance" | Data security and compliance | Auditable and explainable AI | **EU data protection** |
| Distinctive requirement | Legacy integrations stay on-prem | **Human-in-the-loop review** | **Explainable AI**, on-prem plus cloud Kubernetes | **People**: upskilling and training |
| Monitoring pain | Email alerts often ignored | Grafana, Nagios, Elastic | Email alerts | Obsolete AI infrastructure |

Two patterns worth noticing:

- **Three of the four name the same four databases** (MySQL, MS SQL Server, Redis, MongoDB) or name none. If a database question appears, that set is the likely target.
- **Three of the four have an alerting or observability complaint.** Observability is more heavily represented in these cases than its section weight suggests.

## How to work a case study question

1. Read the question stem for the **stated constraint**, not the company name.
2. Ask what that constraint **rules out**. Most case study questions are decided by elimination against a requirement, not by picking the most capable service.
3. Check the option against the **existing environment**. An option that rebuilds something the case says already exists is usually wrong.
4. Apply the case's **stated priority** as the tie-breaker. Altostrat says reliability and cost. Cymbal says reduce call center and hosting costs. EHR says decrease administration costs.
5. Prefer the **cheapest option that meets the stated requirement**, not the most robust one.

## Practice

Practice questions: [case-study-questions.md](../questions/case-study-questions.md). Those 20 questions were rebuilt from these same sources on 2026-08-09, so every stem quotes or closely paraphrases a real requirement.

Use `drill EHR` (or any case name) for a constraint-by-constraint drill against this file.
