---
name: work-recorder
description: Open and maintain a DSL research fellow's session ledger — the running record of which chats belong to which task, what each session produced, and whether the task finished. Use at the start of any working conversation in a DSL project, when a fellow says what they are about to work on, when a fellow asks to log, close out, or review a session, when a fellow asks what they have worked on so far, and at the end of a working chat before or alongside any assessment skill. Trigger on "start a task", "I'm working on", "log this session", "record my work", "close out this chat", "open my ledger", "open the work recorder", "what have I logged", "mark this done", or on the first substantive message of a session in a project where a ledger exists. Asks the fellow at most two short questions per task and writes everything else itself. Produces a markdown ledger the fellow keeps in project knowledge.
---

# Work Recorder

The record every DSL assessment reads from. It answers three questions that cannot be
reconstructed later: **which chats made up one task**, **what kind of task it was**, and
**whether it finished**. Everything else in the record is measurable after the fact; these
three are not.

Over a semester the ledger accumulates into the evidence base for workload estimation at
DSL. That is a downstream benefit, not the fellow's problem. Design rule for this skill:
**two questions per task, nothing per session.** If a field can be measured, measure it.
If it can be derived, derive it. Only ask when neither is possible.

## What the fellow is asked

Once at task open: *what are you working on?* One sentence.

Once at task close: *is this task finished?* Yes, no, or abandoned.

That is the whole burden. Do not ask for hours, turn counts, tool counts, task
classification, or variance explanations. Task class is assigned by this skill from the
fellow's sentence, not chosen by them, so that the vocabulary stays controlled and
comparable across fellows.

## Storage, and the constraint that shapes it

`/mnt/project` is **read-only**. This skill cannot write into project knowledge. So the
ledger lives in two places with different jobs:

| Where | Holds | Updated |
|---|---|---|
| Memory, via `memory_user_edits` | One compact line: fellow, open task IDs, session count | Every session close. Zero fellow action. |
| `ledger-<fellow-slug>.md` in project knowledge | The full auditable table | Regenerated at task close; the fellow uploads it, replacing the old copy |

Per-chat friction is therefore zero, and the fellow does one upload per completed task
rather than one per chat. At session open, read the ledger from `/mnt/project` if it is
there; fall back to the memory line if it is not; if neither exists, this is a first
session and the ledger gets created.

## The ledger record — data contract v1.0

The ledger is only useful to the assessment skills if they can find it mechanically. This
section is the contract. `ai-usage-assessment-per-chat` and `ai-energy-emissions-report`
both parse the record defined here.

### Mechanism

Claude cannot write to `/mnt/transcripts`; it is read-only. But **everything this skill
prints in chat becomes transcript content**, so emitting the record as visible chat text
puts it in the transcript automatically, where a parser can find it. That is the mechanism.
Do not claim to write transcript files.

Emit the block exactly twice per session: once at session open, once at session close. Emit
it verbatim, in a fenced code block, with the sentinel lines intact. Do not reformat it into
a table, do not pretty-print it, and do not omit fields — a reader that cannot match the
sentinel will silently find nothing.

### Schema

```
=== DSL LEDGER RECORD ===
schema_version: 1.0
record_type: open
task_id: TSK-001
session_id: TSK-001-S03
task_class: deck
fellow_ref: DSL-RF-01
opened_at: 2026-09-08T14:02:11Z
last_seen_at: 2026-09-08T14:02:11Z
completion_status: open
time_quality_flag: ok
=== END LEDGER RECORD ===
```

| Field | Values | Notes |
|---|---|---|
| `schema_version` | `1.0` | Bump only with a matching change in both reader skills |
| `record_type` | `open`, `close` | Which end of the session this record marks |
| `task_id` | `TSK-00N` | Groups sessions into one deliverable. The join key. |
| `session_id` | `<task_id>-S0N` | Unique per session; sortable within a task |
| `task_class` | the eight classes above | Controlled vocabulary only |
| `fellow_ref` | pseudonym | **Never a real name or email** |
| `opened_at` | ISO 8601 UTC | From `date -u` at first substantive message |
| `last_seen_at` | ISO 8601 UTC | Equals `opened_at` on an open record |
| `completion_status` | `open`, `complete`, `abandoned` | Task-level, not session-level |
| `time_quality_flag` | `ok`, `paused`, `over-ceiling`, `unclosed` | Governs time-aggregate inclusion |

Elapsed is deliberately **not** a field — it is derivable from the two timestamps, and a
stored duration would drift from them. Active working time is not a field and must never be
added; the rule that it is not derived per person survives into the record.

### Reader expectations

A reader locates records by scanning transcript message text for the literal sentinel
`=== DSL LEDGER RECORD ===`. Discovery order when a transcript is unavailable: live chat
context, then an uploaded `ledger-*.md`, then the memory line. A reader that finds no record
says so and proceeds without one; it never invents field values.

### No automatic hook

