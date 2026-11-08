---
name: ms1-ion-abundance-ranking
description: Use when a QC batch in PeakQC lacks user-specified target ions and you
  need the software to pick abundant, frequently detected MS1 ions per sample group
  as auto-tracked QC targets.
when_to_use_negative:
- The user already provides target ions with theoretical/reference m/z values — PeakQC
  extracts metrics for 'user-specified ions' directly, so abundance ranking is unnecessary
- The data lack MS1 scans (e.g., MS2-only or fragmented-only acquisitions), since
  the ranking relies on 'topmost abundant ions in the MS1 data'
- The input is already a processed feature/peak table rather than raw or mzML MS data
  — PeakQC reads raw vendor formats and mzML, not feature lists
tools:
- name: PeakQC
  role: 'Performs the auto-tracked ion selection: finds the topmost abundant, per-group
    most frequent MS1 ions and extracts their peak metrics for QC'
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Converts vendor raw MS data into MZA format for downstream parsing of MS1
    scans by PeakQC
- name: IonToolPack
  role: GUI suite that hosts PeakQC, imports raw MS files from multiple instrument
    formats, and runs the Process pipeline including test_data examples
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:ef13a60aa02d8f2e7620941de30cf3b9301c247f5098650e156615133783a250
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# ms1-ion-abundance-ranking

## Summary

Ranking MS1 ions by abundance and per-sample-group detection frequency in PeakQC (IonToolPack) to automatically select 'auto-tracked' ions for quality control when the user supplies no target ion list.

## When to use

Use when a QC batch in PeakQC lacks user-specified target ions and you need the software to pick abundant, frequently detected MS1 ions per sample group as auto-tracked QC targets.

## When NOT to use

- The user already provides target ions with theoretical/reference m/z values — PeakQC extracts metrics for 'user-specified ions' directly, so abundance ranking is unnecessary
- The data lack MS1 scans (e.g., MS2-only or fragmented-only acquisitions), since the ranking relies on 'topmost abundant ions in the MS1 data'
- The input is already a processed feature/peak table rather than raw or mzML MS data — PeakQC reads raw vendor formats and mzML, not feature lists

## Inputs

- Raw vendor MS files (Agilent .d, Thermo .raw, Bruker .d) or mzML, converted with MZA
- MS1 scan data for all runs in a batch
- Sample-group assignments for the batch runs

## Outputs

- Auto-tracked ion list (e.g., autotracked_ions.csv)
- Per-ion peak metrics for auto-tracked ions (mass error, retention time error, arrival time error)
- CSV reports and JPG/PDF figures per sample group
- Isolation Forest error heatmaps for outlier detection on individual peak metrics

## How to apply

Load MZA-converted MS data (from Agilent .d, Thermo .raw, Bruker .d, or mzML inputs via the IonToolPack GUI) and parse the MS1 scans of every run in the batch. Compile per-ion intensity observations across runs, then group runs by sample group and tally how frequently each ion is detected within each group. Rank ions by abundance and per-group frequency and select the topmost ions as the 'auto-tracked' ions, which PeakQC writes out (e.g., autotracked_ions.csv). Peak metrics are then extracted for the auto-tracked ions exactly as for user-specified (theoretical/reference) ions, feeding CSV reports, JPG/PDF figures, and Isolation Forest error heatmaps. Validate by comparing the produced auto-tracked ion list against the example output files in the IonToolPack test_data folder.

## Related tools

- **PeakQC** (Performs the auto-tracked ion selection: finds the topmost abundant, per-group most frequent MS1 ions and extracts their peak metrics for QC) — https://github.com/pnnl/IonToolPack
- **MZA** (Converts vendor raw MS data into MZA format for downstream parsing of MS1 scans by PeakQC)
- **IonToolPack** (GUI suite that hosts PeakQC, imports raw MS files from multiple instrument formats, and runs the Process pipeline including test_data examples) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # double-click the Release build, import raw MS files (Agilent .d / Thermo .raw / Bruker .d / mzML), then click "Process" to auto-track abundant MS1 ions
```

## Evaluation signals

- An autotracked_ions.csv is produced listing the selected ions, and it matches the example output files in the repository's test_data folder
- Every selected ion is among the topmost abundant MS1 ions and is 'more frequent per sample group' than lower-ranked candidates
- Peak metrics (e.g., errors in mass, retention time, arrival time) are extracted for each auto-tracked ion in each run of the batch
- CSV reports and JPG/PDF figures are generated per sample group, and error heatmaps from the Isolation Forest outlier detection are produced
- The pipeline runs end-to-end from raw data conversion to error heatmap generation without errors on the test_data inputs

## Limitations

- Auto-tracked ions are chosen purely by abundance and per-group frequency in MS1 data, so they may differ from ions a user would deliberately target with theoretical/reference values
- The selection depends on MS1 data being present; acquisitions without MS1 (or without the relevant ions being abundant/frequent) will yield unrepresentative auto-tracked ions
- Workflow is delivered through the IonToolPack GUI (download IonToolPack.exe from Releases, double-click, import raw MS files, click 'Process'); the article notes missing standalone installation instructions and no changelog

## Evidence
