# Alternative Credit Scoring - Application Preprocessing

## Where to work

Open `notebooks/02_application_preprocessing.ipynb` in VS Code. Select your Python kernel and run cells from the top. All code is directly in the notebook; no src package, helper scripts, or other preprocessing notebooks are needed.

`notebooks/01_data_understanding.ipynb` contains your earlier exploration.

## Simple folder guide

| Folder | Contents |
|---|---|
| notebooks/ | Your two notebooks; this is where you work |
| data/raw/home-credit-default-risk/ | Original Kaggle CSVs, unchanged |
| data/interim/application/ | Saved cleaned and engineered application tables |
| data/processed/application/ | Prepared training/validation matrices, targets, IDs, and unimputed inputs |
| artifacts/preprocessors/application/ | Fitted standard scikit-learn preprocessing object |
| artifacts/metadata/application/ | Feature names, retained columns, settings, and verification |
| reports/data_quality/ | Inspection findings and completed cleaning decisions |
| docs/application_handoff.md | What the saved files mean for your modeling teammate |

## Current task

Complete application_train.csv preprocessing only: confirmed cleaning, application features, training/validation split, imputation, one-hot encoding, numeric scaling, checks, and saving outputs. No model is trained here. Other Kaggle CSVs remain untouched for later work.

Notebook sections 11-21 contain the completed preparation code. Read one section at a time; use the saved outputs or rerun from the beginning.

Python packages are listed in requirements.txt. If needed, install in your intended environment with `python -m pip install -r requirements.txt`. The notebooks locate data from either the project root or notebooks directory.

Raw/processed data and fitted objects are ignored by Git. Review notebook outputs before public sharing. No commits or pushes are performed by the preprocessing notebook.
