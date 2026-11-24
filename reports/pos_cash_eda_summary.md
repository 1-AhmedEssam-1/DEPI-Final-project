# POS/CASH Balance EDA — Findings Summary

Executed descriptive EDA of `POS_CASH_balance.csv`; context: Data Understanding, Bureau EDA, and Previous Application EDA. No `TARGET` or current application table was loaded. Raw CSV content is verified unchanged. Analysis stays at monthly and previous-contract grain; only portfolio counts and unique-applicant coverage are reported at applicant grain. No applicant feature table or final historical integration was created.

## Dataset, grain, and linkage

- **10,001,358 monthly records, 8 columns**, **936,325 previous contracts**, and **337,252 observed applicants**. Compact Pandas data uses **257.5 MiB**.
- Grain: previous contract × relative month. Exact duplicate extra rows: **0**; repeated contract-month extra rows: **0**; conflicting month rows after exact deduplication: **0**. Contract-to-applicant mapping conflicts: **0**. Conflicting monthly keys are excluded from ordered comparisons; all raw rows remain in row descriptions.
- POS IDs matched to previous applications: **898,903/936,325 (96.00%)**; unmatched POS IDs: **37,422**. Previous application IDs without POS history: **771,311/1,670,214 (46.18%)**.

| Linkage metric | Count | Denominator | pct_of_denominator |
| --- | --- | --- | --- |
| POS previous IDs matched to previous_application | 898,903 | 936,325 | 96.00 |
| POS previous IDs unmatched to previous_application | 37,422 | 936,325 | 4.00 |
| Previous application IDs with POS history | 898,903 | 1,670,214 | 53.82 |
| Previous application IDs without POS history | 771,311 | 1,670,214 | 46.18 |
| Matched POS monthly rows | 9,660,797 | 10,001,358 | 96.59 |
| Unmatched POS monthly rows | 340,561 | 10,001,358 | 3.41 |
| Applicant-ID mismatches among matched POS rows | 0 | 9,660,797 | 0.00 |

## Portfolio and history

- Distinct months per contract: median **10**, p95 **24**, maximum **96**. Contracts per observed applicant: mean **2.78**, median **2**, maximum **26**.
- `MONTHS_BALANCE` spans **-96 to -1**; nonnegative rows: **0**. These are application-relative months, not common calendar dates.
- **1,950/936,325 contracts (0.21%)** have gaps inside their observed span. **717,932/936,325 (76.68%)** have an observed status transition. Earliest/latest snapshots and early/middle/late history thirds describe observed lifecycle coverage, not a complete repayment timeline.
- Latest observed snapshots are **698,421 Completed (74.59% of contracts)** and **236,149 Active (25.22%)**. This differs from the row-weighted Active majority because active months repeat while completion often appears once.

| NAME_CONTRACT_STATUS | monthly_rows | pct_of_all_monthly_rows |
| --- | --- | --- |
| Active | 9,151,119 | 91.50 |
| Completed | 744,883 | 7.45 |
| Signed | 87,260 | 0.87 |
| Demand | 7,065 | 0.07 |
| Returned to the store | 5,461 | 0.05 |
| Approved | 4,917 | 0.05 |
| Amortized debt | 636 | 0.01 |
| Canceled | 15 | 0.00 |
| XNA | 2 | 0.00 |

Status frequencies are monthly-row weighted; the same contract can contribute multiple statuses and longer histories contribute more rows. Lookback-period and observed-history-stage plots show lifecycle/composition differences.

## Delinquency and persistence

`SK_DPD_DEF` applies a small-debt tolerance per the source dictionary. Different values are not automatically errors. **183,880 monthly rows (1.84%)** differ; **0** have adjusted DPD greater than ordinary DPD. **181,258 rows** have positive ordinary DPD and zero adjusted DPD; **2,622** have both positive but different.

| DPD field | monthly_rows | negative_rows | zero_rows | positive_rows | severe_rows_ge_90 | zero_pct_of_all_rows | positive_pct_of_all_rows |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SK_DPD | 10,001,358 | 0 | 9,706,131 | 295,227 | 119,274 | 97.05 | 2.95 |
| SK_DPD_DEF | 10,001,358 | 0 | 9,887,389 | 113,969 | 4,688 | 98.86 | 1.14 |

| DPD field | Unit and event | Count | Denominator | pct_of_denominator |
| --- | --- | --- | --- | --- |
| SK_DPD | Monthly records with positive DPD | 295,227 | 10,001,358 | 2.95 |
| SK_DPD | Contracts with ≥1 positive month | 77,032 | 936,325 | 8.23 |
| SK_DPD | Contracts with ≥2 positive months | 37,780 | 936,325 | 4.03 |
| SK_DPD | Applicants with ≥1 positive snapshot on any contract | 62,984 | 337,252 | 18.68 |
| SK_DPD_DEF | Monthly records with positive DPD | 113,969 | 10,001,358 | 1.14 |
| SK_DPD_DEF | Contracts with ≥1 positive month | 54,841 | 936,325 | 5.86 |
| SK_DPD_DEF | Contracts with ≥2 positive months | 22,491 | 936,325 | 2.40 |
| SK_DPD_DEF | Applicants with ≥1 positive snapshot on any contract | 45,949 | 337,252 | 13.62 |

