---
name: regression-model-summary-interpretation
description: Use when a GetFeatistics workflow has produced regression-model summary
  tables (e.g. summary_regression_models from get_targeted_elaboration, or fixed-effects/mixed-effects/Tobit
  model output) and they must be read, validated, or reported.
when_to_use_negative:
- The task is feature-table processing or pooled QC filtering for untargeted studies
  rather than reading fitted regression-model output
- Input data are raw MS files needing feature extraction (e.g. via XCMS) — regression
  summaries do not yet exist
- The analysis requires calibration-curve quantification itself rather than interpretation
  of the already-fitted model summaries
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GetFeatistics
  role: R-package that fits the regression models (fixed effects, mixed effects, Tobit)
    and emits the concise summary tables and ggplot visualisations being interpreted
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language environment in which GetFeatistics regression summaries are generated
    and read (requires R >= 4.3.1 per README)
- name: ggplot
  role: Grammar used by GetFeatistics for the customisable visualisations that accompany
    the model summaries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:d594932e5ccab1b9e9ef8a19cbcf0f46bdb84494fb80e29b34a18ad58dd82b44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - R
  - ggplot
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# regression-model-summary-interpretation

## Summary

Interpreting fitted regression-model summaries returned by the GetFeatistics package, which streamlines regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for values censored beyond the limits of detection.

## When to use

Use when a GetFeatistics workflow has produced regression-model summary tables (e.g. summary_regression_models from get_targeted_elaboration, or fixed-effects/mixed-effects/Tobit model output) and they must be read, validated, or reported.

## When NOT to use

- The task is feature-table processing or pooled QC filtering for untargeted studies rather than reading fitted regression-model output
- Input data are raw MS files needing feature extraction (e.g. via XCMS) — regression summaries do not yet exist
- The analysis requires calibration-curve quantification itself rather than interpretation of the already-fitted model summaries

## Inputs

- summary_regression_models table returned by get_targeted_elaboration
- results_concentrations, results_accuracy, and cv_internal_standards tables from the same elaboration run
- GetFeatistics example data frames (df_example_targeted) from create_df_examples for a reproducible check

## Outputs

- Interpreted per-molecule regression-model summaries (fixed-effects, mixed-effects, or Tobit)
- Report-ready concise informative tables of model fit
- Optional ggplot visualisations of the fitted models/calibration curves

## How to apply

Run the GetFeatistics elaboration or statistics step first so that concise, informative model-summary tables are generated alongside concentration and accuracy outputs. Identify which model family was fitted: fixed-effects regression for standard associations, mixed-effects models for longitudinal study designs, or Tobit regression when values exceed the limits of detection and are censored. Read the summary tables for per-molecule fitted regression-model information, using the package's design principle that 'output tables are concise and informative, facilitating interpretation and reporting'. For targeted quantification, cross-check the regression summaries against the calibration-curve outputs (results_concentrations, results_accuracy, cv_internal_standards) returned by get_targeted_elaboration to confirm the fitted models are consistent with the internal-standard-normalized quantification. Render supporting visualisations via the package's ggplot-based plots, since output visualisations are fully customisable via the ggplot grammar.

## Related tools

- **GetFeatistics** (R-package that fits the regression models (fixed effects, mixed effects, Tobit) and emits the concise summary tables and ggplot visualisations being interpreted) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language environment in which GetFeatistics regression summaries are generated and read (requires R >= 4.3.1 per README))
- **ggplot** (Grammar used by GetFeatistics for the customisable visualisations that accompany the model summaries)

## Examples

```
library(GetFeatistics); res <- get_targeted_elaboration(df_intensity, df_concentrations, df_qc); summary_regression_models <- res$summary_regression_models
```

## Evaluation signals

- summary_regression_models is non-empty and contains one fitted-model entry per targeted molecule in the input tables
- Model family matches the design: mixed-effects present for longitudinal data, Tobit applied where values are censored beyond limits of detection
- Regression summaries are consistent with results_accuracy (% for curve points and QCs) and cv_internal_standards from the same get_targeted_elaboration run
- Output tables are in the package's concise informative format suitable for interpretation and reporting
- Plots rendered from the summaries are valid ggplot objects that can be saved to file

## Limitations

- Tobit regression is specifically for censoring of values exceeding the limits of detection; applying it to uncensored data is outside the package's stated scope
- Mixed-effects models are intended for longitudinal data; other designs should use the fixed-effects path
- Interpretation depends on upstream QC-filtered, pre-processed intensities (missing values and normality handled) — summaries from unfiltered feature tables are not meaningful

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [abstract] Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar.: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
- [discussion] the efficient construction of multiple regression models with mixed-effect and Tobit models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [intro] data intensities are pre-processed to deal with missing values and data normality: "data intensities are pre-processed to deal with missing values and data normality"
- [readme] R (version ≥ 4.3.1): "Before installing, ensure the following are installed:
- R (version ≥ 4.3.1)"
