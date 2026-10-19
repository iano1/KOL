# PROMPT — KOL & Treatment-Centre Mapping and Tiering (asset/disease-agnostic)

**Template ID:** `PROMPT_KOL_Centre_Map_v2`
**Purpose:** Produce a Medical Affairs–grade Excel workbook that maps, tiers, and source-verifies the treatment centres and key opinion leaders (KOLs) for a given asset and its target disease across a defined geography — identical in structure, rigour, and conventions to the GBS / tanruprubart reference build.
**Invocation:** *"Run `PROMPT_KOL_Centre_Map_v2` for [ASSET] in [DISEASE], geography [LIST]."*

---

## 0. HOW TO INVOKE

Minimum input:

* **Asset:** generic + brand + originator (e.g. *tanruprubart / ANX-005, Annexon*).
* **Disease / indication:** (e.g. *Guillain-Barré syndrome*).
* **Geography:** named countries or a region tag (e.g. *International, EU4+UK + EU-rest + Europe-rest*, or *Add MENA/Gulf*, or *Add LATAM*).

Optional input (use if supplied; otherwise infer and state the assumption)

* **Region tags:**
* EU4+UK — DE, FR, IT, ES, UK
* EU-rest — AT, BE, BG, HR, CY, CZ, DK, EE, FI, GR, HU, IE, LV, LT, LU, MT, NL, PL, PT, RO, SK, SI, SE (edit the country list as needed)
* Europe-rest — AL, AD, AM, AZ, BY, BA, GE, IS, XK, LI, MD, MC, ME, MK, NO, SM, RS, CH, TR, UA, GB, VA (edit the country list as needed)
* MENA — UAE, Saudi Arabia, Kuwait, Qatar, Egypt, Türkiye, Israel (edit as needed)
* APAC-developed — Japan, Australia/NZ, South Korea, Taiwan, Singapore, China
* LATAM — Pan-American body + Brazil, Argentina, Colombia (edit as needed)
* NorthAmerica — USA, Canada, Mexico

* Phase of asset (pre-approval / launching / launched) → shifts emphasis between advisory-board building, treatment-site readiness, and distribution.
* Any prior context the requester already holds (existing maps, congress plans, analogue work) to cross-reference rather than duplicate.
* Desired centre depth (default: enumerate Tiers 1–3; only enumerate Tier 4 if explicitly requested).

If any of the three minimum inputs is missing, ask **once** for it, then proceed.

---

## 1. OBJECTIVE \& DELIVERABLE

Build a single `.xlsx` workbook named:
`[Region]_[DISEASE-abbrev]_KOL_and_treatment_centres_-_UPDATED_[YYYY-MM-DD].xlsx`

containing exactly these seven tabs, in this order:

1. **README** — scope, tier definitions for both Centres and KOLs, critical caveats, how-to-use
2. **Country_Centres** — master flat table: every Tier 1–3 centre, one row each. Also include tier 4 if explicitly requested
3. **KOL_Roster** — named clinicians with role/signal and tier (engagement priority)
4. **Tier_Counts** — include 2 tables: Centre table (centres × country × tier matrix + cross-cut summary) and KOL table (KOLs × country × tier matrix + cross-cut summary)
5. **Infrastructure** — disease-specific diagnostic/acute-management requirements mapped to centre tiers
6. **Sources_Caveats** — authoritative sources (with URLs) + data-quality caveats
7. **Verification_Flags** — QA pass: items needing human review, with verdicts

The workbook is a **field-deployment and evidence-partner tool**, not a literature review. Every centre and KOL must be tied to a *named, verifiable signal*. No fabricated names — ever.

---

## 2. STEP 0 — CALIBRATE THE FIVE SIGNAL PILLARS (do this first, every time)

The entire tiering rests on five reusable "signal pillars." Before mapping anything, identify the **disease-specific instance** of each. These become the five flag columns in `Country_Centres`. State your five choices explicitly at the top of the README. (See **Appendix A** for a per-therapeutic-area lookup to speed up this calibration.)

