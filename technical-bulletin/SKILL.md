---
name: technical-bulletin
description: >
  Use this skill to scaffold and generate a DSL technical bulletin as a .pptx file.
  Triggers include: "create a technical bulletin", "scaffold a TB", "new bulletin",
  "start a technical bulletin", "technical bulletin for [topic]", or any request to
  begin, plan, structure, or produce a DSL technical bulletin for any project or topic.
  Also trigger when a researcher says they want to communicate findings, report results,
  or publish a DSL deliverable in slide-deck format. This skill runs an intake sequence,
  confirms structure with the researcher, produces a markdown scaffold, then generates
  a branded .pptx file using DSL visual standards. Always runs all four stages in order.
---
> **Logo asset resolution (plugin migration note).** This skill now ships its own copy of
> the DSL logo PDF(s) under this skill's `assets/` folder rather than assuming they live in
> project knowledge. Resolve the path(s) once, before first use, checking the bundled asset
> first and falling back to the legacy project-knowledge location:
>
> ```bash
> DSL_LOGO_PDF=$(ls "./assets/DSL_Memo_Logo.pdf" 2>/dev/null || ls "assets/DSL_Memo_Logo.pdf" 2>/dev/null || echo "/mnt/project/DSL_Memo_Logo.pdf")
> DSL_WHITE_LOGO_PDF=$(ls "./assets/DSL_White_Logo_OnNavy.pdf" 2>/dev/null || ls "assets/DSL_White_Logo_OnNavy.pdf" 2>/dev/null || echo "/mnt/project/DSL_White_Logo_OnNavy.pdf")
> ```
>
> **⚠ Unverified:** the exact path a plugin-bundled asset resolves to at runtime (e.g. whether
> it's relative to a working directory, or exposed via an environment variable such as
> `CLAUDE_PLUGIN_ROOT`) has not been confirmed against current Claude plugin documentation.
> Test this resolution step once the plugin is actually installed, before relying on it —
> the `/mnt/project/` fallback should keep things working in the meantime for any fellow who
> still has the logo in project knowledge.



Last modified: 2026-08-28

# Technical Bulletin Skill

Produces a DSL technical bulletin in two passes: first a markdown scaffold the researcher
reviews, then a branded .pptx file built to DSL visual standards. Claude populates
structural placeholders only — no prose content, findings, or data interpretations.

---

## Overview

Four stages:

1. **Intake** — gather project context through the intake questions
2. **Structure derivation** — determine section sequence and slide count from intake;
   confirm with researcher before proceeding
3. **Scaffold output** — produce a markdown scaffold for researcher review
4. **PPTX generation** — build the branded .pptx from the confirmed scaffold

---

## Provenance note (read before using anything below)

Every color, font size, position, and slide layout in this version was extracted
directly from `202608_DSLPresentationTemplate_v02.potx` — the DSL's official
PowerPoint template (Aug 2026), the authoritative source per the DSL Brand Guide
(`202608_DSLBrandGuide_v01.pdf`) — by unzipping the file and reading its theme,
slide master, and slide XML. This **replaces** the previous version of this skill,
which was reverse-engineered from a single published deck
(`07_08_DSL_Institutional_AI_Deck.pptx`) that predates the new brand guide and template.
Almost everything changed: colors, the cover/section-title layouts, and the slide
catalog itself (the old deck had no Contributors, Executive Summary, Five Key
Findings, Table of Contents, Glossary, Quote, or References slides — the new
template has all of these as standard, reusable layouts).

**As of 2026-08-27, this skill no longer depends on the .potx file being present
at runtime.** Every design value that mattered — the nine-color palette, the
Verdana type scale, the Five Fixed Zones grid, and the full 27-slide layout
catalog with exact positions and sizes — was already extracted and is fully
encoded as static text throughout this skill file. The .potx was only still
load-bearing for one thing: live extraction of the two logo image assets. That
dependency has been removed — see Logo Handling below, which now sources both
logo variants from the project's standalone PDF assets
(`$DSL_LOGO_PDF`, `$DSL_WHITE_LOGO_PDF`),
neither of which requires the .potx to be present in the current session. A
same-named `.potx` file exists on the DSLAI SharePoint at
`Shared Documents/DSL_Templates/DSL_Templates/202608_DSLPresentationTemplate_v02.potx`,
but Microsoft Graph refuses to convert it for text extraction (likely a
sensitivity/Purview label) — this is no longer an operational blocker for this
skill, only a historical note on where the original values came from. If the
source template is ever revised, someone will need to re-supply the `.potx`
directly in a chat session to re-extract updated values; that is a future
maintenance event, not a per-run requirement.

### Provenance note — Logo Handling correction (2026-08-28)

A live test build (a 14-slide DMP-explainer deck, 2026-08-27/28) surfaced three
defects in the Logo Handling section below, all now corrected in this version:

1. **`DSL_White_Logo_OnNavy.pdf` was documented as a genuine PDF.** It is not —
   it is the same disguised-ZIP archive format as `DSL_Memo_Logo.pdf`. The
   previous version of this skill instructed extraction via `pdfimages`, which
   fails outright against this file (`Syntax Warning: May not be a PDF file`,
   `Couldn't find trailer dictionary`). `unzip` succeeds and is now the
   documented first attempt for both logo assets, with `pdftoppm`/`pdfimages`
   as the fallback if a future version of either asset really is a genuine PDF.
2. **The extracted white-knockout raster carries a spurious solid white band**
   baked into its bottom edge, distinct from both the logo mark and the navy
   background — see Known Inconsistencies below. Chroma-keying without
   trimming it first produces an opaque white bar over navy or teal in the
   final deck. The extraction steps now trim this band before chroma-keying.
3. **The cover/divider corner-logo bleed position and size were calibrated for
   the old template's ~3.2:1 logo asset.** Applied literally to this asset's
   actual ~5.83:1 aspect ratio, the spec'd height produces a logo roughly
   10.6" wide — wide enough to overlap the photo panel on cover/divider
   variants that have one. Sizing and placement now caps the corner logo's
   width instead of deriving it from the spec'd height, and positions it just
   inside the corner rather than bleeding off it (see Sizing and placement
   below for the specifics and the reasoning).

None of these change what the logo looks like once correctly placed — they
were pipeline defects, not palette or asset changes.

---

## Known inconsistencies in the source template (flag, do not silently fix)

These were confirmed by reading the `.potx` XML directly when it was available in
a prior session. They remain valid, static facts about the template's baked assets
and do not need to be re-verified against a live copy of the `.potx` to keep using
them.

