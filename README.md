# DSL Claude Skills

A public Claude plugin marketplace for the Dynamic Sustainability Lab. Each folder in this
repo is one Claude plugin, wrapping one DSL skill. Edits pushed to this repo's default branch
propagate to anyone who has installed the plugin, once org/personal auto-sync picks up the
change — see the caveats below before assuming that happens instantly.

## What's in here

| Plugin | What it does |
| --- | --- |
| `output-style` | Standing conversational tone/formatting rules for DSL sessions. |
| `output-evaluator` | Evaluate the quality of a DSL deliverable against DSL's standard. |
| `ai-energy-emissions-report` | Estimate energy/GHG emissions of a Claude session, DSL-branded PDF. |
| `external-research-assistant` | Structured, cited external web research with source evaluation. |
| `figures` | Guidance for building figures, charts, maps, and tables for DSL work. |
| `excel` | Create a new DSL-formatted Excel spreadsheet or tracker. |
| `memo` | Create a DSL memo as a polished Word (.docx) file. |
| `technical-bulletin` | Scaffold and generate a DSL technical bulletin as a .pptx file. |
| `work-recorder` | Maintain a DSL fellow's session ledger. |
| `ai-usage-assessment-per-chat` | Score one chat against the four AI fluency competencies. |

Deliberately **not** included: `consult-with-dominick` / `consult-with-naoki` (personal
consult-booking skills, contain personal information).

## Installing a plugin (personal install, no admin role needed)

Each fellow does this on their own account — nothing here requires org-level admin access:

1. On claude.ai: **Customize → Plugins → Personal plugins → "+" → Add marketplace → Add from a
   repository**, and point it at this repo's URL.
   In Claude Code: `/plugin marketplace add <org>/dsl-claude-skills`.
2. Install whichever plugin(s) you want from the list.
3. Because this repo is public, no GitHub account or repo-access grant is needed to do this.

## Known gap: update propagation is not instant or guaranteed

Making the repo public removes the "does everyone need a GitHub account" problem. It does
**not** by itself guarantee that an already-installed plugin picks up a change the moment it's
pushed. There's a documented gap between "the marketplace re-synced" and "the installed plugin
actually shows the new content" — sometimes a fellow has to manually check for updates or
remove-and-reinstall the plugin to see a change. Build that step into training rather than
assuming silent auto-propagation.

## Bundled assets

`ai-energy-emissions-report`, `excel`, `memo`, and `technical-bulletin` each bundle the DSL
logo PDF(s) they need under their own `assets/` folder, rather than assuming `/mnt/project/`
(project-knowledge) has them. Each of those SKILL.md files has a "Logo asset resolution" note
near the top flagging that the exact bundled-asset runtime path has **not yet been verified**
against current Claude plugin documentation — test that resolution step for real once a plugin
is actually installed, before relying on it. The `/mnt/project/` fallback keeps things working
in the meantime for anyone still running these as project-knowledge files instead of the
plugin.

## Versioning / update discipline

There is no plugin-level version pinning — the marketplace tracks whatever is on the default
branch. If you want a staging step before a change reaches everyone, enforce it yourself: work
in a branch or PR, review, then merge. A bad edit merged to the default branch reaches every
fellow with the plugin installed at their next sync, silently.
