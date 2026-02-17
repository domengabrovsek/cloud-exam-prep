# GCP Professional Cloud Architect (PCA) Certification Prep

## Exam Overview

| Detail | Value |
|--------|-------|
| **Duration** | 2 hours |
| **Questions** | 50-60 (multiple choice + multiple select) |
| **Passing score** | ~70% (not officially published) |
| **Cost** | $200 USD |
| **Case studies** | 4 available, 2 appear on exam |
| **Recertification** | Every 2 years |

**Registration:** [Google Cloud Certification](https://cloud.google.com/learn/certification/cloud-architect)

## Exam Sections & Weights

| # | Section | Weight | Study Guide |
|---|---------|--------|-------------|
| 1 | Designing and planning a cloud solution architecture | **~25%** | [01-designing-planning-architecture.md](docs/01-designing-planning-architecture.md) |
| 2 | Managing and provisioning a cloud solution infrastructure | ~17.5% | [02-managing-provisioning-infrastructure.md](docs/02-managing-provisioning-infrastructure.md) |
| 3 | Designing for security and compliance | ~17.5% | [03-security-and-compliance.md](docs/03-security-and-compliance.md) |
| 4 | Analyzing and optimizing technical and business processes | ~15% | [04-optimizing-processes.md](docs/04-optimizing-processes.md) |
| 5 | Managing implementation | ~12.5% | [05-managing-implementations.md](docs/05-managing-implementations.md) |
| 6 | Ensuring solution and operations excellence | ~12.5% | [06-solution-operations-excellence.md](docs/06-solution-operations-excellence.md) |

## Supplementary Materials

| File | Content |
|------|---------|
| [07-well-architected-framework.md](docs/07-well-architected-framework.md) | Google Cloud WAF -- 6 pillars, cross-cutting reference |
| [08-case-studies.md](docs/08-case-studies.md) | 4 official case studies with analysis framework |
| [09-decision-trees.md](docs/09-decision-trees.md) | Architect-level service selection decision trees |
| [10-key-commands-and-terraform.md](docs/10-key-commands-and-terraform.md) | gcloud at architect level, Terraform deep dive, kubectl |

## Practice Questions (~260 total)

| File | Questions | Coverage |
|------|-----------|----------|
| [section-1-designing-planning.md](questions/section-1-designing-planning.md) | 65 | Business/tech requirements, DR, migration, AI/ML, networking |
| [section-2-provisioning-infrastructure.md](questions/section-2-provisioning-infrastructure.md) | 45 | Network topologies, storage, compute, Vertex AI |
| [section-3-security-compliance.md](questions/section-3-security-compliance.md) | 45 | IAM, encryption, VPC-SC, AI security, compliance |
| [section-4-optimizing-processes.md](questions/section-4-optimizing-processes.md) | 35 | CI/CD, cost optimization, stakeholder mgmt, DR |
| [section-5-managing-implementations.md](questions/section-5-managing-implementations.md) | 25 | API mgmt, testing, Terraform/IaC, Cloud SDKs |
| [section-6-operations-excellence.md](questions/section-6-operations-excellence.md) | 25 | WAF, observability, reliability, chaos engineering |
| [case-study-questions.md](questions/case-study-questions.md) | 20 | 5 per case study |

## Case Studies

The PCA exam includes case study questions. Four case studies are published in advance; two appear on the actual exam.

1. **EHR Healthcare** -- colocation migration, containers, HIPAA, hybrid connectivity
2. **Cymbal Retail** -- gen AI catalog enrichment, conversational commerce, modernization
3. **Altostrat Media** -- media platform on GKE, content AI, storage optimization
4. **KnightMotives Automotive** -- autonomous vehicles, IoT, mainframe modernization, EU data protection

See [08-case-studies.md](docs/08-case-studies.md) for full analysis of each.

## Key Differences from ACE

| Aspect | ACE | PCA |
|--------|-----|-----|
| Depth | "What does this service do?" | "When and why choose this architecture?" |
| Case studies | None | 4 official cases |
| Question format | Single-choice only | ~20% multi-select |
| WAF | Not covered | Central to all domains |
| AI/ML | Minimal | Extensive (Vertex AI, Gemini, Model Armor) |
| Terraform | Basics | Deep (state, modules, CFT, validation) |
