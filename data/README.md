# Project data

Source: [Home Credit Default Risk on Kaggle](https://www.kaggle.com/competitions/home-credit-default-risk/data). Follow the dataset competition rules and access requirements.

Extract original CSVs into `data/raw/home-credit-default-risk/`. This folder currently contains the downloaded files. Training applications include TARGET; Kaggle test applications do not.

- raw: unchanged source files.
- interim: intermediate cleaned and aggregated tables.
- processed: prepared model inputs, labels, and application IDs.

Data contents are excluded from Git; these instructions and folder guides remain trackable. Filenames should identify their preparation stage and split. Never overwrite the raw CSVs.
