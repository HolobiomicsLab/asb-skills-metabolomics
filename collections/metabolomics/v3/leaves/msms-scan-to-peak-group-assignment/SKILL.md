---
name: msms-scan-to-peak-group-assignment
description: Use when processing LC-MS/MS metabolomics or lipidomics mzML samples
  where fragmentation scans must be grouped into precursor-defined (m/z, RT) slices
  and attached to finalized peak groups rather than analyzed per-scan.
when_to_use_negative:
- Input data is MS1-only LC-MS metabolomics with no MS/MS fragmentation scans, where
  prior-MAVEN peak grouping alone suffices
- You need MS/MS spectral library matching or in-silico lipidomics library search
  rather than peak grouping (those are separate MAVEN2 capabilities)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: 'Performs the full pipeline: slice formation in (m/z, RT) space, cross-sample
    slice merging, peak grouping as in prior MAVEN, and MS/MS scan-to-peak-group attachment'
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Supplies the legacy peak-grouping logic (peak detection on extracted ion chromatograms)
    reused for forming peak groups from merged MS/MS slices
  repo: https://github.com/eugenemel/maven
- name: git
  role: Used to recursively clone the MAVEN2 source so the pipeline code can be built
    and run
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:0ddd57dc4a60844c7f1d3d47da38d9a7f4a07b0d60f7e082421c23f6192287d2
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# msms-scan-to-peak-group-assignment

## Summary

Reproduces the MAVEN2 MS/MS-based peak grouping pipeline: MS/MS fragmentation scans are grouped into (m/z, retention time) slices, slices are progressively merged across samples, peak groups are formed with prior-MAVEN peak-grouping logic, and pre-merged slice scans are attached to finalized peak groups by retention time proximity.

## When to use

Use when processing LC-MS/MS metabolomics or lipidomics mzML samples where fragmentation scans must be grouped into precursor-defined (m/z, RT) slices and attached to finalized peak groups rather than analyzed per-scan.

## When NOT to use

- Input data is MS1-only LC-MS metabolomics with no MS/MS fragmentation scans, where prior-MAVEN peak grouping alone suffices
- You need MS/MS spectral library matching or in-silico lipidomics library search rather than peak grouping (those are separate MAVEN2 capabilities)

## Inputs

- Lipidomics mzML sample files containing MS1 and MS2 scans
- MAVEN2 mzPuller/MSDataScan interface exposing MS1 and MS2 scans

## Outputs

- Finalized peak groups spanning merged MS/MS slices across samples
- peak_groups_msms.csv with fields for group m/z, RT region, and assigned MS/MS scans

## How to apply

Load the deposited lipidomics mzML sample files through the MAVEN2 mzPuller/MSDataScan interface to expose MS1 and MS2 scans. Form MS/MS slices by grouping fragmentation scans into regions of (m/z, RT) space around each precursor isolation window, then progressively merge overlapping MS/MS slices across all samples regardless of sample of origin until no further overlap remains. Form peak groups from the merged slice set using the same peak-grouping logic as prior MAVEN (peak detection on extracted ion chromatograms within slice boundaries). Finally, assign each MS/MS scan to its finalized peak group when the scan precursor falls within the group's (m/z, RT) region, and export the peak-group/scan associations.

## Related tools

- **MAVEN2** (Performs the full pipeline: slice formation in (m/z, RT) space, cross-sample slice merging, peak grouping as in prior MAVEN, and MS/MS scan-to-peak-group attachment) — https://github.com/eugenemel/maven
- **MAVEN** (Supplies the legacy peak-grouping logic (peak detection on extracted ion chromatograms) reused for forming peak groups from merged MS/MS slices) — https://github.com/eugenemel/maven
- **git** (Used to recursively clone the MAVEN2 source so the pipeline code can be built and run) — https://github.com/eugenemel/maven

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- peak_groups_msms.csv exists and is a valid CSV
- peak_groups_msms.csv contains fields for group m/z, RT region, and assigned MS/MS scans
- peak_groups_msms.csv contains a non-zero number of peak-group rows
- Every merged slice set yields peak groups formed with the same grouping logic as prior MAVEN, and no overlap remains after progressive cross-sample merging

## Limitations

- Changes for release 8.0.0 are largely undocumented in the repository CHANGELOG (TODO entries remain), so behavior may need to be verified against the source directly
- Linux builds have been retired as of 2024-11-05; only macOS and Windows executables are produced by Appveyor
- Merging depends on overlap of MS/MS slices across samples; samples with insufficient fragmentation coverage may not form complete slice sets

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS analysis with scan-to-peak-group assignment: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [abstract] Algorithms were developed for MS/MS spectral handling: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [methods] MAVEN2 source is cloned recursively from GitHub before use: "git clone --recursive  git@github.com:eugenemel/maven.git maven"
- [readme] Platform availability caveat for running MAVEN2: "As of 20241105, linux builds have been retired, and both mac os and windows executable are now produced by Appveyor."
- [readme] MAVEN2 dependency library must be cloned separately to build the pipeline: [evidence span withheld — anchor status: external_doc]
