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

Last modified: 2026-10-16

# Excel Skill

Produces professional Excel (.xlsx) spreadsheets following DSL formatting standards.
Output is always a downloadable file built with openpyxl. Gantt-style trackers follow
the DSL Gantt template (`202608_DSLGanttChartTemplate_v01.xltx`) as encoded in the
Gantt section below. All other sheet types follow the rules in Stage 2 and
`Templates/Excel/Excel spreadsheets rules.docx` on the DSLAI SharePoint. Where any
other source conflicts with the Gantt template, the template governs.

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
- **Page orientation** — portrait or landscape? (Not asked for Gantt: always landscape.)
- **Sheet count** — single sheet or multiple tabs?
- **Charts?** — does the researcher want any charts or visualizations embedded?
- **Financial?** — does this involve currency, budgets, or financial modeling?

If the sheet type is a Gantt or project tracker, also ask:

- **Phase structure** — what phases and tasks should be included? Use as many phases as
  the project needs; do not assume a standard set.
- **Leads** — who leads or coordinates each task?
- **Start week and length** — the first week-starting Monday and how many weeks the
  timeline spans.
- **Work and deliverable weeks** — for each task, which weeks are work periods and which
  week a deliverable is due.
- **Project name** — leave the write-in field blank if not provided.
- **Column granularity** — weekly (standard) or monthly?

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

If Gantt/tracker, replace question 3 and add:

> "3. (Skip — Gantt sheets are always landscape.)
> 7. What phases and tasks should the tracker include, and who leads each task?
> 8. What is the first week-starting Monday, and how many weeks should the timeline cover?
> 9. For each task, which weeks are work periods and which week is the deliverable due?
> 10. What is the project name?
> 11. Weekly or monthly timeline columns?"

---

## Stage 2 — DSL Formatting Standard

Apply the following rules to every spreadsheet produced by this skill. Gantt trackers
follow the Gantt section under Common Sheet Types instead of the Logo, Title, Font,
Borders, Shading, and Page layout rules below wherever the two differ; all other rules
(wrap text, number formatting, formulas, grid lines) still apply.

### Logo
Place the DSL logo in the top-left of the sheet as an embedded image. Always build it
at runtime from `/mnt/project/DSL_Memo_Logo.pdf` — never use an uploaded PNG. The file
arrives in one of two forms: a real PDF, or a zip archive holding the page image as
`1.jpeg`. Check which with `zipfile.is_zipfile` and handle both:

```python
import zipfile, subprocess
from PIL import Image
import numpy as np

SRC = '/mnt/project/DSL_Memo_Logo.pdf'
if zipfile.is_zipfile(SRC):
    with zipfile.ZipFile(SRC) as z:
        z.extract('1.jpeg', '/tmp/dsl_logo_src')
    img = Image.open('/tmp/dsl_logo_src/1.jpeg').convert('RGB')
else:
    subprocess.run(['pdftoppm', '-r', '150', '-png', SRC, '/tmp/dsl_logo'], check=True)
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
the title block below it stays clear. (Gantt sheets: rows 1–4, see Gantt section.)

### Title
Applies to every sheet type except Gantt trackers, which have no title.

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
- Arial throughout (Gantt trackers: Calibri, see Gantt section)
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
- Exception: Gantt timeline cells hold no text, so their gray and red fills are allowed

### Bold usage
- Totals and subtotals: always bold
- Column headers: always bold
- Phase and section labels: bold
- Standard data cells: not bold

### Merged cells
- Merge and center cells for phase or section labels that span multiple rows or columns
- Use only where it genuinely aids readability — do not merge arbitrarily
- The A4:A5 title block is a required merge on every non-Gantt sheet (see Title above)

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

### Project / task tracker (Gantt)

Encodes the DSL Gantt template. Build the sheet from scratch with openpyxl using the
rules below; do not load or copy the template file. Use the phases and tasks the
researcher provides, with as many phases as the project needs.

**Sheet setup**
- No title block: no A4:A5 merge and no title text. Rows 4 and 5 stay blank.
- Grid lines off on screen and in print.
- Calibri 10pt in every cell. Bold only where stated below.
- Landscape, fit to one page wide by one page tall, 11×17 paper.
- Margins: left and right 0.25", top and bottom 0.75", header and footer 0.3".
- Print area runs from A1 to the last timeline column and last task row.
- No frozen panes, no legend, no instruction note.

**Columns**
- A: Task # — width 4.375 (widen only if a task number is too long to fit)
- B: Description — width 40.75
- C: Lead / Coordinator — width 25.25
- D onward: one timeline column per week, width 5.875 each

**Header block**
- Logo anchored at A1, about 254pt (339px) wide, height proportional, spanning rows 1–4.
- Row 3: C3 reads "Project Name:", bold, right-aligned. D3:O3 is merged (or D3 through
  the last timeline column if the timeline is shorter than 12 columns), left blank
  unless the researcher provided a name, with a thin bottom border across the whole
  merged range.
- Row 6 is the column header row, height 13.5:
  - A6 is blank with no border.
  - B6 "Description", bold, left-aligned.
  - C6 "Lead / Coordinator", bold, centered.
  - D6 merged across every timeline column, reading "Week Starting " (bold, centered).
  - Medium border box around B6, around C6, and around the merged D6 range.

**Phase rows**
- Each phase gets one row above its tasks. Label in column B, bold, formatted
  "Phase N. Name" (e.g., "Phase 2. Research").
- Fill `D7C8B5` across the full row from column A to the last timeline column. No borders.
- Phase 1 sits in row 7 and carries the week-starting dates in its timeline cells; later
  phase rows carry no dates.
- Dates: D7 is the only hardcoded date and must be a Monday. E7 onward are formulas,
  `=D7+7`, `=E7+7`, and so on. Format `d-mmm`, centered, not bold.
- No blank spacer rows between phases.

**Task rows**
- A: task number as text in "phase.task" form (`"1.1"`, `"1.10"`), centered. Store as text
  so 1.10 does not collapse to 1.1.
- B: description, wrap text. C: lead. Timeline cells hold no text.
- Thin borders on every cell from A to the last timeline column.
- No alternating row shading.
- Row height: 12.75 by default. For descriptions that wrap, set the height to
  12.75 × line count, estimating about 45 characters per line in column B.
- Create rows only for the tasks provided — no blank numbered filler rows.

**Timeline fills**
- Work period: solid gray `808080`.
- Deliverable: solid red `FF0000`, placed in the week the deliverable is due. Weeks of
  work leading up to it are gray. A task that is only a deliverable has a single red cell.
- All other timeline cells stay unfilled.

**Monthly variant (only if the researcher asks for it)**
Same layout. Row 6 banner reads "Month Starting ". D7 is the first day of the starting
month; E7 onward are `=EDATE(D7,1)`, format `mmm-yy`.

```python
tan = PatternFill('solid', fgColor='D7C8B5')
gray = PatternFill('solid', fgColor='808080')
red = PatternFill('solid', fgColor='FF0000')
thin, med = Side(style='thin'), Side(style='medium')

