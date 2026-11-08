---
name: mz-rt-shift-correspondence-matching
description: Use when multiple LC-HRMS metabolomics files exhibit m/z and retention
  time shifts relative to a reference chromatogram and corresponding ions of metabolites
  must be aligned into one data matrix.
when_to_use_negative:
- Input is already a fully aligned feature table or combined data matrix and no shift
  correction is needed
- Data are not LC-HRMS metabolomics chromatograms (e.g. GC-MS or proteomics data),
  which the tool's ion-species and retention-time alignment is not designed for
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Semi-automated tool that matches corresponding ions and metabolites between
    LC-HRMS chromatograms and a reference feature list, correcting m/z and retention
    time shifts and combining results into a single data matrix
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to run MetMatch (JRE version 1.8 or newer)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:8af92ad74803dfb2c624c5b29e14661eaf9dfc80db23abd610f63d484e0155e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetMatch
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mz-rt-shift-correspondence-matching

## Summary

Semi-automated matching/alignment of LC-HRMS metabolomics data against a reference feature list using MetMatch, correcting m/z and retention time shifts and combining corresponding ions and metabolites into a single data matrix of peak areas, m/z values and retention times.

## When to use

Use when multiple LC-HRMS metabolomics files exhibit m/z and retention time shifts relative to a reference chromatogram and corresponding ions of metabolites must be aligned into one data matrix.

## When NOT to use

- Input is already a fully aligned feature table or combined data matrix and no shift correction is needed
- Data are not LC-HRMS metabolomics chromatograms (e.g. GC-MS or proteomics data), which the tool's ion-species and retention-time alignment is not designed for

## Inputs

- Reference feature list of processed LC-HRMS chromatograms with m/z values and retention times (e.g. 1482 phenylalanine-derived metabolite features)
- Target LC-HRMS chromatogram(s)/measurement sequences exhibiting m/z and retention time shifts relative to the reference
- Java runtime environment JRE 1.8 or newer

## Outputs

- Single combined data matrix of chromatographic peak areas, m/z values and retention times for corresponding ions and metabolites
- Matching counts (true positives, false negatives) and quality metrics (precision, recall, F-measure) e.g. exported to phe_benchmark_table3.csv

## How to apply

Load a reference feature list (processed LC-HRMS chromatograms with m/z values and retention times) into MetMatch together with the target chromatogram(s) to be matched. Run MetMatch's semi-automated matching pipeline, which recognizes both m/z and retention time shifts of single or multiple datafiles/batches and accounts for differently formed ion species of detected metabolites. Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix. MetMatch requires Java runtime (JRE) version 1.8 or newer and is licensed under the GNU GPL v2 or later. For validation, run a benchmark match of the unmodified reference chromatogram against the unmodified reference feature list (e.g. 1482 Phe-derived features) and compare true positives, false negatives, precision, recall and F-measure against the reported benchmark (1446 of 1482 features matched, F-measure 0.988).

## Related tools

- **MetMatch** (Semi-automated tool that matches corresponding ions and metabolites between LC-HRMS chromatograms and a reference feature list, correcting m/z and retention time shifts and combining results into a single data matrix) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to run MetMatch (JRE version 1.8 or newer))

## Evaluation signals

- Benchmark match of the unmodified reference chromatogram against the unmodified 1482-feature reference list must reproduce 1446 correctly matched features with 36 unmatched
- Precision, recall and F-measure of the benchmark match must equal the Table 3 entries (F-measure 0.988) for the unmodified reference-vs-reference comparison
- Every row of the output data matrix must contain peak area, m/z and retention time for matched corresponding ions and metabolites
- Matched m/z and retention time values in the combined matrix must be consistent with corrected shift values relative to the reference chromatogram
- Counts and metrics written to phe_benchmark_table3.csv must match the reproduced true-positive/false-negative counts from the matching result

## Limitations

- MetMatch is semi-automated and requires a curated reference feature list as the alignment basis
- Reported bugs or issues should be tracked via the GitHub repository issue tracker; the software is distributed WITHOUT ANY WARRANTY under GNU GPL v2 or later
- Requires Java runtime (JRE) version 1.8 or newer; the discussion section of the article contains no substantive text (no changelog found)

## Evidence

- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [abstract] corresponding ions and metabolites are matched and combined into a single data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] MetMatch is a free software; you can redistribute it and/or modify it under the terms of the GNU General Public License: "MetMatch is a free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation"
