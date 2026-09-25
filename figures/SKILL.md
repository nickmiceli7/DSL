---
name: figures
description: >
  Use this skill whenever a researcher wants to create, design, or get guidance on any
  figure, chart, map, table, or data visualization for DSL work. Triggers include:
  "make a figure", "create a chart", "visualize this data", "help me make a map",
  "what graph should I use", "how do I make a Sankey", "create a table", "visualize
  my results", or any request involving turning data into a visual output. Also trigger
  when a researcher shares a dataset or describes their data and asks how to present it.
  Produces step-by-step instructions tailored to the researcher's data, goal, and
  intended deliverable.
---

# Figures & Visualizations

Guides DSL researchers through selecting the right visualization type and producing
step-by-step instructions to create it using DSL-standard tools. Output is always
a concrete, numbered walkthrough the researcher can follow independently.

---

## Overview

The skill has three stages:

1. **Intake** — gather the minimum context needed to give tailored instructions
2. **Type selection** — identify the right visualization type for the researcher's goal
3. **Step-by-step instructions** — produce a full, tool-specific walkthrough

---

## Stage 1 — Intake

Before generating any instructions, ask the researcher these three questions in a
single message. If any are already clear from context, skip them.

> "Before I walk you through this, I need a few quick things:
> 1. **What are you trying to show?** Describe the story or finding your visualization
>    should communicate (e.g., 'how feedstock supply has changed over time', 'where
>    our supplier facilities are located', 'how energy flows through the system').
> 2. **What does your data look like?** Briefly describe it — what are the rows and
>    columns, what format is it in (Excel, CSV, Power BI dataset, etc.), and roughly
>    how many data points.
> 3. **Where will this appear?** (e.g., slide deck, research report, policy brief,
>    conference poster, social media)"

Use the answers to select the visualization type (Stage 2) and calibrate the
instructions (Stage 3) to the researcher's specific data and deliverable context.

---

## Stage 2 — Visualization Type Selection

Match the researcher's goal to the correct type from the catalog below. If the match
is ambiguous, recommend the best fit and briefly explain why. If the researcher has
already named a type, confirm it's appropriate before proceeding.

### Visualization Type Catalog

#### Facility Map
- **What it is:** A geographic map showing the physical location of facilities as point markers.
- **Best used for:** Supply chain mapping, site selection analysis, regional coverage studies.
- **Tool chain:** Python (`oneworld` package) → Google Colab → export SVG → Inkscape (optional refinement) → embed in deliverable.
- **Claude's role:** Generate the Python code tailored to the researcher's data; guide them through running it in Google Colab.
- **Key API parameters:** `view` (bounding box as `[lon_min, lon_max, lat_min, lat_max]`), `projection` (use `'AlbersEqualArea'` for US maps), `inner_borders` (e.g. `['USA']` to show state lines), `add_markers()` accepts latitude/longitude as lists or as pandas DataFrame column names, with optional `color`, `size`, `marker`, `palette`, and `legend_pos` arguments. Save output with `savemap('filename.svg')`.

#### Flow Map
- **What it is:** A map showing directional movement between geographic origins and
  destinations using lines or arcs. Line thickness or color encodes flow magnitude.
- **Best used for:** Trade flow analysis, logistics and supply chain visualizations,
  migration patterns, commodity flows between states or countries.
- **Tool chain:** Power BI (FlowMap visualization) → export → Inkscape (refinement and labels) → embed.
- **Claude's role:** Guide Power BI setup and Inkscape refinement steps.

#### Infographic Map
- **What it is:** A stylized geographic map combining spatial information with graphic
  design elements — icons, callout boxes, data labels, color fills. Less technically
  precise than GIS maps; designed for visual storytelling.
- **Best used for:** Policy briefs, executive summaries, public-facing reports,
  conference posters/presentations, stakeholder presentations, social media content.
- **Tool chain:** Inkscape (import base map, add design elements) → export → embed.
- **Claude's role:** Advise on layout, suggest label placements, draft callout text.

#### Simple Charts
- **What it is:** Standard statistical charts — bar (horizontal comparisons), column
  (vertical comparisons or time series), line (trends over time), pie/donut
  (part-to-whole). Default choice for straightforward data communication.
- **Best used for:** Comparing values across categories, showing trends over time,
  illustrating proportions.
- **Tool chain:** Power BI or Excel → export image → Inkscape (refinement) → embed.
- **Claude's role:** Recommend the right chart subtype, guide Power BI/Excel setup,
  advise on labeling and formatting.

#### Sunburst
- **What it is:** A radial hierarchical chart showing part-to-whole relationships
  across multiple levels simultaneously. Innermost ring is the top-level category;
  each outer ring breaks it down further.
- **Best used for:** Feedstock category trees, organizational hierarchies,
  multi-level market share analysis.
- **Tool chain:** Power BI or Excel → export image → Inkscape (refinement) → embed.
- **Claude's role:** Guide data structuring for hierarchical input, walk through
  Power BI/Excel setup.

#### Sankey
- **What it is:** A flow diagram where band width is proportional to quantity.
  Shows how a resource moves through a system — including splits, merges, and losses.
- **Best used for:** Energy system flow analysis, supply chain material flows,
  biofuel feedstock-to-product pathways.
- **Tool chain:** Power BI → export image → Inkscape (refinement) → embed.
- **Claude's role:** Guide data formatting for Sankey input, walk through Power BI setup
  and Inkscape refinement.

#### Infographic
- **What it is:** A visual communication piece combining charts, icons, text, and
  graphic design to tell a data-driven story on a single page or slide. Designed
  for rapid comprehension by non-expert audiences.
- **Best used for:** Policy briefs, executive summaries, public-facing reports,
  conference posters/presentations, stakeholder presentations, social media content.
- **Tool chain:** Lab ChatGPT account (generate as .svg) → Inkscape (refinement) → export → embed.
- **Claude's role:** Draft the generation prompt (text, layout intent, data elements to
  highlight) for the researcher to run once in the lab ChatGPT account, producing a single
  .svg output — avoid multiple iterative regenerations in ChatGPT. Then guide Inkscape
  refinement of that .svg (labels, layout, color adjustments) rather than looping back to
  ChatGPT for changes.

