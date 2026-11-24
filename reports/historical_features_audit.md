# Historical Features Audit

Audited 2026-11-22T15:34:53+00:00 (UTC). Read-only inspection of five Parquet tables and their feature notebooks. No raw CSV was opened; no tables were merged, no values were changed, and no existing file was overwritten.

## 1. Overall summary

**Structurally ready for one-to-one integration, with Bureau completeness/zero-total semantics requiring review.** All five tables have nonmissing, unique `SK_ID_CURR` values. The 395 feature names are distinct across tables; all features are numeric, with no object columns, infinities, or entirely missing columns.

The four mapping-based tables contain the same **338,857 applicants**. Bureau contains **305,811**; the historical union is **353,577**, with **291,091** present in all five. A mapping-population left join would discard **14,720 Bureau-only applicants**. Use an explicit intended-cohort ID spine; for historical-only coverage, use the union and left-join each table.

Bureau review items: **9 previously engineered Bureau Balance features plus one debt ratio are absent from the saved table**; **8,372** applicants have zero debt totals despite all debt values being missing. Full findings and recommendations appear below.

## 2. Dataset dimensions and applicant IDs

| Dataset | Rows | Columns | Features | ID exists | ID dtype | Missing IDs | Unique IDs | Duplicate extra rows | Repeated distinct IDs | One row/applicant | ID range |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Bureau | 305,811 | 22 | 21 | Yes | int64 | 0 | 305,811 | 0 | 0 | Yes | 100,001–456,255 |
| Previous Application | 338,857 | 56 | 55 | Yes | int32 | 0 | 338,857 | 0 | 0 | Yes | 100,001–456,255 |
| POS/CASH | 338,857 | 89 | 88 | Yes | int32 | 0 | 338,857 | 0 | 0 | Yes | 100,001–456,255 |
| Installments | 338,857 | 97 | 96 | Yes | int32 | 0 | 338,857 | 0 | 0 | Yes | 100,001–456,255 |
| Credit Card | 338,857 | 136 | 135 | Yes | int32 | 0 | 338,857 | 0 | 0 | Yes | 100,001–456,255 |

“Duplicate extra rows” counts rows beyond the first for each nonmissing applicant ID. “Repeated distinct IDs” counts distinct IDs appearing more than once. All ranges are basic sanity checks, not validation against a main application cohort, which was outside this audit.

## 3. Feature names and data types

| Dataset | Numeric features | Boolean features | Other nonnumeric features | Object features | Duplicate columns | Duplicate feature names | Entirely missing features | Infinity cells |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Bureau | 21 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Previous Application | 55 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| POS/CASH | 88 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Installments | 96 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Credit Card | 135 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

Counts exclude the identifier. Numeric excludes boolean; the three dtype categories sum to each table’s feature count. Float32/float64 and int32/int64 feature types are compatible. Bureau IDs are int64 and the other IDs int32; their shared integer values align exactly. Binary indicators stored numerically are not boolean-dtype columns. The full name/dtype inventory is in Appendix A.

## 4. Missingness

| Dataset | Missing cells incl. ID | Missing feature cells | Feature-cell missing % | Features with nulls | Features >50% null | Features 100% null |
| --- | --- | --- | --- | --- | --- | --- |
| Bureau | 16,748 | 16,748 | 0.2608 | 4 | 0 | 0 |
| Previous Application | 78,320 | 78,320 | 0.4202 | 11 | 0 | 0 |
| POS/CASH | 1,656,488 | 1,656,488 | 5.5551 | 52 | 2 | 0 |
| Installments | 1,635,632 | 1,635,632 | 5.0280 | 54 | 2 | 0 |
| Credit Card | 20,389,766 | 20,389,766 | 44.5720 | 81 | 81 | 0 |

Feature-cell percentages use rows × feature columns, excluding the ID. Identifier missingness is audited separately and is zero here. Entirely missing features: **none**. “Very high” below means ≥90%; this is a reporting threshold, not a feature-drop recommendation.

| Dataset | Feature | Missing count | Missing % |
| --- | --- | --- | --- |
| Credit Card | credit_card_dpd_def_last_positive_month | 327,703 | 96.7083 |
| Credit Card | credit_card_dpd_last_positive_month | 324,643 | 95.8053 |

### Source availability versus table membership

| Dataset | History indicator | Applicants with observed/mapped history | Applicants with zero history | Unknown indicator | Null feature cells in zero-history applicants | Other null feature cells |
| --- | --- | --- | --- | --- | --- | --- |
| Bureau | bureau_total_accounts | 305,811 | 0 | 0 | 0 | 16,748 |
| Previous Application | prev_total_applications | 338,857 | 0 | 0 | 0 | 78,320 |
| POS/CASH | pos_contracts_with_history | 334,359 | 4,498 | 0 | 233,896 | 1,422,592 |
| Installments | installments_contracts_with_history | 336,935 | 1,922 | 0 | 103,788 | 1,531,844 |
| Credit Card | credit_card_contracts_with_history | 92,447 | 246,410 | 0 | 19,959,210 | 430,556 |

Bureau’s indicator describes observed bureau accounts, not Bureau Balance monthly coverage, which cannot be measured from the omitted features. Previous Application’s indicator describes recorded applications. POS/CASH, Installments, and Credit Card count mapped source contracts; all their table rows still belong to the previous-mapping population.

High missingness can be expected: no mapped source history; no usable payment/limit/timing; no eligible recent window or consecutive pair; or no positive delinquency/late-payment event. A missing last-positive-event date does not establish missing source history. Zero-history counts are already deliberately encoded by the source notebooks; this audit neither converts them to null nor fills any nulls. Appendix A identifies the no-history null count for every feature. Appendix B lists every feature’s missingness in descending order.

## 5. Pairwise applicant ID overlap

| Dataset A | Dataset B | A unique IDs | B unique IDs | Shared | Only A | Only B | Jaccard |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Bureau | Previous Application | 305,811 | 338,857 | 291,091 | 14,720 | 47,766 | 0.8233 |
| Bureau | POS/CASH | 305,811 | 338,857 | 291,091 | 14,720 | 47,766 | 0.8233 |
| Bureau | Installments | 305,811 | 338,857 | 291,091 | 14,720 | 47,766 | 0.8233 |
| Bureau | Credit Card | 305,811 | 338,857 | 291,091 | 14,720 | 47,766 | 0.8233 |
| Previous Application | POS/CASH | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |
| Previous Application | Installments | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |
| Previous Application | Credit Card | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |
| POS/CASH | Installments | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |
| POS/CASH | Credit Card | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |
| Installments | Credit Card | 338,857 | 338,857 | 338,857 | 0 | 0 | 1.0000 |

Jaccard = shared / union of the two unique ID sets. Table-level overlap does not imply that every shared applicant has monthly/payment history in every source.

## 6. Combined and dataset-specific applicant coverage

| Coverage | Applicants |
| --- | --- |
| Union of all five | 353,577 |
| Exactly 5 table(s) | 291,091 |
| Exactly 4 table(s) | 47,766 |
| Exactly 3 table(s) | 0 |
| Exactly 2 table(s) | 0 |
| Exactly 1 table(s) | 14,720 |

Exactly-four applicants are the **47,766 mapping-population applicants without Bureau rows**; exactly-one applicants are the **14,720 Bureau-only applicants**. Exactly-two and exactly-three populations are zero because the other four tables have identical ID sets.

| Dataset | Shared with ≥1 other table | Shared with all other four | Exclusive to this dataset |
| --- | --- | --- | --- |
| Bureau | 291,091 | 291,091 | 14,720 |
| Previous Application | 338,857 | 291,091 | 0 |
| POS/CASH | 338,857 | 291,091 | 0 |
| Installments | 338,857 | 291,091 | 0 |
| Credit Card | 338,857 | 291,091 | 0 |

### Applicants absent from each other dataset

Each cell counts IDs present in the **row** dataset and absent from the **column** dataset.

| Row dataset \ Column dataset | Bureau | Previous Application | POS/CASH | Installments | Credit Card |
| --- | --- | --- | --- | --- | --- |
| Bureau | — | 14,720 | 14,720 | 14,720 | 14,720 |
| Previous Application | 47,766 | — | 0 | 0 | 0 |
| POS/CASH | 47,766 | 0 | — | 0 | 0 |
| Installments | 47,766 | 0 | 0 | — | 0 |
| Credit Card | 47,766 | 0 | 0 | 0 | — |