There is no automatic end-of-chat hook. The closing row and its `close` record are written
when the fellow says they are done, or when an assessment skill runs at chat close and calls
this one. If a chat ends without a closing row, the next session's open reconciles it: mark
the prior session `unclosed`, carry its opening timestamp, and do not invent an elapsed
figure.

## At session open

1. **Timestamp.** `date -u '+%Y-%m-%dT%H:%M:%SZ'`. Record it. This is the only way elapsed
   time becomes measurable rather than remembered.
2. **Load the ledger** — `/mnt/project` first, memory line second.
3. **Continuing or new?** If the fellow's sentence matches an open task, attach this
   session to that task ID. If not, open a new task: assign the next `TSK-00N`, classify it
   per the table below, record the title and the open date, set status `open`.
4. **Emit the `open` record** in a fenced block, per the data contract above. This is what
   makes the session discoverable to the assessment skills later. `last_seen_at` equals
   `opened_at`; `completion_status` is the task's current status; `time_quality_flag` is `ok`.
5. **Confirm in one line and get out of the way.** "Logged as session 3 of TSK-002, deck
   build." Then answer whatever they actually came to do. Do not present the ledger, do not
   explain the fields, do not offer a summary unless asked.

## Task classes

Assign exactly one. Controlled vocabulary — do not invent classes, and do not let the
fellow's phrasing create a new one. `other` requires a one-line note saying why nothing
fit; a recurring `other` is a signal the vocabulary needs revision, which is a DSL
decision, not a fellow one.

| Class | Covers |
|---|---|
| `correspondence` | Emails, short memos, replies |
| `research-synthesis` | Literature scans, landscape reviews, evidence gathering |
| `document` | Reports, briefs, guidebooks, framework documents |
| `deck` | Slide decks, technical bulletins |
| `data` | Spreadsheets, trackers, datasets, figures |
| `tooling` | Skills, scripts, prototypes |
| `review` | QC passes, evaluations, critique of existing work |
| `other` | Requires a note |

## At session close

1. **Timestamp** again. Elapsed runs from the open stamp to the **last sign of activity** —
   the last-seen heartbeat if one was recorded, otherwise this close stamp. If the close
   stamp is materially later than last-seen, note the trailing span as idle; do not include
   it in elapsed.
2. **Write the session row.** Fill what is measurable:
   - elapsed, per above, with the time flag: `ok`, `over-ceiling`, `unclosed`, or `paused`
   - model calls and tool calls, from the transcript if `/mnt/transcripts` has one for this
     session; otherwise count tool calls made in this chat and mark the figure `counted`
   - long-gap count, if an energy assessment ran and reported cache-expiry events;
     otherwise leave blank rather than guessing
   - deliverables produced, by filename
3. **Ask the one question:** is the task finished? Set status `complete`, leave it `open`,
   or set `abandoned`. An abandoned task must be recorded as abandoned — a half-done task
   left looking complete will distort every future baseline it lands in.
4. **Emit the `close` record** in a fenced block, per the data contract. Carry the same
   `task_id` and `session_id` as the open record, set `last_seen_at`, and set the final
   `completion_status` and `time_quality_flag`. A session whose open and close records
   disagree on `task_id` cannot be aggregated, so copy, do not retype.
5. **Update memory** — one line, no figures.
6. **On task completion only:** regenerate the full ledger file, `present_files` it, and
   tell the fellow to replace the copy in project knowledge.

## Time: elapsed wall-clock, and the protocol that makes it usable

Elapsed wall-clock is the time measure. Per-message timestamps are not confirmed available
and nothing here depends on them.

Elapsed is only meaningful if a session is roughly the sitting it claims to be. A fellow who
opens a chat, leaves for a lecture, and closes it three hours later logs three hours of
elapsed against twenty minutes of work. DSL fellows are enrolled students, so this is the
normal case, not the edge case. Two layers handle it: a protocol the fellow follows, and
guards that catch what the protocol misses.

### Session hygiene protocol — three rules for fellows

State these once at onboarding and print them in the ledger header. Do not repeat them at
every session open.

1. **One sitting, one session.** Open a working chat when you can actually work it. A chat
   is not a place to park a task.
2. **Say "pause" when you stop.** One word closes the session row. On return, keep working
   in the same chat or a new one and say what you are picking up — it becomes a new session
   row under the same task ID. Two honest short rows beat one long unreliable row.
3. **Say "done" when the task is finished.** This sets the completion marker, which is the
   one thing nothing else can recover.

Rules 2 and 3 are single words a fellow would say anyway. Nothing else is asked.

### Guards, requiring no fellow action

- **Open-stamp on first substantive message.** Take the open timestamp when the fellow says
  what they are working on, not when the chat window opens. A greeting is not a session start.
- **Last-seen heartbeat.** Whenever any DSL skill runs during the session, record a
  last-seen timestamp. If a close arrives well after the last sign of activity, elapsed runs
  open to last-seen, and the trailing span is reported separately as idle rather than folded
  into the session.
