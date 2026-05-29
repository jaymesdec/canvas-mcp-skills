---
name: plan-lesson
description: Plan a single class lesson using Understanding by Design (UbD) — backwards design from learning targets → evidence of understanding → learning activities. Use when the teacher says "plan a lesson", "design a lesson", "help me think through a class", "what should I teach for X", or wants pedagogical scaffolding before deciding what students will do. Produces a conversational UbD brief (Stages 1–3 scaled for one class session) and offers handoff to `post-lesson-page` when ready to materialize on Canvas. Does NOT create any Canvas artifacts itself.
---

# Plan Lesson

Think a single class session through using **Understanding by Design (UbD)** — start with what students should understand, design how you'll know they got there, then build the activities that get them there. Lesson-scaled: 1–2 learning targets, one formative check, one class arc. Produces a UbD brief in chat; offers handoff to `post-lesson-page` when the teacher's ready to materialize it on Canvas.

> **This skill writes nothing to Canvas.** It's a pedagogical thinking partner. Posting is a separate skill (`post-lesson-page`) the teacher can run once the brief is locked.

## When to use

A teacher is figuring out what to teach for a single class session and wants a framework. Common phrasings:

* "Plan a lesson on the water cycle for DSGN 9 — help me think through it"

* "I'm teaching prototyping tomorrow, not sure how to structure it"

* "Design a lesson on parallel circuits using UbD"

* "What's a good way to frame this watershed lesson?"

* "Help me think backwards from what I want students to understand"

## When NOT to use

* **The teacher already knows what they're teaching and just wants it on Canvas.** That's `post-lesson-page`. Don't make them think backwards if they don't want to.

* **Multi-class units or modules.** That's `plan-module` (which has its own UbD path scaled to multi-lesson arcs).

* **Assessments / tests / projects.** That's `plan-assessment` for the page, `post-quiz` for the actual Canvas Quiz object.

* **Whole-course scaffolding.** That's `plan-course`.

## Prerequisites

* canvas-mcp installed and configured *only if* the teacher wants to pull module context (existing essential questions, enduring understandings) or compete with the school's competency framework. The skill works without canvas-mcp for pure ideation.

* No Canvas writes happen in this skill. No `published: false` worry, no draft creation — this is a chat-only working document.

## Workflow

### 1. Establish context

