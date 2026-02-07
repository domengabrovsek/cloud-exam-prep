# GCP Associate Cloud Engineer - Renewal Exam Prep

Study guide for the Google Cloud Associate Cloud Engineer certification renewal exam.

## Exam Overview

| Detail | Value |
|--------|-------|
| **Format** | Multiple choice & multiple select |
| **Duration** | ~60 minutes |
| **Questions** | ~20-25 |
| **Passing score** | ~70% |
| **Cost** | $75 USD |
| **Validity** | 3 years from renewal |
| **Official page** | [Associate Cloud Engineer Certification](https://cloud.google.com/learn/certification/cloud-engineer) |
| **Exam guide** | [Exam Guide (detailed)](https://cloud.google.com/learn/certification/guides/cloud-engineer/) |
| **Register** | [CertMetrics](https://cp.certmetrics.com/google/en/login) |

## Exam Domains

**Standard exam:**

| # | Domain | Weight | Study Guide |
|---|--------|--------|-------------|
| 1 | Setting up a cloud solution environment | ~23% | [docs/01-cloud-environment-setup.md](./docs/01-cloud-environment-setup.md) |
| 2 | Planning and implementing a cloud solution | **~30%** | [docs/02-planning-and-configuring.md](./docs/02-planning-and-configuring.md), [docs/03-deploying-and-implementing.md](./docs/03-deploying-and-implementing.md) |
| 3 | Ensuring successful operation of a cloud solution | ~27% | [docs/04-operations.md](./docs/04-operations.md) |
| 4 | Configuring access and security | ~20% | [docs/05-access-and-security.md](./docs/05-access-and-security.md) |

**Renewal exam** (Section 1 not included):

| # | Domain | Weight |
|---|--------|--------|
| 2 | Planning and implementing a cloud solution | **~40%** |
| 3 | Ensuring successful operation of a cloud solution | **~40%** |
| 4 | Configuring access and security | ~20% |

**Additional study resources:**
- [docs/06-key-gcloud-commands.md](./docs/06-key-gcloud-commands.md) - CLI cheat sheet
- [docs/09-decision-trees.md](./docs/09-decision-trees.md) - Service selection decision trees and scenario quick reference

**Practice questions** (140 questions organized by exam section):

| Section | Questions | File |
|---------|-----------|------|
| 1 -- Setting up cloud environment | 25 | [questions/section-1-cloud-environment-setup.md](./questions/section-1-cloud-environment-setup.md) |
| 2 -- Planning and implementing | 58 | [questions/section-2-planning-and-implementing.md](./questions/section-2-planning-and-implementing.md) |
| 3 -- Ensuring successful operation | 34 | [questions/section-3-operations.md](./questions/section-3-operations.md) |
| 4 -- Configuring access and security | 23 | [questions/section-4-access-and-security.md](./questions/section-4-access-and-security.md) |
| Google official samples | 20 | [questions/google-official-sample.md](./questions/google-official-sample.md) |

**Quizzes** (gitignored, local only):
- `quizzes/` -- Quiz attempt results with scores and wrong answer analysis
- Naming: `{number}-{date}.md` (e.g., `001-2026-02-07.md`)

## 1-Day Study Strategy

Since you use GCP daily, focus on:

1. **Skim all study guides in `docs/`** - flag anything unfamiliar (~1-2 hours)
2. **Deep dive on weak spots** - especially Section 2 (30% standard / 40% renewal) (~1 hour)
3. **Review the CLI cheat sheet** - know the key commands for each service (~30 min)
4. **Do all practice questions** - aim for 85%+ before taking the real exam (~1-2 hours)
5. **Review wrong answers** - check your quiz results in `quizzes/` and re-read relevant sections (~30 min)

**Key exam trends:**
- Scenario-based questions (not pure memorization)
- Heavy on choosing the right service for a workload
- IAM and least-privilege principles come up frequently
- IaC (Terraform) is now part of the exam
- Cloud Operations suite (formerly Stackdriver) naming

## Practice Exams (External)

- [Google Official Sample Questions](https://docs.google.com/forms/d/e/1FAIpQLSfexWKtXT2OSFJ-obA4iT3GmzgiOCGvjrT9OfxilWC1yPtmfQ/viewform)
- [ExamTopics - GCP ACE](https://www.examtopics.com/exams/google/associate-cloud-engineer/)
- [Tutorials Dojo Practice Exams](https://tutorialsdojo.com/courses/google-certified-associate-cloud-engineer-practice-exams/)
- [Ditectrev GitHub - Practice Q&A](https://github.com/Ditectrev/Google-Cloud-Platform-GCP-Associate-Cloud-Engineer-Practice-Tests-Exams-Questions-Answers)

## Official Learning Resources

- [Cloud Engineer Learning Path (Cloud Skills Boost)](https://www.cloudskillsboost.google/paths/11)
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Google Cloud Cheat Sheet](https://googlecloudcheatsheet.withgoogle.com/)
- [GCP Flowcharts](https://grumpygrace.dev/posts/gcp-flowcharts/)
- [Awesome GCP Certifications (GitHub)](https://github.com/sathishvj/awesome-gcp-certifications)
