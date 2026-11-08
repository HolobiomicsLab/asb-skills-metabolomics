---
name: narcotic-concentration-quantification
description: Use when a standard concentration series (e.g., narcotics at multiple
  ppb levels) plus a test sample of known target concentration (e.g., 500 ppb) is
  available and concentrations must be quantified via calibration-curve regression
  in R.
when_to_use_negative:
- The goal is untargeted peak detection or peak annotation rather than targeted quantification
  against a standard series.
- The test sample's true concentration is unknown, so no relative error can be computed
  to validate the calibration.
- The standard series lacks the concentration levels needed to build calibration curves
  for the analytes of interest.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMART 2.0
  role: Fits weighted linear and quadratic calibration curves, performs Cook's-distance
    outlier removal, and predicts test-sample concentrations via its concentration
    calibration module
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Predecessor tool whose untargeted analysis workflow SMART 2.0 extends with
    targeted modules including concentration calibration
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which the SMART 2.0 example data are loaded and the
    calibration workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:1d492188e1235b1baaec8d987f70a318d9dd5ee6a0354736c4d1d079bdab7c2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# narcotic-concentration-quantification

## Summary

Estimates analyte concentrations in a targeted LC-MS metabolomics test sample by fitting weighted linear and quadratic calibration curves to a standard concentration series with SMART 2.0's concentration calibration module, then reporting relative errors against a known target concentration.

## When to use

Use when a standard concentration series (e.g., narcotics at multiple ppb levels) plus a test sample of known target concentration (e.g., 500 ppb) is available and concentrations must be quantified via calibration-curve regression in R.

## When NOT to use

- The goal is untargeted peak detection or peak annotation rather than targeted quantification against a standard series.
- The test sample's true concentration is unknown, so no relative error can be computed to validate the calibration.
- The standard series lacks the concentration levels needed to build calibration curves for the analytes of interest.

## Inputs

- Narcotics calibration example data from the YuJenL/SMART GitHub repository (LC-MS peak areas for a 12-narcotic standard concentration series)
- Test sample measurements at a known target concentration (500 ppb)
- SMART 2.0 R tool (concentration calibration module)

## Outputs

- Calibration-curve parameters and adjusted R2 values for linear and quadratic fits per narcotic
- Estimated concentrations of the 500 ppb test sample with relative errors per narcotic (e.g., morphine -0.0891%, amphetamine 54.7643%)
- calibration_results.csv

## How to apply

Load the narcotics calibration example data shipped with the YuJenL/SMART GitHub repository into R, then invoke SMART 2.0's concentration calibration module to fit both linear and quadratic calibration curves with weighting to the narcotic standard series for each of the 12 narcotics. Apply the module's outlier-removal procedure based on Cook's distance and confidence intervals, and refit the curves after outlier exclusion. Verify curve quality by checking adjusted R2 against reported values (for heroin: 0.9873 linear, 0.9983 quadratic), then use the fitted curves to predict the 500 ppb test sample concentration per narcotic and compute relative errors against the known target. The rationale is that weighted fitting plus Cook's-distance outlier exclusion improves calibration reliability, but low-S/N analytes (e.g., amphetamine, S/N 0.19) can still yield large relative errors (54.7643%) even when the fit itself is strong.

## Related tools

- **SMART 2.0** (Fits weighted linear and quadratic calibration curves, performs Cook's-distance outlier removal, and predicts test-sample concentrations via its concentration calibration module) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Predecessor tool whose untargeted analysis workflow SMART 2.0 extends with targeted modules including concentration calibration) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which the SMART 2.0 example data are loaded and the calibration workflow is executed)

## Evaluation signals

- Adjusted R2 for the heroin calibration curves must equal 0.9873 (linear) and 0.9983 (quadratic) after outlier removal.
- Relative error for morphine in the 500 ppb test sample must equal -0.0891%.
- Relative error for amphetamine in the 500 ppb test sample must equal 54.7643%.
- calibration_results.csv contains curve parameters, adjusted R2 values, and estimated concentrations with relative errors for the 12 narcotics.
- Refitted curves after Cook's-distance outlier exclusion should show adjusted R2 consistent with (i.e., not worse than) the reported values.

## Limitations

- Low signal-to-noise analytes are unreliable: the S/N ratio of amphetamine in the narcotics data set was only 0.19, and MS/MS showed both MS1 peaks at 500 ppb produce identical fragments (m/z 91, 119, 136), so amphetamine quantification accuracy needs re-evaluation of LC-MS experimental conditions and peak-detection parameters.
- Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve, but variations were observed and cannot be guaranteed.
- More experimental replicates are recommended to enhance calibration/quantification accuracy.

## Evidence

- [abstract] SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [abstract] Codes, user guide, example data location: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [discussion] Low S/N of amphetamine: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] Calibration curve reliability depends on consistent LC-MS conditions: "Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification"
- [discussion] More replicates recommended: "Increasing the number of replicates in future experiments is recommended to enhance accuracy"
