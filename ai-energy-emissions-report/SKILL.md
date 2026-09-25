---
name: "ai-energy-emissions-report"
description: "Estimate the electricity consumption and GHG emissions of AI inference from Claude session token data, and produce a DSL-branded PDF report with five headline metrics, everyday-life equivalents, scenario bounds, a per-turn ledger, and back-end evidence. Use whenever someone asks how much energy, electricity, power, carbon, CO2, or emissions a session, chat, inference, turn, or AI workload used; asks for an AI emissions estimate or AI footprint; asks to apply the Watershed AI emissions framework or an Activity Tier calculation; or asks where a session's tokens went in energy terms. Trigger on phrasing like \"how much energy did this use\", \"carbon footprint of this chat\", \"estimate emissions for this inference\", \"AI energy report\", or \"run the energy numbers\". Built for the DSL Sustainable Agentic AI Usage pillar. Do NOT use for non-AI emissions accounting, or for pure token-count breakdowns with no energy question."
---
> **Logo asset resolution (plugin migration note).** This skill now ships its own copy of
> the DSL logo PDF(s) under this skill's `assets/` folder rather than assuming they live in
> project knowledge. Resolve the path(s) once, before first use, checking the bundled asset
> first and falling back to the legacy project-knowledge location:
>
> ```bash
> DSL_LOGO_PDF=$(ls "./assets/DSL_Memo_Logo.pdf" 2>/dev/null || ls "assets/DSL_Memo_Logo.pdf" 2>/dev/null || echo "/mnt/project/DSL_Memo_Logo.pdf")
> ```
>
> **⚠ Unverified:** the exact path a plugin-bundled asset resolves to at runtime (e.g. whether
> it's relative to a working directory, or exposed via an environment variable such as
> `CLAUDE_PLUGIN_ROOT`) has not been confirmed against current Claude plugin documentation.
> Test this resolution step once the plugin is actually installed, before relying on it —
> the `/mnt/project/` fallback should keep things working in the meantime for any fellow who
> still has the logo in project knowledge.



# AI Energy and Emissions Report

Measures first-party token volumes from Claude session transcripts and estimates electricity and GHG emissions using the **Watershed Activity Tier (Detailed)** framework, then renders a DSL-branded PDF.

## What this produces

One PDF containing:

1. DSL header, subject/author/date/scope block
2. KPI band — five headline tiles in this fixed order: **total GHG emissions**, **inference-specific emissions**, **total electricity used**, **inference-specific electricity**, **energy saved through sustainable AI use** (a deliberate placeholder)
3. Directly beneath the band: a compact **everyday-life equivalents** table for the *inference-specific electricity* figure
4. **Key numbers** table
5. **Measured token volumes** — cache read / cache write / output / fresh input, with shares
6. **Scenario bounds** — six scenarios on identical token volumes, with the spread stated
7. **Diagnostics and reduction levers** — context re-reading share, cache-expiry events, five most expensive calls
8. **Method** — tier placement, equations, cache-disaggregation rationale
9. **Parameters and assumptions** — every factor with source and default/override status
10. **Limitations ordered by severity**
11. **Back-end evidence** — transcript IDs, byte counts, SHA-256 prefixes, dedupe key, generator version
12. Appendix A — per-turn ledger (omit with `--no-ledger`)
13. Appendix B — source citation, including every equivalence factor

Numbered sections run 1–7 (Key numbers through Back-end evidence). The equivalents table is not a
numbered section — it sits inline under the headline numbers, because its whole job is to make those
numbers legible at a glance.

### Headline metric definitions

Fixed. Do not reorder or redefine these without being asked.

| Tile | Definition | Unit shown |
|---|---|---|
| Total GHG emissions | operational + amortized embodied + amortized training | **g / kg CO2e** |
| Inference-specific emissions | operational emissions of the compute alone (`wh_infer / 1000 x GI`) — excludes overhead, hardware, training | **g / kg CO2e** |
| Total electricity used | full facility draw, PUE-inclusive | Wh / kWh |
| Inference-specific electricity | compute-only draw, `total / PUE` | Wh / kWh |
| Energy saved through sustainable AI use | **placeholder — never populate with an estimate** | — |

**Emissions tiles always carry `CO2e`.** A bare mass ("5.30 kg") is wrong — it reads as a mass of
something physical rather than a carbon-dioxide-equivalent. `fmt_kpi_g()` and `fmt_g()` both append
the unit; do not bypass them.

The total GHG tile deliberately leads with a figure that includes amortized training, the least
knowable term. That is a presentational choice, so the note under the band must say so plainly and
point to the breakout in Key numbers. Do not quietly drop training from the total to make the
headline look tighter.

### The energy-saved placeholder

This tile stays blank until DSL has paired empirical measurements from fellows: the same class of
task performed under refined practice (dense prompts, warm cache, few purposeful model calls,
scoped context) against unrefined practice (repeated thin calls, lapsed cache, unbounded context),
with the difference reported as energy avoided. A single transcript cannot produce that number
because it has no baseline. Render it as a visually distinct tile reading "Not yet measured" —
never as `0`, never as an interpolated guess. The note beneath the band explains it in one sentence;
it does not need a section of its own.

## Intake

Ask only what you cannot determine yourself. Skip anything already answered in the conversation.

1. **Subject line** for the report.
2. **Prepared by** — name and affiliation.
3. **Scope** — this session (default), or roll up every transcript in the project folder (`--all-sessions`).
4. **Per-turn ledger** — include (default) or omit. If the person is unsure, explain it in one line: one row per model call showing cache-write, cache-read, output, watts and grams. It is what makes cache-expiry events visible; a session total alone hides them in the average.
5. **Parameter overrides** — only ask if they have mentioned a known serving region, a provider disclosure, or a specific model class. Otherwise use hyperscaler defaults.

Then state the assumptions you are about to apply and proceed.

## Procedure

1. **Locate transcripts.** `ls $HOME/mnt/.claude/projects/*/*.jsonl`. The newest is the live session.
2. **Prepare the logo**, per the Logo handling steps in the design spec below. Search the known paths, extract/rasterize, crop and clean the background into `/tmp/dsl_logo_clean.png`. If no source asset is found anywhere, skip this and note the typographic fallback will be used — do not stop the run for a missing logo.
3. **Write the generator** to `/tmp/dsl_ai_energy_report.py` (full source below). If it already exists and prints `v1.0`, reuse it rather than rewriting — this saves output tokens.
4. **Check reportlab.** `python3 -c "import reportlab"`. If missing: `pip install reportlab --break-system-packages`.
5. **Run it**, writing the PDF into the outputs folder so the person can open it. Pass `--logo /tmp/dsl_logo_clean.png` if step 2 produced one. Use `--outdir` pointed at the outputs folder rather than a hardcoded `--out` filename — the generator names the file itself as `DSL-AI-Energy-Report-<author-slug>-<YYYYMMDD-HHMM>.pdf` (or `...-Project-...` for `--all-sessions`), so the filename itself always records who produced it and when. This also prevents a second run from silently colliding with the first.
6. **Verify before presenting.** Freeze a snapshot (`cat` the transcript to `/tmp/snap.jsonl` — plain `cp` can be blocked in the sandbox), run the generator against the snapshot, and independently recompute electricity, emissions and at least one equivalence in a few lines of inline Python. They must agree to the digit. Do not skip this — it is the auditable standard.
7. **Render and look at page 1** (`pdftoppm -png -r 120`) to confirm no text wrapping or layout faults, that the logo renders cleanly, that the five KPI labels break where intended, and that emissions tiles read `CO2e`.
8. **Present the PDF** with `present_files` and give the central figure, the scenario range, and the single most actionable diagnostic in prose.

## Method

### Equations (Watershed Activity Tier, Detailed)

```
Electricity (Wh) = (I_fresh x EI_in + I_cached x EI_in x f_cache + O x EI_out) x PUE
Operational (gCO2e) = Electricity / 1000 x GI
Embodied (gCO2e)    = E_emb x MTok x 1000
Training (gCO2e)    = E_train x MTok x 1000
```

Token volumes in thousands; EI factors are Wh per 1,000 tokens. Dividing the electricity result by
PUE isolates the inference-specific (compute-only) draw.

### Cache disaggregation — the key move

The framework's base equation applies one `EI_in` to all input tokens. **Footnote 54** permits splitting the prefill term where cache-hit counts are exposed, because cached input reuses key-value states computed in a prior request. **Footnote 48** warns a blended factor overstates repeat-context workloads. Agentic sessions are overwhelmingly cache reads — often 85–95% of all tokens — so applying a flat factor inflates the result several-fold.

