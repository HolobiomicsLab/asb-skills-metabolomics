---
name: retention-time-shift-correction
description: Use when aligning XCMS-derived feature lists or target chromatograms
  from different LC-HRMS batches where retention times are shifted (e.g. 7–52 s or
  -9 to -24 s ranges) and m/z deviates slightly (e.g. -3.4 ppm) relative to a reference.
when_to_use_negative:
- Input is already an aligned, combined feature matrix with corrected retention times
  — re-alignment would not apply
- Data were measured in a single batch against its own reference, so no cross-experiment
  m/z or retention-time shift exists
- Target files contain no reference-metabolite-derived features in the expected windows
  (e.g. peaks eluting early with too few neighboring peaks for shift-function fitting)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Semi-automated alignment tool that corrects m/z and non-linear retention-time
    shifts relative to a reference chromatogram and combines corresponding ions into
    a single data matrix
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment (JRE 1.8 or newer) required to execute MetMatch
- name: XCMS
  role: Produces the reference/target feature lists and the retention-time shift function
    used to validate the MetMatch alignment by Pearson correlation
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:3554fcd069d8bbd06a9d21a43f244f9bee0f8ed67c509f9adb9a36a88f1564c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Java JRE 1.8
  - MetMatch
  - XCMS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Retention-time shift correction for LC-HRMS alignment

## Summary

Semi-automated correction and alignment of LC-HRMS datasets that exhibit m/z and non-linear retention-time shifts relative to a reference feature list, performed with MetMatch (Java JRE 1.8+), including accounting for differently formed ion adducts.

## When to use

Use when aligning XCMS-derived feature lists or target chromatograms from different LC-HRMS batches where retention times are shifted (e.g. 7–52 s or -9 to -24 s ranges) and m/z deviates slightly (e.g. -3.4 ppm) relative to a reference.

## When NOT to use

- Input is already an aligned, combined feature matrix with corrected retention times — re-alignment would not apply
- Data were measured in a single batch against its own reference, so no cross-experiment m/z or retention-time shift exists
- Target files contain no reference-metabolite-derived features in the expected windows (e.g. peaks eluting early with too few neighboring peaks for shift-function fitting)

## Inputs

- Reference feature list (m/z values, retention times, peak areas), e.g. a 1549-feature merged phenylalanine-derived list or XCMS-derived list
- Target LC-HRMS datafiles or feature lists from a different measurement sequence/batch (e.g. 22 wheat files or 5 barley files)
- Fitted retention-time shift function from XCMS for correlation-based validation

## Outputs

- Combined data matrix of matched chromatographic peak areas, m/z values and retention times (e.g. metmatch_alignment_results.csv)
- Match statistics: number of features matched against the reference (e.g. 92/151) and matched in all/most target files
- MetMatch retention-time shift function and its Pearson correlation against the XCMS shift function

## How to apply

Load the reference feature list (m/z values, retention times, peak areas) plus target LC-HRMS datafiles into MetMatch, which requires Java JRE 1.8 or newer. Run the semi-automated alignment allowing m/z deviations (the wheat case showed a -3.4 ppm average shift) and non-linear retention-time shifts (7–52 s for wheat vs. a 1549-feature phenylalanine-derived reference; -9 to -24 s for barley T-2/HT-2 toxin data), and enable adduct-aware matching so differently formed ion species of the same metabolite are recognized. Restrict to features detected in a minimum number of target files (e.g. at least 5 of 22 wheat files) and count matches — 92 of 151 Phe-derived features with adduct accounting vs. only 42 without. Validate by computing the Pearson correlation between the MetMatch and XCMS retention-time shift functions over defined windows (expected 0.858 and 0.926 over 3–38 min and 7–38 min for wheat; 0.966 over 4–22.5 min for barley). Export the combined data matrix of matched chromatographic peak areas, m/z values and retention times.

## Related tools

- **MetMatch** (Semi-automated alignment tool that corrects m/z and non-linear retention-time shifts relative to a reference chromatogram and combines corresponding ions into a single data matrix) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment (JRE 1.8 or newer) required to execute MetMatch)
- **XCMS** (Produces the reference/target feature lists and the retention-time shift function used to validate the MetMatch alignment by Pearson correlation)

## Evaluation signals

- Matched-feature count matches reported values within reporting precision (92 of 151 Phe-derived features in ≥5 target files with adduct accounting; 42 without)
- MetMatch retention-time shift function correlates with the XCMS shift function at Pearson r = 0.858 and 0.926 (3–38 min and 7–38 min windows, wheat) and r = 0.966 (4–22.5 min, barley)
- Observed retention-time shift range matches expectations (7–52 s wheat; -9 to -24 s barley) and m/z deviation averages about -3.4 ppm
- All matched ions are combined into a single data matrix containing peak areas, m/z values and retention times for every target file

## Limitations

- Peaks eluting early with too few neighboring peaks cannot be matched reliably (three T-2/HT-2-derived ions failed for this reason)
- Low-abundance features with poor peak shape may be missed in some target files (one ion missed in two of five barley files)
- Without adduct-aware matching, differently formed ion species of the same metabolite are not recognized, drastically reducing matches (42 vs. 92)
- MetMatch requires Java runtime (JRE) version 1.8 or newer and is distributed under GPLv2+ without any warranty

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [intro] Corrects m/z and Rt shifts: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [abstract] Combined data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [readme] Java requirement: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