- **Logo asset orange does not match the new palette.** Both embedded logo PNGs
  originally found inside the template (`ppt/media/image1.png` — standard
  black+orange; `ppt/media/image8.png` — white knockout) use an orange around
  `#F76900` baked into the raster art, not either of the new guide's approved
  oranges (`#E67A2F` SU Orange, `#EE7623` DSL Orange). This is the *old* deck's
  orange, carried over into the new template's logo asset. Per the Logo Misuse
  rules (see Brand Reference below), never recolor a logo to "fix" this — flag it
  to the researcher/admin as a template asset that needs updating at the source.
  This same inconsistency is very likely present in the project's own
  `DSL_Memo_Logo.pdf` and `DSL_White_Logo_OnNavy.pdf` assets, since those predate
  the new brand guide too — visually confirm during the QA pass (Stage 4) rather
  than assuming either pipeline produces an on-palette orange.
- **The "Stay Connected" slide (39) also hardcodes the old orange.** Its
  "STAY CONNECTED" label text is set to `#F76900` directly rather than the theme's
  `accent1` (`#E67A2F`). Reproduce the template as built when generating this
  slide (don't silently swap in the theme color), but flag the same inconsistency.
- **Slide 7's logo guidance says to download the logo `.png` from Microsoft
  Teams**, which conflicts with this project's own asset pipeline (`DSL_Memo_Logo.pdf`,
  `DSL_White_Logo_OnNavy.pdf` in `/mnt/project/`). This skill now uses that
  project asset pipeline exclusively (see Logo Handling below) — the Teams-download
  instruction from the template is not followed.
- **The `DSL_White_Logo_OnNavy.pdf` source raster carries a spurious solid white
  band baked into its bottom edge** (confirmed 2026-08-27: roughly the bottom 10%
  of the raster, a flat near-`#FFFFFF` block distinct from both the logo mark and
  the navy fill above it — not a JPEG compression artifact, a genuine feature of
  the stored asset). Left untrimmed, chroma-keying the navy background leaves
  this band fully opaque, so it renders as a stray white bar across the bottom of
  the knockout logo wherever it's placed. Trim it before chroma-keying (see Logo
  Handling below) rather than shipping it. This is a defect in the stored asset,
  not something this skill should silently paper over at the source — flag it to
  the researcher/admin the same way as the other asset-level inconsistencies here.
- Both of these are the template's own inconsistencies, not something this skill
  should "correct" on its own initiative — surface them, let the researcher/admin
  decide whether to fix the source template.

---

## DSL Brand Reference

Encode these values directly — do not ask the researcher for branding details.

### Layout
- Format: `LAYOUT_WIDE` (13.333" × 7.5") — confirmed from the template's
  `<p:sldSz cx="12192000" cy="6858000"/>` (EMU), which converts to exactly
  13.333" × 7.5". This is the same as PowerPoint's standard 16:9 widescreen
  ratio — the two are not actually in conflict (a point the previous version of
  this skill got wrong by implying `LAYOUT_WIDE` was something other than 16:9).

### Colors — nine approved hex values, no others
Per the template's `theme1.xml` (`<a:clrScheme name="DSL Color Palette">`) and
guidance slide 5 ("Color Palette and Design Guidelines"), which names every swatch:

| Role | Hex | Theme slot | Tier |
|---|---|---|---|
| SU Orange | `#E67A2F` | accent1 | Primary |
| DSL Teal | `#246F83` | accent2 | Primary |
| DSL Gray | `#262625` | dk1 | Primary |
| DSL Green | `#779514` | accent3 | Secondary |
| DSL Tan | `#D7C8B5` | accent4 | Secondary |
| White | `#FFFFFF` | lt1 / lt2 | Secondary |
| DSL Orange | `#EE7623` | accent5 | Tertiary |
| Black | `#000000` | dk2 | Tertiary |
| SU Navy | `#00205C` | accent6 / hyperlink | Tertiary |

No colors outside this list may appear anywhere in a bulletin. Data-visualization
palettes (for charts with more than one series) must come from Color Brewer, not
be invented.

**60/30/10 rule:** one color should hold ~60% of a slide — usually White or DSL
Gray. Give ~30% to DSL Teal or DSL Tan. Keep SU Orange/DSL Orange to the ~10%
accent that directs the eye.

**Contrast rule:** SU Orange and DSL Orange on White fail WCAG 2.0 AA for body
text. Use orange for large display type, rules, and fills — never for small body
text on white. Set small text in DSL Gray instead.

### Typography
- **Font: Verdana only**, confirmed in the template's `fontScheme` (`majorFont`
  and `minorFont` both Verdana) — matches the DSL Brand Guide, which reserves
  Calibri for Word documents only. Never mix Verdana and Calibri in a deck.
- **Presentation type scale** (from guidance slide 6, "Verdana at Presentation Scale"):

| Element | Size | Weight |
|---|---|---|
| Cover title | 40–44 pt | Bold |
| Slide title | 26–28 pt | Bold |
| Body text (plain paragraphs) | 14–16 pt | Regular |
| Bullet lists | 12–16 pt | Regular |
| Table text | 11–14 pt | Regular |
| Captions & sources | 9–10 pt | Regular |
| Stat callouts | 54–66 pt | Bold |

- Verdana sets wide — titles that look small in points will read large on
  screen. Resist inflating sizes beyond this table unless there's a specific
  reason.
- Kicker label (see Five Fixed Zones below): 9.5 pt, bold, SU Orange, all-caps.
- **Long titles need room reserved for wrapping.** A cover title (40 pt) or
  divider/section title (36 pt) can run to two or three lines depending on the
  researcher's actual topic wording — the box heights and the vertical position
  of whatever sits beneath the title (subtitle, presenter block, explanation
  line) must be sized and positioned for the *rendered* line count, not a
  single-line assumption. When building a slide, estimate the wrap (roughly:
  Verdana bold at cover-title scale fits about 16–20 characters per line at the
  cover text-block width used in variants A–C) and reserve height for up to
  three lines before placing anything below it. Confirmed defect (2026-08-27
  test build): a cover title box sized for one line collided with the subtitle
  directly beneath it once the actual title wrapped to three lines. Check this
  during the visual QA pass on every cover and divider slide, not just once
  during development.

### The Five Fixed Zones (grid — from guidance slide 8)
Every content slide (not covers or dividers) follows this same zone structure.
Violating it is the most common way a generated slide will look off-brand even
with the right colors:

1. **Safe margin** — maintain margins on every layout so content never touches
   the edge or disappears when projected.
