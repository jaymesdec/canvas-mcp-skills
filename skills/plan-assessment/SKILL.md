---
name: plan-assessment
description: Plan a Canvas assessment page (test, quiz, project, presentation, essay, etc.) using the school's assessment template. Use when the teacher says "plan an assessment", "create a test/quiz/project page", "draft an assessment", or describes any graded task that needs a Canvas page describing it. Generates content for all 6 required slots — description, pre-work, structure + grading + grade-boundaries table, submission instructions, time allotment, and AI-use policy — then publishes as a draft Canvas page. Generic — works with whatever assessment template the school config provides.
---

# Plan Assessment

Plan a graded task (test, quiz, project, presentation, essay) and create a Canvas **draft** page describing it. Covers all 6 required slots of the school's assessment template. Generic across schools.

> **This skill never publishes pages.** Every page lands in Canvas as a draft (`published: false`). The teacher reviews the draft in Canvas and clicks **Publish** themselves when ready. This is a hard policy — don't try to flip the published flag, and don't describe what the skill does as "publishing."

## When to use

A teacher describes a graded task that needs a Canvas page. Common phrasings:

- "Plan a 20-question quiz for DSGN 9 on the water cycle"
- "Create an assessment page for the prototyping project — 3 weeks, 25% of trimester"
- "I need an assessment for tomorrow's presentation day"
- "Draft a take-home essay assignment, 1500 words, due Friday"

The same template is used whether it's an in-class test or a long-term project — the slot content adapts.

## When NOT to use

- **Just a Canvas Assignment object** (with submission upload, due date, etc.) — that's not a Page. This skill creates the *describing page*, not the Canvas Assignment itself.
- **Single lessons** — that's `plan-lesson`.
- **Editing an existing assessment** — see "Updating an existing assessment" below.

## Prerequisites

- canvas-mcp v0.3.11+ installed and configured.
- A school config with an `assessment` page template (Franklin: bundled by default; other schools: set `SCHOOL_CONFIG` to point at your own).

## Workflow

### 1. Confirm the basics

Before generating any content, you need these answers. Some come from the teacher's initial request; ask only for what's missing:

- **Canvas course?** If named by code/fragment, call `list_courses` to confirm.
- **Assessment title?** Propose one if the teacher gave only a topic.
- **Assessment type?** Test / quiz / project / presentation / essay / lab / portfolio / other. This drives the AI-use policy defaults and shapes every slot.
- **Total points?** Drives the grade boundaries table. If the teacher said "out of 100" or "20 questions × 1 point each = 20 pts", use that. If they didn't say, ask.
- **Weighting?** What % of the trimester / term grade. If the teacher didn't say, ask.
- **Time?** For tests/quizzes: minutes (standard + extended time). For projects: duration (e.g., "3 weeks" or "Sept 8 → Sept 29") and key milestones if there are checkpoints.

What you don't need to ask:

- Module placement — handle that after the draft is created if the teacher mentioned a module.
- Publishing — the MCP creates the page as a draft; teachers publish manually in Canvas.
- HTML structure — the template handles it.
- "Should I include a rubric?" — if the assessment uses one, the teacher will mention it; if they don't mention one, just generate the grade boundaries table.

### 2. Discover the assessment template

Call `list_page_templates` and find the entry named `assessment`. Read its `slots` list — every slot has a `description` telling you what content goes there. The Franklin preset (and most schools that fork it) defines 6 required slots:

- `description` — top-level description of the assessment (intro + what students will do)
- `pre_work` — what students should complete before the assessment
- `structure_and_grading` — structure + weighting + grade boundaries table
- `submission` — how students submit
- `time` — time allotment / timeline
- `ai_use` — acceptable + unacceptable AI uses

If no `assessment` template exists in the school config, fall back to `template: "default"` or `template: "none"`, but flag this to the teacher so they know the page won't have the school's formal assessment chrome.

### 3. Generate slot content

For each slot, generate concrete classroom-ready HTML. Key guidance per slot:

#### `description`

The "what is this assessment" overview. Intro paragraph naming the topic + a list of what students will do.

```html
<p>20-question multiple-choice test on watershed geography and the water cycle.</p>
<p>Students will:</p>
<ol>
  <li>Identify the components of a watershed (precipitation, runoff, infiltration, evapotranspiration)</li>
  <li>Diagram water movement through a watershed</li>
  <li>Apply concepts to identify their home watershed on a regional map</li>
</ol>
```

