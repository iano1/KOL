# PROMPT — KOL & Treatment-Centre Mapping and Tiering (asset/disease-agnostic)

**Template ID:** `PROMPT_KOL_Centre_Map_v3.2`
**Purpose:** Produce a Medical Affairs–grade Excel workbook that maps, tiers and source-verifies the treatment centres and key opinion leaders (KOLs) for a given asset and its target disease across a defined geography.
**Invocation:** *"Run `PROMPT_KOL_Centre_Map_v3.2` Stage [1/2/3/4] for [ASSET] in [DISEASE], geography [LIST]."*

**Origin:** this template was first reverse-engineered from a Guillain-Barré syndrome KOL and treatment-centre workbook; that workbook is no longer used as an input.

**What changed from v2:** staged runs sized for chat-based assistants with fixed hand-over files; an evidence-access check, with a web-only evidence route as the default and PubMed exports as an optional enhancement; a precedence rule for conflicts (no fabrication beats coverage targets); a defined Tier 1 centre substitution; physician-only KOL roster with an optional wider-experts tab; a corrected 0/1/3/5 KOL scoring scale with recalibrated cut-offs and visible per-category scores; explicit engagement-priority override rules; compliance and data-protection handling; region tags de-duplicated; no claims of checks the environment cannot perform.

**What changed in v3.2:** KOL scoring and tiers now follow the organisation's fair-market-value work instruction (WI-FMV-V1) word for word, with separate tables for physicians, pharmacists and nurses and the instruction's cut-offs; a Profession column; a Global-tier candidate flag for Medical Director review; KOLs are no longer physicians only (anyone who authors a relevant guideline or relevant TA / clinical-trial publications), so the Wider_Experts tab is removed; a stated inclusion threshold for publication-only authors; the BD clinical assessment workbook accepted as a source for asset facts; key congresses taken from the conference-mapping prompt output; PubMed / ClinicalTrials.gov plugins recognised as an EXPORT-mode source; the column is labelled `KOL Tier for TOV/FMV`; the reference-exemplar input is removed. Fee rates are deliberately not included.

**What changed in v3.1:** centre tiers as an ordered decision procedure with explicit AND/OR logic; a centre-level P2 threshold; a one-row-per-site rule; one as-of date and evidence window; scoring levels de-overlapped with defined terms and worked examples; engagement priority relabelled A/B/C; controlled vocabularies for every flag column; Row_IDs, per-row evidence URLs and last-verified dates; a Coverage_Log; fix-versus-flag rule; stop rule when no web access; defined asset-phase effects; extension mode and filename rule for combined regions; neutral asset-overlay wording; sourced infrastructure coverage; KOL Tier use for transfer-of-value governed by compliance policy (superseded in v3.2).

---

## 0. HOW TO INVOKE

### 0.1 Inputs

Minimum input:

* **Asset:** generic/INN + development code + brand (if any) + originator (e.g. *obudanersen / ION582, Ionis*).
* **Disease / indication:** (e.g. *Angelman syndrome*).
* **Geography:** named countries or region tags (0.2).

Optional input (use if supplied; otherwise infer and state the assumption in README → Decisions & substitutions):

* Phase of asset (pre-approval / launching / launched). Default: infer from Asset_Premise and state it. Effect:
  * **Pre-approval** — emphasise Tier 1–2 centres, trial sites (asset and competitor) and KOLs with trial/registry roles; README adds an advisory-board shortlist (KOLs with priority A or overrides O1–O3).
  * **Launching** — as pre-approval, plus Tier 3 centres with the administration infrastructure the asset needs (Infrastructure tab).
  * **Launched** — full Tier 1–3 coverage; Tier 4 only if requested; README adds an HCP-education target list (Tier 2–3 centres).
* Prior context the requester already holds (existing maps, congress plans, analogue work) — cross-reference rather than duplicate.
* Centre depth (default: Tiers 1–3; enumerate Tier 4 only if explicitly requested).
* **PubMed exports** (optional; see 0.4 and 4.5) — improve publication scoring when available. The build does not depend on them. A PubMed or ClinicalTrials.gov plugin/connector in the environment (e.g. the Microsoft 365 Copilot plugins) does the same job: see EXPORT mode in 0.4.
* **BD clinical assessment workbook** for the asset, if one has been run — an accepted source for Asset_Premise (Stage 1). Use its claims with the sources it cites; it does not replace checking those sources.
* **Conference-mapping output** (the key-congress workbook from the conference-mapping prompt) for the disease area — the list of national, regional and international congresses used for Speaker scoring and congress searches. If not supplied, Stage 1 identifies the key congresses itself and says so.
* **KOL inclusion threshold** for publication-only authors (4.2). Default: at least 2 relevant publications in the evidence window, or at least 1 as first or last author.

If any of the three minimum inputs is missing, ask **once** for it, then proceed. If you cannot ask (agent, scheduled or non-interactive mode), stop and report which input is missing — never guess the asset, disease or geography. For optional inputs, use the defaults stated here and log them in README → Decisions & substitutions.

**As-of date and evidence window:** the as-of date is the run date. The default evidence window for time-bound criteria (publications, trial roles, P2 flags) is **the 7 years up to the as-of date** (e.g. as-of 2026-10-19 → 2019-10-19 to 2026-10-19). Where a KOL scoring level in 4.3 states its own period (e.g. *last 3 years* for international speaking, *last 5 years* for pharmacist publications, *current or past* for some trial and society roles, or no period), that level's own wording applies. State the as-of date and default window in README. "Current" means current on the as-of date.

### 0.2 Region tags (each country belongs to exactly one tag)

* **EU4+UK** — DE, FR, IT, ES, GB (UK)
* **EU-rest** — AT, BE, BG, HR, CY, CZ, DK, EE, FI, GR, HU, IE, LV, LT, LU, MT, NL, PL, PT, RO, SK, SI, SE
* **Europe-rest** — AL, AD, AM, AZ, BY, BA, GE, IS, XK, LI, MD, MC, ME, MK, NO, SM, RS, CH, TR, UA, VA (Norway is a full ERN participant: treat P4 as for EU countries)
* **MENA** — AE, SA, KW, QA, EG, IL (Türkiye sits in Europe-rest; include it here only if Europe-rest is not in scope)
* **APAC-developed** — JP, AU, NZ, KR, TW, SG, CN
* **LATAM** — BR, AR, CO (+ Pan-American bodies)
* **NorthAmerica** — US, CA, MX

* **International** — all of the tags above.

If the requester combines tags, de-duplicate countries. The requester may edit any list.

**Filename region rule:** a single tag uses its name (`EU4+UK`); combined tags are joined with `+` in the order listed above (`EU4+UK+EU-rest`); all tags = `International`; a custom country list uses ISO codes joined with `-` (`DE-FR-IT`).

### 0.3 Run mode

* **STAGED (default).** Four runs (Section 2), each producing a hand-over file that is attached to the next run. Use this whenever the geography has more than 3 countries or the environment cannot create files reliably.
* **SINGLE.** Permitted for 3 countries or fewer. Perform the four stages in order within one run, skip the human checkpoint after Stage 1, and record every assumption in README → Decisions & substitutions.

For more than 5 countries, run Stages 2 and 3 in country batches of no more than 2–3 countries per run.

**Chat-based assistants (e.g. Microsoft Copilot with a fixed model):** a single chat turn has a limited research and output budget, so keep each run small enough to finish properly:

* Select the model explicitly; do not use Auto routing, so that every stage runs on the same model.
* Stage 2 (centres): no more than **2 countries per run**.
* Stage 3 (KOLs): **1 country per run** (attach that country's PubMed export if one is available).
* SINGLE mode is not used.
* If the run is approaching its output limit, stop at a complete row, list what remains, and end with `CONTINUE NEEDED — [what remains]`. The requester replies "continue" and the run resumes from that point. Never truncate a table mid-row or summarise remaining rows instead of listing them.
* If the environment cannot create an `.xlsx`, output each hand-over tab as a table with exactly the specified columns.

A typical EU4+UK build is therefore about ten runs: Stage 1 (1), Stage 2 (3), Stage 3 (5), Stage 4 (1).

**EXTENSION mode** (adding countries to an existing workbook): input = the prior final workbook plus the new geography. Run Stage 1 only for what changes (new national guidelines, registries, network equivalents), then Stages 2–3 for the new countries only, then Stage 4 to merge. Keep every existing Row_ID; new rows take the next free IDs. Do not re-tier existing rows unless a verified change is logged in Verification_Flags. Save as a new file with the run date and add a scope note (6.6).

### 0.4 Evidence-access check (first thing in every run)

Before any research, state in a short block what this environment can actually do, by attempting each rather than assuming:

| Capability | Answer |
|-|-|
| Live web search / page retrieval | Yes / No |
| Direct PubMed or Europe PMC queries — via API, plugin or connector (structured author/affiliation results, not search-engine snippets) | Yes / No |
| ClinicalTrials.gov (incl. plugin) and EU CTIS retrieval | Yes / Partial / No |
| Code execution and `.xlsx` file creation | Yes / No |
| Link checking (can test whether URLs resolve) | Yes / No |

Rules that follow from it:

* **Evidence basis.** Each run works in one of two modes, stated in the evidence-access block:
  * **WEB** (default) — no direct PubMed/Europe PMC queries and no exports attached. Publications are found through ordinary web retrieval using the route in 4.5. Do not stop or wait for exports.
  * **EXPORT** — a PubMed export is attached for the country, **or** the access check shows direct PubMed / Europe PMC queries work (API, or a PubMed plugin/connector that returns author order and affiliations). In the second case, run the Stage 1 Search_Pack query for the country yourself (via the plugin, the E-utilities or the Europe PMC REST API, retrieving author lists and affiliations) and treat the result exactly as an attached export. Publication evidence is taken from it.
  Record the mode per KOL in the Evidence basis column (6.3) and per country in README. Stage 1 always produces the optional PubMed search pack so the requester *can* supply exports, but no stage depends on them.
* **No live web search or page retrieval at all** → **stop.** Say that this environment cannot research the task, and do not produce any centre, KOL or asset content from memory.
* **Plugin check.** If a PubMed plugin is present, test it on one known paper from Stage 1 and confirm it returns the full author list in order with affiliations. If it returns titles only, stay in WEB mode and say so.
* **No link checking** → mark every source `Link checked = No`. Never state that links were verified.
* **No code execution** → do not claim to have recalculated the workbook; state `Not recalculated in this environment — open in Excel and recalculate (Ctrl+Alt+F9)`.
* Never claim to have run a tool, script or search that was not actually run.

### 0.5 Precedence when rules conflict

Apply in this order, and record every conflict you resolve in README → Shortfalls & deviations:

1. **No fabrication or unsupported claims** — every name, role, score point and status is traceable to a named source.
2. **Compliance and data-protection rules** (Section 7).
3. **Tiering rules as written** (Section 4), including the defined substitutions.
4. **Coverage targets** — the KOL target per country and centre depth are *search targets, not quotas*. Never fill a shortfall with weaker names; disclose it.
5. **Formatting conventions** (Section 8).

**Fix versus flag:** errors in applying this prompt's rules (a tier that does not meet 4.1, a score that does not match its cited evidence, a wrong formula, a flag value outside its list) are **fixed before delivery** and logged in Coverage_Log notes. Uncertainty about external facts (affiliation currency, unsourced or single-source claims, compliance concerns, asset premise) is **flagged, not changed**, so a human decides.

---

## 1. OBJECTIVE & DELIVERABLE

The final output (Stage 4) is a single `.xlsx` workbook named:
`[Region]_[DISEASE-abbrev]_KOL_and_treatment_centres_-_UPDATED_[YYYY-MM-DD].xlsx`

containing these tabs, in this order:

1. **README** — scope, evidence-access block, pillars, tier definitions for centres and KOLs, decisions and substitutions, shortfalls and deviations, caveats, data-protection note, how to use
2. **Country_Centres** — master flat table: every Tier 1–3 centre, one row each (Tier 4 only if requested)
3. **KOL_Roster** — named KOLs of any profession with signals, profession, per-category scores, KOL Tier for TOV/FMV, Global-tier candidate flag and engagement priority
4. **Tier_Counts** — centre and KOL country × tier matrices plus cross-cuts
5. **Infrastructure** — disease-specific diagnostic/treatment requirements mapped to centre tiers, with the asset overlay
6. **Sources_Caveats** — authoritative sources with URLs and link-check status, plus caveats
7. **Verification_Flags** — QA pass: items needing human review, with verdicts
8. **Coverage_Log** — per country and stage: searches run, sources checked, completeness

The workbook is a **field-deployment and evidence-partner tool**, not a literature review. Every centre and KOL must be tied to a *named, verifiable signal*.

---

## 2. STAGES AND HAND-OVER FILES

Each stage starts with the evidence-access check (0.4) and ends with a hand-over `.xlsx` that includes a `Coverage_Log` tab (6.8). Attach all earlier hand-over files to each later stage. If the environment cannot create files, output each tab as a table with exactly the specified columns so it can be pasted into Excel.

### Stage 1 — Calibrate, confirm premise, prepare searches

Output `..._S1_Calibration.xlsx` with tabs:

* **Calibration** — the five pillars (Section 3), the instance chosen for each, the source, and every substitution or `N/A`, including the Tier 1 substitution decision per country (4.1). Also record:
  * **Relevant TA** — the therapeutic-area scope used for KOL scoring (4.3): the disease plus a stated, bounded set of closely related conditions or modalities (e.g. for Angelman syndrome: Angelman syndrome, *UBE3A*/15q11-13 disorders, and antisense-oligonucleotide trials in neurodevelopmental disorders). Keep it narrow enough that "relevant" means something; record the reasoning.
  * **Key congresses** — from the conference-mapping output if supplied, otherwise identified here: each congress with its level (`International` / `Regional (multi-country)` / `National`), society, and where its programme or abstract book can be found.
* **Asset_Premise** — every factual claim about the asset and its competitors used anywhere later (trial names, NCT/CTIS numbers, phase, enrolment status, results, designations, endorsements, ownership changes). Columns: `Claim | Source 1 | Source 2 | Status`. Status = `Confirmed` (primary source such as a sponsor release or registry, or two independent secondary sources) / `Single-source` / `Unconfirmed`. If a BD clinical assessment workbook is supplied, start from its claims: a claim it supports with a primary source that you can open is `Confirmed`; one it states without a source you can open is `Single-source` until checked. Only `Confirmed` claims may be stated as fact later; others must be worded as reported and flagged. Dose, regimen and regulatory designations are stated only when `Confirmed`. Third-party mirrors of trial registries (used when the official register cannot be read) are `Single-source`: acceptable for `Site` flags, flagged, never for named-investigator claims.
* **Search_Pack** — the exact searches Stages 2–3 will run, in two parts:
  * **Candidate sources** — the specific lists to be searched (named guideline author lists, trial registrations, registry/natural-history studies, ERN/national network lists, programme directories, patient-organisation scientific/medical advisory boards, and the programmes of the key congresses listed in Calibration).
  * **PubMed queries (optional)** — for use only if the requester is able to run them. Ready-to-paste PubMed query strings, **one combined query per in-scope country** (so each Stage 3 run receives one export), each covering: the disease term (MeSH + title/abstract) OR the asset INN and code OR each competitor INN and code OR each relevant NCT number; restricted to that country by affiliation (`[ad]`) and to the evidence window (0.1). If a query is likely to exceed ~1,000 records, split it by year range. Include the export instruction: *PubMed → Save → Selection: All results → Format: PubMed (MEDLINE text)* (this format keeps full author names and affiliations; CSV does not). Name each file `PubMed_[country]_[YYYY-MM-DD].txt`.

**Human checkpoint (STAGED mode):** stop after Stage 1 and ask the requester to confirm the pillars, substitutions and premise before Stage 2.

### Stage 2 — Centres (per country batch)

Input: S1. Output `..._S2_Centres_[countries].xlsx` with tabs `Country_Centres` (Section 6.2 columns), `Sources` (Section 6.6 columns), `Flags` (Section 6.7 columns).

### Stage 3 — KOLs (per country batch)

Input: S1, S2 for the same countries, and the PubMed export for the country if one is available (otherwise WEB mode, 4.5). With an export attached, derive publication counts and first/last/corresponding authorship from the export itself (author order and affiliation fields), and cite PMIDs. Output `..._S3_KOLs_[countries].xlsx` with tabs `KOL_Roster` (6.3), `Sources`, `Flags`.
Every named lead clinician in Country_Centres column G must appear in KOL_Roster. Because the KOL definition (4.2) can produce long lists, a Stage 3 run for one country may need several continuations (0.3): finish each KOL row completely before starting the next.

### Stage 4 — Assemble and QA

Input: all S1–S3 files. Build the final workbook (Section 1): merge and de-duplicate, write README, Tier_Counts, Infrastructure and Sources_Caveats, then run the full adversarial Verification_Flags pass (6.7) across the merged content. Do not re-research or change Stage 2–3 content except to fix a verified error, which must be logged as a flag.

---

## 3. STEP 0 — CALIBRATE THE FIVE SIGNAL PILLARS (Stage 1)

The tiering rests on five reusable signal pillars. Before mapping anything, identify the **disease-specific instance** of each. These become the five flag columns in `Country_Centres`. State the five choices explicitly in README. (See **Appendix A** for a per-therapeutic-area lookup.)

|Pillar|What it is|GBS reference instance|How to find it for a new disease|
|-|-|-|-|
|**P1 — Guideline-author signal**|The single authoritative *current* international clinical guideline or consensus for the disease and its named author cohort, plus national guidelines where they exist.|EAN/PNS 2023 GBS guideline (van Doorn et al.; 21 named authors)|Latest society guideline or consensus (EAN, ERS, ESC, ASH, NCCN, EULAR, ESMO, disease-specific consortium). Pull the **full** author and affiliation list — if only part of it can be retrieved, say so and flag it. Also identify national guidelines (e.g. France PNDS, AWMF, NICE) and their authors/coordinators.|
|**P2 — Publication-authorship signal**|Authorship of peer-reviewed publications on the disease, or on clinical studies of *this* asset or of *other* assets in current/past development for the disease (competitors).|GBS cohort and trial literature; tanruprubart Phase 3 publication; earlier ANX-005 reports|PubMed / Europe PMC searches (or the attached exports) on the disease term + asset INN/code + competitor INNs/codes + NCT numbers; ClinicalTrials.gov / EU CTIS; congress author lists. **Centre-level P2 = Yes** when, within the evidence window, the centre has (a) ≥2 disease, asset or competitor publications with an author affiliated there, at least one with that author first, last or corresponding; **or** (b) is a listed site of an asset or competitor interventional trial. Record which in the P2 type column.|
|**P3 — Registry / outcome-study signal**|The defining registry, natural-history or outcome study, or trial network, with its coordinators and participating sites.|IGOS (International GBS Outcome Study), Erasmus MC|Disease registry/natural-history study (international, national or regional). **Coordinator** = named national coordinator or chief investigator of a registry or natural-history study. **Site** = a participating site of such a study or of an interventional trial in the disease.|
|**P4 — Reference-network signal**|Formal reference-centre network membership for the therapeutic area. Defines the Tier 2 baseline.|ERN-EURO-NMD Healthcare Provider list|The matching ERN (Appendix A), verified against the EC ERN directory; national analogues for non-EU geographies; disease-foundation Centres of Excellence.|
|**P5 — Disease-programme signal**|A dedicated specialty programme/clinic/service for the disease or its class.|Inflammatory-neuropathy programme|A named clinic/unit/service for the disease (`Yes`) or its broader class (`Partial`).|

> **P1 vs P2:** P1 is guideline/consensus authorship, recorded as `Yes — international` or `Yes — national` (diagnostic/laboratory guidelines are recorded under P1 as `Diagnostic only` and do not count as P1 for tiering). P2 is the broader publication and trial footprint. Keep them as separate columns.

**Substitution rule:** if a pillar has no clean equivalent, substitute the closest authoritative analogue and **document the substitution** in README and Sources_Caveats. Never silently drop a pillar.

**Non-applicability rule:** where a pillar does not apply to a geography, set the flag to `N/A (reason)` (e.g. `N/A (post-Brexit)`, `N/A (non-EU)`, `No (country not in [registry])`). Tier those geographies on national leadership/publication signal and say so.

---

## 4. TIERING RUBRIC

### 4.1 Centre tier

**Unit of analysis:** one row = one physical hospital site. Hospital groups (e.g. AP-HP, NHS trusts, multi-campus university hospitals) are split into the sites that host the relevant unit; group-level network membership is copied only to sites that host the member unit. Adult and paediatric hospitals are separate rows.

**Decision procedure** — test each centre in this order and assign the first tier it meets. Terms are defined below the procedure.

1. **Tier 1** = (A) apex signal **AND** (B) P4 (or accepted equivalent) **AND** (C) P2 = Yes.
2. **Tier 2** = **any one of**: (i) P4 **AND** P5 ∈ {Yes, Partial}; (ii) P3 = Coordinator or Site; (iii) P2 = Yes **AND** the centre is a university or tertiary hospital with specialist services for the disease area.
3. **Tier 3** = a national/regional reference centre or university/tertiary hospital with the core diagnostic and treatment infrastructure for the disease (Infrastructure tab, minimum requirements) **AND** none of the Tier 1–2 conditions met.
4. **Tier 4** = district/general hospital of first presentation — **only if explicitly requested**.
5. **Otherwise** — not listed. If a centre with a notable signal fails every test (e.g. P1 author but no infrastructure), list it in Coverage_Log notes with the reason.

|Centre Tier|Role in plan|
|-|-|
|**Tier 1 — Anchor / reference**|First-wave engagement; advisory-board core; RWE and observational lead candidates; likely evidence partners.|
|**Tier 2 — National referral**|Second-wave outreach; regional referral hubs.|
|**Tier 3 — Tertiary capable**|Launch-phase distribution; HCP education.|
|**Tier 4 — Community / acute presentation**|Case-finding, patient pathway, pharmacovigilance — via national-society channels.|

**Apex signal (A)** — in order of preference:

1. Hosts an author of the current **international** guideline/consensus (P1); or
2. Hosts a **P3 coordinator** (national coordinator / chief investigator of a registry or natural-history study).

**Defined substitution** — used only for a country where no in-scope centre meets (1) or (2), decided and recorded in Stage 1:

3. Hosts the author or coordinator of the **national** guideline for the disease; or
4. Hosts the **national lead / coordinating investigator** for an interventional trial of this asset or a competitor in the disease, named in a primary source (registry, sponsor release, publication, hospital release). Participation as a trial *site* alone is a Tier 2 signal, not an apex signal.

Record the route used in Country_Centres column O (`International guideline` / `Registry coordinator` / `National guideline (substitution)` / `National trial lead (substitution)`). **P4 evidence (all assets):** network member lists are often not machine-readable. Accept any of: the network's own list; an ERN or EC document naming the hospital; the hospital's own page stating membership; or the national designation (e.g. CRMR/CCMR, CSUR/XUEC, ZSE, IRCCS). If none can be found, the centre may still be assigned **"Tier 1 — provisional (P4 unverified)"** when it meets (A) and (C); flag it for human checking rather than capping it.

**Countries outside the ERNs (e.g. UK, CH and non-European geographies; note that Norway participates fully in the ERNs and some non-EU countries hold affiliated-partner status):** the P4 national equivalent is a national or regional tertiary/university hospital that hosts a regional specialist service for the disease area (e.g. clinical genetics, tertiary paediatric neurology) **and** either a named disease clinic (P5 = Yes) or a national registry/natural-history study (P3 = Coordinator). Record the basis in column M as `Yes — national equivalent` and give the basis in Notes.

**Registry coordination must be sourced:** a centre counts as a P3 Coordinator only when a source names it (or a named clinician there) as coordinator or chief investigator. A registry whose coordinating centre is not stated makes its participating centres `Site` until confirmed.

**Calibration note for acute diseases:** if patients typically first present at the nearest hospital rather than a reference centre (as in GBS), add the critical caveat that Tiers 1–2 are academic/treatment-leadership anchors, not the primary point of care. Omit for chronic/managed diseases where care concentrates at reference centres.

### 4.2 Who is a KOL

* **Definition.** A KOL is anyone, of any profession, who in the relevant TA (Stage 1 Calibration):
  * authors or contributes to a guideline, consensus statement or position paper (any level, including diagnostic/laboratory guidelines); **or**
  * is an author on relevant TA literature or clinical-trial publications that meets the **inclusion threshold**; **or**
  * holds another verifiable signal: trial investigator role, registry coordinator, P4 network or P5 programme lead, society board role, or patient-organisation scientific/medical board role.
* **Inclusion threshold for publication-only authors** (those with no other signal): at least 2 relevant publications in the evidence window, **or** at least 1 as first or last author (default; the requester may set a different threshold in 0.1). Authors below the threshold are not listed individually; record their number per country in Coverage_Log notes. Guideline authors are always included, whatever their publication count.
* **Profession** is recorded for every KOL as `Physician` / `Pharmacist` / `Nurse` / `Other`. It decides which scoring table applies (4.3). `Other` covers scientists (e.g. PhD geneticists), psychologists, allied health professionals, epidemiologists and anyone else without an FMV table. Record profession only when a source states it (a medical degree, a registration, or a role that requires one); if unclear, use `Other` and flag `PART-UNVERIFIABLE`.
* **Industry employees** (current employees of any pharmaceutical/biotech company) are excluded and listed only in Verification_Flags as `COMPLIANCE`.
* **Inclusion basis:** each roster entry is either `Signal-based` (meets the definition above) or `Directory-sourced` (named from a hospital directory as a centre lead with no relevant signal found). Directory-sourced entries may be listed but **do not count toward the per-country target**.
* **Per-country target:** at least 15 signal-based KOLs per country, of any profession, with no upper limit: list everyone who meets the definition. The target is a search target, not a quota (0.5). If it is not met, give the count achieved and the reason in Tier_Counts notes and README → Shortfalls & deviations. Never list names below the inclusion threshold, or directory names, to reach the number.

### 4.3 KOL scoring and KOL Tier for TOV/FMV

KOL scoring follows the tiering criteria of the organisation's fair-market-value work instruction (**WI-FMV-V1**, Section 4 *Tiering Determination*). The tables below reproduce its criteria; if the work instruction changes, it governs and this section must be updated to match. Fee rates are not part of this prompt.

**General rules (all professions):**

* Use the scoring table for the KOL's profession. Score each category at the **highest level fully supported by cited evidence**. **No evidence = 0** (the work instruction's lowest band starts at 0). Do not infer or assume achievements.
* Each level's own time period applies exactly as worded (e.g. *last 7 years*, *last 3 years*, *current or past*, or no period). Where no period is stated, any date counts.
* *Relevant* and *relevant TA* = within the relevant-TA scope set in Stage 1 Calibration.
* Every non-zero score must be traceable to a source in the Sources column.