ws.merge_cells(start_row=6, start_column=4, end_row=6, end_column=last_col)
ws['D6'] = 'Week Starting '
ws['D7'] = first_monday                      # only hardcoded date
for c in range(5, last_col + 1):
    ws.cell(7, c).value = f'={get_column_letter(c-1)}7+7'
ws.page_setup.orientation = 'landscape'
ws.page_setup.paperSize = ws.PAPERSIZE_TABLOID
ws.sheet_properties.pageSetUpPr.fitToPage = True
ws.page_setup.fitToWidth = ws.page_setup.fitToHeight = 1
```

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
   (Gantt: phases, tasks, leads, start Monday, week count, work and deliverable weeks)
3. **Build logo** — extract or rasterize `/mnt/project/DSL_Memo_Logo.pdf` (PDF or zip
   form), crop white margins (see Logo section above)
4. **Build** — use openpyxl; apply the DSL formatting rules from Stage 2 (non-Gantt) or
   the Gantt section (Gantt), with `showGridLines = False` on every sheet
5. **Recalculate** — run `python scripts/recalc.py output.xlsx`; resolve any errors
   before proceeding
6. **Verify** — on every sheet confirm grid lines off, no borders on empty cells, no
   formula errors. Non-Gantt: title merged in A4:A5 at 14pt bold Arial. Gantt: no title,
   Calibri 10, D7 is a Monday with `+7` formulas across, phase rows unbordered with
   `D7C8B5` fill, timeline fills only `808080` or `FF0000`, print area set
7. **Save** to `/mnt/user-data/outputs/<descriptive-name>.xlsx`
8. **Present** using `present_files`; note any assumptions made during intake

---

## Reference Files

- `/mnt/skills/public/xlsx/SKILL.md` — technical implementation (openpyxl, pandas,
  recalc workflow, formula best practices)
- `/mnt/project/DSL_Memo_Logo.pdf` — logo source (real PDF or zip holding `1.jpeg`); always build at runtime
- `202608_DSLGanttChartTemplate_v01.xltx` — source of the Gantt rules above; the rules
  are self-contained, so the file is not needed at runtime
- `figures-skill.md` (SharePoint: `Claude Skill Files/figures-skill.md`) — chart type
  selection and best practices; apply when charts are requested
- `Templates/Excel/Excel spreadsheets rules.docx` (DSLAI SharePoint) — source for DSL
  Excel formatting rules on non-Gantt sheets
- `Templates/Excel/Example Excel Sheet.xlsx` (DSLAI SharePoint) — reference example of a
  DSL Gantt-style tracker; the Gantt template governs where they differ
