# Installments Payments EDA — Findings Summary

Descriptive EDA of `installments_payments.csv`, using Data Understanding, Bureau, Previous Application, and POS/CASH EDA as context. No `TARGET` or current application dataset was loaded. Original input SHA-256 fingerprints are verified unchanged. No applicant features or final historical feature table were created.

## Data, grain, and linkage

- **13,605,401 payment/schedule records × 8 columns**, **997,752 previous contracts**, **339,587 observed applicants**; compact raw Pandas memory **519.0 MiB**. Monetary values retain float64 precision. Schedule-key work is partitioned by contract to limit memory.
- Grain is a recorded entry, not necessarily a settled installment. There are **12,951,918 complete contract/version/installment keys**, **653,483 extra records at these keys**, **0 exact duplicate extras**, and **0 missing-key records**. Contract/applicant mapping conflicts: **0**. Repeated keys remain intact.
- Previous-ID matching is **958,905/997,752 (96.11%)**; unmatched IDs: **38,847**. Applicant-ID disagreements among matched records: **0**.

| Linkage metric | Count | Denominator | pct_of_denominator |
| --- | --- | --- | --- |
| Payment previous IDs matched to previous_application | 958,905 | 997,752 | 96.11 |
| Payment previous IDs unmatched to previous_application | 38,847 | 997,752 | 3.89 |
| Previous application IDs with payment records | 958,905 | 1,670,214 | 57.41 |
| Previous application IDs without payment records | 711,309 | 1,670,214 | 42.59 |
| Matched payment records | 12,354,575 | 13,605,401 | 90.81 |
| Unmatched payment records | 1,250,826 | 13,605,401 | 9.19 |
| Applicant-ID mismatches among matched records | 0 | 12,354,575 | 0.00 |

## Observed payment-entry timing

`days_late = DAYS_ENTRY_PAYMENT - DAYS_INSTALMENT`; missing dates remain unknown. Mean **-8.79 days**, median **-6**, p95 **3**, p99 **18**, p99.9 **80**; minimum **-3189**, maximum **2884**. Signed mean includes early entries; it does not represent the mean among late payments.

| Recorded-entry timing | Records | Denominator | pct_of_denominator |
| --- | --- | --- | --- |
| Early (<0) | 9,309,477 | 13,602,496 | 68.44 |
| On time (=0) | 3,146,350 | 13,602,496 | 23.13 |
| Late (>0) | 1,146,669 | 13,602,496 | 8.43 |
| Timing unknown | 2,905 | 13,605,401 | 0.02 |

Missing payment-entry dates: **2,905**; missing payment amounts: **2,905**. These are not imputed as zero delay/payment. Delay thresholds below overlap and are exploratory, not observed defaults.

| Delay threshold (≥ days) | Unit/event | Count | Denominator | pct_of_denominator |
| --- | --- | --- | --- | --- |
| 30 | Timed records | 43,602 | 13,602,496 | 0.32 |
| 30 | Contracts with ≥1 severe record | 25,720 | 997,752 | 2.58 |
| 30 | Observed applicants with ≥1 severe record | 23,707 | 339,587 | 6.98 |
| 60 | Timed records | 16,352 | 13,602,496 | 0.12 |
| 60 | Contracts with ≥1 severe record | 10,249 | 997,752 | 1.03 |
| 60 | Observed applicants with ≥1 severe record | 9,891 | 339,587 | 2.91 |
| 90 | Timed records | 12,820 | 13,602,496 | 0.09 |
| 90 | Contracts with ≥1 severe record | 8,290 | 997,752 | 0.83 |
| 90 | Observed applicants with ≥1 severe record | 8,060 | 339,587 | 2.37 |

## Amounts and multiple payment entries

- Median scheduled amount **8,884.080**, median recorded payment **8,125.515**; maxima **3,771,487.845 / 3,771,487.845** dataset units. Zero scheduled amounts: **290**; undefined ratios: **3,150**. Extreme amounts/ratios are retained and inspected.
- Ratios require a positive schedule and observed payment. Individual smaller payments are not automatically installment underpayments.

