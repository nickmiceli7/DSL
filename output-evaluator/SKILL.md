---
name: output-evaluator
description: >
  Evaluate the quality of any DSL research deliverable. Use this skill whenever a researcher
  asks for feedback, a quality check, a review, or a critique on any artifact or deliverable —
  including memos, prompt templates, slide decks, guidebooks, project documentation, knowledge
  repository entries, bi-weekly updates, or Claude skills. Trigger on phrases like "evaluate
  this", "review my draft", "how good is this", "does this pass", "check this against the
  standard", "give me feedback on", "critique this", or when a researcher shares a file or
  output and asks whether it's ready. Also trigger mid-session when a skill has just produced
  an artifact and the researcher wants to verify quality before finalizing. Always run an
  intake check before evaluating — but skip questions already answered in the current
  conversation.
---

Last modified: 2026-08-27

# Output Evaluator

Critically evaluates DSL research deliverables against three standards: the researcher's
stated intent, the audience fit, and the artifact-class formatting and content standard.
Returns an inline critique and a single overall **PASS** or **FAIL** verdict.

---

## Overview

The evaluator has four stages:

1. **Locate context** — find the index and any skill-specific standard for the artifact being
   evaluated
2. **Intake** — confirm what's needed before evaluating (skip anything already in context)
3. **Evaluate** — assess against all three standards
4. **Report** — deliver an inline critique with a single verdict

---

## Stage 1 — Locate Context

Before asking anything or evaluating anything, locate the reference materials.

### 1a. Index

The project index lives at `/mnt/project/index.md`. Attempt to read it with `view`.

- **If found:** read it and use it to locate any skill-specific reference files needed
  for the artifact class being evaluated.
- **If not found:** flag `[INDEX NOT VERIFIED]` in the report footer and proceed using
  any skill SKILL.md files available in `/mnt/skills/`.
- **If still not resolved:** check whether a OneDrive/SharePoint connector is active
  (tool list or connected MCPs). If yes, search for `index.md` or the relevant skill
  reference there. If found, note `[INDEX SOURCE: OneDrive]` in the footer.

### 1b. Skill-specific standard

For the artifact class identified in intake (Stage 2), locate the corresponding SKILL.md.
Use the index to find it if available. Fall back to `/mnt/skills/` directory scan if not.
If no SKILL.md exists for the class, note `[NO SKILL STANDARD FOUND — evaluating against
general professional standards only]` in the report footer.

---

## Stage 2 — Intake

Check the current conversation for already-answered context before asking anything.
Extract from conversation history:

- **Artifact class** — what type of deliverable is this?
- **Intent** — what was this artifact meant to accomplish?
- **Audience** — who will read or use it?
- **Draft stage** — is this a first draft, a revision, a near-final?
- **Origin** — was this produced by a DSL skill, or brought in fresh?

For each item not already present in the conversation, ask the researcher. Ask all
missing items at once (not one at a time) to minimize friction. If all five are already
clear from context, skip the intake entirely and proceed to evaluation.

Example prompt when context is missing:

> "Before I evaluate, I need a few quick things — answer whichever aren't already obvious:
> 1. What type of artifact is this? (e.g., memo, prompt template, slide deck)
> 2. What was it meant to accomplish?
> 3. Who's the audience?
> 4. Draft stage — first pass, revision, or near-final?
> 5. Was this produced by a DSL skill or brought in externally?"

---

## Stage 3 — Evaluate

Evaluate the artifact against **three standards in order**. Each informs the verdict.

### Standard A — Intent fit

Does the artifact accomplish what it was meant to do?

- Identify the core purpose stated in intake.
- Check whether the artifact's content, structure, and emphasis actually serve that purpose.
- Flag any significant misalignment between stated intent and what the artifact delivers.
- Note gaps: things the researcher said it should do that it does not.

### Standard B — Audience fit

Is this appropriate for the stated audience?

- Assess language register (too technical, too plain, appropriate).
- Assess assumed knowledge level.
- Assess tone (formal/collegial/casual — does it match what the audience would expect?).
- Flag anything that would confuse, alienate, or underserve the audience.

### Standard C — Artifact-class standard

Does this meet the formatting and content standard for its class?

- Use the class-specific SKILL.md (located in Stage 1b) as the primary standard.
- Check: required sections present, formatting rules followed, iteration profile expectations
  met (e.g., validation pass completed if required), and any class-specific conventions.
- Flag specific deviations by name (e.g., "Missing Distribution section", "Ref line not bold",
  "No validation pass documented").

### Verdict logic

| Condition | Verdict |
|---|---|
| All three standards substantially met | **PASS** |
| Minor gaps only (easily corrected, no missing required elements) | **PASS** with notes |
| Any required element missing, significant intent/audience mismatch, or major formatting deviation | **FAIL** |

A **PASS with notes** is still a pass. Make this distinction explicit so researchers
understand the bar was met even if improvements exist.

---

## Stage 4 — Report

Deliver the critique inline (not as a file, unless the researcher asks for a saved report).

### Report structure

```
## Output Evaluation — [Artifact Name or Type]

**Draft stage:** [as stated]
**Evaluated against:** [skill standard name | general professional standards]

### Intent fit
[2–4 sentences. What it does well, what it misses. Be specific.]

### Audience fit
[2–4 sentences. Register, tone, assumed knowledge. Be specific.]

### Artifact-class standard
[Bulleted list of specific findings — what passes, what deviates. Name the rule, not just the symptom.]

---

**Verdict: PASS / PASS with notes / FAIL**

[One sentence explaining the verdict anchor — the most important factor that determined it.]

---
*Reference standard: [file path or source used]*
*[INDEX NOT VERIFIED] / [INDEX SOURCE: OneDrive] / [NO SKILL STANDARD FOUND] — if applicable*
```

### Tone calibration

- **Critically honest** — do not soften a FAIL into a PASS to spare feelings.
- **Actionable** — every criticism should name the specific problem, not just the
  symptom. "The Ref line is not bold" is better than "formatting issues were found."
- **Not punishing** — a FAIL means specific things need fixing, not that the work is
  bad. Frame failures as a checklist, not a judgment.
- **Proportionate** — a first draft is held to a lower bar than a near-final. Note
  draft stage in the verdict framing if it affected the call.

---

## Edge cases

- **No SKILL.md exists for the artifact class.** Treat as "Other / custom". Use the closest
  analogous skill standard, note the assumption, evaluate against it with a wider tolerance.
  Flag in footer.
- **No artifact provided, only a description.** Ask the researcher to share the actual
  artifact or its content before evaluating. Do not evaluate from description alone.
- **Artifact produced mid-session by another skill.** It is already in context — no upload
  needed. Confirm the file path or content reference and proceed.
- **Researcher disputes a FAIL.** Do not reverse the verdict without a specific argument
  that maps to a named standard. If they provide one, re-evaluate that dimension only and
  update the verdict if warranted. Note the revision in the footer.
- **Researcher asks for a saved report.** Save to
  `/mnt/user-data/outputs/eval-report-<artifact-name>-<date>.md` and use `present_files`.

---

## Reference files

- `/mnt/project/index.md` — project index (primary navigation to all skill and reference files)
- Skill-specific SKILL.md files — located via index or `/mnt/skills/` scan

---

## Provenance note

This version (2026-08-27) removes the prior dependency on `/mnt/project/artifact-classes.md`,
which was found not to exist in the project or on SharePoint. Standard C now evaluates solely
against the class-specific SKILL.md; when no SKILL.md exists for the artifact class, the
evaluator notes this explicitly and evaluates against general professional standards with a
wider tolerance rather than a project-defined baseline.