## 7. Feature-name collision and naming report

| Category | Result |
| --- | --- |
| Expected shared identifier | SK_ID_CURR appears in all five tables; retain one identifier at integration. |
| Cross-table duplicated feature names | 0 |
| Within-table duplicated feature names | 0 |
| Prospective schema | 396 columns: 395 distinct features plus one identifier; no combined dataset created. |
| Prefix conventions | bureau_, prev_, pos_, installments_, credit_card_; all feature names follow their dataset prefix. |
| Documented spelling | credit_card_recivable_* follows the source AMT_RECIVABLE spelling; it was intentionally preserved. |

No actual feature-name collisions were found. Different dtype widths on the shared ID are compatible, not a naming collision.

### Semantic redundancy without a name collision

| Dataset | Feature |
| --- | --- |
| Previous Application | prev_total_applications |
| POS/CASH | pos_total_previous_contracts |
| Installments | installments_total_previous_contracts |
| Credit Card | credit_card_total_previous_contracts |

These four previous-application count columns have identical values for **all 338,857 aligned applicants**. They intentionally support each source’s coverage calculations, but retaining all four creates redundant model inputs. No column was renamed or dropped.

## 8. Integration readiness and issues requiring attention

| Check | Assessment |
| --- | --- |
| Shared ID and dtype | Compatible signed integer IDs; width differs (int64 versus int32). |
| Applicant uniqueness | All five tables have one row per applicant; no missing or repeated IDs. |
| Feature schema | 395 distinct numeric features; no unexpected object/boolean storage, collisions, or infinities. |
| Entirely missing features | None; source-dependent partial missingness remains meaningful. |
| Population alignment | Four equal mapping populations; Bureau adds 14,720 unique IDs and lacks 47,766 mapping IDs. |
| Unexpected rows/IDs | No malformed/missing/duplicated IDs detected. Bureau-only IDs reflect its documented population; master-cohort membership was not checked. |
| Semantic completeness | Review Bureau omitted features and unknown-amount zero totals before treating this as the final feature set. |

### Bureau features engineered but not exported

In [01_bureau.ipynb](../notebooks/history_data/feature_engineering/01_bureau.ipynb), one-based cell **64** merges the Bureau Balance applicant features. Cell **68** then rebuilds `bureau_features` from the base Bureau aggregation; the saved 21-feature table omits the following columns (including the earlier debt ratio from cell 49). This is an absent-column issue, not a 100%-null column issue. Confirm whether these omissions are intended; nothing was repaired.

| Absent engineered feature |
| --- |
| bureau_bb_accounts |
| bureau_bb_accounts_with_history |
| bureau_bb_delinquent_accounts |
| bureau_bb_delinquent_months |
| bureau_bb_delinquent_rate_mean |
| bureau_bb_has_delinquency |
| bureau_bb_history_coverage |
| bureau_bb_max_status |
| bureau_bb_months_mean |
| bureau_accounts_with_debt_ratio |

### Bureau zero totals with missing underlying statistics

| Feature | Affected applicants | Evidence from the saved table |
| --- | --- | --- |
| bureau_debt_total | 8,372 | All account debt values missing according to bureau_missing_debt_accounts; mean/max missing; total is zero. |
| bureau_credit_sum_total | 2 | Credit mean/max missing; total is zero. |

The notebook uses ordinary Pandas `sum()` for these totals, which returns zero for an all-missing group. For debt, the existing missing-account count explicitly identifies the all-missing source values. These zeros should not be interpreted as confirmed zero debt/credit. This audit preserves them and flags the interpretation for review.

Other attention items: retain eligibility/history context when interpreting high missingness; avoid automatic zero filling; review redundant previous-count columns for later model feature selection. Source notebooks document unmatched child contracts and reporting/allocation limits; those cannot be reconstructed from these applicant tables alone.

## 9. Recommended merge strategy

| Strategy (prediction only) | Resulting applicants | Historical-union IDs lost |
| --- | --- | --- |
| Union ID spine + five left joins | 353,577 | 0 |
| Outer joins across all five | 353,577 | 0 |
| Previous-application population as left base | 338,857 | 14,720 |
| Bureau population as left base | 305,811 | 47,766 |
| Inner join across all five | 291,091 | 62,486 |

For a historical-only integration, use the unique **union ID spine** and left-join each feature table on `SK_ID_CURR` with `validate="one_to_one"`. Use a consistent integer ID dtype at the future integration boundary, retain one ID column, assert unchanged spine IDs/row count, and preserve missing features for applicants absent from a table. Outer joins have the same historical ID coverage, but an explicit spine makes the intended population easier to validate.

If the intended final population comes from a main application/training/scoring table, use **that table’s unique applicant IDs** as the spine instead. Historical union cannot include applicants absent from all five sources. A previous-mapping left join is appropriate only if that narrower cohort is explicitly intended; otherwise it loses the Bureau-only applicants. No join was performed here.

## 10. Validation and preservation

| Validation | Result |
| --- | --- |
| Five files read; inventory and missingness cover every feature | PASS |
| Feature dtype counts partition the feature columns | PASS |
| Missing-cell totals equal detailed feature missing counts | PASS |
| All ten pairwise overlaps conserve each ID set and union | PASS |
| Union/intersection and exact-membership histogram agree | PASS |
| Membership-weighted coverage equals sum of dataset unique IDs | PASS |
| Dataset shared/exclusive counts conserve each ID set | PASS |
| Four mapping-population ID sets equal | PASS |
| Four differently named previous-count vectors equal by applicant | PASS |
| Five Parquet and five notebook SHA-256 fingerprints unchanged | PASS |

| Dataset | Audited Parquet file | SHA-256 |
| --- | --- | --- |
| Bureau | `data/interim/bureau_features.parquet` | `d931ccda8e9f691da622a073156aacd07adb36846ff1a1fba555b638a89340a4` |
| Previous Application | `data/interim/previous_application_features.parquet` | `5bd1d79b98b535ebf31dce7ec9438c0dba79937a98ac3a020dd074f63d000e84` |
| POS/CASH | `data/interim/pos_cash_features.parquet` | `cf6aabb979d6c1495e3850f67bd1ccf60c515e2dd250fb8063c627dbcc131b91` |
| Installments | `data/interim/installments_features.parquet` | `696972ad4ebccfccea127761ecf7a5f6aebaf0cb628b832a371233216e1618b4` |
| Credit Card | `data/interim/credit_card_features.parquet` | `5a2a31a5163ce7ffcfccf8804cd9ea1d6a6b73d0ce374e4484d5da61b6c51be0` |

The five Parquet and five notebook content hashes matched before and after the audit. Only this new report is written, with exclusive creation. All aggregate calculations use the saved Parquets; notebook code was read only to explain definitions and omissions. Tables were inspected one at a time, retaining only ID arrays and small summaries/count vectors.

## Appendix A. Complete feature-name and dtype inventory

“No-history nulls” counts nulls for applicants whose table-specific history indicator is zero. This separates structural absence from other eligibility/event-related missingness; it is not an imputation rule.

### Bureau — 21 features

| Feature | Dtype | No-history nulls |
| --- | --- | --- |
| bureau_total_accounts | int64 | 0 |
| bureau_active_accounts | int64 | 0 |
| bureau_closed_accounts | int64 | 0 |
| bureau_consumer_credit_accounts | int64 | 0 |
| bureau_credit_card_accounts | int64 | 0 |
| bureau_credit_sum_total | float64 | 0 |
| bureau_credit_sum_mean | float64 | 0 |
| bureau_credit_sum_max | float64 | 0 |
| bureau_debt_total | float64 | 0 |
| bureau_debt_mean | float64 | 0 |
| bureau_debt_max | float64 | 0 |
| bureau_overdue_total | float64 | 0 |
| bureau_overdue_max | float64 | 0 |
| bureau_overdue_accounts | int64 | 0 |
| bureau_days_overdue_max | int64 | 0 |
| bureau_days_overdue_mean | float64 | 0 |
| bureau_missing_debt_accounts | int64 | 0 |
| bureau_accounts_with_debt | int64 | 0 |
| bureau_days_credit_min | int64 | 0 |
| bureau_days_credit_max | int64 | 0 |
| bureau_days_credit_mean | float64 | 0 |

### Previous Application — 55 features

