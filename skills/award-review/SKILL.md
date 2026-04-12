---
name: award-review
description: >
  Post-award contract review combining USASpending and SAM.gov data into a
  structured checklist of 12 factual observations about a federal contract award.
  Trigger on: award review, post-award review, contract review, PIID review,
  award checklist, review this award, check this contract, post-award check,
  contract observations, award check, contract check, review award.
  Requires usaspending-api and sam-gov-api skills installed.
---

## Overview

This skill produces a structured post-award review report for a federal contract award. It documents 12 factual observations about the award's structure, entity registration, competition, and related data patterns. Each observation is either **present** or **not present** based on data from USASpending.gov and SAM.gov.

**What this skill produces:** A review report with award summary, entity profile, and 12 observation checks. The report is for contract file documentation and CO decision support.

**What this skill does NOT do:** It does not score, rank, or classify observations. It does not draw conclusions. It does not use the words "risk," "fraud," "anomaly," or "suspicious." Observations are factual data points. The reviewing officer determines their significance.

**Regulatory context:** Supports periodic contract oversight and file documentation. COs may use this review during pre-award responsibility determinations, post-award administration, or periodic contract reviews under FAR 42.302.

## Dependencies

| Skill | Provides |
|-------|----------|
| `usaspending-api` | Award detail, transaction history, subaward data, recipient search |
| `sam-gov-api` | Entity registration, entity search by address/CAGE/UEI, contract awards by PIID |

This skill contains no API code. It orchestrates the two API skills above.

**API keys required:** SAM.gov API key (configured in sam-gov-api skill).

## Input Collection

Collect all inputs in a single pass.

| Input | Required | Description |
|-------|----------|-------------|
| PIID | Yes | Procurement Instrument Identifier (e.g., W912BV22P0112, 75A50126C00004) |
| Awarding Agency | No | Narrows search if PIID matches multiple awards |
| Solicitation NAICS | No | Provides additional context for entity evaluation |

### PIID Resolution

1. Search USASpending: `POST /api/v2/search/spending_by_award/` with filters:
   - `award_type_codes`: ["A", "B", "C", "D"] (contracts)
   - `keywords`: [PIID]
   - `limit`: 5
2. If zero results, retry with IDV codes: ["IDV_A", "IDV_B", "IDV_B_A", "IDV_B_B", "IDV_B_C", "IDV_C", "IDV_D", "IDV_E"]
3. If zero results on both, try SAM.gov Contract Awards: `search_contract_awards(piid=PIID)`
4. If still zero, report "PIID not found" and stop
5. If multiple matches, present them to the user and ask which award to review
6. Extract `generated_internal_id` (USASpending) or `generated_unique_award_id` for subsequent calls

---

## Step 1: Award Lookup

Fetch full award metadata from USASpending.

```
GET /api/v2/awards/{generated_internal_id}/
```

Extract and store:
- `piid`
- `description`
- `recipient.recipient_name`
- `recipient.uei`
- `total_obligation`
- `base_and_all_options_value`
- `date_signed`
- `period_of_performance.start_date`
- `period_of_performance.end_date`
- `awarding_agency.toptier_agency.name` (agency)
- `awarding_agency.office_agency_name` (office)
- `funding_agency.toptier_agency.name`
- `latest_transaction_contract_data.extent_competed`
- `latest_transaction_contract_data.type_of_contract_pricing`
- `latest_transaction_contract_data.number_of_offers_received`
- `latest_transaction_contract_data.naics`
- `latest_transaction_contract_data.product_or_service_code`
- `executive_details.officers[]` (names and amounts)
- `generated_unique_award_id`
- `type` (award type code)

Also fetch modification history from SAM.gov Contract Awards:
```
lookup_award_by_piid(piid=PIID)
```
From modification records, identify:
- Base award (modification_number "0") `actionObligation`
- Latest modification `totalActionObligation`
- Count of modifications
- `dateSigned` of base award

Build the USASpending URL: `https://www.usaspending.gov/award/{generated_unique_award_id}`

---

## Step 2: Entity Profile

Fetch entity data from SAM.gov using the recipient UEI from Step 1.

```
lookup_entity_by_uei(uei=RECIPIENT_UEI, include_sections="entityRegistration,coreData,assertions")
```

