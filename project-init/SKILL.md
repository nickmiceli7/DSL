---
name: project-init
description: >
  Use this skill to initialize a new Claude project for DSL or any analogous research lab.
  Triggers include: "set up this project", "initialize the project", "run init", "/init",
  "configure this project", "start a new project", or any message indicating a new project
  environment is being established for the first time. Also trigger when an admin uploads
  a new set of skill files and asks Claude to configure them for a specific project context.
  Do NOT trigger for researcher onboarding — that is a separate skill. This skill configures
  the project environment; researcher onboarding configures individual researchers within it.
---

Last modified: 2026-07-23

# Project Init

Configures a new Claude project environment for a research lab or team. Gathers project
context through a structured admin interview, executes agentic configuration work, and
returns two handoff documents: an `init-log.md` for the project record and an
`init-procedure.md` for the admin to complete required human steps.

---

## Overview

Three phases:

1. **Discovery (HITL)** — interview the admin; pre-fill from available context first;
   confirm before proceeding
2. **Execution (Agentic)** — configure skill files, generate reference files, produce
   updated outputs; pause only at genuine blockers requiring admin input
3. **Handoff (HITL)** — deliver the init log and admin procedure; admin completes
   human-only steps

---

## Before Starting — Check for Existing Init Log

Before doing anything else, check for an existing `init-log.md` in `/mnt/project/`.

**If found:**
- Read it. Identify any incomplete steps, open decisions, or flags.
- Present a summary to the admin:
  > "I found an existing init log from [date]. It looks like init was previously
  > run but [X steps were incomplete / the following decisions were left open: ...].
  > Would you like to resume from where it left off, or start a full re-init?"
- **Default to resume.** If the admin chooses re-init, require explicit confirmation:
  > "A full re-init will overwrite all locked decisions from the previous run.
  > Type CONFIRM to proceed, or anything else to resume instead."
- Never discard prior context unless the admin explicitly confirms. Treat an unexpected
  re-init request as a likely mistake or incomplete prior session.

**If not found:** proceed directly to Phase 1.

---

## Phase 1 — Discovery

### 1a. Pre-fill from context

Before asking the admin anything, read all available sources and attempt to answer the
discovery question set from context. Sources to check, in order:

1. `/mnt/project/index.md` — project name, file inventory, skill list
2. All other files in `/mnt/project/` — any existing skill files, reference docs, assets
3. Memory — any prior project context stored from previous conversations
4. Any files the admin has uploaded in the current session

For each discovery question, record:
- **Inferred** — answered from context (note the source)
- **Needs confirmation** — partially answerable but uncertain
- **Unknown** — no basis to answer; must ask

### 1b. Discovery question set

These are the standard questions. Only ask what cannot be inferred. Ask all unanswered
questions at once — do not drip them one at a time.

| # | Question | Blocks execution if missing? |
|---|---|---|
| 1 | Project name | Yes |
| 2 | Lab / organization name and institutional affiliation | Yes |
| 3 | Project leads and key personnel (name, role, email) | Yes |
| 4 | Project goal — 2–3 sentence overview | Yes |
| 5 | Pillar or workstream structure (if applicable) | No |
| 6 | Logo — is one available? Where? | No |
| 7 | AI use restrictions or guidelines specific to this project | No |
| 8 | Confidentiality or data sensitivity constraints | No |
| 9 | Client or funder context — is there a client packet? | No |
| 10 | SharePoint — is a SharePoint site being used? If so, which files are relevant to this project and what context do they provide? | No |
| 11 | Key documents the team will reference (titles, locations) | No |
| 12 | Domain-specific background questions to inject into researcher onboarding | No |
| 13 | Domain-specific project familiarity questions and document list for researcher onboarding | No |

Items marked "Blocks execution if missing" must be resolved before Phase 2 begins.
Items not marked may be noted as open decisions and flagged in the init log.

### 1c. Discovery confirmation

After gathering responses, present a consolidated discovery record:

```
## Discovery Record — [Project Name]

**Inferred from context:**
- [item]: [value] (source: [file or memory])
- ...

**Provided by admin:**
- [item]: [value]
- ...

**Open (not provided — will be flagged):**
- [item]
- ...
```

Ask: "Does this look correct? Confirm to proceed with execution, or correct anything above."

Do not begin Phase 2 until the admin confirms.

---

## Phase 2 — Execution

Work through the following checklist in order. Each step is either **completed**,
**skipped with reason**, or **blocked** (triggers a HITL pause).

### Execution checklist

