---
title: create-quiz skill — requirements
type: feat
status: revised
date: 2026-05-23
---

# create-quiz skill — requirements

> **Revision note (2026-05-23):** This document was revised after a multi-persona document review surfaced 11 substantive design concerns. The biggest shifts: verification reframed as source-quote anchoring (not LLM self-check); auto-detection of generation-vs-conversion replaced with explicit teacher confirmation; plan-assessment ↔ create-quiz linkage enforced (not aspirational); success criteria changed from unfalsifiable correctness rates to observable process metrics; essay_question deferred from v1; spec-preview step collapsed into the draft-review step.

## Problem

`plan-assessment` creates a Canvas **page** describing an assessment (intro, structure, AI-use policy, grade boundaries). It does **not** create the actual graded Canvas Quiz object with questions inside. There's a clear gap: when a teacher's assessment is a quiz or test, they want both — a description page (handled by `plan-assessment`) AND a Canvas Quiz object with actual questions (no skill exists yet).

Today, teachers wanting an AI-assisted quiz either:

1. Manually type questions one-by-one in Canvas's quiz builder
2. Draft questions in a Word/Google Doc and copy-paste into Canvas
3. Use a separate AI tool to generate questions, then copy them in

All three are slow and lossy. The MCP already has `create_quiz` and `create_quiz_question` tools — there's no skill that orchestrates them around a sane teacher workflow.

Note that pain points #2 and #3 are **conversion / digitization** workflows (teacher already has questions, needs them in Canvas) while #1 is closer to **generation** (teacher starts from scratch). The skill must serve both — see "Workflow modes" below.

## Goals

- Let a teacher generate a Canvas Quiz from existing course content (Canvas Page, PDF, Google Doc, or pasted text) in **under 5 minutes including their review.**
- Let a teacher convert a pre-written quiz document into a Canvas Quiz (extraction).
- Produce questions where the marked-correct answer is **anchored to a verbatim source quote or specific source reference**, so the teacher can verify it in seconds, not minutes.
- Always create the quiz as a **draft** — the teacher publishes manually after review in Canvas.
- Compose cleanly with `plan-assessment` when the assessment is quiz-style (no hidden coordination cost on the teacher).
- Generic across schools (no Franklin-specific assumptions).

## Non-goals

