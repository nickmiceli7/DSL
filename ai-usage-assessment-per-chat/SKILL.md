---
name: ai-usage-assessment-per-chat
description: "Assess how a person worked with Claude in ONE specific chat — the live session or a named past session pulled from project history — and render a single visual AI Usage Assessment scorecard with growth coaching. Use this whenever someone wants a particular session scored rather than the whole project at once: 'assess this chat', 'score the deck-build session', 'how did I use you in the June 11 chat', 'run the per-chat assessment', 'grade that session', or 'go through my project chats one at a time'. Trigger it even when phrased casually. It scores the four competencies of AI fluency (Delegation, Description, Discernment, Diligence) from cited moments in that one chat, and bundles its own scorecard template so nothing needs to be uploaded. When asked to cover several chats, it does them one (or a small batch) at a time, newest first, never all at once. Built for the DSL AI Policy Initiative M&E framework; pairs with the manual DSL AI Fluency Scorecard demo, which shares this record format."
---

# AI Usage Assessment (per chat)

This skill closes the loop on a *single* working session — the one happening now, or a specific earlier one in the project — by reading that chat and reflecting back how the person worked with the AI. It scores the four competencies of AI fluency, then renders one visual scorecard that names what worked and turns each gap into a single forward action. Score and coaching happen in the same run.

The guiding belief: people learn from mistakes when the mistake is specific, tied to the moment it happened, and paired with a clear next move. The scorecard is not a paragraph of prose and not a grade to rank against. It is a growth instrument.

This is the per-chat variant. It differs from the base skill in three ways: (1) it bundles its own template at `assets/scorecard.html`, so the person never has to upload one; (2) it explicitly supports scoring a *past* chat retrieved from project history, with an honesty rule for the lower-fidelity evidence that comes from retrieval; (3) when asked to cover many chats, it works through them deliberately — one or a small batch at a time — rather than rendering a wall of cards.

## Hard rules of tone and framing

Not optional; they are what makes the instrument help rather than harm.

- **Formative, never punitive.** The job is improvement, not judgment. Never rank the person against others.
- **Strengths first.** Name what they did well before any gap. All-gap feedback makes people defensive.
- **Task, not ego.** Keep the big number quiet; make the next-time actions the centerpiece.
- **Contextual, from that chat.** Every call cites the actual moment in the session being scored. Never use a generic line where a real one is available. This is the difference between coaching and a horoscope.
- **No piling on.** If the overall is low, coach the two highest-leverage gaps only.
- **Show the evidence so a human can override.** Propose the score with its evidence; the human owns the final call before it is logged.
- **Forward, not red.** Frame gaps as "to grow," never as failures. The visual uses an upward mark, not a cross.

## Step 0 — Find the ledger record

Do this before scoring. It supplies the task grouping and the session's quality flags, and it
is what lets several sessions of one deliverable be read together rather than as unrelated chats.

Scan for the literal sentinel `=== DSL LEDGER RECORD ===` in, in this order:

1. the session transcript under `/mnt/transcripts`, if one exists;
2. the live chat context, where `work-recorder` emitted it;
3. an uploaded `ledger-*.md` file;
4. the memory ledger line.

Parse the fields defined by **data contract v1.0** (`schema_version`, `record_type`, `task_id`,
`session_id`, `task_class`, `fellow_ref`, `opened_at`, `last_seen_at`, `completion_status`,
`time_quality_flag`). Carry `task_id`, `session_id`, `task_class` and `time_quality_flag` into
`meta`, and use `fellow_ref` as `recordId` so the pseudonym is consistent across skills.

How to use what you find:

- **`task_class` and session position** set expectations. Session 3 of a deck build is judged
  differently from a first session, and a `review` task legitimately has fewer turns than a
  `document` one.
- **`time_quality_flag`** is contextual evidence, never a criterion. An `over-ceiling` session
  may indicate a chat left parked, which bears on right-sizing — but only alongside evidence
  from the chat itself. Never fail a criterion on a flag alone.
- **`completion_status: abandoned`** means the session should be read as an abandoned effort,
  not a failed one. Say so on the card and coach accordingly.

