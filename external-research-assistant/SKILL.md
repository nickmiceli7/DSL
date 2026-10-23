---
name: external-research-assistant
description: >
  Conduct structured external web research with cited findings, source evaluation,
  uncertainty flags, and research gaps. Use when a DSL researcher asks Claude to
  research, look up, investigate, compare, verify, scan, summarize, or gather
  information from external sources. Especially use for sustainability landscape
  reviews, energy and trade data gathering, ESG and policy research, company or
  industry background research, and evidence gathering where source quality,
  citation traceability, and factual accuracy are required. Trigger on research-intent
  language: "research," "investigate," "find sources on," "literature scan," "landscape
  review," "look into," "gather evidence on," "what does the research say about," or
  similar phrasing indicating a structured information-gathering task rather than a
  simple factual lookup. Do NOT trigger for casual one-off questions that require a
  single web search and a direct answer.
---

# External Research Assistant

## Purpose

Conduct structured external research and return a transparent, cited research response.
Gather information from authoritative sources, synthesize findings with in-text citations,
and clearly flag uncertainty throughout.

The goal is not only to answer the researcher's question, but to show:
- what information was found
- where each piece came from
- how reliable each source appears to be
- where assumptions were made
- where evidence is weak, outdated, conflicting, or incomplete
- what remains unknown or requires further investigation

This skill is calibrated for DSL research domains: sustainability, energy systems,
trade and supply chains, ESG frameworks, climate policy, and corporate environmental
reporting. It applies to general research tasks as well, but source prioritization
and uncertainty standards are tuned for these domains.

---

## Core Rules

1. State all assumptions before presenting any findings.
2. Use APA-style plain-text in-text citations for every factual claim. Platform-native
   citation features (hover links, source chips, auto-generated citation markers) do NOT
   satisfy this rule, even when the interface renders them automatically.
3. Prefer primary and authoritative sources. See the Source Hierarchy below.
4. Do not hide uncertainty — flag it directly where it appears.
5. Distinguish between confirmed findings, likely interpretations, assumptions,
   and unsupported or weakly supported claims.
6. Classify unknown information as unknown. Never fill a gap with inference
   presented as fact.
7. Never fabricate: sources, citations, statistics, dates, URLs, author names,
   organization names, study findings, quotes, methodology details, or conclusions.
8. Include a Sources section at the end with credibility notes.
9. Include research gaps for substantive requests.
10. Do not overstate the certainty of a finding. Use careful confidence language.
11. Before returning the final response, complete the Step 8 Verification Pass. It is a
    blocking gate: the response must include the visible Verification Checklist, and a
    response without it is incomplete and must not be sent.
12. Flag data vintage explicitly for fast-changing domains (energy prices, policy,
    emissions inventories, trade flows). Data older than 24 months in these domains
    should be flagged as potentially outdated.
13. Flag greenwashing risk when a source has a commercial or advocacy interest in
    the sustainability claims it makes.

---

## Source Hierarchy

Apply this hierarchy when selecting and weighting sources. Always reach for higher-tier
sources before falling back to lower tiers. Note the tier in the Sources section.

### Tier 1 — Primary and institutional sources
- Government agencies and regulatory bodies (EPA, DOE, USDA, EU Commission, etc.)
- International institutions: IEA, OECD, UN agencies (FAO, UNEP, UNIDO), World Bank,
  UN Comtrade
- Peer-reviewed research and academic reviews
- SEC filings and official company financial disclosures (EDGAR)
- Official press releases for company-specific factual claims

### Tier 2 — SU Library databases
Preferred for market, industry, and company research. DSL researchers can access
these through Syracuse University. Claude cannot query these directly — when a
research gap points to a Tier 2 database, that will be noted in Research Gaps with
suggested search terms. If a researcher pulls PDFs or exports from these databases
and brings them back to the conversation, Claude will incorporate them into the
findings.

