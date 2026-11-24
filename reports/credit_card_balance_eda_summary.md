# Credit Card Balance EDA — Findings Summary

Descriptive EDA of `credit_card_balance.csv`, informed by the five preceding EDA notebooks. Only previous-application IDs are loaded for linkage validation. No `TARGET`, current application dataset, applicant features, or final historical feature table is used or created. Source SHA-256 checks pass unchanged.

## Structure and linkage

- **3,840,312 monthly rows × 23 columns**, **104,307 previous contracts**, **103,558 observed applicants**; compact Pandas memory **567.7 MiB**, with float64 amounts/counts.
- Grain is contract × relative month. Exact duplicate extras **0**, repeated monthly-key extras **0**, inconsistent applicant mappings **0**. Matched-row applicant-ID disagreements **0**.
- Previous IDs matched **92,935/104,307 (89.10%)**; unmatched **11,372**. Unmatched monthly rows **1,082,816/3,840,312 (28.20%)**. Previous applications without card snapshots **1,577,279/1,670,214**. Unmatched histories remain in the EDA.
- **`AMT_TRANS_TOTAL` is absent**, so no substitute is constructed. Source spelling `AMT_RECIVABLE` is preserved.

## Monthly history

Relative months span **-96 to -1**. Distinct months per contract: median **21**, p95 **96**, maximum **96**. Internal gaps occur in **14/104,307 contracts**. All **104,307 contracts** have an unambiguous actual latest snapshot; **62,356** are at month −1. Missing months are not inserted as zero activity or zero DPD; observed span is not contractual term.

## Balances and utilization

Median balance **0.000**, p99 **466,295.116**, maximum **1,505,902.185** dataset units. There are **2,345 negative balances**, **753,823 zero limits**, and **0 missing limits**.

Utilization requires a positive finite limit and finite observed balance; signed negative balances remain. Eligible monthly ratios **3,086,489/3,840,312**, latest ratios **79,335/104,307**. Median monthly/latest utilization **0.011/0.000**.

| Snapshot population | Threshold | Count | Eligible utilization denominator | pct_of_eligible |
| --- | --- | --- | --- | --- |
| All raw monthly rows | ≥80% | 887,937 | 3,086,489 | 28.77 |
| All raw monthly rows | ≥90% | 704,619 | 3,086,489 | 22.83 |
| All raw monthly rows | ≥100% | 334,551 | 3,086,489 | 10.84 |
| All raw monthly rows | >100% | 334,362 | 3,086,489 | 10.83 |
| Actual latest snapshots | ≥80% | 23,227 | 79,335 | 29.28 |
| Actual latest snapshots | ≥90% | 19,664 | 79,335 | 24.79 |
| Actual latest snapshots | ≥100% | 9,835 | 79,335 | 12.40 |
| Actual latest snapshots | >100% | 9,828 | 79,335 | 12.39 |

Above-limit values remain unchanged. **819/107,357 observed consecutive-month onsets** above 100% coincide with a decreasing limit; this is context, not proof of the mechanism. **43,752/103,134 utilization-observed contracts (42.42%)** ever exceed 100%.

## Payments, drawings, counts, and receivables

- Current-payment missingness **20.00%**, total-payment **0.00%**, minimum-installment **7.95%**, drawing-category amounts/counts **19.52%** of all monthly rows. Missing is not zero. **31,862 contracts** have no observed current-payment amounts.
- Total payment is below current payment in **1,187,232/3,072,324 observed pairs (38.64%)** and above it in **0**. Reporting scope needs clarification; field names do not establish ordering or interchangeability.
- Total payment is below the same-month positive minimum in **214,321/1,606,212 eligible monthly rows (13.34%)**. Equally weighted mean contract share is **17.92%** across **70,179 eligible contracts**. This is not verified billing compliance. Payment/balance comparisons also use positive denominators; lagged comparisons require consecutive months and a known positive prior balance.
- Total drawing amount/count is zero in **83.94%/84.11%** of observed rows. Maximum drawing amount **2,287,098.315**, total payment **4,278,315.690**, drawing count **165**. Full tails and negative values remain in audits.
- Drawing amounts disagree with complete category sums in **7,150/3,090,496 rows**; drawing counts agree in all **3,090,496 complete rows**. Eligible category composition is **ATM 64.67%**, **POS 32.21%**, **other 3.13%** of summed category flows, not credit exposure.
- Mature counts increase/stay unchanged/decrease in **45.76%/54.24%/0.00003%** of **3,430,755 eligible consecutive pairs**; **1/103,585 evaluable contracts** have a decrease.
- Negative principal/receivable/total-receivable values occur in **2,428/109,338/109,330 rows**. Ledger corrections/refunds and stock/flow cutoffs remain unresolved; none are replaced with zero.