`f_cache = 0.10` is a **proxy taken from cache-read pricing, not a measured energy ratio.** Say so every time. It is the weakest assumption in the estimate, which is why the un-credited result is always reported as Scenario B.

### Hardcoded defaults (Watershed Table 2, June 2026)

| Parameter | Hyperscaler | Unknown provider |
|---|---|---|
| `EI_in` | 0.32 Wh / 1,000 tok | 0.70 |
| `EI_out` | 0.96 Wh / 1,000 tok | 2.10 |
| `PUE` | 1.10 | 1.56 |
| `E_emb` | 0.017 kgCO2e / MTok | 0.025 |
| `E_train` | 0.10 kgCO2e / MTok | 0.50 |

`GI` = 341 gCO2e/kWh (eGRID 2024 U.S. national average, location-based). `f_cache` = 0.10. Reasoning uplift for Scenario E = 3x.

Any override is logged in the report's parameter table with status `override`.

### Transcript parsing

Assistant messages are recorded **twice** (streaming + final) with identical `usage`. **Deduplicate by `requestId`** or every figure doubles. Read `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`, `input_tokens`, and `output_tokens_details.thinking_tokens`.

### Cache-expiry detection

A cache-expiry event is a call where `cache_read` fell below 90% of the previous call's while `cache_write` exceeded 10,000 — the prompt cache lapsed and the whole standing context was recomputed. Report it: it is pure waste and avoidable by reducing idle gaps.

## Ledger records — data contract v1.0

`work-recorder` emits a machine-readable record in chat at session open and close. Because
chat text lands in the transcript, the record is in the same JSONL this skill already parses.
Reading it is what lets several sessions of one deliverable be reported together instead of
as unrelated chats.

### Discovery

Scan transcript message text for the literal sentinel `=== DSL LEDGER RECORD ===`, ending at
`=== END LEDGER RECORD ===`. Parse the `key: value` lines. Expect two records per session,
`record_type: open` and `record_type: close`.

Fallback order when the transcript has no record: the live chat context, then an uploaded
`ledger-*.md`, then the memory ledger line. Found nothing? Report the session unaggregated and
say on the report that no ledger record was located. Never synthesize a `task_id`.

Fields, all as defined by the contract: `schema_version`, `record_type`, `task_id`,
`session_id`, `task_class`, `fellow_ref`, `opened_at`, `last_seen_at`, `completion_status`,
`time_quality_flag`. Reject a record whose `schema_version` this skill does not know rather
than parsing it optimistically — a silently misread field is worse than a missing one.

`fellow_ref` is a pseudonym and is the only identifier that may appear in the report. Never
substitute a real name or email even if one is available in context. Do not derive or report
active working time per person; elapsed spans and the `time_quality_flag` are the only time
facts in the record, and elapsed is an upper bound.

### Task-level aggregation

With `--task TSK-00N`, or when several transcripts carry the same `task_id`:

1. Group transcripts by `task_id`. Order sessions by `session_id`.
2. Sum token volumes across the group, then run the equations **once on the summed volumes**.
   Do not sum per-session Wh figures — identical arithmetic, but summing inputs keeps a single
   rounding path and one parameter set.
3. Report the task total, then a per-session breakdown table, then the existing per-turn ledger
   for each session.
4. Carry `task_class` and `completion_status` into the scope block. A task marked `abandoned`
   must be labelled abandoned in the report; its figures are real but it is not a completed
   deliverable and must not be cited as the cost of one.
5. **Sessions flagged `over-ceiling` or `unclosed` are excluded from time aggregates only.**
   Their tokens, calls, energy and emissions stay fully counted. Never drop a session's energy
   because its timing is unreliable, and never repair an excluded session's elapsed by estimate.

### Energy class

Assign a class from the measured **central scenario (A) total electricity, PUE-inclusive** —
the same figure as the "total electricity used" tile:

| Class | Measured total electricity |
|---|---|
| Light | under 0.25 kWh |
| Moderate | 0.25 to 1.0 kWh |
| Heavy | 1.0 to 3.0 kWh |
| Very Heavy | over 3.0 kWh |

Thresholds are DSL-set parameters version-tagged per semester (Fall 2026 values above), not
derived quantities, and the report says so where it prints the class. They are anchored so a
short working session lands Light, a standard one Moderate, and this project's 14 August 2026
reference session (2.315 kWh) lands Heavy.

The class is a communication aid layered on the measurement, never a substitute for it: print
it beside the kWh figure and the scenario spread, never alone. `ai-usage-assessment-per-chat`
reads this class rather than declaring its own, so a session must not carry two different
classes from the two skills.

### When no transcript exists

`/mnt/transcripts` can be empty — confirmed on 27 August 2026, when it held no files at all.
This skill has no input in that case and must fail loudly and specifically:

- Say plainly that no transcript was found, name the path checked, and state that token volumes
  cannot be measured without it.
- **Do not** estimate token volumes, fall back to modelled counts, or produce a report with
  projected figures under a measured heading. A report that looks measured but is not defeats
  the entire point of the tier placement.
- Offer the two available paths instead: the Enterprise admin console for organizational
  figures, and a note that per-session measurement requires the transcript to be present.
- If a ledger record is available from chat context even without a transcript, report the task
  and session identifiers and the fact that measurement was unavailable, so the gap is on the
  record rather than silent.

## Honesty requirements

Non-negotiable. The report is worthless if it implies false precision.

- **Never present a single number alone.** Always give the scenario range and its multiple.
- **State that the central estimate is more likely low than high** for frontier reasoning models. `EI_in = 0.32` derives from a median consumer text prompt on a heavily optimized serving path; the framework cites up to 30x for reasoning variants.
- **Keep training visible as its own term** inside the total, and note its lifetime-token denominator is undisclosed, with a stated range of 1e13–1e15 (two orders of magnitude either way).
- **Say the serving region is unknown** when it is, and that no market-based adjustment was applied.
- **Note that embodied is charged on all tokens**, including cache reads, which likely overstates it.
- **Note the measurement is not final while the session is open** — generating the report adds tokens to the session it measures.
- **Never populate the energy-saved tile** without paired empirical data.
- **Label emissions as CO2e**, never as a bare mass.
- If asked for a number the data cannot support, say what disclosure would be needed instead of guessing. Per-token cache-hit energy intensity is the highest-value ask.

## DSL design spec

Encode directly; do not ask for branding details.

- **Colors:** DSL Navy `#1B2A6B`, DSL Orange `#C2551F`, near-black `#222222`, light grey `#F2F2F2`, muted `#999999`, rule grey `#D9D9D9`, placeholder navy `#4A5891`
- **Font:** Arial. Register Liberation Sans (`/usr/share/fonts/truetype/liberation2/LiberationSans-{Regular,Bold,Italic}.ttf`), which is metric-compatible. Fall back to Helvetica. **`registerFont` alone is not enough** — you must also call `pdfmetrics.registerFontFamily(...)`, or every `<b>` and `<i>` tag in the document silently renders as regular weight.
- **Page:** US Letter, 1-inch left/right margins
- **Header:** the real DSL logo, top-left, per the Logo handling steps below. If the logo genuinely cannot be located, fall back to the typographic wordmark — "DYNAMIC SUSTAINABILITY LAB" navy bold 9pt, "Syracuse University" orange 7.5pt beneath — and print a one-line console note so the fallback is visible rather than silent. Either way, an orange 1.6pt rule runs the full header width beneath it.
- **Tables:** navy header row with white bold text, light-grey zebra striping, hairline grid, no dark fills in data cells
- **Section headings:** navy 13pt bold numbered; subheadings orange 11pt bold
- **KPI band:** navy tiles, white text, orange separators. Five tiles at 6.5in leaves ~1.3in each, so labels are passed **already upper-cased and with explicit `<br/>` break points** — `kpi_band` must not call `.upper()`, which would mangle the tag, and an unbroken label like "INFERENCE-SPECIFIC" will otherwise split mid-word. Label 7pt. Values render the number at 14pt with the unit inline at 9pt (`_kpi()`), which is what keeps "5.30 kg CO2e" on one line. Placeholder tiles use placeholder navy with a 10pt italic value.
- **Footer:** grey hairline, report name + tier + generator version left, page number right
- **Never use Unicode sub/superscripts** in ReportLab — use `<sub>` and `<super>` tags

### Logo handling

Every other DSL deliverable (the memo and technical-bulletin skills) carries the real logo, not a typed-out lab name — treat the logo here as required too, and fall back to text only if it truly cannot be found.

