---
name: project-updater
description: >
  Use this skill to check an already-initialized DSL project for drift and refresh
  index.md. Trigger on explicit requests: "update the project", "sync the index",
  "check for drift", "is everything in sync", "run a project update", "health check
  the project". Also OFFER (do not auto-run) this skill — with a confirm-before-running
  step — after project-init finishes, after any *-skill.md file is added or edited
  mid-session, or when index.md's "Last updated" date looks stale relative to changes
  observed in the conversation. Do NOT trigger for initial project setup — that is
  project-init's job. This skill assumes the project already exists and only checks
  whether its record of itself (index.md) still matches reality.
---

Last modified: 2026-07-23

# Project Updater

Ongoing project-health check and `index.md` sync tool. Distinct from `project-init`:
init is one-time setup; this skill is the recurring maintenance check that runs after
the project already exists.

---

## What this skill checks

Four sources of truth, compared against each other:

1. **`/mnt/project/index.md`** — the project's declared index
2. **SharePoint `Claude Skill Files/index.md`** — the SharePoint copy of the declared index
3. **Actual `/mnt/project/` file listing** — ground truth, project side
4. **Actual SharePoint `Claude Skill Files/` folder listing** — ground truth, SharePoint side

Any mismatch across these four — a file listed in one place but not another, a file
present but unlisted, listings that disagree with each other — counts as drift and is
a blocker. Do not silently resolve drift. Stop and confirm with the researcher which
source is authoritative before writing anything.

---

## Getting a true SharePoint folder listing

Do NOT use `sharepoint_folder_search` or the `folderName` filter on `sharepoint_search`
to enumerate the folder — both return empty results on this site regardless of query
(confirmed limitation, also noted in `project-init-skill.md`'s Edge Cases).

Use this method instead, which returns a complete, unfiltered listing:

1. Run any `sharepoint_search` query scoped to the site to obtain a `driveId` (any
   hit on any document works — the `driveId` is the same across the whole site).
2. Call `read_resource` on `file:///{driveId}/root`. This returns the top-level
   folders and files of the drive, each with its own `file:///{driveId}/{itemId}` URI.
3. Find the `Claude Skill Files` folder in that listing and note its item URI.
4. Call `read_resource` on that folder's own URI. This returns the complete file
   listing inside it — names, sizes, and URIs for every item, no search filtering
   involved.

This is more reliable than the content-term-search workaround described elsewhere —
prefer it for this skill. (If asked, it's worth suggesting the same fix be reflected
in `project-init-skill.md`'s Edge Cases section, since that skill's Step E7 does the
same kind of reconciliation.)

---

## Workflow

### Step 1 — Gather the four sources

- `view /mnt/project/index.md`
- `bash_tool ls /mnt/project/` (or `view` the directory) for the actual project file listing
- `read_resource` the SharePoint `index.md` (locate via the folder-listing method above,
  or via a direct `sharepoint_search` hit if one surfaces it)
- `read_resource` the SharePoint `Claude Skill Files/` folder listing via the method above

If the Microsoft 365 connector is unavailable or fails, do not fail the whole check —
run the two project-side comparisons only (declared vs. actual in `/mnt/project/`) and
note `[INCOMPLETE CHECK — SharePoint unreachable]` in the report.

### Step 2 — Build the comparison

Construct a table: one row per filename seen anywhere across the four sources, one
column per source, marking presence (✅) or absence (❌).

### Step 3 — Evaluate drift

- **No mismatches anywhere** → proceed to Step 5 (nothing to resolve).
- **Any mismatch** → STOP. This is a blocker. Present the specific mismatches (not a
  vague "things are out of sync" — name the file and which sources disagree). Ask
  which source is authoritative for each mismatched file. Do not guess, and do not
  default to any one source being "more correct" — SharePoint, the project, and the
  two index files are all just as capable of being stale.
- Wait for the researcher's resolution before writing anything.

### Step 4 — Apply resolution

Once the researcher has said which source wins for each mismatch, incorporate that
into the working picture of "true" project state.

### Step 5 — Regenerate index.md

If drift was found and resolved in Step 4, or if no drift was found but the index is
otherwise due for a refresh (e.g., stale date), regenerate `index.md` following the
existing structure (Skills table, Reference Files table, Assets table) with the
current, agreed-upon state and today's date in "Last updated." Save to
`/mnt/user-data/outputs/index.md`.

If nothing changed and the date isn't stale, skip regeneration — report "in sync,
no changes needed" instead of producing a file nobody needs.

### Step 6 — Report

Report inline in chat (not as a separate log file — this is a lightweight recurring
check, not an init-scale event):

- The comparison table from Step 2
- Any drift found and how it was resolved
- Whether `index.md` was regenerated
- The SharePoint write limitation, if relevant (see below)

### Step 7 — Flag what a human still has to do

The Microsoft 365 connector is read/search only — this skill cannot write back to
SharePoint. If SharePoint's `index.md` was the outdated one, or if SharePoint is
missing a file the project has, say so plainly and tell the researcher they'll need
to update SharePoint manually. Don't imply the sync is "complete" if a SharePoint-side
edit is still outstanding.

---

## Secondary checks (non-blocking — report as notes, not blockers)

- **Naming convention violations** — any skill file not matching `*-skill.md`
  (lowercase, hyphen-separated, `-skill` suffix).
- **Stale "Last updated" date** — if `index.md`'s date is meaningfully older than the
  most recent file modification time seen across the four sources, note it even if
  no drift was found.

These do not stop execution the way drift does — just flag them in the report.

---

## Proactive offering rules

Never run this skill automatically. Offer it — a short one-line suggestion, not a
forced check — when:

- `project-init` has just finished a run
- Any `*-skill.md` file was added or edited earlier in the current session
- `index.md`'s "Last updated" date looks stale relative to changes observed in
  conversation

Wait for the researcher to confirm before running Steps 1–7. If they decline, don't
ask again in the same session unless something new changes.

---

## Edge cases

- **Only 3 of 4 sources reachable.** Run the check on what's available, mark the
  unreachable source `[NOT CHECKED]` in the comparison table, and don't claim a clean
  bill of health across all four — say explicitly which pair(s) were actually compared.
- **Sources disagree and the researcher isn't sure which is right.** Don't pick for
  them. Lay out what each source currently shows (including last-modified dates, which
  can help them decide) and let them call it.
- **A file exists in all four but under inconsistent names** (e.g. casing, spacing
  differences between the SharePoint filename and the project filename). Flag as a
  naming mismatch, not a presence/absence drift — different failure mode, still worth
  surfacing.
- **No index.md.png entries changed but files still added.** If new files exist in the
  actual listings but neither index.md agrees on their presence yet, that is drift —
  not a "safe to ignore" case just because nothing was deleted.

---

## Reference files

- `/mnt/project/index.md` — project's declared index
- `/mnt/project/` — actual project file listing (ground truth, project side)
- SharePoint `Claude Skill Files/index.md` — SharePoint's declared index
- SharePoint `Claude Skill Files/` folder — actual listing (ground truth, SharePoint side)
- `project-init-skill.md` — one-time setup skill; this skill does not duplicate its
  logic and should not be invoked by it or vice versa
