---
name: balanced-accuracy-and-auc-evaluation
description: Use when comparing normalization methods (e.g. none, median, EigenMS,
  NOMIS, CRMN) on a feature table to decide which produces the downstream classification
  performance before pipeline adoption.
when_to_use_negative:
- The input is raw peak-intensity data or a feature table that has not yet been filtered,
  imputed, and log-transformed with tbManipulate — normalize first before cross-validation.
- The goal is single-method performance reporting on one normalization variant rather
  than head-to-head comparison of multiple normalization methods.
- The pipeline uses MultiABLER's built-in normalization functions alone; this skill
  evaluates a suite of alternatives (NOMIS, CRMN) outside MultiABLER's scope.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3174
tools:
- name: MultiABLER
  role: Provides tbManipulate to filter, impute, and log-transform the feature table
    before cross-validation, and supplies normMedian / normEigenMS used in the normalization
    variants being compared.
  repo: https://github.com/holab-hku/MultiABLER
- name: ClassifyR
  role: Runs 5-fold cross-validation with DLDA over 20 repetitions to produce balanced
    accuracy and AUC per normalization variant.
- name: R
  role: Runtime environment for installing MultiABLER via devtools and executing the
    ClassifyR cross-validation loop.
- name: devtools
  role: Installs MultiABLER from GitHub with `devtools::install_github("holab-hku/MultiABLER",
    dependencies = TRUE)`.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:81a8f332aaeeb35457090d6019588d3175ece0a13709f2917a97d3d9244b4e74
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - ClassifyR
  - MultiABLER
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# balanced-accuracy-and-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a lipidomics/metabolomics normalization or classification workflow using ClassifyR 5-fold cross-validation with DLDA over 20 repetitions, reporting median balanced accuracy (BA) and median AUC so normalization variants can be ranked head-to-head on the same data split.

## When to use

Use when comparing normalization methods (e.g. none, median, EigenMS, NOMIS, CRMN) on a feature table to decide which produces the best downstream classification performance before pipeline adoption.

## When NOT to use

- The input is raw peak-intensity data or a feature table that has not yet been filtered, imputed, and log-transformed with tbManipulate — normalize first before cross-validation.
- The goal is single-method performance reporting on one normalization variant rather than head-to-head comparison of multiple normalization methods.
- The pipeline uses MultiABLER's built-in normalization functions alone; this skill evaluates a suite of alternatives (NOMIS, CRMN) outside MultiABLER's scope.

## Inputs

- peak-intensity feature table from MultiABLER extraction (e.g. liver lipidomic dataset, positive mode)
- preprocessed feature table from MultiABLER's tbManipulate (feature filtering, imputation, log-transformation)
- design/contrast matrices compatible with runLimma (only where needed for downstream comparisons)

## Outputs

- normalization_cv_metrics.csv with median balanced accuracy and median AUC per normalization variant
- summary table comparing median BA and AUC across no-norm, median, EigenMS, NOMIS, CRMN, and EigenMS+median scenarios

## How to apply

Preprocess the peak-intensity feature table with MultiABLER's tbManipulate(df, filter_threshold, impute, log_base) to apply feature filtering, imputation, and log-transformation before evaluation. For each normalization variant (no normalization, normMedian, normEigenMS, NOMIS, CRMN, and EigenMS followed by median normalization), run ClassifyR cross-validation using DLDA as the classifier with 5 folds and 20 repetitions. Collect the median balanced accuracy and median AUC across the 20 repetitions for each variant and assemble them into a summary table (e.g. normalization_cv_metrics.csv). On the Chen et al. liver lipidomic dataset (35 samples, positive mode from MetaboLights S-BSST1038), EigenMS ranked best with median BA ≈ 0.68 and AUC ≈ 0.75, and EigenMS followed by median normalization performed similarly (BA ≈ 0.69; AUC ≈ 0.76). Validate by checking that reproduced BA and AUC values for each scenario match the published benchmarks within roughly ±0.05 and that EigenMS-based variants rank among the top performers.

## Related tools

- **MultiABLER** (Provides tbManipulate to filter, impute, and log-transform the feature table before cross-validation, and supplies normMedian / normEigenMS used in the normalization variants being compared.) — https://github.com/holab-hku/MultiABLER
- **ClassifyR** (Runs 5-fold cross-validation with DLDA over 20 repetitions to produce balanced accuracy and AUC per normalization variant.)
- **R** (Runtime environment for installing MultiABLER via devtools and executing the ClassifyR cross-validation loop.)
- **devtools** (Installs MultiABLER from GitHub with `devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)`.)

## Examples

```
Rscript -e 'library(MultiABLER); normed <- normEigenMS(tbManipulate(df, filter_threshold, impute, log_base)); results <- runClassifyR(normed, folds=5, reps=20, classifier=DLDA)'
```

## Evaluation signals

- Median balanced accuracy for each normalization scenario matches the published benchmarks (BA: 0.40 no-norm, 0.68 EigenMS, 0.48 NOMIS, 0.38 CRMN, 0.69 EigenMS+median) within ~±0.05.
- Median AUC values match published benchmarks (AUC: 0.35, 0.75, 0.48, 0.48, 0.37, 0.76) within ~±0.05.
- EigenMS-based variants rank among the top performers on both BA and AUC.
- normalization_cv_metrics.csv is produced with median BA and AUC columns for all six normalization variants tested.
- Cross-validation settings (DLDA classifier, 5 folds, 20 repetitions) are applied consistently across all normalization variants before metric aggregation.

## Limitations

- MultiABLER does not implement kNN imputation, mean normalization, or quantile normalization; those methods are outside the scope of the pipeline and must be applied separately if desired.
- EigenMS is identified as the best-suited normalization for MultiABLER, but performance can vary by dataset; BA/AUC values here are specific to the liver lipidomic dataset and should not be assumed transferable to other sample types without re-evaluation.
- MultiABLER currently does not provide functions to analyze proteomic data, so the evaluation pipeline is specific to metabolomics/lipidomics feature tables.
- The benchmark tolerances (~±0.05) are loose; exact reproduction depends on stochastic cross-validation splits and may require the same fold/repetition scheme.

## Evidence

- [results] Cross-validation setup and metric collection are grounded in the paper's normalization-comparison benchmark.: [evidence span withheld — anchor status: not_found]
- [methods] The specific preprocessing call that prepares the feature table is named in the source card.: [evidence span withheld — anchor status: not_found]
- [discussion] EigenMS identified as the best-suited normalization method for the MultiABLER pipeline.: "In our analyses, we identified EigenMS to be the best suited normalization method for MultiABLER."
- [readme] From the README: installation command and dependency list that ground the runtime environment.: "`devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)`"