1. **Search in this order** for the source asset (a ZIP archive despite the `.pdf` extension):
   - `$DSL_LOGO_PDF` (shared project asset — the common case)
   - `<memo skill dir>/assets/DSL_Memo_Logo.pdf`, if the memo skill is installed alongside this one
   - Any `DSL_Memo_Logo.pdf` or `DSL*Logo*.pdf` uploaded to the current session
   If none exist, skip straight to the typographic fallback and note it in the console output — do not block the report on a missing logo.
2. **Extract and rasterize:**
   ```bash
   unzip -o DSL_Memo_Logo.pdf -d /tmp/dsl_logo_extracted
   ```
   (If it turns out to be a genuine PDF rather than a disguised ZIP, use `pdftoppm -r 150 -png DSL_Memo_Logo.pdf /tmp/dsl_logo` instead — try `unzip` first, fall back to `pdftoppm` on failure.)
3. **Crop white margins and remove the white background**, same thresholds as the memo skill:
   ```python
   from PIL import Image
   import numpy as np
   img = Image.open('/tmp/dsl_logo_extracted/1.jpeg').convert('RGBA')
   arr = np.array(img, dtype=np.uint8)
   r, g, b = arr[:,:,0].astype(int), arr[:,:,1].astype(int), arr[:,:,2].astype(int)
   nonwhite = (r + g + b) < 750
   rows, cols = np.any(nonwhite, axis=1), np.any(nonwhite, axis=0)
   rmin, rmax = np.where(rows)[0][[0, -1]]
   cmin, cmax = np.where(cols)[0][[0, -1]]
   img = img.crop((max(0, cmin-20), max(0, rmin-20), cmax+20, rmax+20))
   arr = np.array(img, dtype=np.uint8)
   r, g, b, a = arr[:,:,0].astype(int), arr[:,:,1].astype(int), arr[:,:,2].astype(int), arr[:,:,3].copy()
   near_white = (r > 230) & (g > 230) & (b > 230)
   a[near_white] = 0
   aa = (r > 200) & (r <= 230) & (g > 200) & (g <= 230) & (b > 200) & (b <= 230)
   for row, col in zip(*np.where(aa)):
       dist = ((255-int(r[row,col]))**2 + (255-int(g[row,col]))**2 + (255-int(b[row,col]))**2) ** 0.5
       a[row, col] = int(255 * min(dist / ((55**2*3) ** 0.5), 1.0))
   arr[:,:,3] = a
   Image.fromarray(arr).save('/tmp/dsl_logo_clean.png')
   ```
4. **Place it in the header** top-left at `x = 1 inch`, `y = letter[1] - 0.80 * inch`, height ≈ 0.42 inch with width scaled to the source aspect ratio, drawn via `canvas.drawImage(..., preserveAspectRatio=True, mask='auto')` inside `deco()`. The orange rule still runs beneath it at the same position as the typographic version.
5. **Visually confirm** the logo reads correctly against the white header band during the page-render check before presenting the PDF.

## Everyday-life equivalents

The report converts **one** figure into everyday terms: the **inference-specific electricity**. Not
the total, not the emissions. That is the number the reader is being asked to build intuition about,
and converting several figures at once buries it.

The reference table lives in `ELEC_EQUIV` near the top of the generator. Every entry is a published
figure or an explicitly-flagged first-principles calculation, and Appendix B carries the citations.

Two rules govern their use:

- **Magnitude filtering is mandatory.** `pick_equivalents()` keeps only references whose count lands
  between 0.5 and 500. A comparison yielding 0.00006 of something, or forty million of something,
  conveys nothing and is dropped rather than printed. This is why the table spans four orders of
  magnitude — the selector picks whatever fits the session at hand.
- **Equivalences are conversion aids, not findings.** They inherit every parameter uncertainty in
  the report and add their own, since each reference is itself an average over a population of
  devices or households. The report states this; keep it.

Factor vintages: EPA equivalencies rest on a 2022 data year (eGRID2022); the EIA residential average
is 2024. If either is refreshed upstream, update the constant and the Appendix B citation together.

Do not use EPA's marginal "electricity avoided" factor (672 gCO2/kWh) for consumption accounting —
that rate is only appropriate for efficiency or renewable-displacement claims.

## Generator source

Write this verbatim to `/tmp/dsl_ai_energy_report.py`.