#### Physicians — Table 1

Score six categories at **0, 1, 3 or 5**, plus the Award bonus.

|Category|1 point|3 points|5 points|
|-|-|-|-|
|**Guideline / Authorship**|Authorship/contribution to any national guideline, consensus statement or position paper|Authorship/contribution to any European or regional guideline, consensus statement or position paper, **or** editorial board member of a relevant TA journal|Authorship/contribution to a current European or US society guideline, consensus statement or position paper|
|**Publications**|At least 3 publications (any authorship) in a relevant, peer-reviewed journal|At least 5 publications (any authorship) in a relevant, peer-reviewed journal in the last 7 years|At least 5 publications in a relevant, peer-reviewed journal in the last 7 years as first or last author|
|**Clinical trials**|Investigator in any national clinical trial in a relevant TA, current or past|Trial PI in a national trial, **or** investigator in a multi-country clinical trial in a relevant TA, in the last 7 years|Site PI or trial PI in an international clinical trial (any phase) in the last 7 years|
|**Academic appointment**|Physician with specialist expertise in the relevant TA|Senior physician, professor, or specialised working/research group member in the relevant TA, or retired department head|Department head, special unit head, or working/research group head in the relevant TA, or emeritus professor|
|**Societies**|Current or past president or board member of a national medical/scientific society|Current or past board member of a European or other regional relevant TA society|Current or past president or vice president of any European, US or other over-regional society in the relevant TA|
|**Speaker engagement**|Speaker at a national conference (scientific session or symposium)|Advisor at regional advisory board meetings, **or** speaker at regional conferences (incl. industry symposia)|Speaker in a scientific session at international conferences in the relevant TA in the last 3 years|
|**Award**|— |— |**2 points** for a relevant award (flat bonus; 0 if none)|

