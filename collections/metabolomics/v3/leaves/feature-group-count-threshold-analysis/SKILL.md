---
name: feature-group-count-threshold-analysis
description: Use when you must establish how small a reference feature list can be
  (relative to a full common-feature list) before retention time shift computation
  and alignment accuracy of LC-HRMS chromatograms degrade.
when_to_use_negative:
- The task is a single routine alignment of LC-HRMS datafiles with a complete reference
  feature list — no threshold estimation is needed
- Input is a data matrix of already-aligned features rather than a reference feature
  list plus target chromatograms
- The reference and target chromatograms share too few common feature groups to support
  progressive subsampling (the experiment depends on a large starting list such as
  505 groups)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetMatch
  role: Semi-automated alignment tool that computes the Rt shift function from a reference
    feature list and aligns LC-HRMS chromatograms; run repeatedly per subsampled list
    size to measure accuracy and F-measure
  repo: https://github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:17606c3a4f4cb1f6133ea0caaeb4a9173b37113c81f4d5a7c99892c6b1811c8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# feature-group-count-threshold-analysis

## Summary

Determines the minimum number of common feature groups (m/z and retention time entries) a reference feature list must contain for MetMatch to reliably compute the retention time (Rt) shift function and maintain LC-HRMS alignment accuracy, via progressive subsampling with repeated alignment runs.

## When to use

Use when you must establish how small a reference feature list can be (relative to a full common-feature list) before retention time shift computation and alignment accuracy of LC-HRMS chromatograms degrade.

## When NOT to use

- The task is a single routine alignment of LC-HRMS datafiles with a complete reference feature list — no threshold estimation is needed
- Input is a data matrix of already-aligned features rather than a reference feature list plus target chromatograms
- The reference and target chromatograms share too few common feature groups to support progressive subsampling (the experiment depends on a large starting list such as 505 groups)

## Inputs

- Full reference feature list of common feature groups with m/z and retention time entries (e.g. simulation-6 modified reference list, 505 groups)
- Target LC-HRMS datafiles to align (e.g. simulation-6 dataset)
- Subsampling scheme parameters (reduction factor per step, e.g. 25%, and number of repetitions per list size, e.g. 10)

## Outputs

- Aggregated per-list-size statistics table (group count, repetitions, mean F-measure, mean accuracy), e.g. figure10_subsampling_statistics.csv
- Minimum common feature group count required to retain full-list alignment accuracy (e.g. 159 groups)
- F-measure threshold curves (F > 0.9 for >=89 groups; F > 0.8 down to 49 groups)

## How to apply

Start from a full reference feature list of common feature groups (e.g. 505 groups with m/z and retention time entries from simulation-6 LC-HRMS datafiles). Generate a series of progressively subsampled reference lists, each 25% smaller than the previous, down to a single group (20 lists in the reported experiment). For each subsampled list, run the MetMatch alignment workflow multiple times against the same LC-HRMS datafiles (10 repetitions yielded 200 total runs) and compute the accuracy and F-measure of each alignment relative to the full-list (505-group) reference alignment. Aggregate per-list-size statistics to read off thresholds: in the reported case an average of 159 common feature groups was needed to match full-list accuracy, F-measure stayed above 0.9 for at least 89 groups and above 0.8 down to 49 groups. Write the aggregated statistics (group count, repetitions, mean F-measure, mean accuracy) to a CSV (e.g. figure10_subsampling_statistics.csv) for validation against reported values.

## Related tools

- **MetMatch** (Semi-automated alignment tool that computes the Rt shift function from a reference feature list and aligns LC-HRMS chromatograms; run repeatedly per subsampled list size to measure accuracy and F-measure) — https://github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- Reproduced average minimum group count matches the reported value of 159 common feature groups for full-list accuracy
- Reproduced mean F-measure exceeds 0.9 for at least 89 common feature groups
- Reproduced mean F-measure exceeds 0.8 down to 49 common feature groups
- Statistics file contains the expected columns and rows: one entry per subsampled group count with repetitions, mean F-measure, and mean accuracy
- Total number of alignment runs equals number of subsampled lists times repetitions per list (20 x 10 = 200 runs)

## Limitations

- Thresholds (159 groups for full accuracy; F > 0.9 at >=89 groups; F > 0.8 down to 49 groups) were derived from a specific simulation dataset (simulation-6, 505 common feature groups) and may not transfer to other metabolomics datasets
- Estimates are stochastic: they are averaged over repeated alignment runs (10 per list size), so a single run per list size may not yield stable threshold values
- MetMatch is distributed without warranty under the GNU GPL (v2 or later); issues or bugs must be reported to the repository

## Evidence

- [results] MetMatch required on average 159 common feature groups to maintain the same alignment accuracy as achieved with the complete list of 505 common feature groups (Figure 10).: [evidence span withheld — anchor status: not_found]
- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