```python
#!/usr/bin/env python3
"""
DSL AI Inference Energy & Emissions Report
Framework: Watershed, "Estimating GHG Emissions from AI Use" (June 2026).
Activity Tier equations p.22; Table 2 defaults pp.23-24; fn.48 per-token limits;
fn.54 cache-hit disaggregation; Detailed worked example pp.29-30.
"""
import argparse, glob, hashlib, json, os, re, sys
from datetime import datetime, timezone
from reportlab.lib import colors
from reportlab.lib.enums import TA_LEFT
from reportlab.lib.pagesizes import letter
from reportlab.lib.utils import ImageReader
from reportlab.lib.styles import ParagraphStyle, getSampleStyleSheet
from reportlab.lib.units import inch
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.platypus import (BaseDocTemplate, Frame, PageBreak, PageTemplate,
                                Paragraph, Spacer, Table, TableStyle)

VERSION = "1.0"
DSL_NAVY = colors.HexColor("#1B2A6B")
DSL_ORANGE = colors.HexColor("#C2551F")
NEAR_BLACK = colors.HexColor("#222222")
LIGHT_GREY = colors.HexColor("#F2F2F2")
MUTED = colors.HexColor("#999999")
RULE_GREY = colors.HexColor("#D9D9D9")
PLACEHOLDER_NAVY = colors.HexColor("#4A5891")

ARIAL_CANDIDATES = [
    ("/usr/share/fonts/truetype/liberation2/LiberationSans-Regular.ttf",
     "/usr/share/fonts/truetype/liberation2/LiberationSans-Bold.ttf",
     "/usr/share/fonts/truetype/liberation2/LiberationSans-Italic.ttf"),
    ("/usr/share/fonts/truetype/liberation/LiberationSans-Regular.ttf",
     "/usr/share/fonts/truetype/liberation/LiberationSans-Bold.ttf",
     "/usr/share/fonts/truetype/liberation/LiberationSans-Italic.ttf"),
]
FONT, FONT_B, FONT_I = "Helvetica", "Helvetica-Bold", "Helvetica-Oblique"
for reg, bold, ital in ARIAL_CANDIDATES:
    if os.path.exists(reg) and os.path.exists(bold):
        pdfmetrics.registerFont(TTFont("DSLArial", reg))
        pdfmetrics.registerFont(TTFont("DSLArial-Bold", bold))
        if os.path.exists(ital):
            pdfmetrics.registerFont(TTFont("DSLArial-Italic", ital)); FONT_I = "DSLArial-Italic"
        FONT, FONT_B = "DSLArial", "DSLArial-Bold"
        pdfmetrics.registerFontFamily("DSLArial", normal="DSLArial", bold="DSLArial-Bold",
                                      italic=FONT_I, boldItalic="DSLArial-Bold")
        break

DEFAULTS = {
    "hyperscaler": {"ei_in": 0.32, "ei_out": 0.96, "pue": 1.10, "emb": 0.017, "train": 0.10},
    "unknown":     {"ei_in": 0.70, "ei_out": 2.10, "pue": 1.56, "emb": 0.025, "train": 0.50},
}
GRID_DEFAULT = 341.0
CACHE_FACTOR_DEFAULT = 0.10
REASONING_UPLIFT = 3.0


def discover_transcripts():
    home = os.path.expanduser("~")
    pats = [os.path.join(home, "mnt", ".claude", "projects", "*", "*.jsonl"),
            os.path.join(home, ".claude", "projects", "*", "*.jsonl")]
    found = []
    for p in pats:
        found.extend(glob.glob(p))
    return sorted(set(found), key=lambda f: os.path.getmtime(f), reverse=True)


def parse_transcript(path):
    """One row per unique model call, deduped by requestId. Assistant messages
    are recorded twice with identical usage; requestId is the dedupe key."""
    rows, seen, tool_calls = [], set(), 0
    with open(path, "r", errors="replace") as fh:
        for line in fh:
            line = line.strip()
            if not line:
                continue
            try:
                e = json.loads(line)
            except json.JSONDecodeError:
                continue
            m = e.get("message")
            if not isinstance(m, dict):
                continue
            c = m.get("content")
            if isinstance(c, list):
                for b in c:
                    if isinstance(b, dict) and b.get("type") == "tool_use":
                        tool_calls += 1
            u = m.get("usage")
            if not u:
                continue
            rid = e.get("requestId") or m.get("id")
            if rid in seen:
                continue
            seen.add(rid)
            rows.append({"rid": rid, "ts": e.get("timestamp", ""),
                         "cw": int(u.get("cache_creation_input_tokens", 0) or 0),
                         "cr": int(u.get("cache_read_input_tokens", 0) or 0),
                         "out": int(u.get("output_tokens", 0) or 0),
                         "inp": int(u.get("input_tokens", 0) or 0),
                         "think": int((u.get("output_tokens_details") or {}).get("thinking_tokens", 0) or 0)})
    with open(path, "rb") as fh:
        digest = hashlib.sha256(fh.read()).hexdigest()[:16]
    return rows, {"path": path, "session_id": os.path.splitext(os.path.basename(path))[0],
                  "sha256_16": digest, "bytes": os.path.getsize(path),
                  "tool_calls": tool_calls // 2 if tool_calls else 0}


def electricity_wh(cw, cr, inp, out, ei_in, ei_out, pue, cache_factor, mult=1.0):
    fresh_k, cached_k, out_k = (cw + inp) / 1000.0, cr / 1000.0, out / 1000.0
    return (fresh_k * ei_in + cached_k * ei_in * cache_factor + out_k * ei_out) * pue * mult


def totals(rows):
    t = {"cw": 0, "cr": 0, "out": 0, "inp": 0, "think": 0}
    for r in rows:
        for k in t:
            t[k] += r[k]
    t["total"] = t["cw"] + t["cr"] + t["out"] + t["inp"]
    t["mtok"] = t["total"] / 1e6
    return t


def scenarios(t, P):
    U = DEFAULTS["unknown"]; f = P["cache_factor"]
    defs = [("A", "Hyperscaler defaults + cache credit (central)", P["ei_in"], P["ei_out"], P["pue"], f, 1.0),
            ("B", "Hyperscaler defaults, no cache credit", P["ei_in"], P["ei_out"], P["pue"], 1.0, 1.0),
            ("C", "Unknown-provider defaults + cache credit", U["ei_in"], U["ei_out"], U["pue"], f, 1.0),
            ("D", "Unknown-provider defaults, no cache credit", U["ei_in"], U["ei_out"], U["pue"], 1.0, 1.0),
            ("E", "Central x %.0f frontier-reasoning uplift" % P["uplift"], P["ei_in"], P["ei_out"], P["pue"], f, P["uplift"]),
            ("F", "Central, cache credit %.2f instead of %.2f" % (min(0.25, f * 2.5), f), P["ei_in"], P["ei_out"], P["pue"], min(0.25, f * 2.5), 1.0)]
    out = []
    for key, lab, a, b, c, d, m in defs:
        wh = electricity_wh(t["cw"], t["cr"], t["inp"], t["out"], a, b, c, d, m)
        out.append((key, lab, wh, wh / 1000.0 * P["grid"]))
    return out


def diagnostics(rows, P):
    per = []
    for i, r in enumerate(rows, 1):
        wh = electricity_wh(r["cw"], r["cr"], r["inp"], r["out"], P["ei_in"], P["ei_out"], P["pue"], P["cache_factor"])
        per.append({"n": i, **r, "wh": wh, "g": wh / 1000.0 * P["grid"]})
    expiry = []
    for i in range(1, len(per)):
        prev, cur = per[i - 1], per[i]
        if cur["cr"] < prev["cr"] * 0.9 and cur["cw"] > 10000:
            wasted = electricity_wh(cur["cw"], 0, 0, 0, P["ei_in"], P["ei_out"], P["pue"], P["cache_factor"])
            expiry.append({"n": cur["n"], "cw": cur["cw"], "wh": wasted})
    return per, expiry


# ---------------------------------------------------------------------------
# Everyday-life equivalents for the inference-specific electricity figure.
# Every value is a published figure or an explicitly-flagged first-principles
# calculation; Appendix B carries the citations. Order does not matter -- the
# selector picks whichever land at a sensible magnitude for the session.
# ---------------------------------------------------------------------------
ELEC_EQUIV = [
    ("full smartphone charges", 19.0,
     "EPA GHG Equivalencies Calculator: 0.019 kWh per full charge (US DOE 2024)"),
    ("full laptop battery charges", 53.8,
     "Apple MacBook Air 13-inch (M5): 53.8 Wh battery; excludes charging losses"),
    ("hours of a 10 W LED bulb", 10.0,
     "ENERGY STAR: 800 lumens at >= 80 lm/W for a 60 W-equivalent lamp"),
    ("litres of water boiled in an electric kettle", 110.0,
     "First principles: 93 Wh of heat (20-100 C) at ~85% kettle efficiency; not a published factor"),
    ("miles driven by an average US electric vehicle", 277.8,
     "EPA/DOE: 3.60 miles per kWh (US EV sales, 2019 and earlier)"),
    ("days of household refrigeration", 2329.0,
     "EIA RECS 2020: 2.9 MBtu per household per year for refrigeration (existing stock)"),
    ("days of an average US household's total electricity", 28400.0,
     "EIA 2024: 863 kWh per month, US residential average"),
]


def fmt_wh(wh):
    """Compact electricity string for body text."""
    if wh >= 1000.0:
        return "%.2f kWh" % (wh / 1000.0)
    return "%.0f Wh" % wh


def fmt_g(g):
    """Compact CO2e mass string for body text. Always carries the unit."""
    if g >= 1000.0:
        return "%.2f kg CO2e" % (g / 1000.0)
    return "%.0f g CO2e" % g


def _kpi(value, unit):
    """KPI tile value: number at full size, unit smaller on the same line. Keeps
    a five-tile band from wrapping while still stating the unit explicitly."""
    return '%s <font size="9">%s</font>' % (value, unit)


def fmt_kpi_wh(wh):
    if wh >= 1000.0:
        return _kpi("%.2f" % (wh / 1000.0), "kWh")
    return _kpi("%.0f" % wh, "Wh")


def fmt_kpi_g(g):
    """Emissions tiles are always labelled CO2e -- never a bare mass."""
    if g >= 1000.0:
        return _kpi("%.2f" % (g / 1000.0), "kg CO2e")
    return _kpi("%.0f" % g, "g CO2e")


def fmt_count(n):
    if n >= 100:
        return "%.0f" % n
    if n >= 10:
        return "%.0f" % n
    if n >= 1:
        return "%.1f" % n
    return "%.2f" % n


def pick_equivalents(total, table, lo=0.5, hi=500.0, limit=5):
    """Keep only references that land at a legible magnitude for this session.
    A comparison yielding 0.00002 of something, or 40 million of something, tells
    the reader nothing, so both tails are dropped rather than printed."""
    out = []
    for label, per_unit, source in table:
        if per_unit <= 0:
            continue
        n = total / per_unit
        if lo <= n <= hi:
            out.append((n, label, source))
    out.sort(key=lambda r: -r[0])
    return out[:limit]


def styles():
    ss = getSampleStyleSheet(); S = {}
    S["title"] = ParagraphStyle("t", parent=ss["Normal"], fontName=FONT_B, fontSize=20, leading=24, textColor=DSL_NAVY, spaceAfter=2)
    S["sub"] = ParagraphStyle("s", parent=ss["Normal"], fontName=FONT, fontSize=11, leading=14, textColor=NEAR_BLACK, spaceAfter=2)
    S["h1"] = ParagraphStyle("h1", parent=ss["Normal"], fontName=FONT_B, fontSize=13, leading=16, textColor=DSL_NAVY, spaceBefore=16, spaceAfter=6)
    S["h2"] = ParagraphStyle("h2", parent=ss["Normal"], fontName=FONT_B, fontSize=11, leading=14, textColor=DSL_ORANGE, spaceBefore=10, spaceAfter=4)
    S["body"] = ParagraphStyle("b", parent=ss["Normal"], fontName=FONT, fontSize=10, leading=13.5, textColor=NEAR_BLACK, spaceAfter=6, alignment=TA_LEFT)
    S["note"] = ParagraphStyle("n", parent=ss["Normal"], fontName=FONT_I, fontSize=8.5, leading=11, textColor=MUTED, spaceBefore=2, spaceAfter=6)
    S["kpi_lab"] = ParagraphStyle("kl", parent=ss["Normal"], fontName=FONT, fontSize=7, leading=8.6, textColor=colors.white)
    S["kpi_val"] = ParagraphStyle("kv", parent=ss["Normal"], fontName=FONT_B, fontSize=14, leading=17, textColor=colors.white)
    S["kpi_val_ph"] = ParagraphStyle("kvp", parent=ss["Normal"], fontName=FONT_I, fontSize=10, leading=13, textColor=colors.white)
    S["cell"] = ParagraphStyle("c", parent=ss["Normal"], fontName=FONT, fontSize=8.5, leading=11, textColor=NEAR_BLACK)
    S["cellh"] = ParagraphStyle("ch", parent=ss["Normal"], fontName=FONT_B, fontSize=8.5, leading=11, textColor=colors.white)
    return S


def data_table(header, body, widths, S, align_right=None, zebra=True):
    align_right = align_right or []
    rows = [[Paragraph(h, S["cellh"]) for h in header]]
    for r in body:
        rows.append([Paragraph(str(c), S["cell"]) for c in r])
    t = Table(rows, colWidths=widths, repeatRows=1, hAlign="LEFT")
    cmds = [("BACKGROUND", (0, 0), (-1, 0), DSL_NAVY), ("GRID", (0, 0), (-1, -1), 0.4, RULE_GREY),
            ("BOX", (0, 0), (-1, -1), 0.6, DSL_NAVY), ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
            ("TOPPADDING", (0, 0), (-1, -1), 3), ("BOTTOMPADDING", (0, 0), (-1, -1), 3),
            ("LEFTPADDING", (0, 0), (-1, -1), 5), ("RIGHTPADDING", (0, 0), (-1, -1), 5)]
    for c in align_right:
        cmds.append(("ALIGN", (c, 1), (c, -1), "RIGHT"))
    if zebra:
        for i in range(1, len(rows)):
            if i % 2 == 0:
                cmds.append(("BACKGROUND", (0, i), (-1, i), LIGHT_GREY))
    t.setStyle(TableStyle(cmds)); return t


def kpi_band(items, S, width):
    """items are (label, value) or (label, value, is_placeholder). Labels are
    passed already upper-cased so they may contain <br/> to control where a long
    label breaks; upper-casing here would mangle the tag. Placeholder tiles get a
    lighter fill and italic text so an unpopulated metric can never be mistaken
    for a measured zero."""
    cells, w = [], width / len(items)
    ph_cols = []
    for i, item in enumerate(items):
        lab, val = item[0], item[1]
        is_ph = len(item) > 2 and item[2]
        if is_ph:
            ph_cols.append(i)
        vstyle = S["kpi_val_ph"] if is_ph else S["kpi_val"]
        inner = Table([[Paragraph(lab, S["kpi_lab"])], [Paragraph(val, vstyle)]], colWidths=[w - 10])
        inner.setStyle(TableStyle([("TOPPADDING", (0, 0), (-1, -1), 1), ("BOTTOMPADDING", (0, 0), (-1, -1), 1),
                                   ("LEFTPADDING", (0, 0), (-1, -1), 0), ("RIGHTPADDING", (0, 0), (-1, -1), 0)]))
        cells.append(inner)
    t = Table([cells], colWidths=[w] * len(items), hAlign="LEFT")
    cmds = [("BACKGROUND", (0, 0), (-1, -1), DSL_NAVY), ("VALIGN", (0, 0), (-1, -1), "MIDDLE"),
            ("TOPPADDING", (0, 0), (-1, -1), 8), ("BOTTOMPADDING", (0, 0), (-1, -1), 8),
            ("LEFTPADDING", (0, 0), (-1, -1), 8), ("RIGHTPADDING", (0, 0), (-1, -1), 5),
            ("LINEAFTER", (0, 0), (-2, -1), 1.2, DSL_ORANGE)]
    for c in ph_cols:
        cmds.append(("BACKGROUND", (c, 0), (c, 0), PLACEHOLDER_NAVY))
    t.setStyle(TableStyle(cmds))
    return t


def build_pdf(out_path, rows, meta_list, P, args):
    S = styles(); t = totals(rows); sc = scenarios(t, P); per, expiry = diagnostics(rows, P)
    wh_c, g_op = sc[0][2], sc[0][3]
    kwh_c = wh_c / 1000.0
    wh_infer = wh_c / P["pue"] if P["pue"] else wh_c
    kwh_infer = wh_infer / 1000.0
    g_emb = P["emb"] * t["mtok"] * 1000.0
    g_train = P["train"] * t["mtok"] * 1000.0
    g_sub = g_op + g_emb
    g_total = g_op + g_emb + g_train
    g_infer = wh_infer / 1000.0 * P["grid"]
    mtok_fresh = (t["cw"] + t["inp"] + t["out"] + t["cr"] * P["cache_factor"]) / 1e6
    g_emb_disc = P["emb"] * mtok_fresh * 1000.0

    doc = BaseDocTemplate(out_path, pagesize=letter, leftMargin=inch, rightMargin=inch,
                          topMargin=0.9 * inch, bottomMargin=0.85 * inch,
                          title="DSL AI Inference Energy and Emissions Report",
                          author=args.author, subject=args.label)
    CW = doc.width

    def deco(canvas, d):
        canvas.saveState()
        logo_path = getattr(args, "logo", None)
        drew_logo = False
        if logo_path and os.path.exists(logo_path):
            try:
                img = ImageReader(logo_path)
                iw, ih = img.getSize()
                logo_h = 0.42 * inch
                logo_w = logo_h * (iw / float(ih))
                canvas.drawImage(img, inch, letter[1] - 0.80 * inch, width=logo_w, height=logo_h,
                                 preserveAspectRatio=True, mask="auto")
                drew_logo = True
            except Exception:
                drew_logo = False
        if not drew_logo:
            canvas.setFillColor(DSL_NAVY); canvas.setFont(FONT_B, 9)
            canvas.drawString(inch, letter[1] - 0.62 * inch, "DYNAMIC SUSTAINABILITY LAB")
            canvas.setFillColor(DSL_ORANGE); canvas.setFont(FONT, 7.5)
            canvas.drawString(inch, letter[1] - 0.75 * inch, "Syracuse University")
        canvas.setStrokeColor(DSL_ORANGE); canvas.setLineWidth(1.6)
        canvas.line(inch, letter[1] - 0.82 * inch, letter[0] - inch, letter[1] - 0.82 * inch)
        canvas.setStrokeColor(RULE_GREY); canvas.setLineWidth(0.5)
        canvas.line(inch, 0.72 * inch, letter[0] - inch, 0.72 * inch)
        canvas.setFillColor(MUTED); canvas.setFont(FONT, 7.5)
        canvas.drawString(inch, 0.57 * inch, "AI Inference Energy and Emissions Report  |  "
                          "Watershed Activity Tier (Detailed)  |  generator v%s" % VERSION)
        canvas.drawRightString(letter[0] - inch, 0.57 * inch, "Page %d" % d.page)
        canvas.restoreState()

    doc.addPageTemplates([PageTemplate(id="dsl", frames=[Frame(doc.leftMargin, doc.bottomMargin, doc.width, doc.height, id="f")], onPage=deco)])
    st = []; A = st.append

    A(Paragraph("AI Inference Energy and Emissions Report", S["title"]))
    A(Paragraph("Watershed Activity Tier (Detailed) &mdash; first-party token measurement", S["sub"]))
    A(Spacer(1, 10))
    scope = ("%d sessions (roll-up)" % len(meta_list)) if len(meta_list) > 1 else "Single session"
    hdr = [["Subject", args.label], ["Prepared by", args.author],
           ["Report date", datetime.now().strftime("%d %B %Y")],
           ["Scope", "%s | %d model calls | %s tokens" % (scope, len(rows), format(t["total"], ","))],
           ["Framework", "Watershed, Estimating GHG Emissions from AI Use (June 2026)"],
           ["Parameter set", P["provider"] + (" (overridden)" if P["overrides"] else " defaults")]]
    ht = Table([[Paragraph("<b>%s</b>" % k, S["cell"]), Paragraph(v, S["cell"])] for k, v in hdr],
               colWidths=[1.35 * inch, CW - 1.35 * inch], hAlign="LEFT")
    ht.setStyle(TableStyle([("VALIGN", (0, 0), (-1, -1), "TOP"), ("TOPPADDING", (0, 0), (-1, -1), 2),
                            ("BOTTOMPADDING", (0, 0), (-1, -1), 2), ("LEFTPADDING", (0, 0), (-1, -1), 0),
                            ("LINEBELOW", (0, 0), (-1, -2), 0.3, RULE_GREY)]))
    A(ht); A(Spacer(1, 14))

    A(kpi_band([("TOTAL GHG<br/>EMISSIONS", fmt_kpi_g(g_total)),
                ("INFERENCE-SPECIFIC<br/>EMISSIONS", fmt_kpi_g(g_infer)),
                ("TOTAL ELECTRICITY<br/>USED", fmt_kpi_wh(wh_c)),
                ("INFERENCE-SPECIFIC<br/>ELECTRICITY", fmt_kpi_wh(wh_infer)),
                ("ENERGY SAVED,<br/>SUSTAINABLE AI USE", "Not yet<br/>measured", True)], S, CW))
    A(Paragraph("Central estimate (Scenario A). <b>Total GHG emissions</b> combine operational "
                "electricity, amortized embodied hardware and amortized training. Training is the "
                "least knowable of the three and is broken out in section 1 &mdash; read the total with "
                "that in mind. <b>Inference-specific emissions</b> are the operational emissions of the "
                "compute alone (%s), excluding overhead, hardware and training. <b>Total electricity</b> "
                "is the full facility draw including data-centre overhead at PUE = %.2f; "
                "<b>inference-specific electricity</b> strips that overhead out, leaving %.0f%% of the "
                "total. <b>Energy saved through sustainable AI use</b> is a deliberate placeholder: it "
                "will be populated once DSL has paired measurements of refined against unrefined AI-use "
                "practice, and is left blank rather than estimated. Carbon intensity: %.0f gCO2e/kWh."
                % (fmt_g(g_infer), P["pue"], 100.0 * wh_infer / wh_c if wh_c else 0, P["grid"]), S["note"]))

    e_elec = pick_equivalents(wh_infer, ELEC_EQUIV)
    if e_elec:
        A(Paragraph("<b>What the inference electricity looks like in everyday terms.</b> The %s of "
                    "inference-specific electricity above is roughly equivalent to any one of these:"
                    % fmt_wh(wh_infer), S["body"]))
        A(data_table(["Quantity", "Everyday equivalent"],
                     [[fmt_count(n), lab] for n, lab, _src in e_elec],
                     [0.8 * inch, CW - 0.8 * inch], S, [0]))
        A(Paragraph("Only references that land at a legible magnitude for this session are shown; "
                    "comparisons that would read as a rounding error, or as an implausibly large "
                    "multiple, are dropped rather than printed. Factors: EPA Greenhouse Gas "
                    "Equivalencies Calculator (smartphone charge, EV efficiency), ENERGY STAR (LED "
                    "efficacy), Apple published battery capacity (laptop), EIA (household "
                    "refrigeration and total household electricity); kettle energy is a "
                    "first-principles calculation at ~85% efficiency, not a published factor. Full "
                    "citations in Appendix B. These are conversion aids, not additional findings "
                    "&mdash; they inherit every uncertainty in section 6 and add their own, since each "
                    "reference is itself an average over a population of devices or households.",
                    S["note"]))

    A(Paragraph("1. Key numbers", S["h1"]))
    A(data_table(["Metric", "Central estimate"], [
        ["Total electricity used (incl. data-centre overhead)", "%.3f kWh (%.1f Wh)" % (kwh_c, wh_c)],
        ["Inference-specific electricity (compute only, pre-PUE)", "%.3f kWh (%.1f Wh)" % (kwh_infer, wh_infer)],
        ["Operational emissions (from total electricity)", "%.2f gCO2e" % g_op],
        ["Inference-specific emissions (compute only)", "%.2f gCO2e" % g_infer],
        ["Amortized embodied hardware", "%.2f gCO2e" % g_emb],
        ["Subtotal (operational + embodied)", "%.2f gCO2e" % g_sub],
        ["Amortized training (highest-uncertainty term)", "%.2f gCO2e" % g_train],
        ["Total GHG emissions (operational + embodied + training)", "%.2f gCO2e" % g_total],
        ["Energy saved through sustainable AI use", "Not yet measured &mdash; see note above"],
        ["Functional unit", "%.1f gCO2e per million tokens (excl. training)" % (g_sub / t["mtok"] if t["mtok"] else 0)],
        ["Electricity intensity", "%.1f Wh per million tokens" % (wh_c / t["mtok"] if t["mtok"] else 0)],
        ["Mean per model call", "%.2f Wh / %.2f gCO2e" % (wh_c / len(rows), g_sub / len(rows))],
    ], [3.15 * inch, CW - 3.15 * inch], S, [1]))

    A(Paragraph("2. Measured token volumes", S["h1"]))
    A(Paragraph("Measured, not estimated. Counts are read directly from session transcript usage "
                "records and deduplicated by request identifier.", S["body"]))
    pct = lambda x: ("%.2f%%" % (100.0 * x / t["total"])) if t["total"] else "0%"
    A(data_table(["Token class", "Tokens", "Share", "Treatment"], [
        ["Cache read (key-value reuse)", format(t["cr"], ","), pct(t["cr"]), "charged at %.2f x EI_in" % P["cache_factor"]],
        ["Cache write (fresh prefill)", format(t["cw"], ","), pct(t["cw"]), "charged at full EI_in"],
        ["Output / decode", format(t["out"], ","), pct(t["out"]), "charged at EI_out (incl. %s thinking)" % format(t["think"], ",")],
        ["Fresh uncached input", format(t["inp"], ","), pct(t["inp"]), "charged at full EI_in"],
        ["Total", format(t["total"], ","), "100%", "%.3f million tokens" % t["mtok"]],
    ], [2.1 * inch, 1.0 * inch, 0.7 * inch, CW - 3.8 * inch], S, [1, 2]))

    A(Paragraph("3. Scenario bounds", S["h1"]))
    A(Paragraph("Every scenario below uses the same measured token volumes. The spread is entirely "
                "attributable to parameter uncertainty, which is the source framework's central argument.", S["body"]))
    whs = [s[2] for s in sc]
    A(data_table(["#", "Scenario", "Wh", "kWh", "gCO2e (op.)"],
                 [[k, l, "%.2f" % w, "%.4f" % (w / 1000.0), "%.2f" % g] for k, l, w, g in sc],
                 [0.3 * inch, CW - 2.75 * inch, 0.75 * inch, 0.85 * inch, 0.85 * inch], S, [2, 3, 4]))
    A(Paragraph("Range: %.1f to %.1f Wh, a spread of %.1fx." % (min(whs), max(whs), max(whs) / min(whs) if min(whs) else 0), S["note"]))

    A(Paragraph("4. Diagnostics and reduction levers", S["h1"]))
    reread = electricity_wh(0, t["cr"], 0, 0, P["ei_in"], P["ei_out"], P["pue"], P["cache_factor"])
    A(Paragraph("Context re-reading", S["h2"]))
    A(Paragraph("%s of the %s tokens processed (%.1f%%) were cache reads: standing instructions and "
                "prior conversation re-read on each call rather than new material. Under the central "
                "scenario this accounts for %.1f Wh, or %.1f%% of total electricity. Session cost "
                "therefore scales with the number of model calls more than with the amount of work "
                "performed, so fewer and denser calls are cheaper than many short ones."
                % (format(t["cr"], ","), format(t["total"], ","),
                   100.0 * t["cr"] / t["total"] if t["total"] else 0, reread,
                   100.0 * reread / wh_c if wh_c else 0), S["body"]))
    A(Paragraph("Cache-expiry events", S["h2"]))
    if expiry:
        A(data_table(["Call #", "Tokens re-prefilled", "Wh", "Share of session"],
                     [[e["n"], format(e["cw"], ","), "%.2f" % e["wh"], "%.1f%%" % (100.0 * e["wh"] / wh_c if wh_c else 0)] for e in expiry],
                     [0.8 * inch, 1.6 * inch, 0.8 * inch, CW - 3.2 * inch], S, [1, 2, 3]))
        A(Paragraph("A cache-expiry event occurs when the prompt cache lapses between calls and the "
                    "entire standing context must be recomputed from scratch. This is compute spent "
                    "re-reading material already read, and it is avoidable by reducing idle gaps "
                    "within a working session.", S["note"]))
    else:
        A(Paragraph("No cache-expiry events detected. All calls after the first reused a warm cache.", S["body"]))
    A(Paragraph("Most expensive calls", S["h2"]))
    A(data_table(["Call #", "Cache write", "Cache read", "Output", "Wh", "gCO2e"],
                 [[r["n"], format(r["cw"], ","), format(r["cr"], ","), format(r["out"], ","),
                   "%.3f" % r["wh"], "%.3f" % r["g"]] for r in sorted(per, key=lambda r: -r["wh"])[:5]],
                 [0.65 * inch, 1.05 * inch, 1.0 * inch, 0.8 * inch, 0.8 * inch, CW - 4.3 * inch], S, [1, 2, 3, 4, 5]))

    A(Paragraph("5. Method", S["h1"]))
    A(Paragraph("Tier placement", S["h2"]))
    A(Paragraph("This estimate sits between the framework's Activity and Provider Tiers. Token volumes "
                "are exact first-party counts, which is better than the Activity Tier assumes; energy "
                "intensity, PUE and grid intensity are modelled from published defaults, which is what "
                "keeps it below the Provider Tier. A true Provider Tier estimate would require per-token "
                "electricity intensity published by the model provider, which is not currently available "
                "for frontier models.", S["body"]))
    A(Paragraph("Equations", S["h2"]))
    A(Paragraph("Electricity (Wh) = (I<sub>fresh</sub> &times; EI<sub>in</sub> + I<sub>cached</sub> &times; "
                "EI<sub>in</sub> &times; f<sub>cache</sub> + O &times; EI<sub>out</sub>) &times; PUE<br/>"
                "Operational emissions (gCO2e) = Electricity &divide; 1000 &times; GI<br/>"
                "Embodied emissions (gCO2e) = E<sub>emb</sub> &times; MTok &times; 1000<br/>"
                "Training emissions (gCO2e, separate) = E<sub>train</sub> &times; MTok &times; 1000", S["body"]))
    A(Paragraph("Token volumes are expressed in thousands; EI factors are Wh per 1,000 tokens. "
                "Dividing the electricity figure by PUE isolates the compute-only draw &mdash; reported "
                "as &ldquo;inference-specific electricity&rdquo; in the KPI band and Key numbers above, distinct "
                "from the PUE-inclusive &ldquo;total electricity used.&rdquo;", S["note"]))
    A(Paragraph("Cache disaggregation", S["h2"]))
    A(Paragraph("The framework's base equation applies a single EI_in to all input tokens. Footnote 54 "
                "permits disaggregating the prefill term where cache-hit token counts are exposed, because "
                "cached input reuses key-value states computed in a prior request. Footnote 48 warns that a "
                "blended factor overstates repeat-context workloads. Agentic sessions are exactly that case, "
                "so this report applies a cache credit of %.2f and reports the un-credited result as "
                "Scenario B." % P["cache_factor"], S["body"]))

    A(Paragraph("6. Parameters and assumptions", S["h1"]))
    ov = P["overrides"]
    st_of = lambda k: "override" if k in ov else "default"
    A(data_table(["Symbol", "Parameter", "Value", "Source", "Status"], [
        ["EI_in", "Energy intensity, input (prefill)", "%.3f Wh / 1,000 tok" % P["ei_in"], "Watershed Table 2; Elsworth et al. 2025", st_of("ei_in")],
        ["EI_out", "Energy intensity, output (decode)", "%.3f Wh / 1,000 tok" % P["ei_out"], "Watershed Table 2; Patel et al. 2024 (3:1)", st_of("ei_out")],
        ["f_cache", "Cached input as fraction of EI_in", "%.2f" % P["cache_factor"], "Proxy from cache-read pricing discount; not a measured energy ratio", st_of("cache_factor")],
        ["PUE", "Power usage effectiveness", "%.2f" % P["pue"], "Watershed Table 2", st_of("pue")],
        ["GI", "Grid carbon intensity (location-based)", "%.0f gCO2e/kWh" % P["grid"], "eGRID 2024 U.S. national average", st_of("grid")],
        ["E_emb", "Amortized embodied emissions", "%.3f kgCO2e / MTok" % P["emb"], "Open Compute Project method; H100 at 164 kgCO2e/card", st_of("emb")],
        ["E_train", "Amortized training emissions", "%.2f kgCO2e / MTok" % P["train"], "10,000 tCO2e over an assumed 1e14 lifetime tokens", st_of("train")],
    ], [0.6 * inch, 1.55 * inch, 1.15 * inch, CW - 4.15 * inch, 0.85 * inch], S))

    A(Paragraph("Limitations, ordered by severity", S["h2"]))
    lims = [
        "<b>Parameter spread dominates.</b> The scenario range is %.1fx on identical measured token "
        "volumes. Any single figure in this report implies more precision than the underlying data "
        "supports." % (max(whs) / min(whs) if min(whs) else 0),
        "<b>Defaults may not describe the model measured.</b> EI_in of 0.32 Wh/1,000 tokens derives from "
        "a median consumer text prompt on a heavily optimized serving path. Frontier reasoning models "
        "with extended thinking are materially more intensive; the source framework cites up to 30x for "
        "reasoning variants. The central estimate is therefore more likely low than high.",
        "<b>Training is the largest and least knowable term.</b> At %.1f gCO2e it %s the operational and "
        "embodied terms combined, and rests on a lifetime-token denominator no frontier provider "
        "discloses. The framework's stated range of 1e13 to 1e15 lifetime tokens moves this term between "
        "%.1f and %.1f gCO2e." % (g_train, "exceeds" if g_train > g_sub else "does not exceed", g_train / 10, g_train * 10),
        "<b>Serving region unknown.</b> A national-average grid factor is used in place of a regional "
        "one, and no market-based adjustment is applied. U.S. subregions vary more than five-fold in "
        "carbon intensity, so the regional figure could differ substantially.",
        "<b>The cache credit is a price proxy, not a measurement.</b> f_cache = %.2f is taken from "
        "commercial pricing, which correlates with but does not equal energy. Per-token cache-hit energy "
        "intensity is the single most valuable disclosure that would tighten this estimate." % P["cache_factor"],
        "<b>Embodied emissions are charged on all tokens.</b> Cached tokens occupy less accelerator time "
        "than fresh ones, so charging E_emb flat per token likely overstates this term. Discounting "
        "embodied by the same cache factor would give %.2f gCO2e instead of %.2f gCO2e." % (g_emb_disc, g_emb),
        "<b>Equivalences are illustrative.</b> The everyday comparisons under the headline numbers "
        "convert this report's own estimates; they cannot be more accurate than those estimates, and "
        "each reference factor is itself a population average with its own vintage.",
        "<b>Measurement is not final while the session is open.</b> Generating this report adds tokens to "
        "the session it measures. Figures are current as of generation time only.",
    ]
    for i, L in enumerate(lims, 1):
        A(Paragraph("%d. %s" % (i, L), S["body"]))

    A(Paragraph("7. Back-end evidence", S["h1"]))
    A(Paragraph("Every figure in this report derives from the files below. A reviewer can re-run the "
                "generator against the same transcripts and reproduce it exactly.", S["body"]))
    A(data_table(["Session identifier", "Bytes", "SHA-256 (first 16)", "Tool calls"],
                 [[m["session_id"][:18], format(m["bytes"], ","), m["sha256_16"], str(m["tool_calls"])] for m in meta_list],
                 [2.1 * inch, 0.9 * inch, 1.7 * inch, CW - 4.7 * inch], S, [1, 3]))
    A(Spacer(1, 6))
    A(data_table(["Provenance", "Detail"], [
        ["Deduplication key", "requestId (assistant messages are recorded twice per call; both copies carry identical usage)"],
        ["Model calls counted", str(len(rows))],
        ["Generator version", "dsl_ai_energy_report.py v%s" % VERSION],
        ["Generated (UTC)", datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%SZ")],
        ["Reviewer action", "Verify the parameter table in section 6 before citing these figures"],
    ], [1.6 * inch, CW - 1.6 * inch], S))

    if not args.no_ledger:
        A(PageBreak())
        A(Paragraph("Appendix A. Per-turn ledger", S["h1"]))
        A(Paragraph("One row per model call, in order. Wh and gCO2e use the central scenario. This table "
                    "is what makes cache-expiry events and cost drift visible; a session total alone "
                    "conceals them inside the average.", S["body"]))
        lrows = []
        for r in per:
            flag = "cache expiry" if any(e["n"] == r["n"] for e in expiry) else ""
            lrows.append([r["n"], format(r["cw"], ","), format(r["cr"], ","), format(r["out"], ","),
                          format(r["cw"] + r["cr"] + r["out"] + r["inp"], ","), "%.3f" % r["wh"], "%.3f" % r["g"], flag])
        W = [0.32 * inch, 0.85 * inch, 0.85 * inch, 0.62 * inch, 0.85 * inch, 0.62 * inch, 0.68 * inch]
        A(data_table(["#", "Cache write", "Cache read", "Output", "Total", "Wh", "gCO2e", "Flag"],
                     lrows, W + [CW - sum(W)], S, [1, 2, 3, 4, 5, 6]))
        A(Spacer(1, 6))
        A(data_table(["Session total", "Cache write", "Cache read", "Output", "Total", "Wh", "gCO2e"],
                     [["", format(t["cw"], ","), format(t["cr"], ","), format(t["out"], ","),
                       format(t["total"], ","), "%.2f" % wh_c, "%.2f" % g_op]],
                     [CW - sum(W[1:]) - 0.0] and [CW - (sum(W) - W[0]), W[1], W[2], W[3], W[4], W[5], W[6]],
                     S, [1, 2, 3, 4, 5, 6], zebra=False))

    A(Paragraph("Appendix B. Source citation", S["h1"]))
    A(Paragraph("<b>Primary framework.</b> Watershed. <i>Estimating GHG Emissions from AI Use: Framework "
                "for Corporate-Level Measurement.</i> June 2026. Activity Tier equations, p. 22; Table 2 "
                "default factors, pp. 23&ndash;24; footnote 48 on the limits of constant per-token factors; "
                "footnote 54 on cache-hit disaggregation; Detailed Activity Tier worked example, "
                "pp. 29&ndash;30.", S["body"]))
    A(Paragraph("<b>Equivalence factors.</b> US EPA, <i>Greenhouse Gas Equivalencies Calculator "
                "&mdash; Calculations and References</i> (eGRID2022 data vintage; smartphone charge "
                "and EV efficiency). "
                "US EIA, <i>Electric Sales, Revenue, and Average Price</i>, Table 5.a, 2024 (residential "
                "average 863 kWh/month) and <i>RECS 2020</i>, Table CE3.1 (household refrigeration). "
                "EPA ENERGY STAR lamp specification (LED efficacy) and Apple published battery capacity "
                "(laptop charge). Kettle energy is a first-principles calculation, not a published "
                "factor, and is labelled as such beneath the headline numbers.", S["body"]))

    doc.build(st)
    return {"wh": wh_c, "kwh": kwh_c, "wh_infer": wh_infer, "g_op": g_op, "g_infer": g_infer,
            "g_emb": g_emb, "g_sub": g_sub, "g_train": g_train, "g_total": g_total,
            "totals": t, "expiry": expiry, "n_calls": len(rows),
            "spread": max(whs) / min(whs) if min(whs) else 0}


def slugify_author(author):
    """First name/segment of --author, lowercased and hyphenated, for use in filenames."""
    s = (author or "").split(",")[0].strip().lower()
    s = re.sub(r"[^a-z0-9]+", "-", s).strip("-")
    return s or "report"


def default_filename(author, all_sessions):
    """DSL-AI-Energy-Report[-Project]-<author-slug>-<YYYYMMDD-HHMM>.pdf so every report's
    name records who produced it and when, and never silently collides with a prior run."""
    ts = datetime.now().strftime("%Y%m%d-%H%M")
    tag = "DSL-AI-Energy-Report-Project" if all_sessions else "DSL-AI-Energy-Report"
    return "%s-%s-%s.pdf" % (tag, slugify_author(author), ts)


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--transcript"); ap.add_argument("--all-sessions", action="store_true")
    ap.add_argument("--out", default=None, help="Full output path. If omitted, a name is "
                     "generated automatically: DSL-AI-Energy-Report-<author-slug>-<timestamp>.pdf, "
                     "written under --outdir.")
    ap.add_argument("--outdir", default=".", help="Directory for the auto-generated filename "
                     "(ignored if --out is given).")
    ap.add_argument("--no-ledger", action="store_true")
    ap.add_argument("--logo", default=None, help="Path to a cleaned, transparent DSL logo PNG "
                     "(see Logo handling). Falls back to the typographic wordmark if omitted "
                     "or the file doesn't exist.")
    ap.add_argument("--label", default="AI inference energy and emissions accounting")
    ap.add_argument("--author", default="Dynamic Sustainability Lab")
    ap.add_argument("--provider", choices=["hyperscaler", "unknown"], default="hyperscaler")
    for k in ["ei-in", "ei-out", "pue", "grid", "cache-factor", "emb", "train", "reasoning-uplift"]:
        ap.add_argument("--" + k, type=float)
    args = ap.parse_args()

    if args.transcript:
        paths = [args.transcript]
    else:
        found = discover_transcripts()
        if not found:
            sys.exit("No session transcripts found under ~/mnt/.claude/projects/")
        paths = found if args.all_sessions else [found[0]]

    rows, meta_list = [], []
    for p in paths:
        r, m = parse_transcript(p)
        if r:
            rows.extend(r); meta_list.append(m)
    if not rows:
        sys.exit("No usage records found in the selected transcript(s).")

    out_path = args.out or os.path.join(args.outdir, default_filename(args.author, args.all_sessions))

    d = DEFAULTS[args.provider]
    P = {"provider": args.provider, "ei_in": d["ei_in"], "ei_out": d["ei_out"], "pue": d["pue"],
         "emb": d["emb"], "train": d["train"], "grid": GRID_DEFAULT,
         "cache_factor": CACHE_FACTOR_DEFAULT, "uplift": REASONING_UPLIFT, "overrides": []}
    for cli, key in [("ei_in", "ei_in"), ("ei_out", "ei_out"), ("pue", "pue"), ("grid", "grid"),
                     ("cache_factor", "cache_factor"), ("emb", "emb"), ("train", "train"),
                     ("reasoning_uplift", "uplift")]:
        v = getattr(args, cli, None)
        if v is not None:
            P[key] = v; P["overrides"].append(key)

    if not (args.logo and os.path.exists(args.logo)):
        print("  [logo] no usable logo at --logo; header falls back to typographic wordmark")

    res = build_pdf(out_path, rows, meta_list, P, args); t = res["totals"]
    print("Report written: %s" % out_path)
    print("  model calls        %d" % res["n_calls"])
    print("  tokens             %s  (cw %s | cr %s | out %s | in %s)"
          % (format(t["total"], ","), format(t["cw"], ","), format(t["cr"], ","),
             format(t["out"], ","), format(t["inp"], ",")))
    print("  electricity        %.2f Wh (%.5f kWh)" % (res["wh"], res["kwh"]))
    print("  inference elec     %.2f Wh" % res["wh_infer"])
    print("  operational        %.2f gCO2e" % res["g_op"])
    print("  inference emis     %.2f gCO2e" % res["g_infer"])
    print("  embodied           %.2f gCO2e" % res["g_emb"])
    print("  subtotal           %.2f gCO2e" % res["g_sub"])
    print("  training (sep.)    %.2f gCO2e" % res["g_train"])
    print("  TOTAL GHG          %.2f gCO2e" % res["g_total"])
    print("  scenario spread    %.1fx" % res["spread"])
    print("  cache-expiry events %d" % len(res["expiry"]))


if __name__ == "__main__":
    main()
```