- New Quizzes API support (deferred — Canvas's New Quizzes API has partial coverage and risky maturity; revisit when Canvas publishes a stable API).
- Complex question types in v1: `matching`, `fill_in_multiple_blanks`, `numerical`, `multiple_dropdowns`, `calculated`, `file_upload`, **and `essay_question`** (deferred alongside rubric attachment — essay-without-rubric is a blank text box that adds little AI value over manual creation).
- Rubric attachment to quiz questions (would need an `attach_rubric_to_quiz_question` MCP tool).
- Cloning / refactoring existing Canvas quizzes (would need `list_quiz_questions` MCP tool; deferred).
- Question groups / question banks for randomization.
- Auto-publish to students. Quizzes are always created unpublished; teachers post manually.
- Auto-recovery from partial failures during quiz creation (the MCP has no `delete_quiz` tool; partial-success quizzes are left for teacher cleanup with a clear error report — see workflow step 5).

## Locked decisions (from brainstorm, revised after review)

1. **Workflow mode**: skill supports both **generation** (drafting questions from source content) and **conversion** (extracting pre-written questions from a quiz document). **Mode is set by explicit teacher confirmation, not auto-detection.** When the input has quiz-like structure, the skill asks: *"I see structured questions in this input — extract them as-is, or use this as reference to generate new questions?"*

2. **Quiz engine**: Classic Quizzes only. The MCP's existing `create_quiz` + `create_quiz_question` tools target Classic; New Quizzes is deferred. **Step 1 of the workflow includes a precondition check**: the skill probes whether Classic Quizzes is enabled in the target course before any drafting work. If Classic is disabled at the school/account level, the skill surfaces a clear "your school appears to have moved to New Quizzes — this skill currently supports Classic only" message and exits cleanly.

3. **Question types (v1)**: `multiple_choice_question`, `true_false_question`, `short_answer_question`, `multiple_answers_question`, `text_only_question` (for section dividers). `essay_question` deferred to v2 alongside rubric attachment. All other types deferred.

4. **Input sources (v1)**: plain text pasted into chat (always works), Canvas Page URL (via `get_page_content`), PDF file (via Claude's Read tool), Google Doc (via `mcp__claude_ai_Google_Drive__read_file_content` — the same Drive MCP tool the existing `grade-submissions` skill uses).

5. **Teacher review workflow**: hybrid — skill drafts all N questions at once in a markdown table; teacher can approve the batch, edit individual questions in natural language, batch-edit ("make all MC questions harder"), or flag for redraft. **Skill maintains canonical state across turns using stable question IDs (Q-A, Q-B, Q-C, …) that don't renumber on remove. After every edit, the skill re-prints the full draft table so the teacher always sees the current canonical state.** On approval, all questions get created together.

6. **Quality safeguard**: **source-quote anchoring** (revised from "internal verification"). Each drafted question MUST cite a verbatim source quote or specific source reference (paragraph number, section heading, slide number) that grounds the correct answer. The teacher's review is anchored to the cited source — they verify the citation matches the question, which takes seconds, rather than re-evaluating the question against the whole source document. **There is no model-self-check ✅/⚠️ vocabulary** — that would manufacture false confidence by using the same LLM that drafted the wrong answer to "verify" it. Source citations are the safeguard; teacher judgment is the gate.

## User experience flow

### Trigger phrases

"create a quiz", "make a quiz", "generate quiz questions", "build a Canvas quiz", "convert this quiz to Canvas", "quiz this lesson", and a teacher dropping in a PDF/Google Doc/lesson reference with quiz-creation intent.

### Workflow

#### 1. Capture inputs + precondition checks

Confirm: which course, what's the source content (link/PDF/text), what's the quiz title.

Then run two precondition checks before any drafting:

- **Classic Quizzes available?** The skill attempts a lightweight probe (e.g., `list_modules` on the course; if needed, a dry-run `create_quiz` with a sentinel title that the skill then deletes) to confirm Classic is enabled. If disabled, exit with the message in Locked Decision 2. If unsure (the probe is inconclusive), proceed but warn the teacher to verify on their first published quiz.
- **Source sufficiency.** If the source is a Canvas Page or short PDF (< ~500 words), the skill estimates whether the content can support the requested question count. If the source has only ~3–5 testable propositions and the teacher asked for 10 questions, the skill says: *"This source is about 200 words — I can reliably generate 4–5 questions about it. Want fewer questions, or supplement the source with more material (another page, your lecture notes, the textbook chapter)?"*

If the input has clear quiz-like structure (numbered question list, answer keys, "T/F:" prefixes, etc.), the skill **asks the teacher explicitly** which mode they want: *"I see structured questions in this input — extract them as-is (conversion mode), or use this as reference to generate new questions about the same topic (generation mode)?"*

#### 2. Draft the questions

(The previous "spec preview" step is collapsed into this step — sensible defaults are applied directly and shown to the teacher inline at the top of the draft table, where they can adjust the mix in the same review loop.)

**Default spec:**
- 10 questions
- Type mix: 7 MC, 2 short answer, 1 multiple-answers (no essay in v1)
- 1 point per question (essay's higher point value n/a since essay is v2)
- Total: 10 points
- No time limit
- 1 attempt allowed

Generate all questions per the default spec (or whatever the teacher specified up-front). For each question, **produce a source citation** — a verbatim quote or specific reference that grounds the marked-correct answer.

**Source-coverage rule**: questions must distribute across the source's identifiable sections proportional to section length, unless the teacher specifies a focus. This prevents the typical LLM batch-generation failure where 7 of 10 questions cluster on the first paragraph of the source.

Present in a single markdown table with stable question IDs:

```
Planned quiz: "Watershed Unit Quiz" (course DSGN 9)
Source: Canvas Page "The Water Cycle"
Spec: 10 questions • 7 MC + 2 SA + 1 MA • 10 points • No time limit • 1 attempt
Coverage: 4 questions on watershed structure, 4 on the cycle phases, 2 on human impact

| ID  | Type | Question | Correct | Distractors / Notes | Pts | Source citation |
|-----|------|----------|---------|---------------------|-----|-----------------|
| Q-A | MC | What drives the water cycle? | the Sun | gravity, plate tectonics, atmospheric pressure | 1 | "Solar energy drives evaporation, the engine of the water cycle" (paragraph 2) |
| Q-B | MC | Which is NOT a phase of the water cycle? | sublimation | precipitation, evaporation, runoff | 1 | "The four main phases are precipitation, evaporation, condensation, and runoff. Sublimation is mentioned briefly but not as a core phase." (paragraph 4) |
| Q-C | T/F | All watersheds drain to the ocean. | False | — | 1 | "Endorheic watersheds drain to interior basins, not the ocean" (paragraph 6) |
| Q-D | SA | What is evapotranspiration? | (accepts: water moving from plants AND soil into the atmosphere; the combined process of evaporation and plant transpiration) | — | 1 | "Evapotranspiration is the combined process by which water is transferred from the land to the atmosphere via evaporation from soil and transpiration from plants" (paragraph 3) |
| Q-E | MA | Which of these are types of precipitation? (select all) | Correct: rain, snow, sleet, hail; Incorrect: dew, runoff | (4 correct, 2 incorrect listed) | 1 | "Precipitation forms include rain, snow, sleet, and hail" (paragraph 5) |
| ... | ...  | ... | ... | ... | ... | ... |
```

**Per-type table conventions:**

- **MC**: `Correct` = the single correct answer text; `Distractors` = the 3 incorrect option texts (semicolon-separated)
- **T/F**: `Correct` = "True" or "False"; `Distractors` = `—`
- **Short answer**: `Correct` = the expected answer or a list of accepted variants in `(accepts: A; B; C)` format; `Distractors` = `—`
- **Multiple answers**: `Correct` = list of correct options; `Distractors` = list of incorrect options. Both sub-lists shown explicitly because MA correctness requires verifying both halves (correct options actually correct AND incorrect options actually incorrect).
- **text_only**: `Correct` and `Distractors` both `—`; `Source citation` becomes `(section divider — not graded)`

#### 3. Teacher review loop

Teacher can:

- Approve the whole batch ("looks good, create it")
- Edit individual questions by ID ("change Q-B — sublimation IS a phase, swap the answer")
- Batch-edit ("rewrite all MC questions to be harder")
- Flag for redraft ("redraft Q-E to focus on the carbon cycle instead")
- Add questions ("add 2 more short-answer about evapotranspiration")
- Remove questions ("drop Q-G")

**Multi-edit messages** (e.g., "change Q-A and also drop Q-C and add a question about runoff") are parsed and all operations applied in one pass. The skill always re-prints the full canonical draft table after every edit so the teacher sees the current state. Stable IDs don't renumber when questions are removed — Q-D stays Q-D even if Q-C is dropped.

Iterate until the teacher approves. Don't proceed until explicit approval.

#### 4. Confirm quiz settings

Before creating: title (confirmed), description (optional, auto-generated from spec but teacher can edit), time limit (default none), allowed_attempts (default 1), due_at (optional). Other settings get sensible defaults: `quiz_type: "assignment"`, `shuffle_answers: true`, `show_correct_answers: false`. **These defaults can be adjusted by the teacher inline during this step.**

#### 5. Create the quiz

Two-step in the MCP:

- `create_quiz(course_identifier, title, description, ...)` → returns `quiz_id`. **Always unpublished** (the MCP forces `published: false`).
- For each question in the approved draft, `create_quiz_question(course_identifier, quiz_id, question)`.

**Partial-failure policy:** if `create_quiz` succeeds but one or more `create_quiz_question` calls fail (Canvas 5xx, malformed payload, etc.), the skill:

1. Continues attempting the remaining questions (don't abort the batch on one failure)
2. Lists every failed question with the specific error message and the question payload
3. Offers to retry the failed questions
4. **Does NOT auto-delete the partial quiz** — the MCP has no `delete_quiz` tool, and the teacher may prefer to keep the partial draft and add missing questions manually in Canvas. The confirmation message explicitly notes the partial state.

#### 6. Confirm to teacher

On full success:

> Drafted **Watershed Unit Quiz** in DSGN 9 — 10 questions, 10 points, no time limit. Saved as a draft. Open in Canvas at <quiz `html_url` from create_quiz response>, review the source citations, and click **Publish** when you're ready.

On partial success:

> Drafted **Watershed Unit Quiz** in DSGN 9 — **8 of 10 questions created** (2 failed; details below). Saved as a draft.
>
> Failed: Q-D (Canvas returned 422 — short_answer with empty `accepts` list), Q-G (network timeout).
>
> Want me to retry the failed questions? Or open in Canvas at <html_url> and finish them manually.

### Conversion mode (explicit, not auto-detected)

When the teacher confirmed conversion mode at step 1, the skill:

- Extracts each question from the source
- Identifies question type from explicit structural cues (e.g., presence of "(a) (b) (c)" lettered options → MC; "T/F:" → T/F; long-response space or "Essay:" → would map to v2-deferred type and gets flagged for manual handling)
- Maps answers to Canvas format
- Cites the source location for each extracted question (e.g., "page 2, question 3" of the source PDF)
- Flags low-confidence extractions in the draft table with a specific reason ("question stem was truncated by page break", "answer key was not visible — please fill in", etc.)

Conversion mode uses the same draft-table → teacher-review → create-quiz flow from step 3 onward.

## Quality requirements

- **Source-quote anchoring is mandatory.** Every drafted question MUST include a verbatim source quote or a specific source reference (paragraph/section/slide number) that grounds the correct answer. Questions without a source citation are not approved-ready and the table flags them for the teacher.
- **Coverage and balance.** Questions distribute across the source's identifiable sections proportional to section length unless the teacher specifies a focus. The draft-table preamble shows the planned coverage (e.g., "4 questions on watershed structure, 4 on the cycle phases, 2 on human impact") so the teacher can spot clustering bias before reviewing individual questions.
- **Distractor quality.** For MC, distractors must be plausible but wrong. No "Earth is square" obvious-wrong choices. Distractors should be similar in length and grammatical form to the correct answer.
- **No double-negatives or ambiguous wording.** Each question tests one concept. Avoid "all of the above" / "none of the above" unless the teacher specifically requested them.
- **Multiple-answers questions need both halves verified.** For MA questions, the source citation must support both *which options are correct* AND *which options are incorrect*. The draft table shows correct and incorrect option lists separately for this reason.
- **Bloom's level matching.** When the teacher specifies "test recall" vs "test application", question stems use the appropriate verbs (identify/describe/list for recall; apply/analyze/explain for application).
- **Source faithfulness.** Generation-mode questions must be answerable from the source content. If a question's correct answer requires knowledge not in the source, fix the question or flag it as out-of-scope.

## Skill design principles (per repo conventions)

- **Never publishes.** Canvas Quiz created unpublished. Teacher publishes manually.
- **Generic by default.** No Franklin assumptions. Reads competency framework via `list_competencies` only if the teacher asks to align questions to competencies (optional, secondary).
- **Don't over-ask.** Confirm course + title + source. Use sensible defaults for everything else. Let the teacher adjust during the draft-table review (not in a separate pre-draft preview).
- **No voice / tone matching.** Default question phrasing only. If the teacher wants their personal voice in question stems, that's their manual edit step.

## Success criteria (revised — observable process metrics, not unfalsifiable claims)

These criteria are measurable from inside the skill or via lightweight teacher self-report. They replace the previous draft's unfalsifiable correctness rates.

- **Workflow time**: a teacher can generate a 10-question quiz from a Canvas lesson page in **under 5 minutes including review**, as self-reported.
- **Review burden**: the teacher edits or rejects **fewer than 1 in 5 drafted questions** during the review step (observable from the skill's interaction state — count edits/rejects against total draft size).
- **Source-citation completeness**: 100% of generation-mode questions ship with a source citation. (Observable: the skill should not create a quiz with any uncited question; this is enforced at draft-table generation, not post-hoc verified.)
- **Repeat-use intent**: the teacher reports they would use the skill again for their next quiz. (Qualitative; surveyable via a one-line follow-up the skill optionally surfaces after creation.)
- **Never-publish invariant holds**: zero quizzes ever created in a published state. (Enforced by the MCP's `published: false` hard-coding.)

We deliberately drop the previous draft's ">90% question correctness" and ">95% extraction accuracy" criteria — those required external grading on a sample set that isn't built into the workflow, and they collapsed into "as judged by the teacher" which makes the teacher the biased grader of their own approved content. The criteria above are what the skill can actually observe or honestly ask about.

## Relationship to other skills (revised — linkage is enforced, not aspirational)

The repo's existing skills split between **pedagogical planning** (plan-lesson, plan-assessment, plan-module, plan-course) and **operational artifact creation** (grade-submissions, this skill). `create-quiz` is the second member of the operational-artifact family.

### When `plan-assessment` produces a quiz-style assessment description page

`plan-assessment` creates a Canvas Page describing an assessment (intro, structure, AI policy, grade boundaries, etc.) — typically used for projects, presentations, essays. When the assessment **type** the teacher chose is "test" or "quiz", `plan-assessment` produces a description page that says things like "20 multiple-choice questions, 15% of trimester, 45-minute time limit."

The teacher then needs the actual graded Canvas Quiz object to match.

**The handoff is enforced**, not aspirational:

- When `plan-assessment` finishes a quiz-style assessment, it offers: *"Want me to create the actual Canvas Quiz for this assessment now? I'll use the spec you just confirmed (20 MC questions, 1 point each, 45 minutes, no AI tools allowed). Run `/create-quiz` and I'll inherit those values."*
- When `create-quiz` runs and the teacher mentions an assessment context that has a corresponding plan-assessment page in the same course (detected via `list_pages` matching the title), the skill **reads that page** and uses it as the spec source — title, point total, type mix, time limit, AI policy — instead of starting fresh.
- If the teacher edits one after the other (e.g., changes the quiz's point total after the page is published), the skill warns about drift: *"The plan-assessment page says 20 points but I just drafted 15. Do you want me to update the page, or update the quiz to match?"*

### Other relationships

- **`plan-lesson`** has an `assessment` slot in its Canvas page template. When that slot is intended to link to a graded Canvas Quiz, the teacher runs `create-quiz` to make the quiz object exist (or uses the plan-assessment → create-quiz handoff above), then the lesson's `assessment` slot links to the published quiz URL.
- **`plan-module`** mentions the summative assessment for each module. That can be `plan-assessment` (page) + `create-quiz` (quiz object) together, with the enforced linkage above.

The skill description should mention these relationships so Claude knows when to suggest `create-quiz` alongside the others.

## Open questions deferred to planning

These are implementation-level decisions for the planning phase:

- Exact wording of the explicit conversion-vs-generation question in step 1 (when the input has quiz-like structure)
- Heuristics for "what counts as quiz-like structure" that triggers the explicit confirmation question (just punctuation cues, or richer parsing?)
- How to implement the Classic-Quizzes-available precondition probe (which Canvas API call gives the cleanest signal; how to handle inconclusive results)
- Source-sufficiency estimation heuristic (testable-proposition density — word count + content-density signal, or just word-count thresholds?)
- Specific markdown table format for the draft preview (column widths, truncation rules for long question stems, how to render in chat UIs that may not support full markdown tables)
- How aggressively to use `text_only_question` for section dividers (e.g., between Q-F and Q-G to mark "Part 2: Short Answer")
- How the skill handles MIXED input (e.g., a Canvas Page + a PDF together — coverage rule applies across both?)
- Drift-detection scope when checking against a `plan-assessment` page — full-field check or only headline fields (point total, question count, time limit)?
- How to surface the spec inline at the top of the draft table (it's part of step 2 but the exact format above can be refined during skill prompt-engineering)

## Out-of-scope follow-ups (future skills / MCP work)

Worth noting so they're not forgotten:

- **`essay_question` support** — paired with rubric attachment. Defer together since essay-without-rubric provides little AI value over manual creation.
- **New Quizzes API support** — when Canvas publishes a stable, complete API for New Quizzes (currently partial). Likely a separate skill or a mode switch in this one.
- **`list_quiz_questions` MCP tool** — would unlock the "clone / refactor an existing quiz" use case. Single-purpose tool addition; could land in a small MCP release.
- **`delete_quiz` MCP tool** — would enable cleaner partial-failure rollback. Currently the partial-failure policy leaves the broken draft for teacher cleanup; with a delete tool, the skill could offer "discard and start over."
- **Complex question types** — `matching_question`, `fill_in_multiple_blanks_question`, `numerical_question`, `multiple_dropdowns_question`. Each has a specific Canvas API payload shape. Worth adding when teachers ask for them.
- **Question banks** — for randomization across attempts. Bigger feature; defer.
- **Auto-generate practice quizzes from grading data** — "create a quiz of the questions students got wrong on the last test." Very compelling but needs a `list_quiz_statistics` MCP tool and a fundamentally different skill design.
- **Scanned PDF support** — Claude's PDF read handles text-layer PDFs well but degrades on scanned PDFs (image-based, OCR-dependent). Out of scope for v1. If a teacher provides a scanned PDF, the skill should warn that OCR quality may affect output and recommend a text-layer PDF.
- **`list_unpublished_quizzes` MCP tool + a quiz-publication-review skill** — when multiple skills (e.g., `plan-module`) create quizzes in batches, teachers may forget to publish some. A small "show me all my unpublished quizzes" tool plus a follow-up skill would close that loop. Same pattern as the proposed `list_quiz_questions`.

## Implementation notes (for planning, not requirements)

- Skill lives at `canvas-mcp-skills/skills/create-quiz/SKILL.md` following the existing repo conventions.
- No MCP changes required for v1. The existing `create_quiz` + `create_quiz_question` tools cover all 5 needed question types (MC, T/F, short_answer, multiple_answers, text_only). Essay is deferred to v2 with rubric attachment.
- If a teacher asks for a deferred question type (matching, fill_in_multiple_blanks, essay, etc.), the skill should clearly say "v1 doesn't support that yet; create that question manually in the Canvas UI after the quiz is created" rather than failing silently or trying to fake it.
- **Quiz URL in confirmation message**: use the `html_url` field from the `create_quiz` response (already exposed on `CanvasQuizLite` in `src/tools/quizzes.ts` line 65) rather than constructing a URL from `CANVAS_API_URL` + course_id + quiz_id.
- **Encoding MC correctness**: the MCP's `create_quiz_question` expects each answer in the `answers[]` array to have an `answer_weight` of `100` for the correct option and `0` for incorrect options (per the zod schema in `src/tools/quizzes.ts` lines 22–28). The skill's prompt template should render questions in that shape, not assume Canvas infers correctness from some other field.
- **Encoding MA correctness**: same `answer_weight` 100/0 convention applies. Every option in the answer array gets a weight; the correct ones get 100, the incorrect ones get 0. Canvas computes "got all corrects + no incorrects = full credit" automatically.
- **Source citations in question metadata**: Canvas's question schema doesn't have a "source" field, so the skill should embed the citation in the question's `neutral_comments` or `correct_comments` field (visible in Canvas's SpeedGrader and the question editor). That keeps the source provenance with the question for future teacher edits, not just in the chat conversation that drafted it.