#### Policy Line Chart
- **What it is:** A line chart annotated with policy events, legislative milestones,
  or program launch dates. Vertical reference lines mark policy dates with labeled
  callouts explaining the event.
- **Best used for:** Evaluating policy impact on production or prices, regulatory
  timeline communication.
- **Tool chain:** PowerPoint or Inkscape → export → embed.
- **Claude's role:** Help identify and draft policy annotations, guide layout in
  PowerPoint or Inkscape.

---

## Stage 3 — Step-by-Step Instructions

Once the visualization type is confirmed, produce a complete numbered walkthrough.
Instructions must be:

- **Tool-specific** — name the exact menu, button, field, or setting the researcher
  needs to interact with. Do not say "configure your data"; say "drag [field name]
  to the X-axis well."
- **Sequential** — every step builds on the last. The researcher should be able to
  follow without backtracking.
- **Data-aware** — reference the researcher's actual data structure from the intake.
  Name their columns, their categories, their units where known.
- **Deliverable-aware** — tailor export format and embed guidance to where the figure
  will appear (slide deck vs. report vs. poster).

### Instruction Template

Use this structure for every set of instructions:

```
## [Visualization Type] — Step-by-Step

### Step 1 — Prepare your data
[Specific data formatting or cleaning steps based on the researcher's data.]

### Step 2 — Build in [primary tool]
[Numbered sub-steps for the primary tool: Power BI / Excel / Python / PowerPoint / Inkscape.]

### Step 3 — Refine in Inkscape [omit if not in toolchain]
[Numbered sub-steps for Inkscape refinement: labels, color, layout adjustments.]

### Step 4 — Export
[Exact export settings: format (SVG preferred for embedding), resolution, file name convention.]

### Step 5 — Embed in your deliverable
[How to insert the exported image into the target deliverable — slide deck, Word doc, etc.]

### Step 6 — Apply figure best practices checklist
[Run through the checklist below before finalizing.]
```

Always end with the best practices checklist (see below) so the researcher can
self-verify before submitting.

---

## Figure Best Practices

Apply these to every figure. Include the checklist at the end of every instruction set.

**Content & accuracy**
- [ ] The most effective graph type was chosen for the data
- [ ] Proportions are accurate
- [ ] Axis intervals are equidistant
- [ ] Data are intentionally ordered (not arbitrary)

