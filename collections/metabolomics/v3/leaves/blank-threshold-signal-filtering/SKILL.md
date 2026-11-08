---
name: blank-threshold-signal-filtering
description: Use when a vendor feature table from a real sample (e.g., AFFF) must
  be de-noised against matched solvent-blank feature tables before PFAS annotation,
  to strip features attributable to background.
when_to_use_negative:
- The input is already a blank-filtered feature table — reapplying the sample-vs-blank
  threshold would only remove features that legitimately exceed background.
- The goal is annotation or identification of retained features — blank filtering
  is only the second step of FluoroMatch Flow 2.0, which continues with exact mass
  matching, mass defect filtering, and homologous series detection.
- No matched solvent-blank injections exist for the sample acquisition, since the
  threshold is computed from blank average and SD across blanks.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FluoroMatch 2.0
  role: Automated non-targeted PFAS annotation software whose pipeline includes the
    feature blank-filtering step this skill implements
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular distribution containing the blank-filtering workflow applied to the
    sample and solvent-blank vendor feature tables
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch
  role: Original FluoroMatch release whose suspect-screening approach FluoroMatch
    2.0's filtering workflow extends
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:20f0648f6b1e44c505764141755c37f3945541e3136b04aecbb4ee46e88191e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# blank-threshold-signal-filtering

## Summary

Removes background/sample-container features from LC-HRMS feature tables by comparing each sample feature's maximum abundance against the average plus 3× standard deviation of that feature across solvent-blank injections, as implemented in the FluoroMatch 2.0 blank-filtering workflow.

## When to use

Use when a vendor feature table from a real sample (e.g., AFFF) must be de-noised against matched solvent-blank feature tables before PFAS annotation, to strip features attributable to background.

## When NOT to use

- The input is already a blank-filtered feature table — reapplying the sample-vs-blank threshold would only remove features that legitimately exceed background.
- The goal is annotation or identification of retained features — blank filtering is only the second step of FluoroMatch Flow 2.0, which continues with exact mass matching, mass defect filtering, and homologous series detection.
- No matched solvent-blank injections exist for the sample acquisition, since the threshold is computed from blank average and SD across blanks.

## Inputs

- Sample vendor feature table (e.g., AFFF sample; 31,132 detected features)
- Three solvent-blank vendor feature tables matched to the sample acquisition
- Precursor mass alignment tolerance (±0.005 Da)

## Outputs

- Blank-filtered feature table (e.g., blank_filtered_features.csv) containing only features exceeding blank average + 3× blank SD (3740 rows in the AFFF reference case)
- Removed-feature fraction (~88% of detected features in the AFFF reference case)

## How to apply

Load the sample vendor feature table and the solvent-blank vendor feature tables into FluoroMatch Modular's blank-filtering workflow. Align features between sample and blank files using a precursor mass tolerance of ±0.005 Da so that the same m/z-retention entries are compared across injections. For each feature, compute the average and standard deviation of abundance across the three solvent blanks and flag any feature whose sample maximum is not greater than blank average + 3× blank SD. Remove all flagged features from the sample feature table and export the survivors (e.g., as blank_filtered_features.csv); in the AFFF reference application this reduced 31,132 detected features to 3740, an ~88% removal. Blank filtering is the second step of the broader FluoroMatch Flow 2.0 pipeline, which proceeds to exact mass matching, mass defect filtering, and homologous series detection.

## Related tools

- **FluoroMatch 2.0** (Automated non-targeted PFAS annotation software whose pipeline includes the feature blank-filtering step this skill implements) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular distribution containing the blank-filtering workflow applied to the sample and solvent-blank vendor feature tables) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch** (Original FluoroMatch release whose suspect-screening approach FluoroMatch 2.0's filtering workflow extends) — github.com/InnovativeOmics/Core-Match

## Evaluation signals

- The exported blank-filtered table contains exactly 3740 rows when run on the AFFF reference sample.
- The removed fraction matches the reported blank-filtering outcome (~88% of the 31,132 detected features removed).
- Every retained feature satisfies sample maximum > blank average + 3× blank SD across the three solvent blanks.
- Sample and blank features were aligned within the ±0.005 Da precursor mass tolerance before thresholding.
- No features flagged above the blank threshold remain in the exported table.

## Limitations

- The blank threshold (blank average + 3× SD over three solvent blanks) depends on blank replicate count; with only three blanks the SD estimate is unstable.
- Blank filtering alone removed 88% of features in the AFFF case; the paper reports 96% of features likely not PFAS filtered out overall, so additional FluoroMatch filters are needed to reach that level.
- Per the Core-Match README, users should download the latest stable release from innovativeomics.com/software; the GitHub repository is the developer integration code rather than a stable distribution.

## Evidence

- [abstract] Workflow steps include feature blank filtering as a distinct pipeline stage: "These steps include feature detection, feature blank filtering, exact mass matching to catalogued PFAS, mass defect filtering, homologous series detection"
