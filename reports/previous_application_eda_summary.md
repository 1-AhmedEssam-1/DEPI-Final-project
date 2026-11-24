# Previous Application EDA — Findings Summary

Analysis covers `previous_application.csv`. No `TARGET` values were loaded, the original CSV was not modified, and applicant-level feature aggregation was not performed.

## Dataset Overview

| Metric | Value |
|---|---|
| Table grain | One row per previous application (`SK_ID_PREV`; unique) |
| Total rows | **1,670,214** |
| Total columns | **37** |
| `SK_ID_PREV` uniqueness | Confirmed 1:1 with rows ✓ |
| Unique applicants (`SK_ID_CURR`) | **338,857** |
| Fully duplicated rows | **0** |
| Downstream linkage | `SK_ID_PREV` → `POS_CASH_balance`, `installments_payments`, `credit_card_balance` |

## Application Portfolio

| Metric | Value |
|---|---|
| Mean previous apps per applicant | **4.93** |
| Median previous apps per applicant | **4** |
| Max previous apps per applicant | **77** |
| Applicants with exactly 1 previous app | **60,458 (17.8%)** |
| Applicants with 2+ previous apps | **278,399 (82.2%)** |
| p95 apps per applicant | 13 |
| p99 apps per applicant | 20 |

- The majority of applicants (82.2%) have multiple previous applications, making prior-application history a rich signal for feature engineering.
- `DAYS_DECISION` spans a multi-year historical window; all values are negative (decisions occurred before the current application), confirming correct date encoding.
- Application volume shows clear weekday (Mon–Fri) and daytime-hour patterns, consistent with branch-based processing.

## Contract Status and Decisions

`NAME_CONTRACT_STATUS` has four distinct values — not a simple approved/refused binary:

| Status | Count | % |
|---|---|---|
| **Approved** | 1,036,781 | 62.07% |
| **Canceled** | 316,319 | 18.94% |
| **Refused** | 290,678 | 17.40% |
| **Unused offer** | 26,436 | 1.58% |

**Consumer loans** have the highest absolute approval counts. Revolving products show different status distributions. Canceled and Unused offer are structurally distinct from Refused and must not be conflated.

Top rejection reasons (Refused only): credit history (`HC`) and exceeding limits (`LIMIT`). `XAP` is a sentinel meaning "not applicable" and appears in non-refused records.

## Financial Analysis

| Column | Missing | Key Observation |
|---|---|---|
| `AMT_APPLICATION` | 0 (0%) | Requested amount; median **71,046**; max 6.9M; right-skewed |
| `AMT_CREDIT` | 1 (0%) | Granted amount; median **80,541**; tracks AMT_APPLICATION closely |
| `AMT_ANNUITY` | 372,235 (22.3%) | Median **11,250**; missing for refused/canceled |
| `AMT_DOWN_PAYMENT` | 895,844 (53.6%) | Structurally absent for no-down-payment loan types; **not** a data error |
| `AMT_GOODS_PRICE` | 385,515 (23.1%) | Median **112,320**; absent for cash loans |
| `CNT_PAYMENT` | 372,230 (22.3%) | Clusters at 12/24/36 (monthly installments); 0 for revolving |

**Credit vs. Application amount:** `AMT_CREDIT` is very close to `AMT_APPLICATION` for most records. Discrepancies may reflect rounding to standard product tiers, insurance bundling, or internal evaluation values for refused cases — not all differences indicate errors.

**`AMT_CREDIT` in refused records:** This field is populated even when the application was refused. These values likely represent the requested or evaluated amount, **not** disbursed funds. Exercise caution when using this field in aggregations.

## Application Behavior

- **Contract type:** Consumer loans dominate, followed by cash loans and revolving loans.
- **Weekday / Hour:** Processing peaks Mon–Fri, daytime hours. Weekend traffic exists but is much lower.
- **Payment type:** `Cash through the bank` dominates approved applications; distribution differs for refused (partly driven by product mix — not necessarily a causal factor).
- **Client type:** Repeater clients (returning customers) constitute the majority of previous applications. Approval rates differ by client type, but this is a descriptive observation.
- **Goods category:** Consumer electronics, furniture, and mobile phones dominate for consumer loans.
- **Channel type:** Branch-originated applications are dominant; self-service channels vary by status.