|Pillar|What it is|GBS reference instance|How to find it for a new disease|
|-|-|-|-|
|**P1 — Guideline-author signal**|The single authoritative *current* clinical guideline for the disease, and its named author cohort. Defines the apex KOL set.|EAN/PNS 2023 GBS guideline (van Doorn et al.; 21 named authors)|Latest society guideline (EAN, ERS, ESC, ASH, NCCN, EULAR, ASCO, ESMO, disease-specific consortium). Use country-level guidelines for regional Centre and KOL tiering. Pull the full author list — these are the must-engage KOLs.|
|**P2 — Publication-authorship signal**|Authorship of major peer-reviewed publications on the disease, **or** on clinical studies of *this* asset or of *other* assets in current/past clinical development for the disease. Captures research-active centres and trial investigators that guideline and registry signals alone miss — including the investigators on *competitor* programmes (trial-ready, displaceable sites with high competitive-intelligence value).|Senior/recurring authorship on the GBS cohort and trial literature (e.g. the *Lancet* GBS reviews, IGOS-1000 / electrodiagnostic papers, the tanruprubart Phase 3 NCT04701164 publication and earlier ANX-005 trial reports)|PubMed / Europe PMC author searches on the disease term + the asset INN + competitor INNs + trial NCT numbers; ClinicalTrials.gov / EU CTR investigator and site lists; conference (incl. late-breaker) author lists. Weight the last \~5 years and senior/corresponding authorship.|
|**P3 — Registry / outcome-study signal**|The flagship international registry, natural-history study, outcome study, or trial network, plus its country coordinators and participating sites. Identifies RWE-ready / trial-ready centres.|IGOS (International GBS Outcome Study), Erasmus MC|The disease's defining cohort/registry (e.g. ENMC, TREAT-NMD, a CIDP/INCbase analogue, a disease-foundation registry, pivotal-trial site networks). Do not restrict search to international registries. Use country-level and regional registries if warranted. Use site author affiliations where the full site list isn't public.|
|**P4 — Reference-network signal**|Formal reference-centre network membership relevant to the therapeutic area. Defines the Tier 2 baseline.|ERN-EURO-NMD Healthcare Provider list|The matching European Reference Network (see Appendix A), plus national analogues for non-EU geographies and disease-foundation "Centers of Excellence".|
|**P5 — Disease-programme signal**|Presence of a dedicated specialty programme/service/clinic for the disease or its category.|Inflammatory-neuropathy programme|A named clinic/unit/service for the disease or its broader class (e.g. a comprehensive haemophilia centre, an ILD unit, a rare-immunology service).|

> **Note on P1 vs P2:** P1 is *guideline* authorship (a small, fixed, authoritative cohort). P2 is the broader *peer-reviewed and trial-publication* footprint, which surfaces research-active centres and asset/competitor trial investigators who may not be guideline authors. Keep them as separate columns — a centre can score on one, both, or neither, and the combination is diagnostic of tier.

**Substitution rule:** If a pillar has no clean equivalent for the disease, substitute the closest authoritative analogue and **document the substitution** in README and Sources\_Caveats. Never silently drop a pillar.

