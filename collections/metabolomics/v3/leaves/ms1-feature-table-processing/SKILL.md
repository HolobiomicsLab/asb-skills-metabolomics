---
name: ms1-feature-table-processing
description: Use when a raw vendor MS1 feature table (sample plus ≥3 solvent-blank
  feature tables) must be de-noised prior to exact mass matching, mass defect filtering,
  or homologous-series detection for PFAS.
when_to_use_negative:
- The feature table has already been blank-filtered against the same solvent blanks
  (re-filtering would over-remove genuine low-abundance PFAS).
- No solvent-blank feature tables are available, so the blank average + 3× SD comparison
  cannot be computed.
- The analysis goal is MS/MS-based annotation or scoring of already-retained features
  rather than removal of background MS1 features.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FluoroMatch Modular
  role: Runs the blank-filtering workflow that aligns sample and blank feature tables
    at ±0.005 Da and removes features whose sample maximum does not exceed blank average
    + 3× blank SD
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: Software suite whose feature blank filtering step this skill implements; part
    of FluoroMatch Flow 2.0 data processing for PFAS discovery
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:77d55dae155a7cda69f05cb4c351db4ff26745fa4d30c6c3350ea1996c73a5a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ms1-feature-table-processing

## Summary

Blank-filter an LC-HRMS/MS vendor feature table against solvent blanks to remove background/non-sample features before PFAS annotation, as implemented in the FluoroMatch 2.0 feature blank filtering step. On the AFFF sample this step reduced 31,132 detected features to 3740 (≈88% removed).

## When to use

Use when a raw vendor MS1 feature table (sample plus ≥3 solvent-blank feature tables) must be de-noised prior to exact mass matching, mass defect filtering, or homologous-series detection for PFAS.

## When NOT to use

- The feature table has already been blank-filtered against the same solvent blanks (re-filtering would over-remove genuine low-abundance PFAS).
- No solvent-blank feature tables are available, so the blank average + 3× SD comparison cannot be computed.
- The analysis goal is MS/MS-based annotation or scoring of already-retained features rather than removal of background MS1 features.

## Inputs

- Sample vendor feature table (e.g. AFFF sample, 31,132 features)
- Three solvent-blank vendor feature tables
- Precursor mass tolerance setting (±0.005 Da)

## Outputs

- blank_filtered_features.csv with retained sample features (3740 rows for the AFFF reference run)
- Count of removed background features (≈88% of detected features)

## How to apply

Load the sample vendor feature table and the three solvent-blank vendor feature tables into FluoroMatch Modular's blank-filtering workflow (set FLOW <- FALSE and use csv inputs). Align features between sample and blanks using a precursor mass tolerance of ±0.005 Da. For each feature, compute the mean and standard deviation of abundance across the three solvent blanks and flag it if the sample maximum is not greater than blank average + 3× blank SD; remove all flagged features. Rationale: blank filtering is the first post-detection step in the FluoroMatch Flow 2.0 pipeline, so downstream steps (exact mass matching to catalogued PFAS, mass defect filtering, homologous series detection) operate only on sample-derived features. Export the survivors and confirm the reduction matches the reference outcome (31,132 → 3740, ≈88% removed).

## Related tools

- **FluoroMatch Modular** (Runs the blank-filtering workflow that aligns sample and blank feature tables at ±0.005 Da and removes features whose sample maximum does not exceed blank average + 3× blank SD) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (Software suite whose feature blank filtering step this skill implements; part of FluoroMatch Flow 2.0 data processing for PFAS discovery) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r  # with FLOW <- FALSE; csvInput <- TRUE to run the blank-filtering workflow on sample + 3 solvent-blank vendor feature tables
```

## Evaluation signals

- Exported blank_filtered_features.csv contains exactly 3740 rows for the AFFF reference sample.
- Removed fraction matches the reported blank-filtering outcome (≈88% of 31,132 features removed).
- Every retained feature satisfies sample maximum > blank average + 3× blank SD across the three solvent blanks.
- Feature alignment between sample and blanks was performed at the specified ±0.005 Da precursor mass tolerance.
- Downstream steps (exact mass matching, mass defect filtering, homologous series detection) run only on the blank-filtered table, not the full 31,132-feature table.

## Limitations

- Blank filtering relies on only three solvent blanks, so the mean and SD estimates are noisy; borderline features near the 3× SD threshold are sensitive to blank variability.
- Blank filtering removes features correlated with background but does not itself distinguish PFAS from other sample components; FluoroMatch 2.0 reports filtering 96% of features likely not PFAS via the broader pipeline, not this step alone.
- Only vendor feature tables paired with matching solvent-blank tables are supported; unmatched blank/sample formats break the ±0.005 Da alignment.

## Evidence
