---
name: pearson-correlation-of-alignment-functions
description: Use when comparing or validating the retention-time shift correction
  produced by MetMatch against an independent alignment (e.g., XCMS) on LC-HRMS feature
  lists, to confirm both tools recovered consistent non-linear Rt shifts.
when_to_use_negative:
- Only a single datafile or batch is involved and no independent alignment (e.g.,
  XCMS) exists to compare the Rt shift functions against — the correlation step has
  no second function to correlate.
- The m/z and retention-time shifts between experiments are already negligible, so
  no shift correction or its validation is needed.
- The analysis goal is peak matching/adduct-aware feature alignment itself (e.g.,
  reproducing 92/151 Phe-derived matches) rather than validating the alignment's Rt
  correction.
tools:
- name: MetMatch
  role: Semi-automated alignment tool whose fitted retention-time shift functions
    (correcting LC-HRMS Rt shifts relative to a reference chromatogram) are the subject
    of the correlation validation
  repo: github.com/S-T-K/MetMatch
- name: XCMS
  role: Independent alignment pipeline providing the reference Rt shift functions
    against which MetMatch's shift functions are Pearson-correlated
- name: Java
  role: Runtime requirement for executing MetMatch (JRE version 1.8 or newer)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:07b7863d72476f3af531e78742048360517085f6643d6d0318f9fea592187a2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetMatch
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pearson-correlation-of-alignment-functions

## Summary

Validates that MetMatch's retention-time shift correction of LC-HRMS datafiles agrees with an independent alignment tool (XCMS) by computing Pearson correlations between the two tools' Rt shift functions over defined chromatographic windows, as done for the wheat phenylalanine-derived metabolite benchmark.

## When to use

Use when comparing or validating the retention-time shift correction produced by MetMatch against an independent alignment (e.g., XCMS) on LC-HRMS feature lists, to confirm both tools recovered consistent non-linear Rt shifts.

## When NOT to use

- Only a single datafile or batch is involved and no independent alignment (e.g., XCMS) exists to compare the Rt shift functions against — the correlation step has no second function to correlate.
- The m/z and retention-time shifts between experiments are already negligible, so no shift correction or its validation is needed.
- The analysis goal is peak matching/adduct-aware feature alignment itself (e.g., reproducing 92/151 Phe-derived matches) rather than validating the alignment's Rt correction.

## Inputs

- MetMatch retention-time shift functions fitted over the 3-38 min and 7-38 min windows (from aligning 22 wheat target feature lists to a 1549-feature Phe-derived reference list)
- Corresponding XCMS retention-time shift functions from the independent alignment of the same LC-HRMS datafiles
- MetMatch and Java JRE 1.8+ runtime environment

## Outputs

- Pearson correlation coefficient of Rt shift functions over the 3-38 min window (expected 0.858)
- Pearson correlation coefficient of Rt shift functions over the 7-38 min window (expected 0.926)
- Combined data matrix of matched chromatographic peak areas, m/z values and retention times exported to metmatch_alignment_results.csv

## How to apply

Fit MetMatch retention-time shift functions between the target datafiles (here the 22 wheat target files) and the reference chromatogram over two windows — the full 3-38 min range and the restricted 7-38 min range — using non-linear shift models that capture the 7-52 s shifts reported relative to the reference. Independently extract the corresponding Rt shift functions from the XCMS alignment of the same raw data. Compute the Pearson correlation coefficient between the MetMatch and XCMS Rt shift functions for each window; for the wheat Phe benchmark the expected values are 0.858 (3-38 min) and 0.926 (7-38 min). Correlations near or above ~0.85 indicate that MetMatch's m/z- and Rt-shift correction is concordant with the established XCMS result, supporting trust in the combined matched-feature matrix.

## Related tools

- **MetMatch** (Semi-automated alignment tool whose fitted retention-time shift functions (correcting LC-HRMS Rt shifts relative to a reference chromatogram) are the subject of the correlation validation) — github.com/S-T-K/MetMatch
- **XCMS** (Independent alignment pipeline providing the reference Rt shift functions against which MetMatch's shift functions are Pearson-correlated)
- **Java** (Runtime requirement for executing MetMatch (JRE version 1.8 or newer))

## Evaluation signals

- Pearson correlation of MetMatch vs XCMS Rt shift functions over the 3-38 min window matches the reported 0.858 within reporting precision.
- Pearson correlation over the restricted 7-38 min window matches the reported 0.926 within reporting precision.
- MetMatch was run under JRE 1.8 or newer, as required by the tool.
- Fitted shift functions capture the reported non-linear retention-time shifts in the 7-52 s range and average m/z deviation of -3.4 ppm.
- The validated alignment exports peak areas, m/z values and retention times combined into a single data matrix (metmatch_alignment_results.csv).

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer, so execution environments lacking this runtime cannot run the alignment being validated.
- The correlations are specific to comparing against a particular reference alignment (XCMS); correlation with a different tool's shift functions is not established by this skill.
- MetMatch is distributed WITHOUT ANY WARRANTY under the GNU GPL; correlation agreement does not guarantee correctness of individual feature matches.

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [abstract] Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix.: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