## Worked invocation

Let the generator name the file — do not hand it a fixed `--out` filename. It builds
`DSL-AI-Energy-Report-<author-slug>-<YYYYMMDD-HHMM>.pdf` from `--author` and the current
time, so the filename itself always shows who produced the report and when:

```bash
python3 /tmp/dsl_ai_energy_report.py \
  --outdir "$OUTPUTS" \
  --label "Session energy and emissions accounting" \
  --author "Name, Dynamic Sustainability Lab" \
  --logo /tmp/dsl_logo_clean.png
```

Roll up every session, drop the ledger, and model a cleaner serving region (the filename
gets a `-Project-` tag automatically for `--all-sessions` runs):

```bash
python3 /tmp/dsl_ai_energy_report.py --all-sessions --no-ledger --grid 282 \
  --outdir "$OUTPUTS" \
  --author "Name, Dynamic Sustainability Lab" \
  --logo /tmp/dsl_logo_clean.png
```

Only pass `--out <exact path>` if the person asks for a specific filename — otherwise the
auto-generated name is the naming protocol for this report.

Omit `--logo` (or point it at a nonexistent path) to force the typographic-wordmark fallback.

## Known framework gap worth raising

The paper's `E_emb` and `E_train` terms are charged flat per token, with no cache adjustment, while `EI_in` can be disaggregated. For agentic workloads where 85–95% of tokens are cache reads, this makes embodied emissions rival or exceed operational emissions — an artifact of the asymmetry, not a physical finding. The report flags this in the limitations and shows the cache-discounted alternative. This is a legitimate methodological comment to send back to the framework authors.

## What this skill does not do

- It does not produce Provider Tier figures. Those require provider disclosure that does not exist for frontier models.
- It does not measure energy. It applies published intensity factors to measured token counts.
- It does not measure energy saved. That tile stays a placeholder until DSL collects paired practice data.
- It does not benchmark against global or national energy totals. A single session is too small for that comparison to mean anything.
- It does not track calendar time, cost in dollars, or task-level productivity.
