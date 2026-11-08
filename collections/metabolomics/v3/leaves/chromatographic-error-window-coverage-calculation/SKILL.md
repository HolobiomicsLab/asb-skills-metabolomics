---
name: chromatographic-error-window-coverage-calculation
description: Use when you have trained retention-time prediction models (e.g. Retip's
  fit.rf, fit.keras, fit.xgboost, fit.lightgbm, fit.brnn) and experimental RT values
  for test/validation compounds, and need to quantify what fraction of true positives
  fall within a fixed RT error window.
when_to_use_negative:
- You are adjusting predictions for small changes in chromatographic methods — the
  paper explicitly did not test this
- Predictions are being made outside the chemical landscape of the training set (e.g.
  high molecular weight lipids like triacylglycerols or phosphatidylcholines were
  largely excluded from training)
- Your chromatographic method uses narrow retention-time search windows in short UPLC
  runs (e.g. 12 min methods) where 1 min errors cannot discriminate closely eluting
  compounds
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: Provides the trained ML models (fit.rf, fit.brnn, fit.keras, fit.xgboost,
    fit.lightgbm) and the RT.spell() prediction function used to compute per-compound
    RT errors
  repo: https://github.com/olobion/Retip
- name: ggplot2
  role: Generates the violin plots of prediction error distributions with coverage
    percentages annotated (Figure 2 style)
- name: Keras
  role: One of the five ML algorithms whose error distributions and true-positive
    coverage are compared; yielded MAE of 0.78 min (HILIC) and 0.57 min (RPLC)
- name: XGBoost
  role: One of the five ML algorithms whose error distributions and true-positive
    coverage are compared
- name: LightGBM
  role: One of the five ML algorithms whose error distributions and true-positive
    coverage are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:39ca47be60375720323c832034eb6d68760503009c5473de62e4d98903b751e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - LightGBM
  - R
  - Retip
  - XGBoost
  - brnn
  - caret
  - ggplot2
  - h2o
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# chromatographic-error-window-coverage-calculation

## Summary

Computes and visualizes how well predicted retention times cover true positives within a ±1 min error window across machine learning models (random forest, BRNN, XGBoost, LightGBM, Keras) on HILIC and reversed-phase LC test/validation sets, producing violin plots of prediction error distributions with coverage percentages annotated.

## When to use

Use when you have trained retention-time prediction models (e.g. Retip's fit.rf, fit.keras, fit.xgboost, fit.lightgbm, fit.brnn) and experimental RT values for test/validation compounds, and need to quantify what fraction of true positives fall within a fixed RT error window.

## When NOT to use

- You are adjusting predictions for small changes in chromatographic methods — the paper explicitly did not test this
- Predictions are being made outside the chemical landscape of the training set (e.g. high molecular weight lipids like triacylglycerols or phosphatidylcholines were largely excluded from training)
- Your chromatographic method uses narrow retention-time search windows in short UPLC runs (e.g. 12 min methods) where 1 min errors cannot discriminate closely eluting compounds

## Inputs

- Trained Retip model objects (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm)
- HILIC and RPLC test/validation compound descriptor tables (e.g. from Retip example data)
- Experimental retention time values for each compound in each set
- Chosen error window threshold in minutes (paper uses ±1 min)

## Outputs

- Per-compound prediction error values (predicted minus experimental RT) for each model and set combination
- True-positive coverage percentages within the ±1 min window per model and set
- coverage_summary.csv with aggregate coverage statistics
- violin_error_distributions.png mirroring Figure 2

## How to apply

For each model and data set (HILIC test, RP test, validation), predict RTs for every compound, then compute per-compound prediction error as predicted minus experimental RT. Calculate the fraction of compounds whose absolute error falls within the ±1 min window — this threshold comes from the paper's finding that both HILIC and RP LC predictions have mean absolute errors of about 1 min, which they used as a threshold for excluding false positives. Visualize the full error distributions as violin plots (e.g. with ggplot2) mirroring Figure 2, annotating the coverage percentage per model/set. Expected averages: 65% for HILIC test, 87% for RPLC test, and 76% for validation across the five models. Check that RP model error distributions are highly symmetrical while HILIC model errors skew slightly positive.

## Related tools

- **Retip** (Provides the trained ML models (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm) and the RT.spell() prediction function used to compute per-compound RT errors) — https://github.com/olobion/Retip
- **ggplot2** (Generates the violin plots of prediction error distributions with coverage percentages annotated (Figure 2 style))
- **Keras** (One of the five ML algorithms whose error distributions and true-positive coverage are compared; yielded MAE of 0.78 min (HILIC) and 0.57 min (RPLC))
- **XGBoost** (One of the five ML algorithms whose error distributions and true-positive coverage are compared)
- **LightGBM** (One of the five ML algorithms whose error distributions and true-positive coverage are compared)

## Examples

```
predicted <- RT.spell(fit.keras, test_data); errors <- predicted - test_data$RT; coverage <- mean(abs(errors) <= 1)
```

## Evaluation signals

- Average true-positive coverage should match paper-reported values: HILIC test ≈65%, RPLC test ≈87%, validation ≈76% within tolerance
- RP model error distributions should appear highly symmetrical in the violin plot; HILIC model errors should skew slightly positive
- For each compound, computed error = predicted RT − experimental RT; the sum of compounds with |error| ≤ 1 min divided by total true positives must equal the reported coverage
- Keras models should show the smallest MAE (0.78 min HILIC, 0.57 min RPLC) among the five algorithms
- Median relative error over all predictions should be ≈10.8% of absolute RT (or ≈3.4% of total run time)

## Limitations

- A ±1 min window excludes about 25% of true positives in the paper's models
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity
- Molecular descriptors by themselves seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths), microenvironments, and adsorption/desorption processes
- No current models yield validation MAEs better than ±1 min in retention prediction
- For typical UPLC methods with 12 min run times, 1 min prediction errors preclude discriminating closely eluting compounds

## Evidence

- [results] Violin-plot shape and coverage calibration: [evidence span withheld — anchor status: not_found]
- [discussion] 1 min threshold rationale for false-positive exclusion: "both HILIC and reversed-phase LC retention time predictions had mean absolute errors of about 1 min, which we used as thresholds for excluding false positives"
- [results] Symmetry difference between RP and HILIC error distributions: [evidence span withheld — anchor status: not_found]