## Temporal Fields

| Field | Missing | Sentinel 365243 | Key Finding |
|---|---|---|---|
| `DAYS_DECISION` | 0 | 0 | Always negative; spans multi-year window ✓ |
| `DAYS_FIRST_DRAWING` | High | **934,444 (55.9%)** | Open-ended / revolving products dominate sentinels |
| `DAYS_LAST_DUE` | Moderate | **211,221 (12.6%)** | Missing structurally for refused/canceled |
| `DAYS_LAST_DUE_1ST_VERSION` | Moderate | **93,864 (5.6%)** | Differs from `DAYS_LAST_DUE` for rescheduled loans |
| `DAYS_TERMINATION` | Moderate | **225,913 (13.5%)** | Open-ended for revolving products |
| `DAYS_FIRST_DUE` | Moderate | **40,645 (2.4%)** | Negative when loan ended before current app |

**Sentinel 365243** (≈1,000 years) is a known domain convention for "open-ended / no termination date." It must be masked and replaced with a binary flag before any numerical modeling.

A small number of rows show logically inconsistent date ordering (e.g., drawing before decision, last due before first due). These should be flagged for investigation but are a minor fraction of the dataset.

## Data Quality Issues

| Issue | Action |
|---|---|
| Sentinel `365243` in date fields | Mask; create `{field}_is_open_ended` binary flag |
| `XNA` / `XAP` in categorical fields | Treat as a separate category, not null |
| High missingness in date fields | Expected for refused/canceled; retain missingness indicator |
| `AMT_DOWN_PAYMENT` missing for most applications | Structural by loan type; do not impute as zero |
| `AMT_CREDIT` in refused applications | Does not represent disbursed funds; use with caution |
| Impossible date ordering (small count) | Flag for review |
| Extreme monetary values | Do not clip automatically; review context first |

## Candidate Features for Applicant-Level Aggregation

*(Feature engineering belongs to a later stage)*

- Previous application counts by contract status (approved, refused, canceled, unused)
- Refusal rate and cancellation rate across all previous applications
- Recency of most recent previous application (`min(DAYS_DECISION)`)
- Recency of last approval / last refusal
- Median / max `AMT_APPLICATION`, `AMT_CREDIT`
- Median `CNT_PAYMENT` (reflects preferred loan term)
- Indicator for ever having an unused offer
- Number of distinct `NAME_CONTRACT_TYPE` and `NAME_GOODS_CATEGORY` values
- Loan rescheduling indicator: `DAYS_LAST_DUE ≠ DAYS_LAST_DUE_1ST_VERSION`
- Most common rejection reason code
- Dominant channel type and payment type

## Limitations and Open Questions

1. **AMT_CREDIT discrepancies**: Not all differences between `AMT_CREDIT` and `AMT_APPLICATION` are errors — some reflect standard tier rounding or fee bundling. Requires domain-expert clarification.
2. **Sentinel `365243` semantics**: Does it always mean "open-ended" or could it mean "unknown end date"? This affects feature engineering strategy.
3. **Refused application financials**: Are financial fields in refused applications pre-screening evaluations, underwriting estimates, or applicant-stated amounts?
4. **XNA in `NAME_CASH_LOAN_PURPOSE`**: "Purpose not provided" vs. "purpose not applicable" matters for encoding.
5. **Applicant overlap**: Not all `SK_ID_CURR` from `application_train.csv` appear here, and not all here appear there. Overlap should be characterized before merging.
6. **Date lookback window**: Whether records should be filtered by recency (e.g., last 2 years only) depends on modeling assumptions.
7. **Downstream linkage**: `SK_ID_PREV` links to repayment behavior tables — their EDA is deferred to later notebooks.
