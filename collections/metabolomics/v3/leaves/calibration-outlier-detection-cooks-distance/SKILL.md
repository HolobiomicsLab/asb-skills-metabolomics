---
name: calibration-outlier-detection-cooks-distance
description: Use when a test sample's concentration must be estimated from a multi-level
  analyte standard series and suspected calibration outliers need removal via Cook's
  distance before curve fitting.
when_to_use_negative:
- When the goal is untargeted peak identification or annotation (SMART 2.0's peak
  identification postanalysis module using HMDB/MassBank) rather than concentration
  quantification from a standard series
- When the calibration standard series lacks multiple concentration levels/replicates,
  since the article notes increasing replicates is recommended to enhance accuracy
  and the one-to-one identification caveat applies to peak matching, not curve fitting
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: SMART 2.0
  role: Concentration calibration module used to fit weighted linear and quadratic
    curves, apply Cook's-distance outlier removal, and estimate test-sample concentrations
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Predecessor tool whose untargeted analysis foundation SMART 2.0 extends with
    the concentration calibration module
  repo: https://github.com/YuJenL/SMART
- name: R
  role: Runtime environment in which SMART 2.0's calibration module is invoked and
    results (calibration_results.csv) are written
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:bc0c0f48115703cb0bbaa41a377d65141dd14e3f59be33cd1c378218c1c4d7d3
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# calibration-outlier-detection-cooks-distance

## Summary

Fits weighted linear and quadratic calibration curves for targeted metabolomics analytes (e.g., the 12 narcotics in SMART 2.0), removes calibration outliers using Cook's distance and confidence intervals, and refits the curves to quantify concentration-estimation accuracy via adjusted R2 and relative errors.

## When to use

Use when a test sample's concentration must be estimated from a multi-level analyte standard series and suspected calibration outliers need removal via Cook's distance before curve fitting.

## When NOT to use

- When the goal is untargeted peak identification or annotation (SMART 2.0's peak identification postanalysis module using HMDB/MassBank) rather than concentration quantification from a standard series
- When the calibration standard series lacks multiple concentration levels/replicates, since the article notes increasing replicates is recommended to enhance accuracy and the one-to-one identification caveat applies to peak matching, not curve fitting

## Inputs

- Narcotics calibration example data (standard series at multiple concentration levels) from the SMART 2.0 GitHub repository
- A test sample at a known concentration (e.g., 500 ppb) for each of the 12 narcotics
- R environment with SMART 2.0 installed

## Outputs

- Fitted linear and quadratic weighted calibration curves per analyte (with outlier removal via Cook's distance and confidence intervals)
- Adjusted R2 values per fitted curve (target: heroin 0.9873 linear, 0.9983 quadratic)
- Estimated concentrations and relative errors for the test sample (target: morphine −0.0891%, amphetamine 54.7643%)
- calibration_results.csv containing calibration-curve parameters, adjusted R2 values, estimated concentrations, and relative errors

## How to apply

Load the narcotics calibration example data shipped with the SMART 2.0 GitHub repository into R and invoke SMART 2.0's concentration calibration module to fit both linear and quadratic calibration curves with weighting to the narcotic standard series. Apply the module's outlier-removal procedure based on Cook's distance and confidence intervals, then refit the curves after outlier exclusion. Verify curve quality by checking adjusted R2 values against the article's reported values (heroin: 0.9873 linear, 0.9983 quadratic), which confirms outlier removal did not distort the fit. Finally, use the fitted curves to estimate the concentration of the 500 ppb test sample and compute relative errors; the article reports morphine as the best-estimated analyte (−0.0891%) and amphetamine as the worst (54.7643%), which serves as a known performance envelope. Rationale: outliers in the standard series can bias the weighted regression, so the Cook's-distance/confidence-interval screen plus refit protects downstream quantification.

## Related tools

- **SMART 2.0** (Concentration calibration module used to fit weighted linear and quadratic curves, apply Cook's-distance outlier removal, and estimate test-sample concentrations) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Predecessor tool whose untargeted analysis foundation SMART 2.0 extends with the concentration calibration module) — https://github.com/YuJenL/SMART
- **R** (Runtime environment in which SMART 2.0's calibration module is invoked and results (calibration_results.csv) are written)

## Evaluation signals

- Adjusted R2 of the fitted heroin curve must equal 0.9873 (linear) and 0.9983 (quadratic) after outlier removal
- Relative error for morphine in the 500 ppb test sample must equal −0.0891%
- Relative error for amphetamine in the 500 ppb test sample must equal 54.7643%
- Curves must be refitted after Cook's-distance/confidence-interval outlier exclusion, and the adjusted R2 values recorded for both linear and quadratic fits
- Output file calibration_results.csv must contain curve parameters, adjusted R2 values, estimated concentrations, and relative errors

## Limitations

- Amphetamine showed an S/N ratio of only 0.19 and a 54.7643% relative error; MS/MS at 500 ppb revealed two amphetamine MS1 peaks producing identical fragments (m/z 91, 119, 136), so peak-detection parameters and LC-MS conditions need re-evaluation to improve quantification accuracy
- Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve but is not guaranteed; variations were observed, and increasing replicates in future experiments is recommended to enhance accuracy

## Evidence

- [abstract] Concentration calibration module workflow and outlier removal: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [discussion] Amphetamine quantification failure mode: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] Identical fragments from amphetamine MS1 peaks: "MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136)"
- [discussion] Replicate recommendation for calibration accuracy: "Increasing the number of replicates in future experiments is recommended to enhance accuracy"
- [discussion] LC-MS condition consistency requirement for calibration curves: "Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification"
- [readme] Example data availability in the SMART repository: "Post-analysis: Execute peak identification and concentration calibration."
