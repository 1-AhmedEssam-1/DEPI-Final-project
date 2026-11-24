# Bureau Balance Account Aggregation — PASS

Input: `/run/media/momen/Life/Projects/DEPI Final Project/data/raw/bureau_balance.csv`. Output: `/run/media/momen/Life/Projects/DEPI Final Project/data/processed/bureau_balance_features.parquet`.

- Input monthly records: **27,299,925**; unique source account IDs: **817,395**.
- Output rows: **817,395**; features: eight plus `SK_ID_BUREAU`.
- Processing: 500,000-row CSV chunks; 32 temporary account-ID partitions. An account is never split between aggregation partitions; cross-chunk duplicates are included in checks.
- Elapsed: **42.46 seconds**; peak process RSS: **572.9 MiB** (Linux).
- Python/Pandas/PyArrow: `3.14.10` / `3.0.5` / `26.0.1`.

## Duplicate monthly-key investigation

- Repeated `(SK_ID_BUREAU, MONTHS_BALANCE)` keys: **0**.
- Extra rows beyond one per key: **0**; all rows at repeated keys: **0**.
- Exact duplicate extra rows: **0**; conflicting-status monthly keys: **0**.
- Accounts with conflicting statuses at their latest month: **0**.
Policy: retain every source record, including any repeated rows, in record-based counts. Audit repeated keys and their status agreement. Latest ties may share the same status (including all missing); conflicting latest statuses stop publication. No monthly rows are silently deduplicated.

## Missingness and source statuses

Input missingness uses all observed monthly records as its denominator. Missing account IDs or months prevent publication; missing statuses remain unknown.

| Input field | Missing records |
| --- | --- |
| SK_ID_BUREAU | 0 |
| MONTHS_BALANCE | 0 |
| STATUS | 0 |

| Observed source status | Records |
| --- | --- |
| 0 | 7,499,507 |
| 1 | 242,347 |
| 2 | 23,419 |
| 3 | 8,924 |
| 4 | 5,847 |
| 5 | 62,406 |
| C | 13,646,993 |
| X | 5,810,482 |

Unexpected nonmissing status frequencies: `{}`. Only literal 0–5 map to numeric levels; other codes remain categorical for latest status.

| Output field | Missing accounts |
| --- | --- |
| SK_ID_BUREAU | 0 |
| bb_months_count | 0 |
| bb_delinquent_months | 0 |
| bb_delinquent_rate | 0 |
| bb_max_status | 130,368 |
| bb_distinct_delinquent_statuses | 0 |
| bb_latest_status | 0 |
| bb_history_oldest_month | 0 |
| bb_has_delinquency | 0 |

`bb_max_status` is null when an account has no observed numeric 0–5 status. `bb_latest_status` is null when the status at the actual latest month is missing. `C`, `X`, and missing statuses are never encoded as numeric zero. They still contribute to the denominator of the requested record-based delinquency rate.

## Feature definitions

| Feature | Definition |
| --- | --- |
| bb_months_count | Number of all observed monthly records, including missing status or repeated rows |
| bb_delinquent_months | Count of records with literal STATUS 1–5 |
| bb_delinquent_rate | Delinquent records / all observed records; fraction on [0, 1] |
| bb_max_status | Maximum among literal numeric statuses 0–5; null if none |
| bb_distinct_delinquent_statuses | Number of distinct literal statuses among 1–5 |
| bb_latest_status | Raw categorical status at largest MONTHS_BALANCE; retain C/X/null |
| bb_history_oldest_month | Smallest observed MONTHS_BALANCE |
| bb_has_delinquency | 1 iff delinquent record count is positive; otherwise 0 |

## Validation results

| Check | Result |
| --- | --- |
| Input has exactly the expected three columns | PASS |
| No missing source account IDs | PASS |
| No missing source months for temporal features | PASS |
| Latest status equals the status at the largest source month | PASS |
| Latest endpoint uses the maximum observed MONTHS_BALANCE | PASS |
| Partition record counts retain every source row | PASS |
| Partition delinquency counts use only statuses 1–5 | PASS |
| No conflicting statuses at any account latest month | PASS |
| Output SK_ID_BUREAU is unique | PASS |
| Output rows equal independently counted unique source account IDs | PASS |
| Output account IDs exactly match source account IDs | PASS |
| No null output account IDs | PASS |
| No null month counts | PASS |
| Every output account has a positive nonnegative month count | PASS |
| Sum of observed month counts equals source rows | PASS |
| Delinquent months are between zero and observed month counts | PASS |
| Delinquency rates are defined and within [0, 1] | PASS |
| Delinquency rates equal delinquent records / all observed records | PASS |
| Has-delinquency flags agree with delinquent counts | PASS |
| Maximum numeric statuses are within 0–5 or genuinely missing | PASS |
| Distinct delinquent statuses are within 0–5 | PASS |
| Distinct status counts do not exceed delinquent record counts | PASS |
| Total delinquent counts equal source status 1–5 frequencies | PASS |
| Null numeric maxima match accounts without any source status 0–5 | PASS |
| Original monthly CSV SHA-256 is unchanged | PASS |
| Parquet read-back exactly preserves feature values, columns, nulls and dtypes | PASS |

Input SHA-256 before: `33e09f06174c26f0be6b8b7398886c69e7bf0abbb29b4122f7841ffe545729a9`. After: `33e09f06174c26f0be6b8b7398886c69e7bf0abbb29b4122f7841ffe545729a9`.

The Parquet is written to a temporary output file, read back and compared with the validated features, then atomically published. Temporary monthly partitions are removed automatically. No `TARGET`, current application table, or `bureau.csv` is loaded; no join or applicant aggregation is performed.

Reproduce from the project root: `.venv/bin/python scripts/aggregate_bureau_balance.py`. Dependencies: `pandas` and `pyarrow` from `requirements.txt`.
