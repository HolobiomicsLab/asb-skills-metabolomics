---
name: maven-peak-group-detection
description: Use when raw mzML LC-MS/MS metabolomics or lipidomics samples need peak
  detection and grouping into peak groups with associated MS/MS scans, rather than
  a precomputed feature table.
when_to_use_negative:
- Input is already a feature/peak table rather than raw mzML scan data
- Data lack MS/MS scans and only MS1 peak detection in prior-MAVEN style is needed
  without slice formation
- Analysis goal is in-silico lipid spectra library generation rather than peak grouping
  from measured data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MAVEN2
  role: Runs the LC-MS/MS peak detection and MS/MS-based slice formation, merging,
    and peak-group assignment pipeline
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Provides the original peak-grouping logic reused by MAVEN2 for forming peak
    groups from extracted ion chromatograms
  repo: https://github.com/eugenemel/maven
- name: git
  role: Recursive clone of the maven source used to build the tool locally
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:d59148f6e9681b985ff555120fff725cee178e6ff0b554233a30e7a30e11bda8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN
  - MAVEN2
  - git
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# maven-peak-group-detection

## Summary

Detect and group chromatographic peak groups from LC-MS metabolomics data using MAVEN2's peak-grouping logic, optionally extended with MS/MS-based slice formation and merging in (m/z, retention time) space so fragmentation scans are attached to finalized peak groups.

## When to use

Use when raw mzML LC-MS/MS metabolomics or lipidomics samples need peak detection and grouping into peak groups with associated MS/MS scans, rather than a precomputed feature table.

## When NOT to use

- Input is already a feature/peak table rather than raw mzML scan data
- Data lack MS/MS scans and only MS1 peak detection in prior-MAVEN style is needed without slice formation
- Analysis goal is in-silico lipid spectra library generation rather than peak grouping from measured data

## Inputs

- Lipidomics mzML sample files with MS1 and MS2 scans
- Precursor isolation-window boundaries per fragmentation scan

## Outputs

- Finalized peak groups spanning samples with (m/z, RT) region boundaries
- MS/MS scan-to-peak-group associations
- Exported peak_groups_msms.csv containing group m/z, RT region, and assigned MS/MS scans

## How to apply

Load deposited lipidomics mzML files via the MAVEN2 mzPuller/MSDataScan interface to expose both MS1 and MS2 scans. Form MS/MS slices by grouping fragmentation scans into regions of (m/z, RT) space around each precursor isolation window, then progressively merge overlapping MS/MS slices across all samples regardless of sample of origin until no further overlap remains. Run peak detection on extracted ion chromatograms within slice boundaries using the same peak-grouping logic as prior MAVEN to form peak groups. Finally, assign each MS/MS scan to its finalized peak group when the scan precursor falls within the group's (m/z, RT) region, and export the results.

## Related tools

- **MAVEN2** (Runs the LC-MS/MS peak detection and MS/MS-based slice formation, merging, and peak-group assignment pipeline) — https://github.com/eugenemel/maven
- **MAVEN** (Provides the original peak-grouping logic reused by MAVEN2 for forming peak groups from extracted ion chromatograms) — https://github.com/eugenemel/maven
- **git** (Recursive clone of the maven source used to build the tool locally) — https://github.com/eugenemel/maven

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- peak_groups_msms.csv exists and is a valid CSV with fields for group m/z, RT region, and assigned MS/MS scans
- peak_groups_msms.csv contains a non-zero number of peak-group rows
- Every MS/MS scan assigned to a peak group has a precursor falling within the group's (m/z, RT) region
- Merged MS/MS slices show no remaining overlaps across samples after the merging step terminates
- Build completes via the documented qmake/make workflow (qmake -r build.pro; make -j4), confirming a working MAVEN2 binary

## Limitations

- Merging MS/MS slices across all samples regardless of origin may conflate scans from different samples into a single slice region
- Linux builds have been retired (as of 2024-11-05); mac os and windows executables are produced by Appveyor
- MAVEN2's MS/MS pipeline requires mzML input exposed through the mzPuller/MSDataScan interface; other raw formats are not described here

## Evidence

- [full_text] The pipeline assigns MS/MS scans to peak groups after grouping: slices are merged across samples, peak grouping is performed as in prior MAVEN, and pre-merged slice scans are attached to finalized peak groups by retention time proximity.: [evidence span withheld — anchor status: not_found]
- [abstract] MAVEN2 supports LC-MS/MS analysis of metabolomics and lipidomics samples.: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [methods] Build instructions for cloning and compiling MAVEN.: "git clone --recursive  git@github.com:eugenemel/maven.git maven"
- [readme] README statement on platform support for executables.: "As of 20241105, linux builds have been retired, and both mac os and windows executable are now produced by Appveyor."