| Individual recorded amount | Records | Denominator | pct_of_denominator |
| --- | --- | --- | --- |
| Unknown / zero or invalid amount | 3,150 | 13,605,401 | 0.02 |
| Recorded less than scheduled | 1,295,493 | 13,602,251 | 9.52 |
| Recorded equal to scheduled | 12,127,604 | 13,602,251 | 89.16 |
| Recorded more than scheduled | 179,154 | 13,602,251 | 1.32 |

- **640,905/12,951,918 keys (4.95%)** have multiple entries; **640,238** have different recorded entry dates. **0** keys have differing scheduled dates; **0** have differing scheduled amounts.
- Summed outcomes require a consistent positive schedule, complete nonnegative recorded amounts, and no exact duplicate members. **12,948,792/12,951,918 keys** qualify; exclusions remain unknown/ambiguous. No payment is summed across schedule versions. Schedule amount is counted once per eligible key; raw repeated schedule rows are not summed as debt.

| Eligible version-specific key outcome | Keys | Denominator | pct_of_eligible_keys |
| --- | --- | --- | --- |
| Recorded sum below schedule | 2,934 | 12,948,792 | 0.02 |
| Recorded sum equals schedule (numerical tolerance) | 12,766,670 | 12,948,792 | 98.59 |
| Recorded sum above schedule | 179,188 | 12,948,792 | 1.38 |

- **640,645 keys** contain smaller individual payments yet their combined recorded amounts cover the schedule, involving **1,292,318 smaller-payment records**. This is a material distinction between record and key-level shortfalls.
- **66 eligible keys** have late payment entries despite recorded cumulative coverage by the scheduled date; later entries alone would overstate observed coverage delay for these keys.
- For eligible keys with complete positive-payment dates, first cumulative coverage is measured chronologically. The timing denominator below includes only keys whose observed sums cover the schedule; incomplete or insufficient keys have no known coverage date. Numerical amount tolerance is **1e-6 dataset units**, for summation precision only. Coverage does not establish final settlement or correct allocation.

| Eligible, covered and date-complete schedule key | Keys | Denominator | pct_of_denominator |
| --- | --- | --- | --- |
| Early observed coverage | 8,705,018 | 12,945,858 | 67.24 |
| On-time observed coverage | 3,138,067 | 12,945,858 | 24.24 |
| Late observed coverage | 1,102,773 | 12,945,858 | 8.52 |

## Schedule versions and contract histories

- Version **0 denotes credit-card schedules**, per the dictionary. **399,399/997,752 contracts** have multiple observed versions. **89,897 contract/installment-number pairs** occur across versions. Among **89,897 pairs with coherent schedules in every version**, **99.97%** have changing scheduled amounts and **0.34%** have changing scheduled dates. Versions are neither automatically errors nor independent debts.
- Observed keys per contract: median **10**, p95 **33**; median observed due-date span **240 days**. Internal installment-number gaps occur in **20,102 contracts**; they can reflect censoring or schedule mechanics, not necessarily missing payments.
- Within-version adjacent observed installment-number pairs: **11,391,402**; a higher number has an earlier scheduled date in **884,846 pairs**, and the same date in **59,060**. These are ordering checks, not proof that omitted numbers represent unpaid installments.
- **884,825 of the earlier-date reversals occur in version 0 (credit cards)**; **21** occur in positive versions and warrant source-calendar review. Card installment numbers are not reliably chronological. Timing calculations and cumulative coverage use dates directly.
- **313,271/997,674 timing-observed contracts (31.40%)** have a late record; **198,139 (19.86%)** have repeated late records. **195,138** have ≥2 distinct version-specific keys with late records, separating entry multiplicity from key recurrence.

| Contract event | Contracts | Eligible contract denominator | pct_of_eligible_contracts |
| --- | --- | --- | --- |
| Contracts without any observed timing | 78 | 997,752 | 0.01 |
| Any late payment record | 313,271 | 997,674 | 31.40 |
| Repeated late records (≥2) | 198,139 | 997,674 | 19.86 |
| ≥2 distinct version-specific keys with late records | 195,138 | 997,674 | 19.56 |
| Any smaller-than-scheduled record | 225,704 | 997,658 | 22.62 |
| Repeated smaller-than-scheduled records (≥2) | 224,222 | 997,658 | 22.47 |
| Any eligible key with insufficient recorded sum | 2,832 | 997,658 | 0.28 |
| Repeated eligible keys with insufficient recorded sums (≥2) | 60 | 997,658 | 0.01 |
| Any late observed-coverage key | 313,156 | 997,625 | 31.39 |

