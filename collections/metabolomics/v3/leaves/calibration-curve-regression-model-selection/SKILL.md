---
name: calibration-curve-regression-model-selection
description: Use when processing vendor-integrated targeted LC-MS/MS peak-area data
  (e.g. MultiQuant exports of calibration standards and samples) and needing to choose,
  per compound, between linear and quadratic regression, set weights, and exclude
  unsuitable standard levels before quantification.
when_to_use_negative:
- The chromatographic data have not yet been peak-integrated — QuantyFey 'works as
  an additional software to already integrated mass spectrometry data, and does not
  provide any integration capabilities'
- The goal is selecting a drift-correction strategy (IS correction, QC-based, custom
  bracketing, weighted bracketing) rather than optimizing the calibration regression
  itself
- A lab SOP mandates a fixed single model and level set with no model/weight/level
  selection to be performed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QuantyFey
  role: Provides the Interactive Regression Model Optimization and Automatic Optimization
    Module used to select linear/quadratic regression models, weights, and appropriate
    standard levels per compound
  repo: https://github.com/CDLMarkus/QuantyFey
- name: R 4.2.0
  role: Runtime environment in which the QuantyFey regression optimization is executed
    (R-portable 4.2.0 for the standalone version)
  repo: https://github.com/CDLMarkus/QuantyFey
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:3d1e80eb6b2f4a387a276299cb4d7e7da90d8e0e7b6307154a9dda84daf0b81b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CDLMarkus/QuantyFey
  tools:
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# calibration-curve-regression-model-selection

## Summary

Automated or interactive selection of linear vs. quadratic calibration regression models, weighting, and retention/exclusion of calibration standard levels for external-calibration LC-MS/MS quantification in QuantyFey. It refines each compound's curve by detecting non-linearity and removing unsuitable upper and lower levels until linearity and accuracy are acceptable across retained standards.

## When to use

Use when processing vendor-integrated targeted LC-MS/MS peak-area data (e.g. MultiQuant exports of calibration standards and samples) and needing to choose, per compound, between linear and quadratic regression, set weights, and exclude unsuitable standard levels before quantification.

## When NOT to use

- The chromatographic data have not yet been peak-integrated — QuantyFey 'works as an additional software to already integrated mass spectrometry data, and does not provide any integration capabilities'
- The goal is selecting a drift-correction strategy (IS correction, QC-based, custom bracketing, weighted bracketing) rather than optimizing the calibration regression itself
- A lab SOP mandates a fixed single model and level set with no model/weight/level selection to be performed

## Inputs

- Peak areas and retention times exported from vendor integration software (e.g. MultiQuant 3.0.3) for targeted LC-MS/MS runs
- Calibration standard measurements spanning the calibration range, labeled with compound identity and level
- Sample peak areas per compound (with injection order for drift-correction context)

## Outputs

- Per-compound selected regression model type (linear or quadratic) with weighting
- Retained calibration standard levels after iterative removal of unsuitable upper/lower levels
- Fitted calibration curve parameters
- Sample concentrations computed with the automatically optimized calibration curves

## How to apply

Load the deposited peak-area dataset (calibration standards and samples) into QuantyFey, a Shiny app running on R 4.2.0. For each compound, fit linear and quadratic regression models (with weighting) across the full calibration range and detect non-linearity by comparing model fit. Iteratively remove the uppermost calibration level when non-linearity is detected, and exclude lower levels showing poor accuracy, refitting the regression after each removal. Stop refinement when the selected model achieves acceptable linearity and accuracy across the retained standard levels based on the samples' intensities; alternatively use the Interactive Regression Model Optimization for manual adjustment of model, weights, and standard levels. Record the automatically selected model type, retained standard levels, and fitted curve parameters, then apply the optimized curves to compute sample concentrations.

## Related tools

- **QuantyFey** (Provides the Interactive Regression Model Optimization and Automatic Optimization Module used to select linear/quadratic regression models, weights, and appropriate standard levels per compound) — https://github.com/CDLMarkus/QuantyFey
- **R 4.2.0** (Runtime environment in which the QuantyFey regression optimization is executed (R-portable 4.2.0 for the standalone version)) — https://github.com/CDLMarkus/QuantyFey

## Examples

```
setwd("path_to_QuantyFey_app.r"); source("app.r")  # launch QuantyFey (R 4.2.x or 4.5.x), then run the Automatic Optimization Module per compound
```

## Evaluation signals

- Automatically optimized curves must reproduce sample concentrations consistent with the manually optimized regression models reported in the paper, assessed via mean relative bias between automatic and manual quantification
- Per compound, the output records the selected model type (linear or quadratic), the retained standard levels, and the fitted curve parameters
- Non-linearity is detected by comparing model fit across the full calibration range; each level removal must be followed by a refit
- After stopping, linearity and accuracy must be acceptable across the retained standard levels based on the samples' intensities
- Lower levels excluded must be those exhibiting poor accuracy; upper levels excluded must be those flagged by the non-linearity detection

## Limitations

- QuantyFey is compatible with Windows operating systems only (standalone version); Linux requires the Apptainer version
- Regression optimization follows drift correction (IS correction, QC-based, custom bracketing, weighted bracketing), which must be configured first
- Automatic optimization results can diverge from manual optimization for some compounds; agreement should be quantified via mean relative bias rather than assumed

## Evidence

- [readme] Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards.: "Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards."
- [abstract] all regression models were manually and automatically optimized: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [methods] manual adjustment of model, weights, standard levels etc.: "Interactive Regression Model Optimization - manual adjustment of model, weights, standard levels etc."
- [readme] additional software to already integrated mass spectrometry data: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
