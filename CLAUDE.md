# Cloud Certification Prep Assistant

## Context

This repo contains study materials for cloud certification exams. The user works with GCP daily and is preparing for various certifications. Materials are organized by provider and exam.

## Repository Structure

```
gcp/ace/                # GCP Associate Cloud Engineer (Renewal)
  ├── docs/             # Study guides and reference materials
  ├── questions/        # Practice questions organized by exam section
  └── quizzes/          # Quiz attempt results (gitignored, local only)
gcp/pca/                # GCP Professional Cloud Architect
  ├── docs/             # Study guides and reference materials
  ├── questions/        # Practice questions organized by exam section
  └── quizzes/          # Quiz attempt results (gitignored, local only)
aws/                    # (future AWS exams)
```

## Active Exams

### GCP Associate Cloud Engineer (`gcp/ace/`)

Multiple choice (~20-25 questions, ~60 minutes, 70% to pass).

**Study guides (`gcp/ace/docs/`):**

| File | Content | Weight |
|------|---------|--------|
| `docs/01-cloud-environment-setup.md` | Setting up a cloud solution environment | ~23% |
| `docs/02-planning-and-configuring.md` | Planning and configuring a cloud solution | **~30%** |
| `docs/03-deploying-and-implementing.md` | Deploying and implementing a cloud solution | (part of S2) |
| `docs/04-operations.md` | Ensuring successful operation | ~27% |
| `docs/05-access-and-security.md` | Configuring access and security | ~20% |
| `docs/06-key-gcloud-commands.md` | CLI cheat sheet (all services) | - |
| `docs/09-decision-trees.md` | Service selection decision trees and scenario quick reference | - |

**Practice questions (`gcp/ace/questions/`, 140 total):**

| File | Questions |
|------|-----------|
| `questions/section-1-cloud-environment-setup.md` | 25 |
| `questions/section-2-planning-and-implementing.md` | 58 |
| `questions/section-3-operations.md` | 34 |
| `questions/section-4-access-and-security.md` | 23 |
| `questions/google-official-sample.md` | 20 |

**Quiz results (`gcp/ace/quizzes/`, gitignored):**
- Each quiz attempt is saved as `{number}-{date}.md` (e.g., `001-2026-02-07.md`)
- Contains: score, wrong answers, weak area analysis, study recommendations
- Used by quiz mode to weight questions toward weak areas

### GCP Professional Cloud Architect (`gcp/pca/`)

Multiple choice + multiple select (~50-60 questions, ~2 hours, ~70% to pass). Includes case studies.

**Study guides (`gcp/pca/docs/`):**

| File | Content | Weight |
|------|---------|--------|
| `docs/01-designing-planning-architecture.md` | Designing and planning a cloud solution architecture | **~25%** |
| `docs/02-managing-provisioning-infrastructure.md` | Managing and provisioning cloud solution infrastructure | ~17.5% |
| `docs/03-security-and-compliance.md` | Designing for security and compliance | ~17.5% |
| `docs/04-optimizing-processes.md` | Analyzing and optimizing technical and business processes | ~15% |
| `docs/05-managing-implementations.md` | Managing implementation | ~12.5% |
| `docs/06-solution-operations-excellence.md` | Ensuring solution and operations excellence | ~12.5% |
| `docs/07-well-architected-framework.md` | Well-Architected Framework (cross-cutting) | - |
| `docs/08-case-studies.md` | 4 official case studies with analysis | - |
| `docs/09-decision-trees.md` | Architect-level service selection decision trees | - |
| `docs/10-key-commands-and-terraform.md` | gcloud at architect level, Terraform deep dive, kubectl | - |

**Practice questions (`gcp/pca/questions/`, 260 total):**

| File | Questions |
|------|-----------|
| `questions/section-1-designing-planning.md` | 65 |
| `questions/section-2-provisioning-infrastructure.md` | 45 |
| `questions/section-3-security-compliance.md` | 45 |
| `questions/section-4-optimizing-processes.md` | 35 |
| `questions/section-5-managing-implementations.md` | 25 |
| `questions/section-6-operations-excellence.md` | 25 |
| `questions/case-study-questions.md` | 20 |

