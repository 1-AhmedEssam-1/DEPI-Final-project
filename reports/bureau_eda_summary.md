# Bureau historical EDA summary

Analysis covers 1,716,428 bureau accounts (17 columns) and 27,299,925 monthly bureau-balance records (3 columns), linked through `SK_ID_BUREAU`; the account table links to 305,811 applicants through `SK_ID_CURR`. The monthly CSV was processed in 500,000-row chunks. No `TARGET` values were loaded, original CSVs were not modified, and applicant-level feature aggregation was not performed.

## Portfolio and timing

- Observed applicants have a median of 4 external bureau accounts (p95 14; maximum 116).
- `CREDIT_ACTIVE` is 62.9% Closed and 36.7% Active. Consumer credit (1,251,615 accounts) and credit cards (402,195) dominate. `currency 1` is 99.92% of reported accounts.
- `CNT_CREDIT_PROLONG` is zero for 99.47% of accounts.
- Median `DAYS_CREDIT` is −987 days; median reported actual end-date offset is −897 days. Expected end date is missing for 6.15% of accounts. There are 164 negative planned durations (`DAYS_CREDIT_ENDDATE - DAYS_CREDIT`) to validate.

## Financial exposure

- `AMT_CREDIT_SUM` is nearly complete (13 missing) but highly right-skewed: median 125,519, p99 3.70 million, maximum 585 million.
- `AMT_CREDIT_SUM_DEBT` is missing for 15.01%; its median is zero, 8,418 recorded values are negative, and its maximum is 170.1 million. Among observed balances, 76.65% of Active accounts have positive debt, versus 0.74% of Closed accounts. Debt exceeds positive credit sum for 27,644 accounts.
- `AMT_CREDIT_SUM_OVERDUE` has no missing values and is zero for 99.76% of accounts; 4,158 accounts have positive overdue, with a maximum of 3.76 million.
- `AMT_CREDIT_SUM_LIMIT` is missing for 34.48%. The actual source field `AMT_CREDIT_MAX_OVERDUE` is missing for 65.51%, with a maximum of 115.99 million.
- **Schema discrepancy:** `AMT_CREDIT_SUM_MAX` is not present in `bureau.csv`. The notebook analyzes `AMT_CREDIT_MAX_OVERDUE` under its actual name; it is a distinct historical-maximum-overdue field, not an alias.
- Credit sum and reported debt have a positive account-level correlation (~0.68); other amount relationships are much weaker. Extreme values and negative debt/limits warrant review rather than automatic clipping or replacement.

## Monthly history and historical patterns

- Monthly data covers 817,395 bureau IDs; 43,041 of those IDs do not match an account in `bureau.csv`. Monthly status rows are therefore not applicant counts, and unmatched IDs are excluded from account-level joined comparisons.
- Status `C` is 49.99% of monthly records, `0` is 27.47%, and `X` is 21.28%. Delinquent statuses `1`–`5` together account for 1.26% of records.
- Median history length is 26 monthly rows per bureau account. Among accounts with monthly history, 12.63% have at least one delinquent month and 1.64% experienced multiple distinct delinquent status codes.
- Accounts with and without monthly history both have a zero median reported debt. Positive overdue prevalence is 0.24% among accounts with history and 0.25% among those without. Monthly delinquency is observable only for accounts with history; no-history accounts are not treated as non-delinquent.
- Among major credit types, positive overdue is uncommon (0.20% for consumer credit; 0.37% for credit cards). Active accounts show more reported debt than closed accounts, but this is descriptive and does not establish causation or default prediction. Accounts without monthly history have unobserved, not zero, monthly delinquency.

## Data quality and next-stage candidates

Retain explicit missingness indicators for debt, credit limit, and maximum-overdue measures; do not impute missing amounts as zero. Check extreme monetary values, negative debt/limits, the `AMT_CREDIT_SUM_MAX` schema discrepancy, unmatched monthly IDs, and the 164 negative planned durations. Potential later features include bureau-account counts by active state/type, observed-value counts or rates, debt/overdue summaries with missingness indicators, most-recent report/update timing, planned duration, monthly-history length, delinquent-month frequency, maximum status, and distinct delinquent-status count. Any aggregation to applicants, currency handling, and missing-value strategy belongs to the later feature-engineering stage.
