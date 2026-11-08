---
name: calibration-level-exclusion-refinement
description: Use when a targeted LC-MS/MS peak-area dataset (calibration standards
  plus samples) has been drift-corrected and you need per-compound regression models
  whose calibration range is automatically trimmed to the linearly/accurately responding
  standard levels.
when_to_use_negative:
- Peak areas have not yet been integrated — QuantyFey works as additional software
  to already integrated MS data and does not provide integration capabilities
- The user needs full manual control over model, weights, and standard levels — use
  QuantyFey's Interactive Regression Model Optimization instead of the automatic module
- Data are not on a Windows-compatible setup and no Apptainer/R launch path has been
  prepared (the standalone version only works on Windows systems)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: QuantyFey
  role: Provides the Automatic Optimization Module that selects the linear or quadratic
    regression model and appropriate standards, and refines calibration levels per
    compound
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime environment in which the automatic optimization is implemented and
    executed
- name: MultiQuant 3.0.3
  role: Upstream vendor software used for peak integration and manual review whose
    exported peak areas and RTs feed the calibration optimization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:f9f6be45357daed788d651bfa920238997e0280f072d75f4061c28139b27f710
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# calibration-level-exclusion-refinement

## Summary

The Automatic Optimization Module of QuantyFey (R 4.2.0 / Shiny 1.10.0) refines external-calibration regression models per compound by detecting non-linearity, iteratively removing unsuitable upper calibration levels, and excluding lower levels with poor accuracy, automatically selecting a linear or quadratic fit with appropriate weights.

## When to use

Use when a targeted LC-MS/MS peak-area dataset (calibration standards plus samples) has been drift-corrected and you need per-compound regression models whose calibration range is automatically trimmed to the linearly/accurately responding standard levels.

## When NOT to use

- Peak areas have not yet been integrated — QuantyFey works as additional software to already integrated MS data and does not provide integration capabilities
- The user needs full manual control over model, weights, and standard levels — use QuantyFey's Interactive Regression Model Optimization instead of the automatic module
- Data are not on a Windows-compatible setup and no Apptainer/R launch path has been prepared (the standalone version only works on Windows systems)

## Inputs

- Peak-area table for targeted LC-MS/MS data (calibration standards and samples) exported from MultiQuant 3.0.3, including retention times
- Drift-corrected intensities per compound (from IS correction, QC-based drift correction, custom bracketing, or weighted bracketing)
- Per-compound assignment of calibration standard levels and sample positions in the sequence

## Outputs

- Automatically selected regression model type (linear or quadratic) per compound
- Retained (non-excluded) calibration standard levels per compound
- Fitted curve parameters and weights for each compound's calibration curve
- Sample concentrations computed with the automatically optimized calibration curves

## How to apply

Load the deposited peak-area table (calibration standards and samples, exported with peak areas and retention times from MultiQuant 3.0.3) into QuantyFey's Automatic Optimization Module. For each compound, fit linear and quadratic regression models with weighting across the full calibration range and detect non-linearity by comparing model fit across the whole range. Iteratively remove the uppermost calibration level when non-linearity is detected, and exclude lower calibration levels showing poor accuracy, refitting the regression after each removal per compound. Stop refinement when the selected model (linear or quadratic) achieves acceptable linearity and accuracy across the retained standard levels based on the samples' intensities. Record the automatically chosen model type, retained standard levels, and fitted curve parameters, then apply the curves to compute sample concentrations. Validate by comparing automatically optimized quantification against manually optimized fits via mean relative bias.

## Related tools

- **QuantyFey** (Provides the Automatic Optimization Module that selects the linear or quadratic regression model and appropriate standards, and refines calibration levels per compound) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime environment in which the automatic optimization is implemented and executed)
- **MultiQuant 3.0.3** (Upstream vendor software used for peak integration and manual review whose exported peak areas and RTs feed the calibration optimization)

## Examples

```
setwd("path_to_QuantyFey_app.r"); source("app.r")  # then run the Automatic Optimization Module per compound in the Shiny UI
```

## Evaluation signals

- Automatically optimized calibration curves must reproduce concentration results consistent with the manually optimized regression models, quantified via mean relative bias between automatic and manual optimization
- Per compound, the output must record exactly one model type (linear or quadratic) plus the retained set of calibration levels after iterative removal
- No excluded upper levels remain in the fitted range where non-linearity was detected; no lower levels with poor accuracy remain in the retained set
- Each compound's final model must show acceptable linearity and accuracy across retained standard levels given the samples' intensities
- Regression refits must occur after each level removal, i.e., intermediate fits correspond to each candidate calibration range

## Limitations

- QuantyFey is compatible with Windows operating systems only (standalone version); Linux use requires Apptainer, and macOS setup is slow and difficult
- The Automatic Optimization Module does not handle peak integration; upstream integration in MultiQuant is required
- Compound-specific drift-correction strategy choice matters — unsuitable IS selection (e.g., 4-hydroxy-proline, methionine sulfoxide) introduced biases, so the optimization operates on already-strategy-corrected intensities
- Drift-correction methods other than IS correction (custom/weighted bracketing, QC-based) showed variable performance across compounds, which propagates into calibration-level decisions

## Evidence

- [methods] Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards.: "Automatic Optimization Module - automatic selection of linear or quadratic regression model, and selection of appropriate standards."
- [abstract] Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized.: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [readme] Interactive Regression Model Optimization - manual adjustment of model, weights, standard levels etc.: "Interactive Regression Model Optimization - manual adjustment of model, weights, standard levels etc."
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