Make it concrete. Avoid generic phrasings like "students will demonstrate their understanding." Say what the demonstration *is*.

#### `pre_work`

What students should have completed before the assessment. Most teachers under-specify this — fill it in with the actual prerequisite work from the unit. If the teacher hasn't told you what came before, ask, or propose something reasonable (e.g., "review the lesson pages from Week 3" with a checklist).

```html
<p>Before taking this assessment, complete the following:</p>
<ol>
  <li>Review the watershed geography lesson page</li>
  <li>Complete the watershed mapping worksheet from class</li>
  <li>Read pages 142–158 in the textbook</li>
  <li>Try the practice quiz (ungraded) on Canvas</li>
</ol>
```

#### `structure_and_grading`

Three pieces in one slot:

1. **Structure description** — one paragraph: number of questions / sections / tasks, point values per part, format.
2. **Weighting** — one paragraph stating what % of the trimester (or term) this assessment is worth.
3. **Grade boundaries table** — see "Grade boundaries" below.

Example:

```html
<p>20 multiple-choice questions, 1 point each. No partial credit.</p>
<p>This assessment is worth <strong>15% of the trimester grade</strong>.</p>
<p><strong>Grade Boundaries</strong></p>
<table border="1" style="border-collapse: collapse; width: 30%;">
  <thead>
    <tr><th>Letter Grade</th><th>Points</th></tr>
  </thead>
  <tbody>
    <tr><td>A+</td><td>20</td></tr>
    <tr><td>A</td><td>19</td></tr>
    <tr><td>A-</td><td>18</td></tr>
    <tr><td>B+</td><td>17</td></tr>
    <tr><td>B</td><td>16</td></tr>
    <tr><td>B-</td><td>15</td></tr>
    <tr><td>C+</td><td>14</td></tr>
    <tr><td>C</td><td>13</td></tr>
    <tr><td>C-</td><td>12</td></tr>
    <tr><td>D</td><td>10–11</td></tr>
    <tr><td>F</td><td>0–9</td></tr>
  </tbody>
</table>
```

#### `submission`

How students submit. Be specific about the mechanism:

- **In-class on paper:** "Take in class on paper. Hand in to the teacher when complete."
- **Canvas upload:** "Upload your file to the linked Canvas Assignment. Accepted formats: PDF, DOCX. File size limit: 25 MB."
- **In-class digital:** "Take in Canvas during class. Lock down browser may be enabled."
- **Project presentation:** "Present in class on the scheduled day. Upload slides to the linked Canvas Assignment by 8 AM that morning."

#### `time`

For tests/quizzes:

```html
<p><strong>Standard time:</strong> 45 minutes.</p>
<p><strong>Extended time:</strong> 60 minutes for students with accommodations.</p>
```

For projects, include the timeline and checkpoints:

```html
<p><strong>Duration:</strong> 3 weeks (March 4 – March 25)</p>
<p><strong>Milestones:</strong></p>
<ul>
  <li><strong>March 11:</strong> Design proposal due (ungraded check-in)</li>
  <li><strong>March 18:</strong> Working prototype due (in-class feedback)</li>
  <li><strong>March 25:</strong> Final submission</li>
</ul>
```

#### `ai_use`

This is the one slot where you should *not* use a generic default. The right AI policy depends on the assessment type. Defaults to start from (teacher should review and adjust):

**Closed-book test / quiz:**
```html
<p>Acceptable:</p>
<ul>
  <li>None. This is a closed-book individual assessment.</li>
</ul>
<p>Unacceptable:</p>
<ul>
  <li>All AI tools (ChatGPT, Claude, Copilot, etc.)</li>
  <li>Other people's notes or work</li>
  <li>Reference materials not explicitly permitted</li>
</ul>
```

**Take-home essay / writing:**
```html
<p>Acceptable:</p>
<ul>
  <li>Using AI for brainstorming and ideation before you write</li>
  <li>Using AI to research background topics</li>
  <li>Asking AI to explain a concept you don't understand</li>
</ul>
<p>Unacceptable:</p>
<ul>
  <li>Using AI to draft any part of your final submission</li>
  <li>Using AI to revise or edit your draft</li>
  <li>Submitting AI-generated text as your own</li>
</ul>
```

