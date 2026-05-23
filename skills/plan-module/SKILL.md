---
name: plan-module
description: Plan an entire Canvas module — module outcomes page, a sequence of lesson pages, and at least one summative assessment — then create the Canvas Module itself (unpublished) with all items added in order. Use when the teacher says "plan a module", "create a module", "design a module", "draft a unit", or describes a multi-lesson stretch of curriculum (often spanning several weeks). Enforces the school's module naming convention. Generic — works with whatever templates and competency framework the school config provides.
---

# Plan Module

Plan a complete Canvas module: the Module Outcomes page, the lessons inside it, the summative assessment at the end, and the Canvas Module container that holds them all. All drafts. Teacher publishes manually.

> **This skill never publishes anything.** Every page lands in Canvas as a draft. The Canvas Module itself is created in Canvas's default unpublished state. The teacher reviews everything in Canvas and clicks Publish themselves when ready.

## When to use

A teacher describes a multi-lesson stretch of curriculum they want to scaffold in Canvas. Common phrasings:

- "Plan a module on vibe coding — Weeks 24–36, course DSGN 9"
- "Design Module 4 for FSV 117 — pitch development, 4 weeks"
- "Draft a watershed unit, 6 lessons + a test"
- "I need to set up the next module — AI tools for designers, 8 weeks"

## When NOT to use

- **Single lesson** — that's `plan-lesson`.
- **Single assessment** — that's `plan-assessment`.
- **Adding to an existing module** — call `list_modules`, then `add_module_item` directly (or use `plan-lesson` / `plan-assessment` and then add the resulting page).

## Prerequisites