**Step E1 — Scan project skill files**
Read all `*-skill.md` files present in `/mnt/project/`. For each, identify:
- Placeholder hooks that need project-specific injection (e.g., `[PROJECT PLACEHOLDER]`)
- Any references to project name, lab name, personnel, or documents that should be updated
- Any skill-specific question sets for researcher onboarding modules

Also check `/mnt/skills/user/` for any skills installed at the account level that are
not present in `/mnt/project/`. These are available to the admin in their own sessions
but will not be available to other researchers unless uploaded to the project files.
Note any such skills in the init log and flag them in the handoff procedure so the
admin knows to upload them explicitly.

**Step E2 — Inject project context into skill files**
For each skill file identified in E1, produce an updated version with placeholders
replaced by values from the discovery record. Rules:
- Replace all `[PROJECT PLACEHOLDER]` hooks with appropriate project-specific content
- Update any hardcoded references to lab name, project name, or personnel
- Do not remove or restructure skill logic — only fill placeholders and update references
- Output updated files to `/mnt/user-data/outputs/` using the same `*-skill.md` filename
- If a placeholder cannot be filled (value was an open discovery item), leave a
  `[INIT INCOMPLETE — <item name>]` marker in place and flag in the init log

**Step E3 — Generate or update `index.md`**
Using the discovery record and the file inventory from E1:
- If `/mnt/project/index.md` exists: produce an updated version reflecting any new files,
  personnel changes, or SharePoint references added during this init
- If it does not exist: generate a complete `index.md` following the structure of the
  DSL index template (Skills table, Reference Files table, Assets table, SharePoint
  references if applicable)
- Output to `/mnt/user-data/outputs/index.md`

**Step E4 — Generate `artifact-classes.md` if absent**
Check `/mnt/project/artifact-classes.md`. If missing:
- Generate a baseline version using the standard DSL artifact class set as a starting point
- Note any project-specific artifact types suggested by the discovery record
- Output to `/mnt/user-data/outputs/artifact-classes.md`
- Flag in init log: `[GENERATED — admin should review and customize]`

**Step E5 — Generate `energy-model.md` if absent**
Check `/mnt/project/energy-model.md`. If missing:
- Generate a baseline version using the standard DSL energy model
- Output to `/mnt/user-data/outputs/energy-model.md`
- Flag in init log: `[GENERATED — admin should review]`

**Step E6 — Validate outputs**
Review all generated files for internal consistency:
- Do all skill files reference the correct project name and personnel?
- Does `index.md` list all files that will exist after the admin completes the handoff procedure?
- Are all `[INIT INCOMPLETE]` markers accounted for in the init log?

**Step E7 — Reconcile skill files across all three locations**
This is the final execution step. Produce a reconciliation table covering every skill
file across three locations:

1. **SharePoint `Claude Skill Files/`** — the authoritative source. Do NOT use the
   `folderName` filter on `sharepoint_search` or the `sharepoint_folder_search` tool to
   enumerate this folder — both return empty results on the DSLAI site regardless of
   query. This is a known connector limitation, not a permissions issue (see Edge
   Cases). Use this method instead to get a complete, unfiltered listing:
   1. Run any `sharepoint_search` query scoped to the site to obtain a `driveId`
      (any hit on any document works — the `driveId` is the same across the whole site).
   2. Call `read_resource` on `file:///{driveId}/root` to list the drive's top-level
      folders and files, each with its own `file:///{driveId}/{itemId}` URI.
   3. Find `Claude Skill Files` in that listing and note its item URI.
   4. Call `read_resource` on that folder's own URI. This returns every item inside
      it — names, sizes, and URIs — with no search filtering involved.
   `read_resource` on a returned file URI works reliably to confirm a specific file
   exists and to read its content.
2. **Claude project files (`/mnt/project/`)** — what researchers actually have access to
3. **Admin account skills (`/mnt/skills/user/`)** — what the admin can use personally
   but which are NOT automatically available to researchers

For each skill, record presence (✅) or absence (❌) in each location. Flag any mismatch:

- **In SharePoint but not in project** → admin needs to upload it to the project
- **In account but not in project** → admin needs to upload it to the project
- **In project but not in SharePoint** → flag for awareness; may be intentional
- **Missing from SharePoint** → flag; admin should verify it exists there and upload if not

Include this reconciliation table in the init log and in the handoff procedure.
Do not mark init complete if any skill is missing from the project files that is
present in SharePoint or the admin's account — surface it as an action item.

### Conditional HITL during execution

Pause and return to the admin only if a genuine blocker arises — for example:
- A placeholder requires a decision the admin hasn't made (e.g., confidentiality scope
  is ambiguous and affects skill behavior)