Extract and store:
- `entityRegistration.legalBusinessName`
- `entityRegistration.ueiSAM`
- `entityRegistration.cageCode`
- `entityRegistration.registrationStatus`
- `entityRegistration.registrationDate`
- `entityRegistration.activationDate`
- `entityRegistration.registrationExpirationDate`
- `coreData.entityInformation.entityStartDate`
- `coreData.physicalAddress` (addressLine1, city, stateOrProvinceCode, zipCode)
- `coreData.generalInformation.entityStructureDesc`
- `coreData.generalInformation.profitStructureDesc`
- `assertions.goodsAndServices.primaryNaics`

If entity not found, note "Entity not found in SAM.gov" and skip entity-dependent observations (OBS-01, 02, 10, 12).

Pause 0.5s between SAM.gov calls to respect rate limits.

---

## Steps 3-4: Observations Batch 1 (Entity-Derived)

These use data already collected. No additional API calls.

**OBS-01: RECENT_INCORPORATION**
- Compare SAM `entityStartDate` vs award `date_signed`
- If `date_signed - entityStartDate <= 180 days`: **Present**
- If SAM entity not found or `entityStartDate` missing: **Not evaluated**
- Output: "Entity formation date [date] is within [N] days of award action date [date]."

**OBS-02: POST_AWARD_REGISTRATION**
- Compare SAM `registrationDate` vs award `date_signed`
- If `date_signed < 2003-01-01`: **Not evaluated** (pre-SAM era)
- If `registrationDate > date_signed`: **Present**
- Output: "SAM registration date [date] is [N] days after award action date [date]."

**OBS-04: NO_PRIOR_AWARDS**
- Search USASpending for prior awards to this recipient:
  ```
  POST /api/v2/search/spending_by_award/
  filters: {
    "recipient_search_text": [UEI],
    "time_period": [{"end_date": date_signed}],
    "award_type_codes": ["A","B","C","D"]
  }
  ```
- Determine if sole-source: `extent_competed` in [B, C, E, G, NDO]
- If zero prior awards AND sole-source AND `total_obligation > $10,000,000`: **Present**
- Output: "No prior federal contract awards found for [recipient] before [date]. Award is sole-source above $10M."

**OBS-07: SINGLE_OFFER**
- From Step 1: `number_of_offers_received` and `extent_competed`
- If `number_of_offers_received == 1` AND `extent_competed` in [A, D] (full and open): **Present**
- Output: "1 offer received on a full-and-open competition (extent_competed: [code])."

---

## Steps 5-6: Observations Batch 2 (Award-Derived)

**OBS-05: OBLIGATION_GROWTH**
- From Step 1 modification history: compare base mod obligation to current total
- If `current / base >= 3.0` AND modification count >= 2: **Present**
- Output: "Current obligation $[X] is [N]% of initial obligation $[Y] across [Z] modifications."

**OBS-08: HIGH_VALUE_COST_TYPE**
- From Step 1: `type_of_contract_pricing`
- If code in [J, T, U, V, S, Y] AND `total_obligation > $50,000,000`: **Present**
- See `award-review-reference` Section 2 for code-to-description mapping
- Output: "Contract type [description] ([code]) with obligation of $[X]."

**OBS-03: NO_SUBAWARD_REPORTING**
- Fetch subawards:
  ```
  POST /api/v2/subawards/
  { "award_id": generated_unique_award_id, "limit": 10 }
  ```
- If zero subawards AND `total_obligation > $5,000,000`:
  - Check exclusion list (see `award-review-reference` Section 4):
    - PSC starts with AN, AR, AZ, or AJ: **Excluded**
    - Recipient matches FFRDC/M&O operator name: **Excluded**
    - Agency is DOE (8900) + M&O recipient pattern: **Excluded**
    - Description contains "management and operating," "M&O contract," "GOCO": **Excluded**
  - If not excluded: **Present**
- Output: "Zero FFATA subaward records on file for an award with $[X] in obligations."

**OBS-09: SUBAWARD_CONCENTRATION**
- From same subaward fetch: sum all `amount` fields
- If `total_subaward / total_obligation > 0.50`: **Present**
- Output: "Subaward total $[X] represents [N]% of prime obligation $[Y]."

---

## Steps 7-8: Observations Batch 3 (Cross-Entity)

These require additional SAM.gov API calls. Pause 0.5s between calls.

**OBS-10: SHARED_ADDRESS**
- Search SAM.gov for entities at the same physical address:
  ```
  search_entities(free_text="[street] [city] [state]", registration_status="A", size=10)
  ```
