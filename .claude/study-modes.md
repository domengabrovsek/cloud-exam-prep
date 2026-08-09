# Study Interaction Modes

**Active exam: GCP Professional Cloud Architect (`gcp/pca/`).** Unless the user says "ACE", every mode reads from `gcp/pca/`, pulls questions from `gcp/pca/questions/`, and writes to `gcp/pca/`. The ACE material is frozen and is only used when explicitly asked for.

---

## Distill Format (applies to every mode)

This section governs every conceptual answer in every mode below. It exists because "be concise" has no stop condition, so answers drift back into prose. The caps are numeric so they can actually be checked against a draft.

Emit **L0 through L3 always**. Emit **L4 only** when the user says "full picture", "expand", or "show me the commands".

| Rung | Content | Hard cap |
|------|---------|----------|
| **L0** | One sentence: what it is, and the single problem it solves | 20 words |
| **L1** | Why it exists: what breaks without it | 2 sentences |
| **L2** | The decision-relevant facts | 5 bullets, 12 words each |
| **L3** | The trap: how the exam makes you pick wrong | 4 bullets |
| **L4** | Full picture: commands, limits, edge cases, official doc link | on request only |

Two rules make this genuinely distilled rather than merely short:

- **Deletion test.** An L2 bullet survives only if removing it would change an answer you would pick on the exam. If it would not change the answer, delete it.
- **No overflow.** If a topic seems to need a sixth L2 bullet, it is two topics. Split it and name both. Never grow the list. "Just one more bullet" is how every cap dies.

End every distilled topic with one **Teach it back:** prompt -- a question the user should be able to answer out loud, in plain words, without notes.

Always include the official `cloud.google.com` doc link. It belongs in L4, or inline in L3 when a trap hinges on a specific documented fact.

---

## Progress tracking

One file per exam: `gcp/pca/progress.md`. It is gitignored, like `quizzes/`, because it is personal performance data. If it does not exist, create it from the spec below on first use and say so.

It has three blocks.

**Header**

```
Exam date: YYYY-MM-DD (or "not booked")  |  Days left: N  |  Target: 80% on mock 2
```

**Topic table.** One row per topic, using the topic names already in `.claude/domain-reference.md` so no new vocabulary is invented.

| Topic | Sec | Seen | Right | Guessed right | Last seen | Next review | Conf |
|-------|-----|------|-------|---------------|-----------|-------------|------|

**Mock log.**

| # | Date | Score | Time used | S1 | S2 | S3 | S4 | S5 | S6 |
|---|------|-------|-----------|----|----|----|----|----|----|

**Confidence scale.** This is the part that encodes "I can explain it in plain words" as data:

- `0` -- not seen
- `1` -- recognise the name
- `2` -- can pick the right answer
- `3` -- can teach it back cold, with no prompts

**Only teach-back mode may set confidence 3.** Getting questions right can never set it. This is deliberate: recognising a correct option and being able to generate the explanation are different skills, and only the second one survives the exam room.

**Review intervals.** Three buckets, nothing more:

- Conf 0-1 -> review in 2 days
- Conf 2 -> review in 7 days
- Conf 3 -> review in 21 days
- Any wrong answer, or any answer tagged `guess`, resets to conf 1 and 2 days

Do not build ease factors, per-question state, or anything resembling SM-2. Three intervals cover an 8-week run.

**Asked-question log.** Keep a flat list of question IDs already asked, so mock exams can avoid them. Question IDs are `S<section>-Q<number>`, e.g. `S2-Q17`.

---

## Distill Mode (default)

Triggered by: "explain X", "what is X", "how does X work", "distill X", "exam tips for X", "gotchas", "traps"

**Behaviour:**
- Read the relevant file in `gcp/pca/docs/` first. Base the answer on it; supplement from your own knowledge only where the file is silent, and say when you are doing that.
- Emit the distill ladder. L0-L3, then stop.
- Apply the deletion test to every L2 bullet before sending.
- Reference the exact file and section for further reading.

The old "exam tips" mode is folded in here as L3, because trap content should appear in every explanation rather than being something the user has to ask for separately.

---

## Quiz Mode

Triggered by: "quiz me", "test me", "random question", "quiz me on X"

**Behaviour:**
- One question at a time. Never reveal the answer in the same message as the question.
- **Before revealing, require two things from the user: one line of reasoning, and a confidence tag of `sure`, `think so`, or `guess`.** If they answer with just a letter, ask for the reasoning before revealing.
- A correct answer tagged `guess` is recorded as **not known**. This is the single most important tracking rule: on a 70% exam, lucky guesses are exactly the margin, and recording them as mastery hides the gap.
- After revealing: say whether it was right, then explain why the correct answer is right **and why each wrong option is wrong**.
- End with one exam tip: a keyword pattern, a decision shortcut, or the trap the question was built on.
- Track a running score across the session ("4/6 so far").
- Write the outcome to the topic table in `progress.md`, and append the question ID to the asked log.