If no record is found, say so on the card in one line and score from the chat alone. Do not
invent a task ID and do not guess a class.

## Step 1 — Identify and retrieve the target chat

Score exactly one chat per card. First settle which one.

- **The live chat.** If the person means the session in progress, read it directly. This is the highest-fidelity case — full evidence is in front of you.
- **A past chat.** If they name an earlier session ("the deck-build chat", "June 11", "the matrix one"), retrieve it before scoring. Use `conversation_search` with topic + date keywords, or `recent_chats` for a time-anchored ask. The retrieved summary/snippets become the evidence base. **This is lower fidelity** — see the fidelity rule below.
- **Ambiguous.** If you cannot tell which chat is meant, ask once, then proceed.

### The fidelity rule (retrieved chats)

Snippets and summaries show *what was produced and decided*, not every prompt-level move. So:

- State on the card and in chat that the assessment is **reconstructed from retrieved evidence, not the full transcript**, and keep evidence lines at session-pattern granularity ("across this session you…") rather than quoting a turn you cannot see.
- Before defaulting a criterion to fail, try **one more retrieval** (a second `conversation_search` with different keywords) to surface the missing evidence.
- Distinguish three cases when judging:
  - *Clear evidence it happened* → pass, cite it.
  - *Clear evidence it did not happen, in a session where it plausibly should have* → fail, coach it.
  - *Genuinely not observable from the retrieved material* (the opportunity may or may not have arisen) → do **not** penalize the person for what retrieval simply didn't surface; mark it pass and say in the evidence line that it wasn't observable in the retrieved record. Formative-not-punitive wins ties.

### Multiple chats

If asked to assess several sessions, do **not** render them all in one turn. Work newest → oldest (or as directed), **one card or a small batch (3–4) at a time**, checking in between. **Skip sessions with no assessable work** — a bare slash-command invocation, an empty status ping, a "hi" — with a one-line note ("nothing to score here") instead of forcing a hollow card. Note that a large batch of heavy renders is itself a resource-intensive use; right-size it.

## The rubric

Seven criteria across four competencies, each competency worth 10. Three competencies split
6 primary / 4 secondary; **Diligence has a single criterion worth 10**. The template does all
arithmetic from the pass/fail judgments — do not compute scores by hand; just judge each
criterion.

| Competency | Primary criterion (6) | Secondary criterion (4) |
| --- | --- | --- |
| Delegation | Right-sized the task (broke big asks up; did not use the model for trivia) | Separated human-only judgment from AI-suitable drafting |
| Description | Stated goal, context, and constraints explicitly | Structured the request to succeed in one or two turns |
| Discernment | Checked output against a source or known fact | Considered staleness (model has a knowledge cutoff) |
| Diligence | Sourced or flagged AI output when carrying it onward (10) | *(none — see note)* |

**Why Diligence has no secondary criterion.** It previously carried "Kept the interaction
reproducible (logged, one traceable thread)." Under `work-recorder` the ledger is written by
Claude, not by the fellow, so that criterion scored an automated feature rather than a human
habit — free marks when the tooling worked, an unfair penalty when it did not. It was removed
in schema v1.0 rather than rewritten. Do not supply a `dil2` key; the template no longer
defines one and an extra key is ignored silently, which would hide a stale caller.

### Foreseeability (del1 and des2 only)

Two criteria — "Right-sized the task" (`del1`) and "Structured to succeed in few turns" (`des2`) — are sensitive to whether the complexity that caused a miss was **foreseeable** before the session began. A user who scoped well but hit an unforeseeable wall (an API changed, a tool broke, the task turned out to be far harder than anyone expected) should be coached differently from one who could have planned better.

For `del1` and `des2`, add a `foreseeable` boolean (`true` / `false` / omit if not applicable). When `foreseeable` is `false` on a failed criterion, the card shows an "unforeseeable complexity" chip and a `foreNote` explaining why the miss is contextual rather than a skill gap. The score itself is unchanged — the rubric is binary — but the coaching tone shifts from "plan better" to "adapt when surprised." The override line in the rationale should note that a reviewer may choose to treat an unforeseeable miss as a soft pass.

