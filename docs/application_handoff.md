# Simple application-data handoff

## What was prepared

Only application_train.csv. The application notebook contains every step directly, using pandas, NumPy, SciPy, scikit-learn, and joblib. There are no custom project modules to import.

Cleaning corrects employment placeholder 365243 and preserves its flag. Features include age, employment duration, three financial ratios, and a missing car-age flag. The two original birth/employment day offsets are excluded after conversion to years.

Numeric gaps use medians learned from training rows. Missing text becomes __MISSING__, and categories are one-hot encoded using training categories. Numeric scaling supports a Logistic Regression baseline. XNA, Unknown, rare valid categories, and extreme incomes are retained. Validation only uses transform, not fit.

## What to give your modeling teammate

| File | Contents |
|---|---|
| data/processed/application/X_train.npz | Prepared training inputs |
| data/processed/application/X_valid.npz | Prepared validation inputs |
| data/processed/application/train_labels_ids.csv | Original row index, application ID, and target in matrix row order |
| data/processed/application/valid_labels_ids.csv | Validation labels and IDs in matrix row order |
| data/processed/application/unimputed_splits.joblib | Selected feature DataFrames and labels before filling/encoding, with text missing markers normalized |
| artifacts/preprocessors/application/preprocessor.joblib | Fitted standard scikit-learn ColumnTransformer; no src imports required |
| artifacts/metadata/application/manifest.json | Feature schema, retained/dropped columns, versions, and checks |
| data/interim/application/applications_cleaned.joblib | Cleaned original table with ID and TARGET |
| data/interim/application/applications_featured.joblib | Engineered labeled table with ID and TARGET |
| reports/data_quality/application_completion.md | Completion summary |
| reports/data_quality/application_decisions.csv | Cleaning/preprocessing decisions |

The notebook saves these and reloads a sample to verify them. Only load trusted joblib files. Matrices are sparse; do not expand the entire dataset to a dense table unnecessarily.

## Simple loading example for your friend

Run from ProJect:

```python
from pathlib import Path
import pandas as pd
from scipy import sparse

folder = Path("data/processed/application")
X_train = sparse.load_npz(folder / "X_train.npz")
X_valid = sparse.load_npz(folder / "X_valid.npz")
y_train = pd.read_csv(folder / "train_labels_ids.csv")["TARGET"]
y_valid = pd.read_csv(folder / "valid_labels_ids.csv")["TARGET"]
```

Training and evaluation belong to your friend's work. No prediction model has been fitted by this notebook. Prepared matrices support the first fixed validation experiment; cross-validation should rebuild preprocessing within each fold using unimputed inputs. Raw categorical markers are normalized to np.nan before the imputer; use the same column order and marker normalization for future data.

Other source CSVs, historical integration, and Kaggle test predictions have not been completed. They will be separate work when you choose to start.