| Feature | Dtype | No-history nulls |
| --- | --- | --- |
| prev_total_applications | int64 | 0 |
| prev_approved_applications | int64 | 0 |
| prev_refused_applications | int64 | 0 |
| prev_canceled_applications | int64 | 0 |
| prev_unused_offer_applications | int64 | 0 |
| prev_approval_rate | float64 | 0 |
| prev_refusal_rate | float64 | 0 |
| prev_cancellation_rate | float64 | 0 |
| prev_unused_offer_rate | float64 | 0 |
| prev_application_amount_mean | float64 | 0 |
| prev_application_amount_max | float64 | 0 |
| prev_application_amount_min | float64 | 0 |
| prev_credit_amount_mean | float64 | 0 |
| prev_credit_amount_max | float64 | 0 |
| prev_annuity_mean | float64 | 0 |
| prev_annuity_max | float64 | 0 |
| prev_down_payment_mean | float64 | 0 |
| prev_down_payment_max | float64 | 0 |
| prev_goods_price_mean | float64 | 0 |
| prev_goods_price_max | float64 | 0 |
| prev_application_amount_total | float64 | 0 |
| prev_credit_amount_total | float64 | 0 |
| prev_down_payment_total | float64 | 0 |
| prev_cnt_payment_mean | float64 | 0 |
| prev_cnt_payment_max | float64 | 0 |
| prev_valid_term_applications | int64 | 0 |
| prev_planned_duration_days_mean | float64 | 0 |
| prev_planned_duration_days_max | float64 | 0 |
| prev_valid_planned_duration_applications | int64 | 0 |
| prev_days_decision_max | int64 | 0 |
| prev_days_decision_min | int64 | 0 |
| prev_days_since_decision_mean | float64 | 0 |
| prev_distinct_contract_types | int64 | 0 |
| prev_distinct_product_combinations | int64 | 0 |
| prev_distinct_goods_categories | int64 | 0 |
| prev_distinct_channels | int64 | 0 |
| prev_distinct_payment_types | int64 | 0 |
| prev_applications_with_down_payment | int64 | 0 |
| prev_nonfinal_contract_applications | int64 | 0 |
| prev_missing_annuity_applications | int64 | 0 |
| prev_missing_down_payment_applications | int64 | 0 |
| prev_consumer_missing_down_payment_applications | int64 | 0 |
| prev_cash_or_revolving_missing_down_payment_applications | int64 | 0 |
| prev_zero_down_payment_applications | int64 | 0 |
| prev_negative_down_payment_applications | int64 | 0 |
| prev_missing_cnt_payment_applications | int64 | 0 |
| prev_installment_missing_term_applications | int64 | 0 |
| prev_revolving_without_fixed_term_applications | int64 | 0 |
| prev_unknown_contract_type_applications | int64 | 0 |
| prev_unknown_payment_type_applications | int64 | 0 |
| prev_missing_goods_price_applications | int64 | 0 |
| prev_first_due_missing_applications | int64 | 0 |
| prev_original_last_due_missing_applications | int64 | 0 |
| prev_first_due_sentinel_applications | int64 | 0 |
| prev_original_last_due_sentinel_applications | int64 | 0 |

### POS/CASH — 88 features

| Feature | Dtype | No-history nulls |
| --- | --- | --- |
| pos_total_previous_contracts | int32 | 0 |
| pos_contracts_with_history | int32 | 0 |
| pos_latest_active_contracts | int32 | 0 |
| pos_latest_completed_contracts | int32 | 0 |
| pos_latest_demand_contracts | int32 | 0 |
| pos_latest_amortized_debt_contracts | int32 | 0 |
| pos_latest_other_status_contracts | int32 | 0 |
| pos_latest_unknown_status_contracts | int32 | 0 |
| pos_previous_contracts_without_pos_history | int32 | 0 |
| pos_history_coverage_ratio | float64 | 0 |
| pos_active_contract_ratio | float64 | 4,498 |
| pos_completed_contract_ratio | float64 | 4,498 |
| pos_monthly_records_total | int32 | 0 |
| pos_history_months_mean | float64 | 4,498 |
| pos_history_months_min | float64 | 4,498 |
| pos_history_months_max | float64 | 4,498 |
| pos_history_span_months_mean | float64 | 4,498 |
| pos_history_span_months_max | float64 | 4,498 |
| pos_contracts_with_history_gaps | int32 | 0 |
| pos_gap_months_total | int32 | 0 |
| pos_oldest_observed_month | float64 | 4,498 |
| pos_latest_observed_month | float64 | 4,498 |
| pos_dpd_observed_months | int32 | 0 |
| pos_dpd_missing_months | int32 | 0 |
| pos_dpd_observed_contracts | int32 | 0 |
| pos_dpd_contracts_with_missing | int32 | 0 |
| pos_dpd_delinquent_contracts | float32 | 4,498 |
| pos_dpd_repeated_contracts | float32 | 4,498 |
| pos_dpd_consecutive_contracts | float32 | 4,498 |
| pos_dpd_consecutive_evaluable_contracts | int32 | 0 |
| pos_dpd_ge90_contracts | float32 | 4,498 |
| pos_dpd_positive_months | float32 | 4,498 |
| pos_dpd_max | float64 | 4,498 |
| pos_dpd_mean | float64 | 4,498 |
| pos_dpd_positive_rate | float64 | 4,498 |
| pos_dpd_contract_delinquency_ratio | float64 | 4,498 |
| pos_dpd_def_observed_months | int32 | 0 |
| pos_dpd_def_missing_months | int32 | 0 |
| pos_dpd_def_observed_contracts | int32 | 0 |
| pos_dpd_def_contracts_with_missing | int32 | 0 |
| pos_dpd_def_delinquent_contracts | float32 | 4,498 |
| pos_dpd_def_repeated_contracts | float32 | 4,498 |
| pos_dpd_def_consecutive_contracts | float32 | 4,498 |
| pos_dpd_def_consecutive_evaluable_contracts | int32 | 0 |
| pos_dpd_def_ge90_contracts | float32 | 4,498 |
| pos_dpd_def_positive_months | float32 | 4,498 |
| pos_dpd_def_max | float64 | 4,498 |
| pos_dpd_def_mean | float64 | 4,498 |
| pos_dpd_def_positive_rate | float64 | 4,498 |
| pos_dpd_def_contract_delinquency_ratio | float64 | 4,498 |
| pos_recent_6m_monthly_records | int32 | 0 |
| pos_recent_6m_observed_contracts | int32 | 0 |
| pos_dpd_recent_6m_observed_months | int32 | 0 |
| pos_dpd_recent_6m_positive_months | float32 | 4,498 |
| pos_dpd_last_positive_month | float64 | 4,498 |
| pos_dpd_recent_6m_positive_rate | float64 | 4,498 |
| pos_dpd_has_recent_6m_delinquency | float32 | 4,498 |
| pos_dpd_def_recent_6m_observed_months | int32 | 0 |
| pos_dpd_def_recent_6m_positive_months | float32 | 4,498 |
| pos_dpd_def_last_positive_month | float64 | 4,498 |
| pos_dpd_def_recent_6m_positive_rate | float64 | 4,498 |
| pos_dpd_def_has_recent_6m_delinquency | float32 | 4,498 |
| pos_latest_total_instalments_mean | float32 | 4,498 |
| pos_latest_total_instalments_max | float32 | 4,498 |
| pos_latest_remaining_instalments_mean | float32 | 4,498 |
| pos_latest_remaining_instalments_max | float32 | 4,498 |
| pos_remaining_instalments_contract_mean | float32 | 4,498 |
| pos_remaining_instalments_max | float32 | 4,498 |
| pos_latest_zero_remaining_contracts | float32 | 4,498 |
| pos_total_instalments_observed_contracts | int32 | 0 |
| pos_remaining_instalments_observed_contracts | int32 | 0 |
| pos_contracts_with_missing_total_instalments | int32 | 0 |
| pos_contracts_with_missing_remaining_instalments | int32 | 0 |
| pos_latest_missing_total_instalment_contracts | int32 | 0 |
| pos_latest_missing_remaining_instalment_contracts | int32 | 0 |
| pos_latest_remaining_observed_contracts | int32 | 0 |
| pos_changing_total_instalment_contracts | float32 | 4,498 |
| pos_remaining_increase_contracts | float32 | 4,498 |
| pos_remaining_increase_evaluable_contracts | int32 | 0 |
| pos_dpd_comparable_months | int32 | 0 |
| pos_dpd_disagreement_months | float32 | 4,498 |
| pos_dpd_disagreement_contracts | float32 | 4,498 |
| pos_remaining_exceeds_total_contracts | float32 | 4,498 |
| pos_latest_completed_with_remaining_contracts | float64 | 4,498 |
| pos_latest_active_zero_remaining_contracts | float64 | 4,498 |
| pos_dpd_latest_zero_remaining_positive_contracts | float64 | 4,498 |
| pos_dpd_def_latest_zero_remaining_positive_contracts | float64 | 4,498 |
| pos_dpd_disagreement_rate | float64 | 4,498 |

