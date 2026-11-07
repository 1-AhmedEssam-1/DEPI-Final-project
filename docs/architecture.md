# Planned workflow

Raw CSVs -> cleaning -> application/historical features -> train/validation split membership -> training-fitted preprocessing -> prepared matrices -> model training/evaluation -> inference and BI outputs.

Preserve split membership as features evolve. Estimate imputation, encoding vocabularies, scaling, and data-derived selection rules from training rows only. Cross-validation requires preprocessing to be refitted inside each training fold.

Code: src holds reusable functions; pipelines coordinates them; notebooks explains experiments. Data: raw/interim/processed. Fitted tools: artifacts. Human-readable results: reports.

This describes the intended architecture, not an already implemented system.
