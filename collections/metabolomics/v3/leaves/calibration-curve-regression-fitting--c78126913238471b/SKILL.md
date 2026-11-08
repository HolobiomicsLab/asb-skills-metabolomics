---
name: calibration-curve-regression-fitting
description: Use when a targeted metabolomics dataset contains calibration points,
  QC samples, and internal standard intensities and absolute quantification is needed
  from signal intensities.
when_to_use_negative:
- Analysis is untargeted (feature-table processing, pooled-QC filtration, annotation
  confidence levels) rather than calibration-curve-based quantification
- Dataset lacks calibration points or internal standard intensities, so absolute quantification
  from response vs. concentration curves cannot be fitted
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GetFeatistics
  role: Provides the calibration curve fitting functions with selectable 1/Y, 1/X,
    1/X² weighting and internal-standard mapping for targeted quantification
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language environment in which GetFeatistics calibration fitting is executed
    (≥ 4.3.1 required)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:a381af302ae49078f6d671847366da2913d4a9068ccc865ce08ed97562a25a58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# calibration-curve-regression-fitting

## Summary

Fits calibration curves with user-selectable weighting schemes (1/Y, 1/X, 1/X²) in GetFeatistics to back-calculate analyte concentrations from signal intensities in targeted metabolomics, returning regression summaries and accuracy (%) for standards and QCs.

## When to use

Use when a targeted metabolomics dataset contains calibration points, QC samples, and internal standard intensities and absolute quantification is needed from signal intensities.

## When NOT to use

- Analysis is untargeted (feature-table processing, pooled-QC filtration, annotation confidence levels) rather than calibration-curve-based quantification
- Dataset lacks calibration points or internal standard intensities, so absolute quantification from response vs. concentration curves cannot be fitted

## Inputs

- Targeted metabolomics dataset in R containing calibration-point intensities and concentrations
- QC sample intensities
- Internal standard intensities
- Input metadata mapping each analyte to its internal standard

## Outputs

- Per-analyte, per-weighting table of slope, intercept, R-squared, and adjusted R-squared
- Back-calculated concentrations with accuracy (%) for standards and QCs
- calibration_results.csv

## How to apply

Load the targeted dataset (calibration points, QC samples, and internal standard intensities) in R with GetFeatistics, and map each analyte to its internal standard via the input metadata. Fit an ordinary least squares calibration model of response vs. concentration per analyte, then refit with the weighted least squares schemes provided by GetFeatistics (1/Y, 1/X, 1/X²), which reduce bias at low concentrations — choose the weighting by comparing which scheme yields back-calculated accuracies closest to 100% for low-level standards and QCs. Compute slope, intercept, R-squared, and adjusted R-squared for each fitted curve, and back-calculate concentrations in calibration points and QC samples to obtain accuracy (%) per fit. Write the combined per-analyte, per-weighting results to calibration_results.csv.

## Related tools

- **GetFeatistics** (Provides the calibration curve fitting functions with selectable 1/Y, 1/X, 1/X² weighting and internal-standard mapping for targeted quantification) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language environment in which GetFeatistics calibration fitting is executed (≥ 4.3.1 required))

## Examples

```
library(GetFeatistics); calibration_results <- fit_calibration(intensity_concentration_data, weighting = "1/X"); write.csv(calibration_results, "calibration_results.csv")
```

## Evaluation signals

- Weighted fits return regression summaries with slope, intercept, R-squared, and adjusted R-squared populated for every analyte
- Back-calculated accuracy (%) for calibration standards and QCs is reported per fit; weighting scheme reducing bias at low concentrations shows accuracy closer to nominal at low levels
- Output table contains one row-set per analyte per weighting scheme (1/Y, 1/X, 1/X²) plus the OLS reference fit
- Combined results are written to calibration_results.csv with per-analyte, per-weighting structure

## Limitations

- Weighting scheme choice (1/Y, 1/X, 1/X²) is user-selectable; the package does not enforce one scheme, so inappropriate weighting can bias back-calculated concentrations
- Internal-standard mapping depends on correct input metadata; mis-mapping analytes to internal standards corrupts quantification

## Evidence

- [abstract] For targeted analyses, the package enables calibration curve-based quantification with different data weighting options.: "For targeted analyses, the package enables calibration curve-based quantification with different data weighting options."
- [results] GetFeatistics implements targeted quantification by fitting calibration curves with user-selectable weighting schemes (1/Y, 1/X, or 1/X2) to reduce bias at low concentrations: [evidence span withheld — anchor status: not_found]
- [other] workflow step: refit calibration models with weighting schemes and write calibration_results.csv: [evidence span withheld — anchor status: not_found]
- [readme] README installation requirements for running GetFeatistics in R: "Before installing, ensure the following are installed:
- R (version ≥ 4.3.1)
- Java, with the same architecture as R (64-bit or 32-bit).
- git"
- [other] workflow: map analytes to internal standards and compute accuracy (%) for standards and QCs: [evidence span withheld — anchor status: not_found]