### Installments — 96 features

| Feature | Dtype | No-history nulls |
| --- | --- | --- |
| installments_total_previous_contracts | int32 | 0 |
| installments_contracts_with_history | int32 | 0 |
| installments_contracts_with_timing | int32 | 0 |
| installments_contracts_without_timing | int32 | 0 |
| installments_amount_comparable_contracts | int32 | 0 |
| installments_contracts_without_amount_comparison | int32 | 0 |
| installments_raw_records | int32 | 0 |
| installments_schedule_keys | int32 | 0 |
| installments_keys_per_contract_mean | float64 | 1,922 |
| installments_keys_per_contract_max | float64 | 1,922 |
| installments_previous_contracts_without_history | int32 | 0 |
| installments_history_coverage_ratio | float64 | 0 |
| installments_timed_records | int32 | 0 |
| installments_early_records | float64 | 1,922 |
| installments_ontime_records | float64 | 1,922 |
| installments_late_records | float64 | 1,922 |
| installments_timed_keys | int32 | 0 |
| installments_keys_with_late_records | float64 | 1,922 |
| installments_contracts_with_late_records | float64 | 1,922 |
| installments_contracts_with_repeated_late_records | float64 | 1,922 |
| installments_contracts_with_repeated_late_keys | float64 | 1,922 |
| installments_early_record_rate | float64 | 1,922 |
| installments_ontime_record_rate | float64 | 1,922 |
| installments_late_record_rate | float64 | 1,922 |
| installments_late_key_rate | float64 | 1,922 |
| installments_delay_max | float32 | 1,922 |
| installments_late_delay_max | float64 | 1,922 |
| installments_delay_ge30_records | float64 | 1,922 |
| installments_delay_ge60_records | float64 | 1,922 |
| installments_delay_ge90_records | float64 | 1,922 |
| installments_contracts_with_ge90_delay | float64 | 1,922 |
| installments_delay_mean | float64 | 1,922 |
| installments_late_delay_mean | float64 | 1,922 |
| installments_recorded_payment_amount_total | float64 | 1,922 |
| installments_version_key_scheduled_amount_total | float64 | 1,922 |
| installments_coherent_scheduled_keys | int32 | 0 |
| installments_amount_eligible_keys | int32 | 0 |
| installments_eligible_key_payment_amount_total | float64 | 1,922 |
| installments_eligible_key_scheduled_amount_total | float64 | 1,922 |
| installments_payment_below_schedule_keys | float64 | 1,922 |
| installments_payment_equal_schedule_keys | float64 | 1,922 |
| installments_payment_above_schedule_keys | float64 | 1,922 |
| installments_payment_covering_schedule_keys | float64 | 1,922 |
| installments_recorded_payment_amount_mean | float64 | 1,922 |
| installments_version_key_scheduled_amount_mean | float64 | 1,922 |
| installments_payment_to_scheduled_amount_ratio | float64 | 1,922 |
| installments_amount_coverage_rate | float64 | 1,922 |
| installments_coverage_timed_keys | int32 | 0 |
| installments_coverage_early_keys | float64 | 1,922 |
| installments_coverage_ontime_keys | float64 | 1,922 |
| installments_coverage_late_keys | float64 | 1,922 |
| installments_late_coverage_delay_max | float32 | 1,922 |
| installments_coverage_late_rate | float64 | 1,922 |
| installments_late_coverage_delay_mean | float64 | 1,922 |
| installments_multiple_entry_keys | int32 | 0 |
| installments_additional_payment_entries | int32 | 0 |
| installments_contracts_with_multiple_entries | int32 | 0 |
| installments_versions_per_contract_mean | float64 | 1,922 |
| installments_versions_per_contract_max | float64 | 1,922 |
| installments_contracts_with_multiple_versions | int32 | 0 |
| installments_version0_contracts | int32 | 0 |
| installments_version0_keys | int32 | 0 |
| installments_version0_timed_records | int32 | 0 |
| installments_version0_late_records | float64 | 1,922 |
| installments_positive_version_timed_records | int32 | 0 |
| installments_positive_version_late_records | float64 | 1,922 |
| installments_cross_version_number_pairs | int32 | 0 |
| installments_comparable_revision_pairs | int32 | 0 |
| installments_changed_amount_revision_pairs | int32 | 0 |
| installments_changed_due_revision_pairs | int32 | 0 |
| installments_contracts_with_revision_pairs | int32 | 0 |
| installments_multiple_entry_key_rate | float64 | 1,922 |
| installments_version0_late_record_rate | float64 | 1,922 |
| installments_positive_version_late_record_rate | float64 | 1,922 |
| installments_oldest_payment_day | float32 | 1,922 |
| installments_latest_payment_day | float32 | 1,922 |
| installments_latest_late_payment_day | float32 | 1,922 |
| installments_oldest_scheduled_day | float32 | 1,922 |
| installments_latest_scheduled_day | float32 | 1,922 |
| installments_payment_day_mean | float64 | 1,922 |
| installments_days_since_latest_late_payment | float32 | 1,922 |
| installments_scheduled_history_span_days | float32 | 1,922 |
| installments_missing_payment_date_records | int32 | 0 |
| installments_missing_scheduled_date_records | int32 | 0 |
| installments_missing_timing_records | int32 | 0 |
| installments_missing_payment_amount_records | int32 | 0 |
| installments_missing_scheduled_amount_records | int32 | 0 |
| installments_observed_payment_amount_records | int32 | 0 |
| installments_contracts_with_missing_payment_dates | int32 | 0 |
| installments_contracts_with_missing_payment_amounts | int32 | 0 |
| installments_contracts_without_payment_amounts | int32 | 0 |
| installments_incomplete_payment_keys | int32 | 0 |
| installments_incomplete_schedule_keys | int32 | 0 |
| installments_inconsistent_schedule_keys | int32 | 0 |
| installments_zero_scheduled_amount_keys | int32 | 0 |
| installments_missing_key_records | int32 | 0 |

### Credit Card — 135 features

