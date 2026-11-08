---
name: retention-time-prediction-model-inference
description: Use when you have SMILES/InChI structures or a compound descriptor table
  and need predicted retention times for HILIC or reversed-phase LC-MS runs to filter
  isomer candidates, rank unknowns, or remove unlikely annotations in metabolomics
  workflows.
when_to_use_negative:
- When the target compounds fall outside the chemical landscape of the training set
  — predictions may be more erroneous if Retip is applied to complex high molecular
  weight lipids (e.g., triacylglycerols or phosphatidylcholines) that were largely
  excluded from training.
- When very narrow retention time search windows are required — the paper notes that
  for typical UPLC methods with 12 min run times, 1 min prediction errors preclude
  discriminating closely eluting compounds.
- When the analysis involves small changes in chromatographic methods — the study
  explicitly did not test whether retention time predictions can be adjusted for such
  changes.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: Retip
  role: R/Python package providing machine learning models for retention time prediction
    and filtering of false positives
  repo: https://www.retip.app
- name: MS-FINDER
  role: Downstream compound identification software where Retip-predicted retention
    times are used to filter isomer candidates
- name: MS-DIAL
  role: Mass spectrometry software integrating Retip predictions for metabolome-wide
    compound annotations
- name: Keras
  role: Deep learning model used within Retip to train retention time prediction models
- name: XGBoost
  role: Gradient boosting model used within Retip to train retention time prediction
    models
- name: LightGBM
  role: Gradient boosting framework used within Retip to train retention time prediction
    models
- name: BRNN
  role: Bayesian-regularized neural network model used within Retip to train retention
    time prediction models
- name: Random Forest
  role: Decision tree model used within Retip to train retention time prediction models
- name: H2O autoML
  role: Automatic machine learning tool available in Retip 2.0 for retention time
    prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:9ea2cf2bc309b1867a748f16248a910366e3f625dceb0a0f1884b91803ef13ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BRNN
  - H2O autoML
  - Keras
  - LightGBM
  - MS-DIAL
  - MS-FINDER
  - R
  - Random Forest
  - Retip
  - XGBoost
  - brnn
  - caret
  - ggplot2
  - h2o
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# retention-time-prediction-model-inference

## Summary

Apply trained Retip machine learning models (Random Forest, BRNN, XGBoost, LightGBM, Keras, or H2O autoML) to predict retention times for small molecules in HPLC-MS, enabling retention-time-based filtering of false positive annotations in untargeted metabolomics.

## When to use

Use when you have SMILES/InChI structures or a compound descriptor table and need predicted retention times for HILIC or reversed-phase LC-MS runs to filter isomer candidates, rank unknowns, or remove unlikely annotations in metabolomics workflows.

## When NOT to use

- When the target compounds fall outside the chemical landscape of the training set — predictions may be more erroneous if Retip is applied to complex high molecular weight lipids (e.g., triacylglycerols or phosphatidylcholines) that were largely excluded from training.
- When very narrow retention time search windows are required — the paper notes that for typical UPLC methods with 12 min run times, 1 min prediction errors preclude discriminating closely eluting compounds.
- When the analysis involves small changes in chromatographic methods — the study explicitly did not test whether retention time predictions can be adjusted for such changes.

## Inputs

- compound descriptor table (SMILES/InChI to molecular descriptors)
- trained Retip model object (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm, or H2O autoML)
- list of target compounds with known molecular structures

## Outputs

- predicted retention times (min) for each compound
- prediction error distribution (predicted minus experimental RT)
- fraction of true positives within ±1 min RT window
- filtered candidate structure list (with retention time filter applied)

## How to apply

Load a trained Retip model (e.g., fit.rf, fit.keras, fit.xgboost, fit.lightgbm, fit.brnn) and compute molecular descriptors (e.g., via rcdk) for the target compounds. Predict retention times for each compound, then evaluate prediction error distributions. Apply a ±1 min retention time window as threshold for excluding unlikely structure candidates — this reduced candidate structures by 68% in MS-FINDER isomer search on mouse blood plasma while excluding about 25% of true positives. For validating model quality, check that Keras yields mean absolute errors of 0.78 min for HILIC and 0.57 min for RPLC, and that true positive coverage within ±1 min windows averages 87% (RP test) and 65% (HILIC test).

## Related tools

- **Retip** (R/Python package providing machine learning models for retention time prediction and filtering of false positives) — https://www.retip.app
- **MS-FINDER** (Downstream compound identification software where Retip-predicted retention times are used to filter isomer candidates)
- **MS-DIAL** (Mass spectrometry software integrating Retip predictions for metabolome-wide compound annotations)
- **Keras** (Deep learning model used within Retip to train retention time prediction models)
- **XGBoost** (Gradient boosting model used within Retip to train retention time prediction models)
- **LightGBM** (Gradient boosting framework used within Retip to train retention time prediction models)
- **BRNN** (Bayesian-regularized neural network model used within Retip to train retention time prediction models)
- **Random Forest** (Decision tree model used within Retip to train retention time prediction models)
- **H2O autoML** (Automatic machine learning tool available in Retip 2.0 for retention time prediction)

## Examples

```
devtools::install_github('olobion/Retip'); retip::RT.spell(fit.keras, descriptors)
```

## Evaluation signals

- Mean absolute error (MAE) should be ~0.78 min for HILIC and ~0.57 min for RPLC when using the best-performing Keras model.
- True positive coverage within ±1 min RT window should average ~87% for reversed-phase test models and ~65% for HILIC test models.
- Applying a ±1 min window should reduce candidate structures by ~68% in MS-FINDER isomer search, consistent with the paper's mouse blood plasma benchmark.
- Prediction errors should be about 3-fold less than previous models with a total median 10.8% error relative to absolute retention times.
- Validation MAE should not exceed ±1 min — models performing worse than this threshold indicate insufficient molecular descriptor complexity for the target analyte class.

## Limitations

- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity — many chemicals in test and validation sets were not well represented by the training models.
- Molecular descriptors may not fully explain complex interactions between LC buffer systems (including ionic strengths), microenvironments, and adsorption/desorption processes.
- No chromatographic processing tool to date actively employs retention time predictions for metabolome-wide compound annotations — Retip predictions must be manually integrated into downstream workflows.
- Predictions outside the chemical landscape of the training set (e.g., high molecular weight lipids like triacylglycerols or phosphatidylcholines) may be more erroneous.
- For typical UPLC methods with 12 min run times, 1 min prediction errors preclude discriminating closely eluting compounds — narrow retention time windows are not yet adequate but are useful to disclose likely in-source fragmentation MS/MS spectra and rank positional isomers.

## Evidence
