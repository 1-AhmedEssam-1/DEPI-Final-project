# Data definitions

Use `data/raw/home-credit-default-risk/HomeCredit_columns_description.csv` as the primary local column reference (read with cp1252 encoding if required).

- SK_ID_CURR: current application identifier.
- TARGET: recorded payment-difficulty outcome as defined by the competition, present in application_train only.
- DAYS_BIRTH: relative birth-day offset; negative values are expected.
- DAYS_EMPLOYED: employment start offset; 365243 is handled as the identified placeholder.

Record engineered feature definitions, units, unknown-value handling, and source columns as they are finalized. Historical join keys and prediction-time availability must be verified before joining.
