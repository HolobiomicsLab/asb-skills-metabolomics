---
name: hilic-rplc-dataset-handling
description: Use when fitting or evaluating retention time models that must be run
  separately per chromatographic mode (HILIC vs RPLC) on metabolomics RT training
  sets, or when comparing coverage statistics across modes.
when_to_use_negative:
- The input is a single dataset spanning both chromatographic modes that cannot be
  separated into HILIC and RPLC subsets — pooling them masks the mode-specific error
  distributions reported in the paper.
- The task involves adjusting retention time predictions for small changes in chromatographic
  methods; the authors explicitly stated this was beyond their study's scope.
- Compounds fall outside the chemical landscape of the training set (e.g. complex
  high-molecular-weight lipids like triacylglycerols or phosphatidylcholines, which
  were excluded from model training).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: R package providing the five machine learning models and the RT.spell() prediction
    function used to generate HILIC and RPLC RT predictions
  repo: https://github.com/olobion/Retip
- name: Keras
  role: Best-performing model for both chromatographic modes, yielding the lowest
    MAE and minimal overfitting
- name: XGBoost
  role: One of the five models trained separately on HILIC and RPLC datasets to compare
    error distributions
- name: LightGBM
  role: Gradient boosting model trained per chromatographic mode as part of the cross-model
    error comparison
- name: ggplot2
  role: Used to generate the violin plots of error distributions with true-positive
    coverage annotations (Figure 2)
- name: randomForest
  role: Random forest model trained on both HILIC and RPLC datasets for the multi-model
    error comparison
- name: brnn
  role: Bayesian-regularized neural network model included in the HILIC/RPLC error-distribution
    comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:bd09f0976c44c8ed755f688d20c12eb808f329e68e796e36fa809e053ed0e85d
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
  - randomForest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8148148148148148
n_claims: 27
---

# hilic-rplc-dataset-handling

## Summary

Handles parallel training and validation of Retip retention-time prediction models on the HILIC (981 primary metabolites/biogenic amines) and reversed-phase (RPLC) datasets, tracking chromatography-specific error distributions and true-positive coverage within ±1 min windows.

## When to use

Use when fitting or evaluating retention time models that must be run separately per chromatographic mode (HILIC vs RPLC) on metabolomics RT training sets, or when comparing coverage statistics across modes.

## When NOT to use

- The input is a single dataset spanning both chromatographic modes that cannot be separated into HILIC and RPLC subsets — pooling them masks the mode-specific error distributions reported in the paper.
- The task involves adjusting retention time predictions for small changes in chromatographic methods; the authors explicitly stated this was beyond their study's scope.
- Compounds fall outside the chemical landscape of the training set (e.g. complex high-molecular-weight lipids like triacylglycerols or phosphatidylcholines, which were excluded from model training).

## Inputs

- trained Retip model objects (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm)
- HILIC test/validation compound descriptor table
- RPLC test/validation compound descriptor table
- experimental retention times for each compound

## Outputs

- per-model, per-set error table
- true-positive coverage fraction within ±1 min (HILIC, RPLC, validation)
- violin plots of error distributions
- coverage_summary.csv
- violin_error_distributions.png

## How to apply

Load the trained Retip models (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm) alongside the HILIC and RPLC test/validation descriptor tables included as example data in the Retip package. Predict RT for each compound with each model using RT.spell(), then compute per-compound errors (predicted minus experimental). For each model–set combination, calculate the fraction of true positives whose |error| ≤ 1 min, mirroring Figure 2: HILIC test averages 65%, RPLC test 87%, and validation 76%. Also compare training/test/validation MAE gaps to detect overfitting (Keras was best with mean absolute errors of 0.78 min for HILIC and 0.57 min for RPLC).

## Related tools

- **Retip** (R package providing the five machine learning models and the RT.spell() prediction function used to generate HILIC and RPLC RT predictions) — https://github.com/olobion/Retip
- **Keras** (Best-performing model for both chromatographic modes, yielding the lowest MAE and minimal overfitting)
- **XGBoost** (One of the five models trained separately on HILIC and RPLC datasets to compare error distributions)
- **LightGBM** (Gradient boosting model trained per chromatographic mode as part of the cross-model error comparison)
- **ggplot2** (Used to generate the violin plots of error distributions with true-positive coverage annotations (Figure 2))
- **randomForest** (Random forest model trained on both HILIC and RPLC datasets for the multi-model error comparison)
- **brnn** (Bayesian-regularized neural network model included in the HILIC/RPLC error-distribution comparison)

## Examples

```
rt <- RT.spell(train = tr_table, test = te_table, ext = ex_table)
```

## Evaluation signals

- Recomputed average true-positive coverage within ±1 min must match the reported values: HILIC test ≈ 65%, RPLC test ≈ 87%, validation ≈ 76%.
- Keras model MAE should be ≈0.78 min on HILIC and ≈0.57 min on RPLC; deviations beyond this suggest a dataset-assembly error.
- Violin plot of error distributions should show highly symmetric errors for RPLC models and slight positive-error skew for HILIC models.
- Error distributions must be computed separately per chromatographic mode and per model — pooled or averaged distributions invalidate the comparison.
- Small differences between training, test and validation set errors (low overfitting) should be reproducible for the best models.

## Limitations

- Training sets excluded complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines, so coverage estimates are not representative for those compound classes.
- If predictions are made on compounds outside the chemical landscape of the training set, results may be more erroneous.
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity.
- The ±1 min window excludes about 25% of true positives, so narrow coverage thresholds trade false positives for missed true positives.
- It was beyond the scope of the study to test whether retention time predictions could be adjusted for small changes in chromatographic methods.

## Evidence

- [results] The five machine learning models' error distributions and true-positive coverage within ±1 min RT windows are shown in Figure 2: RP test models covered on average 87% and HILIC test models 65% of true positives, while test and validation data both averaged 76% coverage for validation.: [evidence span withheld — anchor status: not_found]