**Non-applicability rule:** Where a pillar genuinely does not apply to a geography (e.g. P3/P4 outside the network's footprint, post-Brexit UK, non-EU countries), set the flag to `N/A` **with the reason in parentheses** (e.g. `N/A (post-Brexit)`, `N/A (non-EU)`, `No (country not in \[registry])`). Tier those geographies on national academic/leadership/publication signal instead, and say so.

---

## 3. TIERING RUBRIC (generalized)

**Centre Tier**

|Centre Tier|Definition|Role in plan|
|-|-|-|
|**Tier 1 — Anchor / reference**|Hosts a **P1 guideline author** *or* a **P3 registry coordinator**, **AND** is a **P4 reference-network HCP** (or its national equivalent), **AND** carries the **P2 publication signal** (disease- or asset-/competitor-trial authorship in the last \~5 years).|First-wave KOL engagement; advisory-board core; RWE and observational lead candidates; the asset's likely evidence partners.|
|**Tier 2 — National referral**|A **P4 HCP** with the relevant **P5 programme**, *or* a **P3 participating site**; university hospital with full critical/specialist care and disease-relevant diagnostics; recurring **P2** authorship on the relevant literature.|Second-wave outreach; regional referral hubs.|
|**Tier 3 — Tertiary capable**|National/regional reference centre or large university hospital with the core treatment infrastructure but typically **not research-active** in the disease.|Launch-phase distribution; HCP education; not advisory core.|
|**Tier 4 — Community / acute presentation**|District/general hospitals where patients first present and receive supportive care, then refer on.|**Enumerate only if explicitly asked to** (too dispersed). Matters for case-finding, patient pathway, and pharmacovigilance — address via national-society channels, not individual outreach unless instructed otherwise. State this explicitly.|

**Calibration note for acute diseases:** If patients typically first present at the *nearest* hospital rather than a reference centre (as in GBS), add the "critical caveat" that Tier 1–2 are academic/treatment-leadership anchors, **not** the primary point of care — and that field deployment must address this dispersion and focus on Tier 3 and 4 as well. Omit this note for chronic/managed diseases where care concentrates at reference centres.



**KOL Tier**

valuate and classify each identified physician (KOL) based on their level of expertise, influence, and contribution to the relevant therapeutic area.

---

### Step 1: Scoring Criteria

Use the table below to assign scores to each KOL across key domains:

|**KOL Score**|**Guideline Authorship**|**Publications**|**Clinical Trials**|**Academic Appointment**|**Society Roles**|**Speaker Engagement**|**Awards**|
|-|-|-|-|-|-|-|-|
|**1 point**|Contributor to local protocols or internal standards; co-author of minor/regional guidelines|≥3 publications (any authorship) in relevant peer-reviewed journals|Participant or site investigator in local/national trials|Specialist physician; working/research group member; completed specialist training|Member of local/regional society|Occasional local/regional speaker|1–2 points depending on calibre of award|
|**3 points**|Lead/co-author of national guidelines; contributor to multicentre consensus statements|≥5 publications in reputable journals (last 7 years)|National PI or investigator in multi-country/regional trials (last 7 years)|Senior physician; associate professor; research group member; retired department head|Committee/board member in national society|Regular invited speaker at national conferences|—|
|**5 points**|Lead/co-author of international or globally influential guidelines|≥5 publications as first/last author (last 7 years)|Lead/site PI in large international or pivotal trials influencing practice/regulation|Department head; research group leader; emeritus professor|Executive leadership in major national/international societies|Keynote/plenary speaker at major international congresses|—|

---

### Step 2: Scoring Rules

* Evaluate each physician independently across all categories
* Assign **1, 3, or 5 points per category** based on best fit
* Use the **highest level fully supported by evidence**
* Do **not infer or assume** achievements without evidence
* If information is limited, assign the **lowest reasonable score (default to 1)**
* Prioritize **recent (e.g., last 5–7 years) and verifiable contributions**

---

### Step 3: Total Score Calculation

* Sum all category scores to generate a **total score per KOL**

---

### Step 4: Tier Assignment

Classify each physician into one of the following tiers:

* **Tier 1 (Top KOLs):** 24–30 points
* **Tier 2 (Established KOLs):** 12–23 points
* **Tier 3 (Emerging/Local KOLs):** 0–11 points

---

### Quality & Consistency Guidance

* Apply scoring consistently across all KOLs/physicians
* Avoid bias toward geography, institution size, or reputation without evidence
* Ensure alignment between evidence, score, and final tier



---

## 4\. TAB-BY-TAB SPECIFICATION

### 4.1 README

Rows (label | value style, navy `#1F3864` section header, white bold):

* Title: *"[Region] [Disease] Treatment Centres and KOLs — Mapping & Tiering"*
* Prepared for: *Medical Affairs workstream — [asset] ([disease])*
* Geographies: list
* Date compiled / Compiled by / method line (*"cross-referenced from public sources"*)
* **Tab guide** (one line per tab)
* **The five signal pillars chosen for this disease** (P1–P5 instances) — this is mandatory and disease-specific
* **Centre Tier definitions** (Section 3, written out in full, including the criteria)
* **KOL Tier definitions** (Section 4, written out in full, including the criteria)
* **Critical caveat(s)** — acute-presentation dispersion if applicable; any geography-specific gaps (post-Brexit, not-in-registry countries, single-HCP countries)
* **KOL gap-fill methodology** (Section 6)
* **HOW TO USE** — field deployment (sort by Tier asc then Country), trial-site shortlist (filter P3 = site/coord AND P2 publications = Yes), verification reminder (re-verify affiliation + trial activity before outreach)

### 4.2 Country_Centres (master flat table — the heart of the build)

Thirteen columns, header fill blue `#2E75B6`, white bold, **freeze top row, autofilter on, SUBTOTAL-compatible**:

|#|Column|Content|
|-|-|-|
|A|Country||
|B|Tier|1 / 2 / 3 / 4 (integer, sortable)|
|C|City||
|D|Centre / Hospital|full official name|
|E|Unit / Department|the relevant service/centre within the hospital|
|F|KOL / Lead clinician(s)|named, with titles; `Not publicly identified — verify locally` where none traceable|
|G|Role / Affiliation signal|the *specific* evidence that earns the tier (guideline author, registry coordinator, antibody-reference lab, society presidency, named endorsement, etc.)|
|H|**P1 flag** (e.g. "[Guideline] author")|Yes / No / qualified|
|I|**P2 flag** (e.g. "Key publications / trial author (5y)")|Yes — disease / Yes — [asset] trial / Yes — competitor trial / No / qualified. Name the key paper(s) or NCT inline or in Notes.|
|J|**P3 flag** (e.g. "[Registry] coord/site")|Coordinator / Site / No / qualified|
|K|**P4 flag** (e.g. "[Network] HCP")|Yes / No / N/A (reason) / Affiliated partner|
|L|**P5 flag** (e.g. "[Disease] programme")|Yes / Partial / No|
|M|Notes|distinguishing facts, asset-specific signals, caveats|

Rename H–L headers to the actual P1–P5 instances chosen in Step 0. One centre = one row. Sort by Country then Tier.

### 4.3 KOL_Roster

Ten columns (freeze top row, autofilter):
`Country | KOL | Primary affiliation | Role/Signal | [P1] guideline author | [P3] registry role | Key publications/datasets ([P2]) | Presentation at international conferences  | Engagement priority | KOL Tier for TOV`

* The **Key publications/datasets** column *is* the P2 pillar at the person level — list the KOL's defining disease papers, plus any authorship/PI role on this asset's or competitors' trials (with NCT where known).
* Include every named clinician who carries a real signal (guideline author, registry coordinator/PI, reference-lab PI, society leader, trial PI, named asset endorser).
* **Presentation at international conferences** should include oral presentations at international conferences or poster presentations if none are oral
* **Engagement priority** ties to tier and intent: `TIER 1 — must-engage`, `TIER 1 — must-engage ([asset]-specific)` for the asset's own endorsers/evidence partners, `TIER 2 — engage`, `TIER 2 — high value`, plus qualifiers (`— verify current lead`, `— national anchor`, `— neurocritical-care voice`, `— competitor-trial PI`, etc.).
* **KOL Tier for TOF** is based on KOL scoring rules
* Include ALL KOLs listed in Country/Centre tab plus additional as needed to reach minimum of 15 KOLs per country across the different tiers. Search hospital directories as needed.

### 4.4 Tier_Counts

* Matrix Centres: `Country | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Total enumerated | Notes`, one row per country, with a per-country interpretive note. TOTAL row at bottom.
* Matrix KOLs: `Country | Tier 1 | Tier 2 | Tier 3 | Total enumerated | Notes`, one row per country, with a per-country interpretive note. TOTAL row at bottom.
* **Cross-cuts** block beneath with section for Centres and section for KOLs: count of centres hosting a P1 guideline author; with P2 disease/trial publication authorship (and, separately, with this-asset or competitor-trial authorship); with a P3 coordinator role; listed as P3 sites; that are P4 HCPs; with an explicit P5 programme. Count of KOLs with guideline publications; with clinical trial publications; with disease publications; with presentations at international meetings

### 4.5 Infrastructure

Five columns: `Domain | Capability | Minimum requirement | Typical Tier 1-2 coverage | Notes / failure modes`.
Organise by domain. Use the GBS domain set as the *template*, re-specified for the disease:

* **Diagnostics** — the disease's confirmatory tests, biomarkers, imaging, reference-lab dependencies, and any documented practice variation.
* **Severity assessment** — validated scores / triage rules that gate escalation.
* **Treatment** — standard-of-care modalities; flag the **single most differentiating piece of infrastructure** (for GBS, plasma exchange — many ICUs lack it); reimbursement pathways.
* **Rehabilitation / chronic management** — as relevant to the disease course.
* **Future-state** — (a) the **asset's own administration capability** and how its profile compares to SoC infrastructure burden; (b) RWE-registry participation capability.

> \*\*Asset overlay (important):\*\* In Future-state, state the asset's commercial-infrastructure argument explicitly — e.g. \*"\[asset]'s single-dose IV profile is infrastructure-light vs multi-day/ multi-session SoC; addressable centre base is potentially broader than the \[differentiating-infrastructure] footprint."\* This is what turns a disease map into an asset map.

### 4.6 Sources\_Caveats

* **Sources table:** `Authoritative source | URL / reference | What it provides`. Every source named, with a CLICKABLE working URL and full citation (DOI/PMID). Include: the P1 guideline; the P2 evidence base (key disease cohort/epidemiology papers per geography **and** the publications/registrations for this asset's and competitors' trials — DOI/PMID + NCT); the P3 registry; the P4 network HCP list; the relevant Orphanet/foundation directories; national network registries; and the **originator's corporate communications** (press releases on filings, endorsements, RWE collaborations).
* **CAVEATS block:** coverage limits (e.g. registry site lists not fully public → site attribution via author affiliations), per-geography gaps (post-Brexit, not-in-registry, single-HCP countries), KOL-movement warnings, gap-fill disclosure, Tier-4 omission rationale, trial-readiness validation checklist, and **asset-specific signal** (who the endorsers / evidence partners are).
* For **regional extensions** (MENA/Gulf, LATAM, etc.), add a **scope note** stating countries added, centre/KOL counts, that data is thinner, and that EU-specific columns read `No / N/A` throughout — with the anchor sources that *do* exist per country.

### 4.7 Verification_Flags (the QA pass that makes this defensible)

Run an adversarial self-check of the highest-stakes claims **before** delivery. Columns:
`# | Sheet | Cell(s) | Item | Finding | Recommended correction | Verdict`

Check, at minimum:

* **Affiliation currency** — has the KOL moved? (KOLs move constantly.) → `OUTDATED`
* **Independence / compliance** — is anyone listed as an independent KOL actually a pharma employee or otherwise conflicted? Flag as **COMPLIANCE-relevant**. → `CONTRADICTED`
* **Provenance** — is any affiliation/claim unsourced or unverifiable? → `PART-UNVERIFIABLE`
* **Wording / counts** — imprecise geography or figures. → `WORDING`
* **Drug premise** — confirm the asset's regulatory/trial status claims are real (filing date, trial NCT + primary-endpoint result, named endorsements).

**Highlight the flagged source cells in red** on their home sheets (fill `#FFC7CE`, font `#9C0006`) — flag, do **not** auto-correct, so the human decides. Close with a note stating what was checked and *passed* (e.g. *"all guideline authors / registry coordinators / network memberships verified; these N flags are the only items requiring your decision"*).

---

## 5\. SOURCING, GAP-FILL \& EVIDENCE STANDARDS

* **Named sources for every claim.** Prefer primary/authoritative sources: society guidelines, registry sites, ERN/network HCP lists, Orphanet, peer-reviewed cohorts (DOI/PMID), originator IR pages. Avoid aggregators.
* **Verify URLs** resolve; cite the access route where relevant.
* **No fabrication.** Where guideline authorship (P1) or registry coordination (P3) doesn't yield a named KOL for a Tier 2/3 centre, gap-fill in this order: (a) **P2 publication signal** — the senior/corresponding author on the centre's disease or asset/competitor-trial papers; (b) reference-network HCP coordinator (Orphanet); (c) national rare-disease/specialty-network coordinator; (d) authorship on a relevant national/consortium database or registry paper. If none is traceable, attempt to identify from Hospital directories or other local sources and write `Not publicly identified — verify locally` if not successful.
* **Tie the map to the asset:** surface the asset's trials, named endorsers, and evidence partners (often the Tier 1 cohort itself) so the workbook reads as an asset deployment and engagement planning tool, not a generic directory.

---

## 6. EXCEL / FORMATTING CONVENTIONS (match the reference build exactly)

* **Font:** Arial throughout.
* **Headers:** section titles navy `#1F3864`, table headers blue `#2E75B6`, white bold text (size 14 for section titles, 12 for table headers).
* **Freeze panes** on the header row of every data tab; **autofilter** on flat tables; formulas (counts) SUBTOTAL-compatible so filtering stays correct.
* **Verification highlights:** red fill `#FFC7CE` + red font `#9C0006` on flagged cells only.
* **Zero formula errors** — recalculate (`recalc.py`) and confirm no `#REF! / #DIV/0! / #VALUE! / #N/A / #NAME?` before delivery; use formulas (e.g. `SUBTOTAL`/`COUNTIFS`) rather than hardcoded counts in Tier\_Counts.
* Column widths sized for readability (signal/notes columns wide, flag columns narrow).

---

## 7. ACCEPTANCE CHECKLIST (self-verify before presenting)

* [ ] Five signal pillars (P1–P5) explicitly chosen and stated in README; substitutions/N-A documented.
* [ ] Every Tier 1 centre satisfies the Tier-1 compound criterion (P1 guideline author *or* P3 registry coordinator **and** P4 network HCP **and** P2 recent publications).
* [ ] P2 column populated from real searches (disease term + asset INN + competitor INNs + NCTs); competitor-trial investigators flagged.
* [ ] No fabricated KOLs; gap-fills follow the (a)/(b)/(c)/(d) order; untraceable cells marked `verify locally`.
* [ ] Make sure tier 4 Centres are included if explicitly asked to do so
* [ ] Every source named with a working and clickable URL / DOI / PMID.p
* [ ] Asset overlay present: endorsers/evidence partners surfaced; Future-state infrastructure argument stated.
* [ ] Tier_Counts cross-cuts reconcile with the flat table (counts via formulas).
* [ ] Verification_Flags pass run; affiliation currency + independence/compliance checked; red highlights applied; nothing auto-corrected.
* [ ] Non-applicable geographies handled with reasoned `N/A`; regional extensions carry a scope note.
* [ ] Zero formula errors; Arial; freeze panes + autofilter; professional header styling; no hidden text- increase cell size if needed.
* [ ] Minimum number of KOLs presented
* [ ] Filename and README "prepared for" line name the actual asset and disease.

---

### Reference exemplar

This template was reverse-engineered from `European\_GBS\_KOL\_and\_treatment\_centres\_-\_UPDATED\_2026-06-03.xlsx`
(tanruprubart / ANX-005, Guillain-Barré syndrome): 200 centres across 35 countries, 76 KOLs. Five pillars =
P1 EAN/PNS 2023 guideline · P2 GBS \& ANX-005/competitor-trial publications · P3 IGOS · P4 ERN-EURO-NMD ·
P5 inflammatory-neuropathy programme. (The reference build folded P2 publications into the Role/Signal and
Notes columns and the README trial-shortlist filter; v1 promotes it to a dedicated column.) Use the workbook as
the gold-standard output to match for structure, rigour, and conventions.

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
|**US**|NORD Rare Disease Centers of Excellence; NIH/NCATS RDCRN; disease-foundation "Centers of Excellence" (e.g. GBS|
|**Switzerland, non-EU EFTA**|National university-hospital reference designations; ERN *affiliated-partner* status where it exists|
|**MENA / Gulf, LATAM, APAC**|National referral-hospital status; regional society leadership; disease-foundation Centers of Excellence; cohort-paper authorship as the primary signal (data is thinner — say so in the scope note)|
|**Any geography, any disease**|Disease-foundation "Center of Excellence" networks are the most transferable cross-border analogue — check whether the relevant foundation maintains one.|

> **Reminder:** For each pillar you instantiate from this appendix, record the *actual* source (guideline citation, registry URL, ERN HCP-list entry, programme page) in **Sources_Caveats** — the appendix is a pointer, not a citation.