2. **Kicker** — 9.5 pt bold SU Orange, all-caps, positioned at roughly
   x: 0.68", y: 0.42". Names the *section*, not the individual slide (e.g. all
   slides within a "Findings" section carry the kicker "FINDINGS", not a
   per-slide label).
3. **Title** — one line, 26–28 pt bold DSL Gray, eight words or fewer,
   positioned at roughly x: 0.68", y: 0.92", width ~11.97".
4. **Content field** — two to three columns with 0.3–0.4" gutters. Leave the
   bottom inch of the slide clear where possible.
5. **Footer** — logo bottom-left (already placed via the slide master — do not
   move it), slide number bottom-right. See Logo Handling and Slide Number below.

**Global content rule:** one idea per slide. If a slide needs more than about 40
words of body copy (excluding figures/diagrams), it should probably be split into
two slides.

### Logo Handling

Two logo variants are needed: a **standard logo** (black text + orange icon, for
white/light backgrounds) and a **white knockout logo** (for dark/navy/photo
backgrounds). This skill no longer extracts either one from the `.potx` — both
are built at runtime from the project's standalone PDF assets, neither of which
depends on the presentation template being present.

**Both project logo files are disguised ZIP archives despite the `.pdf`
extension, not genuine PDFs** — this was previously documented correctly for
`DSL_Memo_Logo.pdf` but incorrectly for `DSL_White_Logo_OnNavy.pdf` (the prior
version of this skill called it "a genuine PDF" and pointed straight at
`pdfimages`, which fails against it: `Syntax Warning: May not be a PDF file`,
`Couldn't find trailer dictionary`). Both assets get the same treatment: try
`unzip` first, fall back to `pdftoppm`/`pdfimages` only if `unzip` fails — this
ordering matters, since attempting the PDF-rasterizing tools first on a
disguised ZIP simply fails or produces garbage. Confirmed 2026-08-27: `unzip`
succeeds on both files and extracts an embedded `1.jpeg` from each.

**Standard logo — from `$DSL_LOGO_PDF`:**

```bash
mkdir -p /tmp/dsl_std_logo && cd /tmp/dsl_std_logo
unzip -o $DSL_LOGO_PDF -d extracted \
  || pdftoppm -r 150 -png $DSL_LOGO_PDF /tmp/dsl_std_logo/extracted/raster
find extracted -iname "*.jpeg" -o -iname "*.jpg" -o -iname "*.png"
```

Take the largest image file found (the embedded logo raster), then crop white
margins:

```python
from PIL import Image
import numpy as np
img = Image.open('<path to found image>').convert('RGB')
arr = np.array(img)
brightness = arr[:,:,0].astype(int) + arr[:,:,1].astype(int) + arr[:,:,2].astype(int)
nonwhite = brightness < 750
rows = np.any(nonwhite, axis=1)
cols = np.any(nonwhite, axis=0)
rmin, rmax = np.where(rows)[0][[0, -1]]
cmin, cmax = np.where(cols)[0][[0, -1]]
cropped = img.crop((max(0, cmin-20), max(0, rmin-20), cmax+20, rmax+20))
cropped.save('/tmp/dsl_standard_logo_clean.png')
```

**White knockout logo — from `$DSL_WHITE_LOGO_PDF`:**

Not a genuine PDF — same disguised-ZIP format as `DSL_Memo_Logo.pdf` (see above).
The logo inside it is **pre-composited onto a solid navy fill**, not rendered
with a transparent background, so it still needs chroma-keying after
extraction. Extract with `unzip` first, falling back to `pdfimages` (which pulls
an embedded raster directly, preserving quality) and then `pdftoppm` only if
`unzip` genuinely fails:

```bash
mkdir -p /tmp/dsl_white_logo && cd /tmp/dsl_white_logo
unzip -o $DSL_WHITE_LOGO_PDF -d extracted \
  || pdfimages -png $DSL_WHITE_LOGO_PDF /tmp/dsl_white_logo/extracted/img \
  || pdftoppm -r 150 -png $DSL_WHITE_LOGO_PDF /tmp/dsl_white_logo/extracted/raster
find extracted -iname "*.jpeg" -o -iname "*.jpg" -o -iname "*.png"
```

**Before chroma-keying, trim the spurious white band baked into the bottom of
this raster** (see Known Inconsistencies above — confirmed 2026-08-27, a solid
near-white block distinct from the navy background, roughly the bottom 10% of
the image). Detect it by scanning rows from the bottom for a run of
near-white rows, rather than hardcoding a fixed row offset, since the exact
extent could shift if the source asset is ever re-exported:

```python
from PIL import Image
import numpy as np

orig = Image.open('<path to found image>').convert('RGB')
arr = np.array(orig)

# Detect and trim a spurious near-white band at the bottom of the raster,
# distinct from the navy background and the logo mark. Scan upward from the
# last row; a row counts as part of the band only if nearly every pixel in it
# is near-white (this rules out ordinary anti-aliased logo edges, which are
# only white over a small fraction of any given row).
def is_near_white_row(row, frac_thresh=0.98, val_thresh=245):
    return (row.min(axis=-1) > val_thresh).mean() > frac_thresh

h = arr.shape[0]
cut = h
for r in range(h - 1, -1, -1):
    if is_near_white_row(arr[r]):
        cut = r
    else:
        break
if cut < h:
    print(f"Trimming spurious white band: rows {cut}-{h-1} of {h}")
    arr = arr[:cut, :, :]

img = Image.fromarray(arr).convert('RGBA')
```

Then chroma-key out the navy fill so the logo can sit over photos or the DSL Teal
panel, not just literal navy. Sample the background color from a corner pixel
rather than hardcoding a guessed hex, since the exact navy shade in this asset
hasn't been independently confirmed against the theme's `#00205C`:

```python
arr2 = np.array(img, dtype=np.uint8)
bg = arr2[2, 2, :3].astype(int)  # sample the fill from a near-corner pixel
dist = np.sqrt(((arr2[:, :, :3].astype(int) - bg) ** 2).sum(axis=2))
# soft-edged threshold: fully transparent close to the sampled fill color,
# fully opaque past a wider distance, linear ramp between the two
lo, hi = 15.0, 60.0
alpha = np.clip((dist - lo) / (hi - lo) * 255.0, 0, 255).astype(np.uint8)
arr2[:, :, 3] = alpha
img2 = Image.fromarray(arr2)

# crop to the non-transparent content bounding box
alpha_mask = arr2[:, :, 3] > 10
rows = np.any(alpha_mask, axis=1)
cols = np.any(alpha_mask, axis=0)
rmin, rmax = np.where(rows)[0][[0, -1]]
cmin, cmax = np.where(cols)[0][[0, -1]]
img2.crop((max(0, cmin-20), max(0, rmin-20), cmax+20, rmax+20)).save('/tmp/dsl_white_logo_clean.png')
```

