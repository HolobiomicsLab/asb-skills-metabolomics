---
name: mass-defect-based-feature-grouping
description: Use when you have a blank-filtered feature table from PFAS LC-HRMS/MS
  data and want to discover homologous PFAS series (e.g., CF2 ladders) without a suspect
  list, including series lacking MS/MS corroboration.
when_to_use_negative:
- Raw LC-HRMS data has not yet been feature-detected and blank-filtered — run feature
  detection and feature blank filtering first
- You are performing suspect screening on compounds with available MS/MS spectra —
  the original FluoroMatch suspect screening approach is the appropriate route instead
- 'Sample matrix is not PFAS-amenable chemistry (e.g., lipidomics): set Lipid <- TRUE
  rather than relying on the PFAS-default CF2/50 Da logic'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FluoroMatch Modular
  role: Runs the homologous series detection module that computes CF2-normalized KMD
    grouping and RT-order validation on the feature table
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: Parent software whose automated non-targeted PFAS workflow includes homologous
    series detection as a pipeline step
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: Full data-processing pipeline (feature detection through MS/MS screening)
    that feeds and consumes the homologous-series step
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:35b952e384be3c9eea00e3824fe1b5601c7cefb279c72a1829f70bc513f3ecb1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-defect-based-feature-grouping

## Summary

Groups blank-filtered LC-HRMS features into CF2-normalized homologous series using Kendrick mass defect analysis in FluoroMatch 2.0, separating candidates whose mass differences are divisible by the 50 Da CF2 repeating unit and validating retention-time order.

## When to use

Use when you have a blank-filtered feature table from PFAS LC-HRMS/MS data and want to discover homologous PFAS series (e.g., CF2 ladders) without a suspect list, including series lacking MS/MS corroboration.

## When NOT to use

- Raw LC-HRMS data has not yet been feature-detected and blank-filtered — run feature detection and feature blank filtering first
- You are performing suspect screening on compounds with available MS/MS spectra — the original FluoroMatch suspect screening approach is the appropriate route instead
- Sample matrix is not PFAS-amenable chemistry (e.g., lipidomics): set Lipid <- TRUE rather than relying on the PFAS-default CF2/50 Da logic

## Inputs

- Blank-filtered feature table (CSV) from FluoroMatch feature detection and feature blank filtering
- Feature list with precursor m/z and retention times from LC-HRMS/MS acquisition of PFAS samples (e.g., AFFF)

## Outputs

- homologous_series_table.csv — series-membership table grouping features into CF2-normalized homologous series with ≥3 members
- Flags for candidate series lacking MS/MS corroboration, marked for expert review

## How to apply

Load the blank-filtered feature table from the upstream feature detection/blank filtering steps into FluoroMatch Modular's homologous series detection module. Compute the Kendrick mass defect (KMD) for every feature normalized to the CF2 repeating unit (50 Da), then group features whose KMDs agree within 0.005 Da and whose pairwise mass differences are divisible by 50 Da. Validate each candidate series by checking that retention time increases in the correct order along the series, flagging any series that violates RT-order logic. Retain only series with at least 3 members and write the membership table to homologous_series_table.csv; on the legacy AFFF sample this yielded 50 such series, with 13 lacking MS/MS corroboration flagged for expert review. This step complements exact mass matching to the EPA CompTox PFAS list (8000+ structures) and mass defect filtering within the FluoroMatch Flow 2.0 pipeline.

## Related tools

- **FluoroMatch Modular** (Runs the homologous series detection module that computes CF2-normalized KMD grouping and RT-order validation on the feature table) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (Parent software whose automated non-targeted PFAS workflow includes homologous series detection as a pipeline step) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (Full data-processing pipeline (feature detection through MS/MS screening) that feeds and consumes the homologous-series step) — github.com/InnovativeOmics/Core-Match

## Examples

```
FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE
```

## Evaluation signals

- Every retained series has at least 3 member features (series with <3 members are dropped)
- All members of a series share CF2-normalized Kendrick mass defect within 0.005 Da
- Pairwise mass differences within each series are divisible by 50 Da (the CF2 unit)
- Retention time increases monotonically in the correct order along each retained series; violations are flagged
- On a legacy AFFF sample, approximately 50 series with ≥3 members should be recovered, with series lacking MS/MS corroboration flagged for expert review

## Limitations

- Series lacking MS/MS corroboration (13 of the 50 detected in the legacy AFFF sample) remain tentative and require expert review
- Series violating retention-time-order logic are flagged but must be manually adjudicated
- The grouping logic is specific to fluorinated-chain chemistry (CF2 repeating unit); it will not generalize to non-fluorinated homologous ladders without changing the repeating-unit normalization

## Evidence

- [other] Homologous-series detection groups features with identical CF2-normalized Kendrick mass defect (within 0.005 Da) separated by CF2-unit mass differences (divisible by 50 Da), verified by retention time order: [evidence span withheld — anchor status: not_found]
