---
name: back-calculated-accuracy-evaluation-of-qc-samples
description: Use when a targeted metabolomics dataset contains calibration points,
  QC samples, and internal standard intensities and you must verify quantification
  accuracy (%) by back-calculation from fitted calibration curves.
when_to_use_negative:
- Input is an untargeted feature table requiring pooled-QC filtration or annotation
  confidence assignment rather than calibration-curve quantification
- No calibration standards or internal standards are available, so response-vs-concentration
  curves cannot be fitted
- The goal is longitudinal or censored-data regression (mixed-effects/Tobit models)
  rather than calibration-curve quantification
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GetFeatistics
  role: R-package providing calibration curve-based quantification with selectable
    weighting schemes (1/Y, 1/X, 1/X²), internal standard mapping, and accuracy (%)
    back-calculation for targeted analyses
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language/runtime in which the calibration fitting, weighting, and back-calculation
    workflow is executed (R ≥ 4.3.1 required)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:d818546042a38d8b6db1de2dfef34929aa6649a5adeb7c413c7989bb345afab4
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
yes_rate: 0.9
n_claims: 10
---

# Back-calculated accuracy evaluation of QC samples

## Summary

Evaluates targeted metabolomics quantification quality by back-calculating concentrations of calibration standards and QC samples from fitted calibration curves and reporting accuracy (%), alongside regression summaries (slope, intercept, R-squared). Implemented in GetFeatistics with user-selectable weighting schemes (1/Y, 1/X, 1/X²).

## When to use

Use when a targeted metabolomics dataset contains calibration points, QC samples, and internal standard intensities and you must verify quantification accuracy (%) by back-calculation from fitted calibration curves.

## When NOT to use

- Input is an untargeted feature table requiring pooled-QC filtration or annotation confidence assignment rather than calibration-curve quantification
- No calibration standards or internal standards are available, so response-vs-concentration curves cannot be fitted
- The goal is longitudinal or censored-data regression (mixed-effects/Tobit models) rather than calibration-curve quantification

## Inputs

- Targeted metabolomics dataset with calibration points, QC samples, and internal standard intensities (R data loaded from user metadata)
- Analyte-to-internal-standard mapping specified in input metadata

## Outputs

- Combined per-analyte, per-weighting results table (calibration_results.csv) with slope, intercept, R-squared, adjusted R-squared
- Back-calculated concentrations and accuracy (%) for calibration standards and QC samples
- Fitted regression-model summaries for each weighting scheme

## How to apply

Load the targeted dataset (calibration points, QC samples, internal standard intensities) in R with GetFeatistics, and map each analyte to its internal standard per the input metadata. For each analyte, fit the calibration model of response vs. concentration, then refit with weighted least squares using the 1/Y, 1/X, or 1/X² weighting schemes — weighting reduces bias at low concentrations. Compute slope, intercept, R-squared (and adjusted R-squared) for each curve, then back-calculate concentrations in calibration points and QC samples to obtain accuracy (%) per fit. Combine per-analyte, per-weighting results into a single table (e.g. calibration_results.csv) for review.

## Related tools

- **GetFeatistics** (R-package providing calibration curve-based quantification with selectable weighting schemes (1/Y, 1/X, 1/X²), internal standard mapping, and accuracy (%) back-calculation for targeted analyses) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language/runtime in which the calibration fitting, weighting, and back-calculation workflow is executed (R ≥ 4.3.1 required))

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- calibration_results.csv exists with one row per analyte per weighting scheme (1/Y, 1/X, 1/X²)
- Each fitted curve reports slope, intercept, and R-squared; R-squared values close to 1 indicate acceptable calibration fits
- Back-calculated concentrations of standards and QCs are non-negative and paired with accuracy (%) values for every analyte
- Each analyte is matched to exactly one internal standard as declared in the input metadata
- Weighted fits (1/X or 1/X²) show reduced deviation at low concentrations compared with the unweighted ordinary least squares fit

## Limitations

- Weighting schemes are limited to the three options provided by the package (1/Y, 1/X, 1/X²); other weighting schemes are not supported
- Requires complete calibration points, QC samples, and internal standard intensities; absolute quantification is not possible for analytes lacking internal standards unless mapped otherwise
- Accuracy (%) reflects only back-calculation agreement within the fitted calibration range; extrapolation beyond calibrated concentrations is not validated

## Evidence

- [abstract] For targeted analyses, the package enables calibration curve-based quantification with different data weighting options.: "For targeted analyses, the package enables calibration curve-based quantification with different data weighting options."
- [other] GetFeatistics implements targeted quantification by fitting calibration curves with user-selectable weighting schemes (1/Y, 1/X, or 1/X2) to reduce bias at low concentrations, optionally mapping each analyte to its internal standard, and returning calculated concentrations plus accuracy (%) for standards and QCs, along with fitted regression-model summaries (slope, intercept, R-squared).: [evidence span withheld — anchor status: not_found]
- [abstract] GetFeatistics is an R-package developed to streamline the elaboration and statistical analysis of metabolomics data.: "GetFeatistics is an R-package developed to streamline the elaboration and statistical analysis of metabolomics data."