| Feature | Dtype | No-history nulls |
| --- | --- | --- |
| credit_card_total_previous_contracts | int32 | 0 |
| credit_card_contracts_with_history | int32 | 0 |
| credit_card_latest_active_contracts | int32 | 0 |
| credit_card_latest_completed_contracts | int32 | 0 |
| credit_card_latest_other_status_contracts | int32 | 0 |
| credit_card_latest_unknown_status_contracts | int32 | 0 |
| credit_card_status_change_evaluable_contracts | int32 | 0 |
| credit_card_contracts_with_status_changes | float32 | 246,410 |
| credit_card_previous_contracts_without_card_history | int32 | 0 |
| credit_card_active_contract_ratio | float64 | 246,410 |
| credit_card_completed_contract_ratio | float64 | 246,410 |
| credit_card_history_coverage_ratio | float64 | 0 |
| credit_card_monthly_records | int32 | 0 |
| credit_card_history_months_mean | float64 | 246,410 |
| credit_card_history_months_min | float64 | 246,410 |
| credit_card_history_months_max | float64 | 246,410 |
| credit_card_history_span_months_mean | float64 | 246,410 |
| credit_card_history_span_months_max | float64 | 246,410 |
| credit_card_contracts_with_history_gaps | int32 | 0 |
| credit_card_gap_months | int32 | 0 |
| credit_card_latest_observed_month | float64 | 246,410 |
| credit_card_oldest_observed_month | float64 | 246,410 |
| credit_card_utilization_observed_months | int32 | 0 |
| credit_card_utilization_observed_contracts | int32 | 0 |
| credit_card_utilization_min | float64 | 246,410 |
| credit_card_utilization_max | float64 | 246,410 |
| credit_card_ever_over100_utilization_contracts | float32 | 246,410 |
| credit_card_latest_utilization_observed_contracts | int32 | 0 |
| credit_card_latest_utilization_mean | float64 | 246,410 |
| credit_card_latest_utilization_max | float64 | 246,410 |
| credit_card_latest_over100_utilization_contracts | float32 | 246,410 |
| credit_card_utilization_mean | float64 | 246,410 |
| credit_card_utilization_ge80_rate | float64 | 246,410 |
| credit_card_utilization_ge90_rate | float64 | 246,410 |
| credit_card_utilization_ge100_rate | float64 | 246,410 |
| credit_card_balance_mean | float64 | 246,410 |
| credit_card_latest_balance_total | float64 | 246,410 |
| credit_card_credit_limit_mean | float64 | 246,410 |
| credit_card_latest_credit_limit_total | float64 | 246,410 |
| credit_card_principal_receivable_mean | float64 | 246,410 |
| credit_card_latest_principal_receivable_total | float64 | 246,410 |
| credit_card_recivable_mean | float64 | 246,410 |
| credit_card_latest_recivable_total | float64 | 246,410 |
| credit_card_total_receivable_mean | float64 | 246,410 |
| credit_card_latest_total_receivable_total | float64 | 246,410 |
| credit_card_balance_max | float64 | 246,410 |
| credit_card_credit_limit_max | float64 | 246,410 |
| credit_card_current_payment_observed_months | int32 | 0 |
| credit_card_current_payment_mean | float64 | 246,410 |
| credit_card_current_payment_max | float64 | 246,410 |
| credit_card_total_payment_observed_months | int32 | 0 |
| credit_card_total_payment_mean | float64 | 246,410 |
| credit_card_total_payment_max | float64 | 246,410 |
| credit_card_minimum_instalment_observed_months | int32 | 0 |
| credit_card_minimum_instalment_mean | float64 | 246,410 |
| credit_card_minimum_instalment_max | float64 | 246,410 |
| credit_card_drawings_observed_months | int32 | 0 |
| credit_card_drawings_mean | float64 | 246,410 |
| credit_card_drawings_max | float64 | 246,410 |
| credit_card_drawings_amount_total | float64 | 246,410 |
| credit_card_zero_drawing_rate | float64 | 246,410 |
| credit_card_total_payment_to_balance_observed_months | int32 | 0 |
| credit_card_total_payment_to_balance_mean | float64 | 246,410 |
| credit_card_total_payment_to_minimum_observed_months | int32 | 0 |
| credit_card_total_payment_to_minimum_mean | float64 | 246,410 |
| credit_card_latest_current_payment_mean | float64 | 246,410 |
| credit_card_latest_total_payment_mean | float64 | 246,410 |
| credit_card_dpd_observed_months | int32 | 0 |
| credit_card_dpd_missing_months | int32 | 0 |
| credit_card_dpd_observed_contracts | int32 | 0 |
| credit_card_dpd_positive_months | float32 | 246,410 |
| credit_card_dpd_delinquent_contracts | float32 | 246,410 |
| credit_card_dpd_repeated_contracts | float32 | 246,410 |
| credit_card_dpd_consecutive_contracts | float32 | 246,410 |
| credit_card_dpd_consecutive_evaluable_contracts | int32 | 0 |
| credit_card_dpd_ge90_contracts | float32 | 246,410 |
| credit_card_dpd_max | float64 | 246,410 |
| credit_card_dpd_latest_mean | float64 | 246,410 |
| credit_card_dpd_latest_max | float64 | 246,410 |
| credit_card_dpd_latest_observed_contracts | int32 | 0 |
| credit_card_dpd_mean | float64 | 246,410 |
| credit_card_dpd_positive_rate | float64 | 246,410 |
| credit_card_dpd_contract_delinquency_ratio | float64 | 246,410 |
| credit_card_dpd_def_observed_months | int32 | 0 |
| credit_card_dpd_def_missing_months | int32 | 0 |
| credit_card_dpd_def_observed_contracts | int32 | 0 |
| credit_card_dpd_def_positive_months | float32 | 246,410 |
| credit_card_dpd_def_delinquent_contracts | float32 | 246,410 |
| credit_card_dpd_def_repeated_contracts | float32 | 246,410 |
| credit_card_dpd_def_consecutive_contracts | float32 | 246,410 |
| credit_card_dpd_def_consecutive_evaluable_contracts | int32 | 0 |
| credit_card_dpd_def_ge90_contracts | float32 | 246,410 |
| credit_card_dpd_def_max | float64 | 246,410 |
| credit_card_dpd_def_latest_mean | float64 | 246,410 |
| credit_card_dpd_def_latest_max | float64 | 246,410 |
| credit_card_dpd_def_latest_observed_contracts | int32 | 0 |
| credit_card_dpd_def_mean | float64 | 246,410 |
| credit_card_dpd_def_positive_rate | float64 | 246,410 |
| credit_card_dpd_def_contract_delinquency_ratio | float64 | 246,410 |
| credit_card_recent_6m_monthly_records | int32 | 0 |
| credit_card_recent_6m_observed_contracts | int32 | 0 |
| credit_card_recent_6m_utilization_observed_months | int32 | 0 |
| credit_card_recent_6m_utilization_mean | float64 | 246,410 |
| credit_card_recent_6m_utilization_ge90_rate | float64 | 246,410 |
| credit_card_has_recent_6m_high_utilization | float32 | 246,410 |
| credit_card_dpd_recent_6m_observed_months | int32 | 0 |
| credit_card_dpd_recent_6m_positive_months | float64 | 246,410 |
| credit_card_dpd_recent_6m_positive_rate | float64 | 246,410 |
| credit_card_dpd_has_recent_6m_delinquency | float32 | 246,410 |
| credit_card_dpd_last_positive_month | float64 | 246,410 |
| credit_card_dpd_def_recent_6m_observed_months | int32 | 0 |
| credit_card_dpd_def_recent_6m_positive_months | float64 | 246,410 |
| credit_card_dpd_def_recent_6m_positive_rate | float64 | 246,410 |
| credit_card_dpd_def_has_recent_6m_delinquency | float32 | 246,410 |
| credit_card_dpd_def_last_positive_month | float64 | 246,410 |
| credit_card_contracts_with_missing_current_payments | int32 | 0 |
| credit_card_contracts_without_current_payment_amounts | int32 | 0 |
| credit_card_latest_missing_current_payment_contracts | int32 | 0 |
| credit_card_drawing_category_observed_months | int32 | 0 |
| credit_card_missing_drawing_category_months | int32 | 0 |
| credit_card_contracts_without_complete_drawing_categories | int32 | 0 |
| credit_card_negative_balance_months | int32 | 0 |
| credit_card_negative_balance_contracts | int32 | 0 |
| credit_card_negative_receivable_months | int32 | 0 |
| credit_card_negative_receivable_contracts | int32 | 0 |
| credit_card_zero_credit_limit_months | int32 | 0 |
| credit_card_zero_credit_limit_contracts | int32 | 0 |
| credit_card_dpd_comparable_months | int32 | 0 |
| credit_card_dpd_disagreement_months | float64 | 246,410 |
| credit_card_dpd_disagreement_contracts | float32 | 246,410 |
| credit_card_current_payment_missing_months | int32 | 0 |
| credit_card_total_payment_missing_months | int32 | 0 |
| credit_card_minimum_instalment_missing_months | int32 | 0 |
| credit_card_drawings_missing_months | int32 | 0 |
| credit_card_utilization_missing_months | int32 | 0 |

## Appendix B. Complete feature missingness, descending

All feature columns are included, including those with zero missingness. Percentages are sorted on unrounded values; ties are sorted by dataset and feature name.