* **Total** = the six category scores + the Award bonus (by formula).
* **KOL Tier for TOV/FMV** (by formula): **Tier 1 = 24 or more; Tier 2 = 12–23; Tier 3 = 0–11.** (The work instruction states Tier 1 as 24–30 for the six categories; an Award bonus that takes a total above 30 stays Tier 1.)

**Physician definitions:**

* *Regional* (Table 1) = a multi-country region below European level or alongside it (e.g. Nordic, Ibero-American, Mediterranean). A guideline or society for one part of a single country counts as national at most.
* *Publication* = a PubMed- or Europe PMC-indexed peer-reviewed article. Abstracts and posters count only for Speaker engagement. First or last authorship only, as the work instruction states (corresponding authorship does not count for 5 points).
* *Investigator* = named sub-investigator or study-team member; *site PI* = principal investigator at a trial site; *trial PI* = principal/coordinating investigator for the whole trial (national or international). *International* and *multi-country* = sites in more than one country.
* *International / regional / national conference* = the levels assigned in the Stage 1 key-congress list.
* *Advisor at regional advisory board meetings* — score only where a public source states it (e.g. a conflict-of-interest statement in a publication); advisory roles for this asset or competitors also go to Verification_Flags as `COMPLIANCE`.
* Patient-organisation scientific/medical board roles are recorded in Role / Signal but are **not** scored (the work instruction does not list them). Society membership without a board role scores 0.

**Physician worked examples:** Guideline — co-author of a national care protocol (e.g. PNDS) → 1; co-author of a European consensus statement → 3; author of a current European society guideline → 5. Publications — 4 relevant papers ever, any position → 1; 6 in the last 7 years, 2 as last author → 3; 5 in the last 7 years as first or last author → 5. Clinical trials — sub-investigator in a 10-country Phase 3 → 3; site PI in that trial → 5. Academic — consultant leading a named rare-disease clinic → 3 (*senior physician*); head of a paediatric neurology department → 5. Societies — board member of the national child-neurology society → 1; board member of the European Paediatric Neurology Society → 3. Speaker — scientific-session talk at an international congress in 2024 → 5; the same talk in 2021 (outside 3 years) → falls back to the next level the evidence supports.

#### Pharmacists — Table 2

Score five categories at **0, 1 or 3**.

|Category|1 point|3 points|
|-|-|-|
|**Guideline authorship**|Authorship/contribution to regional guidelines|Authorship/contribution to national guidelines|
|**Publications**|At least 3 publications in relevant TA/journals in the last 7 years|At least 5 publications in relevant TA/journals in the last 5 years|
|**Academic appointment**|Specialist with experience in the relevant TA|Consultant pharmacist / senior specialist with experience in the relevant TA|
|**Societies**|Member of a national society in the relevant TA|Board member of a national society in the relevant TA|
|**Speaker engagement**|Speaker at regional meetings|Advisor at national or regional advisory board meetings, or speaker at national or regional conferences (including symposium)|

* **Total** = sum of the five scores (by formula). **KOL Tier for TOV/FMV: Tier 1 = 6–15; Tier 2 = 0–5.**
* *Regional* in Tables 2 and 3 = a region **within** one country (it scores below national).

#### Nurses — Table 3

Score four categories at **0, 1 or 2**.

|Category|1 point|2 points|
|-|-|-|
|**Academic appointment**|Specialist with experience in the relevant TA|Consultant nurse / senior specialist with experience in the relevant TA|
|**Society**|Member of a national society in the relevant TA|Board member of a national society in the relevant TA|
|**Speaker**|Speaker at regional events|Speaker at national events|
|**Professional appointment**|Works in the specialised TA|Works in a specialised department or hospital|

* **Total** = sum of the four scores (by formula). **KOL Tier for TOV/FMV: Tier 1 = 5–8; Tier 2 = 1–4.** A total of 0 has no tier in the work instruction: record `Below Tier 2`.

