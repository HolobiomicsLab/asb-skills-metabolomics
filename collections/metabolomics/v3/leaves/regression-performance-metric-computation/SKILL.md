---
name: regression-performance-metric-computation
description: Decision support for comparing predicted CCS values against experimental
  references across training regimes (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase,
  cross-dataset), and for testing whether a confidence-model filter improves error
  metrics.
when_to_use_negative:
- The predictions are in-domain with negligible outliers (e.g. train-METLIN-test-METLIN
  where models already achieve sub-5% MRE) and filtering would add no informative
  change
- The task is classification rather than regression (e.g. molecule typing or confidence
  classification itself), where MAE/MRE on continuous CCS values is not the target
  metric
- The dataset is not CCS prediction but a different property (e.g. retention time);
  the metric definition and the 5% relative-error threshold are CCS-specific
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: Mol2CCS
  role: Produces the CCS predictions whose MAE/MRE are computed before and after confidence-model
    filtering
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Implementation environment for computing MAE/MRE and training the scikit-learn
    random forest confidence filter
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:4db053c5b4890a7922878c1b2db480e1a45a87c6f844024cdd9b2beb4be450a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# regression-performance-metric-computation

## Summary

Computes MAE and MRE on predicted vs. experimental CCS values, both on the full prediction set and on a confidence-filtered subset, to quantify regression performance and the error-reduction gained by removing low-confidence Mol2CCS predictions.

## When to use

Decision support for comparing predicted CCS values against experimental references across training regimes (train-METLIN-test-METLIN, train-CCSBase-test-CCSBase, cross-dataset), and for testing whether a confidence-model filter improves error metrics.

## When NOT to use

- The predictions are in-domain with negligible outliers (e.g. train-METLIN-test-METLIN where models already achieve sub-5% MRE) and filtering would add no informative change
- The task is classification rather than regression (e.g. molecule typing or confidence classification itself), where MAE/MRE on continuous CCS values is not the target metric
- The dataset is not CCS prediction but a different property (e.g. retention time); the metric definition and the 5% relative-error threshold are CCS-specific

## Inputs

- CSV/parquet of Mol2CCS predictions on METLIN-CCS and CCSBase test splits with columns: smiles, adduct, ccs (experimental), predicted ccs
- Binary labels (prediction correct if absolute relative error ≤ 5% of true CCS) used to train the confidence filter
- Confidence scores from the random forest filter model for each prediction

## Outputs

- Per-regime MAE (Å²) and MRE (%) on unfiltered predictions
- Per-regime MAE and MRE on the confidence-filtered prediction subset
- Precision–recall curve data from threshold sweeps of the filter classifier

## How to apply

For each prediction set (e.g. Mol2CCS predictions on METLIN-CCS or CCSBase test splits), compute MAE as the mean of |predicted CCS − experimental CCS| and MRE as the mean relative error |predicted CCS − experimental CCS| / experimental CCS, per regime. Repeat the computation on the subset of predictions retained by the confidence model (predictions whose absolute relative error falls within 5% of the true CCS value form the positive class used to train the filter). The filter is validated by confirming that filtered-subset MAE and MRE are strictly lower than the unfiltered values, and by sweeping the classifier's probability threshold to map precision against recall before selecting the operating point. Comparing filtered vs. unfiltered metrics per training regime reproduces the error-decrease pattern reported in Fig. 5 and Supplementary Figs. 9–13.

## Related tools

- **Mol2CCS** (Produces the CCS predictions whose MAE/MRE are computed before and after confidence-model filtering) — https://github.com/enveda/ccs-prediction
- **Python** (Implementation environment for computing MAE/MRE and training the scikit-learn random forest confidence filter) — https://github.com/enveda/ccs-prediction

## Examples

```
python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --adduct-column-name "adduct" --ccs-column-name "ccs"
```

## Evaluation signals

- Filtered-subset MAE and MRE must be strictly lower than the unfiltered MAE and MRE for each training regime
- As in-domain training sample size increases, the drop in MAE/MRE after filtering should increase, matching the trend in Fig. 5
- The confidence model's training labels must have been derived from an absolute relative error threshold of 5% against experimental CCS
- Threshold sweep should show monotonic precision–recall trade-off before the operating point is selected
- Metric values should be reproducible by recomputing from the stored prediction CSV files using the same formula (MAE = mean(|Δccs|), MRE = mean(|Δccs|/ccs))

## Limitations

- Metric improvement depends on the quality of the confidence model; if the filter is poorly calibrated it can discard valid predictions and reduce coverage without materially lowering MAE/MRE
- The paper's evaluation is restricted to small molecules and specific chemical-space regions (METLIN-CCS is based on synthetic structures), so metric gains may not transfer to broader or non-synthetic chemical space
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values, which can bias both unfiltered and filtered error estimates

## Evidence

- [results] Filtering Mol2CCS predictions by confidence model improved error metrics: [evidence span withheld — anchor status: not_found]
- [supplementary] Confidence model training threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Confidence model purpose: "confidence models can support by enhancing the reliability of the CCS estimates"
- [intro] Filtering rationale: "we show that using confidence models to filter predictions can result in increased performance of the models"
- [discussion] High accuracy does not generalize: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [readme] Makefile command to train models: "poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet""
