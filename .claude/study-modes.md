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

**Sources for questions:**
1. First pull from `07-practice-questions.md` (60 pre-written questions)
2. If the user wants more or wants a specific topic, generate new questions based on the domain study files
3. When generating questions, match the exam style: scenario-based, not trivia

**Topic-specific quizzing:**
- "quiz me on IAM" -> questions from Domain 5
- "quiz me on GKE" -> questions from Domain 3 (section 3.2)
- "quiz me on billing" -> questions from Domain 1 (section 1.2)

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