#### Other professions

The work instruction has no table for other professions (e.g. PhD scientists, psychologists, allied health professionals). Do **not** score them on another profession's table. Leave their score cells blank and set KOL Tier for TOV/FMV to `No FMV table – review`, so a tier or rate decision is made by Medical Affairs. Their signals are still recorded in full.

#### Global tier

The work instruction defines a **Global tier** above Tier 1 (international/global thought leaders; compensation through a rate exception signed off by the Medical Director / Associate Medical Director). It has no scoring criteria, so this prompt **never assigns it**. Instead, set **Global-tier candidate = Yes** for a physician who is KOL Tier 1 **and** scores 5 on Guideline **and** 5 on at least one of Clinical trials, Societies or Speaker engagement; and for any KOL of any profession who is first or last author of the current international P1 guideline. Give the reason in Notes. Everyone else = `No`. The flag is for Medical Director review only.

#### Use of the tier and evidence limits

* The KOL Tier for TOV/FMV is a desk-research **pre-assessment** against WI-FMV-V1 from public sources. The tier used for any contract is confirmed under the work instruction at contracting time; README says so.
* **Evidence gaps:** where a category could not be searched in this environment (e.g. `Congress programmes not accessible`), record it in the Evidence gaps column. A 0 with an evidence gap means *not found because not searchable*, not *absent*.
* **Publications in WEB mode:** score only papers actually found and cited (PMID/DOI) using 4.5. Because web retrieval under-counts, a WEB-mode publication score is a **lower bound**; this is recorded via the Evidence basis column, not by inflating the score.
* **Score distribution:** report the distribution per country and profession in README (min / median / max). The cut-offs are fixed by the work instruction and are not recalibrated here.
* Apply scoring consistently; avoid bias toward geography, institution size or reputation without evidence.

### 4.4 Engagement priority

Default engagement priority follows the KOL Tier for TOV/FMV:

|KOL Tier for TOV/FMV|Physician|Pharmacist|Nurse|Other|
|-|-|-|-|-|
|Tier 1|`A — must-engage`|`A — must-engage`|`A — must-engage`|—|
|Tier 2|`B — engage`|`C — monitor`|`C — monitor`|—|
|Tier 3 / `Below Tier 2`|`C — monitor`|—|`C — monitor`|—|
|`No FMV table – review`|—|—|—|`B — engage` if a guideline author (P1 role ≠ No) or Global-tier candidate; otherwise `C — monitor`|

(Tier 2 is the lowest tier for pharmacists and nurses, so it maps to C.)

Priority labels use letters so they cannot be confused with centre tiers or KOL tiers.

The Global-tier candidate flag does not change priority. Only these overrides are permitted, and each must be stated in the **Override reason** column with its source:

|Override|Trigger|Resulting priority|
|-|-|-|
|**O1 — Asset role**|Named PI, national lead, coordinating investigator, steering-committee member or author on this asset's trials, or a named endorser/evidence partner of the asset|`A — must-engage`|
|**O2 — Competitor-trial lead**|Site PI or national lead on a competitor's interventional trial in the disease|At least `B — engage`|
|**O3 — National anchor**|Author/coordinator of the national guideline, or national coordinator of a P3 registry/natural-history study|At least `B — engage`|
|**O4 — Compliance hold**|An open `COMPLIANCE` flag on the person|`HOLD` (overrides all others)|

The priority cell holds only `A`, `B`, `C` or `HOLD` with its label. The override code goes in the Override column and its reason and source in Override reason. Verification needs (e.g. current affiliation) go in Verification_Flags, not in the priority cell. No other overrides.

### 4.5 Publication evidence without PubMed (WEB mode)

WEB mode applies until a PubMed export, API or plugin is available (0.4). For each KOL, in this order, stopping once the publication score is settled:

1. **Papers already in hand** — guideline/consensus author lists, trial publications for the asset and competitors (from Stage 1 Asset_Premise and P2), registry papers, and papers cited by the centre's own pages.
2. **Targeted web searches** — `"[surname] [initial]" [disease]`, then the other relevant-TA terms from Calibration, then `"[surname]" [asset INN or code]` and `"[surname]" [competitor INN]`. Prefer results on pubmed.ncbi.nlm.nih.gov, europepmc.org, journal sites, or the institution's own publication pages.
3. **Profile pages** — institutional profile, ORCID, or university research portal listing publications.

Rules:

* Cite each counted paper (PMID or DOI) in the Sources column; count only relevant-TA papers within the period the scoring level states (4.3).
* **Speaker evidence:** search the programmes and abstract books of the key congresses in Calibration for each KOL; note conflict-of-interest statements in papers, which may also evidence trial or advisory roles.
* Record author position (first / last / corresponding / other) only where the retrieved page shows it.
* Spend a bounded effort per KOL; when the search order is exhausted, score what was found. Do not estimate counts ("likely >10 papers").
* Engagement priority (4.4) does not depend on publication scores, so asset- and competitor-trial roles still surface the right people in WEB mode.

---

## 5. SOURCING, GAP-FILL & EVIDENCE STANDARDS

* **Named sources for every claim.** Prefer primary/authoritative sources: guidelines, registry sites, ERN/network lists, Orphanet, peer-reviewed papers (DOI/PMID), trial registries, sponsor IR pages, hospital pages. Avoid aggregators; if one is the only source, mark the claim `Single-source`.
* **Candidate search order** (KOLs and centre leads): P1 author lists → trial registrations and named-investigator releases (asset and competitors) → P3 coordinators → P4 network lists and national reference-centre coordinators → P5 programme leads → patient-organisation scientific/medical boards → PubMed/Europe PMC authors meeting the inclusion threshold (4.2) → faculty and abstracts of the key congresses → hospital directories (gap-fill only).
* **Gap-fill for a centre with no named lead:** in this order — (a) P2 senior/corresponding author on the centre's disease or trial papers; (b) reference-network HCP coordinator; (c) national rare-disease/specialty-network coordinator; (d) authorship on a national/consortium registry paper; (e) hospital directory (mark `Directory-sourced`). If none is traceable, write `Not publicly identified — verify locally`.
* **Trial investigators:** registries often do not name site PIs. Where a PI is inferred from a press release, publication or patient-organisation letter, name the source and mark `verify`.
* **Tie the map to the asset:** surface the asset's trials, named investigators, endorsers and evidence partners, and competitor-trial sites, so the workbook reads as an asset deployment tool.

---

## 6. TAB-BY-TAB SPECIFICATION

### 6.1 README

Rows in label | value style, navy `#1F3864` section headers, white bold:

* Title: *"[Region] [Disease] Treatment Centres and KOLs — Mapping & Tiering"*
* Prepared for: *Medical Affairs workstream — [asset] ([disease])*
* Geographies; date compiled; method line (*"cross-referenced from public sources"*); run mode and stages completed
* **Evidence-access block** (0.4) for each stage that contributed
* **Tab guide** (one line per tab)
* **The five signal pillars** (P1–P5 instances) — mandatory, disease-specific
* **Centre tier definitions** (Section 4.1 in full, including the apex signal and substitution rule)
* **KOL definitions** (Sections 4.2–4.4 in full: who is a KOL, inclusion threshold used, relevant-TA scope, the three WI-FMV-V1 scoring tables and cut-offs, the Global-tier candidate rule, engagement defaults and overrides)
* **Key congresses** used for Speaker scoring, with their levels and source (conference-mapping output or Stage 1)
* **Decisions & substitutions** — every pillar substitution, Tier 1 route per country, assumptions made
* **Shortfalls & deviations** — KOL target shortfalls by country, pillars not searchable, any rule conflict resolved under 0.5
* **KOL score distribution** per country and profession (min / median / max); counts of Global-tier candidates and of `No FMV table – review` entries
* **Critical caveats** — acute-presentation dispersion if applicable; geography gaps (post-Brexit, not-in-registry, single-HCP countries)
* **Data protection & purpose** (Section 7)
* **As-of date and evidence window** (0.1); **asset phase** and its effect
* **Advisory-board shortlist** (pre-approval / launching) or **HCP-education target list** (launched) per 0.1
* **KOL Tier for TOV/FMV** statement: tiers follow WI-FMV-V1 tiering criteria; they are a desk-research pre-assessment, confirmed under the work instruction at contracting; Global tier is never assigned, only flagged for Medical Director review (4.3)
* **HOW TO USE** — the workbook is delivered sorted by Country then Tier. For field deployment, filter Tier = 1, then 2. For a trial-site shortlist, filter P3 = Coordinator or Site **and** P2 type contains "Asset trial" or "Competitor trial". Re-verify affiliation, trial activity and disclosures before outreach.