Confirm with the teacher (asking only for what's missing):

* **Lesson topic / focus.** What's the lesson about, in one phrase. If they gave a working title, use it; otherwise propose one.

* **Class duration.** 45 minutes? 75? 90? This shapes Stage 3 timing. If they don't say and you can't infer, ask.

* **Where does this lesson sit?** Standalone, or part of an existing Canvas module / unit plan? If part of a module, optionally call `list_modules(course_identifier)` + `get_page_content` on the module's outcomes page to pull the module's enduring understandings and essential questions — the lesson should advance those, not invent new ones.

* **Which Canvas course?** Only needed if you'll pull module context or call `list_competencies`. If the teacher is purely ideating without a course in mind, skip this — you don't need a course to think about a lesson.

Don't over-confirm. If the teacher said "plan a 60-min lesson on prototyping for DSGN 9," you have topic, duration, and course — move on.

### 2. Stage 1 — Desired Results

For a single lesson, scope is smaller than a module. Identify:

**Learning targets** (1–2 statements, SWBAT form — "Students will be able to…"). Specific, observable, achievable in one class. Distinguish for the teacher:

| Not this                            | But this                                                                     |
| ----------------------------------- | ---------------------------------------------------------------------------- |
| Topic: "watersheds"                 | Target: "SWBAT diagram the path of water through a local watershed"          |
| Fact: "watersheds drain to oceans"  | Target: "SWBAT explain why endorheic basins don't drain to oceans"           |
| Activity: "build a watershed model" | Target: "SWBAT predict how land use changes affect downstream water quality" |

**Essential question** (1, sometimes 2). The open-ended question students wrestle with during this class. Often borrowed from the module's essential questions — pick the one most relevant to today's targets. If standalone (no module), draft one.

**Enduring understanding(s)** (1, sometimes 2). The big idea this lesson builds toward. Almost always borrowed from a parent module if there is one; if standalone, draft one but don't overreach — a single lesson advances an EU, it doesn't deliver it.

**Competency alignment.** Call `list_competencies` if a course is set and the school config has one. Pick **1–2 competencies** this lesson naturally exercises through the targets + planned activity. Don't shoehorn — a vocab-memorization lesson doesn't target Systems Thinking just because the topic is ecosystems.

**For conventional content** (Algebra topics, US History units, standard Biology, world languages): draft these for the teacher to react to. Claude has reasonable pattern-matching here.

**For unconventional or teacher-designed content** (transdisciplinary projects, school-specific curriculum, novel framings): don't assume. Share initial ideas as conversation starters, let the teacher shape direction.

**Stage 1 checkpoint** — show the teacher:

```
## Stage 1: Desired Results

**Learning targets** (SWBAT):
1. [...]
2. [...]

**Essential question:** [...]

**Enduring understanding:** [...]

**Competency focus:** [Knowledge-Based Reasoning, Systems Thinking, ...]
```

Wait for the teacher to confirm or revise before moving to Stage 2. This is the foundation — getting it right matters more than speed.

### 3. Stage 2 — Evidence of Understanding

At the lesson scale, design **one** formative check that tells the teacher whether students hit the targets. The check should:

* **Match the targets.** If the target is "explain why endorheic basins don't drain to oceans," the check needs students producing an explanation — not multiple choice.

* **Fit inside the class period.** 3–10 minutes typically. Exit tickets, mini-demos, quick-writes, peer explanations, artifact checks, observation rubrics.

* **Be specific.** "I'll see if they got it" is not a formative check. "I'll collect a 2-sentence written response to *'Why does the Great Salt Lake have no outflow?'* and look for mention of evaporation + closed-basin geography" is.

Format the formative check:

* **What it is** (exit ticket / quick-write / observed demo / mini-artifact / discussion prompt)

* **When in the lesson** (last 5 min? mid-class transition? continuous observation?)

* **Success signal** (what a student who hit the target produces vs. one who didn't)

If the lesson feeds a graded summative arc (unit test next week, project due Friday), note the connection but don't design the summative here — that's `plan-assessment`'s job.

**Stage 2 checkpoint** — show the teacher:

```
## Stage 2: Evidence of Understanding

**Formative check:**
- Type: [exit ticket / quick-write / ...]
- When: [end of class / mid-class / ...]
- Prompt or task: [...]
- Success signal: [what a student who hit the target produces]
- Miss signal: [what a struggling student produces]

**Connection to summative** (if any): [...]
```

Wait for confirmation before Stage 3. If the formative check feels mismatched to the targets, push back — that's the most common UbD failure mode at the lesson scale.

### 4. Stage 3 — Learning Plan

Plan the arc of the class session — phase-by-phase activities that get students from where they walked in to where the targets land. Match the rough total to the duration confirmed in step 1.

Format as a timed sequence:

```
0:00–0:05  Hook: [what gets students into the question]
0:05–0:15  Activator: [activate prior knowledge / surface misconceptions]
0:15–0:35  Main investigation: [the heart of the lesson]
0:35–0:50  Consolidation: [synthesis, discussion, structured talk]
0:50–0:55  Formative check (from Stage 2)
0:55–1:00  Closing / preview next class
```

Per phase, capture:

* **Name + minutes** (so total adds up to the class length)

* **What students are doing** (active verbs — investigating, comparing, defending, building, sketching, arguing)

* **What the teacher is doing** (modeling, observing, questioning, scaffolding — not always lecturing)

* **Materials / prompts / links** (only if known — leave TBD otherwise)

**Sequencing principles:**

* Hook → exploration → consolidation → check → close. UbD prefers students encountering the question *before* receiving the answer.

* Active beats passive. If a phase has students "listening to the teacher explain X," ask whether they could discover X instead.

* Less direct instruction than feels natural. 10–15 min of teacher talk per 60 min is plenty for most secondary classes.

* Make space for student talk + iteration — turn-and-talks, peer feedback, draft-revise cycles.

* Don't pack the schedule. Buffer time matters; ambitious pacing that requires perfection will fail.

* The formative check from Stage 2 has a slot in the timeline — show it explicitly.

**Stage 3 checkpoint** — show the teacher the timed sequence. Wait for confirmation.

### 5. Compose the full UbD brief

Once Stages 1–3 are locked, present a single composed brief:

```
# Lesson Plan — [Title]
*[Course] · [Date or Week] · [Duration] min*

## Stage 1: Desired Results
**Learning targets (SWBAT):**
1. [...]
2. [...]

**Essential question:** [...]

**Enduring understanding:** [...]

**Competency focus:** [...]

## Stage 2: Evidence of Understanding
**Formative check:** [type] — [prompt or task]
- When: [...]
- Success signal: [...]

## Stage 3: Learning Plan
| Time | Phase | Students do | Teacher does |
|---|---|---|---|
| 0:00–0:05 | Hook | [...] | [...] |
| ... | ... | ... | ... |
```

This is the working document the teacher walks away with — copy-pastable into their own notes, or fed directly into `post-lesson-page`.

### 6. Offer the handoff

Ask the teacher:

> Want me to post this to Canvas as a draft lesson page? I'll hand the brief to `post-lesson-page` and it'll fill the school's lesson template using what we just designed — targets land in the "to" slot, the formative goes in the assessment slot, the learning plan phases land in tasks, etc.

* **Yes** → invoke the `post-lesson-page` workflow, passing the brief as established context. `post-lesson-page` should skip its own Stage-1-style thinking step (everything's done) and go straight to template slot rendering. The competency alignment and module context already exist.

* **No, leave it in chat** → confirm the brief stays where it is. The teacher can copy it, iterate later, or come back.

* **Save it somewhere** → this skill doesn't write files. If they want a saved artifact, they can copy the brief into their own notes/Drive/wherever. Don't volunteer to write a file the teacher didn't ask for.

## Common mistakes to avoid

* **Don't draft EUs and EQs from scratch when the lesson belongs to an existing module.** Borrow from the module's outcomes page. A lesson advances a module-level EU; it doesn't reinvent one per class.

* **Don't over-scale Stage 2.** One sharp formative check beats a four-part assessment rubric at the lesson level. If you're tempted to add a summative here, that's `plan-assessment`'s territory.

* **Don't write activities (Stage 3) before targets and evidence (Stages 1–2).** That's the whole point of backwards design. If the teacher resists and just wants to brainstorm activities, push back gently once — "We'll plan the activities, but we'll get there faster if we know what students should be able to do by the end" — then yield if they insist. Don't be a UbD purist if the teacher needs a quick scaffold.

* **Don't pack Stage 3.** A 60-minute lesson with 8 phases of 7.5 minutes each is brittle. Leave breathing room.

* **Don't produce Canvas HTML in this skill.** This is a brief, not a page. `post-lesson-page` handles template-specific HTML output.

* **Don't skip checkpoints.** Stage 1 must be confirmed before Stage 2; Stage 2 before Stage 3. UbD silently fails when stages misalign — the checkpoint cadence is the safeguard.

* **Don't shoehorn competencies.** If the lesson doesn't naturally exercise a competency, don't list it. Saying every lesson hits all 9 makes the framework meaningless.

* **Don't fabricate module / course context.** If the teacher didn't tell you what module this belongs to, ask — don't infer one and produce a brief that misaligns with a real Canvas unit plan.