- **Ceiling.** Any single session whose elapsed exceeds the ceiling is recorded at its raw
  value and flagged `over-ceiling`. Default ceiling **3 hours** — a continuous sitting longer
  than that is possible but uncommon, and past it the chance the span contains a lecture
  outweighs the value of the data point. This is a DSL-set parameter, not a derived one, and
  it is version-tagged per semester like every other estimator parameter.
- **Long-gap count.** If an energy assessment ran and reported cache-expiry events, record
  the count. It is independent evidence of mid-session gaps.

### The exclusion rule

A session flagged `over-ceiling` or `unclosed` is **excluded from time aggregates only**.
Its model calls, tool calls, token volumes, and deliverables stay fully valid and fully
counted — nothing about a long idle gap makes the work record wrong. Never repair an
excluded session by estimating what the elapsed should have been.

### What elapsed is not

State this in the ledger itself, once, so no reader mistakes it. Elapsed is an upper bound
on working time, not a measure of it. Active working time is not collected per fellow; the
lab-wide ratio of active to elapsed comes from the end-of-semester survey. Never label an
elapsed figure as hours worked, and never write a per-fellow active-time estimate into the
ledger.

## Ledger format

```markdown
# Session Ledger — <Fellow Name>

Project: <Claude project> · Opened: <YYYY-MM-DD> · Ledger version: 1.0
Session ceiling: 3h (DSL parameter, Fall 2026)

Elapsed is wall-clock from session open to last sign of activity. It includes idle time and
is an upper bound, not hours worked. Active time is not recorded per fellow.

**Session hygiene:** one sitting per session · say "pause" when you stop · say "done" when
the task is finished.

## Tasks

| Task ID | Title | Class | Opened | Closed | Status | Sessions |
|---|---|---|---|---|---|---|
| TSK-001 | <title> | deck | 2026-09-08 | 2026-09-15 | complete | 3 |

## Sessions

| # | Task ID | Date | Open (UTC) | Last seen | Elapsed | Time flag | Model calls | Tool calls | Long gaps | Deliverables |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | TSK-001 | 2026-09-08 | 14:02 | 15:47 | 1h45m | ok | 61 | 38 | 1 | outline.md |
| 2 | TSK-001 | 2026-09-11 | 09:14 | 13:58 | 4h44m | over-ceiling | 40 | 22 | 2 | draft-v1.pptx |

## Notes

| Task ID | Note |
|---|---|
```

Time flags, and their effect on aggregates:

| Flag | Meaning | In time aggregate? |
|---|---|---|
| `ok` | Open and last-seen both recorded, elapsed under ceiling | Yes |
| `paused` | Fellow said "pause"; row closed at that point | Yes |
| `over-ceiling` | Elapsed exceeds the session ceiling | No |
| `unclosed` | Session ended without a closing row | No |

Other marks used in the tables and nowhere else: `counted` on a tool-call figure counted in
chat rather than read from a transcript; blank for anything not measured. Do not fill a
blank with an estimate, and do not change a flag to bring a row back into the aggregate.

## What this skill does not do

- It does not estimate anything. Workload estimation is deferred at DSL pending enough
  ledger data to calibrate it; see the deferred-scope note. If a fellow asks how long a
  task will take, say the lab does not yet have a calibrated estimate and give a judgement
  answer without dressing it as a calculation.
- It does not score the session. `ai-usage-assessment-per-chat` does that.
- It does not measure energy. `ai-energy-emissions-report` does that, from transcripts.
- It does not collect active working time per fellow.
- It does not write to project knowledge. The fellow uploads the file.

## Edge cases

- **Fellow works two tasks in one chat.** Attach the session to the dominant task and note
  the second in the notes table. Do not split one chat into two session rows; the
  transcript cannot be split, so the measured figures would be fiction.
- **Fellow forgets to open the ledger and asks to log at the end.** Record the close
  timestamp, leave the open timestamp and elapsed blank, mark the row `unclosed`.
- **Fellow says "pause" then resumes in the same chat.** Close the row at the pause stamp
  flagged `paused`, then open a new session row under the same task ID at the resume
  message. Two rows in one chat is correct here — unlike the two-tasks case, the split is
  supported by two real timestamps. Transcript-derived call counts cover the whole chat, so
  attach them to the last row and mark them `counted` for the chat rather than the row.
- **Chat left open overnight, closed the next morning.** Elapsed runs to last-seen, not to
  the close stamp. If there is no last-seen heartbeat, flag `over-ceiling` and let the
  exclusion rule handle it. Do not ask the fellow to reconstruct when they stopped.
- **Ledger in project knowledge is older than the memory line.** Memory wins for the
  session count; regenerate the file from memory plus this session and tell the fellow the
  uploaded copy was stale.
- **Fellow asks to remove a task or session.** Do it, and note the removal in the notes
  table with the date. A silently shortened ledger is not auditable.
- **No memory available in the project.** Say so plainly — the ledger will depend entirely
  on the uploaded file, so it must be regenerated and re-uploaded at every session close
  rather than at task close.
