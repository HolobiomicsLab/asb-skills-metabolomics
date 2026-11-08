---
name: f-measure-aggregation-statistics
description: Use when you must quantify how many common feature groups between a reference
  feature list and target chromatograms are needed for LC-HRMS alignment (MetMatch)
  to retain full accuracy, via subsampled-run F-measure aggregation.
when_to_use_negative:
- You need to perform the LC-HRMS alignment itself rather than benchmark/aggregate
  its accuracy statistics
- Your input is not a paired reference feature list plus target chromatograms from
  the same experiments (e.g., you already have a scored alignment table)
- You cannot run the full 200-run subsampling design (20 lists x 10 repetitions),
  since the aggregation thresholds are calibrated on that specific protocol
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Executes the LC-HRMS feature alignment workflow for each of the 200 subsampled
    reference-list runs against the simulation-6 datafiles
  repo: https://github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch alignment runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:3946efa79ca43d1d2efeda9973f33f0e9dd9ef80c9baf38237dd7f773d0e817b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# f-measure-aggregation-statistics

## Summary

Reproduces the MetMatch Figure 10 minimum-common-feature-groups experiment by aggregating accuracy and F-measure statistics over 200 LC-HRMS alignment runs across 20 progressively subsampled reference feature lists.

## When to use

Use when you must quantify how many common feature groups between a reference feature list and target chromatograms are needed for LC-HRMS alignment (MetMatch) to retain full accuracy, via subsampled-run F-measure aggregation.

## When NOT to use

- You need to perform the LC-HRMS alignment itself rather than benchmark/aggregate its accuracy statistics
- Your input is not a paired reference feature list plus target chromatograms from the same experiments (e.g., you already have a scored alignment table)
- You cannot run the full 200-run subsampling design (20 lists x 10 repetitions), since the aggregation thresholds are calibrated on that specific protocol

## Inputs

- Simulation-6 modified reference feature list with 505 common feature groups (m/z and retention time entries)
- Simulation-6 LC-HRMS datafiles used as alignment targets
- MetMatch alignment software run under Java JRE 1.8 or newer

## Outputs

- figure10_subsampling_statistics.csv with per-list-size rows: group count, repetitions, mean F-measure, mean accuracy
- Aggregated statistics: average 159 groups needed for full-list accuracy, F-measure >0.9 for >=89 groups, F-measure >0.8 down to 49 groups

## How to apply

Load the simulation-6 modified reference list containing 505 common feature groups with m/z and retention time entries, then generate 20 progressively subsampled reference lists, each successive list 25% smaller than the previous one, down to a single group. For each subsampled list, run the MetMatch alignment workflow 10 times against the simulation-6 LC-HRMS datafiles (Java JRE 1.8 required), yielding 200 total alignment runs. For every run, compute accuracy and F-measure of the alignment relative to the full-list (505-group) reference alignment, then aggregate the 200 runs to obtain per-list-size statistics (group count, repetitions, mean F-measure, mean accuracy). Accept the result if it reproduces the reported thresholds: an average of 159 common feature groups to reach full-list accuracy, F-measure >0.9 for at least 89 groups, and F-measure >0.8 down to 49 groups. Write the aggregated per-list-size statistics to figure10_subsampling_statistics.csv.

## Related tools

- **MetMatch** (Executes the LC-HRMS feature alignment workflow for each of the 200 subsampled reference-list runs against the simulation-6 datafiles) — https://github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch alignment runs)

## Evaluation signals

- Mean accuracy of the subsampled alignment matches the full-list (505-group) reference alignment at an average of 159 common feature groups
- Aggregated F-measure exceeds 0.9 for reference lists with at least 89 common feature groups
- Aggregated F-measure exceeds 0.8 down to 49 common feature groups
- Output CSV contains exactly 20 per-list-size rows (group count, repetitions, mean F-measure, mean accuracy) spanning 505 groups down to 1 group
- Total run count is 200 (20 subsampled lists x 10 repetitions each)

## Limitations

- Statistics are calibrated on the simulation-6 LC-HRMS dataset and the 505-group reference list; other datasets may require recalibration
- MetMatch requires Java JRE 1.8 or newer, and discussion-section content was unavailable, limiting documented caveats
- MetMatch is distributed WITHOUT ANY WARRANTY under GPL v2 or later, per the README

## Evidence

- [results] MetMatch required on average 159 common feature groups to maintain the same alignment accuracy as achieved with the complete list of 505 common feature groups: [evidence span withheld — anchor status: not_found]
- [abstract] the tool recognizes shifts: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [intro] Java version requirement: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] README tool description: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