This is an approximation, not a true alpha channel — the source asset was never
transparent to begin with, so edge pixels along the logo's outline may carry a
faint navy fringe after chroma-keying. **Always visually inspect the result
before embedding** (see the visual-confirmation step below); if the fringe is
visible at slide scale, tighten `lo`/`hi` and re-run rather than shipping it.

**Do not re-color either extracted logo** to match the new palette's oranges
even though the baked-in orange is almost certainly the old `#F76900` (see Known
Inconsistencies above) — that would violate the Logo Misuse rules below. Flag it
instead.

**Sizing and placement, confirmed from the template's own layout XML (these
positions are unaffected by which extraction pipeline produced the pixels):**
- **Footer logo** (every standard content slide, placed via the slide master —
  do not move or resize it): standard logo, height ~1.086", positioned bleeding
  off the left edge at x: −0.484", y: 6.68" (width scaled to whatever the
  extracted asset's actual aspect ratio is — don't assume a fixed ~3.2:1, since
  that figure came from the old `.potx`-embedded PNGs and the new extraction
  pipeline may not reproduce it exactly).
- **Cover / section-title-with-photo slides:** white knockout logo, spec height
  ~1.818" tall, bleeding off the top-left corner at roughly x: −0.337", y: −0.197"
  (width likewise scaled to the extracted asset's actual aspect ratio) — **but
  cap the width instead of deriving it from this height whenever the derived
  width would exceed roughly 3.0"–3.5" or would overlap another slide element
  (the photo panel on variants with one, or the title/subtitle text block).**
  Confirmed 2026-08-27: this asset's actual aspect ratio (~5.83:1) is
  substantially flatter/wider than the old template-embedded PNGs the 1.818"
  figure was calibrated against (~3.2:1), so applying the height literally
  produces a logo roughly 10.6" wide — enough to overlap the photo panel on
  every cover/divider variant that has one. When the width is capped, derive
  height from the capped width instead, and position the logo just inside the
  corner (small positive x/y offset, e.g. ~0.2"–0.25" from each edge) rather
  than at the spec'd negative-offset bleed position — the bleed math assumed
  the old asset's proportions and clips through the wordmark at this asset's
  proportions instead of reading as a deliberate bleed. Confirmed present on
  all three cover variants (slides 13–15) and two of the three divider variants
  (slides 23–24, both of which have photos).
- **Divider variant with no photo (slide 25 style):** no separate logo placement
  was found on this variant in the source template — the teal panel occupies the
  full left side with just the number and title/subtitle on the right. Do not
  invent a logo placement for this variant; flag it to the researcher if they
  pick this style and want a logo added.
- **Minimum size (per guidance slide 7):** never below 1" wide on screen; 1.25"
  is the target minimum.
- **Clear space:** equivalent space on all sides. Always place over a solid
  field or a calm (uncluttered) area of a photo — never over a busy image area.
  This matters more with the chroma-keyed white logo than it did with the
  template's native transparent PNG, since a busy background will show through
  any imperfectly-keyed edge pixels more visibly.

**Visual confirmation is mandatory, not optional, for this pipeline.** Because
neither extracted asset is a verified-transparent RGBA source, render both logos
against a sample of their intended background (white for the standard logo, the
DSL Teal fill and a sample photo for the white knockout) before using them in the
actual deck, and inspect at typical slide-viewing scale. This replaces the
`.potx` pipeline's simpler visual-inspection step, which only had to confirm
correct placement, not confirm the transparency itself. This check is also what
would have caught the spurious white band and the oversized corner-logo bleed
before either reached a delivered deck — treat this step as covering asset
integrity, not just placement.

### Logo Misuse — never do these (per guidance slide 11 and slide 7)
- Recolor, outline, or add effects to the logo mark
- Stretch, rotate, or crop it
- Set it on a busy photo or a low-contrast field
- Rebuild it by typing the wordmark in Verdana instead of using the image asset

### Imagery — Rights Checklist (from guidance slide 9)
- Use DSL and Syracuse University photography first
- Avoid stock photography unless unavoidable
- If stock is unavoidable, use royalty-free sources only — Pixabay, Magnific,
  Unsplash — and cite them
- Check licensing on any government-published photography, video, or data
  visualization before using it
- Add alt text to every image (required for web publication)

**Photography style:** active, inviting, and unexpected. Show diverse
individuals and groups doing the work, not posed for it — bold, boundless,
welcoming, transformational.

**Image border/shadow convention** (confirmed independently on guidance slide 9,
consistent with the same rule already in `memo-creator-skill.md`): for any
floating photograph placed on a slide, apply Shape Effects → Shadow → Outer →
"Offset: Bottom." This applies to photographs, not to the DSL logo (which must
never carry a drop shadow — see Logo Misuse above) and not to full-bleed cover
photos.

### Accessibility Checklist (from guidance slide 10)
- Body text never below 12 pt
- Test the deck's legibility from the back of the room
- Confirm every text/background color pair clears WCAG 2.0 AA
- Never carry meaning in color alone
- Alt text on all images and charts
- Check reading order in PowerPoint's Selection pane
- Run PowerPoint's built-in Accessibility Checker (Review ▸ Check Accessibility)
  before sending or presenting the deck — this applies to Claude's generated
  output too; note in the scaffold footer that this manual step still needs to
  happen before publishing
- Describe charts aloud when presenting them (a delivery note for the
  researcher, not something Claude can encode into the file)

### Presentation Recommendations (from guidance slide 11)

**Do:**
- Write titles that state a finding, not just a topic
- Leave white space — empty area is a design choice, not a gap to fill
- Left-align paragraphs and lists
- Vary layouts across the talk: cards, stats, two-column, full-bleed image
- Put the source line directly under the thing it sources
- Reuse the layouts in the template as-is rather than inventing new ones

**Don't:**
- Read a wall of bullets to the room
- Substitute fonts, or mix Verdana with Calibri in the same deck
- Introduce colors outside the nine approved hex values
- Stretch the logo or place it on a busy photo
- Paste a screenshot of a chart or table — insert the actual chart/table object

---

## Slide Catalog (from the official template — 39 slides total)

The template's own "Contents" panel (slide 2) maps the slide ranges as follows.
Slides 1–12 are the template's built-in guidance for a human author and are
**never reproduced in a generated bulletin** — their content has already been
absorbed into the Brand Reference above. Slides 13–39 are the real, reusable
layouts this skill draws from.

| Range | Purpose |
|---|---|
| 13–15 | Cover / title slides (3 layout variants) |
| 16–22 | Introductory slides |
| 23–25 | Dividers (section-title slides, 3 layout variants) |
| 26–34 | Content & stats (9 layout variants) |
| 35–36 | Large images and data visualization (2 layout variants) |
| 37 | Quotation slide |
| 38–39 | Concluding slides (References, Stay Connected) |

All positions below are given in inches from the top-left, confirmed by reading
each slide's XML directly (not estimated).

### Cover slides (13–15) — pick one variant per bulletin

Shared text elements across all three variants:
- Title: 40 pt bold, White, Verdana. **Size the title box for up to three
  wrapped lines, not one** — a realistic bulletin title at this scale can wrap
  to two or three lines depending on wording, and every element positioned
  beneath it (subtitle, presenter block) needs to sit far enough down to clear
  the worst-case wrap rather than a fixed offset calibrated to a short example
  title. See the Typography note above.
- Subtitle (one-sentence topic summary): 13 pt bold, DSL Tan (`#D7C8B5`)
- Presenter block (two lines — "Presenters, Credentials" / "Titles, The Dynamic
  Sustainability Lab · Month Day, Year"): 11 pt bold, White
- Photo credit line (bottom corner, opposite the logo): 9 pt regular, White,
  e.g. "Photo: [credit]"
- White knockout logo, sized and positioned per the Sizing and placement
  guidance above (spec height ~1.818" but width-capped in practice — see that
  section for the corrected figures and reasoning)

**Variant A (slide 13) — full-bleed photo:**
- Full-bleed photo (13.466"×7.5", slightly overscanned to allow the scrim)
- Dark scrim overlay: DSL Gray (`#262625`) at 65% alpha over the whole photo
- Text block anchored bottom-left: title at x: 0.572", y: 4.38" (6.8"×1.65"),
  subtitle below it, presenter block below that
- Logo bleeds off the top-left at x: −0.337", y: −0.197" per spec — apply the
  width cap from Sizing and placement above rather than this position's
  original height-derived width

**Variant B (slide 14) — photo right half:**
- Photo occupies x: 7.923"–13.333" (right ~40%)
- Text block on the left, higher up: title at x: 0.562", y: 3.04" — reserve
  height for up to three wrapped lines before placing the subtitle/presenter
  block beneath it (see Typography note above; this is the variant where the
  collision was actually observed in testing)
- Logo bleeds off the top-left corner (same position as Variant A, same width
  cap applies)

**Variant C (slide 15) — photo left half:**
- Photo occupies x: 0"–9.931" (left ~75%)
- Text block on the right at x: 5.95", y: 3.04" (mirrored from Variant B)
- Logo bleeds off the top-right area at x: 5.048", y: −0.197" — positioned at
  the seam between photo and open space, not the true top-left corner; same
  width cap applies

### Introductory slides (16–22)

**16 — Contributors:** up to four headshot cards in a row, each: circular/square
headshot (2.753"×1.835"), role/title 10 pt bold SU Orange, name+credentials 14 pt
bold DSL Gray, role/contribution description 11 pt regular, email 11 pt bold DSL
Teal.

**17 — About the Dynamic Sustainability Lab:** two-panel layout (each panel
5.82"×4.1"). Left panel: photo placeholder on White. Right panel: DSL Tan
(`#D7C8B5`) fill with the DSL mission boilerplate text, bold DSL Gray. Use the
same boilerplate as the memo/bulletin brand text: "The Dynamic Sustainability
Lab is a non-partisan think tank and research lab that examines the
opportunities, risks, and unintended consequences resulting from the rapid
transition to a new generation of sustainable technologies, strategies, and
policies." Flag as `[VERIFY BEFORE PUBLISHING]` per the usual convention.

**18 — About the Client:** same two-panel geometry as slide 17, but both panels
DSL Tan, or a photo placeholder in one panel — statement text is
researcher-supplied, not boilerplate. Only include this slide if the researcher
named a client/partner during intake.

**19 — Executive Summary:** text column on the left (7.3" wide), photo/data-viz
placeholder card on the right (White fill, 4.67"×4.28"), source line below the
card (9 pt, muted gray `#5C5C5A`).

**20 — Five Key Findings:** five numbered cards in a single row, each DSL Tan
(2.203"×3.1"), numbered circle in SU Orange, heading 13 pt bold DSL Gray, support
sentence 10.5 pt regular muted gray.

**21 — Table of Contents:** numbered rows (01–07 max observed) with thin divider
rules between them — number 17 pt bold SU Orange, section name 14.5 pt bold DSL
Gray, short explanation 11 pt regular muted gray. A DSL Teal side panel
(2.9"×4.36") holds a "Presentation Time" stat (label 9 pt bold white, value
27 pt bold white, e.g. "20 min") — the template notes this panel can be swapped
for a stat or logo instead, or deleted if not needed.

**22 — Glossary of Terms:** same numbered-row structure as the TOC (term 14.5 pt
bold DSL Gray, definition 12 pt regular muted gray), with an optional DSL Tan
side panel for a stat or logo (same swap-or-delete note as slide 21).

### Divider / section-title slides (23–25) — pick one variant per section

**Variant A (slide 23) — full-bleed photo:** same dark-scrim treatment as Cover
Variant A. Section title 36 pt bold White at x: 1.28", y: 2.994" (5.99"×1.3");
subtitle 14 pt regular White below it. Same title-wrap caution as the cover
applies here — reserve room below the title for its worst-case wrap before
placing the subtitle.

**Variant B (slide 24) — photo right half, DSL Teal left panel:** photo at
x: 7.923"–13.333"; DSL Teal (`#246F83`) fill on the left panel (0"–7.945"); same
title/subtitle position, sizing, and wrap caution as Variant A; white knockout
logo bleeding off the top-left, same width cap as the cover logo.

**Variant C (slide 25) — no photo, DSL Teal panel with large number:** DSL Teal
fill on the left ~38% of the slide (0"–5.109"); a large section number (e.g.
"01") at 84 pt bold White; section title 30 pt bold at x: 5.752", y: 2.85"
(color not fixed by a fill in the source — apply DSL Gray, consistent with the
type scale's default body/title color, unless the researcher requests white for
contrast reasons); short explanation 13 pt regular below the title. **No logo
placement was found on this variant** — see the flag under Logo Handling above.

### Content & stats slides (26–34) — nine variants, pick per slide based on content

All nine share the Five Fixed Zones (kicker, title, content field, footer).
Differences are in the content field only:

- **26 — Text left / visual right:** bullets or prose (13.5 pt) in a 7.3"-wide
  left column; photo/data-viz placeholder card (White fill, 4.67"×4.28") on the
  right; source line below the card.
- **27 — Visual left / text right:** mirror of 26.
- **28 — Key-finding sentence + bullets + visual:** a one-sentence key finding
  (16 pt, DSL Teal) directly under the title, bullets below that (13.5 pt), an
  optional accent band at the bottom of the text column (DSL Tan/White fill —
  the template marks this "Optional Band – delete" if unused), and a
  photo/data-viz card on the right.
- **29 — Two-column comparison:** two side-by-side columns (5.767" each), each
  with a colored heading bar (SU Orange for the left column, DSL Teal for the
  right — or swap per content) and up to three bullets, plus a "chart/photo/
  diagram" placeholder card beneath each column.
- **30 — Three-card row:** exactly three cards side by side (3.751" each, DSL
  Tan fill), each numbered (colors rotate SU Orange → DSL Teal → DSL Green),
  heading 14 pt bold DSL Gray, 2–3 lines of support text. **Use this layout only
  for exactly three parallel items** — the template explicitly notes that four
  is tight and five needs the numbered-row layout (31) instead.
- **31 — Numbered horizontal rows:** alternating DSL Tan / White row
  backgrounds, numbered circles alternating SU Orange / DSL Teal, heading +
  support sentence per row, plus a supporting image/diagram card on the right
  side of the slide.
- **32 — 2×2 grid:** four cards (5.787"×1.72" each) in a 2×2 arrangement, numbers
  rotate through SU Orange → DSL Teal → DSL Green → SU Navy, heading 9.5 pt bold
  (colored to match its number) + support sentence 12 pt regular.
- **33 — Five-card row:** same geometry as slide 20 (Five Key Findings) — use
  this variant when a body section (not the intro) needs the same five-across
  card treatment.
- **34 — Three stat callouts:** three columns, each with a colored top rule (SU
  Orange / DSL Teal / DSL Green), a large stat number (54 pt bold — color not
  fixed in the source; leave as a placeholder color or use DSL Gray), a
  measurement label (14 pt bold, colored to match its rule), and supporting
  evidence text (11.5 pt regular). Title text on this slide is White, suggesting
  this variant is intended for a darker background — confirm the researcher's
  intended background treatment before building, since the source slide didn't
  carry an explicit background fill in the extracted XML.

### Large image / data-visualization slides (35–36)

**35 — Native chart + side panel:** an embedded PowerPoint chart object
(7.874"×4.3") on the left, with a DSL Tan side panel (3.623"×4.3") on the right
carrying a heading (9 pt bold DSL Teal) and up to three supporting bullets
(11 pt regular). **Always insert a real chart object here — never paste a
chart image** (this is the template's own explicit instruction, reinforced by
the Presentation Recommendations "don't paste a screenshot" rule above).

**36 — Full-width visual:** a single large placeholder card spanning nearly the
full slide width (11.973"×4.713", White/DSL Tan fill) with a centered caption
inside it and a source line below.

### Quotation slide (37)

Large orange quotation mark (150 pt bold, SU Orange) at the top-left; the quote
itself (25 pt bold, DSL Gray) filling most of the slide width, ending with the
closing quotation mark inline; a short DSL Teal rule beneath it; attribution on
two lines (12 pt regular DSL Gray — name/role, then organization/year). Keep the
quote short enough to read aloud from the back of a room, per the template's own
placeholder instruction.

### Concluding slides (38–39)

**38 — References:** title + a single large text block (11 pt regular DSL Gray)
for the full source list. No other structural elements.

**39 — Stay Connected** (replaces the old "Follow Us" slide entirely — do not
use the previous version's Follow Us layout):
- DSL Teal band/shape near vertical center
- "STAY CONNECTED" label, 11 pt bold — reproduced as `#F76900` in the source
  template (see Known Inconsistencies above; do not silently correct to the
  theme's `#E67A2F`)
- URL ("WWW.DYNAMICSLAB.ORG"), 40 pt bold, centered
- A short DSL Green accent rule
- Four social-platform icon images (0.6"×0.6" each, evenly spaced), each
  hyperlinked: LinkedIn, Instagram, Facebook, YouTube (confirmed from the
  template's relationship targets — note this is a different platform set than
  the previous skill version's Twitter/X + LinkedIn + Instagram guess; use
  this confirmed set, and flag to the researcher if the actual current DSL
  social presence differs)
- Small standard logo bottom-left

---

## Stage 1 — Intake

Ask all questions at once before doing anything else.

> "Before I build the scaffold, I need a few things:
> 1. What is this bulletin about, and what is its primary purpose?
> 2. Who is the audience?
> 3. What sections or themes do you know this bulletin needs to cover?
>    (rough is fine — a topic list works)
> 4. Is there a partner organization or funder to acknowledge?
> 5. Are there named authors or contributors to include? (If yes, I can use the
>    Contributors layout — up to four people per slide.)
> 6. Do you want any of these optional slides: an Executive Summary, a Five Key
>    Findings summary, a Table of Contents, a Glossary of Terms, or an About the
>    Client slide? These are all standard layouts in the current template but
>    aren't required.
> 7. Which cover style do you want — full-bleed photo, photo on the right half,
>    or photo on the left half? (I can also just pick one and you can swap it
>    later — the three are interchangeable.)"

Wait for all responses before proceeding. If any answer is vague but workable,
proceed and note the assumption in the scaffold footer. If a blocking answer is
missing (topic, audience, or section list), ask for it before continuing.

---

## Stage 2 — Structure Derivation

Using the intake responses, determine:

- **Section list:** derived directly from the researcher's topic list. Do not
  impose a predetermined section sequence.
- **Slide count per section:** estimate based on content complexity. Label all
  estimates as adjustable.
- **Content-slide variant per slide:** for each slide, pick the best-fit variant
  from the Content & Stats catalog (26–34) or the Large Image/Data-Viz catalog
  (35–36) based on what that slide's content logically needs — text+visual,
  comparison, three-card, numbered rows, 2×2 grid, stat callouts, native chart,
  or full-width visual. Do not invent a new layout; pick the closest fit from
  the nine + two variants above.
- **Divider variant:** pick one of the three divider styles (23/24/25) to use
  consistently across all section breaks in the bulletin — mixing divider styles
  within one deck is allowed by the template but should be a deliberate choice,
  not an accident. Confirm with the researcher if unstated.
- **Special slides:** Contributors (if named contributors given), About the
  Client (if partner/funder given and the researcher wants a dedicated slide,
  distinct from a brief acknowledgment), Executive Summary / Five Key Findings /
  Table of Contents / Glossary (per Stage 1 Q6).

### Confirmation step

Before producing the scaffold, present the proposed section sequence, chosen
cover variant, and chosen divider variant to the researcher:

> "Here's the section structure I'm proposing based on what you described:
> [list sections with estimated slide counts and chosen content-slide variants]
> Cover style: [variant]. Divider style: [variant].
> Does this look right, or would you like to adjust before I build the scaffold?"

Do not produce the scaffold or the .pptx until the researcher confirms or
provides corrections. If they adjust, revise the structure and confirm once more
before proceeding.

---

## Stage 3 — Scaffold Output

Once structure is confirmed, produce a markdown file saved to:
`/mnt/user-data/outputs/tb-scaffold-<short-topic-name>-<YYYYMMDD>.md`
Use today's date in YYYYMMDD format for the filename.

### Scaffold structure

Fixed opening → researcher-derived body → fixed closing.

#### Fixed opening slides (always present)

**Slide — Cover** (variant confirmed in Stage 2)
```
Title: [RESEARCHER TO SUPPLY — short title/subtitle pairing]
Subtitle: [one-sentence topic summary, DSL Tan text]
Presenters: [name(s), credentials]
Titles, org, date: [role(s)], The Dynamic Sustainability Lab · [Month Day, Year]
Photo credit: [RESEARCHER TO SUPPLY, or delete the line if not needed]
[COVER PHOTO PLACEHOLDER — replace with photo from SharePoint Templates/Logo_&_Pictures/SU Photos]
[VERIFY BEFORE PUBLISHING: confirm presenter names, titles, and date]
```

**Optional — Table of Contents** (only if requested in Stage 1 Q6)
```
[Numbered list of confirmed section titles, each with a one-line explanation]
[PRESENTATION TIME PANEL — insert expected duration, or delete this panel]
```

**Optional — Executive Summary** (only if requested)
```
[EXECUTIVE SUMMARY PLACEHOLDER — one paragraph]
[PHOTO OR DATA VISUALIZATION PLACEHOLDER]
Source: [RESEARCHER TO SUPPLY]
```

**Optional — Five Key Findings** (only if requested)
```
[Five numbered cards, each: heading + one-sentence support]
```

**Optional — Glossary of Terms** (only if requested)
```
[Numbered term/definition rows]
```

#### Body slides (researcher-derived)

For each section, output a divider slide followed by content slides, using the
divider variant and content-slide variants chosen in Stage 2:

```
**Divider Slide — [Section Title]** (variant: [A/B/C])
Subtitle: [short explanation line]
[SECTION PHOTO PLACEHOLDER — if the chosen variant uses one; replace with photo
 from SharePoint Templates/Logo_&_Pictures/SU Photos]

**Slide N — [Slide Title]** (content variant: [26–36])
Kicker: [section name, all-caps — matches the divider it falls under]
Purpose: [one sentence describing what this slide should accomplish]
Placeholders:
  - [PLACEHOLDER: description of what goes here]
  - [FIGURE NEEDED: description — remember figures/tables get a centered
    "Figure N: description" caption if the deliverable requires one]
  - [DATA NEEDED: description of stat, finding, or data point]
```

Only include placeholder types relevant to that slide and its chosen variant.

#### Fixed closing slides

**Optional — Contributors** (only if named contributors were given)
```
[Up to four cards: headshot placeholder, title, name+credentials, role, email]
[VERIFY BEFORE PUBLISHING: confirm all contributor details]
```

**Optional — About the Dynamic Sustainability Lab**
```
[DSL mission boilerplate — see Brand Reference]
[ABOUT PHOTO PLACEHOLDER]
[VERIFY BEFORE PUBLISHING: confirm About DSL text is current at www.dynamicslab.org]
```

**Optional — About the Client** (only if partner/funder given and a dedicated
slide was requested, not just an acknowledgment line elsewhere)
```
[CLIENT STATEMENT PLACEHOLDER]
```

**References** (always present if any sources are cited)
```
[List of sources]
```

**Stay Connected** (always present)
```
WWW.DYNAMICSLAB.ORG
[Social icons: LinkedIn, Instagram, Facebook, YouTube — confirm handles are current]
[VERIFY BEFORE PUBLISHING: confirm URL and social handles are current]
```

#### Scaffold footer

```
---
## Scaffold Notes
- Generated: [YYYYMMDD]
- Source template (design spec origin): 202608_DSLPresentationTemplate_v02.potx
- Confirmed section structure: [list sections]
- Cover variant: [A/B/C]. Divider variant: [A/B/C].
- Estimated slide count: [total] (adjustable)
- Assumptions: [list any assumptions made where intake was vague]
- Logo source used: DSL_Memo_Logo.pdf (standard) / DSL_White_Logo_OnNavy.pdf
  (white knockout, chroma-keyed) — both extracted at build time via unzip
  (pdftoppm/pdfimages fallback), no .potx dependency

Known template inconsistencies carried forward as-built (not corrected):
- Logo asset orange (#F76900-ish) does not match the new palette's approved
  oranges (#E67A2F / #EE7623).
- "Stay Connected" label color is hardcoded to the old orange in the source
  template.
- The white-knockout logo's source raster carries a spurious white band, trimmed
  during extraction (see Logo Handling) rather than left in the delivered asset.

This scaffold is a starting point. All [VERIFY BEFORE PUBLISHING] flags must be
resolved before the bulletin is finalized.
```

After presenting the scaffold, ask:
> "Does the scaffold look right? I'll generate the .pptx from this structure.
> Let me know if you want to adjust anything first."

Proceed to Stage 4 once confirmed. If the researcher wants changes, update the
scaffold file first, then proceed.

---

## Stage 4 — PPTX Generation

Read `/mnt/skills/public/pptx/SKILL.md` and `/mnt/skills/public/pptx/pptxgenjs.md`
before writing any code.

### Setup

```bash
npm list -g pptxgenjs 2>/dev/null | grep pptxgenjs || npm install -g pptxgenjs
```

### Logo preparation (run before building slides)

Follow the Logo Handling steps in the Brand Reference above in full: extract the
standard logo from `$DSL_LOGO_PDF` (unzip first, `pdftoppm`
fallback), extract the white knockout logo from
`$DSL_WHITE_LOGO_PDF` (unzip first, `pdfimages`/`pdftoppm`
fallback — it is not a genuine PDF either), trim the spurious white band from
the white-knockout raster, then chroma-key and crop it. None of these steps
touch the `.potx`.

**Always visually inspect both logos before embedding into slides** — confirm
the standard logo reads clearly on white and the white knockout logo reads
clearly on navy/dark/photo backgrounds, specifically check the knockout logo's
edges for chroma-key fringing at normal slide-viewing scale, and confirm no
stray white band survived into the cropped asset. For the cover/divider corner
placement, also confirm the (width-capped) logo does not overlap the photo
panel or the title/subtitle text block before finalizing positions.

### File output

Save to: `/mnt/user-data/outputs/tb-<short-topic-name>-<YYYYMMDD>.pptx`

### Implementation rules

- Use `pres.layout = "LAYOUT_WIDE"` — 13.333" × 7.5".
- Font: `"Verdana"` on every text element. No exceptions, no Calibri.
- Use only the nine approved hex colors from the Brand Reference. Apply the
  60/30/10 balance and the orange-on-white contrast rule.
- Build each slide from the exact positions, sizes, and font sizes given in the
  Slide Catalog above — these came directly from the source template's XML, not
  estimates — **except where Sizing and placement above explicitly documents a
  corrected figure** (the corner-logo width cap and position), which takes
  precedence over the raw XML-derived number.
- Use `logoStandard` on white/light slide backgrounds; `logoWhite` on dark,
  navy, teal, or photo backgrounds. Never use a raw, unprocessed, or recolored
  logo file. Size by target height (not a flat width) per the Logo Handling
  specs for the footer logo; for the cover/divider corner logo, size by the
  capped width instead and derive height from it (see Sizing and placement).
  Always derive the non-capped dimension from each extracted asset's actual
  aspect ratio rather than assuming a fixed ratio between the two variants.
- Every boilerplate/placeholder slide that needs one carries a visible
  `[VERIFY BEFORE PUBLISHING]` flag, matching the researcher-facing scaffold.
- Placeholder boxes (structural placeholders Claude inserts, not part of DSL's
  own palette): light grey `#F2F2F2` fill (this is Claude's own scaffold
  convention — not one of the nine approved hex values, and should look
  visually distinct from real DSL content for that reason), `#DDDDDD` border,
  muted italic `#999999` text. **This applies to content-card placeholders on
  white backgrounds only.** Photo placeholders standing in for a cover or
  divider photo need a darker fill (mid-gray, e.g. `#707070`) so their white
  caption text stays legible — light grey with white text fails contrast and is
  effectively invisible (confirmed defect, 2026-08-27 test build).
- Figure/table placeholders get a centered "Figure N: [description]" caption
  beneath them, matching the DSL figures-skill caption convention.
- Photo placeholders (cover, divider slides): darker fill (see above) with
  italic white text directing the researcher to
  `SharePoint Templates/Logo_&_Pictures/SU Photos`.
- Apply the image border/shadow convention (Outer, Offset: Bottom) to any
  floating photograph — never to the logo, never to full-bleed cover/divider
  photos.
- For slide 34 (three stat callouts), confirm the intended background with the
  researcher before building — the source slide's title/text colors imply a
  dark background, but no explicit slide background fill was found in the
  extracted XML. Don't guess a background color; ask.
- Size cover and divider title text boxes for their worst-case wrap (up to
  three lines at cover scale, per the Typography note above), and position
  every element beneath the title relative to that reserved height rather than
  a fixed offset copied from a short example title.

### QA

After generating, run visual QA per the pptx SKILL.md verification loop:
1. Convert to PDF using LibreOffice headless
2. Rasterize to JPEG at 120dpi
3. Inspect the cover, first content slide, one divider slide, the Stay
   Connected slide, and any optional slides included (Contributors, Executive
   Summary, etc.)
4. Check for: overflow, overlap, misalignment, placeholder text in boilerplate
   slides, white boxes or navy fringing around logos on dark backgrounds,
   incorrect hex values (only the nine approved values should appear), that
   Verdana is specified throughout (LibreOffice will substitute a generic
   sans-serif for rendering — expected and fine, since the file itself carries
   the correct font spec), **title text colliding with the subtitle or
   presenter block beneath it on cover/divider slides**, and **low-contrast
   placeholder captions** (white text on a fill lighter than roughly `#808080`
   will read as invisible or near-invisible — confirmed defects from the
   2026-08-27 test build, both worth checking on every run, not just once).

### Present

Use `present_files` to deliver both the scaffold markdown and the .pptx together.

---

## Output rules

- Every content slot in the .pptx is a placeholder. Claude does not write
  bulletin prose, findings, or data interpretations.
- Every boilerplate slide carries a `[VERIFY BEFORE PUBLISHING]` flag — visible
  on the slide itself, not just in the scaffold.
- Do not add sections, themes, or content beyond what the researcher provided in
  intake.
- The bulletin structure reflects the researcher's topics, not Claude's
  interpretation of what the bulletin should cover.
- Scaffold and .pptx are always produced together.
- Never invent a layout variant not present in the Slide Catalog above. If
  nothing in the catalog fits a slide's content well, say so and ask the
  researcher how they'd like to handle it rather than improvising a new layout.

---

## Reference files

- `$DSL_LOGO_PDF`, `$DSL_WHITE_LOGO_PDF` —
  the sole logo sources this skill now uses (see Logo Handling above). Both are
  confirmed present in `/mnt/project/`, and both are confirmed disguised-ZIP
  archives despite the `.pdf` extension (not just the memo logo, as previously
  documented).
- `/mnt/project/index.md` — project index
- `202608_DSLBrandGuide_v01.pdf` — the DSL Brand Guide (Aug 2026), the
  authoritative source this template was built to match
- `/mnt/skills/public/pptx/SKILL.md` — pptx skill (read before Stage 4)
- `/mnt/skills/public/pptx/pptxgenjs.md` — pptxgenjs implementation guide

**Historical only — not a runtime dependency:** `202608_DSLPresentationTemplate_v02.potx`
was the original source of every color, font size, position, and slide-catalog
entry in this skill (see Provenance note at top). Those values are now fully
encoded above and do not require the file to be present in future sessions. A
same-named file exists on the DSLAI SharePoint at
`Shared Documents/DSL_Templates/DSL_Templates/202608_DSLPresentationTemplate_v02.potx`
but could not be read via the Microsoft 365 connector (Graph refused text
conversion) — this is noted for completeness only; it does not block this skill.
If the template is ever revised, a future session will need the updated file
supplied directly in chat to re-extract the changed values.
