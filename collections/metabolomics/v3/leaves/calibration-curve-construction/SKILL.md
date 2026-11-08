---
name: calibration-curve-construction
description: Use when you have a multi-level standard series plus a test sample (e.g.,
  a 500 ppb narcotic sample) and need to estimate analyte concentrations via fitted
  calibration curves with outlier handling and quality checks.
when_to_use_negative:
- When the goal is untargeted peak detection or peak annotation rather than quantification
  of known analytes against standards
- When no multi-level standard series is available to construct a calibration curve,
  or LC-MS experimental conditions were inconsistent across the standard series
- When the analyte of interest cannot be reliably detected in the data (e.g., SMART
  2.0 could not detect heroin and nimetazepam in the narcotics example)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMART 2.0
  role: Provides the concentration calibration module that fits weighted linear/quadratic
    curves, removes outliers via Cook's distance, and predicts test-sample concentrations
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Prior integrated untargeted metabolomics tool on which SMART 2.0's concentration
    calibration module builds
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Environment in which SMART 2.0 is run to load calibration data and execute
    the calibration workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:7e82a3fce39b985490020f73a39cbdd4bda2c8a46b85427b7a8816fd331a9c30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - R
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# calibration-curve-construction

## Summary

Fits weighted linear and quadratic calibration curves to serially diluted analyte standards (e.g., the 12 narcotics in SMART 2.0's example data) using SMART 2.0's concentration calibration module, applies Cook's-distance-based outlier removal, and predicts concentrations of unknown samples with relative errors.

## When to use

Use when you have a multi-level standard series plus a test sample (e.g., a 500 ppb narcotic sample) and need to estimate analyte concentrations via fitted calibration curves with outlier handling and quality checks.

## When NOT to use

- When the goal is untargeted peak detection or peak annotation rather than quantification of known analytes against standards
- When no multi-level standard series is available to construct a calibration curve, or LC-MS experimental conditions were inconsistent across the standard series
- When the analyte of interest cannot be reliably detected in the data (e.g., SMART 2.0 could not detect heroin and nimetazepam in the narcotics example)

## Inputs

- Calibration example data from the SMART 2.0 GitHub repository (narcotics standard series across concentration levels)
- Test sample LC-MS peak intensities at a known nominal concentration (500 ppb in the example)
- R session with SMART 2.0 installed

## Outputs

- Fitted linear and quadratic calibration-curve parameters per analyte
- Adjusted R2 values for each fitted curve (e.g., heroin: 0.9873 linear, 0.9983 quadratic)
- Estimated concentrations with relative errors for the test sample (e.g., morphine -0.0891%, amphetamine 54.7643%)
- calibration_results.csv

## How to apply

Load the calibration example data (e.g., the narcotics data shipped with the SMART 2.0 GitHub repository) into R and invoke SMART 2.0's concentration calibration module, a postanalysis task. Fit both linear and quadratic calibration curves with weighting to the standard series for each of the 12 narcotics. Apply the module's outlier-removal procedure based on Cook's distance and confidence intervals, then refit the curves after excluding the outliers. Verify curve quality via adjusted R2 (for heroin in the example: 0.9873 linear, 0.9983 quadratic) before using the fitted curves to estimate the concentration of the test sample (500 ppb) and compute relative errors (morphine -0.0891%, amphetamine 54.7643%). Write curve parameters, adjusted R2 values, and estimated concentrations with relative errors to an output file such as calibration_results.csv.

## Related tools

- **SMART 2.0** (Provides the concentration calibration module that fits weighted linear/quadratic curves, removes outliers via Cook's distance, and predicts test-sample concentrations) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Prior integrated untargeted metabolomics tool on which SMART 2.0's concentration calibration module builds) — https://github.com/YuJenL/SMART
- **R** (Environment in which SMART 2.0 is run to load calibration data and execute the calibration workflow)

## Examples

```
Rscript -e 'source("SMART_concentration_calibration.R"); run_calibration(narcotics_data, test_ppb=500, models=c("linear","quadratic"), write_csv="calibration_results.csv")'
```

## Evaluation signals

- Adjusted R2 for the heroin calibration curve matches reported values: 0.9873 (linear) and 0.9983 (quadratic)
- Relative error for morphine in the 500 ppb test sample equals -0.0891%
- Relative error for amphetamine in the 500 ppb test sample equals 54.7643%
- Outlier removal step (Cook's distance + confidence intervals) triggers refitting of curves after exclusion, changing the adjusted R2 relative to the initial fit
- Output file calibration_results.csv contains curve parameters, adjusted R2 values, and estimated concentrations with relative errors for the 12 narcotics

## Limitations

- Low-abundance analytes give unreliable quantification: the S/N ratio of amphetamine in the narcotics data set was only 0.19, contributing to its large relative error (54.7643%)
- Consistency in LC-MS experimental conditions across the standard series is crucial for constructing a reliable calibration curve and achieving accurate quantification, but variations were observed
- More experimental replicates are recommended to enhance calibration and quantification accuracy
- SMART 2.0 was unable to detect heroin and nimetazepam, and HMDB does not include them, limiting downstream identification/quantification for such analytes

## Evidence

- [abstract] SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [intro] we significantly enhance the capacities of SMART 1.0 by introducing advanced analytical modules such as targeted data analysis, omics data integration, and concentration calibration: "we significantly enhance the capacities of SMART 1.0 by introducing advanced analytical modules such as targeted data analysis, omics data integration, and concentration calibration"
- [discussion] the S/N ratio of amphetamine was found to be only 0.19: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification: "Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification"
- [readme] The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART.: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
