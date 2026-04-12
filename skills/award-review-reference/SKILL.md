---
name: award-review-reference
description: >
  Reference file for the Award Review skill. Do not trigger directly.
  Contains observation definitions with thresholds, data source field paths,
  exclusion criteria, code tables, sample output, and troubleshooting.
  Loaded on demand by the main award-review skill.
---

## Contents

1. Observation Definitions (12 observations)
2. Contract Pricing Type Codes
3. Competition Extent Codes
4. FFRDC/M&O Exclusion List
5. Nonstandard Office Keywords
6. Sample Output
7. Troubleshooting

---

## 1. Observation Definitions

Each observation is a factual check that documents a structural attribute of the contract award. "Present" means the condition was found in the data. "Not present" means it was not. Neither implies any conclusion.

---

**OBS-01: RECENT_INCORPORATION**
- Source: SAM.gov Entity Management v3, `entityInformation.entityStartDate`
- Comparison field: USASpending award `date_signed`
- Threshold: `date_signed - entityStartDate <= 180 days` (entity formed within 180 days of award)
- Exclusion: None
- Output when present: "Entity formation date [YYYY-MM-DD] is within [N] days of award action date [YYYY-MM-DD]."
- Output when not present: "Entity formation date predates award by more than 180 days."
- Output when data unavailable: "Entity start date not available in SAM.gov record."
- Note: 180-day window is wider than post-award audit standards because COs evaluate vendors well before the award action date.

---

**OBS-02: POST_AWARD_REGISTRATION**
- Source: SAM.gov Entity Management v3, `entityRegistration.registrationDate`
- Comparison field: USASpending award `date_signed`
- Threshold: `registrationDate > date_signed`
- Exclusion: Skip if `date_signed < 2003-01-01`. SAM.gov (and its predecessor CCR) did not require universal registration until FAR Case 2003-021, effective January 1, 2003. Pre-2003 awards cannot be meaningfully compared to SAM registration dates.
- Output when present: "SAM registration date [YYYY-MM-DD] is [N] days after award action date [YYYY-MM-DD]."
- Output when excluded: "Award predates SAM/CCR mandatory registration (pre-2003). Not evaluated."
- Note: Uses `registrationDate` (original registration), NOT `activationDate` (most recent renewal). `activationDate` always postdates old awards and would produce false positives on every legacy contract.

---

**OBS-03: NO_SUBAWARD_REPORTING**
- Source: USASpending `/api/v2/subawards/` endpoint
- Threshold: Zero FFATA subaward records AND `total_obligation > $5,000,000`
- Exclusion: Skip if any of:
  - PSC code starts with AN, AR, AZ, or AJ (R&D services; see Section 4)
  - Recipient name matches known FFRDC/M&O operator (see Section 4)
  - Awarding department code is 8900 (DOE) AND recipient matches DOE M&O pattern
  - Award description contains "management and operating," "M&O contract," or "GOCO"
- Output when present: "Zero FFATA subaward records on file for an award with $[X] in obligations."
- Output when excluded: "FFRDC/M&O contract excluded from subaward evaluation."
- Note: FFATA subaward reporting has a 90-day lag. Very recent awards may show zero subawards even when subs exist. Check award action date.

---

**OBS-04: NO_PRIOR_AWARDS**
- Source: USASpending `/api/v2/search/spending_by_award/` with recipient filter
- Threshold: Zero prior federal contract awards for this recipient AND award is sole-source AND `total_obligation > $10,000,000`
- Sole-source codes: B (Not Available for Competition), C (Not Competed), E (Follow On), G (Not Competed under SAP), NDO (Not Competed - Other)
- Output when present: "No prior federal contract awards found for [recipient] before [date]. Award is sole-source with obligation above $10M."
- Output when not present (has history): "Recipient has [N] prior federal contract awards totaling $[X]."
- Note: USASpending recipient search uses UEI. If UEI is missing, this observation cannot be evaluated.

---

**OBS-05: OBLIGATION_GROWTH**
- Source: SAM.gov Contract Awards API (modification history) or USASpending transaction history
- Threshold: `current_total_obligation / base_award_obligation >= 3.0` (300% growth)
- Exclusion: Skip if fewer than 2 modification records exist
- Output when present: "Current obligation $[X] is [N]% of initial obligation $[Y] across [Z] modifications."
- Output when not present: "Obligation growth within normal range ([N]% of base)."
- Note: Base award obligation is mod 0 or the earliest recorded transaction. Some multi-year IDIQ task orders legitimately grow beyond 300% through planned option exercises.

---