| Dataset | Feature | Missing Count | Missing Percentage |
| --- | --- | --- | --- |
| Credit Card | credit_card_dpd_def_last_positive_month | 327,703 | 96.7083 |
| Credit Card | credit_card_dpd_last_positive_month | 324,643 | 95.8053 |
| POS/CASH | pos_dpd_def_last_positive_month | 294,887 | 87.0240 |
| POS/CASH | pos_dpd_last_positive_month | 279,610 | 82.5156 |
| Credit Card | credit_card_total_payment_to_minimum_mean | 278,778 | 82.2701 |
| Credit Card | credit_card_total_payment_to_balance_mean | 278,392 | 82.1562 |
| Credit Card | credit_card_current_payment_max | 276,916 | 81.7206 |
| Credit Card | credit_card_current_payment_mean | 276,916 | 81.7206 |
| Credit Card | credit_card_latest_current_payment_mean | 276,916 | 81.7206 |
| Installments | installments_version0_late_record_rate | 276,584 | 81.6226 |
| Installments | installments_version0_late_records | 276,584 | 81.6226 |
| Credit Card | credit_card_latest_over100_utilization_contracts | 265,018 | 78.2094 |
| Credit Card | credit_card_latest_utilization_max | 265,018 | 78.2094 |
| Credit Card | credit_card_latest_utilization_mean | 265,018 | 78.2094 |
| Credit Card | credit_card_has_recent_6m_high_utilization | 262,878 | 77.5779 |
| Credit Card | credit_card_recent_6m_utilization_ge90_rate | 262,878 | 77.5779 |
| Credit Card | credit_card_recent_6m_utilization_mean | 262,878 | 77.5779 |
| Credit Card | credit_card_ever_over100_utilization_contracts | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_ge100_rate | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_ge80_rate | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_ge90_rate | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_max | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_mean | 247,530 | 73.0485 |
| Credit Card | credit_card_utilization_min | 247,530 | 73.0485 |
| Credit Card | credit_card_contracts_with_status_changes | 247,104 | 72.9228 |
| Credit Card | credit_card_dpd_consecutive_contracts | 247,104 | 72.9228 |
| Credit Card | credit_card_dpd_def_consecutive_contracts | 247,104 | 72.9228 |
| Credit Card | credit_card_dpd_def_has_recent_6m_delinquency | 246,412 | 72.7186 |
| Credit Card | credit_card_dpd_def_recent_6m_positive_months | 246,412 | 72.7186 |
| Credit Card | credit_card_dpd_def_recent_6m_positive_rate | 246,412 | 72.7186 |
| Credit Card | credit_card_dpd_has_recent_6m_delinquency | 246,412 | 72.7186 |
| Credit Card | credit_card_dpd_recent_6m_positive_months | 246,412 | 72.7186 |
| Credit Card | credit_card_dpd_recent_6m_positive_rate | 246,412 | 72.7186 |
| Credit Card | credit_card_active_contract_ratio | 246,410 | 72.7180 |
| Credit Card | credit_card_balance_max | 246,410 | 72.7180 |
| Credit Card | credit_card_balance_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_completed_contract_ratio | 246,410 | 72.7180 |
| Credit Card | credit_card_credit_limit_max | 246,410 | 72.7180 |
| Credit Card | credit_card_credit_limit_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_contract_delinquency_ratio | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_contract_delinquency_ratio | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_delinquent_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_ge90_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_latest_max | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_latest_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_max | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_positive_months | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_positive_rate | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_def_repeated_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_delinquent_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_disagreement_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_disagreement_months | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_ge90_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_latest_max | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_latest_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_max | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_positive_months | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_positive_rate | 246,410 | 72.7180 |
| Credit Card | credit_card_dpd_repeated_contracts | 246,410 | 72.7180 |
| Credit Card | credit_card_drawings_amount_total | 246,410 | 72.7180 |
| Credit Card | credit_card_drawings_max | 246,410 | 72.7180 |
| Credit Card | credit_card_drawings_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_history_months_max | 246,410 | 72.7180 |
| Credit Card | credit_card_history_months_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_history_months_min | 246,410 | 72.7180 |
| Credit Card | credit_card_history_span_months_max | 246,410 | 72.7180 |
| Credit Card | credit_card_history_span_months_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_balance_total | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_credit_limit_total | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_observed_month | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_principal_receivable_total | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_recivable_total | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_total_payment_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_latest_total_receivable_total | 246,410 | 72.7180 |
| Credit Card | credit_card_minimum_instalment_max | 246,410 | 72.7180 |
| Credit Card | credit_card_minimum_instalment_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_oldest_observed_month | 246,410 | 72.7180 |
| Credit Card | credit_card_principal_receivable_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_recivable_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_total_payment_max | 246,410 | 72.7180 |
| Credit Card | credit_card_total_payment_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_total_receivable_mean | 246,410 | 72.7180 |
| Credit Card | credit_card_zero_drawing_rate | 246,410 | 72.7180 |
| Installments | installments_late_coverage_delay_max | 164,761 | 48.6226 |
| Installments | installments_late_coverage_delay_mean | 164,761 | 48.6226 |
| Installments | installments_days_since_latest_late_payment | 164,708 | 48.6069 |
| Installments | installments_late_delay_max | 164,708 | 48.6069 |
| Installments | installments_late_delay_mean | 164,708 | 48.6069 |
| Installments | installments_latest_late_payment_day | 164,708 | 48.6069 |
| POS/CASH | pos_dpd_def_has_recent_6m_delinquency | 147,135 | 43.4210 |
| POS/CASH | pos_dpd_def_recent_6m_positive_months | 147,135 | 43.4210 |
| POS/CASH | pos_dpd_def_recent_6m_positive_rate | 147,135 | 43.4210 |
| POS/CASH | pos_dpd_has_recent_6m_delinquency | 147,135 | 43.4210 |
| POS/CASH | pos_dpd_recent_6m_positive_months | 147,135 | 43.4210 |
| POS/CASH | pos_dpd_recent_6m_positive_rate | 147,135 | 43.4210 |
| Previous Application | prev_down_payment_max | 20,104 | 5.9329 |
| Previous Application | prev_down_payment_mean | 20,104 | 5.9329 |
| Previous Application | prev_down_payment_total | 20,104 | 5.9329 |
| Bureau | bureau_debt_max | 8,372 | 2.7376 |
| Bureau | bureau_debt_mean | 8,372 | 2.7376 |
| POS/CASH | pos_remaining_increase_contracts | 4,828 | 1.4248 |
| POS/CASH | pos_dpd_consecutive_contracts | 4,801 | 1.4168 |
| POS/CASH | pos_dpd_def_consecutive_contracts | 4,801 | 1.4168 |
| Previous Application | prev_planned_duration_days_max | 4,764 | 1.4059 |
| Previous Application | prev_planned_duration_days_mean | 4,764 | 1.4059 |
| Installments | installments_positive_version_late_record_rate | 4,606 | 1.3593 |
| Installments | installments_positive_version_late_records | 4,606 | 1.3593 |
| POS/CASH | pos_changing_total_instalment_contracts | 4,535 | 1.3383 |
| POS/CASH | pos_latest_remaining_instalments_max | 4,535 | 1.3383 |
| POS/CASH | pos_latest_remaining_instalments_mean | 4,535 | 1.3383 |
| POS/CASH | pos_latest_total_instalments_max | 4,535 | 1.3383 |
| POS/CASH | pos_latest_total_instalments_mean | 4,535 | 1.3383 |
| POS/CASH | pos_latest_zero_remaining_contracts | 4,535 | 1.3383 |
| POS/CASH | pos_remaining_exceeds_total_contracts | 4,535 | 1.3383 |
| POS/CASH | pos_remaining_instalments_contract_mean | 4,535 | 1.3383 |
| POS/CASH | pos_remaining_instalments_max | 4,535 | 1.3383 |
| POS/CASH | pos_active_contract_ratio | 4,498 | 1.3274 |
| POS/CASH | pos_completed_contract_ratio | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_contract_delinquency_ratio | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_contract_delinquency_ratio | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_delinquent_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_ge90_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_latest_zero_remaining_positive_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_max | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_mean | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_positive_months | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_positive_rate | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_def_repeated_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_delinquent_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_disagreement_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_disagreement_months | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_disagreement_rate | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_ge90_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_latest_zero_remaining_positive_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_max | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_mean | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_positive_months | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_positive_rate | 4,498 | 1.3274 |
| POS/CASH | pos_dpd_repeated_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_history_months_max | 4,498 | 1.3274 |
| POS/CASH | pos_history_months_mean | 4,498 | 1.3274 |
| POS/CASH | pos_history_months_min | 4,498 | 1.3274 |
| POS/CASH | pos_history_span_months_max | 4,498 | 1.3274 |
| POS/CASH | pos_history_span_months_mean | 4,498 | 1.3274 |
| POS/CASH | pos_latest_active_zero_remaining_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_latest_completed_with_remaining_contracts | 4,498 | 1.3274 |
| POS/CASH | pos_latest_observed_month | 4,498 | 1.3274 |
| POS/CASH | pos_oldest_observed_month | 4,498 | 1.3274 |
| Previous Application | prev_cnt_payment_max | 2,696 | 0.7956 |
| Previous Application | prev_cnt_payment_mean | 2,696 | 0.7956 |
| Installments | installments_amount_coverage_rate | 1,933 | 0.5704 |
| Installments | installments_coverage_early_keys | 1,933 | 0.5704 |
| Installments | installments_coverage_late_keys | 1,933 | 0.5704 |
| Installments | installments_coverage_late_rate | 1,933 | 0.5704 |
| Installments | installments_coverage_ontime_keys | 1,933 | 0.5704 |
| Installments | installments_eligible_key_payment_amount_total | 1,933 | 0.5704 |
| Installments | installments_eligible_key_scheduled_amount_total | 1,933 | 0.5704 |
| Installments | installments_payment_above_schedule_keys | 1,933 | 0.5704 |
| Installments | installments_payment_below_schedule_keys | 1,933 | 0.5704 |
| Installments | installments_payment_covering_schedule_keys | 1,933 | 0.5704 |
| Installments | installments_payment_equal_schedule_keys | 1,933 | 0.5704 |
| Installments | installments_payment_to_scheduled_amount_ratio | 1,933 | 0.5704 |
| Installments | installments_contracts_with_ge90_delay | 1,931 | 0.5699 |
| Installments | installments_contracts_with_late_records | 1,931 | 0.5699 |
| Installments | installments_contracts_with_repeated_late_keys | 1,931 | 0.5699 |
| Installments | installments_contracts_with_repeated_late_records | 1,931 | 0.5699 |
| Installments | installments_delay_ge30_records | 1,931 | 0.5699 |
| Installments | installments_delay_ge60_records | 1,931 | 0.5699 |
| Installments | installments_delay_ge90_records | 1,931 | 0.5699 |
| Installments | installments_delay_max | 1,931 | 0.5699 |
| Installments | installments_delay_mean | 1,931 | 0.5699 |
| Installments | installments_early_record_rate | 1,931 | 0.5699 |
| Installments | installments_early_records | 1,931 | 0.5699 |
| Installments | installments_keys_with_late_records | 1,931 | 0.5699 |
| Installments | installments_late_key_rate | 1,931 | 0.5699 |
| Installments | installments_late_record_rate | 1,931 | 0.5699 |
| Installments | installments_late_records | 1,931 | 0.5699 |
| Installments | installments_latest_payment_day | 1,931 | 0.5699 |
| Installments | installments_oldest_payment_day | 1,931 | 0.5699 |
| Installments | installments_ontime_record_rate | 1,931 | 0.5699 |
| Installments | installments_ontime_records | 1,931 | 0.5699 |
| Installments | installments_payment_day_mean | 1,931 | 0.5699 |
| Installments | installments_recorded_payment_amount_mean | 1,931 | 0.5699 |
| Installments | installments_recorded_payment_amount_total | 1,931 | 0.5699 |
| Installments | installments_keys_per_contract_max | 1,922 | 0.5672 |
| Installments | installments_keys_per_contract_mean | 1,922 | 0.5672 |
| Installments | installments_latest_scheduled_day | 1,922 | 0.5672 |
| Installments | installments_multiple_entry_key_rate | 1,922 | 0.5672 |
| Installments | installments_oldest_scheduled_day | 1,922 | 0.5672 |
| Installments | installments_scheduled_history_span_days | 1,922 | 0.5672 |
| Installments | installments_version_key_scheduled_amount_mean | 1,922 | 0.5672 |
| Installments | installments_version_key_scheduled_amount_total | 1,922 | 0.5672 |
| Installments | installments_versions_per_contract_max | 1,922 | 0.5672 |
| Installments | installments_versions_per_contract_mean | 1,922 | 0.5672 |
| Previous Application | prev_goods_price_max | 1,064 | 0.3140 |
| Previous Application | prev_goods_price_mean | 1,064 | 0.3140 |
| Previous Application | prev_annuity_max | 480 | 0.1417 |
| Previous Application | prev_annuity_mean | 480 | 0.1417 |
| Bureau | bureau_credit_sum_max | 2 | 0.0007 |
| Bureau | bureau_credit_sum_mean | 2 | 0.0007 |
| Bureau | bureau_accounts_with_debt | 0 | 0.0000 |
| Bureau | bureau_active_accounts | 0 | 0.0000 |
| Bureau | bureau_closed_accounts | 0 | 0.0000 |
| Bureau | bureau_consumer_credit_accounts | 0 | 0.0000 |
| Bureau | bureau_credit_card_accounts | 0 | 0.0000 |
| Bureau | bureau_credit_sum_total | 0 | 0.0000 |
| Bureau | bureau_days_credit_max | 0 | 0.0000 |
| Bureau | bureau_days_credit_mean | 0 | 0.0000 |
| Bureau | bureau_days_credit_min | 0 | 0.0000 |
| Bureau | bureau_days_overdue_max | 0 | 0.0000 |
| Bureau | bureau_days_overdue_mean | 0 | 0.0000 |
| Bureau | bureau_debt_total | 0 | 0.0000 |
| Bureau | bureau_missing_debt_accounts | 0 | 0.0000 |
| Bureau | bureau_overdue_accounts | 0 | 0.0000 |
| Bureau | bureau_overdue_max | 0 | 0.0000 |
| Bureau | bureau_overdue_total | 0 | 0.0000 |
| Bureau | bureau_total_accounts | 0 | 0.0000 |
| Credit Card | credit_card_contracts_with_history | 0 | 0.0000 |
| Credit Card | credit_card_contracts_with_history_gaps | 0 | 0.0000 |
| Credit Card | credit_card_contracts_with_missing_current_payments | 0 | 0.0000 |
| Credit Card | credit_card_contracts_without_complete_drawing_categories | 0 | 0.0000 |
| Credit Card | credit_card_contracts_without_current_payment_amounts | 0 | 0.0000 |
| Credit Card | credit_card_current_payment_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_current_payment_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_comparable_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_consecutive_evaluable_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_consecutive_evaluable_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_latest_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_def_recent_6m_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_latest_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_dpd_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_dpd_recent_6m_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_drawing_category_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_drawings_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_drawings_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_gap_months | 0 | 0.0000 |
| Credit Card | credit_card_history_coverage_ratio | 0 | 0.0000 |
| Credit Card | credit_card_latest_active_contracts | 0 | 0.0000 |
| Credit Card | credit_card_latest_completed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_latest_missing_current_payment_contracts | 0 | 0.0000 |
| Credit Card | credit_card_latest_other_status_contracts | 0 | 0.0000 |
| Credit Card | credit_card_latest_unknown_status_contracts | 0 | 0.0000 |
| Credit Card | credit_card_latest_utilization_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_minimum_instalment_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_minimum_instalment_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_missing_drawing_category_months | 0 | 0.0000 |
| Credit Card | credit_card_monthly_records | 0 | 0.0000 |
| Credit Card | credit_card_negative_balance_contracts | 0 | 0.0000 |
| Credit Card | credit_card_negative_balance_months | 0 | 0.0000 |
| Credit Card | credit_card_negative_receivable_contracts | 0 | 0.0000 |
| Credit Card | credit_card_negative_receivable_months | 0 | 0.0000 |
| Credit Card | credit_card_previous_contracts_without_card_history | 0 | 0.0000 |
| Credit Card | credit_card_recent_6m_monthly_records | 0 | 0.0000 |
| Credit Card | credit_card_recent_6m_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_recent_6m_utilization_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_status_change_evaluable_contracts | 0 | 0.0000 |
| Credit Card | credit_card_total_payment_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_total_payment_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_total_payment_to_balance_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_total_payment_to_minimum_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_total_previous_contracts | 0 | 0.0000 |
| Credit Card | credit_card_utilization_missing_months | 0 | 0.0000 |
| Credit Card | credit_card_utilization_observed_contracts | 0 | 0.0000 |
| Credit Card | credit_card_utilization_observed_months | 0 | 0.0000 |
| Credit Card | credit_card_zero_credit_limit_contracts | 0 | 0.0000 |
| Credit Card | credit_card_zero_credit_limit_months | 0 | 0.0000 |
| Installments | installments_additional_payment_entries | 0 | 0.0000 |
| Installments | installments_amount_comparable_contracts | 0 | 0.0000 |
| Installments | installments_amount_eligible_keys | 0 | 0.0000 |
| Installments | installments_changed_amount_revision_pairs | 0 | 0.0000 |
| Installments | installments_changed_due_revision_pairs | 0 | 0.0000 |
| Installments | installments_coherent_scheduled_keys | 0 | 0.0000 |
| Installments | installments_comparable_revision_pairs | 0 | 0.0000 |
| Installments | installments_contracts_with_history | 0 | 0.0000 |
| Installments | installments_contracts_with_missing_payment_amounts | 0 | 0.0000 |
| Installments | installments_contracts_with_missing_payment_dates | 0 | 0.0000 |
| Installments | installments_contracts_with_multiple_entries | 0 | 0.0000 |
| Installments | installments_contracts_with_multiple_versions | 0 | 0.0000 |
| Installments | installments_contracts_with_revision_pairs | 0 | 0.0000 |
| Installments | installments_contracts_with_timing | 0 | 0.0000 |
| Installments | installments_contracts_without_amount_comparison | 0 | 0.0000 |
| Installments | installments_contracts_without_payment_amounts | 0 | 0.0000 |
| Installments | installments_contracts_without_timing | 0 | 0.0000 |
| Installments | installments_coverage_timed_keys | 0 | 0.0000 |
| Installments | installments_cross_version_number_pairs | 0 | 0.0000 |
| Installments | installments_history_coverage_ratio | 0 | 0.0000 |
| Installments | installments_incomplete_payment_keys | 0 | 0.0000 |
| Installments | installments_incomplete_schedule_keys | 0 | 0.0000 |
| Installments | installments_inconsistent_schedule_keys | 0 | 0.0000 |
| Installments | installments_missing_key_records | 0 | 0.0000 |
| Installments | installments_missing_payment_amount_records | 0 | 0.0000 |
| Installments | installments_missing_payment_date_records | 0 | 0.0000 |
| Installments | installments_missing_scheduled_amount_records | 0 | 0.0000 |
| Installments | installments_missing_scheduled_date_records | 0 | 0.0000 |
| Installments | installments_missing_timing_records | 0 | 0.0000 |
| Installments | installments_multiple_entry_keys | 0 | 0.0000 |
| Installments | installments_observed_payment_amount_records | 0 | 0.0000 |
| Installments | installments_positive_version_timed_records | 0 | 0.0000 |
| Installments | installments_previous_contracts_without_history | 0 | 0.0000 |
| Installments | installments_raw_records | 0 | 0.0000 |
| Installments | installments_schedule_keys | 0 | 0.0000 |
| Installments | installments_timed_keys | 0 | 0.0000 |
| Installments | installments_timed_records | 0 | 0.0000 |
| Installments | installments_total_previous_contracts | 0 | 0.0000 |
| Installments | installments_version0_contracts | 0 | 0.0000 |
| Installments | installments_version0_keys | 0 | 0.0000 |
| Installments | installments_version0_timed_records | 0 | 0.0000 |
| Installments | installments_zero_scheduled_amount_keys | 0 | 0.0000 |
| POS/CASH | pos_contracts_with_history | 0 | 0.0000 |
| POS/CASH | pos_contracts_with_history_gaps | 0 | 0.0000 |
| POS/CASH | pos_contracts_with_missing_remaining_instalments | 0 | 0.0000 |
| POS/CASH | pos_contracts_with_missing_total_instalments | 0 | 0.0000 |
| POS/CASH | pos_dpd_comparable_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_consecutive_evaluable_contracts | 0 | 0.0000 |
| POS/CASH | pos_dpd_contracts_with_missing | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_consecutive_evaluable_contracts | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_contracts_with_missing | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_missing_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_observed_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_def_recent_6m_observed_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_missing_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_dpd_observed_months | 0 | 0.0000 |
| POS/CASH | pos_dpd_recent_6m_observed_months | 0 | 0.0000 |
| POS/CASH | pos_gap_months_total | 0 | 0.0000 |
| POS/CASH | pos_history_coverage_ratio | 0 | 0.0000 |
| POS/CASH | pos_latest_active_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_amortized_debt_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_completed_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_demand_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_missing_remaining_instalment_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_missing_total_instalment_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_other_status_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_remaining_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_latest_unknown_status_contracts | 0 | 0.0000 |
| POS/CASH | pos_monthly_records_total | 0 | 0.0000 |
| POS/CASH | pos_previous_contracts_without_pos_history | 0 | 0.0000 |
| POS/CASH | pos_recent_6m_monthly_records | 0 | 0.0000 |
| POS/CASH | pos_recent_6m_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_remaining_increase_evaluable_contracts | 0 | 0.0000 |
| POS/CASH | pos_remaining_instalments_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_total_instalments_observed_contracts | 0 | 0.0000 |
| POS/CASH | pos_total_previous_contracts | 0 | 0.0000 |
| Previous Application | prev_application_amount_max | 0 | 0.0000 |
| Previous Application | prev_application_amount_mean | 0 | 0.0000 |
| Previous Application | prev_application_amount_min | 0 | 0.0000 |
| Previous Application | prev_application_amount_total | 0 | 0.0000 |
| Previous Application | prev_applications_with_down_payment | 0 | 0.0000 |
| Previous Application | prev_approval_rate | 0 | 0.0000 |
| Previous Application | prev_approved_applications | 0 | 0.0000 |
| Previous Application | prev_canceled_applications | 0 | 0.0000 |
| Previous Application | prev_cancellation_rate | 0 | 0.0000 |
| Previous Application | prev_cash_or_revolving_missing_down_payment_applications | 0 | 0.0000 |
| Previous Application | prev_consumer_missing_down_payment_applications | 0 | 0.0000 |
| Previous Application | prev_credit_amount_max | 0 | 0.0000 |
| Previous Application | prev_credit_amount_mean | 0 | 0.0000 |
| Previous Application | prev_credit_amount_total | 0 | 0.0000 |
| Previous Application | prev_days_decision_max | 0 | 0.0000 |
| Previous Application | prev_days_decision_min | 0 | 0.0000 |
| Previous Application | prev_days_since_decision_mean | 0 | 0.0000 |
| Previous Application | prev_distinct_channels | 0 | 0.0000 |
| Previous Application | prev_distinct_contract_types | 0 | 0.0000 |
| Previous Application | prev_distinct_goods_categories | 0 | 0.0000 |
| Previous Application | prev_distinct_payment_types | 0 | 0.0000 |
| Previous Application | prev_distinct_product_combinations | 0 | 0.0000 |
| Previous Application | prev_first_due_missing_applications | 0 | 0.0000 |
| Previous Application | prev_first_due_sentinel_applications | 0 | 0.0000 |
| Previous Application | prev_installment_missing_term_applications | 0 | 0.0000 |
| Previous Application | prev_missing_annuity_applications | 0 | 0.0000 |
| Previous Application | prev_missing_cnt_payment_applications | 0 | 0.0000 |
| Previous Application | prev_missing_down_payment_applications | 0 | 0.0000 |
| Previous Application | prev_missing_goods_price_applications | 0 | 0.0000 |
| Previous Application | prev_negative_down_payment_applications | 0 | 0.0000 |
| Previous Application | prev_nonfinal_contract_applications | 0 | 0.0000 |
| Previous Application | prev_original_last_due_missing_applications | 0 | 0.0000 |
| Previous Application | prev_original_last_due_sentinel_applications | 0 | 0.0000 |
| Previous Application | prev_refusal_rate | 0 | 0.0000 |
| Previous Application | prev_refused_applications | 0 | 0.0000 |
| Previous Application | prev_revolving_without_fixed_term_applications | 0 | 0.0000 |
| Previous Application | prev_total_applications | 0 | 0.0000 |
| Previous Application | prev_unknown_contract_type_applications | 0 | 0.0000 |
| Previous Application | prev_unknown_payment_type_applications | 0 | 0.0000 |
| Previous Application | prev_unused_offer_applications | 0 | 0.0000 |
| Previous Application | prev_unused_offer_rate | 0 | 0.0000 |
| Previous Application | prev_valid_planned_duration_applications | 0 | 0.0000 |
| Previous Application | prev_valid_term_applications | 0 | 0.0000 |
| Previous Application | prev_zero_down_payment_applications | 0 | 0.0000 |
