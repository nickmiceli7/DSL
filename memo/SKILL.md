---
name: memo
description: >
  Use this skill whenever the user wants to create a professional memo or memorandum of any kind.
  Triggers include: "write a memo", "draft a memo", "create a memorandum", "memo to [person]",
  "kick-off memo", "meeting memo", "project memo", "status memo", or any request that involves
  producing a formal internal or external memo document. This skill should be used even for
  casual requests like "quick memo to the team" or "memo about the meeting next week".
  Produces a polished Word (.docx) file following professional memo conventions.
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



Last modified: 2026-08-26

# Memo Creator

Produces professional Word (.docx) memos following established conventions drawn from real-world
research lab and organizational memo practice, and the DSL Brand Guide (Aug 2026) typography
standard for Word documents.

## Memo Anatomy

A well-structured memo contains these sections (include only those relevant to the user's request):

### 1. Header Block (always required)
Four fields, left-aligned with labels in bold and a tab to align values:

| Field | Notes |
|-------|-------|
| **To:** | Recipient name(s) and organization/title |
| **From:** | Sender name and role/organization |
| **Date:** | Full date (e.g., January 14, 2026) |
| **Ref:** | Subject line — bold, concise, descriptive |

### 2. Opening Paragraph (always required)
- Sets context and purpose of the memo
- Warm but professional tone; first-person plural ("we") for organizational voice
- 2–3 sentences max

### 3. Body Sections (as needed)
Each section has a **bold heading** followed by content. Common section types:

- **Meeting Availability** — numbered list of date/time options
- **Proposed Agenda** — numbered list with lettered sub-items for nested points
- **Background / Context** — prose paragraphs
- **Key Findings / Recommendations** — numbered list
- **Next Steps** — numbered action items
- **Open Business** — catch-all for additional items

### 4. Team Members (optional)
Include when introducing a team to an external party. Show: Name (bold), Title/Role, Location, Email.

### 5. Closing Line (always required)
One sentence, warm and forward-looking. Example: "We look forward to working with you on this important project."

### 6. Distribution (optional)
Underlined section heading. List recipients grouped by organization, each group bold.

---

## Tone & Style Guidelines

- **Formal but collegial** — professional language, no slang; warmth is appropriate
- **First-person plural** ("we", "our") for team/organizational memos; use "I" only for individual memos
- **Active voice** preferred
- **Concise** — memos are not essays; every sentence should earn its place
- **Consistent formatting** — parallel structure in lists

---

## Information Gathering

Before drafting, identify what you need. If the user hasn't provided these, ask:

**Always needed:**
- To (recipient name and organization/role)
- From (sender name and role)
- Subject/Ref line
- Purpose of the memo

**Often needed (ask if relevant to the memo type):**
- Key dates, agenda items, or availability windows
- Team members to introduce
- Distribution list
- Any attachments or enclosures to reference

If the user's request is very brief (e.g., "write a kick-off memo"), ask for the missing details before drafting. If they've provided enough context, proceed and note any assumptions.

---

## Logo

The DSL logo PDF is a project file at `$DSL_LOGO_PDF`. Do not ask the user to upload a logo.

**Always rasterize the logo from the PDF at runtime** — never embed a PNG from an upload, as transparent PNGs get flattened to black when uploaded to Claude. The PDF is the reliable source.

```bash
pdftoppm -r 150 -png $DSL_LOGO_PDF /tmp/dsl_logo
```

Then crop white margins:
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

Then embed `/tmp/dsl_logo_clean.png` in the docx. Display at **250pt wide** (height scales proportionally).

**Do not apply the image border/shadow rule (see below) to the logo.** The DSL Brand Guide's
Logo Misuse rules prohibit adding a drop shadow to the logo under any circumstances — the
border/shadow treatment below applies only to non-logo images (photos, figures) placed in the
document body.

---

## Output Format

Always produce a `.docx` file. Follow the docx skill for technical implementation.

### Key Formatting Rules

1. **Page**: US Letter (8.5" × 11"), 1-inch margins on all sides
2. **Font**: Calibri throughout — per the DSL Brand Guide (Aug 2026), Calibri is the
   designated font for internal working pieces, specifically DSL memorandums, reports, and
   articles made in Microsoft Word. (Verdana is reserved for technical bulletins and
   PowerPoint — do not use it here.)
3. **Typography sizes** (per DSL Brand Guide Formatting standard):
   - Title (the "Memorandum" heading): 20pt, bold
   - Section headings (Meeting Availability, Proposed Agenda, Background, etc.): 16pt, bold
   - Any sub-headings within a section (rare in memos, but if used): 14pt, bold
   - Body text, header block values, closing line: 11pt, regular
   - Table text (if any): 10pt, regular
   - Header/Footer text: 9pt, regular
   - Captions (if any figure/image is captioned): 9pt, italic
4. **Spacing** (per DSL Brand Guide Formatting standard):
   - Line spacing: 1.15
   - Paragraph spacing after: 6pt
   - Section heading spacing before: 24pt, after: 12pt
5. **Header block**: Use tab stops to align values; bold the field labels (To, From, Date, Ref)
6. **Ref line**: Bold value text (not just label)
7. **All lists**: Use `LevelFormat.DECIMAL` for all lists — never unicode bullets and never `LevelFormat.BULLET`
8. **Numbered lists**: Use `LevelFormat.DECIMAL`; for sub-items use `LevelFormat.LOWER_LETTER`
9. **Distribution heading**: Underlined (use `underline: {}` on the TextRun)
10. **Team member names**: Bold

### Image Formatting (non-logo images only)

Per the DSL Brand Guide, any photograph or figure placed in a document (not including
social media, and not including the DSL logo — see Logo section above) must have a
**black border** and an **"Offset: Bottom" shadow** applied. In the `docx` implementation,
apply this via the picture formatting equivalent of Word's Picture Format → Picture
Effects → Shadow → Outer → "Offset: Bottom", plus a solid black picture border.

### Header Block Implementation

Use a paragraph with tab stops to create the To/From/Date/Ref alignment:

```javascript
// Tab stop at 1 inch (1440 DXA) for value alignment
new Paragraph({
  tabStops: [{ type: TabStopType.LEFT, position: 1440 }],
  children: [
    new TextRun({ text: "To:", bold: true }),
    new TextRun({ text: "\t" }),
    new TextRun({ text: "Recipient Name, Organization" }),
  ]
})
```

Repeat for From, Date, Ref (bold the Ref value text as well).

---

## Example Structure (Kick-off Meeting Memo)

```
[LOGO]

[MEMORANDUM — 20pt bold]

To:     [Recipient, Organization]
From:   [Sender, Role/Lab]
Date:   [Full Date]
Ref:    [Bold subject line]

[Opening paragraph — context and purpose]

[Section Heading — 16pt bold]
[Content — prose or list]

[Section Heading — 16pt bold]
[Numbered agenda with sub-items]

[Closing line]

[Team Members — if applicable]

[Distribution — if applicable]
```

---

## Workflow

1. **Gather info** — confirm To, From, Date, Ref, and content details with user
2. **Draft content** — write the memo text following tone/style guidelines above
3. **Rasterize logo** — run `pdftoppm` on `$DSL_LOGO_PDF`, then crop (see Logo section)
4. **Build docx** — follow the docx skill; use JS with the `docx` npm package; apply Calibri
   throughout, the Title/Heading size ladder, 1.15 line spacing, and the image border/shadow
   rule to any non-logo images
5. **Validate** — run `python scripts/office/validate.py memo.docx`
6. **Present** — deliver the file to the user

Read `/mnt/skills/public/docx/SKILL.md` for full technical implementation details before writing any code.

---

## Provenance note

Font, heading sizes, spacing, and the image border/shadow rule in this version were updated
2026-08-26 to match `202608_DSLBrandGuide_v01.pdf` (DSL Brand Guide, Aug 2026), which the
DSL project has designated as the authoritative source for these standards. The previous
version of this skill specified Arial 11pt throughout with no explicit heading-size ladder
or image treatment rule — both were superseded by this update.