- **IBISWorld** — industry reports, outlooks, risk ratings, NAICS lookup
- **Business Insights: Essentials** — company profiles, financial reports, SWOT analyses
- **MarketLine** — company, country, industry, and product reports
- **Mergent Intellect / Mergent Online** — public and private company financials,
  SEC/EDGAR filings, international company data
- **PrivCo** — private company profiles, VC/PE funding, M&A (requires syr.edu login)
- **Statista** — cross-sector data, forecasts, consumer and digital markets
- **eMarketer** — digital marketing, media, and ecommerce data
- **MarketResearch.com Academic** — sector reports including healthcare, tech, food/bev
- **Hoover's / Plunkett Research Online** — company and industry profiles
- **Bloomberg Terminal** — industry, financial, commodity, and risk data
  (located at Whitman School of Management; requires in-person access or introduction
  from a DSL fellow)
- **ProQuest Historical Annual Reports** — U.S. company annual reports

### Tier 3 — DSL-approved external research sites
Use these for domain-specific data before falling back to general web search:
- **Global trade:** UN Comtrade (comtrade.un.org), World Bank WITS (wits.worldbank.org),
  OEC (oec.world)
- **Country economic data:** OECD Stats (stats.oecd.org), Harvard Atlas of Economic
  Complexity (atlas.cid.harvard.edu)
- **Energy policy and data:** IEA Policy Database (iea.org/policies), BP Energy
  (bp.com energy charting tool)
- **Wind and renewable energy:** Tethys PNNL (tethys.pnnl.gov), 4C Offshore,
  Wind Exchange (windexchange.energy.gov)
- **ESG and sustainability reporting:** WBCSD Credible Goals Database
  (embeddingproject.org), Corporate Register (corporateregister.com)
- **Mineral resources:** USGS MinDat
- **Human rights / conflict / corruption:** Human Rights Atlas, CFR Global Conflict
  Tracker (cfr.org), Transparency International CPI (transparency.org)

### Tier 4 — High-quality secondary sources
- Academic reviews and meta-analyses
- Reputable consulting and analyst reports (McKinsey, Deloitte, Rocky Mountain
  Institute, WRI, NRDC, Bloomberg NEF, S&P Global)
- Recognized trade and science publications (Nature, Science, Energy Policy,
  Environmental Science & Technology, GreenBiz)

### Tier 5 — Supporting sources (use with explicit flags)
- Expert commentary and blog posts
- Company sustainability reports and ESG disclosures
  (**Greenwashing flag required** when source has commercial interest in claims)
- Advocacy organization reports (flag potential bias)
- News articles and conference summaries

---

## Internal Sources First

Before any external search, check whether relevant DSL internal materials exist.
The project SharePoint is accessible via the Microsoft 365 connector at
`sumailsyr.sharepoint.com/sites/DSLAI`. Use `sharepoint_search` for DSL documents,
datasets, or prior research before going to the web. Note in the Assumptions section
whether internal sources were checked and what was found (or not found).

---

## Workflow

### Step 0 — Workload Gate

Before beginning research, estimate the likely scope:

- **Light** (1–5 web searches, single focused question, no synthesis across sources)
  → proceed directly to Step 1.
- **Moderate** (6–15 web searches, multi-source synthesis, defined scope)
  → proceed directly to Step 1, but note the estimated tool-call range in the
  Assumptions section.
- **Heavy or Very Heavy** (15+ web searches, broad landscape review, multiple
  domains or geographies, extended synthesis)
  → **stop before beginning research.** State the estimated tool-call range and what
  drives it (number of sub-questions, domains, or geographies), and ask the researcher
  directly whether to proceed as scoped, narrow the request, or defer it. Do not begin
  the research workflow until the researcher confirms one of those three options.

There is no automated workload-estimation step for this gate — DSL retired the
Workload Calculator skill, and workload/time estimation is deferred lab-wide pending
calibrated ledger data (see `work-recorder-skill.md`). The Heavy/Very Heavy check above
is a judgment call Claude makes and puts to the researcher directly, not a calculation.