### 6.2 Country_Centres (master flat table)

Eighteen columns; header fill blue `#2E75B6`, white bold; freeze top row; autofilter on. Flag columns use **data-validation dropdown lists** with exactly the values shown; any qualification goes in Notes, never in the flag cell.

|#|Column|Allowed values / content|
|-|-|-|
|A|Row_ID|`C-001`, `C-002`… stable; never renumbered|
|B|Country||
|C|Tier|1 / 2 / 3 / 4 (integer)|
|D|City||
|E|Centre / Hospital|full official name of the physical site|
|F|Unit / Department|the relevant service within the hospital|
|G|KOL / Lead clinician(s)|named, with titles; `Not publicly identified — verify locally` where none traceable|
|H|Role / Affiliation signal|the specific evidence that earns the tier|
|I|**P1 flag** (rename to the actual guideline)|`Yes — international` / `Yes — national` / `Diagnostic only` / `No`|
|J|**P2 flag**|`Yes` / `No`|
|K|P2 type|any of `Disease publications`; `Asset trial`; `Competitor trial` (separated by "; "), with key PMIDs or NCTs in Notes|
|L|**P3 flag** (rename)|`Coordinator` / `Site` / `No` / `N/A`|
|M|**P4 flag** (rename)|`Yes` / `Yes — national equivalent` / `Unverified` / `No` / `N/A`|
|N|**P5 flag** (rename)|`Yes` / `Partial` / `No`|
|O|Tier route / rationale|Tier 1: the apex route (4.1); Tiers 2–3: the qualifying condition (e.g. `T2 (ii) P3 Site`)|
|P|Evidence URL(s)|URLs/PMIDs/NCTs supporting the tier and flags|
|Q|Last verified|date the row's evidence was checked (YYYY-MM-DD)|
|R|Notes|distinguishing facts, asset-specific signals, qualifications, caveats|

One row per physical site (4.1). Deliver sorted by Country then Tier.

### 6.3 KOL_Roster

Freeze top row; autofilter on; dropdown lists on every coded column. Columns:

|#|Column|Allowed values / content|
|-|-|-|
|A|Row_ID|`K-001`, `K-002`… stable; never renumbered|
|B|Country||
|C|KOL|name with title|
|D|Profession|`Physician` / `Pharmacist` / `Nurse` / `Other` (4.2)|
|E|Specialty / discipline|e.g. paediatric neurology, clinical genetics, molecular genetics (PhD)|
|F|Primary affiliation|current, with department; give the Country_Centres Row_ID where the centre is listed|
|G|Role / Signal|the signals that justify inclusion (incl. patient-organisation board roles)|
|H|P1 guideline role|`International` / `National` / `Diagnostic — international` / `Diagnostic — national` / `No`|
|I|P3 registry / trial role|text: role and study name, or `No`|
|J|Asset-trial role|`Yes` / `No`|
|K|Competitor-trial role|`Yes` / `No`|
|L|Key publications/datasets (P2)|defining papers (PMID/DOI) and trial roles (NCT)|
|M|Conference presentations|congress, level and year; oral, else poster; `Not found` or `Not searchable`|
|N|Score — Guideline|Physician 0/1/3/5; Pharmacist 0/1/3; Nurse `n/a`|
|O|Score — Publications|Physician 0/1/3/5; Pharmacist 0/1/3; Nurse `n/a`|
|P|Score — Clinical trials|Physician 0/1/3/5; Pharmacist and Nurse `n/a`|
|Q|Score — Academic appointment|Physician 0/1/3/5; Pharmacist 0/1/3; Nurse 0/1/2|
|R|Score — Societies|Physician 0/1/3/5; Pharmacist 0/1/3; Nurse 0/1/2|
|S|Score — Speaker engagement|Physician 0/1/3/5; Pharmacist 0/1/3; Nurse 0/1/2|
|T|Score — Award bonus|Physician 0/2; Pharmacist and Nurse `n/a`|
|U|Score — Professional appointment|Nurse 0/1/2; Physician and Pharmacist `n/a`|
|V|Total score|`=IF(D2="Other","",SUM(N2:U2))` (text `n/a` is ignored by SUM)|
|W|KOL Tier for TOV/FMV|`=IF(D2="Physician",IF(V2>=24,1,IF(V2>=12,2,3)),IF(D2="Pharmacist",IF(V2>=6,1,2),IF(D2="Nurse",IF(V2>=5,1,IF(V2>=1,2,"Below Tier 2")),"No FMV table – review")))`|
|X|Global-tier candidate|`Yes` / `No` (4.3); reason in Notes|
|Y|Engagement priority|`A — must-engage` / `B — engage` / `C — monitor` / `HOLD`|
|Z|Override|`O1` / `O2` / `O3` / `O4` / blank|
|AA|Override reason|reason and source, or blank|
|AB|Evidence gaps|categories that could not be searched|
|AC|Evidence basis|`WEB` / `EXPORT`|
|AD|Inclusion basis|`Signal-based` / `Directory-sourced`|
|AE|Sources|URLs / PMIDs supporting each non-zero score|
|AF|Last verified|YYYY-MM-DD|
|AG|Notes|Global-candidate reason, profession basis, qualifications|

Score cells for `Other` are left blank. Formulas are shown for row 2; fill them down. Include every lead clinician named in Country_Centres column G.

### 6.4 Tier_Counts

* All cross-cuts count coded columns only (never free text), so every figure is a formula.
* **Centres matrix:** `Country | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Total | Notes`, one row per country, TOTAL row, counts by `COUNTIFS` on Country_Centres.
* **KOLs matrix:** `Country | Tier 1 | Tier 2 | Tier 3 | Below Tier 2 | No FMV table – review | Total | Signal-based | Target met? | Global-tier candidates | Notes`, one row per country, TOTAL row, counts by `COUNTIFS` on KOL_Roster.
* **KOLs by profession:** `Country | Physician | Pharmacist | Nurse | Other | Total`, by `COUNTIFS`.
* Beneath each matrix, a **"Visible rows (respects filters)"** line using `SUBTOTAL(103, …)` on the flat table's Country column, so filtered views can be checked. State in README that the matrices themselves are unfiltered totals.
* **Cross-cuts** (formulas): centres hosting a P1 author (international / national); with P2 disease authorship; with asset-trial and, separately, competitor-trial authorship or roles; with a P3 coordinator; P3 sites; P4 HCPs; explicit P5 programmes; Tier 1 by route. KOLs (from the coded columns): P1 guideline role ≠ No; Asset-trial role = Yes; Competitor-trial role = Yes; physicians with Publications score ≥3; physicians with Speaker score ≥3; Global-tier candidate = Yes; each override O1–O4; Directory-sourced; Evidence basis = WEB.

### 6.5 Infrastructure

Six columns: `Domain | Capability | Minimum requirement | Typical Tier 1–2 coverage | Coverage source | Notes / failure modes`. Every coverage statement needs a source in Coverage source, or reads `Assessment (unsourced)`. Domains, re-specified for the disease:

* **Diagnostics** — confirmatory tests, biomarkers, imaging, reference-lab dependencies, practice variation; any genotype or eligibility testing the asset's label/trials require.
* **Severity / outcome assessment** — validated scales and rater requirements that gate treatment or trial participation.
* **Treatment** — standard-of-care modalities; flag the **single most differentiating piece of infrastructure**; reimbursement pathways.
* **Rehabilitation / chronic management / transition** — as relevant.
* **Future-state** — (a) the asset's administration requirements versus SoC infrastructure; (b) RWE/registry participation capability.

> **Asset overlay (neutral wording):** in Future-state, describe factually what the asset's administration requires (route, setting, monitoring), citing the trial publication, registry entry or — once available — the SmPC/label, and which centre capabilities match those requirements. Do **not** make comparative or promotional claims (e.g. "infrastructure-light", "broader addressable base", superiority over standard of care), particularly for a pre-approval asset.