- A conflict is found between two sources (e.g., memory says project lead is Person A,
  an uploaded file says Person B)
- A skill file contains logic that cannot be safely updated without admin guidance

When pausing, state clearly:
- What the blocker is
- What the two (or more) options are
- What the default will be if the admin does not respond

After resolution, continue execution from the next incomplete step.

---

## Phase 3 — Handoff

Produce two output files.

### Output 1 — `init-log.md`

Save to `/mnt/user-data/outputs/init-log.md`. This file is intended to be uploaded
to the project by the admin and kept as the permanent project init record.

```markdown
# Init Log — [Project Name]

**Date:** [YYYY-MM-DD]
**Run by:** [admin name if known]
**Status:** [Complete | Incomplete — see open decisions]

---

## Locked Decisions

Decisions confirmed during this init run. Do not change without re-running init
or manually updating this log with a dated note.

| # | Decision | Value | Source |
|---|---|---|---|
| 1 | [e.g., Project name] | [value] | [admin / inferred from file] |
| ... | | | |

---

## Open Decisions

Items not resolved during this init. Each must be resolved before the affected
skill or reference file can be considered complete.

| # | Item | Impact | Notes |
|---|---|---|---|
| 1 | [e.g., AI use restrictions] | [which skill(s) affected] | [any context] |
| ... | | | |

---

## Execution Log

| Step | Status | Notes |
|---|---|---|
| E1 — Scan skill files | [Done / Skipped / Blocked] | [notes] |
| E2 — Inject project context | [Done / Partial / Blocked] | [files updated, any INCOMPLETE markers] |
| E3 — index.md | [Generated / Updated / Skipped] | |
| E4 — artifact-classes.md | [Generated / Already present / Skipped] | |
| E5 — energy-model.md | [Generated / Already present / Skipped] | |
| E6 — Validation | [Passed / Flags — see below] | |

---

## Flags

Anything requiring further review, follow-up, or admin judgment.

- [FLAG]: [description]
- ...

---

## Init Incomplete Markers

If any skill files contain `[INIT INCOMPLETE]` markers, they are listed here.

| File | Marker | Blocking item |
|---|---|---|
| [filename] | [placeholder text] | [open decision #] |
```

### Output 2 — `init-procedure.md`

Save to `/mnt/user-data/outputs/init-procedure.md`. This is the admin's step-by-step
guide to completing everything that requires human action. Written for a nontechnical
but capable admin.

```markdown
# Project Init — Admin Procedure

**Project:** [Project Name]
**Date:** [YYYY-MM-DD]
**Prepared by:** Claude (DSL Project Init Skill)

This document tells you exactly what to do to finish setting up the project in Claude.
Follow steps in order. Each step tells you what to do, where to do it, and what success
looks like.

---

## Before You Start

Make sure you have:
- [ ] Access to the Claude project (claude.ai)
- [ ] Access to the SharePoint site [name and URL if known]
- [ ] The output files Claude produced (listed in Step 1)

---

## Step 1 — Download Claude's Output Files

Claude has produced the following files. Download each one before proceeding.

| File | What it is |
|---|---|
| `index.md` | Updated project index |
| `[skill-name]-skill.md` | Updated skill file (one row per updated skill) |
| `artifact-classes.md` | [If generated] Baseline artifact class definitions |
| `energy-model.md` | [If generated] Baseline energy model |
| `init-log.md` | This project's init record |

---

## Step 2 — Remove Old Files from the Claude Project

In the Claude project files panel:

For each file listed below, click the file, then click **Remove from project**.
Do this before uploading new versions — Claude cannot have two files with the same name.

Files to remove:
- [ ] `index.md` [if updating]
- [ ] `[skill-name]-skill.md` [one line per file being replaced]
- [ ] `artifact-classes.md` [if replacing]
- [ ] `energy-model.md` [if replacing]

**Success looks like:** none of the above filenames appear in the project files panel.

---

## Step 3 — Upload New Files to the Claude Project

In the Claude project files panel, click **Add files** and upload:

- [ ] `index.md`
- [ ] `[skill-name]-skill.md` (repeat for each updated skill)
- [ ] `artifact-classes.md` [if generated]
- [ ] `energy-model.md` [if generated]
- [ ] `init-log.md`

**Success looks like:** all uploaded files appear in the project files panel with
today's date.

---

## Step 4 — Add SharePoint Context [if applicable]

[Include this section only if SharePoint was identified in discovery.]

The following SharePoint files provide context that Claude should be able to reference.
For each file, follow the instructions to make it available.

| File | SharePoint location | How to add |
|---|---|---|
| [filename] | [path or URL] | [upload to project / connect via M365 connector] |

**If using the Microsoft 365 connector:** ensure it is enabled in the Claude project
settings under Tools. Claude can then search SharePoint directly during research sessions.

**If uploading files directly:** download each file from SharePoint and upload it to the
Claude project files panel following the same process as Step 3.

---

## Step 5 — Distribute Skill Files to Researchers

Each researcher needs their own copy of the skill files installed in their personal
Claude projects. Claude cannot do this automatically — you or each researcher must do it.

When a new researcher joins, direct them to run the researcher onboarding skill.
During onboarding, Claude will ask them to upload the skill files. Provide each
researcher with the following files:

[List of *-skill.md files to distribute]

Note: these are the same files you uploaded in Step 3. Researchers receive `.skill`
copies — the onboarding skill will instruct them on this.

---

## Step 6 — Resolve Open Decisions [if any]

The following items were not resolved during init and need your attention.
Each one affects a specific skill or reference file.

[Repeat for each open decision from init-log.md:]

**Open decision [#]: [Item name]**
- What it affects: [skill or file]
- What you need to decide: [description]
- How to resolve: [e.g., "Tell Claude: [instruction]" or "Upload [file] to the project"]

---

## Step 7 — Confirm Init Complete

Once all steps above are done, open a new conversation in the Claude project and say:

> "Init procedure complete. Please update the init log to reflect completion."

Claude will update `init-log.md` with a completion timestamp and status.

---

## Troubleshooting

**A file won't upload** — check that you removed the old version first (Step 2).
**Claude doesn't seem to recognize the new files** — start a new conversation; Claude
reloads project files at the start of each conversation.
**An open decision is blocking a skill from working** — resolve it using Step 6 before
asking researchers to use that skill.
```