**OBS-06: SHARED_OFFICERS**
- Source: USASpending award detail `executive_details.officers[]` cross-referenced against other entities
- Threshold: Any officer name appearing as an officer of a different entity (different UEI)
- Exclusion: Officers at entities with an established corporate parent-subsidiary relationship (same CAGE code family) are excluded
- Output when present: "Officer [name] also listed as officer of [entity name] (UEI: [X])."
- Output when unable to evaluate: "Officer cross-reference requires multi-entity dataset. Not evaluated for this single-award review."
- Note: This observation requires either a pre-built officer database or a session where multiple entities have been reviewed. For single-award reviews, this observation will typically return "Not evaluated." The skill should note this limitation rather than silently omitting the check.

---

**OBS-07: SINGLE_OFFER**
- Source: USASpending award detail `latest_transaction_contract_data.number_of_offers_received` and `latest_transaction_contract_data.extent_competed`
- Threshold: `number_of_offers_received == 1` AND `extent_competed` is "A" (Full and Open) or "D" (Full and Open after Exclusion of Sources)
- Exclusion: Does not apply to sole-source awards (they have zero offers by definition)
- Output when present: "1 offer received on a full-and-open competition (extent_competed: [code])."
- Output when not present: "[N] offers received."
- Note: Single-offer competitions are common in specialized or classified work. The observation documents the fact; no inference is implied.

---

**OBS-08: HIGH_VALUE_COST_TYPE**
- Source: USASpending award detail `latest_transaction_contract_data.type_of_contract_pricing`
- Threshold: Contract pricing type in [J, T, U, V, S, Y] AND `total_obligation > $50,000,000`
- Type mapping: J=T&M, T=LH, U=CPFF, V=CPAF, S=Cost No Fee, Y=CPIF (see Section 2)
- Output when present: "Contract type [description] ([code]) with obligation of $[X]."
- Output when not present: "Contract type [description] ([code]). No cost-type observation at this obligation level."
- Note: T&M/LH and cost-reimbursement contracts shift performance and cost variance to the government under FAR 16.3/16.6. This is a structural attribute, not a deficiency.

---

**OBS-09: SUBAWARD_CONCENTRATION**
- Source: USASpending `/api/v2/subawards/` endpoint, sum of `amount` fields
- Threshold: `total_subaward_amount / total_obligation > 0.50` (subawards exceed 50% of prime)
- Exclusion: Skip if zero subawards on file
- Output when present: "Subaward total $[X] represents [N]% of prime obligation $[Y]."
- Output when not present: "Subaward total $[X] represents [N]% of prime obligation. Within typical range."
- Note: High subaward ratios are common in systems integration contracts where the prime manages multiple specialist subcontractors. The observation documents the ratio.

---

**OBS-10: SHARED_ADDRESS**
- Source: SAM.gov Entity Management v3, search by physical address components
- Threshold: 3 or more distinct UEIs registered at the same physical address
- Exclusion: Filter out mailing-address-only matches. Filter out entities within the same corporate family (same top-level CAGE or established parent UEI relationship)
- Output when present: "[N] SAM-registered entities share the physical address [address]. Entities: [list up to 5 names with UEIs]."
- Output when not present: "No other SAM-registered entities found at the same physical address."
- Note: Shared addresses are common in commercial office buildings, business incubators, and registered agent offices. The observation documents the co-location.

---

**OBS-11: NONSTANDARD_OFFICE**
- Source: USASpending award detail, awarding office name field
- Threshold: Office name contains any keyword from the nonstandard list (see Section 5)
- Exclusion: None
- Output when present: "Awarding office name '[name]' contains the term '[keyword]'."
- Output when not present: "Awarding office '[name]' does not contain nonstandard keywords."
- Output when missing: "No awarding office name on file."
- Note: Many legitimate offices contain terms like "joint" or "special." The observation documents the presence of the keyword; operational context determines significance.

---

**OBS-12: SHARED_IDENTIFIERS**
- Source: SAM.gov Entity Management v3, search by CAGE code and UEI prefix
- Threshold (CAGE): 2 or more distinct UEIs sharing the same CAGE code
- Threshold (UEI prefix): 2 or more distinct UEIs sharing the first 8 characters (best-effort; SAM.gov free-text search may not reliably match partial UEIs)
- Exclusion: Entities within the same corporate family
- Output when present (CAGE): "[N] entities share CAGE code [code]: [list names]."
- Output when present (UEI): "[N] entities share UEI prefix [prefix]*: [list names]."
- Output when not present: "No shared CAGE codes or UEI prefixes identified."
- Note: CAGE codes are assigned by DLA and are intended to be unique per entity. Sharing may indicate entity restructuring, acquisition, or data quality issues.

---

## 2. Contract Pricing Type Codes

