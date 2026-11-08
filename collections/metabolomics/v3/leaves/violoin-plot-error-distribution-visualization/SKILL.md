---
name: violoin-plot-error-distribution-visualization
description: Use when Retip RT predictions (predicted minus experimental retention
  time) are available for HILIC or RPLC test/validation compounds and you need to
  visualize per-model error distributions and quantify true-positive coverage within
  ±1 min windows, as in Figure 2 of the Retip paper.
when_to_use_negative:
- When the goal is model training or hyperparameter tuning rather than post-hoc evaluation
  of already-trained Retip models.
- When chromatographic data fall outside the training chemical landscape (e.g., triacylglycerols
  or phosphatidylcholines that were largely excluded) — error distributions and coverage
  percentages will be unreliable.
- When you need to discriminate closely eluting isomers in short UPLC runs; the paper
  notes 1 min prediction errors preclude discriminating closely eluting compounds
  in typical 12 min UPLC methods.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3391
tools:
- name: Retip
  role: Source of trained RT prediction models and example test/validation data; predictions
    feed the error distribution calculation
  repo: https://github.com/olobion/Retip
- name: ggplot2
  role: R plotting library used to render the violin plots of error distributions,
    mirroring Figure 2
- name: Keras
  role: One of the five Retip models whose error distributions and ±1 min coverage
    are visualized; best-performing model with MAE 0.78 min HILIC / 0.57 min RPLC
- name: XGBoost
  role: Retip model whose error distributions and ±1 min coverage are visualized
- name: LightGBM
  role: Retip model whose error distributions and ±1 min coverage are visualized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:36cc6821f4ecc22af49e9735980bbaf03034bf229228d74c03ffc502035daf0c
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# violoin-plot-error-distribution-visualization

## Summary

Generates ggplot2 violin plots of retention time prediction error distributions for Retip machine learning models, annotating the fraction of true positives falling within the ±1 min RT window (HILIC test 65%, RPLC test 87%, validation 76%) to judge model coverage and error skew.

## When to use

Use when Retip RT predictions (predicted minus experimental retention time) are available for HILIC or RPLC test/validation compounds and you need to visualize per-model error distributions and quantify true-positive coverage within ±1 min windows, as in Figure 2 of the Retip paper.

## When NOT to use

- When the goal is model training or hyperparameter tuning rather than post-hoc evaluation of already-trained Retip models.
- When chromatographic data fall outside the training chemical landscape (e.g., triacylglycerols or phosphatidylcholines that were largely excluded) — error distributions and coverage percentages will be unreliable.
- When you need to discriminate closely eluting isomers in short UPLC runs; the paper notes 1 min prediction errors preclude discriminating closely eluting compounds in typical 12 min UPLC methods.

## Inputs

- Trained Retip model objects (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm)
- HILIC and RPLC test/validation compound molecular descriptor tables from Retip example data
- Experimental retention time values for each test/validation compound

## Outputs

- Violin plot of per-model prediction error distributions with true-positive coverage percentages annotated (mirrors Figure 2)
- coverage_summary.csv with per-model, per-dataset ±1 min true-positive coverage fractions

## How to apply

Load the trained Retip models (fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm) and the HILIC/RPLC test and validation descriptor tables from the Retip example data. Predict retention times for each compound with each model, then compute per-compound prediction errors (predicted minus experimental RT). For each model and dataset combination, calculate the fraction of true positives whose absolute error is ≤1 min — the paper's threshold used for excluding false positives. Render violin plots with ggplot2, mirroring Figure 2, and annotate the coverage percentages: RP test models averaged 87%, HILIC test models 65%, and validation sets 76% coverage. Write coverage_summary.csv and export the plot as violin_error_distributions.png.

## Related tools

- **Retip** (Source of trained RT prediction models and example test/validation data; predictions feed the error distribution calculation) — https://github.com/olobion/Retip
- **ggplot2** (R plotting library used to render the violin plots of error distributions, mirroring Figure 2)
- **Keras** (One of the five Retip models whose error distributions and ±1 min coverage are visualized; best-performing model with MAE 0.78 min HILIC / 0.57 min RPLC)
- **XGBoost** (Retip model whose error distributions and ±1 min coverage are visualized)
- **LightGBM** (Retip model whose error distributions and ±1 min coverage are visualized)

## Examples

```
Rscript -e "library(Retip); library(ggplot2); preds <- RT.spell(fit.keras, test_hilic); err <- preds$predicted - preds$RT; cov <- mean(abs(err) <= 1) * 100; print(cov)"
```

## Evaluation signals

- Recomputed average true-positive coverage within ±1 min should match the reported values within tolerance: HILIC test 65%, RPLC test 87%, validation 76%.
- The violin plot should show highly symmetrical error distributions for RP test models and a slight positive-error skew for HILIC models, as described in the paper.
- Per-compound error values must be computed as predicted minus experimental RT for each of the five models; any model missing from the panel indicates incomplete application.
- The exported coverage_summary.csv must contain one row per model × dataset (HILIC test, RPLC test, validation) combination.
- The violin plot file violin_error_distributions.png must render without missing panels and annotate the coverage percentages on each violin.

## Limitations

- The 1 min threshold excluded about 25% of true positives in the paper's models, so coverage percentages below 100% are expected and not a tool failure.
- Predictions made outside the chemical landscape of the training set may be more erroneous; the paper notes many chemicals in test and validation were not well represented by the training models.
- Retention time predictions could not be tested for small changes in chromatographic methods, so error distributions are only validated for the specific HILIC and RPLC conditions used in the study.
- Complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded from training, so their error distributions are not characterized.

## Evidence

- [full_text] Violin plot rendered with ggplot2 as stated in workflow: [evidence span withheld — anchor status: not_found]
