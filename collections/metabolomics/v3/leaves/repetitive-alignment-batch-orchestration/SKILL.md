---
name: repetitive-alignment-batch-orchestration
description: Use when you must determine the minimum reference feature-list size (e.g.
  common feature groups between reference and target chromatograms) needed for reliable
  retention-time alignment, requiring systematic repetition of an alignment workflow
  across subsampled inputs.
when_to_use_negative:
- You already know the reference feature list is complete and representative — a single
  alignment run suffices; subsampling orchestration would add cost without new information
- The target data are not processed LC-HRMS feature chromatograms with m/z and retention
  times, since MetMatch operates on such feature lists rather than raw spectra
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetMatch
  role: Semi-automated alignment tool whose workflow is executed repeatedly (10x per
    subsampled reference list size) to correct retention-time/m/z shifts and yield
    per-run accuracy and F-measure values
  repo: https://github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute the MetMatch tool for each of the
    200 batch alignment runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:ff427ab4dfb54e44f439e8f197513c2f89dc1c355000a15434351ac46aeb99a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# repetitive-alignment-batch-orchestration

## Summary

Orchestrates many repeated MetMatch alignment runs over progressively subsampled reference feature lists (e.g. 20 list sizes × 10 repetitions = 200 runs) against LC-HRMS datafiles to quantify how reference-list size affects alignment robustness, aggregating per-run accuracy and F-measure into statistics like Figure 10.

## When to use

Use when you must determine the minimum reference feature-list size (e.g. common feature groups between reference and target chromatograms) needed for reliable retention-time alignment, requiring systematic repetition of an alignment workflow across subsampled inputs.

## When NOT to use

- You already know the reference feature list is complete and representative — a single alignment run suffices; subsampling orchestration would add cost without new information
- The target data are not processed LC-HRMS feature chromatograms with m/z and retention times, since MetMatch operates on such feature lists rather than raw spectra

## Inputs

- Reference feature list of common feature groups with m/z and retention time entries (e.g. 505 groups from simulation-6)
- Target LC-HRMS datafiles to align against the reference (simulation-6 files)
- MetMatch alignment workflow configuration (Java JRE 1.8 or newer required)

## Outputs

- Aggregated per-list-size statistics table (figure10_subsampling_statistics.csv) with columns: group count, repetitions, mean F-measure, mean accuracy
- Per-run alignment accuracy and F-measure values relative to the full-list (505-group) reference alignment
- Estimated minimum common-feature-group thresholds (e.g. 159 groups for full accuracy, >=89 groups for F > 0.9)

## How to apply

Start from the full reference feature list (m/z and retention time entries; here 505 common feature groups from simulation-6) and generate progressively smaller versions, each ~25% smaller than the previous, down to a single group. For each list size, run the MetMatch alignment workflow multiple times (10 repetitions here) against the same LC-HRMS datafiles, yielding 200 total runs. For every run, compute alignment accuracy and F-measure relative to the alignment obtained with the complete 505-group reference list. Aggregate across repetitions per list size and write per-size statistics (group count, repetitions, mean F-measure, mean accuracy) to a CSV such as figure10_subsampling_statistics.csv. Rationale: single runs cannot distinguish systematic degradation from run-to-run variance, so the batch structure lets you identify threshold group counts — e.g. an average of 159 groups for full-list accuracy, F > 0.9 at >= 89 groups, F > 0.8 down to 49 groups.

## Related tools

- **MetMatch** (Semi-automated alignment tool whose workflow is executed repeatedly (10x per subsampled reference list size) to correct retention-time/m/z shifts and yield per-run accuracy and F-measure values) — https://github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute the MetMatch tool for each of the 200 batch alignment runs)

## Evaluation signals

- The aggregated statistics reproduce the reported Figure 10 values: average of 159 common feature groups needed for full-list accuracy, F-measure > 0.9 for at least 89 groups, and F-measure > 0.8 down to 49 groups
- Exactly 200 alignment runs were completed (20 subsampled list sizes × 10 repetitions)
- Per-list-size rows in figure10_subsampling_statistics.csv contain all four fields: group count, repetitions, mean F-measure, mean accuracy
- The smallest subsampled reference list contains a single common feature group, and successive lists shrink by ~25%
- Per-run F-measure and accuracy are non-degenerate (monotonically decreasing on average as list size shrinks from 505 toward 1)

## Limitations

- The minimum-group thresholds (159 groups for full accuracy, F > 0.9 at >= 89 groups) were derived from simulation-6 LC-HRMS data and may not transfer to other metabolomics datasets
- Results depend on the specific subsampling scheme (25% successive reduction); other decay schedules could shift the estimated thresholds
- MetMatch is distributed WITHOUT ANY WARRANTY under the GNU GPL (version 2 or later), and requires JRE 1.8 or newer

## Evidence

- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.: "MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE."