| Code | Description | Category |
|------|-------------|----------|
| A | Fixed Price Redetermination | FFP family |
| B | Fixed Price Level of Effort | FFP family |
| J | Firm Fixed Price | FFP family |
| K | Fixed Price with EPA | FFP family |
| L | Fixed Price Incentive Firm | FFP family |
| M | Fixed Price Award Fee | FFP family |
| R | Cost Plus Award Fee | Cost family |
| S | Cost No Fee | Cost family |
| T | Cost Sharing | Cost family |
| U | Cost Plus Fixed Fee | Cost family |
| V | Cost Plus Incentive Fee | Cost family |
| Y | Time and Materials | T&M/LH |
| Z | Labor Hours | T&M/LH |

Codes relevant to OBS-08: J (T&M per USASpending mapping), T (LH), U (CPFF), V (CPAF), S (Cost No Fee), Y (CPIF).

Note: USASpending maps contract pricing types differently than FAR. Verify the actual `type_of_contract_pricing` code in the award detail against this table.

---

## 3. Competition Extent Codes

| Code | Description | Category |
|------|-------------|----------|
| A | Full and Open Competition | Competed |
| B | Not Available for Competition | Not competed |
| C | Not Competed | Not competed |
| CDO | Competed Under SAP | Competed |
| D | Full and Open after Exclusion of Sources | Competed |
| E | Follow On to Competed Action | Not competed |
| F | Competed under SAP | Competed |
| G | Not Competed under SAP | Not competed |
| NDO | Not Competed - Other | Not competed |

Full and open (relevant to OBS-07): A and D only.

Sole-source (relevant to OBS-04): B, C, E, G, NDO.

---

## 4. FFRDC/M&O Exclusion List

**PSC code prefixes indicating R&D services (exclude from OBS-03):**
AN (R&D - Medical), AR (R&D - Defense), AZ (R&D - Other), AJ (R&D Management/Support Services)

**Known FFRDC/M&O operators (partial list for pattern matching, case-insensitive):**
Sandia, Los Alamos, Lawrence Livermore, Lawrence Berkeley, Oak Ridge, Argonne, Brookhaven, Pacific Northwest, Idaho National, Fermi National, SLAC, National Renewable Energy, Savannah River, Battelle, UT-Battelle, MITRE, Aerospace Corp, IDA, RAND, Lincoln Laboratory, Jet Propulsion Laboratory, Applied Physics Laboratory, Leidos Biomedical, Oak Ridge Associated Universities, Bechtel National, National Technology & Engineering Solutions, KBC Energy Solutions, Hanford

**DOE department code:** 8900 (Department of Energy), 8901 (NNSA)

**Rationale:** FFRDC and M&O contracts operate under unique statutory authority (e.g., Atomic Energy Act). The prime contractor IS the facility operator. Zero FFATA subawards is a structural feature, not an informational gap.

---

## 5. Nonstandard Office Keywords

Keywords checked against the awarding office name field (case-insensitive):

| Keyword | Rationale |
|---------|-----------|
| task force | Temporary organizational unit |
| special | Ad hoc designation |
| temporary | Non-permanent office |
| emergency | Emergency procurement authority |
| joint | Multi-agency construct |
| provisional | Interim authority |
| ad hoc | Non-standard organizational unit |
| expeditionary | Field operations |
| rapid | Accelerated procurement |
| crisis | Emergency operations |

Many legitimate offices include these terms. The observation is informational.

---

## 6. Sample Output

