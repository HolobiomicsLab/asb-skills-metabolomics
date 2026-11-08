---
name: weighted-regression-model-selection
description: Use when building quantitative calibration curves from a serial concentration
  standard series (e.g., narcotics standards) and estimating an unknown test sample's
  concentration by choosing between weighted linear and quadratic fits.
when_to_use_negative:
- The data are untargeted feature tables without a serial concentration standard series
  — calibration requires known standard concentrations
- Analyte signal quality is inadequate (e.g., amphetamine's S/N ratio of 0.19 in the
  narcotics data set), making curve fitting and quantification unreliable
- LC-MS experimental conditions vary across standard and test injections — consistency
  is crucial for a reliable calibration curve
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: SMART 2.0
  role: Provides the concentration calibration module that fits weighted linear and
    quadratic curves, removes outliers via Cook's distance, and estimates sample concentrations
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Baseline integrated untargeted metabolomics tool that SMART 2.0 extends with
    the calibration module
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which SMART is written and the calibration workflow
    is executed
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:bd7ae2022013fce72dd044b9daf28c057840eb5e2926de885698a34e5f9f2aa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# weighted-regression-model-selection

## Summary

Fit both linear and quadratic calibration curves with weighting to a concentration standard series, apply Cook's-distance-based outlier removal with refitting, and select/apply the model via adjusted R² to estimate unknown sample concentrations with relative errors.

## When to use

Use when building quantitative calibration curves from a serial concentration standard series (e.g., narcotics standards) and estimating an unknown test sample's concentration by choosing between weighted linear and quadratic fits.

## When NOT to use

- The data are untargeted feature tables without a serial concentration standard series — calibration requires known standard concentrations
- Analyte signal quality is inadequate (e.g., amphetamine's S/N ratio of 0.19 in the narcotics data set), making curve fitting and quantification unreliable
- LC-MS experimental conditions vary across standard and test injections — consistency is crucial for a reliable calibration curve

## Inputs

- Calibration standard series with known concentrations (e.g., narcotics standard data from the SMART 2.0 GitHub repository, loaded into R)
- Test sample peak intensities at an unknown concentration (e.g., the 500 ppb narcotics test sample)

## Outputs

- Fitted weighted linear and quadratic calibration curve parameters per analyte
- Adjusted R² values for each fitted curve (e.g., heroin: 0.9873 linear, 0.9983 quadratic)
- Estimated test-sample concentrations with per-analyte relative errors
- calibration_results.csv containing curve parameters, adjusted R² values, and estimated concentrations with relative errors

## How to apply

Load the calibration example data (e.g., the narcotics data shipped in the SMART 2.0 GitHub repository) into R. Invoke the SMART 2.0 concentration calibration module to fit both linear and quadratic calibration curves with weighting to the standard series. Apply the module's outlier-removal procedure based on Cook's distance and confidence intervals, then refit the curves after outlier exclusion. Compare models using adjusted R² — for heroin in the narcotics example the quadratic fit (0.9983) outperforms the linear fit (0.9873) — and use the fitted curves to estimate the concentration of the unknown test sample (500 ppb), reporting relative errors per analyte (e.g., −0.0891% for morphine, 54.7643% for amphetamine). Write curve parameters, adjusted R² values, estimated concentrations, and relative errors to calibration_results.csv.

## Related tools

- **SMART 2.0** (Provides the concentration calibration module that fits weighted linear and quadratic curves, removes outliers via Cook's distance, and estimates sample concentrations) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Baseline integrated untargeted metabolomics tool that SMART 2.0 extends with the calibration module) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which SMART is written and the calibration workflow is executed) — https://github.com/YuJenL/SMART

## Evaluation signals

- Adjusted R² for heroin's fitted curves must equal 0.9873 (linear) and 0.9983 (quadratic)
- Relative error for morphine in the 500 ppb test sample must equal −0.0891% (lowest error in the set)
- Relative error for amphetamine in the 500 ppb test sample must equal 54.7643% (highest error in the set)
- Outlier removal based on Cook's distance and confidence intervals must be applied and curves refitted before reporting final parameters
- Outputs must be persisted to calibration_results.csv with curve parameters, adjusted R² values, and estimated concentrations

## Limitations

- Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve but is not guaranteed; variations were observed
- Low S/N (e.g., amphetamine S/N = 0.19) and possibly inappropriate peak detection parameters can lead to large quantification errors; re-evaluation of LC-MS conditions is recommended
- Increasing the number of experimental replicates is recommended to enhance calibration and quantification accuracy
- SMART 2.0 cannot detect heroin and nimetazepam, and HMDB does not encompass them, limiting downstream peak identification for these analytes

## Evidence

- [abstract] SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [discussion] consistency is crucial for reliable calibration: "Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification"
- [discussion] amphetamine low signal-to-noise ratio: "the S/N ratio of amphetamine was found to be only 0.19"
- [abstract] SMART repo provides codes, user guide, and example data: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [readme] SMART module 8 performs concentration calibration: "Post-analysis: Execute peak identification and concentration calibration."