## Delinquency and contract behavior

Ordinary DPD and small-debt-tolerance-adjusted DPD remain separate; **64,439 rows** differ and **0** have adjusted DPD above ordinary. Monthly positive rates are **3.99%/2.33%**; latest rates **1.28%/0.31%**. Maximum ordinary/adjusted DPD is **3,260/3,260 days**; ≥90-day rows **48,377/1,078** out of **3,840,312 monthly rows**.

| DPD field | Contract event | Contracts | Eligible contract denominator | pct_of_eligible_contracts |
| --- | --- | --- | --- | --- |
| SK_DPD | Any positive month | 20,672 | 104,307 | 19.82 |
| SK_DPD | ≥2 positive months | 14,264 | 104,307 | 13.68 |
| SK_DPD | Consecutive-positive month pair | 10,433 | 103,585 | 10.07 |
| SK_DPD_DEF | Any positive month | 17,036 | 104,307 | 16.33 |
| SK_DPD_DEF | ≥2 positive months | 11,797 | 104,307 | 11.31 |
| SK_DPD_DEF | Consecutive-positive month pair | 8,007 | 103,585 | 7.73 |

Consecutive indicators are unknown without an evaluable pair. Repeated positive months can reflect persistent arrears. Latest status is **87.20% Active**, **12.75% Completed**; **17,888/103,585 evaluable contracts** change known status.

Above-limit ordinary/adjusted positive DPD is **10.87%/10.25%**, versus **3.98%/1.39%** below 80% with nonnegative balances. Ordinary positive DPD among positive-balance zero-total-payment rows is **48.64% (225,405 rows)**, versus **2.91% (1,456,142 rows)** with positive payment. These are monthly co-occurrences, not independent contract comparisons.

**14,155/104,307 contracts** have balance >1 at every observed snapshot; **43,004/104,307** have |balance| ≤1 in ≥80% of observed balance months. These exploratory dataset-unit thresholds do not establish interest-bearing debt. Median histories are **16 months without observed ordinary DPD** versus **90 with repeated DPD**, limiting intrinsic-risk comparisons.

## Exclusions, limitations, and later candidates

Raw values and unmatched histories are preserved. Repeated monthly keys are excluded only from ordered/numeric contract summaries; actual endpoints must be unambiguous. Ratios require observed finite values and positive denominators; behavioral payment comparisons additionally require nonnegative payments. Drawing sums require all components. Missing months/status/amounts remain unknown. Numerical equality uses **1e-6 dataset units**; charts label positive-only/log/display subsets without clipping data. No nonnegative relative months are present, but application-time availability still requires reporting-latency checks.

Billing alignment, allocation, fees/adjustments, tolerance amounts, and history censoring remain unresolved. No-history is not clean history; observations within contracts/applicants are correlated. No causal effect or predictive performance is established.

Candidate applicant features, **not constructed here**: coverage/counts/recency/gaps; latest/mean/max utilization and high-utilization shares; ordinary/adjusted DPD recurrence/severity/recency with known denominators; payment/drawing activity and minimum comparisons; balance persistence and endpoint changes; statuses/transitions; zero-limit, missingness, negative-value, drawing-consistency, and maturity-reset flags. Validate statement alignment and observation windows before later applicant aggregation.

Notebook: `notebooks/06_credit_card_balance_eda.ipynb`; eight figures: `reports/figures/credit_card_*.png`.