**Project (technical/creative):**
```html
<p>Acceptable:</p>
<ul>
  <li>Using AI to debug code or troubleshoot technical problems</li>
  <li>Using AI to research design references and material options</li>
  <li>Using AI to brainstorm ideas before you commit to a direction</li>
</ul>
<p>Unacceptable:</p>
<ul>
  <li>Using AI to generate the final design / writeup / artifact</li>
  <li>Using AI to write your reflection or process documentation</li>
  <li>Submitting AI-generated work without disclosure</li>
</ul>
```

**Presentation / performance:**
```html
<p>Acceptable:</p>
<ul>
  <li>Using AI for research and preparation</li>
  <li>Using AI for slide design assistance</li>
  <li>Using AI to practice answering anticipated questions</li>
</ul>
<p>Unacceptable:</p>
<ul>
  <li>Reading AI-generated content during the presentation</li>
  <li>Submitting AI-generated slides without disclosure</li>
</ul>
```

Show the teacher the default and explicitly say *"This is the default for [type] — review and adjust to match your class's policy."*

### 4. Grade boundaries — how to compute the table

Standard US/Franklin grade scale (use this unless the teacher overrides):

| Grade | Percentage |
|---|---|
| A+ | 97–100% |
| A  | 93–96%  |
| A- | 90–92%  |
| B+ | 87–89%  |
| B  | 83–86%  |
| B- | 80–82%  |
| C+ | 77–79%  |
| C  | 73–76%  |
| C- | 70–72%  |
| D  | 60–69%  |
| F  | 0–59%   |

To compute point ranges, multiply percentages by total points and round to integers:

- For each grade, the **low bound** is `ceil(low_pct × total / 100)`
- The **high bound** is `floor(high_pct × total / 100)`
- Where low and high collapse to a single value (small point totals), show that one number; otherwise show a range

**Example: 20-point test**
- A+: ceil(0.97 × 20) = 20 (just one value, 20)
- A: ceil(0.93 × 20) = 19, floor(0.96 × 20) = 19 → just 19
- A-: ceil(0.90 × 20) = 18, floor(0.92 × 20) = 18 → just 18
- B+: ceil(0.87 × 20) = 18 (collides with A-) — use ranges that prevent overlap by treating each upper percentage as < the next grade's low percentage. So B+: 17 (since 87.5–89.5% → 17 is the floor; A- starts at 18)
- ...

Frankly, for small totals (<25 points), give the teacher the table but flag that the integer-rounding makes some grades collapse. Suggest they bump to a larger point total OR use percentage thresholds in the description.

**Example: 100-point assessment** — percentages map 1:1, no rounding issues:

| Grade | Points |
|---|---|
| A+ | 97–100 |
| A  | 93–96  |
| A- | 90–92  |
| ... | ... |
| F  | 0–59   |

If the teacher wants a different scale ("we use 90/80/70/60 for A/B/C/D"), use that. Don't over-engineer — the table is a quick reference, not a contract.

### 5. Consider competency alignment

Call `list_competencies`. If a framework is configured (Franklin's TD Competencies, or another school's framework), **always** spend a moment identifying which 1–3 competencies this assessment naturally evaluates. Assessments almost always target specific competencies — even a closed-book MC test exercises Knowledge-Based Reasoning; a project usually targets two or three.

This is a *thinking* step that informs your slot content (especially `description` and `structure_and_grading`). A watershed test that asks students to identify components of a watershed and trace water movement is exercising Knowledge-Based Reasoning (recall + application) and Systems Thinking (interconnections). Knowing this, you'd phrase the `description` slot's "students will" list with verbs that match those competencies and design grade boundaries that reflect mastery of those skills.

Pick 1–3 competencies, not all of them. Don't shoehorn — only call out competencies the assessment genuinely measures.

In step 6 (preview), you'll surface the suggested alignment to the teacher and ask whether to call them out explicitly in the `description` slot. If the teacher already told you upfront which competency they're targeting ("this is a Knowledge-Based Reasoning assessment"), treat that as authoritative — don't override their choice, just confirm it during the preview.

If `list_competencies` returns `configured: false`, skip this step entirely.

### 6. Preview before creating the draft

Show the teacher an outline of what each slot will contain, plus the suggested competency alignment if you identified one in step 5:

