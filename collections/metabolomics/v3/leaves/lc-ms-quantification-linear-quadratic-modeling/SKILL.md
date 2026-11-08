---
name: lc-ms-quantification-linear-quadratic-modeling
description: Use when processing already-integrated targeted LC-MS/MS peak-area data
  (calibration standards plus samples) and you need per-compound automated calibration-curve
  regression without manual model/level tuning.
when_to_use_negative:
- Raw, un-integrated chromatographic data — QuantyFey works only on already integrated
  MS data and 'does not provide any integration capabilities'
- Datasets needing manual control over model choice, weights, or standard levels —
  use the Interactive Regression Model Optimization module instead
- Non-Windows local desktop use of the standalone version, which is Windows-only (Linux
  users need the Apptainer version)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QuantyFey
  role: Provides the Automatic Optimization Module that selects linear/quadratic regression
    and appropriate standards per compound, and the Interactive Regression Model Optimization
    module used as the manual benchmark
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime environment in which QuantyFey (v4.2.0 implementation, Shiny v1.10.0,
    renv v1.1.4) is executed to fit and optimize the calibration regressions
- name: MultiQuant 3.0.3
  role: Upstream peak integration and manual chromatographic review whose exported
    peak areas and RTs feed the calibration regression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:bdd309d10dff5ee0f37679c2721abf2c70388b7f5f84d6b2c37f01da72cedade
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lc-ms-quantification-linear-quadratic-modeling

## Summary

Automatic Optimization Module of QuantyFey that, per compound, selects a linear or quadratic regression model (with weighting) for external-calibration quantification of targeted LC-MS/MS data and iteratively removes unsuitable calibration levels.

## When to use

Use when processing already-integrated targeted LC-MS/MS peak-area data (calibration standards plus samples) and you need per-compound automated calibration-curve regression without manual model/level tuning.

## When NOT to use

- Raw, un-integrated chromatographic data — QuantyFey works only on already integrated MS data and 'does not provide any integration capabilities'
- Datasets needing manual control over model choice, weights, or standard levels — use the Interactive Regression Model Optimization module instead
- Non-Windows local desktop use of the standalone version, which is Windows-only (Linux users need the Apptainer version)

## Inputs

- Peak areas and retention times exported from vendor integration software (e.g., MultiQuant v3.0.3, Sciex) for calibration standards and samples of a targeted LC-MS/MS sequence
- Drift-corrected (or raw) intensity dataset per compound, previously processed via IS correction, QC-based drift correction, custom bracketing, or weighted bracketing
- Optional manual reference: interactively optimized regression models (model type, weights, retained standard levels) per compound for comparison

## Outputs

- Automatically selected regression model type (linear or quadratic) per compound
- Retained calibration standard levels per compound after iterative removal of non-linear upper and inaccurate lower levels
- Fitted curve parameters per compound and computed sample concentrations
- Agreement metrics (e.g., mean relative bias) between automatically and manually optimized quantification

## How to apply

Load the deposited peak-area dataset (calibration standards and samples) into QuantyFey, the Shiny app running on R-portable 4.2.0. For each compound, fit linear and quadratic regression models with weighting across the full calibration range and detect non-linearity by comparing model fit. When non-linearity is detected, iteratively remove the uppermost calibration level and refit; separately exclude lower calibration levels showing poor accuracy, refitting after each removal. Stop refinement when the selected model (linear or quadratic) achieves acceptable linearity and accuracy across the retained standard levels based on the samples' intensities, then record the model type, retained levels, and curve parameters and apply the fitted curves to compute sample concentrations. The rationale is that manual optimization of model, weights, and standard levels is laborious per compound, so the module automates the same decisions; results should agree with manually optimized fits as assessed by mean relative bias.

## Related tools

- **QuantyFey** (Provides the Automatic Optimization Module that selects linear/quadratic regression and appropriate standards per compound, and the Interactive Regression Model Optimization module used as the manual benchmark) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime environment in which QuantyFey (v4.2.0 implementation, Shiny v1.10.0, renv v1.1.4) is executed to fit and optimize the calibration regressions)
- **MultiQuant 3.0.3** (Upstream peak integration and manual chromatographic review whose exported peak areas and RTs feed the calibration regression)

## Examples

```
Rscript -e 'setwd("path_to_QuantyFey_app.r"); source("app.r")'  # then use the Automatic Optimization Module tab to auto-select linear/quadratic models and standards
```

## Evaluation signals

- Automatically optimized calibration curves must reproduce concentrations consistent with manually optimized regression models, quantified by mean relative bias between automatic and manual quantification
- For each compound, the output must record the selected model type (linear vs quadratic), the retained standard levels, and fitted curve parameters
- Non-linearity handling: upper calibration levels must be removed only when full-range model-fit comparison indicates non-linearity, with refitting after each removal
- Lower calibration levels excluded must correspond to those exhibiting poor accuracy relative to sample intensity range
- All regression fitting must be performed within the documented environment (R 4.2.0, Shiny 1.10.0, dependencies pinned by renv 1.1.4)

## Limitations

- QuantyFey is compatible with Windows operating systems only for the standalone version; Apptainer setup is complicated and slow on macOS
- Bracketing-based approaches (custom and weighted bracketing) showed variable performance across compounds, which can affect downstream calibration fits
- Unsuitable internal-standard selection (e.g., 4-hydroxy-proline, methionine sulfoxide) introduced biases, so IS choice must be checked before regression optimization
- No changelog is available for the repository

## Evidence

- [methods] Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards.: "Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards."
- [abstract] all regression models were manually and automatically optimized: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [abstract] QuantyFey is implemented in R (v4.2.0): "QuantyFey is implemented in R (v4.2.0) and utilizes the Shiny framework (v1.10.0)"
- [methods] Interactive Regression Model Optimization - manual adjustment of model, weights, standard levels etc.: "Interactive Regression Model Optimization - manual adjustment of model, weights, standard levels etc."
- [readme] does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