- canvas-mcp **v0.3.12 or later** (older versions don't have `create_module`).
- A school config with `lesson`, `assessment`, and `module_outcomes` page templates (Franklin: bundled by default).

## Workflow

### 1. Capture the module metadata

These pieces are required to build the module. Get them from the teacher's initial request; ask only for what's missing:

- **Canvas course** (code or numeric id — confirm via `list_courses` if there's any ambiguity)
- **Module number** (integer; e.g., 4)
- **Module short title** (the marketing name; e.g., "Vibe Coding")
- **Module subtitle** (the elaboration; e.g., "Transforming Text into Functional Software with AI")
- **Week range** (start–end integers, e.g., 24–36)
- **Topic / scope** (what the module is actually about — free-form from the teacher)
- **Approximate number of lessons** (the teacher tells you, or you propose based on week count; see "Default lesson count" below)

Build the **module name** to the exact Franklin convention:

```
Module {number}: {short_title}: {subtitle} (Weeks {start}-{end})
```

Example: `Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)`

Build the **Module Outcomes page title**:

```
Module {number}: {short_title} - Outcomes
```

Example: `Module 4: Vibe Coding - Outcomes`

Do NOT improvise these names — the formatting is part of how the school's Canvas pages are organized.

### Default lesson count

If the teacher didn't specify the number of lessons, propose based on the week range:

- 1–2 weeks: 2–3 lessons
- 3–4 weeks: 4–6 lessons
- 5–8 weeks: 6–10 lessons
- 9+ weeks: ~1 lesson per week, but ask the teacher

Always confirm with the teacher before committing — different teachers run modules at different cadences.

### 2. Identify module-level competencies (when a framework is configured)

Call `list_competencies`. If a framework is loaded (Franklin's TD Competencies, or another school's), spend a moment identifying which **2–4 competencies** this module targets. Modules span more than one competency — they're broader than a single lesson.

Hold these in mind as you draft outcomes and lesson titles. Surface them in the preview (step 6) and ask the teacher whether to mention competencies explicitly in the Module Outcomes page.

If no framework is configured, skip this step.

### 3. Propose the lesson sequence

Generate a list of lesson titles + 1-sentence summaries that flow through the module's scope. Show this to the teacher BEFORE drafting any pages:

```
Proposed lesson sequence for "Module 4: Vibe Coding" (Weeks 24-36):

 1. Introduction to AI Coding Assistants — what they are, how they work
 2. Prompt Engineering Basics for Code — clarity, context, iteration
 3. Building Your First AI-Assisted Function — a simple project
 4. Reading & Debugging AI-Generated Code — critical evaluation
 ...
13. Module Reflection & Showcase — students present their work

Plus a summative assessment at the end (see step 4).

Sound right? Adjust titles, drop lessons, or add more before I draft anything.
```

Iterate with the teacher until they're happy. Don't move on until the sequence is locked.

### 4. Propose the summative assessment

Ask the teacher about the culminating assessment for the module:

- **Type** — project / test / presentation / portfolio / essay / etc.
- **Total points** — for the grade boundaries table
- **Weighting** — % of trimester/term
- **Time/duration** — for tests this is minutes; for projects this is the timeline

Propose if the teacher doesn't have specifics yet. The default for a multi-week module is usually a project or portfolio (rarely a sit-down test).

### 5. Draft the Module Outcomes

Generate 4–8 outcome statements for the module — what students will be able to do BY THE END of the entire module. Each outcome should:

- Start with an action verb (Design, Build, Evaluate, Analyze, Apply, etc.)
- Be observable / measurable (not "students will understand X" — say what they'll DO)
- Map to one of the identified competencies where applicable
- Reflect the scope of the WHOLE module, not just one lesson

Show in the preview (step 6):

```
Module Outcomes draft (4-8 items, each <li>):
  1. Design and prompt-engineer AI assistants to generate working code
  2. Evaluate AI-generated code for correctness, security, and clarity
  3. Debug code by collaborating iteratively with an AI assistant
  4. Build a working software artifact using primarily AI-assisted development
  5. Reflect on the ethics and limitations of AI-generated code

Suggested competency focus: Knowledge-Based Reasoning + Systems Thinking + Adaptability.
Want me to mention these competencies in the outcomes page, or keep the alignment implicit?
```

### 6. Preview the full module structure

Before any tool calls touch Canvas, show the teacher the complete plan:

```
Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)
in course DSGN 9 (id: 60366)

[Module Outcomes page]
  • 5 outcomes (as listed above)
  • Competency focus: Knowledge-Based Reasoning, Systems Thinking, Adaptability

[13 lesson pages]
  1. Introduction to AI Coding Assistants
  2. Prompt Engineering Basics for Code
  ... (13 total)

[1 assessment page]
  • Title: Module 4 Final Project: Build with AI
  • Type: project, 50 points, 25% of trimester, 4-week timeline

[Canvas Module]
  • Name: Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)
  • Position: <next available>
  • Items in order: Outcomes → Lesson 1 → ... → Lesson 13 → Final Project
  • Status: UNPUBLISHED (teacher publishes manually)

How detailed should the lesson and assessment pages be?
  • Outline (default) — title + 1-sentence "about" stub per page
  • Stub — title only, all slots empty
  • Full — every slot filled now (slow, but complete)
```

Get explicit approval. If the teacher wants changes, iterate. Don't proceed to step 7 without "yes, go ahead."

### 7. Create everything as drafts

In this order, calling MCP tools:

#### 7a. Create the Canvas Module

```
create_module(
  course_identifier: <course>,
  name: "Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)"
)
```

The module is created with `workflow_state: "unpublished"`. Capture the returned `module.id` — you'll need it for `add_module_item` calls.

#### 7b. Create the Module Outcomes page

```
create_page(
  course_identifier: <course>,
  title: "Module 4: Vibe Coding - Outcomes",
  template: "module_outcomes",
  slots: {
    outcomes: "<li>Design and prompt-engineer AI assistants...</li><li>...</li>..."
  }
)
```

Note: the outcomes slot is the inner `<li>` items — the template wraps them in the `<ol>`. Generate clean `<li>` content; don't include the surrounding `<ol>` tags (the template adds them).

Capture the returned `url` (the page slug, e.g., `module-4-vibe-coding-outcomes`).

#### 7c. Create lesson pages

For each lesson in the sequence, call `create_page` with `template: "lesson"`.

In **Outline mode** (default), generate a 1-sentence summary for the `about` slot, leave all other slots empty:

```
create_page(
  course_identifier: <course>,
  title: "Introduction to AI Coding Assistants",
  template: "lesson",
  slots: {
    about: "<p><em>This lesson introduces students to AI coding assistants (Claude, Copilot, ChatGPT) — what they are, how they work, and the basics of interacting with them.</em></p>",
    to: "",
    concepts: "",
    resources: "",
    tasks: ""
  }
)
```

In **Stub mode**, leave all slots empty (the page will be created with empty accordions; the teacher fills them later via `plan-lesson` or directly in Canvas).

In **Full mode**, walk the `plan-lesson` workflow for each lesson — generate per-slot content, optional sections, competency alignment, the works. This is slow for 13 lessons but produces complete drafts.

Capture each returned `url`. Don't ask the teacher to confirm each lesson individually in Outline/Stub mode — just power through and produce a summary at the end.

#### 7d. Create the assessment page

`create_page` with `template: "assessment"`. In Outline mode, write a short description and a placeholder for the grade boundaries table; leave the other slots empty. In Full mode, walk the full `plan-assessment` workflow.

#### 7e. Add items to the Canvas Module

In order: Module Outcomes page, then each lesson page in sequence, then the assessment page. Use `add_module_item` for each:

```
add_module_item(
  course_identifier: <course>,
  module_id: <returned-from-7a>,
  type: "Page",
  title: "Module 4: Vibe Coding - Outcomes",
  content_id: "<page-slug-from-7b>"
)
```

Note `position` defaults to "append at end" — since you're calling them in order, that's what you want.

### 8. Confirm to the teacher

Surface the result:

> Drafted **Module 4: Vibe Coding: Transforming Text into Functional Software with AI (Weeks 24-36)** in DSGN 9 — 1 Module Outcomes page, 13 lesson pages, 1 final project page, all added to the new module in order. Everything is saved as drafts (module + items both unpublished). Open the module in Canvas at <module URL> to review and publish.
>
> Next steps:
> - Develop each lesson page (use `/plan-lesson` per lesson, or edit directly in Canvas)
> - Develop the final project page (use `/plan-assessment`, or edit directly in Canvas)
> - When the whole module is ready, publish the module + each item from the Canvas UI

## Common mistakes to avoid

- **Never tell the teacher anything is "published."** Module + every page are drafts. Use the word "drafted" or "created as drafts."
- **Don't improvise the module name format.** The convention is exact: `Module N: Short: Subtitle (Weeks X-Y)`. Same for the Module Outcomes page title: `Module N: Short - Outcomes`.
- **Don't draft lessons in Outline mode and silently flip to Full mode** (or vice versa) — confirm the mode in the preview and stick to it.
- **Don't ask the teacher to confirm each lesson individually** during the bulk-creation step. They confirmed the sequence in step 3; just execute and summarize at the end.
- **Don't add an extra `<ol>` wrapper in the `outcomes` slot.** The template provides the `<ol id="kl_objective_list">`; the slot is just the `<li>` items.
- **Don't make up Canvas URLs.** Use the `url` / `html_url` fields returned by `create_page` / `create_module`, not constructed paths.
- **Don't run `plan-module` on a module that already exists.** It will create a duplicate. Check via `list_modules` first if there's any chance the module already exists.
