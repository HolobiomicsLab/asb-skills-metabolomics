---
name: lc-hrms-reference-list-subsampling
description: Use when quantifying how small a reference feature list can be while
  still supporting accurate LC-HRMS retention-time shift correction/alignment, e.g.
  before deploying MetMatch with a trimmed reference list.
when_to_use_negative:
- The input is already an aligned feature table — this skill evaluates reference-list
  sufficiency for alignment, not the alignment itself
- The LC-HRMS data have no retention time shifts relative to the reference chromatogram,
  so alignment accuracy is insensitive to reference-list size
- The goal is de novo feature detection rather than alignment against an existing
  reference feature list
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Executes the LC-HRMS alignment workflow against each subsampled reference
    feature list to produce the alignments scored by accuracy and F-measure
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to run the MetMatch tool
  repo: github.com/S-T-K/MetMatch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:853dd860e19ab41c88ec2a7937bb1d40e3f3853dc619e65be236300512142762
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-hrms-reference-list-subsampling

## Summary

Reproduces the MetMatch Figure 10 experiment that determines the minimum number of common feature groups (m/z and retention time entries) a reference list must contain for reliable retention-time shift alignment of LC-HRMS chromatograms, via progressive 25% subsampling of the 505-group reference list.

## When to use

Use when quantifying how small a reference feature list can be while still supporting accurate LC-HRMS retention-time shift correction/alignment, e.g. before deploying MetMatch with a trimmed reference list.

## When NOT to use

- The input is already an aligned feature table — this skill evaluates reference-list sufficiency for alignment, not the alignment itself
- The LC-HRMS data have no retention time shifts relative to the reference chromatogram, so alignment accuracy is insensitive to reference-list size
- The goal is de novo feature detection rather than alignment against an existing reference feature list

## Inputs

- simulation-6 modified reference list with 505 common feature groups (m/z and retention time entries)
- simulation-6 LC-HRMS datafiles (processed chromatograms with m/z values and retention times)
- Full-list (505-group) reference alignment used as the ground-truth comparison baseline

## Outputs

- figure10_subsampling_statistics.csv with per-list-size statistics (group count, repetitions, mean F-measure, mean accuracy)
- Per-run alignment accuracy and F-measure values for the 200 subsampled alignment runs

## How to apply

Load the simulation-6 modified reference list containing 505 common feature groups (m/z and retention time entries) and generate 20 progressively subsampled lists, each 25% smaller than the previous, spanning from 505 groups down to a single group. For each subsampled list, run the MetMatch alignment workflow (Java JRE 1.8 or newer) 10 times against the simulation-6 LC-HRMS datafiles, yielding 200 total alignment runs. For every run, compute alignment accuracy and F-measure relative to the full-list (505-group) reference alignment. Aggregate the 200 runs to per-list-size statistics (group count, repetitions, mean F-measure, mean accuracy) and check them against the reported benchmarks: full-list accuracy is maintained on average at 159 common feature groups, F-measure exceeds 0.9 for at least 89 groups, and F-measure exceeds 0.8 down to 49 groups. Write the aggregated statistics to figure10_subsampling_statistics.csv and accept only if the reproduced values match the reported Figure 10 statistics.

## Related tools

- **MetMatch** (Executes the LC-HRMS alignment workflow against each subsampled reference feature list to produce the alignments scored by accuracy and F-measure) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to run the MetMatch tool) — github.com/S-T-K/MetMatch

## Evaluation signals

- The average number of common feature groups needed to maintain full-list alignment accuracy is 159 (matching Figure 10)
- Mean F-measure exceeds 0.9 for at least 89 common feature groups
- Mean F-measure exceeds 0.8 down to 49 common feature groups
- The statistics file figure10_subsampling_statistics.csv contains all 20 subsampled list sizes with group count, repetitions, mean F-measure, and mean accuracy
- The total number of alignment runs across all subsampled lists and repetitions equals 200

## Limitations

- Alignment quality degrades sharply below roughly 49–89 common feature groups (F-measure falls below 0.8/0.9), so heavily subsampled reference lists are unreliable
- MetMatch is semi-automated and implemented in Java, requiring JRE version 1.8 or newer
- Discussion section content was not available, so additional caveats about the subsampling experiment could not be traced
- MetMatch is distributed WITHOUT ANY WARRANTY under the GNU General Public License

## Evidence

- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments.: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [abstract] Based on a feature list that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