- Filter results to matching address. Exclude same-corporate-family entities (same CAGE prefix)
- If 3+ distinct UEIs at the same address: **Present**
- Output: "[N] SAM-registered entities share the physical address [address]."
- List up to 5 entity names with UEIs

**OBS-06: SHARED_OFFICERS**
- From Step 1 `executive_details.officers[]`, extract officer names
- For each officer name, this observation ideally cross-references against other entities in a dataset
- For a single-award review without prior data: **Not evaluated**
- Output: "Officer cross-reference requires multi-entity dataset. Not evaluated for this single-award review."
- If running multiple reviews in a session, accumulate officer names and cross-reference

**OBS-12: SHARED_IDENTIFIERS**
- Search SAM.gov by CAGE code:
  ```
  search_entities(free_text="[CAGE]", registration_status="A", size=10)
  ```
- Filter results to matching CAGE. Exclude same entity (same UEI)
- If 2+ distinct UEIs share the CAGE: **Present**
- UEI prefix check (first 8 chars): best-effort via SAM.gov free-text search. Document as "best-effort" in output.
- Output: "[N] entities share CAGE code [code]: [list names]."

**OBS-11: NONSTANDARD_OFFICE**
- From Step 1: awarding office name
- Check against keyword list (see `award-review-reference` Section 5)
- If office name contains any keyword (case-insensitive): **Present**
- Output: "Awarding office name '[name]' contains the term '[keyword]'."

---

## Step 9: Assemble Report

Present the report in this order:

### 1. Header
```
AWARD REVIEW
============
PIID: [piid]
Date of Review: [today]
```

### 2. Award Summary
Table with: Recipient, UEI, Awarding Agency, Awarding Office, Total Obligation, Date Signed, Period of Performance, Contract Type (with code), Competition (with code), Offers Received, NAICS, PSC, Description (truncated to 200 chars).

### 3. Entity Profile
Table with: Legal Name, UEI, CAGE, SAM Status, Registration Date, Activation Date, Expiration Date, Entity Start Date, Physical Address, Entity Structure, Business Types.

### 4. Observations
Each observation in this format:
```
[N]. OBSERVATION_NAME:    [Present / Not present / Not evaluated / Excluded]
    Detail: [one factual sentence]
    Source: [API and field]
```

### 5. Observations Summary
"[X] of 12 present, [Y] not evaluated, [Z] not present."

### 6. Source Links
- USASpending: [URL]
- SAM.gov: https://sam.gov/entity/[UEI]

### 7. Limitations
- FFATA subaward data may lag up to 90 days
- Officer cross-reference (OBS-06) requires multi-entity review session
- UEI prefix matching (OBS-12) is best-effort
- Uses publicly available data only

### 8. Disclaimer
"This review documents factual attributes of a federal contract award using publicly available data from USASpending.gov and SAM.gov. Observations are structural data points, not assessments or conclusions."

---

## Language Constraints

Do NOT use these terms anywhere in the output:

| Prohibited | Use instead |
|------------|-------------|
| risk | observation |
| fraud | (omit entirely) |
| anomaly | observation |
| red flag | observation present |
| suspicious | notable |
| shell company | recently formed entity |
| bid manipulation | (omit entirely) |
| scheme | (omit entirely) |
| severity | (omit entirely) |
| CRITICAL / HIGH / MEDIUM | (omit entirely) |
| score | (omit entirely) |
| threat | (omit entirely) |
| warning | (omit entirely) |

## Rate Limit Budget

A single award review uses approximately:
- 3-5 USASpending API calls (award detail, subawards, recipient search)
- 4-8 SAM.gov API calls (entity lookup, address search, CAGE search, awards/mods)
- Total: 7-13 API calls per review
- SAM.gov personal keys: 1,000/day (allows ~75 reviews/day)
- SAM.gov system keys: 10,000/day (allows ~750 reviews/day)

Pause 0.5s between SAM.gov calls. USASpending has no API key requirement.

## Example Prompts

- "Review award PIID 75A50126C00004"
- "Run post-award checks on W912BV22P0112"
- "Check this contract: SPE7M125P2263"
- "Do an award review for PIID 80MSFC20C0034"
- "Post-award checklist for N00024-25-C-1234"
- "Review this award for [agency]: [PIID]"

---

*MIT © James Jenrette / 1102tools. Source: github.com/1102tools/federal-contracting-skills*