**Labels & titles**
- [ ] Title is descriptive (tells the reader what the figure shows, not just the topic)
- [ ] Data labels are present where needed
- [ ] Caption follows this format: `Figure #. Description of the figure. Source: Reference.`

**Design**
- [ ] Color scheme is intentional — used to highlight key patterns
- [ ] Color is consistent throughout the document
- [ ] Visualization is kept as simple as possible

**Production**
- [ ] Figure was created in Excel or Power BI and imported as an image (not built natively in PowerPoint or Word)

---

## Table Best Practices

When the researcher's output is a table rather than a chart, apply these rules and
include this checklist instead.

**Numbers**
- [ ] Numbers are right-aligned
- [ ] Comma separators used for thousands (e.g., 1,234)
- [ ] Decimals used only where needed (round where possible)
- [ ] Currency formatted correctly (e.g., $1,234)
- [ ] Percentages do not exceed two decimal places (e.g., 12.34%; prefer 12%)

**Formatting**
- [ ] Important values (totals, subtotals) are bold
- [ ] No text bleeding across cells (wrap text / adjust column widths)
- [ ] Light shading used to differentiate rows/columns where helpful
- [ ] Formatting is consistent throughout the document

**Production**
- [ ] Table was created in Excel and imported as an image (never built natively in Word or PowerPoint)
- [ ] Caption follows this format: `Table #. Description of the table. Source: Reference.`

---

## Toolchain Reference

| Tool | Role | Access |
|---|---|---|
| **Excel** | Simple charts, tables, sunburst | Standard Microsoft 365 |
| **Power BI** | Flow maps, Sankey, simple charts, sunburst | Standard Microsoft 365 (free desktop app) |
| **Python (oneworld)** | Facility maps | Google Colab (free, browser-only, no install) |
| **Lab ChatGPT account** | Infographic generation (.svg output) | Lab account — check with project lead for access |
| **Inkscape** | Refinement and labeling for most outputs | Free download: inkscape.org |
| **PowerPoint** | Policy line charts | Standard Microsoft 365 |

**Google Colab note:** Colab is the recommended Python environment for DSL because it
requires no admin rights or local installation. Navigate to colab.research.google.com,
create a new notebook, and paste the code Claude generates.

**Inkscape note:** Inkscape is used as the final refinement layer for most Power BI
and Excel outputs. It is free and open-source. Download at inkscape.org. Claude can
guide the researcher through specific Inkscape operations step by step.

---

## OneWorld (Facility Maps) Reference

Use this section when generating step-by-step instructions for facility maps in Google Colab.

### Installation (run once per Colab session)
```python
!pip install oneworld cartopy
```
> Cartopy is a required dependency. Colab does not have it pre-installed.

### Basic US facility map with markers from a DataFrame
```python
import oneworld as ow
import pandas as pd

# Load your Excel data
df = pd.read_excel('your_file.xlsx')  # must have latitude and longitude columns

# Create the base map (contiguous US)
mymap = ow.StaticMap(
    view=[-125, -66.5, 21, 50],       # [lon_min, lon_max, lat_min, lat_max]
    central_longitude=-98.6,
    projection='AlbersEqualArea',
    inner_borders=['USA']              # draws state lines
)

# Add facility markers — use your actual column names
mymap.add_markers(
    latitude='latitude',              # column name in your DataFrame
    longitude='longitude',            # column name in your DataFrame
    data=df,
    color='region',                   # optional: color by category column
    size=2.0,                         # marker size (or column name for variable size)
    marker='o',                       # shape: 'o' circle, '*' star, 's' square
    legend_pos='center right'         # omit if no color grouping
)

# Save to SVG
mymap.savemap('facility_map.svg')
```

### Key parameters
| Parameter | What it does |
|---|---|
| `view` | Bounding box: `[lon_min, lon_max, lat_min, lat_max]` |
| `projection` | Map projection — `'AlbersEqualArea'` recommended for US |
| `inner_borders` | List of countries to show internal borders for: `['USA']` |
| `latitude` / `longitude` | Column names in DataFrame, or lists of values |
| `color` | Column name (categorical or numeric) or single color string |
| `size` | Single value or column name for variable marker size |
| `palette` | Color palette name (e.g. `'Set2'`, `'YlOrRd'`) |
| `legend_pos` | Legend placement (e.g. `'center right'`, `'lower left'`) |
| `legend_show=False` | Suppress legend (use with `add_colorbar()` instead) |