### 6.6 Sources_Caveats

* **Sources table:** `Authoritative source | URL / reference | What it provides | Link checked`. Every source with a clickable URL and full citation (DOI/PMID/NCT/CTIS). Include the P1 guidelines; the P2 evidence base (key disease papers per geography and asset/competitor trial publications and registrations); the P3 registries/studies; the P4 network lists; relevant Orphanet/foundation directories; the originator's corporate communications; and the Asset_Premise sources. `Link checked` = Yes only if resolved during a run (0.4).
* **Caveats block:** coverage limits; per-geography gaps; evidence-access limits; KOL-movement warnings; gap-fill and directory-sourced disclosure; Tier 4 omission rationale; trial-readiness validation checklist; asset-specific signal (endorsers/evidence partners); any `Single-source` or `Unconfirmed` premise claims.
* For **regional extensions** and **EXTENSION mode** runs (0.3), add a scope note: countries added, counts, thinner data, EU-specific columns reading `No / N/A`, and the anchor sources that do exist per country.

### 6.7 Verification_Flags (adversarial QA pass)

Columns: `# | Sheet | Row_ID | Column | Item | Finding | Recommended correction | Verdict` (reference rows by Row_ID, not cell address, so sorting does not break the link).

Verdicts: `OUTDATED` · `CONTRADICTED` · `COMPLIANCE` · `PART-UNVERIFIABLE` · `WORDING` · `DRUG-PREMISE` · `VERIFIED`.

Check, at minimum:

* **Affiliation currency** — has the person moved, retired or changed role? → `OUTDATED`
* **Compliance** — industry employees found and excluded; people involved in both this asset's and competitors' programmes; advisory/consulting links found in public sources; recommend a transfer-of-value disclosure check (e.g. EFPIA national disclosure, Disclosure UK, Transparence Santé) for every Tier 1 KOL, every Global-tier candidate and every O1/O2 override; advisory-board roles for this asset or competitors found in conflict-of-interest statements → `COMPLIANCE`
* **Provenance** — unsourced or single-source affiliations, roles or trial attributions → `PART-UNVERIFIABLE`
* **Tier logic** — re-run the 4.1 decision procedure and 4.3 scoring on every row (right table for the profession; each level's own time period; `n/a` cells only where the profession's table has no such category; Global-tier candidate rule); fix any misapplication (0.5) and log it in Coverage_Log; flag only where the underlying evidence itself is uncertain
* **Drug premise** — every Asset_Premise claim used is `Confirmed`, or is worded as reported and flagged → `DRUG-PREMISE`
* **Wording / counts** — imprecise geography or figures → `WORDING`

**Highlight the flagged cells in red** on their home sheets (fill `#FFC7CE`, font `#9C0006`). Apply the fix-versus-flag rule (0.5): rule-application errors are fixed before delivery; external-fact uncertainty is flagged, not changed. Close with a note stating what was checked and passed, and how many flags require the requester's decision.

### 6.8 Coverage_Log

One row per country per stage: `Country | Stage | Date | Source types searched | Sources checked (count) | Centres found by tier | Signal-based KOLs found | Completeness | Notes`. Completeness = `Complete` (search order in 5 exhausted), `Partial` (some source types not reachable — say which), or `INCOMPLETE` (run stopped early — say what remains). Never thin out later countries silently: if a run cannot give a country full treatment, mark it `INCOMPLETE` and stop rather than produce lighter rows. Rule-application fixes (0.5) are logged here, as is the number of authors per country below the KOL inclusion threshold (4.2).

---

## 7. COMPLIANCE & DATA PROTECTION

* **Purpose statement** in README: the workbook supports scientific engagement planning for [asset] in [disease]; it records publicly available professional information only.
* Record **only professional, publicly available information**: name, title, specialty, institutional affiliation, professional roles, publications, trial roles, society roles, public speaking. **Do not** record personal contact details (personal email, phone, home address), photographs, or any personal characteristics, and do not infer any.
* Exclude industry employees (4.2); flag conflicts and recommend disclosure checks (6.7).
* Note in README that use of the workbook for outreach must follow the organisation's data-protection and HCP-engagement procedures, and that affiliations must be re-verified before contact.

---

## 8. EXCEL / FORMATTING CONVENTIONS

* **Font:** Arial throughout.
* **Headers:** section titles navy `#1F3864`, table headers blue `#2E75B6`, white bold text (14 pt section titles, 12 pt table headers).
* **Freeze panes** on the header row of every data tab; **autofilter** on flat tables.
* **Formulas, not hard-coded counts,** in Tier_Counts and for KOL Total and Tier.
* **Verification highlights:** red fill `#FFC7CE` + red font `#9C0006` on flagged cells only.
* **Zero formula errors:** if code execution is available, recalculate (e.g. LibreOffice headless) and confirm no `#REF! / #DIV/0! / #VALUE! / #N/A / #NAME?`; otherwise state that recalculation was not possible (0.4).
* Column widths sized for readability (signal/notes columns wide, flag and score columns narrow); wrap text and size rows so no text is hidden.

---

## 9. ACCEPTANCE CHECKLIST (self-verify before presenting each stage)

* [ ] Evidence-access block stated; no claim of a check, search, recalculation or link test that was not actually performed.
* [ ] Five pillars chosen and stated; substitutions and `N/A` documented; full P1 author list retrieved or the gap flagged.
* [ ] Asset_Premise claims classified (BD clinical assessment workbook used if supplied); only `Confirmed` claims stated as fact.
* [ ] As-of date and evidence window stated; asset phase and its effect stated.
* [ ] Every centre tiered by the 4.1 decision procedure, one row per physical site; every Tier 1 centre has its route in column O; P4 evidence stated (network list, document, hospital page or national designation) or the centre marked "Tier 1 — provisional (P4 unverified)" and flagged.
* [ ] P2 populated from real searches (WEB mode route 4.5, or attached PubMed exports); evidence basis recorded per KOL; competitor-trial roles flagged.
* [ ] KOL_Roster covers every profession meeting the 4.2 definition; profession recorded from a source; inclusion threshold stated and applied; industry employees excluded and flagged.
* [ ] Every physician, pharmacist and nurse scored on their own WI-FMV-V1 table, with formula Total and formula KOL Tier for TOV/FMV; `Other` = `No FMV table – review`; Global-tier candidates flagged with reasons, never assigned; evidence gaps recorded.
* [ ] Relevant-TA scope and key-congress list (with levels and source) recorded in Calibration and README.
* [ ] Engagement priority uses A/B/C/HOLD per 4.4; every override has a code, reason and source.
* [ ] Every flag and coded column uses its dropdown values only; qualifications in Notes.
* [ ] Row_IDs, Evidence URL(s) and Last verified populated on Country_Centres and KOL_Roster.
* [ ] Coverage_Log completed for every country and stage; no silent thinning; INCOMPLETE where applicable.
* [ ] Per-country signal-based KOL count reported; shortfalls disclosed, not filled.
* [ ] No fabricated names; gap-fills follow 5; untraceable leads marked `verify locally`; directory-sourced entries labelled.
* [ ] Tier 4 included only if requested.
* [ ] Every source has a URL/DOI/PMID/NCT and a truthful `Link checked` value.
* [ ] Asset overlay present and neutral: evidence partners surfaced; administration requirements cited; no comparative or promotional claims; infrastructure coverage sourced or marked `Assessment (unsourced)`.
* [ ] Tier_Counts by formula, with SUBTOTAL visible-row lines; cross-cuts reconcile with the flat tables.
* [ ] Verification_Flags pass run by Row_ID; compliance and disclosure checks recommended; red highlights applied; rule errors fixed and logged, external uncertainties flagged.
* [ ] README states that KOL tiers follow WI-FMV-V1, are a pre-assessment confirmed at contracting, and that no fee rates are included.
* [ ] Data-protection rules followed; purpose statement present.
* [ ] Arial; freeze panes; autofilter; header styling; no hidden text.
* [ ] Filename and README "Prepared for" line name the actual asset and disease.

---

## APPENDIX A — PER-DISEASE PILLAR LOOKUP (calibration accelerator)