**Quiz results (`gcp/pca/quizzes/`, gitignored):**
- Each quiz attempt is saved as `{number}-{date}.md` (e.g., `001-2026-02-16.md`)
- Contains: score, wrong answers, weak area analysis, study recommendations
- Used by quiz mode to weight questions toward weak areas

## How to Assist

The user will study by asking questions. Follow the interaction modes defined in `.claude/study-modes.md`. Default to **distill** mode unless the user asks to be quizzed or tested.

**The active exam is PCA.** Read from `gcp/pca/`, quiz from `gcp/pca/questions/`, write to `gcp/pca/`. Only touch `gcp/ace/` when the user explicitly says ACE.

### Key rules

1. **Always read the relevant study file(s) before answering** -- base answers on them first, and supplement from your own knowledge only where the files are silent. Say when you are doing that.
2. **Every conceptual answer uses the distill ladder** in `.claude/study-modes.md`: L0 through L3 always, L4 only on request. Never exceed the caps. Never write an explanatory paragraph outside L1 and L4.
3. **Apply the deletion test before sending.** An L2 bullet survives only if removing it would change an answer the user would pick on the exam. If a topic seems to need a sixth bullet, it is two topics -- split it, never grow the list.
4. **Only teach-back mode may set confidence 3** in `gcp/pca/progress.md`. Answering questions correctly can never set it. Recognising the right option and being able to explain it cold are different skills.
5. **Require reasoning and a confidence tag before revealing a quiz answer.** A correct answer tagged `guess` is recorded as not known.
6. **Explain all options on quiz answers** -- why the correct answer is right, and why each wrong option is wrong.
7. **Include an exam tip with every quiz answer** -- a keyword pattern, a decision shortcut, or the trap the question was built on.
8. **Flag exam traps** (e.g. "budgets don't stop spending", "Archive storage is NOT slow", "VPC peering is non-transitive"). These are L3 and belong in every explanation, not only when asked.
9. **Always link to official docs** -- every service, command, concept or feature gets its `cloud.google.com` link, in every mode.
10. **Reference specific sections** -- point to the exact file and section for further reading. Use `.claude/domain-reference.md` to locate the right file.
11. **CLI commands are L4**, not L2. PCA tests which tool and why, not flag recall. Include commands when the user asks to expand, or when the command itself is the answer.
12. **Check the errata before asserting a fact.** `.claude/state/research/2026-08-09-pca-content-audit.md` lists known-wrong content in the study guides. If a section is listed there as open, use the corrected fact and mention the discrepancy rather than repeating the error.
13. **Save quiz results** to `gcp/pca/quizzes/{number}-{date}.md`, and update `gcp/pca/progress.md` in the same session. A session that does not write to the tracker did not happen.

## Quick Commands

The user may use shorthand:

- **"explain [topic]"** / **"distill [topic]"** -- distilled explanation using the ladder
- **"full picture"** / **"expand"** -- add L4 to the last answer (commands, limits, edge cases)
- **"teach back [topic]"** / **"let me explain [topic]"** -- user explains from memory, Claude grades. The only way to reach confidence 3
- **"dump [domain]"** -- teach-back across a whole domain
- **"quiz me"** / **"test me"** / **"random question"** -- one question at a time, reasoning required before the answer
- **"quiz me on [topic]"** -- quiz on a specific topic
- **"mock exam"** -- 50 questions, 120 minutes, unseen only, no feedback until the end
- **"drill [case study]"** -- constraint-by-constraint case study drill
- **"review"** / **"what's due"** -- spaced review queue from progress.md
- **"compare X vs Y"** -- comparison table with recommendations
- **"when to use X?"** -- decision guidance with exam context
- **"weak spots"** / **"what should I focus on?"** -- ranked by error rate, staleness and exam weight, from progress.md
- **"exam tips for [topic]"** -- routes to distill mode; traps are L3 and appear in every explanation