If scope is ambiguous, ask one question to clarify before estimating.

### Step 1 — Scope Definition

Determine the research scope from the prompt. Infer where possible:
- research question
- geography
- timeframe
- domain (energy, trade, ESG, supply chain, policy, company/industry, other)
- intended use (internal reference, memo input, literature scan, fact-check, etc.)
- desired depth

If the scope is clear enough, proceed and state assumptions. If the prompt is too
broad and missing details would materially change the research direction, ask 1–3
focused intake questions. Do not ask unnecessary questions for clearly scoped prompts.

### Step 2 — Assumptions

Every response must open with an **Assumptions** section before any findings.

List all scope assumptions explicitly:
- Geography
- Timeframe
- Domain focus
- Source tiers prioritized
- Use case assumed
- Whether internal DSL SharePoint sources were checked

If no major assumptions were made, say so briefly.

### Step 3 — Research Plan

Break the request into sub-questions. Identify:
- main question and key sub-questions
- likely source tiers and specific sources
- search angles and key terms
- areas where conflicting or incomplete evidence is likely

For simple fact-checks, keep this internal. For larger requests, surface the plan
briefly so the researcher can redirect before synthesis begins.

### Step 4 — Source Collection

Collect sources in tier order. Do not skip tiers without reason. If a Tier 1 or
Tier 2 source is not used for a major claim, note why in Research Gaps.

For sustainability and energy claims, actively prefer Tier 3 DSL-approved sources
over general web results when both are available.

### Step 5 — Source Evaluation

Before weighting a source heavily, evaluate:
- source type and tier
- publication date and data vintage
- methodology availability
- relevance to the research question
- potential bias: commercial, political, or advocacy
- corroboration by other sources
- for sustainability sources: does the publisher have a commercial interest in the
  claim? (**Greenwashing flag**)

Assign a confidence level (High / Medium / Low) to each major finding.

### Step 6 — Synthesis

Organize findings by theme, sub-question, geography, or evidence strength.
Every factual claim gets an in-text citation, placed immediately after that
specific claim — not clustered at the end of a paragraph. Uncertainty flags
appear inline, directly next to the uncertain claim.

**Placement rule (hard rule):** A citation attaches to the claim it supports,
at the point that claim is made. If a paragraph contains multiple facts, each
fact gets its own citation immediately following it — never a single list of
sources or links dropped at the end of the paragraph. A reader should never
have to work backward from a trailing citation cluster to figure out which
source supports which sentence.

- **Correct:** "The facility reduced water withdrawal by 18% between 2021 and
  2024 (EPA, 2024). Over the same period, energy intensity fell by 9%
  (DOE, 2023)."
- **Incorrect:** "The facility reduced water withdrawal by 18% between 2021
  and 2024. Over the same period, energy intensity fell by 9%. (EPA, 2024;
  DOE, 2023)"

This applies even when the same source supports several consecutive claims —
repeat the citation after each claim rather than citing it once and letting
it silently cover everything that follows.

**Uncertainty flag conventions:**
- `[Uncertainty: single source, no corroboration]`
- `[Uncertainty: data vintage — published YYYY, domain moves quickly]`
- `[Uncertainty: source has commercial interest in this claim — greenwashing risk]`
- `[Uncertainty: methodology not disclosed]`
- `[Uncertainty: sources conflict — see Research Gaps]`
- `[Uncertainty: estimate based on projection, not observed data]`
- `[Uncertainty: figure from Tier 2 database not directly accessed — researcher
  should verify via SU Library]`

### Step 7 — Research Gaps

For substantive requests, include a **Research Gaps** section. Identify:
- information not found in public sources
- claims supported only by weak or indirect evidence
- conflicting findings and what drives the conflict
- data likely outdated
- information requiring SU Library database access, Bloomberg Terminal, or
  primary research — with specific database recommendations and suggested
  search terms
- DSL-specific gaps (e.g., data that may exist in internal SharePoint but
  was not found)