Bands: Beginner 0–39, Basic 40–64, Proficient 65–84 (the target), Advanced 85–100.

Sustainability is a behavioral proxy read from right-sizing and few-turn structure. It is
scored separately from the energy class.

**Energy class is measured, not declared.** This supersedes the earlier rule that no kWh
figures were to be produced here — that rule belonged to a scope in which DSL had no
measurement path. It now has one. The class comes from the session's measured electricity as
reported by `ai-energy-emissions-report`, using the **central scenario (A) total electricity,
PUE-inclusive**:

| Class | Measured total electricity |
|---|---|
| Light | under 0.25 kWh |
| Moderate | 0.25 to 1.0 kWh |
| Heavy | 1.0 to 3.0 kWh |
| Very Heavy | over 3.0 kWh |

Thresholds are DSL-set parameters, version-tagged per semester (Fall 2026 values above), not
derived quantities. They are anchored so that a short working session lands Light, a standard
one Moderate, and the 14 August 2026 measured reference session (2.315 kWh) lands Heavy.

Three rules on using them:

- **Never show the class without the kWh figure and the scenario spread.** A bare "Heavy" implies
  a precision the parameter range does not support.
- **If no measured figure exists for the session, set `energyClass` to `"Not measured"`** and say
  why on the card. Do not estimate a class from how the session felt — that is the declared-class
  behaviour this change removes.
- The Enterprise admin console remains the source of truth for **billing and organizational
  reporting**. The measured session figure is the source for per-session classification.

## Step 2 — Judge, then render

1. **Judge each of the seven criteria** pass or fail from the evidence of that one chat. Apply the evidence-not-benefit-of-the-doubt standard from Step 1's fidelity rule. For **every** criterion, also assemble its **scoring rationale** (see "Scoring rationale" below) — the discrete evidence points, each tagged observed or inferred, plus why the call landed and what would flip it. This is required on every run, pass or fail.
2. **For every criterion, write the evidence line**: the specific moment that justifies the call, in plain language addressed to the person ("you…").
3. **For each failed criterion, add why and next**: one line on why it matters, one concrete action small enough to do in a single future session.
4. **Set the energy class from measurement.** Read the session's measured total electricity
   (central scenario) from `ai-energy-emissions-report` and map it through the threshold table.
   If that report has not been run for this session and cannot be, set `"Not measured"`. Never
   judge the class from impression.
5. **Write one headline next step**: the single highest-leverage move, named to the competency it lives in.
6. **Render the scorecard.** Read `assets/scorecard.html` (bundled with this skill — no upload needed), replace only the `ASSESSMENT` data object with this session's values, and output the result as an HTML artifact saved to the outputs directory. Do not touch the `RUBRIC` block or the `compute()` function. The render code already handles the `rationale` fields — it shows each criterion's trail as a collapsed "How this was scored" expander on its card *and* in a consolidated "Full scoring rationale" section — and it computes scores, bands, qualities, tiles, cards, and the audit record from your judgments. The Outcome Qualities panel includes hover/tap tooltips (the `?` icon next to each quality name) showing which criteria feed that quality and whether each passed or failed, so a reader can trace how the score was derived without extra scrolling. Foreseeability chips render automatically on `del1`/`des2` when you supply the `foreseeable` field.
7. **Hand off the audit record.** The template includes a de-identified record block in a collapsible section — that is the artifact for the activity log. Remind the person to log it and to use a **pseudonym, not a real name**.

## Filling the data object

Replace the `ASSESSMENT` constant. Provide every criterion id (del1, del2, des1, des2, dis1, dis2, dil1). There is no dil2. Passed criteria need only `evidence`. Failed criteria need `evidence`, `why`, and `next`. For a retrieved chat, set `meta.context` to identify the session (e.g. "Ch11 · agentic-landscape triangulation — retrieved") so the card itself shows it was scored from history.

