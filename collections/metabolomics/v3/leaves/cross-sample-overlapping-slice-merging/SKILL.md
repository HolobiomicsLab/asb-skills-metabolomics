---
name: cross-sample-overlapping-slice-merging
description: Use when building peak groups from LC-MS/MS lipidomics or metabolomics
  data (e.g., mzML files) where MS/MS scans must be aggregated across samples and
  assigned to finalized peak groups.
when_to_use_negative:
- Input is already a feature/peak table — merging operates on raw MS1/MS2 scans in
  mzML space
- Data contain only MS1 scans (no fragmentation scans), since slice formation depends
  on MS2 precursor isolation windows
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Performs slice formation, cross-sample merging, peak grouping, and MS/MS scan
    assignment
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Prior peak-grouping logic (peak detection on extracted ion chromatograms within
    slice boundaries) reused by MAVEN2
- name: git
  role: Recursive clone of the maven repository to build MAVEN2 from source
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:61c81d5ead5ee7e0a7d285701cba0eb5fa6e1a596234bcbb1182e2d85efbff40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN
  - MAVEN2
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Cross-sample overlapping slice merging for MS/MS-based peak grouping

## Summary

MAVEN2's MS/MS peak-grouping pipeline: fragmentation (MS2) scans are grouped into (m/z, retention time) slices around precursor isolation windows, overlapping slices are progressively merged across all samples, and the merged slices are then used to form peak groups with MS/MS scans attached afterward by retention time proximity.

## When to use

Use when building peak groups from LC-MS/MS lipidomics or metabolomics data (e.g., mzML files) where MS/MS scans must be aggregated across samples and assigned to finalized peak groups.

## When NOT to use

- Input is already a feature/peak table — merging operates on raw MS1/MS2 scans in mzML space
- Data contain only MS1 scans (no fragmentation scans), since slice formation depends on MS2 precursor isolation windows

## Inputs

- Lipidomics mzML sample files containing MS1 and MS2 scans
- MAVEN2 mzPuller/MSDataScan interface for scan exposure

## Outputs

- peak_groups_msms.csv with fields for group m/z, RT region, and assigned MS/MS scans
- Merged MS/MS slice set across all samples
- Finalized peak groups with attached MS/MS scan associations

## How to apply

Load the deposited lipidomics mzML sample files through the MAVEN2 mzPuller/MSDataScan interface to expose both MS1 and MS2 scans. Form MS/MS slices by grouping fragmentation scans into regions of (m/z, RT) space bounded by each precursor isolation window. Progressively merge overlapping MS/MS slices across all samples regardless of sample of origin, iterating until no further overlap remains — merging before grouping ensures that scans from different samples with the same precursor feed a single peak group. Then form peak groups from the merged slice set using the same peak-grouping logic as prior MAVEN (peak detection on extracted ion chromatograms within slice boundaries). Finally, assign each MS/MS scan to its finalized peak group when the scan's precursor falls within the group's (m/z, RT) region, and export the result.

## Related tools

- **MAVEN2** (Performs slice formation, cross-sample merging, peak grouping, and MS/MS scan assignment) — https://github.com/eugenemel/maven
- **MAVEN** (Prior peak-grouping logic (peak detection on extracted ion chromatograms within slice boundaries) reused by MAVEN2)
- **git** (Recursive clone of the maven repository to build MAVEN2 from source) — https://github.com/eugenemel/maven

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- peak_groups_msms.csv exists and is a valid CSV
- CSV contains fields for group m/z, RT region, and assigned MS/MS scans
- CSV contains a non-zero number of peak-group rows
- No unmerged overlapping slice pairs remain after the merge loop terminates
- Each assigned MS/MS scan's precursor falls within its peak group's (m/z, RT) region

## Limitations

- MS/MS scans are attached only after peak grouping, by retention time proximity to finalized groups — scans near group boundaries may be misassigned
- Changelog incompleteness in the repository means pipeline behavior changes across versions may be undocumented

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS analysis of metabolomics and lipidomics samples: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [methods] Clone and build instructions for MAVEN2: "git clone --recursive  git@github.com:eugenemel/maven.git maven
cd maven
qmake -r build.pro
make -j4"
- [readme] Repository release/build status and download path: "As of 20241105, linux builds have been retired, and both mac os and windows executable are now produced by Appveyor."