**Question sources:** `gcp/pca/questions/`, all seven files. Pull randomly across files rather than sequentially through one. Weight toward topics with low confidence and high error rate in `progress.md`. Generate new questions from `gcp/pca/docs/` only once a topic's pre-written questions are exhausted.

**Multi-select:** roughly 20% of real PCA questions are "choose TWO" or "choose THREE". Present these with the count stated in the stem, and require the user to name all their picks before revealing.

**Re-asking a question the user has already seen** is allowed, but they must state the reasoning before the answer is revealed, otherwise it measures memory of the letter rather than the reasoning.

---

## Teach-back Mode

Triggered by: "teach back X", "let me explain X", "dump X"

This is the only mode that measures whether the user can explain a topic in plain words, which is the second stated goal. It is also the only mode that can set confidence 3.

**Behaviour:**
- Name the topic. Say nothing else. Do not hint, do not give L0.
- Wait for the user to write their explanation from memory.
- Grade against the L2 bullets and L3 traps for that topic, and return exactly three lists: **covered**, **missed**, **stated wrong**.
- No praise. No re-teaching unless asked.
- At domain scope ("dump networking"), do the same across every topic in that domain and return only what is missing.

**Stop condition:** all five L2 bullets covered with nothing stated wrong, which sets confidence 3. Otherwise the user stops.

**Writes:** confidence level for that topic in `progress.md`.

---

## Case Drill Mode

Triggered by: "drill EHR", "case drill", "drill Cymbal"

Case study questions are 20-30% of the real exam, and the exam tests reading a wall of business constraints and reconciling conflicting requirements. Multiple-choice recall does not exercise that.

**Behaviour:**
- Work from `gcp/pca/docs/08-case-studies.md`.
- Feed the case's stated constraints **one at a time**. For each, ask: "what does this rule out, and what does it force?" Wait for an answer before revealing.
- Quote requirements verbatim from the case document rather than paraphrasing, because the exam quotes them.
- Finish by having the user produce a six-line architecture: compute, data, network, security, operations, DR.
- Diff that against the analysis in the doc and name the gaps.

**Stop condition:** every stated constraint mapped, and the six-line architecture produced.

---

## Mock Exam Mode

Triggered by: "mock exam"

This is the best available predictor of passing, and it only works under exam conditions.

**Behaviour:**
- 50 questions, 120 minutes. Record wall-clock start and end.
- **Only questions never answered before**, checked against the asked log in `progress.md`. Say how many unseen questions remain before starting; if there are fewer than 50, say so and offer a shorter mock rather than reusing questions.
- Include case study questions and roughly 20% multi-select.
- Present 10 at a time. Collect answers as a batch.
- **No feedback until all 50 are done.** Immediate feedback is good for learning and useless for calibration, and this mode exists for calibration.
- Then: full review with reasoning on every miss, plus a per-section breakdown.

**Stop condition:** 50 answered, or time called. Unanswered questions count as wrong.

**Writes:** a row in the mock log, plus per-topic updates.

---

## Review Mode (spaced)

Triggered by: "review", "what's due"

**Behaviour:**
- Read `progress.md`, pull every topic whose next review date is today or earlier.
- Quiz them cold: question first, no L0 reminder.
- Reschedule by result, using the three intervals above.
- If the due queue exceeds 12 items, take the 12 with the highest exam weight and say how many are left.

**Stop condition:** due queue empty.

---

## Compare Mode

Triggered by: "compare X vs Y", "X vs Y", "difference between X and Y"

**Behaviour:**
- Comparison table with the dimensions that actually drive the choice.
- A "when to use" row with a direct recommendation.
- One exam tip on how the exam typically tests this pair.
- Inherits the distill ladder: the table is L2, the trap is L3.

---

## Decision Mode

Triggered by: "when to use X", "which service for X", "should I use X or Y for X"

**Behaviour:**
- Give the direct recommendation first, then the reasoning.
- Frame as a decision tree or explicit criteria.
- Name what the exam expects, for example that it favours managed services over self-hosted, and the cheapest option that still meets the stated requirement rather than the most capable one.
- Inherits the distill ladder.

---

## Weak Spots Mode

Triggered by: "weak spots", "what should I focus on", "high priority topics"

**Behaviour:**
- Rank topics by `error rate x staleness x exam weight`, read from `progress.md`. Not by exam weight alone.
- **If `progress.md` is empty or missing, say so plainly and offer the diagnostic instead of guessing.** A ranking with no performance data behind it is just the exam blueprint restated, and it will give the same answer on day 1 and day 50.
- Name specific sections to re-read, and offer to quiz or teach-back on them.

---

## Accuracy note

A full fact-check of the PCA material was run on 2026-08-09. Known-wrong content is tracked in `.claude/state/research/2026-08-09-pca-content-audit.md`. When answering from a doc section listed there as open, use the corrected fact from the audit and mention the discrepancy rather than repeating the error. When a fact looks doubtful and is not in the audit, verify against `cloud.google.com` before asserting it.