```
AWARD REVIEW
============
PIID: W912BV22P0112
Date of Review: 2026-04-11
Analyst: [CO name or "Automated"]

AWARD SUMMARY
Recipient:          ACME FEDERAL SERVICES LLC
UEI:                ABC123DEF456
Awarding Agency:    Department of the Army
Awarding Office:    W912BV USACE DISTRICT TULSA
Total Obligation:   $12,450,000
Date Signed:        2024-03-15
Period of Perf:     2024-04-01 to 2027-03-31
Contract Type:      Firm Fixed Price (J)
Competition:        Full and Open (A)
Offers Received:    3
NAICS:              541330
PSC:                C219

ENTITY PROFILE
Legal Name:         ACME FEDERAL SERVICES LLC
UEI:                ABC123DEF456
CAGE:               7X2Y9
SAM Status:         Active
Registration Date:  2019-06-15
Activation Date:    2024-01-10
Expiration Date:    2025-01-10
Entity Start Date:  2018-11-01
Physical Address:   123 Main St, Tulsa, OK 74103
Entity Structure:   2L - LLC
Business Types:     23 - Minority Owned, QF - SDVOSB

OBSERVATIONS
 1. RECENT_INCORPORATION:    Not present
    Entity formation date predates award by more than 180 days.

 2. POST_AWARD_REGISTRATION: Not present
    SAM registration date 2019-06-15 predates award action date 2024-03-15.

 3. NO_SUBAWARD_REPORTING:   Present
    Zero FFATA subaward records on file for an award with $12,450,000
    in obligations.
    Source: USASpending subawards endpoint

 4. NO_PRIOR_AWARDS:         Not present
    Recipient has 8 prior federal contract awards totaling $34,200,000.

 5. OBLIGATION_GROWTH:       Not present
    Obligation growth within normal range (105% of base).

 6. SHARED_OFFICERS:         Not evaluated
    Officer cross-reference requires multi-entity dataset.

 7. SINGLE_OFFER:            Not present
    3 offers received.

 8. HIGH_VALUE_COST_TYPE:    Not present
    Contract type Firm Fixed Price (J). No cost-type observation
    at this obligation level.

 9. SUBAWARD_CONCENTRATION:  Not present
    No subaward data on file for ratio calculation.

10. SHARED_ADDRESS:          Not present
    No other SAM-registered entities found at the same physical address.

11. NONSTANDARD_OFFICE:      Not present
    Awarding office 'W912BV USACE DISTRICT TULSA' does not contain
    nonstandard keywords.

12. SHARED_IDENTIFIERS:      Not present
    No shared CAGE codes or UEI prefixes identified.

OBSERVATIONS SUMMARY: 1 of 12 present, 1 not evaluated, 10 not present.

SOURCE LINKS
USASpending: https://www.usaspending.gov/award/CONT_AWD_W912BV22P0112_9700_-NONE-_-NONE-
SAM.gov: https://sam.gov/entity/ABC123DEF456

LIMITATIONS
- FFATA subaward data may lag up to 90 days from reporting date
- Officer cross-reference (OBS-06) requires multi-entity review session
- UEI prefix matching (OBS-12) is best-effort via SAM.gov free-text search
- This review uses publicly available data only and does not include
  classified, FOUO, or contractor-proprietary information

DISCLAIMER
This review documents factual attributes of a federal contract award
using publicly available data from USASpending.gov and SAM.gov. Observations
are structural data points, not assessments or conclusions. The reviewing
officer is responsible for interpreting observations in context and
determining their significance to the specific procurement action.
```

---

## 7. Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| PIID not found in USASpending | Data lag (USASpending updates ~30 days after FPDS), or PIID is an IDV parent | Try SAM.gov Contract Awards API with same PIID. Try searching with IDV award type codes (IDV_A, IDV_B, etc.) |
| Entity start date unreasonably old (e.g., 1800s) | SAM.gov data quality issue; some entities have placeholder dates | Note as "Data quality: entity start date appears to be a placeholder" and skip OBS-01 |
| SAM registration date pre-2003 | Expected for legacy contracts. SAM/CCR did not exist before ~2003 | OBS-02 is excluded automatically for awards with `date_signed < 2003-01-01` |
| Subaward endpoint returns 0 but contract clearly has subs | FFATA reporting has a 90-day lag from award action. Very recent awards may not yet have subaward data | Note the lag and award action date in the OBS-03 output |
| SHARED_ADDRESS returns 50+ entities at a single address | Large commercial office building or registered agent address | Note as "Commercial multi-tenant location" and list only the first 5 entities |
| SHARED_OFFICERS returns "Not evaluated" | Single-award review without pre-built officer database | Expected behavior. Document as a limitation. Run multiple reviews in a session to build cross-reference data |
| Modification history unavailable in USASpending | Some older contracts lack transaction-level data | Fall back to SAM.gov Contract Awards API which provides modification-level records |
| SAM.gov API returns 429 (rate limited) | Exceeded API key daily quota (1,000/day personal, 10,000/day system) | Pause and resume. Note which observations were completed and which remain |
| UEI prefix search returns unrelated entities | SAM.gov free-text search matches broadly | Filter results to only entities with matching first 8 UEI characters. Note that this check is best-effort |
| Multiple awards match the same PIID | PIID reuse across agencies or IDV parents | Present all matches to the user and ask which award to review |
| Award detail returns null for competition fields | Award type does not require competition reporting (e.g., grants, cooperative agreements) | Skip OBS-04 and OBS-07. Note as "Competition data not applicable to this award type" |
| Entity not found in SAM.gov | UEI may be incorrect, entity may have let registration lapse, or entity was never registered | Note as "Entity not found in SAM.gov" and skip entity-dependent observations (OBS-01, 02, 10, 12) |

---

*MIT © James Jenrette / 1102tools. Source: github.com/1102tools/federal-contracting-skills*
