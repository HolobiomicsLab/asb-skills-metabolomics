---
name: lipidomics-msms-slice-formation-in-mz-rt-space
description: Use when processing LC-MS/MS lipidomics or metabolomics mzML data where
  MS/MS scans must be linked to quantified MS1 peak groups across multiple samples.
when_to_use_negative:
- Input is MS1-only LC-MS data with no fragmentation scans — grouping reduces to standard
  MAVEN peak detection with no slices to merge
- Input is already a finalized feature/peak table rather than raw mzML scans, since
  slices must be formed from raw fragmentation scans
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MAVEN2
  role: Implements the MS/MS slice formation, cross-sample slice merging, and assignment
    of MS/MS scans to finalized peak groups
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Provides the legacy peak-grouping logic reused for peak detection on extracted
    ion chromatograms within merged slice boundaries
  repo: https://github.com/eugenemel/maven
- name: git
  role: Recursive clone of the maven source to build and run the pipeline
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:0aed1644b3211774a9c8dad232d6212422bbbc52f7683196adf7fcd29673007c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN
  - MAVEN2
  - git
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# MS/MS-based peak grouping via slice formation and merging in (m/z, RT) space

## Summary

MAVEN2's MS/MS-aware peak grouping pipeline: fragmentation scans are grouped into slices of (m/z, RT) space around precursor isolation windows, merged progressively across samples, then peak groups are formed and MS/MS scans are attached to finalized groups by retention time proximity.

## When to use

Use when processing LC-MS/MS lipidomics or metabolomics mzML data where MS/MS scans must be linked to quantified MS1 peak groups across multiple samples.

## When NOT to use

- Input is MS1-only LC-MS data with no fragmentation scans — grouping reduces to standard MAVEN peak detection with no slices to merge
- Input is already a finalized feature/peak table rather than raw mzML scans, since slices must be formed from raw fragmentation scans

## Inputs

- Deposited lipidomics mzML sample files containing paired MS1 and MS2 scans
- Precursor isolation window boundaries per fragmentation scan

## Outputs

- peak_groups_msms.csv with fields for group m/z, RT region, and assigned MS/MS scans

## How to apply

Load the lipidomics mzML sample files through the MAVEN2 mzPuller/MSDataScan interface to expose both MS1 and MS2 scans. Form MS/MS slices by grouping fragmentation scans into regions of (m/z, retention time) space around each precursor isolation window. Progressively merge overlapping MS/MS slices across all samples regardless of sample of origin until no further overlap remains, then form peak groups from the merged slice set using the same peak-grouping logic as prior MAVEN (peak detection on extracted ion chromatograms within slice boundaries). Finally, assign each MS/MS scan to its finalized peak group when the scan precursor falls within the group's (m/z, RT) region, exploiting the fact that pre-merged slice scans are attached after grouping rather than during it.

## Related tools

- **MAVEN2** (Implements the MS/MS slice formation, cross-sample slice merging, and assignment of MS/MS scans to finalized peak groups) — https://github.com/eugenemel/maven
- **MAVEN** (Provides the legacy peak-grouping logic reused for peak detection on extracted ion chromatograms within merged slice boundaries) — https://github.com/eugenemel/maven
- **git** (Recursive clone of the maven source to build and run the pipeline) — https://github.com/eugenemel/maven

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- peak_groups_msms.csv exists and parses as valid CSV
- Output contains a non-zero number of peak-group rows
- CSV schema includes fields for group m/z, RT region, and assigned MS/MS scans
- Every assigned MS/MS scan's precursor falls within its peak group's (m/z, RT) region
- No two merged slices retain residual overlap after the merge loop terminates

## Limitations

- MS/MS scan-to-group assignment depends on retention time proximity, so scans near group boundaries may be ambiguous
- Slice merging is overlap-driven across samples, so poorly sampled precursors may yield thin or absent slices

## Evidence

- [results] MS/MS scans assigned after grouping: [evidence span withheld — anchor status: not_found]
- [abstract] MAVEN2 scope: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [readme] Build from source: "git clone --recursive  git@github.com:eugenemel/maven.git maven"
