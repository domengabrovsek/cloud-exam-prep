# Study Interaction Modes

## Quiz Mode

Triggered by: "quiz me", "test me", "random question"

**Behavior:**
- Present ONE question at a time
- Use scenario-based format matching the real exam (4 options, A-D)
- Wait for the user's answer before revealing the correct one
- After the answer: confirm correct/incorrect, explain WHY, mention common traps
- Track score if doing multiple questions in a row (e.g., "3/5 so far")
- If the user gets it wrong, briefly explain the relevant concept and reference the study file section

**Question sources (140 questions organized by exam section):**
1. `gcp/ace/questions/section-1-cloud-environment-setup.md` -- 25 questions
2. `gcp/ace/questions/section-2-planning-and-implementing.md` -- 58 questions
3. `gcp/ace/questions/section-3-operations.md` -- 34 questions
4. `gcp/ace/questions/section-4-access-and-security.md` -- 23 questions
5. `gcp/ace/questions/google-official-sample.md` -- 20 Google official sample questions
6. Generate new questions from study guide content in `gcp/ace/docs/` when existing questions are exhausted

**PCA question sources (260 questions organized by exam section):**
1. `gcp/pca/questions/section-1-designing-planning.md` -- 65 questions
2. `gcp/pca/questions/section-2-provisioning-infrastructure.md` -- 45 questions
3. `gcp/pca/questions/section-3-security-compliance.md` -- 45 questions
4. `gcp/pca/questions/section-4-optimizing-processes.md` -- 35 questions
5. `gcp/pca/questions/section-5-managing-implementations.md` -- 25 questions
6. `gcp/pca/questions/section-6-operations-excellence.md` -- 25 questions
7. `gcp/pca/questions/case-study-questions.md` -- 20 case study questions
8. Generate new questions from study guide content in `gcp/pca/docs/` when existing questions are exhausted

**Random selection rules:**
- Pick questions randomly across ALL section files -- do NOT go sequentially through one file
- Mix sections and topics to simulate real exam randomness
- Avoid repeating questions within a session
- Weight toward the user's weak areas (check latest file in `gcp/ace/quizzes/` if any exist)

**Saving quiz results:**
- When the user finishes a quiz (says "done", "stop", "wrap up", or after a set number of questions), save results to `gcp/ace/quizzes/{number}-{date}.md`
- Check existing files in `gcp/ace/quizzes/` to determine the next number (001, 002, 003...)
- File must include: date, score, all wrong answers with correct answers and explanations, weak area analysis, and study recommendations
- Reference the study docs in `gcp/ace/docs/` for each weak area

**PCA-specific quiz rules:**
- ~20% of PCA questions are multi-select ("Choose TWO" or "Choose THREE") -- present these with clear multi-select formatting
- Include case study context when asking case study questions (brief summary of relevant case study)
- Align explanations with WAF pillars when applicable
- PCA quiz results are saved to `gcp/pca/quizzes/{number}-{date}.md`

**Topic-specific quizzing:**
When the user asks for a specific topic, pull from the matching section file:
- "quiz me on setup" / "quiz me on billing" -> section-1 file
- "quiz me on compute" / "quiz me on networking" / "quiz me on GKE" -> section-2 file
- "quiz me on operations" / "quiz me on monitoring" -> section-3 file
- "quiz me on IAM" / "quiz me on security" -> section-4 file
- If not enough pre-written questions exist for a topic, generate new ones from the study guide

**PCA topic-specific quizzing:**
When the user specifies PCA or architect-level topics:
- "quiz me on PCA" / "quiz me on architect" → pull from all PCA question files
- "quiz me on PCA architecture" / "quiz me on PCA design" → section-1 file
- "quiz me on PCA networking" / "quiz me on PCA compute" / "quiz me on PCA storage" → section-2 file
- "quiz me on PCA security" / "quiz me on PCA compliance" / "quiz me on VPC-SC" → section-3 file
- "quiz me on PCA CI/CD" / "quiz me on PCA cost" / "quiz me on PCA DR" → section-4 file
- "quiz me on PCA terraform" / "quiz me on PCA API" / "quiz me on PCA IaC" → section-5 file
- "quiz me on PCA monitoring" / "quiz me on PCA SLO" / "quiz me on PCA operations" → section-6 file
- "quiz me on case studies" / "quiz me on EHR" / "quiz me on Cymbal" → case-study-questions file
- If not enough pre-written questions exist for a PCA topic, generate new ones from the PCA study guides

## Explain Mode (default)

Triggered by: "explain [topic]", "what is [topic]", "how does [topic] work"

**Behavior:**
- Read the relevant study file section first
- Give a concise, practical explanation (not textbook-style)
- Include the key gcloud command(s) if applicable
- End with 1-2 exam tips for that topic
- Keep it scannable: use bullet points, short paragraphs, and tables

## Compare Mode

Triggered by: "compare X vs Y", "X vs Y", "difference between X and Y"

**Behavior:**
- Present a comparison table with key dimensions
- Add a "when to use" recommendation row
- Include an exam tip about how the exam typically tests this comparison
- Reference the study file that has the full comparison

## Decision Mode

Triggered by: "when to use X?", "which service for [scenario]?", "should I use X or Y for [use case]?"

**Behavior:**
- Frame the answer as a decision tree or decision criteria
- Give the direct recommendation first, then explain why
- Mention what the exam expects (e.g., "the exam favors managed services over self-hosted")

## Exam Tips Mode

Triggered by: "exam tips", "gotchas", "traps", "what to watch out for"

**Behavior:**
- Pull from the exam tip sections at the end of each domain file
- Organize by domain or topic as requested
- Focus on the counter-intuitive facts that trip people up

## Weak Spots / Focus Mode

Triggered by: "weak spots", "what should I focus on", "high priority topics"

**Behavior:**
- **ACE:** Emphasize Domain 3 (25% weight -- biggest impact). Highlight: IAM least privilege, service selection, networking basics, Terraform
- **PCA:** Emphasize Section 1 (25% weight -- biggest impact). Highlight: architecture trade-offs, DR/HA patterns, WAF pillars, case studies, VPC-SC, Vertex AI, Terraform modules
- Suggest specific sections to re-read
- Offer to quiz on those areas
