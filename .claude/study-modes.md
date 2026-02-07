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

**Topic-specific quizzing:**
When the user asks for a specific topic, pull from the matching section file:
- "quiz me on setup" / "quiz me on billing" -> section-1 file
- "quiz me on compute" / "quiz me on networking" / "quiz me on GKE" -> section-2 file
- "quiz me on operations" / "quiz me on monitoring" -> section-3 file
- "quiz me on IAM" / "quiz me on security" -> section-4 file
- If not enough pre-written questions exist for a topic, generate new ones from the study guide

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
- Emphasize Domain 3 (25% weight -- biggest impact)
- Highlight commonly tested areas: IAM least privilege, service selection, networking basics, Terraform
- Suggest specific sections to re-read
- Offer to quiz on those areas
