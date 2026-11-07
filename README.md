# Alternative Credit Scoring

A learning project using Kaggle Home Credit Default Risk to predict payment-difficulty risk. A demonstration score may be developed later; this is not a validated lending score.

## Current work

Start with `notebooks/01_data_understanding.ipynb`, then `notebooks/02_application_preprocessing.ipynb`. The second notebook currently focuses on `application_train.csv`. Cleaning and application feature engineering are in progress; encoding, final preprocessing, model training, and historical-table integration are not yet complete.

## Setup

1. Open this project folder in VS Code.
2. Select the Python environment used for your notebooks (currently Anaconda base), or create a separate project environment.
3. Install the packages in `requirements.txt` if needed: `python -m pip install -r requirements.txt`.
4. Select that environment as the notebook kernel and run cells in order. The notebooks find project data from either the project root or the notebooks directory.
5. See `data/README.md` for the dataset location and download source.

## Project folders

| Path | Purpose |
|---|---|
| notebooks/ | Learning, investigation, and experiments |
| src/credit_scoring/ | Finalized reusable Python functions |
| configs/ | Shared settings |
| data/raw/ | Original local datasets |
| data/interim/ | Intermediate cleaned tables and summaries |
| data/processed/ | Prepared feature matrices, outcomes, and row IDs |
| pipelines/ | Scripts coordinating repeatable workflows |
| artifacts/ | Fitted preprocessors, models, and metadata |
| reports/ | Data-quality results, evaluations, and figures |
| dashboard/ | BI dashboard files and SQL |
| app/ | Future prediction API or interface |
| tests/ | Meaningful checks for reusable project code |
| deployment/ | Future deployment configuration |
| docs/ | Scope, architecture, definitions, and teamwork |
| .github/workflows/ | Future GitHub automation |

Most folders are documented scaffolding, not completed functionality. See `docs/team_workflow.md` for the handoff to modeling.

## GitHub

Datasets, local secrets, caches, and generated artifacts are ignored. Use `git status` and review staged files before committing. This setup has not initialized a repository, created a remote, or pushed anything. Shared documentation and selected aggregate reports can be committed; review them for applicant-level information first.
