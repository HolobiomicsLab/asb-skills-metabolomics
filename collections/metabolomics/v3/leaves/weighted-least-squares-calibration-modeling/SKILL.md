---
name: weighted-least-squares-calibration-modeling
description: Use when a targeted metabolomics dataset contains calibration points,
  QC samples and internal-standard intensities, and absolute concentrations must be
  back-calculated from signal intensities.
when_to_use_negative:
- Input is an untargeted feature table from tools like patRoon or MS-DIAL — use GetFeatistics'
  untargeted import, annotation-confidence and pooled-QC filtering workflow instead
- The dataset lacks calibration standards and QC samples, so no response-vs-concentration
  curve can be fit
- Only exploratory/statistical modelling (mixed-effects or Tobit regression on intensities)
  is needed, not absolute quantification
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: R-package providing the calibration-curve quantification functions with selectable
    1/Y, 1/X, 1/X² data weighting for targeted analyses
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language/runtime in which the calibration fitting, weighting and back-calculation
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:ed290b0948d81561073a457e2b8a2e49b4379aa2fca8087d05f003b821d54d08
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

# weighted-least-squares-calibration-modeling

## Summary

Targeted-metabolomics absolute quantification in GetFeatistics (R): fit calibration curves of response vs. concentration with user-selectable weighting schemes (1/Y, 1/X, 1/X²) to reduce bias at low concentrations, mapping each analyte to its internal standard.

## When to use

Use when a targeted metabolomics dataset contains calibration points, QC samples and internal-standard intensities, and absolute concentrations must be back-calculated from signal intensities.

## When NOT to use

- Input is an untargeted feature table from tools like patRoon or MS-DIAL — use GetFeatistics' untargeted import, annotation-confidence and pooled-QC filtering workflow instead
- The dataset lacks calibration standards and QC samples, so no response-vs-concentration curve can be fit
- Only exploratory/statistical modelling (mixed-effects or Tobit regression on intensities) is needed, not absolute quantification

## Inputs

- Targeted metabolomics dataset with calibration points, QC samples and internal-standard intensities (loaded in R)
- Input metadata mapping each analyte to its internal standard

## Outputs

- Per-analyte, per-weighting calibration table written to calibration_results.csv (slope, intercept, R², adjusted R²)
- Back-calculated concentrations and accuracy (%) for calibration standards and QCs
- Fitted regression-model summaries for each calibration curve

## How to apply

Load the targeted dataset (calibration points, QC samples, internal-standard intensities) into R after installing GetFeatistics from GitHub (requires R ≥ 4.3.1 and matching-architecture Java). Map each analyte to its internal standard as specified in the input metadata, then use the GetFeatistics calibration functions to fit an ordinary-least-squares calibration of response vs. concentration per analyte. Refit each curve with the package's 1/Y, 1/X, and 1/X² weighted-least-squares schemes — weighting is the rationale here, since heavier weighting of low-concentration points reduces bias where quantification matters most. For each fit, compute slope, intercept, R² (and adjusted R²), then back-calculate concentrations in calibration points and QC samples to obtain accuracy (%) per weighting scheme. Combine per-analyte, per-weighting results (including the fitted regression-model summaries) and write the table to calibration_results.csv.

## Related tools

- **GetFeatistics** (R-package providing the calibration-curve quantification functions with selectable 1/Y, 1/X, 1/X² data weighting for targeted analyses) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language/runtime in which the calibration fitting, weighting and back-calculation are executed)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Regression summaries report slope, intercept, R² (and adjusted R²) for both the OLS fit and each of the three weighted refits
- Accuracy (%) back-calculated for calibration standards and QCs is produced for every analyte × weighting combination
- Weighted fits (1/Y or 1/X-type) show reduced bias at low concentrations relative to the unweighted OLS fit
- Each analyte row is traceable to its mapped internal standard from the input metadata
- A combined per-analyte, per-weighting results table is written to calibration_results.csv

## Limitations

- Weighting schemes are restricted to the options GetFeatistics provides (1/Y, 1/X, 1/X²); other weighting functions are not supported by this path
- Applies only to targeted calibration-curve quantification; the package's other pipelines (patRoon/MS-DIAL feature import, pooled-QC filtering, mixed-effects/Tobit regression) address different goals

## Evidence

- [abstract] For targeted analyses, the package enables calibration curve-based quantification with different data weighting options.: "For targeted analyses, the package enables calibration curve-based quantification with different data weighting options."
- [abstract] GetFeatistics is an R-package developed to streamline the elaboration and statistical analysis of metabolomics data.: "GetFeatistics is an R-package developed to streamline the elaboration and statistical analysis of metabolomics data."
- [readme] README description of scope covering targeted and non-targeted elaboration and separate QC processing.: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
- [readme] README installation prerequisite constraining the runtime environment for the calibration functions.: [evidence span withheld — anchor status: not_found]