### Step 8 — Verification Pass (blocking gate)

This step is a gate, not a reminder. It produces a visible output: the
**Verification Checklist** (format below), which appears at the very end of every
research response. A research response without the checklist is incomplete. Do not
send it.

Run the checks against the drafted response, not from memory of having written it.
For each item, mark PASS or FAIL:

1. **Assumptions first.** An Assumptions section opens the response, before any findings
   (substantive requests).
2. **Citation format.** Every factual claim carries an APA-style plain-text citation,
   e.g. (IEA, 2024). No platform-native hover links, chips, or markers were used in
   place of it.
3. **Citation placement.** Each citation sits immediately after the claim it supports.
   No trailing citation clusters at the end of a paragraph.
4. **Sources reconcile.** Every in-text citation has a Sources entry, and every Sources
   entry is cited in the text.
5. **Sources entries complete.** Each entry has source type, tier, and a credibility
   note, plus any uncertainty or greenwashing flag.
6. **Claims traceable.** Statistics, dates, names, and figures match what the source
   actually says. No claim overstates the evidence.
7. **Uncertainty flags.** Present where needed, including data vintage (24-month rule)
   and greenwashing risk. Conflicting evidence is acknowledged.
8. **Research Gaps.** Section present for substantive requests, with specific database
   recommendations where a Tier 1 or Tier 2 source was not used.
9. **No fabrication.** Nothing is invented. Information that could not be verified is
   classified as unknown.

**If any item is FAIL:** fix the response, then re-run the checklist. Do not send a
response with a FAIL item, and do not use the checklist to disclose a known failure
instead of fixing it. The only exception is an item that cannot be fixed (e.g., a
source is paywalled and could not be accessed). In that case, mark it
"UNRESOLVED" with a one-line reason and carry it into Research Gaps.

**Brief or focused requests:** use the one-line form of the checklist (see Output
Format). The gate still applies.

---

## Output Format

### Brief or focused requests
Respond conversationally with inline APA citations and a short Sources section.
Section headers not required unless the response is long enough to need navigation.
End with a one-line checklist: "Verification: citations APA and adjacent to claims,
Sources reconcile, flags present, no fabrication — PASS." (Or list what was
corrected or left unresolved.)

### Substantive research requests

**Assumptions**
[All scope assumptions before any findings]

**Research Question / Scope**
[Brief restatement of question and coverage]

**Key Findings**
[3–7 bullet summary with citations]

**Detailed Findings**
[By theme, sub-question, geography, or evidence strength.
In-text citations throughout. Uncertainty flags inline.]

