# Gap Review — `PROMPT_KOL_Centre_Map_v2`

Why different LLMs give very different outputs, and what to change. Issues are ranked by how much variance they cause.

---

## Why outputs vary (summary)

Most of the variance comes from five things:

1. **The job is bigger than one run can do.** Your reference build had 200 centres and 76 KOLs. The prompt also asks for at least 15 KOLs per country, which comes to 525+ KOLs for 35 countries, with every one scored, sourced and checked. Each model hits its context or tool budget at a different point and cuts corners in a different way, and the prompt doesn't say which corners can be cut.
2. **The scoring maths is broken** (see A1). Each model fixes it in its own way without telling you.
3. **The tier rules are ambiguous** ("or", "and", semicolons, words like "typically"). With no decision order, each model reads them differently.
4. **The prompt points to things the model can't see.** It says to match the reference workbook exactly but doesn't attach it. It also assumes a `recalc.py` script and live web access. Models without these will guess or make things up.
5. **Many fields are free text** ("qualified", "Yes — competitor trial", open-ended priority labels). Every model words them differently, which also breaks your COUNTIFS formulas.

---

## A. Critical: the KOL scoring is internally inconsistent

| # | Hole | Effect | Fix |
|---|---|---|---|
| A1 | **The score range doesn't match the tier bands.** There are 7 categories. Six score 1/3/5 and Awards scores 1–2, so the real range is **6–32**. Tier 1 is 24–30, so **31–32 has no tier**. Tier 3 starts at 0, but no one can score below 6. | Models handle edge cases differently, or quietly change the bands. | Re-band to the actual range (e.g. T1 ≥24, T2 14–23, T3 ≤13), or rescale Awards to 1/3/5 and use 7–35. |
| A2 | **"Default to 1" conflicts with "do not infer."** Someone with no evidence still gets 7 points, the same as someone who meets every 1-point criterion. | Some models give 0, some give 1. Totals shift by up to 7 points, which can move people across tiers. | Allow **0 = no evidence found**. Keep 1 for evidence that meets the 1-point row. |
| A3 | **Awards breaks the "1, 3 or 5 only" rule.** Awards is "1–2", with "—" for the 3 and 5 rows. Should no award score 0 or 1? | Different handling by each model. | Define Awards as 0/1/2 and say so in the rules, or drop it. |
| A4 | **The levels overlap or are in the wrong order.** "Research group member" appears at both 1 and 3 points. "Retired department head" scores 3 but "emeritus professor" scores 5. A site PI on a pivotal trial gets 5, but a national PI on a multi-country trial gets only 3. | The same person gets different scores. | Make each level clearly distinct and in order. Give one worked example per category. |
| A5 | **Undefined terms:** "reputable journals", "relevant", "minor/regional guideline", "occasional" vs "regular" speaker, "calibre of award". | Each model sets its own threshold. | Define them: e.g. "relevant = disease or its ICD block", "reputable = PubMed-indexed", "regular = ≥2 national talks in 3 years". |
| A6 | **The publication levels aren't comparable.** The 1-point level has no time window, while the 3- and 5-point levels use 7 years. Elsewhere the prompt says "~5 years" (P2, Centre Tier 1) and "5–7 years" (rules). | Different time windows give different counts. | Use **one window**, anchored to an **as-of date** (e.g. "1 Jan 2021 – run date"). |
| A7 | **The scores aren't shown.** KOL_Roster has a fixed 10 columns and no per-category scores, so the tier can't be checked. Some models add score columns (breaking the "10 columns" rule) and others show only the tier. | Results can't be audited or compared between models. | Add 7 score columns + Total + a scoring-evidence/URL column, or a separate **KOL_Scoring** tab. |
| A8 | **"TOV" vs "TOF".** The column says "KOL Tier for TOV" and the bullet says "TOF". Neither is defined. | Models guess (transfer of value? fair-market-value tier?). | Define it, e.g. "TOV = transfer-of-value / FMV tier per [company] HCP compliance policy", and use one spelling. |
| A9 | **Two tier systems in one row.** "Engagement priority" uses `TIER 1/2` labels and sits next to "KOL Tier". It isn't clear whether those labels mean the centre tier or the KOL score tier, and there's no TIER 3 priority label. | Labels contradict each other within a row. | Rename to `PRIORITY A/B/C`, or say outright: "Engagement priority = KOL score tier + qualifier". Make the qualifiers a fixed list. |
| A10 | **"Physician" only.** KOL scoring says "each physician". But you also want reference-lab PIs, who are often PhD scientists, and possibly neurocritical-care staff, neurophysiologists and rehab staff. | Some models include non-MDs and some leave them out. | List the eligible professions and specialties per disease in Step 0. |