### Adding a colorbar instead of a legend
```python
mymap.add_markers(latitude='latitude', longitude='longitude', data=df,
                  color='value_column', palette='YlOrRd', n_colors=6,
                  legend_show=False)
mymap.add_colorbar()
mymap.savemap('facility_map.svg')
```

### Downloading from Colab
After running `savemap()`, the SVG appears in the Colab file browser (folder icon, left sidebar). Right-click the file → **Download**.

### Useful links
- OneWorld documentation: https://oneworld.readthedocs.io
- OneWorld tutorials: https://oneworld.readthedocs.io/en/latest/tutorials.html

---

## Power BI Reference

Use this section when generating step-by-step instructions for any Power BI-based
visualization. Reference these details so instructions are button-level specific.

### Interface layout
1. **Ribbon** — common tasks for reports and visualizations (top bar)
2. **Report view / canvas** — where visualizations are created and arranged
3. **Pages tab** — bottom of screen; select or add report pages
4. **Visualizations panel** — change chart type, customize colors/axes, apply filters,
   drag fields
5. **Fields panel** — drag columns from your dataset onto the canvas or into the
   Visualizations panel

### Importing data from Excel
1. Open Power BI Desktop. On the welcome screen, select **"Import data from Excel"**
   (also accessible from the main page via **"Excel workbook"**).
2. Navigate to your Excel file and select **Open**.
3. Check the box next to the sheet(s) you want to import. Select **Load**.
4. Your data is now available in the **Data** section (left-hand navigation bar).
5. Switch to **Report View** (canvas icon, left nav) to begin building.

> Power BI reads columns with headers and can pull from multiple sheets.

### Chart-specific setup

#### Line chart
1. In the Visualizations panel, select the **Line Chart** button.
2. Drag your time/category field to **X-Axis**; drag your measure to **Y-Axis**.
3. Select the **Format icon** (paintbrush) to edit color, font, text size, title,
   data labels, and legend.

#### Clustered bar chart
1. In the Visualizations panel, select the **Clustered Bar Chart** button.
2. Drag your category field to **Y-Axis**; drag your measure to **X-Axis**.
3. Select the **Format icon** to edit color, font, text size, title, labels, legend.

#### Map (bubble)
1. In the Visualizations panel, select the **Map** button.
2. Drag fields to: **Latitude**, **Longitude**, and **Bubble size**.
3. Select the **Format icon** to edit color, font, map style, and title.

#### Shape map (choropleth)
1. First enable it: **File → Options and Settings → Options → Preview Features**,
   then check **Shape map visual**. Restart Power BI.
2. Select the **Shape Map** button in the Visualizations panel.
3. Drag your location field to **Location** and your measure to **Color Saturation**.
4. Select the **Format icon** to edit color, zoom, and title.

#### Sankey diagram
> Sankey is not installed by default — it must be imported first.
1. In the Visualizations panel, select **"Get more visuals"** (the three-dot or
   marketplace icon).
2. Search **"Sankey"**, select the visualization, and select **Add**.
3. The Sankey button now appears in the Visualizations panel. Select it.
4. Drag fields: origin → **Source**, destination → **Destination**,
   quantity → **Weight**.
5. Select the **Format icon** to edit link color, scale, labels, and title.

### Editing page size
1. Click on the report page background (not on any visualization).
2. In the Visualizations panel, select the **Format icon**.
3. Adjust page size as needed.

### Useful Power BI resources
- Getting started guide: https://learn.microsoft.com/en-us/power-bi/fundamentals/desktop-getting-started
- Power BI community: https://community.powerbi.com
- Additional visuals marketplace: https://appsource.microsoft.com
- Color ramps: http://colorbrewer2.org
- World map shapes: https://github.com/deldersveld/topojson
- US map shapes: https://gist.github.com/sdwfrost/d1c73f91dd9d175998ed166eb216994a

---

## Workflow Summary

1. **Intake** — ask the three questions (goal, data, deliverable)
2. **Select type** — match to the catalog; confirm with researcher
3. **Generate instructions** — full numbered walkthrough using the instruction template
4. **Checklist** — append the relevant best practices checklist
5. **Follow-up** — offer to help with any step the researcher gets stuck on