**Research Gaps**
[What's missing, why, and which specific source or database would close it]

**Recommended Next Steps** *(when useful)*
[Specific databases, search terms, or research actions]

**Sources**
[Title, organization/author, date, source type, tier, URL or reference,
credibility note, uncertainty or greenwashing flags]

**Verification Checklist** *(always last; required)*
[Step 8 results, one line per item, in plain text:]
1. Assumptions first: PASS
2. Citation format (APA, no platform-native markers): PASS
3. Citation placement: PASS
4. Sources reconcile: PASS
5. Sources entries complete: PASS
6. Claims traceable: PASS
7. Uncertainty flags: PASS
8. Research Gaps: PASS
9. No fabrication: PASS
Result: PASS. [Or: "Corrected before sending: <what was fixed>." Or: "UNRESOLVED:
<item> — <reason>; carried into Research Gaps."]

---

## Citation Format

All citations must be portable plain text — no interface-dependent markup, hyperlinks,
or rendering features. Researchers will copy findings into their own documents; citations
must survive that transfer intact.

**Platform-native citations do not count (hard rule).** Some interfaces automatically
attach their own citation mechanism to web-search results, such as hover links or source
chips. These are not portable, do not carry the author-year information, and do not
produce a Sources section. They are never a substitute for the APA-style citations and
Sources section below. Write the APA citations and the Sources section yourself in
every research response, whether or not the interface also renders its own.

- **In-text format:** APA-style parenthetical — (Author or Organization, Year) — e.g.,
  (NIST, 2023) or (Luccioni et al., 2024). Use this format consistently for every
  factual claim.
- **Placement, not just format:** the APA format alone is not sufficient — the citation
  must sit directly after the claim it supports, inside the same sentence or immediately
  following it. Do not satisfy this rule by appending a correctly-formatted citation, or
  a run of several citations, to the end of a paragraph that contains multiple claims.
  One claim, one citation, immediately adjacent — repeated for every claim in the
  paragraph.
- **Author rule:** For institutional sources, use the organization name (NIST, IEA,
  EPA). For authored papers, use the first author's surname followed by "et al." if
  more than two authors.
- **Full entry in Sources section:** Every in-text citation must have a corresponding
  entry in the Sources section at the end. The Sources section is the authoritative
  list — if something is cited inline, it must appear there.
- **Sources section format:** Author/Org, Title, Publisher/Outlet, Year, URL or DOI
  if available, source type, tier, credibility note, any uncertainty or greenwashing
  flags.
- **Conflicts:** If sources conflict on a claim, cite both inline — e.g., (IEA, 2024;
  BloombergNEF, 2024) — and explain the conflict in the Research Gaps section.
- **Tier 2 database figures:** If a figure comes from an SU Library database not
  directly accessed, note this inline — e.g., (IBISWorld, 2024 — not directly
  accessed; verify via SU Library).

---

## Confidence Language

- "indicates," "suggests," "appears" — partial evidence
- "shows," "states" — directly supported by a strong source
- Never use "definitively" unless evidence is strong, current, and multiply corroborated
- For ESG claims: distinguish self-reported corporate data from independently
  verified data

---

## DSL Domain Notes

### Energy and policy data
Prices, policy databases, and emissions inventories change frequently. Flag anything
older than 24 months. Prefer IEA, EIA, and DOE for emissions and capacity figures.

### ESG and sustainability reporting
Corporate sustainability data is largely self-reported and inconsistently methodologized
across frameworks (GRI, SASB, TCFD, CDP, GHG Protocol). When using ESG data:
- Note which framework generated it
- Flag Scope 3 comparability risk across companies
- Flag unverified corporate sustainability report data (greenwashing risk)
- Prefer Corporate Register, CDP disclosures, or SEC climate filings over
  company website claims

### Trade and supply chain data
Prefer UN Comtrade, WITS, and OEC. Note that trade data typically lags 12–24 months.
Flag mirror data discrepancies (importer vs. exporter reported figures) if both are cited.

### Market and industry research
Prefer Tier 2 SU Library databases for market sizing and industry structure. Note
when a market figure comes from a report behind a paywall not directly accessed.

---

## Thresholds Note

The Light / Moderate / Heavy workload thresholds in Step 0 are provisional and are
judged by Claude, not computed by any DSL tool — the Workload Calculator skill this
step previously deferred to has been retired. If the thresholds need recalibrating,
that is a DSL decision, not something this skill derives on its own.

---

## What This Skill Does Not Do

- Does not access SU Library databases directly — directs researchers to the right
  database and search terms; researchers run the query themselves
- Does not produce a formatted downloadable report — findings are returned inline
- Does not replace expert judgment, primary research, or peer review
- Does not verify information behind paywalls it cannot access

---

## Provenance note

This version (2026-10-23) hardens citation and verification compliance: Step 8 is now a
blocking gate with a visible Verification Checklist, and platform-native citation
mechanisms are explicitly ruled out as a substitute for APA-style citations. These
changes follow a session in which both requirements were skipped. The prior version
(2026-08-27) removes the prior reference to `/mnt/project/energy-model.md`
in the Thresholds Note, which was found not to exist in the project or on SharePoint.
The Light/Moderate/Heavy thresholds remain a Claude judgment call, unanchored to any
project reference file, pending a DSL decision on whether such a file should be created.
