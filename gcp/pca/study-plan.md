# PCA Study Plan

**Exam date: not booked.** Book it before starting. A plan without a date produces no sequencing and no urgency, and every week without one is a week the schedule below cannot be anchored to.

Suggested target: **2026-10-04**, which gives the full 8 weeks from 2026-08-09. Register at [Google Cloud Certification](https://cloud.google.com/learn/certification/cloud-architect).

Once booked, replace `D` below with the real date and fill the header of `gcp/pca/progress.md`.

---

## Assumptions

- Around 4-6 hours per week, alongside full-time work.
- Two weekday evenings of 45 minutes, one weekend block of 2 hours.
- You already work with GCP daily, so this plan is weighted toward retrieval practice and architectural judgment, not toward learning services from scratch.

## Session shape

Every session, without exception:

- One topic. 45 minutes maximum.
- Ends with a teach-back.
- Ends with a write to `gcp/pca/progress.md`.

A session that does not write to the tracker did not happen. The tracker is what makes weak-spot ranking, spaced review, and mock-exam hygiene work; skipping the write silently breaks all three.

---

## The schedule

### D-56, week 1: baseline

You cannot plan around performance you have not measured. This week exists to produce data, not to learn.

- 30-question untimed diagnostic, spread across all six sections. Reasoning required on every answer, with a `sure` / `think so` / `guess` tag.
- Seed `progress.md` from the results.
- Distill passes on the three weakest domains that come out of it.

Expect the diagnostic to feel bad. That is the point: it is measuring, not teaching.

### D-49 to D-15, weeks 2 to 5: domain cycles

One to two domains per week, heaviest first. Section 1 (designing and planning) is the largest single block, so it goes first.

- **Evening A (45 min):** distill 3-4 topics, teach-back each one.
- **Evening B (45 min):** 20-question quiz on the week's domain.
- **Weekend (2 h):** 40-question mixed quiz, plus the review queue, plus a tracker update.

Suggested order, heaviest and most architectural first:

1. Week 2: designing and planning (section 1)
2. Week 3: security and compliance (section 3), provisioning infrastructure (section 2)
3. Week 4: optimizing processes (section 4), managing implementations (section 5)
4. Week 5: operations excellence (section 6), plus the Well-Architected Framework as a cross-cutting pass

### D-14, week 6: case studies

Case study questions are 20-30% of the exam, which makes this the highest-value week in the plan.

- One case drill per case study, 45 minutes each. Four cases, four sessions.
- The 20 case study questions in the bank.
- Re-read `docs/08-case-studies.md` for the two cases you found hardest.

Before this week starts, confirm the four published case studies are still current on the [certification page](https://cloud.google.com/learn/certification/cloud-architect). Google rotates them, and drilling a retired case is wasted time.

### D-13, week 7: mock exam 1 and repair

- **Mock exam 1**: 50 questions, 120 minutes, unseen questions only, no feedback until the end.
- The rest of the week is repair. Every miss becomes a distill pass and a teach-back.

### D-6, week 8: mock exam 2 and repair

- **Mock exam 2**, same conditions.
- Repair the misses.

### D-2 and D-1

- D-2: review queue plus L0 and L2 cards only. No new material. Adding material this late displaces consolidation and does not stick.
- D-1: rest.

---

## Go/no-go gate

- Mock 1 at **75% or better**
- Mock 2 at **80% or better**

Below 80% at D-6, move the date.

The pass mark is around 70%, so these targets carry deliberate headroom. A self-administered mock drawn from a bank you have been studying runs easier than the real exam: the phrasing is familiar, there is no time pressure from an unfamiliar interface, and the questions were written by the same process that wrote your study notes. Treat a 75% mock as roughly a coin flip on the day.

---

## What this plan deliberately does not include

- **More practice questions up front.** 260 is more than enough for 8 weeks, and every question consumed in practice is one that cannot appear in a mock. Generate new ones only for weak topics.
- **A third and fourth mock.** Two plus the diagnostic is enough, and mocks eat the unseen-question budget that makes them valid.
- **Re-reading the study guides end to end.** 15,000 lines cannot be re-read in the final week, and re-reading is the weakest form of study for a scenario exam. Retrieval practice beats review.
- **Invented case studies.** Only the four official ones match the exam's phrasing.

---

## Known content issues

A full fact-check ran on 2026-08-09 and found errors in the study guides, tracked in `.claude/state/research/2026-08-09-pca-content-audit.md`. The wrong answer keys are fixed. Doc-level errata is still open, so when a fact looks doubtful, check it against `cloud.google.com` rather than trusting the guide.
