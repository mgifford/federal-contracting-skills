# Repository Overview: 1102tools Claude Skills

This repository provides free Claude Skills for **federal contracting professionals and government contractors**. The skills let Claude interact with real government APIs to answer contracting questions, build IGCEs, draft SOW/PWS documents, and perform market research — all without leaving Claude.

Website: [1102tools.com](https://1102tools.com)

---

## What Are Claude Skills?

Claude Skills are instruction files (`.md`) uploaded to Claude via **Customize > Skills**. When installed, Claude reads the instructions and knows how to call external APIs on demand. No code deployment is required — just upload the file and ask a question naturally. Most skills come in pairs: a **main** skill that calls the API and a **Reference** companion that provides richer workflows and lookup tables. Install both for the best results.

---

## What You Can Do With These Skills

### Spending & Contract Research (USASpending API)

The USASpending skill connects to all federal award data sourced from FPDS. No API key required.

**You can ask Claude to:**
- Look up any contract by PIID and get its description, value, vendor, agency, period of performance, PSC/NAICS codes, and competition status
- Retrieve the full modification history (every transaction, every obligation change, every description update)
- Search for contracts by keyword, agency, NAICS code, PSC code, dollar range, pricing type (FFP/T&M/LH), set-aside type, or competition status
- Find the top vendors receiving awards in a market segment or for a given agency
- Track spending trends over time by fiscal year (government-wide or agency-specific)
- List all task orders under a specific IDIQ/GWAC vehicle
- Recover original base award descriptions that PRISM/internal systems have overwritten
- Pull bulk CSV data for offline analysis
- Look up loan award data (Loan Value, Subsidy Cost)
- Analyze spending by object class, federal account, or program activity (File C data)

**Example questions:**
> "What did FDA award under PIID 75F40123C00042 and what were its modifications?"
> "Show me the top 10 vendors for IT professional services at HHS in FY2024."
> "How much did DoD spend on small business set-asides in NAICS 541512 over the last 3 years?"

---

### Labor Rate Benchmarking (GSA CALC+ Ceiling Rates)

The CALC+ skill accesses 230,000+ awarded hourly ceiling rates from GSA MAS contracts. No API key required. Data refreshes nightly.

**You can ask Claude to:**
- Look up ceiling rates for any labor category (e.g., "Senior Software Developer," "Program Manager") by keyword or exact title
- Filter rates by education level, years of experience, business size (small vs. other), security clearance, and work site
- Get full statistical distributions: min, max, average, median, P10–P90, standard deviation
- Pull all the labor categories and rates on file for a specific vendor (vendor rate card)
- Check whether a proposed rate is reasonable: Claude will compute its z-score, IQR position, and delta from market average
- Build multi-category IGCE benchmarks (e.g., rates for 6 labor categories in a single call)
- Analyze the rate landscape for a specific GSA Special Item Number (SIN)

**Key caveat:** CALC+ rates are **ceiling rates** (maximum NTE), not actuals. Order-level prices should be lower per FAR 8.405-2(d). These are pre-loaded contract rates, not geographic.

**Example questions:**
> "What are the ceiling rates for a Systems Analyst with a BA and 5–10 years of experience?"
> "Is $175/hr for a Senior Cybersecurity Analyst on a small business GSA schedule reasonable?"
> "Pull the full rate card for Leidos on GSA MAS."

---

### Market Wage Research (BLS OEWS)

The BLS OEWS skill accesses employer-survey wage data covering ~830 occupations across every U.S. state and 530+ metro areas. Requires a free BLS API key (500 queries/day).

**You can ask Claude to:**
- Get the annual mean, median, 10th, 25th, 75th, and 90th percentile wages for any SOC occupation nationally, by state, or by metro area
- Compare wages for the same occupation across multiple cities (e.g., DC vs. San Antonio vs. Baltimore)
- Compare wages across multiple occupations in one location
- Look up industry-specific wages (e.g., what Systems Analysts earn specifically in federal government vs. all industries)
- Derive fully burdened hourly IGCE rates from BLS base wages (applies a configurable burden multiplier of 1.5x–2.5x depending on contractor overhead profile)
- Cross-validate BLS-derived burdened rates against GSA CALC+ ceiling rates

**Key distinction vs. CALC+:** BLS OEWS gives you what employers **pay** in base wages (no fringe/overhead/profit). CALC+ gives you what GSA contractors **charge** (fully burdened). Use BLS to build your IGCE from the ground up; use CALC+ to validate it.

**Example questions:**
> "What does a Software Developer (SOC 15-1252) earn in San Antonio, TX?"
> "Compare IT Manager wages across DC, Seattle, and Chicago."
> "What burdened hourly rate should I use for a Systems Analyst in Bethesda at a mid-range contractor?"

---

### Travel Cost Estimation (GSA Per Diem Rates)

The Per Diem skill accesses GSA's official federal per diem rates for every CONUS location. Requires a free api.data.gov key (1,000 req/hr).

**You can ask Claude to:**
- Look up lodging and M&IE rates for any city, state, or ZIP code by fiscal year
- Get monthly lodging breakdowns for locations with seasonal variation (e.g., DC ranges from $183–$276/night)
- Calculate the total per diem cost for a trip (nightly lodging × nights + first/last day M&IE at 75% per 41 CFR 301-11.101)
- Build a multi-trip travel IGCE (multiple destinations, trip counts, varying months)
- Compare per diem rates across candidate performance locations
- Track year-over-year rate changes across FY2024, FY2025, FY2026

**Key caveats:** Rates are CONUS only (OCONUS uses State Dept rates). These are maximum reimbursement ceilings, not actual hotel prices. Lodging taxes are generally excluded.

**Example questions:**
> "What is the per diem rate for Washington, DC in FY2026?"
> "Estimate the travel costs for 4 trips of 3 nights each to San Francisco and Chicago."
> "Build a travel IGCE for a 5-year contract with quarterly site visits to Atlanta and Austin."

---

### Regulatory Text Lookup (eCFR API)

The eCFR skill accesses the full, continuously updated text of the Code of Federal Regulations, with point-in-time history back to January 2017. No API key required.

**You can ask Claude to:**
- Read the current text of any FAR or DFARS clause or section by citation (e.g., FAR 52.212-4, FAR 15.305, DFARS 252.227-7014)
- Look up any CFR definition from FAR 2.101
- List all sections within a FAR Part with their headings
- Find all FAR sections that were substantively amended since a given date
- Compare the text of a section at two different points in time (diff view)
- Search all of Title 48 (FAR + all agency supplements) for a term or concept
- Browse agency FAR supplements (DFARS, HHSAR, GSAR, VAAR, NFS, DEAR, and 15+ others) all in Title 48
- Check when a specific section was last amended
- Look up CFR corrections

**Key caveat:** eCFR is an editorial compilation — not the official legal edition. For official citations use GPO's govinfo.gov.

**Example questions:**
> "What does FAR 52.212-4 say?"
> "What is the FAR 2.101 definition of 'simplified acquisition threshold'?"
> "Which FAR sections were amended in 2025?"
> "Compare FAR 15.305 today versus two years ago."

---

### Rulemaking & Comment Tracking (Federal Register + Regulations.gov)

Two complementary skills cover the entire federal rulemaking pipeline.

**Federal Register API** (no key): The official journal of the U.S. Government since 1994 — proposed rules, final rules, notices, executive orders.

**Regulations.gov API** (api.data.gov key): The public comment portal — dockets, comment periods, comment text.

**You can ask Claude to:**
- Find all open comment periods for FAR/DFARS/SBA/GSA procurement rules right now
- Pull the complete legislative history of any FAR case (every ANPRM, NPRM, final rule, and correction)
- Track all regulatory actions affecting a specific CFR part (e.g., all changes to FAR Part 12 since 2020)
- Monitor upcoming procurement rules through the public inspection feed (pre-publication)
- Get an agency's full regulatory activity summary for a time period
- Read the full text of a proposed or final rule
- Retrieve public comments on a docket and analyze comment volume
- Build a regulatory timeline for any topic or keyword
- Scan multiple agencies for recent rulemaking activity in parallel

**Example questions:**
> "What FAR rules currently have open comment periods?"
> "Show me the complete history of FAR Case 2023-008."
> "What procurement-related rules are on public inspection today?"
> "How many public comments did FAR-2024-0007 receive?"

---

### SOW / PWS Drafting (SOW/PWS Builder)

A document generation skill that walks through structured scope decisions and produces a contract-file-ready work statement. No API key or external data required.

**Workflows:**
- **Full build:** Walks through 24 scope decision questions (service model, technical scope, scale/volume, contract structure, quality standards) and derives a staffing table, then produces a properly formatted 14-section SOW or PWS document
- **SOO conversion:** Takes an existing Statement of Objectives, identifies gaps, and develops it into a full SOW or PWS
- **Scope reduction:** Given an existing work statement that exceeds budget, presents trade-offs and regenerates the relevant sections

**Outputs:**
- SOW (task-based, prescriptive) or PWS (outcome-based, performance-based acquisition preferred by FAR 37.602)
- 14-section document: Introduction, Definitions, Requirements, Deliverables, CLIN Structure, PoP, Place of Performance, GFP/GFI, Security, Key Personnel, Reporting, QASP Summary, Transition, Constraints
- Staffing handoff table (labor category, SOC code, FTE count, phase, hours/yr) formatted for input to any IGCE Builder skill

**Example prompts:**
> "Write a SOW for an IT help desk contract supporting 3,000 users across 5 sites."
> "Convert this SOO into a PWS."
> "This IGCE came in at $42M. What can we cut to get to $35M?"

---

### IGCE Building (FFP / LH-T&M / Cost-Reimbursement)

Three IGCE Builder skills cover the full spectrum of federal contract pricing types. All three orchestrate BLS OEWS, GSA CALC+, and GSA Per Diem automatically. No additional API keys beyond those already installed.

#### IGCE Builder: Firm-Fixed-Price (FFP)

**Workflow:** Layered wrap rate model — BLS base wage → fringe → overhead → G&A → profit. Each cost pool is a separate, auditable layer.

**You can ask Claude to:**
- Build a complete FFP IGCE with a full workbook: rate derivation, labor summary by period, travel, ODCs, pricing basis narrative
- Decompose a SOW/PWS description directly into labor categories and build the IGCE from the text
- Validate a vendor's proposed FFP rates against CALC+ distribution (z-score, IQR position, delta from market)
- Run low/mid/high scenarios by varying fringe (28–35%), overhead (15–25%), G&A (5–12%), and profit (8–15%)
- Build FFP-by-period or FFP-by-deliverable CLIN structures

**Regulatory basis:** FAR 15.402, FAR 15.404-1(a)/(b), FAR 15.404-4, FAR 16.202.

#### IGCE Builder: Labor Hour & Time-and-Materials (LH/T&M)

**Workflow:** Single burden multiplier model — BLS base wage × multiplier (1.5x–2.5x) = fully burdened hourly rate. T&M variant adds materials estimation at cost.

**You can ask Claude to:**
- Build a complete LH or T&M IGCE with the same workbook structure as FFP
- Decompose a SOW/PWS into labor categories and build automatically
- Add a materials estimate for T&M contracts (software licenses, cloud hosting, supplies, GFE)
- Validate proposed hourly rates against CALC+ market data
- Run multiplier scenarios: lean (1.5x–1.7x), mid-range (1.8x–2.2x), high-overhead/clearance (2.0x–2.5x)

**Regulatory basis:** FAR 15.402, FAR 15.404-1(b), FAR 16.601.

#### IGCE Builder: Cost-Reimbursement (CPFF / CPAF / CPIF)

**Workflow:** Same layered cost pool buildup as FFP, but replaces profit with a negotiated fee structure and adds statutory fee cap enforcement.

**You can ask Claude to:**
- Build CPFF, CPAF, or CPIF IGCEs with full cost pool breakdowns
- Analyze and enforce statutory fee caps: CPFF fixed fee capped at 10% (R&D 15%); CPAF award fee 0–6%; CPIF incentive within negotiated range
- Model BAA (Broad Agency Announcement) cost estimates under FAR 35.016
- Build share ratio structures for CPIF (contractor/government share of cost over-/underruns)
- Validate proposed cost-reimbursement rates against CALC+

**Regulatory basis:** FAR 15.402, FAR 15.404-1(a), FAR 15.404-4, FAR 16.301–16.307, 10 USC 3322(a).

**Example prompts for all three IGCE skills:**
> "Build an FFP IGCE for 2 Software Developers, 1 PM, and 1 InfoSec Analyst in DC for a base + 4 option years."
> "Price this SOW as a Labor Hour contract."
> "Build a CPFF IGCE for a BAA R&D effort: 3 researchers, 1 data analyst, 18 months."
> "Is $185/hr for a Senior Developer reasonable for an LH order?"

---

### Market Research Reports (Market Research Builder)

An orchestration skill that drives the USASpending API to produce a FAR Part 10 market research report as a DOCX file. No API key required.

**What the report covers:**
1. Requirement Summary and Methodology
2. Market Overview — government-wide award volume, dollar trends over 5 years by fiscal year
3. Prior Award Analysis — comparable awards at the agency level with PIID, vendor, value, dates, contract type, and set-aside
4. Small Business Availability — breakdown by set-aside type (8(a), SDVOSB, WOSB, HUBZone, VOSB, SBA total); Rule of Two assessment
5. Contract Type Analysis — FFP vs. T&M vs. IDIQ prevalence in the market
6. Competition Analysis — competed vs. sole source rates
7. Pricing Trends — year-over-year average award values
8. Vendor Landscape — top N vendors by total obligated dollars
9. Findings and Recommendations — data-driven set-aside strategy, contract type, and competition approach
10. Appendix — raw filter objects and data methodology

**You can ask Claude to:**
- Build a full FAR Part 10 report from just a NAICS code and a one-sentence requirement description
- Scope the report to a specific agency or government-wide
- Recommend whether a small business set-aside is justified based on historical vendor data
- Identify the most common contract vehicles and vehicles types in a market segment
- Assess pricing reasonableness based on historical award patterns

**Example prompts:**
> "Build a FAR Part 10 market research report for IT help desk services (NAICS 541512) at FDA."
> "I need market research to justify a SDVOSB set-aside for cybersecurity consulting."
> "What does the vendor landscape look like for NAICS 541690 government-wide?"

---

### Federal Grant Budgets (Grants Builder)

An orchestration skill for financial assistance awards. Produces SF-424A-aligned budgets under the Uniform Guidance (2 CFR 200). Uses BLS OEWS for personnel benchmarking and GSA Per Diem for travel. Requires BLS and api.data.gov keys.

**What the output covers (SF-424A object class categories):**
- **Personnel:** Institutional base salary × percent effort, with BLS OEWS benchmarking by occupation and location
- **Fringe benefits:** Calculated by appointment type (faculty, post-doc, graduate student, hourly); typical institutional rate ranges
- **Travel:** Per Diem-derived lodging + M&IE for conferences, site visits, field work; GSA City Pairs for domestic airfare
- **Equipment:** Items ≥$5,000 with justification per 2 CFR 200.439
- **Supplies and materials:** Direct-cost supplies per 2 CFR 200.453
- **Contractual (subawards):** Subaward budgets with F&A pass-through analysis; MTDC exclusions per 2 CFR 200.414
- **Indirect costs:** NICRA rate × Modified Total Direct Costs (MTDC); or de minimis 15% if no NICRA
- **Participant support costs:** Conference fees, stipends — excluded from MTDC per 2 CFR 200.456

**Workflows:**
- **Full build:** Step-by-step collection of personnel, fringe, travel, equipment, supplies, subawards, indirect costs, and multi-year escalation
- **Narrative-driven:** Provide a research plan or project description; Claude decomposes it into budget categories
- **Budget justification narrative:** Generates the written narrative required for grant applications

**Key distinction from IGCE:** Grant budgets are structured by object class under 2 CFR 200, not by labor category under FAR. R01-style, cooperative agreement, and program grant formats are all supported.

**Example prompts:**
> "Build a 3-year NIH-style research grant budget: PI at 25% effort, 2 post-docs, 1 research coordinator."
> "Here's my project narrative. What should the budget look like?"
> "What is the MTDC for this budget and how do I apply our 52% F&A rate?"

---

## How the Skills Work Together

The skills are designed to chain together. A typical acquisition workflow might look like:

1. **Regulatory check** — Use eCFR to read the relevant FAR clause or competition requirement
2. **Market research** — Use Market Research Builder + USASpending to produce a FAR Part 10 report and set-aside recommendation
3. **Requirements drafting** — Use SOW/PWS Builder to walk through scope decisions and produce the work statement + staffing table
4. **Cost estimation** — Use an IGCE Builder (FFP, LH/T&M, or CR) to build the estimate; it auto-calls BLS, CALC+, and Per Diem
5. **Rate validation** — IGCE Builder validates your rates against GSA CALC+ ceiling rate distribution
6. **Regulatory monitoring** — Use Federal Register + Regulations.gov to track open comment periods or rule changes that might affect the acquisition

---

## API Keys

Only two free keys are needed to unlock every skill:

| Key | Source | Limits | Skills Using It |
|-----|--------|--------|----------------|
| **BLS key** | [data.bls.gov/registrationEngine](https://data.bls.gov/registrationEngine/) | 500 queries/day | BLS OEWS, all IGCE Builders, Grants Builder |
| **api.data.gov key** | [api.data.gov/signup](https://api.data.gov/signup/) | 1,000 requests/hour | GSA Per Diem, Regulations.gov, all IGCE Builders, Grants Builder |

Skills requiring **no key**: USASpending, GSA CALC+, Federal Register, eCFR, SOW/PWS Builder, Market Research Builder.

Tell Claude to remember your keys and it will use them automatically in future conversations.

---

## Installation

1. Download and unzip the desired skill folder.
2. In Claude: **Customize > Skills > + > Create skill > Upload a skill** — drag in each `SKILL.md` file individually.
3. For skills that have a Reference companion, install both files.
4. Ask a question naturally — Claude reads the instructions and makes the API call.

---

## License

MIT