---

## File naming conventions

All skill files follow the `*-skill.md` convention within the project. When distributing
to researchers, the researcher onboarding skill handles the `.skill` packaging and
instructs researchers on installation. Init does not need to address this.

All output files from init use lowercase hyphenated names. Never overwrite source files
in `/mnt/project/` directly — always write to `/mnt/user-data/outputs/` and instruct
the admin to upload via the handoff procedure.

---

## Edge cases

- **Admin has skills installed at the account level that aren't in the project.** Skills in `/mnt/skills/user/` are available to the admin personally but not to other researchers — they only have access to what's in the project files panel. During E1, check both locations and flag any gap in the init log. The handoff procedure should explicitly list these skills and instruct the admin to upload them to the project so all researchers have access. Do not assume a skill is available project-wide just because the admin can use it in their own sessions.
- **A skill file has no placeholders.** Note in the execution log that it was scanned
  and required no changes. Do not produce a modified output for it.
- **Admin uploads a client packet or external document.** Read it during discovery and
  extract any relevant context (goals, constraints, personnel, confidentiality terms).
  Log what was extracted and from which document.
- **SharePoint connector is active.** Use it during execution to retrieve any documents
  identified in discovery. Note what was retrieved and from where in the init log.
  Do not rely on connector access being available in future sessions — extract and
  embed all relevant context into project files during init so it persists.
  **Known M365 connector limitation (confirmed 2026-06-11, folder-listing fix added
  2026-07-01):** The `folderName` filter on `sharepoint_search` and the
  `sharepoint_folder_search` tool both return empty results regardless of query on the
  DSLAI site. This is a connector limitation, not a permissions issue. Never conclude a
  file is missing or inaccessible based solely on a folder-filtered search failure.
  Use the root-walk method described in Step E7 instead: get a `driveId` from any
  `sharepoint_search` hit, `read_resource` on `file:///{driveId}/root` to find the
  target folder's own item URI, then `read_resource` on that folder URI for a complete,
  unfiltered listing. This supersedes the older content-term-search workaround — prefer
  it going forward.
- **Project has no existing skill files.** Flag this in the init log. Proceed with
  index and reference file generation. Note in the handoff procedure that skills must
  be uploaded before researchers can be onboarded.
- **Admin triggers init mid-session after other work has been done.** Read all prior
  context in the current conversation before beginning discovery. Treat anything the
  admin has already stated as pre-filled discovery answers.

---

## Reference files

- `/mnt/project/index.md` — project index (primary navigation)
- `/mnt/project/artifact-classes.md` — artifact class definitions
- `/mnt/project/energy-model.md` — energy model and token-class buckets
- All `*-skill.md` files in `/mnt/project/` — scanned and updated during execution