Use this to instantiate Step 0 quickly. The **P4 (Reference-network)** column is authoritative for the EU — it maps to the 24 European Reference Networks (ERNs, established 2017). The **P1 / P3 / P5** columns give the most likely *archetypes* to confirm per disease; treat them as starting points, not facts — always verify the current guideline, registry, and programme for the specific indication. **Verify the live ERN HCP list and current rosters** at the European Commission ERN search tool and each ERN's site before building, as memberships change.

### A.1 — P4: the 24 ERNs by therapeutic area

|ERN|Domain (use when your disease falls here)|
|-|-|
|**ERN EURO-NMD**|Neuromuscular diseases (GBS/CIDP, MG, SMA, DMD/BMD, ALS, myopathies, channelopathies)|
|**ERN-RND**|Rare neurological diseases (ataxias/HSP, Huntington \& choreas, FTD, dystonia/NBIA, leukoencephalopathies, atypical parkinsonism)|
|**ERN EpiCARE**|Rare and complex epilepsies (incl. developmental \& epileptic encephalopathies)|
|**ERN EuroBloodNet**|Rare haematological diseases (haemoglobinopathies, bleeding disorders, rare anaemias, MPN, rare leukaemias)|
|**ERN RITA**|Rare immunodeficiency, autoinflammatory \& autoimmune diseases (PID/IEI, HAE, periodic fever syndromes)|
|**MetabERN**|Hereditary metabolic disorders (LSDs, organic acidaemias, urea-cycle, PKU, mitochondrial)|
|**Endo-ERN**|Rare endocrine conditions (CAH, pituitary, adrenal, growth, DSD, rare diabetes)|
|**ERN GUARD-Heart**|Rare and complex cardiac diseases (cardiomyopathies, channelopathies, ATTR-CM where cardiac-led)|
|**ERN-LUNG**|Rare respiratory diseases (ILD/IPF, PAH where respiratory-led, CF, PCD, sarcoidosis)|
|**ERN VASCERN**|Rare multisystemic vascular diseases (HHT, vascular anomalies, Marfan/HTAD, PAH where vascular-led)|
|**ERN ERKNet**|Rare kidney diseases (aHUS, complement nephropathies, congenital/glomerular)|
|**ERN RARE-LIVER**|Rare hepatological diseases (PBC, PSC, autoimmune hepatitis, rare cholestatic)|
|**ERN ERNICA**|Rare inherited \& congenital digestive / gastrointestinal anomalies|
|**ERN-Skin**|Rare skin disorders (EB, genodermatoses, autoimmune blistering)|
|**ERN-EYE**|Rare eye diseases (inherited retinal dystrophies, etc.)|
|**ERN BOND**|Rare bone disorders (OI, hypophosphatasia, skeletal dysplasias, XLH)|
|**ERN ReCONNET**|Rare connective tissue \& musculoskeletal diseases (SSc, SLE, myositis, EDS)|
|**ERN GENTURIS**|Genetic tumour risk syndromes (Lynch, NF, Li-Fraumeni, hereditary breast/ovarian)|
|**ERN EURACAN**|Rare adult solid cancers (sarcoma, rare GI/GU/endocrine tumours, mesothelioma)|
|**ERN PaedCan**|Paediatric cancer / haemato-oncology|
|**ERN CRANIO**|Craniofacial anomalies \& ENT disorders|
|**ERN ITHACA**|Rare malformation syndromes, intellectual \& neurodevelopmental disorders|
|**ERN eUROGEN**|Rare urogenital diseases \& complex conditions|
|**ERN TransplantChild**|Paediatric transplantation|

### A.2 — Worked archetypes for P1 / P3 / P5 (confirm per indication)

|Therapeutic area (example disease)|P1 — guideline body|P3 — registry / outcome study archetype|P4 — ERN|P5 — programme archetype|
|-|-|-|-|-|
|**Neuromuscular** (GBS, CIDP)|EAN/PNS; AAN|IGOS / INCbase; TREAT-NMD; ENMC cohorts|EURO-NMD|Inflammatory-neuropathy / neuromuscular unit|
|**Neuromuscular** (SMA, DMD)|TREAT-NMD care standards; AAN|SMA/DMD patient registries (TREAT-NMD, iDMD); trial site networks|EURO-NMD|Comprehensive NMD / gene-therapy centre|
|**Rare neuro** (HD, ataxia)|EAN; MDS|Enroll-HD; EUROSCA/ESMI; natural-history cohorts|RND|Movement-disorders / HD clinic|
|**Epilepsy (rare/DEE)**|ILAE|EpiCARE registry; syndrome-specific (e.g. Dravet) registries|EpiCARE|Comprehensive epilepsy centre|
|**Haematology** (haemophilia, SCD)|WFH; ASH; EHA|WBDR / national bleeding-disorder registries; SCD cohorts|EuroBloodNet|Comprehensive haemophilia / haemoglobinopathy centre|
|**Immunology/PID** (HAE, IEI)|WAO/EAACI (HAE); ESID/IUIS|ESID registry; HAE registries (Icatibant/Berinert)|RITA|Immunodeficiency / angioedema centre|
|**Metabolic / LSD** (Fabry, Pompe, NPC)|national/consortium consensus|disease-specific registries (Fabry Registry, Pompe Registry); natural-history (e.g. VTS-270 NPC)|MetabERN|Inborn-errors / lysosomal-storage centre|
|**Endocrine** (CAH, acromegaly)|Endocrine Society; ESE|I-CAH / national endocrine registries|Endo-ERN|Adrenal / pituitary / DSD service|
|**Cardiology** (ATTR-CM, HCM)|ESC; AHA/ACC|THAOS (ATTR); cardiomyopathy registries|GUARD-Heart|Cardiomyopathy / amyloidosis clinic|
|**Respiratory** (IPF/ILD, PAH)|ERS/ATS; ESC/ERS (PAH)|EMPIRE/eurIPFreg; REVEAL/COMPERA (PAH)|LUNG (or VASCERN for PAH)|ILD / pulmonary-hypertension unit|
|**Nephrology** (aHUS, IgAN)|KDIGO|national renal registries; complement-disease cohorts|ERKNet|Complement / glomerular-disease clinic|
|**Hepatology** (PBC, PSC)|EASL; AASLD|Global PBC/PSC consortia|RARE-LIVER|Autoimmune-liver / transplant-hepatology unit|
|**Dermatology** (EB, genodermatoses)|national consensus; EB-CTAP|EB registries (e.g. EB-CARE)|Skin|EB / genodermatosis centre|
|**Bone** (XLH, hypophosphatasia)|national/consortium consensus|XLH / HPP registries|BOND|Metabolic-bone clinic|
|**Connective tissue** (SSc, myositis)|EULAR/ACR|EUSTAR (SSc); myositis registries (EuroMyositis)|ReCONNET|Scleroderma / myositis / vasculitis clinic|
|**Rare oncology** (sarcoma, rare tumours)|ESMO; NCCN|EURACAN/national tumour registries; trial networks|EURACAN (PaedCan if paediatric)|Sarcoma / rare-tumour MDT|

### A.3 — Non-EU / national analogues for P4 (when geography sits outside the ERNs)

|Geography|Use instead of ERN|
|-|-|
|**UK (post-Brexit, from Jan 2021)**|NHS Highly Specialised Services; MRC / national specialist-centre designations; relevant national society leadership (e.g. BPNS for peripheral nerve)|
|**US**|NORD Rare Disease Centers of Excellence; NIH/NCATS RDCRN; disease-foundation "Centers of Excellence" (e.g. GBS/CIDP Foundation International)|
|**Switzerland, non-EU EFTA**|National university-hospital reference designations; ERN *affiliated-partner* status where it exists|
|**MENA / Gulf, LATAM, APAC**|National referral-hospital status; regional society leadership; disease-foundation Centers of Excellence; cohort-paper authorship as the primary signal (data is thinner — say so in the scope note)|
|**Any geography, any disease**|Disease-foundation "Center of Excellence" networks are the most transferable cross-border analogue — check whether the relevant foundation maintains one.|

> **Reminder:** For each pillar you instantiate from this appendix, record the *actual* source (guideline citation, registry URL, ERN HCP-list entry, programme page) in **Sources_Caveats** — the appendix is a pointer, not a citation.

