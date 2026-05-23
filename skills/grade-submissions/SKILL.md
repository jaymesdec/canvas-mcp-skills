---
name: grade-submissions
description: Grade student submissions for a Canvas assignment with teacher-supervised, AI-drafted scores and feedback. Supports text entries, Google Docs (text), Google Slides (text), and file uploads (notebooks, PDFs, code, images). Use when the teacher says "grade submissions", "grade this assignment", "grade student work", or provides a Canvas assignment link. Drafts scores and feedback, presents for teacher review, pushes only after explicit confirmation.
---

# Grade Submissions

Grade every submission for a single Canvas assignment. Drafts scores + 1–2 sentences of feedback per submission, presents a review table, pushes to Canvas only after the teacher confirms.

> **Grades are not posted to students automatically.** The skill writes the grade to Canvas, but whether students *see* the grade is governed by the assignment's Canvas posting policy (manual vs automatic). If your assignment uses **manual posting**, grades remain hidden until you click Post Grades in Canvas yourself. The skill will tell you the posting policy before any writes happen.

## When to use

A teacher has a Canvas assignment with student submissions and wants AI-assisted grading. Common phrasings:

- "Grade this assignment: https://…/courses/123/assignments/456"
- "Grade the watershed quiz"
- "Score all submissions for the prototyping project"

## When NOT to use

- **No submissions yet** — the skill needs actual student work to grade.
- **You want to give per-comment feedback only, no scores** — that's a separate manual workflow; this skill always drafts a score (or uses rubric criteria).
- **Re-grading a single student** — call `grade_submission` / `grade_with_rubric` directly via the MCP.

## Prerequisites

- canvas-mcp installed and configured.
- For Google Docs/Slides submissions: the **Claude Drive MCP** (already authenticated in the Claude.ai / Claude Desktop session).
- For visual / image-heavy grading: file-upload submission type works best (the skill renders PDFs and images directly).

### A note on pseudonyms during grading

By default, the Canvas MCP returns pseudonymized student names (`Student 1`, `Student 2`, …) — that's the FERPA-safe default. **The grade itself uses the numeric Canvas `user_id`** (which is always real, never anonymized), so grades attach to the right student regardless of pseudonymization. But the *teacher* may want to see real names while grading to remember which student is which.

Two options for the teacher:

1. **Run `create_student_anonymization_map` first** — gets a real-name → pseudonym mapping for the course (visible only if `CANVAS_MCP_ALLOW_DEANONYMIZE=true` on the server). The teacher uses this as a local lookup while reviewing the draft.
2. **Set `CANVAS_MCP_ALLOW_DEANONYMIZE=true`** in the MCP server env temporarily — then submission responses include real names. Flip it back to `false` after the grading session.

If neither is done, the review table shows pseudonyms. That's fine for many workflows — the grades land on the right students regardless.

## Workflow

### 1. Parse the assignment URL

Extract `course_id` and `assignment_id` from a URL like `https://{host}/courses/{course_id}/assignments/{assignment_id}`. If the URL can't be parsed or the teacher gave plain numbers ("course 914, assignment 12345"), use those directly. Ask for clarification only if you genuinely can't tell.

### 2. Preflight checks

Call `get_assignment_details(course_identifier, assignment_id)`. Stop and surface clearly if:

- **Course is concluded** (`workflow_state: "completed"`) — grades can't be posted.
- **`points_possible` is 0** — offer feedback-only mode (submission comments, no scores).
- **No submissions** — nothing to grade; surface a message and exit.

Also check:

- **If a rubric is attached** (`assignment.rubric` is non-null), call `get_assignment_rubric_details` to load the full rubric: criteria, ratings (descriptions + points per level), `free_form_criterion_comments` flag, and `use_for_grading`. Cache this — you'll need it in steps 3, 5, 6, and 8. The skill grades per-criterion instead of by total points.
- **Posting policy** — read the assignment's `post_manually` flag (or `post_policy` if available). If `post_manually: true`, surface a clear warning in step 3 and step 7: grades will be written but hidden from students until the teacher posts manually in Canvas.

### 3. Present the assignment and ask how to grade

Show the teacher:

- Assignment name + Canvas URL
- Points possible
- Submission count (and how many are already graded — warn that re-grading overwrites)
- Posting policy (and what that means)

Then ask:

- **No rubric:** "This is worth `<N>` points. How would you like me to grade it?" (Wait for plain-language criteria.)
- **Rubric attached:** Show the rubric — each criterion's ratings + point values. Say: "I'll grade per-criterion using this rubric. Any extra guidance on top of it (e.g., weight criterion X harder)?" Mention whether `free_form_criterion_comments` is on or off — that affects whether per-criterion comments get drafted.

### 4. Fetch and filter submissions

Call `list_submissions(course_identifier, assignment_id, include_rubric_assessment: true, include_submission_comments: true)`. Skip unsubmitted students (`workflow_state: "unsubmitted"`). Auto-grade empty/blank submissions as `0` with the comment "No response provided." For each submission with content, collect:

- The **submission body** (text, URL, or attachment IDs depending on type)
- The **submission's `attempt`** number (you'll need this for step 8)
- Any **teacher-authored submission comments** (filter by `author_id` matching an instructor — students' own comments are NOT notes). Treat these as private grading evidence.

Then route each submission by `submission_type`:

#### `online_text_entry`

Use the `body` text directly. Strip HTML tags for grading; preserve the original for any quoting in feedback.

#### `online_url`

Detect and route the URL:

- **`docs.google.com/document/d/<FILE_ID>/...`** → extract `FILE_ID` from between `/d/` and the next `/`. Call `mcp__claude_ai_Google_Drive__read_file_content(file_id: FILE_ID)` to get the text. Grade the text content.
- **`docs.google.com/presentation/d/<FILE_ID>/...`** → same extraction. Call `mcp__claude_ai_Google_Drive__read_file_content(file_id: FILE_ID)` to get the slide text. **Note: the Drive MCP returns text only; slide images aren't included.** Grade the text + the slide structure (titles, bullet density, slide count). If the teacher needs visual grading (layout, design, imagery), tell them: "For visual evaluation, ask students to also submit a PDF export of their slides next time — that submission type renders the visuals natively."
- **`docs.google.com/spreadsheets/d/...`** → skip with "Google Sheets not supported for AI grading — manual review needed."
- **Other URLs** → skip with "Non-Google URL — manual review needed."
- If the Drive MCP returns an error (e.g., file is restricted) → mark as "Could not access" in the review table; don't fail the whole batch.

#### `online_upload`

Use the `attachments[]` array from `list_submissions` output. For each attachment, call `download_submission_attachment(course_identifier, assignment_id, user_id, attachment_id?)` to save the file locally, then read it:

- **`.ipynb`** (Jupyter notebooks) → use the Read tool; it renders cells + outputs natively.
- **`.pdf`** → use the Read tool; it renders pages.
- **`.docx`** → extract text via `python3 -c "import docx; print('\n'.join(p.text for p in docx.Document('FILE').paragraphs))"`.
- **`.py`, `.js`, `.ts`, `.html`, `.css`, `.md`** → use the Read tool as plain text.
- **`.png`, `.jpg`, `.jpeg`, `.gif`** → use the Read tool; images render visually.
- **Other file types** → skip with "File type not supported for AI grading — manual review needed."

#### Other submission types

Skip with a note. Common cases: `media_recording` (skip, manual review), `online_quiz` (already auto-graded by Canvas), `none`.

### 5. Grade each submission

Evaluate each fetched submission against the teacher's criteria. For Google Slides, evaluate text content + slide structure (titles, sequencing, bullet density). For uploaded images, evaluate what's actually in the image (layout, content, accuracy).

**No rubric:**

Assign a single score clamped to `[0, points_possible]`. Don't exceed the assignment max — ever.

**Rubric attached:**

For each criterion, pick the rating that best matches the work and record `{criterion_id, points, rating_id}`. Sum criterion points to get the total grade (this matches `points_possible` when the top rating is selected everywhere). If a criterion can't be assessed from the submission (e.g., participation, in-class delivery), **leave it ungraded in the draft and flag it for manual review** — don't guess. The review table should surface these explicitly.

**Use existing teacher notes as evidence.** If the submission has teacher-authored comments captured in step 4 (e.g., notes typed during a live presentation), treat them as primary evidence — they reflect what the teacher actually observed, which the submission artifact alone may not capture. Lean on them especially for criteria the artifact can't show (delivery, participation, Q&A, in-class behavior). When notes conflict with the artifact, **prefer the notes** and flag the conflict in the review table.

### 6. Draft feedback

Default feedback style: 1–2 sentences, direct, references something specific from the student's actual work. Address by first name when names are visible; otherwise use the pseudonym Canvas returned.

**No rubric:**

One overall comment per submission. 1–2 sentences. Reference specific things from the work, not vague generalities.

**Rubric attached, `free_form_criterion_comments: true`:**

Draft a per-criterion comment for every criterion. Two sentences:

1. *Why this rating?* Reference something specific from their work.
2. *How to reach the next rating up?* Concrete, actionable. If they already earned the top rating, replace with brief reinforcement ("keep doing X").

**Rubric attached, `free_form_criterion_comments: false`:**

Don't draft per-criterion comments — Canvas uses the rating descriptions for those. Draft a single 1–2 sentence overall comment instead.

**Fold in the teacher's notes from step 4.** When a submission had teacher comments, weave their concrete observations into the drafted feedback (e.g., a note like "rushed the Q&A" should appear in the relevant criterion comment or overall comment). Don't quote the notes verbatim — they're often shorthand. Rewrite into student-facing prose.

**No voice / tone passes.** This skill leaves feedback in its default form. If the teacher wants their personal writing style applied, that's a separate manual step they do themselves — don't try to mimic a specific teacher's voice unless asked.

### 7. Present results for review

Show a summary table:

| user_id | Student | Type | Score | Feedback (truncated) | Teacher notes (if any) |
|---|---|---|---|---|---|
| 1001 | Student 1 | text | 8 / 10 | Strong watershed examples, but missing evapotranspiration. | — |
| 1002 | Student 2 | google_doc | 9 / 10 | Clear diagrams + solid analysis. | rushed conclusion |

Also show:

- **Average + range** of scores.
- **Skipped submissions** (and why) listed separately.
- **Ungraded criteria** (rubric grading only) — submissions where one or more criteria couldn't be assessed and need manual review.
- **Posting policy reminder** — "These grades will be written to Canvas but [hidden from students until you Post Grades manually | visible to students immediately]."

The teacher can:

- Adjust individual grades or feedback in natural language ("change Student 4 to 9 — they did mention the cycle")
- Re-grade everything with different criteria
- Approve the batch to push

**Rubric grading** review table shows per-criterion ratings compactly (e.g., `Evidence: Proficient (3/4); Clarity: Developing (2/4)`) and either per-criterion comments or the overall comment depending on the rubric setting. The teacher can edit any rating, any comment, or the whole row.

**Existing comments warning:** if the assignment uses manual posting and grades are currently hidden, posting grades will *also* reveal any existing teacher comments that were on those submissions. List which submissions are affected and ask whether to keep, edit, or delete them in Canvas first. **The skill does not delete comments — the teacher does that in Canvas if needed.**

### 8. Push grades

Always **dry-run first**, regardless of grading mode.

**No rubric — bulk_grade_submissions:**

```
bulk_grade_submissions(
  course_identifier,
  assignment_id,
  grades: {
    "1001": { posted_grade: 8, comment: "Strong watershed examples..." },
    "1002": { posted_grade: 9, comment: "Clear diagrams + solid analysis." }
  },
  dry_run: true
)
```

Show the would-be payloads. On teacher confirmation, call again with `dry_run: false` to actually post.

**Rubric attached — grade_submission_with_rubric per submission:**

For each submission, call:

```
grade_submission_with_rubric(
  course_identifier,
  assignment_id,
  user_id: 1001,
  rubric_assessment: {
    "_8027": { points: 3, comments: "..." },
    "_8028": { points: 4, comments: "..." }
  },
  comment: "<overall comment, if drafted>"
)
```

There's no dry-run for the rubric tool, so do a manual dry-run first: print the per-submission rubric_assessment payloads to the teacher BEFORE making any tool calls. On confirmation, send them sequentially or in small batches; report any failures and offer to retry.

**Posting against the latest attempt.** When `list_submissions` returns a submission, you captured its `attempt` field in step 4 — that's the latest attempt. Comments must anchor to that attempt, not an older one. If the grading tool doesn't expose an `attempt` parameter (currently it doesn't), verify after pushing by re-fetching the submission and checking that the new rubric assessment + comment are visible on the latest attempt. If they landed on an older attempt, stop and surface this — the student won't see them in their current submission view.

### 9. Report results

After the push:

> Graded N submissions on **Watershed Unit Test** (course DSGN 9).
> Average: 7.4 / 10. Range: 4–10. 2 submissions skipped (manual review).
>
> Grades are now in Canvas. The assignment uses **manual posting** — students cannot see their grades or comments yet. To release them, open the assignment in Canvas and click **Post Grades**.
>
> [Or, for auto-post assignments:]
>
> The assignment uses **auto-posting** — students can see their grades and comments now.

If any pushes failed, list them with the user_id, the error message, and offer to retry.

## Feedback style

Default: 1–2 sentences, direct, references something specific from the student's actual work. Use contractions. Avoid generalities ("good job", "needs improvement" with no specifics). For Slides: comment on the text content + structure (titles, bullets, sequence) — visual layout grading requires the teacher to ask for it explicitly with a PDF export.

**No voice or tone matching.** Feedback stays in the default form unless the teacher explicitly says "rewrite in my voice" — at which point that's the teacher's manual step, not the skill's.

## Common mistakes to avoid

- **Never exceed `points_possible`.** Clamp single-score grades to `[0, points_possible]`. For rubric grading, each criterion's points must match one of the rubric's ratings — don't invent in-between values unless the rubric explicitly allows free-form points.
- **Always dry-run first.** Show the teacher the would-be payloads before any real write. The teacher must explicitly say "go ahead" (or equivalent) before you flip `dry_run: false` or send the rubric writes.
- **Don't guess on un-assessable criteria.** If a rubric criterion measures something the submission artifact doesn't show (delivery, participation), leave it ungraded in the draft and flag for manual review. Guessing here produces worse-than-useless feedback.
- **Don't quote teacher notes verbatim.** They're shorthand. Rewrite into student-facing prose when folding them into feedback.
- **Don't push grades that land on stale attempts.** Verify via re-fetching the submission after the push that the rubric assessment and comment landed on the latest `attempt`.
- **Don't mimic a teacher's voice.** Default feedback only. Voice/tone matching is the teacher's manual step.
- **Don't fail the whole batch on one stuck submission.** Google Drive access errors, file-type-not-supported, etc. should mark the affected submission as skipped and continue.
- **Don't delete student or teacher comments.** If existing comments will be revealed by posting grades, surface them to the teacher and let them clean up in Canvas.

## Example

**Teacher:** "Grade this assignment: https://franklinjc.instructure.com/courses/914/assignments/12345"

**What happens:**

1. Skill parses: course 914, assignment 12345.
2. `get_assignment_details` → 10-point assignment "Watershed Quiz", no rubric, 24 submissions, manual posting policy.
3. Skill: "This is a 10-point quiz, no rubric attached, 24 submissions, manual posting (grades stay hidden until you Post Grades in Canvas). How would you like me to grade it?"
4. Teacher: "Full credit for naming all 3 stages of the water cycle, minus 3 for each missing."
5. `list_submissions` → 24 submissions: 18 text, 5 Google Docs, 1 Google Slides.
6. Drive MCP reads the Docs + Slides; skill grades each; flags 2 with empty submissions as `0`.
7. Skill presents review table — average 7.8/10, range 0–10.
8. Teacher adjusts 2 scores, approves.
9. `bulk_grade_submissions` with `dry_run: true` → confirms payloads.
10. `bulk_grade_submissions` with `dry_run: false` → grades posted.
11. Skill: "Graded 24 submissions. Average 7.7/10. Grades are in Canvas but hidden — open the assignment and click **Post Grades** to release them to students."
