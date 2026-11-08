---
name: automated-vs-manual-optimization-agreement-analysis
description: Decision support for targeted LC-MS/MS external-calibration quantification
  when calibration curves need model/weight/level selection per compound and you must
  verify that automatic regression optimization matches manual expert optimization.
when_to_use_negative:
- Mass spectrometry data is not yet peak-integrated — QuantyFey explicitly 'does not
  provide any integration capabilities' and works only on already integrated data
- The analysis goal is drift correction itself (IS correction, QC-based correction,
  custom/weighted bracketing) rather than calibration regression model selection and
  validation
- The user needs manual, interactive adjustment of model, weights, and standard levels
  instead of the automatic module
tools:
- name: QuantyFey
  role: Runs the Automatic Optimization Module that selects linear/quadratic regression
    and appropriate standards; its interactive mode supplies the manual-optimization
    reference for the agreement analysis
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime environment (R-portable 4.2.0) in which the QuantyFey Shiny app and
    its regression fitting execute
- name: Shiny 1.10.0
  role: Framework hosting the QuantyFey interactive and automatic regression optimization
    interfaces
- name: MultiQuant 3.0.3
  role: Upstream peak integration and manual review that produces the peak areas and
    retention times consumed by the optimization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:af145324062d0c4f9206aaa362cf528dd53c70c9a96d9fee21acceb9814f2b7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny 1.10.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Automated vs. Manual Calibration Regression Optimization Agreement Analysis

## Summary

Reproduces and validates QuantyFey's Automatic Optimization Module, which per compound automatically selects a linear or quadratic (weighted) regression model and prunes unsuitable upper calibration levels (non-linearity) and lower levels with poor accuracy, then benchmarks the resulting concentrations against manually optimized fits via mean relative bias.

## When to use

Decision support for targeted LC-MS/MS external-calibration quantification when calibration curves need model/weight/level selection per compound and you must verify that automatic regression optimization matches manual expert optimization.

## When NOT to use

- Mass spectrometry data is not yet peak-integrated — QuantyFey explicitly 'does not provide any integration capabilities' and works only on already integrated data
- The analysis goal is drift correction itself (IS correction, QC-based correction, custom/weighted bracketing) rather than calibration regression model selection and validation
- The user needs manual, interactive adjustment of model, weights, and standard levels instead of the automatic module

## Inputs

- Exported peak areas and retention times from MultiQuant (v3.0.3, Sciex) for targeted LC-MS/MS data (e.g., porcine plasma dataset)
- Calibration standard levels and sample injection assignments per compound
- Optionally, drift-corrected peak areas from IS correction, QC-based drift correction, custom bracketing, or weighted bracketing

## Outputs

- Per-compound automatically selected model type (linear or quadratic) with weights
- Per-compound retained calibration standard levels after pruning
- Fitted regression curve parameters per compound
- Sample concentrations from automatic optimization
- Mean relative bias between automatic and manual optimization results

## How to apply

Load the deposited targeted LC-MS/MS peak-area dataset (calibration standards and samples) into QuantyFey (R 4.2.0, Shiny 1.10.0). For each compound, fit linear and quadratic weighted regressions across the full calibration range and detect non-linearity by comparing model fit. Iteratively remove the uppermost calibration level when non-linearity is detected and exclude lower levels exhibiting poor accuracy, refitting after each removal. Stop when the selected model achieves acceptable linearity and accuracy across retained standard levels relative to the samples' intensities, then record the model type (linear vs quadratic), retained standard levels, and fitted curve parameters per compound. Apply the optimized curves to compute sample concentrations and quantify agreement with the paper's manually optimized fits using mean relative bias.

## Related tools

- **QuantyFey** (Runs the Automatic Optimization Module that selects linear/quadratic regression and appropriate standards; its interactive mode supplies the manual-optimization reference for the agreement analysis) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime environment (R-portable 4.2.0) in which the QuantyFey Shiny app and its regression fitting execute)
- **Shiny 1.10.0** (Framework hosting the QuantyFey interactive and automatic regression optimization interfaces)
- **MultiQuant 3.0.3** (Upstream peak integration and manual review that produces the peak areas and retention times consumed by the optimization)

## Examples

```
setwd('path_to_QuantyFey_app.r'); source('app.r')  # launch QuantyFey Shiny app, then load MultiQuant-exported peak areas and run the Automatic Optimization Module per compound
```

## Evaluation signals

- Automatically optimized concentrations must reproduce results consistent with the manually optimized regression models reported in the paper
- Agreement is quantified as mean relative bias between automatic and manual quantification for each compound
- Recorded model type, retained standard levels, and curve parameters must be present for every compound after optimization stops
- Non-linearity handling must be verifiable: upper levels removed only when full-range model fit comparison indicates non-linearity, and lower levels removed only for poor accuracy

## Limitations

- Drift-correction strategy performance is compound-specific; e.g., unsuitable IS selection for 4-hydroxy-proline and methionine sulfoxide introduced biases, so agreement analysis should be run per compound and strategy
- IS-corrected concentrations were consistently lower by approximately a factor of two (suggesting matrix enhancement), which can confound cross-method comparison
- Weighted bracketing performed worse than loess-based drift correction in the paper's simulation, affecting the input peak areas fed to optimization
- Standalone QuantyFey only works on Windows; the apptainer route is recommended for Linux and is slow/difficult on macOS

## Evidence

- [abstract] Abstract quote on joint manual/automatic optimization: "Parameters for the drift correction strategies were set up and all regression models were manually and automatically optimized."
- [readme] README description of the Automatic Optimization Module: "**Automatic Optimization Module** - automatic selection of **linear** or **quadratic** regression model, and selection of appropriate standards."
- [readme] README statement that QuantyFey does not integrate raw data: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [readme] README interactive regression optimization feature: "**Interactive Regression Model Optimization** - manual adjustment of model, weights, standard levels etc."
- [abstract] Article evidence that both correction strategies significantly reduced drift effects in the porcine plasma LC-MS/MS dataset: [evidence span withheld — anchor status: not_found]
