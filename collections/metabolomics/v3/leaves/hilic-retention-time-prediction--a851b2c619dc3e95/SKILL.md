---
name: hilic-retention-time-prediction
description: Use when annotating untargeted LC-MS/MS metabolomics features (e.g. HILIC
  plasma spectra) and you need predicted retention times to rank or exclude candidate
  structures from in silico or library MS/MS matches.
when_to_use_negative:
- The analytes are complex, high molecular weight lipids such as triacylglycerols
  or phosphatidylcholines — these were largely excluded from the training set and
  predictions may be erroneous
- The compounds lie outside the chemical landscape of the training set (the article
  warns predictions 'may be more erroneous' and accuracy is limited by training-set
  size and chemical diversity)
- You need to discriminate closely eluting isomers in short ~12 min UPLC methods,
  since 1 min prediction errors preclude this and ~1 min windows excluded about 25%
  of true positives
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: R package providing getCD(), proc.data(), fit.* model training functions,
    RT.spell() prediction, and RT databases used to train HILIC RT models and filter
    MS-FINDER candidates
  repo: https://github.com/olobion/Retip
- name: Keras
  role: High-level neural networks API; fit.keras model preferred for lowest overfitting
    (MAE 0.78 min HILIC, 0.57 min RPLC)
- name: MS-FINDER
  role: Compound identification software (v3.30) into which Retip predicted RT models
    are integrated for isomer search filtering
- name: caret
  role: Used for the 80:20 data split indexed on XlogP and model cross-validation
- name: rcdk
  role: Provides the 286 CDK chemical descriptors computed per compound via getCD()
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:6017a6eb10b3ed001545b370b26f883dfc437f76d8435c12cbd4a95a55291cce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - LightGBM
  - MS-FINDER
  - R
  - Retip
  - caret
  - rcdk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# hilic-retention-time-prediction

## Summary

Predict HILIC retention times for small molecules from 286 CDK chemical descriptors using machine learning models in the Retip R package (random forest, BRNN, XGBoost, LightGBM, Keras), and apply the predictions with a ±1 min window to filter unlikely candidate structures in MS-FINDER compound identification.

## When to use

Use when annotating untargeted LC-MS/MS metabolomics features (e.g. HILIC plasma spectra) and you need predicted retention times to rank or exclude candidate structures from in silico or library MS/MS matches.

## When NOT to use

- The analytes are complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines — these were largely excluded from the training set and predictions may be erroneous
- The compounds lie outside the chemical landscape of the training set (the article warns predictions 'may be more erroneous' and accuracy is limited by training-set size and chemical diversity)
- You need to discriminate closely eluting isomers in short ~12 min UPLC methods, since 1 min prediction errors preclude this and ~1 min windows excluded about 25% of true positives

## Inputs

- Training library of compounds with experimental retention times (e.g. 880 HILIC compounds from MoNA; 398 RPLC compounds from RIKEN PlaSMA) with SMILES/structures for descriptor calculation
- Deconvoluted MS/MS feature list with experimental retention times (e.g. 5,036 HILIC plasma spectra from ST001154)
- MS-FINDER candidate structure lists per elemental formula (e.g. 8,685 candidates for 2,002 formulas)

## Outputs

- Trained RT prediction models (fit.keras preferred) and per-compound predicted retention times
- Metrics table (MAE, RMSE, R2, 95% CI) for training/test/validation sets, e.g. results_table.csv
- Filtered candidate list with excluded candidates flagged, e.g. retention_filter_results.csv with overall, single-candidate, and isomer-group reduction percentages

## How to apply

Compute 286 CDK chemical descriptors with Retip's getCD() function built on rcdk, split the training library 80:20 using XlogP as the index with caret, and clean non-existent values and low-variance columns with proc.data(). Train the candidate models (fit.rf tuning mtry, fit.brnn, fit.xgboost, fit.lightgbm, fit.keras with tuned dense_unit, epochs, dropout and validation_split=0.2); the Keras neural network is the preferred model because it showed minimum overfitting (average test/validation vs training error difference ~0.14 min) with MAE of 0.78 min for HILIC and 0.57 min for RPLC. Predict retention times for candidate structures with RT.spell(), then exclude candidates whose predicted RT differs from the experimental RT by more than ±1 min — the threshold justified by the ~1 min validation MAEs. In the MS-FINDER 3.30 case study on 5,036 deconvoluted ST001154 HILIC plasma MS/MS spectra (2,002 formulas, 8,685 candidates), this filter removed 68% of candidates on average, 48% of single-candidate structures, and 84% of candidates from seven-isomer formula groups (leaving 98 of 609).

## Related tools

- **Retip** (R package providing getCD(), proc.data(), fit.* model training functions, RT.spell() prediction, and RT databases used to train HILIC RT models and filter MS-FINDER candidates) — https://github.com/olobion/Retip
- **Keras** (High-level neural networks API; fit.keras model preferred for lowest overfitting (MAE 0.78 min HILIC, 0.57 min RPLC))
- **MS-FINDER** (Compound identification software (v3.30) into which Retip predicted RT models are integrated for isomer search filtering)
- **caret** (Used for the 80:20 data split indexed on XlogP and model cross-validation)
- **rcdk** (Provides the 286 CDK chemical descriptors computed per compound via getCD())

## Examples

```
devtools::install_github('olobion/Retip'); RTpred <- RT.spell(proc.data(getCD(HILIC_train)), fit.keras)
```

## Evaluation signals

- Reproduced Keras test MAE ≈ 0.78 min for HILIC and ≈ 0.57 min for RPLC
- Keras average [test-training] and [validation-training] MAE error differences ≈ 0.14 min, the smallest of the five models (RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51 min)
- ±1 min RT filter excludes ~68% of MS-FINDER candidate structures on average across the 5,036-spectrum HILIC plasma set
- 48% of the 580 single-candidate formulas are removed, and 84% of the 609 candidates in the 87 seven-isomer formula groups are excluded, leaving 98 surviving structures
- True-positive loss from the ±1 min window should be around ~25%, consistent with the article's stated caveat

## Limitations

- The ±1 min threshold excluded about 25% of true positives in the authors' models, so filtering trades sensitivity for specificity
- None of the current models yield validation MAEs better than ±1 min, suggesting molecular descriptors alone cannot fully explain LC buffer/microenvironment/adsorption interactions
- Prediction accuracy is limited by the number and chemical diversity of training compounds; predictions outside the training chemical landscape may be more erroneous
- It was beyond the scope of the study to test whether RT predictions can be adjusted for small changes in chromatographic methods; models are method-specific (HILIC vs RPLC)

## Evidence