| Severe recorded-delay pattern | Contracts | pct_of_all_contracts |
| --- | --- | --- |
| No ≥90-day record | 989,462 | 99.17 |
| Exactly one ≥90-day record | 6,945 | 0.70 |
| ≥2 severe records, only one distinct schedule key | 106 | 0.01 |
| ≥2 distinct version-specific keys with ≥90-day records | 1,239 | 0.12 |

## Descriptive risk patterns and data quality

- Contracts with no observed late records have median **8 keys**, versus **12** for repeated-late contracts. Longer history raises detection opportunity; the notebook compares counts with within-contract rates and due-history bands.
- Timing/amount cross-tabs use jointly eligible records. Delay differences across versions mix schedule changes and product types; record-level smaller-payment associations can reflect split entries. Severe repeated records can share one key. No association proves causation or predicts default.
- Individual smaller-payment prevalence is **56.95% of jointly eligible late records**, versus **6.78% of early** and **0.37% of on-time** records. This is not the final-installment shortfall rate. Version-0 versus version-1 late-entry rates are **4.52% / 10.49%**, with different product/calendar mechanics.
- Missing dates/amounts, inconsistent schedule keys, zero denominators, extreme amounts/delays, repeated versions, and unmatched previous IDs require review. Raw anomaly counts are preserved below; positive relative days would require application-time availability review.

| Flag | Records | Affected contracts | pct_of_all_records |
| --- | --- | --- | --- |
| Negative scheduled amount | 0 | 0 | 0.00 |
| Negative recorded payment amount | 0 | 0 | 0.00 |
| Zero scheduled amount | 290 | 58 | 0.00 |
| Zero recorded payment amount | 1,440 | 1,098 | 0.01 |
| Positive scheduled day (after reference application) | 0 | 0 | 0.00 |
| Positive payment-entry day (after reference application) | 0 | 0 | 0.00 |
| Date sentinel 365243 in either day field | 0 | 0 | 0.00 |
| Negative installment version | 0 | 0 | 0.00 |
| Non-integer observed installment version | 0 | 0 | 0.00 |
| Nonpositive installment number | 0 | 0 | 0.00 |

## Candidate features for later applicant-level aggregation

- Explicit history/observed-date/observed-amount coverage indicators and contract/key/record counts.
- Late-entry counts/rates, signed and positive-only delay summaries, severe-delay counts, and recency of latest recorded lateness (largest signed relative day).
- Distinct version-specific late keys and recurrence, with separate credit-card version-0 handling; avoid counting split entries as independent episodes.
- Eligible-key recorded coverage ratios/shortfalls, observed coverage delay, and repeated shortfall counts; retain eligibility and censoring flags.
- Multiple-entry prevalence, schedule-version changes, same-number revision flags, missingness, inconsistent-schedule indicators, and observation span.
- Recent-window and contract-normalized behavior summaries with explicit denominators and weighting. These remain proposals; no applicant feature aggregation is performed.

## Limitations and open questions

1. What is the source's payment-allocation rule? Are repeated equal entries genuine separate payments or replayed records? Exact duplicates are not assigned settlement outcomes here.
2. Do versions supersede prior obligations, and how should version 0/card schedules be reconciled? This EDA keeps versions separate; contract key counts are not a unique physical-installment count or contractual term.
3. Recorded insufficient sums do not prove final underpayment: future/missing payments, allocation, revision, and censoring are unresolved. Coverage timing excludes keys lacking sufficient dated positive payments and is subject to selection bias.
4. Relative days have applicant-specific origins. Entries within contracts/applicants are correlated; longer histories and split entries inflate raw event counts. No-history or unknown timing is not clean repayment history.
5. Validate monetary units, extreme values, unmatched IDs, and all future feature availability at application time. Later predictive work requires outcome linkage and applicant-isolated validation; no `TARGET` analysis is performed here.

Notebook: `notebooks/05_installments_payments_eda.ipynb`; figures: `reports/figures/installments_*.png`.
