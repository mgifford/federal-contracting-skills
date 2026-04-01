# Repository Overview: 1102tools Claude Skills

This repository provides free Claude Skills for **federal contracting professionals and government contractors**. The skills let Claude interact with real government APIs to answer contracting questions, build IGCEs, draft SOWs/PWS documents, and perform market research — all without leaving Claude.

Website: [1102tools.com](https://1102tools.com)

---

## What Are Claude Skills?

Claude Skills are instruction files (`.md`) uploaded to Claude via **Customize > Skills**. When installed, Claude reads the instructions and knows how to call external APIs on demand. No code deployment is required — just upload the file and ask a question naturally.

---

## API Data Source Skills

These skills connect Claude to live government data APIs.

| Skill | API Key Required | What It Provides |
|-------|-----------------|-----------------|
| **USASpending API** | None | Federal contract and award data — PIIDs, vendor awards, transaction histories, agency spending |
| **USASpending Reference** | None | Filter tables, composite workflows, bulk download, vendor deduplication |
| **GSA CALC+ Ceiling Rates** | None | Awarded not-to-exceed hourly rates from GSA MAS contracts (230K+ records) |
| **GSA CALC+ Reference** | None | Aggregation schemas, IGCE benchmarking, price reasonableness workflows |
| **BLS OEWS Wages** | BLS key | Market wage data covering ~830 occupations across 530+ metro areas |
| **BLS OEWS Reference** | None | Query recipes, SOC code lookup, IGCE rate derivation |
| **GSA Per Diem Rates** | api.data.gov | Federal travel per diem (lodging + M&IE) for all CONUS locations |
| **GSA Per Diem Reference** | None | Travel cost recipes, common rates table, IGCE travel formula |
| **Federal Register API** | None | All Federal Register documents since 1994 — proposed rules, final rules, notices, executive orders |
| **Federal Register Reference** | None | Composite workflows, FAR case history, regulatory timeline |
| **eCFR Lookup** | None | Full current CFR text, updated daily — FAR/DFARS clauses, version comparison back to 2017 |
| **eCFR Reference** | None | Title 48 chapter map, common FAR sections, composite workflows |
| **Regulations.gov** | api.data.gov | Federal rulemaking dockets, proposed rules, public comments, docket histories |
| **Regulations.gov Reference** | None | Comment tracker, FAR case history, regulatory monitor |

> Most skills come in pairs: a main skill that calls the API and a **Reference** companion that provides richer workflows and lookup tables. Install both for the best results.

---

## Orchestration Skills

These skills combine data sources and structured logic to produce contract-ready documents.

| Skill | What It Produces |
|-------|-----------------|
| **SOW/PWS Builder** | Structured scope decision tree → contract-file-ready Statement of Work or Performance Work Statement with staffing table |
| **IGCE Builder: FFP** | Firm-fixed-price Independent Government Cost Estimate with layered wrap rate model (fringe, overhead, G&A, profit) |
| **IGCE Builder: LH/T&M** | Labor Hour and Time & Materials IGCEs with burden multiplier pricing |
| **IGCE Builder: Cost-Reimbursement** | CPFF, CPAF, and CPIF IGCEs with fee structure analysis and statutory fee caps |
| **Market Research Builder** | FAR Part 10 market research report built from USASpending data |
| **Grants Builder** | Federal grant budgets aligned to 2 CFR 200 / SF-424A |

Orchestration skills reuse API keys already stored in other installed skills — no additional keys needed.

---

## API Keys

Only two free keys are needed to unlock every skill:

| Key | Source | Limits |
|-----|--------|--------|
| **BLS key** | [data.bls.gov/registrationEngine](https://data.bls.gov/registrationEngine/) | 500 queries/day |
| **api.data.gov key** | [api.data.gov/signup](https://api.data.gov/signup/) | 1,000 requests/hour |

The api.data.gov key covers both GSA Per Diem and Regulations.gov. Tell Claude to remember your keys and it will use them automatically in future conversations.

---

## Installation

1. Download and unzip the desired skill folder.
2. In Claude: **Customize > Skills > + > Create skill > Upload a skill** — drag in each `SKILL.md` file individually.
3. For skills that have a Reference companion, install both files.
4. Ask a question naturally — Claude reads the instructions and makes the API call.

---

## License

MIT
