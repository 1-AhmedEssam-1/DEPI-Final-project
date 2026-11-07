# Team workflow

## Data preparation work

Continue in notebooks/02_application_preprocessing.ipynb. Finish and verify application_train cleaning, deterministic features, imputation, encoding, and any agreed scaling. Then extract stable reusable code into src and save documented outputs.

## Modeling handoff

Supply prepared train/validation matrices, labels, application IDs, feature names, fitted preprocessing, cleaning/feature code, split settings, and dependency versions. Also supply unimputed splits or a reproducible reconstruction path for fold-specific preprocessing during cross-validation.

The modeling contributor owns baseline training, evaluation, tuning, and threshold experiments. Keep validation independent from fitted preprocessing. Do not fit on Kaggle test.

## Shared work

Agree on column changes and record decisions. Preserve fixed split membership when adding features. Use small Git commits with clear descriptions. Keep raw data and secrets out of commits. Historical tables, BI, serving, and monitoring are later shared milestones.
