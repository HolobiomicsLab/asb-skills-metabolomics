---
name: metabolomics-chromatogram-alignment
description: Use when multiple LC-HRMS measurement sequences from untargeted metabolomics
  experiments exhibit m/z and/or retention time shifts and need to be matched against
  a reference feature list to build a combined peak-area data matrix.
when_to_use_negative:
- The data are already a combined feature table / data matrix with no chromatogram-level
  shift correction needed
- The experiment is not untargeted LC-HRMS metabolomics and does not require alignment
  relative to a reference chromatogram
- MetMatch's Java JRE 1.8+ runtime requirement cannot be met in the environment
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Semi-automated tool used to correct m/z and retention time shifts relative
    to a reference chromatogram and to match/align corresponding ions and metabolites
    into a single data matrix
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute MetMatch (JRE version 1.8 or newer)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:a640cea84ce649eacf9bcdd921a96049243d113c33490dc3ce68ab1a6bbf6cda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetMatch
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-chromatogram-alignment

## Summary

Semi-automated alignment and comparison of LC-HRMS metabolomics datasets using MetMatch, which corrects m/z and retention time shifts of measured chromatograms relative to a reference chromatogram and combines corresponding ions and metabolites into a single data matrix.

## When to use

Use when multiple LC-HRMS measurement sequences from untargeted metabolomics experiments exhibit m/z and/or retention time shifts and need to be matched against a reference feature list to build a combined peak-area data matrix.

## When NOT to use

- The data are already a combined feature table / data matrix with no chromatogram-level shift correction needed
- The experiment is not untargeted LC-HRMS metabolomics and does not require alignment relative to a reference chromatogram
- MetMatch's Java JRE 1.8+ runtime requirement cannot be met in the environment

## Inputs

- Reference LC-HRMS chromatogram (e.g. original unmodified Phe reference chromatogram)
- Reference feature list with m/z values and retention times (e.g. 1482 features from supplementary Sections 2.1/2.2)
- Measurement-sequence chromatogram files exhibiting m/z and retention time shifts to be corrected

## Outputs

- Single combined data matrix of chromatographic peak areas, m/z values and retention times for corresponding ions and metabolites
- Matching results with true positives/false negatives from which precision, recall and F-measure are computed (e.g. phe_benchmark_table3.csv)

## How to apply

Load the reference LC-HRMS chromatogram and a reference feature list (processed chromatograms with m/z values and retention times, e.g. the 1482-feature list compiled in supplementary Sections 2.1/2.2 for the phenylalanine-derived metabolites in wheat) into MetMatch, which requires Java runtime (JRE) 1.8 or newer. Run the semi-automated alignment so corresponding ions and metabolites are matched, and chromatographic peak areas, m/z values and retention times are combined into a single data matrix. First establish a benchmark by matching the unmodified reference chromatogram directly against the unmodified reference feature list; in the published benchmark this reproduced 1446 of 1482 features correctly matched (36 unmatched), with precision, recall and F-measure computed for comparison against Table 3. For shifted batches, use MetMatch's shift correction to recognize and correct m/z and retention time shifts relative to the reference chromatogram before matching, and account for differently formed ion species of detected metabolites. Judge success by reproducing the published true-positive/false-negative counts and precision/recall/F-measure for the benchmark comparison.

## Related tools

- **MetMatch** (Semi-automated tool used to correct m/z and retention time shifts relative to a reference chromatogram and to match/align corresponding ions and metabolites into a single data matrix) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute MetMatch (JRE version 1.8 or newer))

## Evaluation signals

- Benchmark match of the unmodified Phe reference chromatogram against the unmodified reference list yields 1446 of 1482 reference-list features matched and 36 unmatched
- Reproduced true-positive and false-negative counts match the Table 3 entries for the unmodified reference-vs-reference comparison
- Computed precision, recall and F-measure (published F-measure 0.988 for the benchmark) match Table 3
- After shift correction, m/z and retention time shifts of single or multiple analytical datafiles/batches are recognized and corrected relative to the reference chromatogram
- Output data matrix contains combined peak areas, m/z values and retention times for matched ions and metabolites

## Limitations

- MetMatch is a semi-automated tool, not fully automated; user interaction with its GUI is expected
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- The software is distributed under GNU GPL without any warranty, not even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE
- Some features in the benchmark comparison (36 of 1482) could not be matched even without shifts

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [abstract] Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE: "MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE"
