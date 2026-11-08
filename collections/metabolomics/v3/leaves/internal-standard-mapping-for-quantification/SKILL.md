---
name: internal-standard-mapping-for-quantification
description: Use when a targeted metabolomics dataset contains calibration points,
  QC samples, and internal standard intensities, and the goal is absolute analyte
  concentrations rather than relative feature intensities.
when_to_use_negative:
- Input is an untargeted feature table (e.g. imported from patRoon or MS-DIAL) where
  no calibration points or internal standards exist — QC-filtering and regression
  modelling skills apply instead
- The analysis goal is relative intensity comparison or pooled-QC quality monitoring
  rather than absolute quantification
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: R package providing the calibration curve fitting functions with selectable
    weighting (1/Y, 1/X, 1/X²), internal-standard mapping, and accuracy (%) calculation
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language/environment in which the targeted quantification workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:a2a4715f8b4b7d37609116936e75b03fd4cff7c544435dad8d15c9e91cf054de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# internal-standard-mapping-for-quantification

## Summary

Method for absolute quantification in targeted metabolomics: map each analyte to its internal standard, fit calibration curves (with 1/Y, 1/X, or 1/X² weighting), and back-calculate concentrations and accuracy (%) for standards and QCs using GetFeatistics in R.

## When to use

Use when a targeted metabolomics dataset contains calibration points, QC samples, and internal standard intensities, and the goal is absolute analyte concentrations rather than relative feature intensities.

## When NOT to use

- Input is an untargeted feature table (e.g. imported from patRoon or MS-DIAL) where no calibration points or internal standards exist — QC-filtering and regression modelling skills apply instead
- The analysis goal is relative intensity comparison or pooled-QC quality monitoring rather than absolute quantification

## Inputs

- Targeted metabolomics dataset in R containing calibration-point intensities and nominal concentrations
- QC sample intensities
- Internal standard intensities per analyte
- Input metadata mapping each analyte to its internal standard

## Outputs

- Per-analyte, per-weighting calibration table with slope, intercept, R-squared, and adjusted R-squared (e.g. calibration_results.csv)
- Back-calculated concentrations in calibration points and QC samples
- Accuracy (%) values for standards and QCs

## How to apply

Load the targeted dataset (calibration points, QC samples, and internal standard intensities) in R and map each analyte to its internal standard as specified in the input metadata. For each analyte, fit an ordinary least squares calibration model of response vs. concentration with GetFeatistics calibration functions, then refit with weighted least squares using the package's selectable schemes 1/Y, 1/X, and 1/X² — weighting reduces bias at low concentrations. Compute slope, intercept, R-squared (and adjusted R-squared) per fitted curve, then back-calculate concentrations in calibration points and QC samples to obtain accuracy (%) for each fit. Combine per-analyte, per-weighting summaries (e.g. write calibration_results.csv) to compare weighting schemes before choosing one.

## Related tools

- **GetFeatistics** (R package providing the calibration curve fitting functions with selectable weighting (1/Y, 1/X, 1/X²), internal-standard mapping, and accuracy (%) calculation) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language/environment in which the targeted quantification workflow is executed)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Fitted regression summaries include slope, intercept, and R-squared for every analyte and every weighting scheme (1/Y, 1/X, 1/X²)
- Accuracy (%) back-calculated for calibration standards and QCs falls within expected quantitative-acceptance ranges for the chosen weighting scheme
- Every analyte row is paired with an internal standard from the input metadata (no unmapped analytes)
- Combined output table is written per-analyte and per-weighting (e.g. calibration_results.csv) and parses cleanly in R

## Limitations

- Bias reduction via weighting is specific to low concentrations; weighting choice must be justified per analyte by comparing accuracy (%) across schemes
- Requires calibration points, QC samples, and internal standards to be present — pure intensity-only feature tables cannot be quantified absolutely

## Evidence

- [full_text] GetFeatistics implements targeted quantification by fitting calibration curves with user-selectable weighting schemes (1/Y, 1/X, or 1/X2) to reduce bias at low concentrations: [evidence span withheld — anchor status: not_found]
- [abstract] For targeted analyses, the package enables calibration curve-based quantification with different data weighting options.: "For targeted analyses, the package enables calibration curve-based quantification with different data weighting options."
- [readme] Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing, advanced statistics: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing, advanced statistics such as multiple regression linear"
