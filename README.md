# Cloud Certification Prep

Study materials for cloud certification exams, designed to be used with [Claude Code](https://claude.ai/claude-code) as an interactive study assistant.

## Exams

| Provider | Certification | Status | Guide |
|----------|--------------|--------|-------|
| GCP | Associate Cloud Engineer (Renewal) | Active | [gcp/ace/](./gcp/ace/) |

## How to Use This Repo

### Reading on your own

Browse the study materials directly:

```
gcp/ace/
├── docs/          # Study guides, CLI cheat sheet, decision trees
├── questions/     # 140 practice questions organized by exam section
└── quizzes/       # Your quiz results (local only, gitignored)
```

### With Claude Code (recommended)

This repo includes a `CLAUDE.md` that configures Claude Code as a study assistant. Open the repo in Claude Code and use these commands:

| Command | What it does |
|---------|-------------|
| `quiz me` | Random questions from all sections, tracks score, saves results |
| `quiz me on [topic]` | Focused quiz on a specific topic (e.g., "quiz me on networking") |
| `explain [topic]` | Concise explanation with gcloud commands and exam tips |
| `compare X vs Y` | Comparison table with "when to use" guidance |
| `when to use X?` | Decision tree for choosing the right service |
| `exam tips for [topic]` | Common traps and gotchas for that topic |
| `weak spots` | Suggests high-priority areas based on exam weights and past quiz results |

### Quiz behavior

- Questions are pulled **randomly** across all section files to simulate real exam conditions
- After each quiz, results are saved to `quizzes/{number}-{date}.md` with:
  - Score and pass/fail
  - Every wrong answer with the correct answer and explanation
  - Weak area analysis grouped by topic
  - Specific study guide sections to review
- Future quizzes **weight toward your weak areas** based on past results
- Quiz results are **gitignored** -- they're personal progress tracking, not shared content

## Disclaimer

Practice questions included in this repository are sourced from officially published sample questions provided by the cloud providers themselves (e.g., [Google's official sample questions](https://docs.google.com/forms/d/e/1FAIpQLSfexWKtXT2OSFJ-obA4iT3GmzgiOCGvjrT9OfxilWC1yPtmfQ/viewform)). No actual exam questions covered by NDA are included. Study guide content was generated with the help of [Claude Code](https://claude.ai/claude-code) and references official documentation.
