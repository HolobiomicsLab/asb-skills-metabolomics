---
name: calibration-curve-fitting
description: Use when a targeted LC-MS/MS or GC-MS workflow requires converting measured
  peak areas into concentrations from calibration standards, especially where generic
  linear vendor fits underperform and a non-linear calibration model is needed.
when_to_use_negative:
- When the analysis is untargeted/semi-targeted feature detection without calibration
  standards - SmartPeak's calibration step assumes known calibration-level concentrations
- When peak areas are not yet integrated (the calibration fit operates on picked/integrated
  features, not raw chromatograms)
- For isotopologue correction or fluxomics MDV parsing, which are handled by BFAIR
  rather than the SmartPeak calibration step
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SmartPeak
  role: Runs the calibration curve fitting step (non-linear Gaussian-Mixtures and
    generic linear options) as part of the automated targeted quantitation workflow
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak GUI
  role: Used to inspect the fitted calibration model and the points selected for the
    fit via 'View | Calibrators'
- name: SmartPeak CLI
  role: Command-line equivalent of the GUI for scripting the calibration workflow
    headlessly
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: pyOpenMS
  role: Can be used to parse the FeatureDB report and extract per-feature peak_area
    values to build calibration points outside SmartPeak
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:64ed9ab0a8d208e169911c90b5467881ff99acf6a078641cda28aee1f53d1add
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - SmartPeak GUI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# calibration-curve-fitting

## Summary

Fit a calibration curve (linear, log-linear, or SmartPeak's non-linear Gaussian-Mixtures model) relating analyte-to-internal-standard response to known concentration for quantitation of unknowns in targeted metabolomics/lipidomics workflows.

## When to use

Use when a targeted LC-MS/MS or GC-MS workflow requires converting measured peak areas into concentrations from calibration standards, especially where generic linear vendor fits underperform and a non-linear calibration model is needed.

## When NOT to use

- When the analysis is untargeted/semi-targeted feature detection without calibration standards - SmartPeak's calibration step assumes known calibration-level concentrations
- When peak areas are not yet integrated (the calibration fit operates on picked/integrated features, not raw chromatograms)
- For isotopologue correction or fluxomics MDV parsing, which are handled by BFAIR rather than the SmartPeak calibration step

## Inputs

- FeatureDB report with per-feature metadata (e.g., peak_area) for calibration standard injections
- Quantitation method file specifying calibration-level concentrations, component/transition mappings, and reference internal standards
- Raw chromatogram files (mzML) for calibration standards, processed through peak picking to yield integrated peak areas

## Outputs

- Fitted calibration curve parameters (e.g., mixture component weights, means, variances for Gaussian-Mixtures; slope/intercept for linear)
- Coefficient of determination (R-squared) of the fitted model against calibration points
- Calculated concentrations for unknown sample injections

## How to apply

Compute the response variable as the analyte-to-internal-standard peak area ratio for each calibration-level injection loaded from a FeatureDB report with per-feature metadata such as peak_area. Assign each standard sample to its sample class (SmartPeak supports 6 sample types) and supply the quantitation method (concentrations, transitions, and reference IS) so the calibration routine can build the concentration-vs-response points. Fit the curve using SmartPeak's calibration step, preferring the non-linear Gaussian-Mixtures model, which the article reports outperformed generic linear methods in targeted proteomics; inspect the fitted parameters via 'View | Calibrators' in the SmartPeak GUI. Use the fitted model to quantitate unknown sample injections and validate by checking the fit's coefficient of determination - the paper's calibration achieved R-squared = 0.99, so R-squared in 0.98-1.00 is the expected range.

## Related tools

- **SmartPeak** (Runs the calibration curve fitting step (non-linear Gaussian-Mixtures and generic linear options) as part of the automated targeted quantitation workflow) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak GUI** (Used to inspect the fitted calibration model and the points selected for the fit via 'View | Calibrators')
- **SmartPeak CLI** (Command-line equivalent of the GUI for scripting the calibration workflow headlessly) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **pyOpenMS** (Can be used to parse the FeatureDB report and extract per-feature peak_area values to build calibration points outside SmartPeak)

## Examples

```
smartpeak-cli --session Targeted_quantitation_with_LC-MSMS_5500_QTRAP_RapidRIP --workflow CALIBRATE
```

## Evaluation signals

- R-squared of the fitted calibration model against calibration points falls in 0.98-1.00 (paper reports 0.99)
- Fitted mixture component parameters (weights, means, variances) are physically meaningful and available for inspection
- Quantitated concentrations of unknowns are computed without manual re-fitting, and integration counts match the paper's reported correct-integration percentages
- All calibration standards are correctly assigned to the standard sample class among SmartPeak's 6 sample types

## Limitations

- The calibration model quality depends on prior peak-picking accuracy; SmartPeak's automated workflow recovered only ~89-95% of features correctly across datasets, so misintegrated peaks degrade the fit
- Quantitation is relative; it requires a valid quantitation method with calibration concentrations and reference IS rather than being absolute standard addition
- For performance reasons, the SmartPeak GUI limits viewing of large feature tables (5000 entries) and chromatogram points (9000), which can hinder manual calibration inspection on large datasets

## Evidence

- [results] Non-linear Gaussian-Mixtures calibration model: [evidence span withheld — anchor status: not_found]
- [methods] Calibration as relative quantitation: [evidence span withheld — anchor status: not_found]
- [methods] Six sample classes: [evidence span withheld — anchor status: not_found]
- [methods] Calibration R-squared 0.99: [evidence span withheld — anchor status: not_found]
- [readme] View calibrators in GUI: "A detailed look at the calibration fitted model and selected points for the model can be seen with ``View | Calibrators``"
- [methods] FeatureDB peak_area metadata: [evidence span withheld — anchor status: not_found]