- **49.04% of contracts with any ordinary delinquency** have ≥2 distinct positive months. **30,649/936,325 contracts (3.27%)** have at least one consecutive-calendar-month positive pair. Repeated observations can represent persistent arrears, separate late payments, or reporting gaps; they are not necessarily separate events.
- Exploratory severe threshold: **≥90 days**, not an observed default label. Extreme values remain unchanged; inspection considers persistence and tolerance differences.

| Field | Monthly rows ≥90 | Contracts with ≥90 | Applicants with ≥90 | Maximum DPD |
| --- | --- | --- | --- | --- |
| SK_DPD | 119,274 | 5,360 | 5,253 | 4,231 |
| SK_DPD_DEF | 4,688 | 174 | 151 | 3,595 |

## Installments and descriptive risk patterns

- Total/remaining installment missingness: **26,071 / 26,087 rows (0.261% / 0.261% of all raw monthly rows)**. Remaining > total: **7,420 paired-value rows**. **304,293 contracts (32.50%)** have changing observed total terms.
- Consecutive-month evaluable remaining-count pairs: **9,038,612**; decreasing **94.10%**, unchanged **5.87%**, increasing **0.03%**. **2,651/926,464 evaluable contracts (0.29%)** have any increase. Term changes and reporting/settlement conventions need investigation; total minus remaining is not a verified payment count.
- **85.36% of remaining-count increases** coincide with an increasing total term; the rest have unchanged terms in this dataset. A changing schedule can explain some increases, while others need reporting-context review.

| Flag | Monthly rows | Affected contracts | Row denominator | pct_of_denominator |
| --- | --- | --- | --- | --- |
| Negative total installments | 0 | 0 | 10,001,358 | 0.00 |
| Negative remaining installments | 0 | 0 | 10,001,358 | 0.00 |
| Remaining > total (both observed) | 7,420 | 4,116 | 9,975,174 | 0.07 |
| Non-integer total (observed) | 0 | 0 | 10,001,358 | 0.00 |
| Non-integer remaining (observed) | 0 | 0 | 10,001,358 | 0.00 |
| Total = 0 but remaining > 0 | 0 | 0 | 10,001,358 | 0.00 |
| Completed status but remaining > 0 | 4 | 4 | 10,001,358 | 0.00 |
| Active status with remaining = 0 | 432,501 | 207,342 | 10,001,358 | 4.32 |

- Median observed history is **9 months** for no observed delinquency, **12** for one positive month, and **13** for repeated delinquency. Observation length changes the opportunity to detect delinquency.
- The notebook compares DPD across monthly statuses, remaining-installment bands (including missing), and contract-history lengths. Zero remaining installments and terminal-status rows can retain DPD; lifecycle status alone is insufficient to characterize repayment. No descriptive association establishes causation or default prediction.
- Ordinary DPD prevalence is **49.64% in Demand** (7,065 rows, 368 contracts) and **93.08% in Amortized debt** (636 rows, only 17 contracts). Rare status groups have limited independent contract support.
- In **1,185,960 zero-remaining rows**, ordinary DPD is positive in **12.41%**, versus **1.36%** for adjusted DPD. Zero remaining scheduled installments do not prove that all arrears have been extinguished.

## Candidates for later applicant-level aggregation

- Number of observed POS/cash contracts and monthly-history coverage, with an explicit no-history flag.
- Counts/fractions of contracts with any, repeated, consecutive, and ≥90 DPD; keep ordinary and tolerance-adjusted DPD separate.
- Contract maximum DPD, contract mean DPD, fraction of observed months delinquent, and candidate applicant summaries with explicit contract/row weighting.
- Recency of latest observed positive-DPD month (largest signed relative month), recent-window delinquency, and latest observed status/DPD.
- History length and gaps, installment-count missingness, changes in reported total term, remaining-count increases, and status transitions.
- DPD discrepancy frequency and terminal-status/remaining-count inconsistencies as review flags, subject to confirmed source semantics.

These are proposals only. Applicant feature aggregation, normalization, lookback selection, and historical-dataset integration belong to later work.

## Limitations and open questions

1. Missing/no POS history and zero observed DPD do not prove no lifetime delinquency. Histories can be left/right censored or have gaps; longer histories raise detection opportunity.
2. Relative months are not calendar cohorts, and monthly rows within contracts/applicants are correlated. Status/remaining bands overlap in their contract membership.
3. The small-debt tolerance amount and precise update rules are unavailable. Extreme DPD, adjusted-DPD reversals, installment increases, and terminal-status inconsistencies require source/payment context; no automatic clipping or imputation is justified.
4. Match coverage validates IDs, not complete product or repayment coverage. Unmatched histories require investigation; previous applications without POS may reflect product selection or applications that did not create this repayment history.
5. Validate future feature definitions against payment records and product terms. Any predictive claim needs outcome linkage, application-time availability checks, applicant-isolated validation, and evaluation in a later stage.

Notebook: `notebooks/04_pos_cash_eda.ipynb`. Figures: `reports/figures/pos_cash_*.png`.