---

## B. Critical: the centre tier rules are ambiguous

| # | Hole | Effect | Fix |
|---|---|---|---|
| B1 | **Tier 2's logic isn't stated.** "P4 HCP with P5, *or* P3 site; university hospital with…; recurring P2". It's unclear whether the semicolons mean AND or OR. | This alone can change centre counts a lot. | Write it as Boolean logic: `T2 = (P4 AND P5) OR P3_site OR (Univ hospital AND P2 ≥ N papers)`. |
| B2 | **No order of rules and no fallback.** What tier is a centre with P1 but no P4? Or P2 but not P4/P5? Or one that fits no tier? | Models put it in a tier as they see fit. | Add a decision tree: test T1 first, then T2, then T3, else leave it out. List the leftover cases. |
| B3 | **Tier 1 requires P4, but P4 is N/A outside the EU.** The N/A rule says to "tier on national signal", but not *what replaces P4* in the Tier 1 test. | Some models put no UK/CH/US centre in Tier 1; others promote freely. | State: "Where P4 = N/A, the national analogue in Appendix A.3 meets the P4 requirement." |
| B4 | **Factual error: Norway is in the ERNs.** The prompt treats it as "Europe-rest" and suggests N/A for non-EU countries. In fact the [European Commission](https://health.ec.europa.eu/rare-diseases-and-european-reference-networks/european-reference-networks_en) says there are 24 ERNs "across the EU and Norway". | Norwegian centres get wrongly marked N/A. | Note that Norway takes part fully, and that some non-EU countries have affiliated-partner status. |
| B5 | **P1 is described in two incompatible ways.** It's "the *single* authoritative guideline… a small, fixed cohort", but also "use country-level guidelines for regional tiering". | Some models count national guideline authors as P1 and some don't. P1 counts can vary a lot. | Split into P1a (international guideline) and P1b (national guideline), or keep P1 international only and record national authorship under P2/Role. |
| B6 | **P2 has no threshold.** "Major", "recurring", "senior" are undefined. When does P2 = Yes? | Different numbers of P2 flags. | E.g. "≥2 papers from the centre in the window, ≥1 with senior/corresponding authorship, or site in a listed NCT." |
| B7 | **"Typically not research-active" (Tier 3)** is a description, not a rule. | Unclear what to do with research-active hospitals that have no P4. | Define T3 by what it has (infrastructure) and what it lacks (T1/T2 signals). |
| B8 | **No rule for what counts as one centre.** Is a multi-site trust or group (e.g. AP-HP, Charité campuses, NHS trusts) one centre or several? Are adult and paediatric hospitals separate? | This is a big source of differing centre counts. | "One row = one physical hospital site. Group-level ERN membership is copied to each site that hosts the unit. Paediatric centres are listed separately [in/out of scope]." |

---

## C. Critical: scale, scope and stopping rules

| # | Hole | Effect | Fix |
|---|---|---|---|
| C1 | **The minimum of 15 KOLs per country conflicts with "no fabrication".** Malta, Liechtenstein, San Marino, Vatican, Andorra, Monaco and Kosovo are in scope but can't realistically produce 15 traceable disease KOLs. | This pushes models to pad the list with weak or invented names, which is the exact failure you want to avoid. | "Target 15. If fewer can be traced, list what exists and record the shortfall in Tier_Counts Notes. **Never pad.**" Scale the target by population or centre count. |
| C2 | **No budget or batching plan.** 35+ countries × centres × KOLs × scoring × URL checks is more than one run can do. | Each model truncates differently, often silently (e.g. full detail for DE/FR, then thin rows). | Allow phased runs (e.g. one region per run, fixed batch order) and add a **Coverage_Log**: per country, searches run, sources checked, completeness %. Require "INCOMPLETE" flags instead of silent truncation. |
| C3 | **"Ask once, then proceed"** doesn't say what to do in non-interactive or agent modes. | Some models stop and wait; others assume. | "If you can't ask, use these defaults: [..] and log them in README." |
| C4 | **"Phase of asset shifts emphasis"** but the prompt never says how. | Each model invents its own emphasis. | Spell out what changes per phase (e.g. pre-approval: P2/P3 weighting and an advisory-board shortlist; launched: T3/T4 focus). |
| C5 | **Competitors are never defined.** P2 depends on "competitor INNs/NCTs", but no step lists them. | Each model finds a different competitor set, so P2 flags differ. | Add **Step 0b**: list competitor programmes (INN, sponsor, phase, NCT, source URL) in README before mapping. |
| C6 | **No as-of date.** "Last ~5 years" and "current" aren't anchored. | Different models use different windows (and their training cut-offs). | "As-of date = run date; window = [as-of − 5y, as-of]. State both in README." |

---

## D. High: things the model can't see or doesn't have

| # | Hole | Fix |
|---|---|---|
| D1 | "Match the reference build **exactly**". The workbook isn't attached, so models imagine it. | Attach it, or include a cell-level schema with 3–5 example rows per tab. |
| D2 | `recalc.py` exists only in some environments (e.g. Claude's xlsx tooling). | "Recalculate using available tooling (e.g. LibreOffice headless). If none, state formulas weren't recalculated." |
| D3 | "Verify URLs resolve" and "search hospital directories" need live browsing. Models without it will produce plausible-looking but unchecked URLs. | Add a precondition: "If you lack live web access, STOP and say so. Don't produce the workbook from memory." |
| D4 | Cross-references point to the wrong sections. README says "KOL Tier definitions (Section 4)", but they're in Section 3. It says "gap-fill methodology (Section 6)", but that's Section 5 (Section 6 is formatting). The exemplar note says "v1 promotes it", but this is v2. | Correct the numbering. |
| D5 | Markdown escaping left over from export: `\\&`, `\\_`, `\\~`, `\\[asset]`, plus typos "valuate" and "PMID.p". The **Appendix A.3 US row is cut off** at "(e.g. GBS\|": the pipe in "GBS\|CIDP Foundation" breaks the table. | Clean the escapes and write "GBS/CIDP Foundation International Centers of Excellence". |

---

## E. High: formulas and data structure

| # | Hole | Fix |
|---|---|---|
| E1 | **Free-text flag values** ("qualified", "Yes — disease / Yes — [asset] trial / Yes — competitor trial", "Affiliated partner"). COUNTIFS can't count these reliably. The README filter "P2 publications = Yes" won't match "Yes — disease". | Use fixed dropdown values (data validation lists). Put any qualification in Notes, not in the flag cell. Use one P2 column for Yes/No plus a separate P2_Type column. |
| E2 | **"SUBTOTAL-compatible" and COUNTIFS conflict.** COUNTIFS ignores filters, and SUBTOTAL can't count with conditions. Tier_Counts also sits on a separate sheet, so filtering Country_Centres has no effect on it. | Choose one approach: COUNTIFS on Tier_Counts (static), and/or SUBTOTAL(103,…) totals on the data tabs. |
| E3 | **The KOL cross-cuts can't be calculated with formulas.** "KOLs with guideline / trial / disease publications" come from free-text columns. | Add Yes/No helper columns to KOL_Roster. |
| E4 | **The sort order is stated two ways:** "Sort by Country then Tier" (4.2) vs "sort by Tier asc then Country" (HOW TO USE). | Choose one for delivery. The other is a user filter instructional note. |
| E5 | **Verification_Flags cell references break** if rows are sorted after the flags are written. | Add a stable `Row_ID` to every data tab and reference that instead of cell addresses. |
| E6 | **No per-row source.** "Every centre and KOL tied to a verifiable signal", but Country_Centres and KOL_Roster have no source URL column. Sources_Caveats lists only the main sources. | Add `Evidence URL(s)` and `Last verified (date)` columns to both tabs. |
| E7 | **The Verdict list is incomplete.** There's no verdict for "Drug premise" and no PASS/VERIFIED value. Mapping compliance issues to "CONTRADICTED" is a stretch. | Fixed list: `VERIFIED / OUTDATED / CONTRADICTED / PART-UNVERIFIABLE / WORDING / COMPLIANCE`. |
| E8 | **"Flag, don't auto-correct" vs the acceptance checklist** (e.g. every Tier 1 must meet the compound test). If a Tier 1 row fails, should the model fix it or flag it? | "Fix build errors (rule misapplication) before delivery; flag only external-fact uncertainty." |

---

## F. Medium: geography definitions

- **The UK is counted twice:** "UK" is in EU4+UK and "GB" is in Europe-rest.
- **Türkiye** is in both Europe-rest and MENA. **Mexico** is in NorthAmerica and could also fit LATAM.
- **China** is in "APAC-developed", which is a questionable label and will affect how much data to expect.
- **Israel** is in "MENA" but the tag is described as "MENA/Gulf".
- **"International"** is used as an example tag but never defined.
- **"Pan-American body"** in LATAM is vague. Does it mean PAHO, or a regional society?
- **"[Region]" in the filename** is undefined for combined tags, and "UPDATED" plus "Add MENA" suggest you're adding to an existing workbook. But the prompt never says whether to merge into a prior file or build a new one.

Fix: each country appears in exactly one tag. Define "International". Set a filename rule for combined regions. Add an explicit "extension mode" section (input = prior workbook; append rows; keep existing Row_IDs).

---

## G. Medium: compliance and content risk

- **GDPR / data minimisation.** Telling models to search hospital directories, with no limits, leads some to collect emails, phone numbers or photos. State: "Professional name, title and affiliation only. No personal contact data."
- **Promotional framing.** The Future-state example ("single-dose IV profile is infrastructure-light…") is specific to GBS and tanruprubart. Models will copy it for other assets, and for a pre-approval asset in a Medical Affairs document, comparative claims can be a promotional-compliance risk. Make it a neutral template: "Describe the asset's administration requirements vs SoC, citing the label/SmPC or trial publication."
- **Infrastructure "Typical Tier 1–2 coverage"** is an estimate with no stated source. Require a source, or label it "Assessment (unsourced)".
- **The asset premise is time-sensitive.** For tanruprubart, the EU MAA is under review and a US BLA is planned for Q4 2026 ([NeurologyLive](https://www.neurologylive.com/view/tanruprubart-shows-rapid-responses-first-forward-cohort-guillain-barre-syndrome)). Models working from memory will report out-of-date status. Require the regulatory status to be checked live against originator press releases on the run date.

---

## Verified anchors (no change needed)

- The EAN/PNS 2023 GBS guideline (van Doorn et al., Eur J Neurol 2023;30:3646–74) is correctly cited ([PubMed](https://pubmed.ncbi.nlm.nih.gov/37814552/)).
- 24 ERNs is correct ([European Commission](https://health.ec.europa.eu/rare-diseases-and-european-reference-networks/european-reference-networks_en)).
- The tanruprubart Phase 3 met its primary endpoint on GBS-DS at Week 8 ([Annexon EAN 2026 poster](https://annexonbio.com/wp-content/uploads/2026/07/EAN-2026-Safety_ePoster_v5.0_22Jun2026_Final.pdf)).

---

## Top 10 fixes, in priority order

1. Re-band the KOL scores to the real range and allow 0 for no evidence.
2. Rewrite the Centre Tier 1–3 rules as Boolean logic with a test order and P4-substitution rule.
3. Make the 15-KOL minimum a target with a "never pad, log the shortfall" rule.
4. Add batching or phased runs and a Coverage_Log; ban silent truncation.
5. Add Step 0b (competitor list) and a fixed as-of date and time window.
6. Use fixed dropdown values for all flag, verdict and priority columns.
7. Add per-row Evidence URL, Last-verified date, Row_ID, and visible KOL score columns.
8. Attach the exemplar or an example-row schema; state tool preconditions (web access, recalculation).
9. Fix the cross-references, the TOV/TOF spelling, the Norway/ERN error, the duplicate geographies, and the cut-off Appendix A.3 row.
10. Add GDPR data-minimisation and neutral (non-promotional) asset-overlay wording.
