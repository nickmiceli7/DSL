---
name: excel
description: >
  Use this skill whenever a researcher needs to create a new Excel spreadsheet (.xlsx)
  following DSL formatting standards. Triggers include: "create a spreadsheet", "make an
  Excel file", "build a tracker", "I need a table in Excel", "Gantt chart", "project
  timeline", "phase tracker", "task schedule", "data table", "project tracker", "budget
  spreadsheet", or any request to produce a structured spreadsheet from scratch. Also
  trigger when a researcher provides raw data (CSV, bullet points, notes) and wants it
  formatted as a proper DSL Excel file. This skill handles all Excel output types
  including Gantt-style project trackers — do not use a separate Gantt skill. Do NOT
  trigger for editing or analyzing existing spreadsheets — this skill produces new .xlsx
  files only.
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



Last modified: 2026-08-13

# Excel Skill

Produces professional Excel (.xlsx) spreadsheets following DSL formatting standards.
Output is always a downloadable file built with openpyxl, formatted consistently with
DSL's visual conventions and the rules defined in `Templates/Excel/Excel spreadsheets
rules.docx` on the DSLAI SharePoint.

---

## Overview

Three stages:

1. **Intake** — gather what's needed before building anything
2. **Build** — produce the formatted .xlsx applying all DSL standards
3. **Deliver** — present the file and note any assumptions

---

## Stage 1 — Intake

Check the current conversation for already-provided context before asking. Extract:

- **Content** — what data or structure goes in the sheet?
- **Sheet type** — tracker, data table, Gantt chart, budget, other?
- **Page orientation** — portrait or landscape?
- **Sheet count** — single sheet or multiple tabs?
- **Charts?** — does the researcher want any charts or visualizations embedded?
- **Financial?** — does this involve currency, budgets, or financial modeling?

If the sheet type is a Gantt or project tracker, also ask:

- **Phase structure** — what phases and tasks should be included? (do not assume a
  standard set — ask the researcher to describe their project structure)
- **Column granularity** — should timeline columns represent weeks (with date-stamped
  headers) or months?

Ask all missing items at once. If enough context exists to proceed, note any assumptions
and begin.

Example prompt when context is missing:

> "Before I build, a few quick questions:
> 1. What data or structure should the sheet contain?
> 2. What type of spreadsheet is this — tracker, data table, budget, Gantt, other?
> 3. Portrait or landscape orientation?
> 4. Single sheet or multiple tabs?
> 5. Do you want any charts or visualizations in the sheet?
> 6. Does this involve currency or financial data?"

If Gantt/tracker, add:

> "7. What phases and tasks should the tracker include?
> 8. Should timeline columns be weekly (e.g., '19-Jan', '26-Jan') or monthly (e.g., 'Jan', 'Feb')?"

---

## Stage 2 — DSL Formatting Standard

Apply ALL of the following rules to every spreadsheet produced by this skill.

### Logo
Place the DSL logo in the top-left of the sheet as an embedded image. Always rasterize
from `$DSL_LOGO_PDF` at runtime — never use an uploaded PNG.

```bash
pdftoppm -r 150 -png $DSL_LOGO_PDF /tmp/dsl_logo
```

```python
from PIL import Image
import numpy as np
img = Image.open('/tmp/dsl_logo-1.png').convert('RGB')
arr = np.array(img)
brightness = arr[:,:,0].astype(int)+arr[:,:,1].astype(int)+arr[:,:,2].astype(int)
nonwhite = brightness < 750
rows = np.any(nonwhite, axis=1)
cols = np.any(nonwhite, axis=0)
rmin,rmax = np.where(rows)[0][[0,-1]]
cmin,cmax = np.where(cols)[0][[0,-1]]
cropped = img.crop((max(0,cmin-20), max(0,rmin-20), cmax+20, rmax+20))
cropped.save('/tmp/dsl_logo_clean.png')
```

Embed `/tmp/dsl_logo_clean.png` using openpyxl's `Image` class. Scale to approximately
250pt wide (height proportional). Position anchored to cell A1, occupying rows 1–3 so
the title block below it stays clear.

### Title
- Place the title in **column A, rows 4–5**, merged as a single block (`A4:A5`)
- Bold, font size 14pt
- Left-aligned horizontally, vertically centered within the merged block
- Enable wrap text so long titles stay inside the block rather than bleeding right
- Describes the sheet contents clearly (e.g., "Off-Shore Wind Policies Review")
- If multiple sheets exist, each sheet must have its own title in the same A4:A5 position
- Leave row 6 blank as a spacer; data, headers, and any "Project Name:" field begin at
  row 7 or below

```python
ws.merge_cells('A4:A5')
c = ws['A4']
c.value = 'Off-Shore Wind Policies Review'
c.font = Font(name='Arial', size=14, bold=True)
c.alignment = Alignment(horizontal='left', vertical='center', wrap_text=True)
```

### Font
- Arial throughout
- 10pt for all cell content
- 14pt for title only

### Grid lines
- Turn off the worksheet background grid lines on **every** sheet:
  `ws.sheet_view.showGridLines = False`
- Structure comes from cell borders and shading only — never from the default grid
- Also suppress printed grid lines: `ws.print_options.gridLines = False`

### Borders
- Apply borders to ALL cells containing data (these are the only visible lines on the
  sheet, since background grid lines are off)
- Use heavier/thicker borders to visually group logical sections (e.g., phase groupings,
  header rows, totals rows)
- Do not apply borders to empty cells — including the logo and title block area, which
  should remain unbordered

### Column widths and cell content
- No cell content may bleed into adjacent cells
- Enable wrap text on all data cells
- Adjust column widths to fit content — optimize for the sheet's orientation and
  target page size
- Column A carries the title block, so verify its width still suits the data below it;
  widen the merged title block across additional columns only if the title genuinely
  cannot fit

### Page layout
- Set orientation per intake (portrait or landscape)
- Target: fit each sheet on one page — 8.5×11 for portrait, 11×17 for landscape
- Adjust margins as needed to achieve fit; do not sacrifice readability

### Shading
- Use light-colored fills to differentiate rows, columns, or logical groupings
- Never use dark fills — all shading must remain readable with black text
- Phase or section header rows may use a slightly darker (but still light) fill than
  standard data rows

### Bold usage
- Totals and subtotals: always bold
- Column headers: always bold
- Phase and section labels: bold
- Standard data cells: not bold

### Merged cells
- Merge and center cells for phase or section labels that span multiple rows or columns
- Use only where it genuinely aids readability — do not merge arbitrarily
- The A4:A5 title block is a required merge (see Title above)

### Number formatting
- Integers: comma separator for thousands (e.g., 2,567 not 2567)
- Currency: $ symbol and comma separator (e.g., $2,567); omit decimals unless required;
  if decimals are needed, use exactly two (e.g., $2,567.12)
- Percentages: maximum two decimal places; round where practical (e.g., 87% not 86.723%)
- Numbers: always right-aligned
- Text: left-aligned

### Financial color coding (financial sheets only)
Apply only when the sheet contains financial modeling, projections, or budget formulas:

- **Blue text** (RGB 0, 0, 255): hardcoded inputs the researcher will adjust for scenarios
- **Black text** (RGB 0, 0, 0): all formulas and calculated values
- **Green text** (RGB 0, 128, 0): values linked from other sheets within the same workbook
- **Red text** (RGB 255, 0, 0): links to external files

### Formulas
- Always use Excel formulas for calculated values — never compute in Python and hardcode
  the result
- Place all assumptions (rates, multipliers, targets) in dedicated cells; reference them
  in formulas rather than embedding literal values
- After building, run `python scripts/recalc.py output.xlsx` to recalculate and verify
- Deliver with zero formula errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)

---

## Charts and Visualizations

If the researcher requests a chart embedded in the spreadsheet, apply the following
before building:

### 1. Read the figures skill
Read `figures-skill.md` (SharePoint: `Claude Skill Files/figures-skill.md`) and apply
its **type selection logic** and **best practices checklist** to determine the right
chart type and verify the output meets DSL visualization standards.

### 2. Scoped application
Apply from the figures skill:
- ✅ Type selection catalog (bar, line, sunburst, etc.)
- ✅ Figure best practices checklist (proportions, labels, title, color, simplicity)

Do NOT apply from the figures skill:
- ❌ Export/Inkscape refinement workflow — charts stay embedded in the workbook
- ❌ Import-as-image rule — that rule applies to standalone figures in reports and
  slide decks, not to charts living natively inside a spreadsheet

### 3. Chart formatting rules
- Use Arial font, consistent with the rest of the sheet
- Keep color intentional — use shading to highlight key patterns, not for decoration
- Include a descriptive chart title (not just the topic — say what the chart shows)
- Include axis labels and units
- Keep the chart as simple as possible; remove chartjunk

---

## Common Sheet Types

### Project / task tracker (Gantt-style)

Use this structure for any project timeline, phase tracker, task schedule, or Gantt chart request.
**Do not prescribe a fixed phase structure** — always use the phases and tasks provided by the
researcher during intake.

**Column structure (left to right):**
1. Task # — decimal notation combining phase and task (e.g., 1.1, 1.2, 2.1)
2. Description — task name or description
3. Lead / Coordinator — responsible person or team
4. Timeline columns — one column per time period, date-stamped if weekly

**Timeline column format:**
- **Weekly (default if researcher prefers granularity):** date-stamped headers showing
  the week-starting date (e.g., "19-Jan", "26-Jan", "2-Feb"). Add a "Week Starting"
  label in the bottom-left or as a row label to clarify the convention.
- **Monthly:** abbreviated month names (e.g., "Jan", "Feb") under a merged year header

**Phase rows:**
- Insert a phase label row above each phase's tasks (e.g., "Phase 1. Project Management")
- Bold, light fill, spanning the full row width
- Do not use a separate Phase column — phase identity comes from the decimal task number
  and the phase label row above the group

**Task rows:**
- Use filled/shaded cells (or "X" markers) in timeline columns to indicate active periods
- Filled cells are preferred for weekly Gantt charts — use a solid DSL orange or dark fill
  to indicate active weeks, leaving inactive weeks blank/white
- Alternate light shading on task rows within each phase for readability

**Borders:**
- Standard thin borders on all data cells
- Medium border on the bottom of the column header row
- Medium border between phase groups
- Thick outer border around the entire data range

**Project Name field:**
- Include a "Project Name:" label and value cell below the A4:A5 title block and above
  the column headers (row 7 or below)
- Allows the template to be reused across projects

**Logo sizing:**
- Use 250pt wide for Gantt sheets — the logo should be prominent in the header area

### Data table
- Bold column headers in the first row below the title block and spacer (row 7 or below)
- Alternating row shading for readability
- Totals row at bottom, bold, with heavier top border

### Budget sheet
- Apply financial color coding (see above)
- Totals and subtotals bold, with heavier border above the totals row
- Currency formatting on all dollar values
- Assumption cells clearly labeled; referenced by formula cells

---

## Stage 3 — Workflow

Follow this sequence for every build:

1. **Read skills** — read this skill and `/mnt/skills/public/xlsx/SKILL.md` before
   writing any code; if charts are requested, also read `figures-skill.md`
2. **Run intake** — confirm content, type, orientation, tabs, charts, financial flag
3. **Rasterize logo** — run `pdftoppm` on `$DSL_LOGO_PDF`, crop
   white margins (see Logo section above)
4. **Build** — use openpyxl; apply all DSL formatting rules from Stage 2, including the
   merged A4:A5 title block and `showGridLines = False` on every sheet
5. **Recalculate** — run `python scripts/recalc.py output.xlsx`; resolve any errors
   before proceeding
6. **Verify** — confirm on each sheet: grid lines off, title merged in A4:A5 at 14pt
   bold Arial, no borders on empty cells, no formula errors
7. **Save** to `/mnt/user-data/outputs/<descriptive-name>.xlsx`
8. **Present** using `present_files`; note any assumptions made during intake

---

## Reference Files

- `/mnt/skills/public/xlsx/SKILL.md` — technical implementation (openpyxl, pandas,
  recalc workflow, formula best practices)
- `$DSL_LOGO_PDF` — logo source; always rasterize at runtime
- `figures-skill.md` (SharePoint: `Claude Skill Files/figures-skill.md`) — chart type
  selection and best practices; apply when charts are requested
- `Templates/Excel/Excel spreadsheets rules.docx` (DSLAI SharePoint) — primary source
  for DSL Excel formatting rules
- `Templates/Excel/Example Excel Sheet.xlsx` (DSLAI SharePoint) — reference example
  showing a properly formatted DSL Gantt-style tracker
