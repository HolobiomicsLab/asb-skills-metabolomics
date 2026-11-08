---
name: linear-quadratic-regression-model-fitting
description: Use when you have a concentration-series calibration data set (e.g.,
  narcotics standards) plus a test sample of unknown concentration, and need fitted
  linear/quadratic calibration curves with adjusted R2 and per-analyte relative errors.
when_to_use_negative:
- The task is untargeted metabolomics preprocessing or peak annotation rather than
  targeted concentration quantification — use SMART's untargeted modules instead.
- Calibration standards were not acquired under consistent LC-MS experimental conditions;
  the article states consistency is crucial for constructing a reliable calibration
  curve and accurate quantification.
- The analyte cannot be detected at all (e.g., SMART 2.0 could not detect heroin and
  nimetazepam in the narcotics data), leaving no peak response to regress against
  concentration.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: SMART 2.0
  role: Provides the concentration calibration module that fits weighted linear and
    quadratic curves, performs Cook's-distance-based outlier removal, and estimates
    test-sample concentrations
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Base integrated R tool upon which SMART 2.0's concentration calibration module
    is built
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Execution environment in which the SMART calibration workflow is run and calibration_results.csv
    is generated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:188494decaec765fe9456f544f52589e8ded30c538f9d58505cf6bb1c9e52fef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# linear-quadratic-regression-model-fitting

## Summary

Fits weighted linear and quadratic calibration curves to concentration-series data, removes outliers via Cook's distance and confidence intervals, refits, and uses the models to estimate unknown-sample concentrations with relative errors, as implemented in SMART 2.0's concentration calibration module for the 12-narcotics data set.

## When to use

Use when you have a concentration-series calibration data set (e.g., narcotics standards) plus a test sample of unknown concentration, and need fitted linear/quadratic calibration curves with adjusted R2 and per-analyte relative errors.

## When NOT to use

- The task is untargeted metabolomics preprocessing or peak annotation rather than targeted concentration quantification — use SMART's untargeted modules instead.
- Calibration standards were not acquired under consistent LC-MS experimental conditions; the article states consistency is crucial for constructing a reliable calibration curve and accurate quantification.
- The analyte cannot be detected at all (e.g., SMART 2.0 could not detect heroin and nimetazepam in the narcotics data), leaving no peak response to regress against concentration.

## Inputs

- narcotics calibration example data from the SMART 2.0 GitHub repository (concentration series plus 500 ppb test sample)
- concentration-peak-response data for the 12 narcotics standards
- MS/MS confirmation data for suspect analytes (e.g., amphetamine fragments m/z 91, 119, 136 at 500 ppb)

## Outputs

- linear and quadratic calibration-curve parameters per analyte with weighting
- adjusted R2 values for each fitted curve (e.g., heroin: 0.9873 linear, 0.9983 quadratic)
- estimated concentrations and relative errors for the test sample per analyte
- calibration_results.csv containing curve parameters, adjusted R2 values, estimated concentrations, and relative errors

## How to apply

Load the calibration example data (e.g., the narcotics data shipped with SMART 2.0 at https://github.com/YuJenL/SMART) into R and invoke the SMART 2.0 concentration calibration module to fit both linear and quadratic calibration curves with weighting to the standard series. Apply the module's outlier-removal procedure based on Cook's distance and confidence intervals, then refit the curves after outlier exclusion. Validate the fits by checking adjusted R2 values against reported expectations (for heroin: 0.9873 linear, 0.9983 quadratic). Apply the fitted curves to the test sample (e.g., the 500 ppb narcotics sample) to estimate concentrations and compute relative errors per analyte (morphine -0.0891%, amphetamine 54.7643% in the reference run), then write curve parameters, adjusted R2, and estimated concentrations with relative errors to a results file such as calibration_results.csv. Interpret large relative errors (e.g., amphetamine's 54.7643%, whose S/N ratio was only 0.19) as a signal to re-evaluate LC-MS experimental conditions or peak detection parameters rather than the regression itself.

## Related tools

- **SMART 2.0** (Provides the concentration calibration module that fits weighted linear and quadratic curves, performs Cook's-distance-based outlier removal, and estimates test-sample concentrations) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Base integrated R tool upon which SMART 2.0's concentration calibration module is built) — https://github.com/YuJenL/SMART
- **R** (Execution environment in which the SMART calibration workflow is run and calibration_results.csv is generated)

## Examples

```
Rscript -e 'SMART::concentration_calibration(narcotics_data, models=c("linear","quadratic"), weighting=TRUE, outlier_removal=TRUE); write.csv(calibration_results, "calibration_results.csv")'
```

## Evaluation signals

- Adjusted R2 for heroin must equal 0.9873 for the linear curve and 0.9983 for the quadratic curve after outlier removal and refitting.
- Relative error for morphine in the 500 ppb test sample must equal -0.0891%.
- Relative error for amphetamine in the 500 ppb test sample must equal 54.7643%; a large positive deviation should trigger investigation of peak detection parameters and LC-MS conditions.
- calibration_results.csv exists and contains curve parameters, adjusted R2 values, estimated concentrations, and relative errors for the analytes.

## Limitations

- Amphetamine's S/N ratio in the narcotics data set was only 0.19, producing the largest relative error (54.7643%); LC-MS experimental conditions and peak detection parameters for amphetamine need re-evaluation to improve quantification accuracy.
- Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve but is not guaranteed; variations were observed.
- Increasing the number of experimental replicates is recommended to enhance calibration and quantification accuracy; low replicate counts limit reliability.
- SMART 2.0 could not detect heroin and nimetazepam in the narcotics data, and HMDB does not encompass them.

## Evidence

- [abstract] SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [discussion] S/N ratio of amphetamine in the narcotics data set was 0.19: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] Consistency of LC-MS experimental conditions required for reliable calibration curves: "Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification"
- [discussion] More experimental replicates recommended to enhance calibration/quantification accuracy: "Increasing the number of replicates in future experiments is recommended to enhance accuracy"
- [readme] README: post-analysis module executes peak identification and concentration calibration: "Post-analysis: Execute peak identification and concentration calibration."