```javascript
const ASSESSMENT = {
  meta:{recordId:"<pseudonym>", context:"<which chat / cohort / task>", date:"<YYYY-MM-DD>",
        energyClass:"<Light|Moderate|Heavy|Very Heavy|Not measured>",
        energyKwh:"<e.g. 2.315 kWh central, range 2.31-46.25>",
        taskId:"<TSK-00N or blank>", sessionId:"<TSK-00N-S0N or blank>",
        taskClass:"<ledger class or blank>", timeFlag:"<ok|paused|over-ceiling|unclosed|blank>"},
  results:{
    del1:{passed:true,  evidence:"...", foreseeable:true},
    del2:{passed:false, evidence:"...", why:"...", next:"..."},
    // des2 also supports foreseeable / foreNote:
    des2:{passed:false, evidence:"...", why:"...", next:"...", foreseeable:false, foreNote:"The task's complexity only became apparent after the API returned an unexpected schema."},
    // ... all seven ids
  },
  headlineNext:"..."
};
```

Keep evidence lines to one or two sentences. Address the person directly. Make "next" an action, not a principle. The `foreseeable` field is only used on `del1` and `des2`; omit it on other criteria. When `foreseeable` is `false`, include `foreNote` (one sentence explaining why).

### Scoring rationale (required on every criterion)

Each criterion also carries a `rationale` object — the transparency trail that lets a reader see *how* the call was reached and override it. Produce it for all seven criteria, every run.

```javascript
del1:{ passed:true, evidence:"...",
  rationale:{
    points:[
      {text:"You opened by asking what to specify before executing.", basis:"observed"},
      {text:"The deck landed staged \u2014 priority slides first, the rest deferred.", basis:"inferred"}
    ],
    call:"Both halves of the criterion clear the bar, so: pass.",
    override:"If you actually ran it as one big pass, the 'broke it up' half weakens \u2014 consider downgrading."
  }
}
```

Rules:

- **`points`** is the heart of it: the discrete things you actually used to judge. **Tag each `basis` honestly** — `"observed"` when it's in the evidence in front of you, `"inferred"` when you reasoned it from ledger notes, retrieved summaries, or pattern. For a `retrieved` chat most points will be inferred; for the live chat most will be observed. Do not launder an inference into an observation to make the card look more certain.
- **`call`** (one line) states why the points add up to the pass/fail you gave.
- **`override`** (one line) names the condition under which a human should flip your call — the soft-call escape hatch. Always include it; for a clean pass it reads "would weaken if…". This is the verify-then-trust contract made concrete: you show the seam, the human decides.
- Keep each piece to a sentence. The rationale is an audit trail, not a second essay.

## A clean swap (mechanics)

The `ASSESSMENT` object is the only block to change. A reliable way to swap it without disturbing the rest:

```python
import re
src = open('assets/scorecard.html').read()
new_block = '''const ASSESSMENT = { ... };'''   # your filled object
src2, n = re.subn(r'const ASSESSMENT = \{.*?\n\};', lambda m: new_block, src, count=1, flags=re.DOTALL)
assert n == 1
open('/mnt/user-data/outputs/AI_Usage_Assessment_<chat>.html','w').write(src2)
```

Then sanity-check that `const RUBRIC`, `const ASSESSMENT`, and `function render` are all still present before presenting the file.

## Relationship to the other pieces

This is the per-chat, retrieval-aware path: one chat, one assessment pass, one rendered card. The base **ai-usage-assessment** skill assumes the live session only. The manual **DSL AI Fluency Scorecard** demo is the deterministic self-assessment path for scoring a transcript by hand without a model call. All three emit the same record format, so cards from any of them can sit in the same activity log.

## Example trigger and response

**Trigger:** "Can you assess how I used you in the matrix-build chat?"

**Response shape:** Identify the chat; if it's not the live one, retrieve it and say the assessment is reconstructed from the record. Judge the eight criteria with cited moments. Note briefly in chat the headline read ("Strong sourcing; the gap is staleness-checking"). Render the scorecard artifact with strengths first, each gap turned into a next-time action, the score kept quiet, and the audit record in the collapsible section. Close by reminding the person to log the de-identified record under a pseudonym.