```
Assessment page draft — "Watershed Unit Test"
Type: in-class MC test • 20 points • 15% of trimester • 45 minutes

• Description: 20-question MC test on watershed geography
• Pre-Work: 4-item review checklist (lesson pages, worksheet, textbook, practice quiz)
• Structure & Grading: 20 × 1pt, no partial credit; 15% weight; grade boundaries table (A+ = 20, A = 19, ..., F = 0-9)
• Submission: In-class on paper, hand in to teacher
• Time: 45 min standard, 60 min extended
• AI Use: Default policy for closed-book test (no AI tools allowed)

Suggested competencies this assessment evaluates: Knowledge-Based Reasoning + Systems Thinking.
Want me to call these out explicitly in the description, or keep them implicit?

Ready to create this as a draft in Canvas?
```

Wait for approval or revision requests. If the teacher asks for changes, iterate on the affected slots — don't regenerate everything.

If the teacher wants competencies called out explicitly, add a paragraph to the `description` slot referencing them (e.g., `<p>This assessment evaluates <strong>Knowledge-Based Reasoning</strong> (recall + application) and <strong>Systems Thinking</strong> (the watershed cycle questions).</p>`). If they want them implicit, the description stays as you generated it — the competency lens already shaped it.

If no competency framework was configured (step 5 was skipped), omit the "Suggested competencies" line from the preview.

### 7. Create the draft in Canvas

```
create_page(
  course_identifier: "<course code or id>",
  title: "<assessment title>",
  template: "assessment",
  slots: {
    description: "...",
    pre_work: "...",
    structure_and_grading: "...",
    submission: "...",
    time: "...",
    ai_use: "..."
  }
)
```

The page is created as a draft (`published: false` — enforced by the MCP). Confirm to the teacher with the page URL:

> Created draft assessment page **Watershed Unit Test** in DSGN 9. It's saved as a draft — open it in Canvas at <Canvas link>, review, and click **Publish** when you're ready.

**Never tell the teacher you "published" the page.** It's a draft until they publish it manually in Canvas.

### 8. Module placement (only if asked)

Same as `plan-lesson`: if the teacher said "add it after the watershed lesson," call `list_modules(course_identifier, include_items: true)`, find the right module + position, then `add_module_item(type: "Page", content_id: "<the-page-slug>", position: <n>)`.

## Updating an existing assessment

Use `edit_page_content` with the same template machinery — not `create_page`:

```
edit_page_content(
  course_identifier: ...,
  page_url: "<existing-slug>",
  template: "assessment",
  slots: { ...all 6 slots, with the changes... }
)
```

You must provide all 6 slots in the rebuild — the MCP doesn't preserve previous slot content from the on-Canvas HTML. If you have the content from earlier in the conversation, use it. If not, call `get_page_content` first and reconstruct from what's visible, OR ask the teacher what should stay vs. change.

## Linking from a lesson page

If the teacher asks to link this assessment from a lesson page (e.g., "add it as the related assessment on the watershed lesson"), use `edit_page_content` on the lesson:

```
edit_page_content(
  course_identifier: ...,
  page_url: "<lesson-slug>",
  template: "lesson",
  include_sections: ["assessment"],   // turn on the assessment section if not already
  slots: {
    ...all lesson slots...,
    assessment: '<p>See the <a href="<canvas-host>/courses/<id>/pages/<assessment-slug>">Watershed Unit Test</a>, due Friday.</p>'
  }
)
```

Same caveat: you must provide all the lesson's slots in the rebuild.

## Common mistakes to avoid

- **Don't use `create_page` for updates.** Use `edit_page_content` — same template machinery, no duplicate page.
- **Don't generate grade boundaries with rounding bugs.** Verify each grade has a sensible point range for the total. For small point totals where percentages collapse, either bump to a bigger total or use percentage thresholds explicitly.
- **Don't use a generic AI policy.** Match it to the assessment type. A take-home essay has very different AI rules than a closed-book test.
- **Don't pad pre-work with generic items.** If you don't know what the unit covered, ask. "Review your notes" is not pre-work.
- **Don't make up Canvas URLs.** When linking lessons / assignments / rubrics, use the actual URLs from `list_pages` / `list_assignments` / `list_modules` — not constructed paths.
- **Never publish.** The MCP forces `published: false`; don't try to flip it, don't suggest you have, and don't tell the teacher you "published" anything. Teachers publish manually in Canvas after reviewing the draft.
