---
name: sample-group-ion-frequency-analysis
description: Use when QC of MS1 data is needed but no user-specified target/reference
  ions exist, so abundant ions that recur per sample group must be selected automatically
  for tracking.
when_to_use_negative:
- User already provides theoretical or reference target ions — PeakQC extracts peak
  metrics directly for user-specified ions and auto-tracking is unnecessary
- Analysis targets MS2-level diagnostics (fragment ions, mobiligrams) rather than
  MS1-based global QC, which requires user-specified ions with expected fragments
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: Executes the auto-tracked ion selection and subsequent peak metric extraction/outlier
    detection on MS1 data
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Converts vendor raw data (e.g., Agilent .d, Thermo .raw, Bruker .d) prior
    to PeakQC processing
- name: IonToolPack
  role: GUI suite housing PeakQC; accepts Agilent .d, Thermo .raw, Bruker .d, and
    mzML inputs and provides example input/output files in test_data
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:20a7df5983df12fcd3dea268458237c8ce58e6e8ba71bc48cd13a3ca5b68616b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# sample-group-ion-frequency-analysis

## Summary

PeakQC's strategy for selecting 'auto-tracked' ions: it ranks ions found in MS1 data by abundance and by how frequently each ion is detected per sample group, then uses the topmost ions as targets for automated QC metric extraction without requiring user-specified reference ions.

## When to use

Use when QC of MS1 data is needed but no user-specified target/reference ions exist, so abundant ions that recur per sample group must be selected automatically for tracking.

## When NOT to use

- User already provides theoretical or reference target ions — PeakQC extracts peak metrics directly for user-specified ions and auto-tracking is unnecessary
- Analysis targets MS2-level diagnostics (fragment ions, mobiligrams) rather than MS1-based global QC, which requires user-specified ions with expected fragments

## Inputs

- Batch of MS1 raw data files (Agilent .d, Thermo .raw, Bruker .d, or mzML; MZA-converted MS data from the repository's test_data folder)
- Sample group assignments for the runs in the batch

## Outputs

- List of auto-tracked ions (topmost abundant ions frequent per sample group)
- CSV reports of peak metrics for auto-tracked ions (mass error, retention time error, arrival time error)
- JPG/PDF figures including error heatmaps from Isolation Forest outlier detection

## How to apply

Load the batch of raw MS files (Agilent .d, Thermo .raw, Bruker .d, or mzML; converted via MZA where needed) into the PeakQC pipeline in IonToolPack, which compiles per-ion intensity observations from the MS1 scans across all runs. Group runs by sample group and tally how frequently each ion is detected within each group. Rank ions by abundance and per-group frequency and take the topmost abundant ions, which are more frequent per sample group, as the 'auto-tracked' ions. Peak metrics (e.g., mass error, retention time error, arrival time error) are then extracted for these auto-tracked ions alongside any user-specified ions, with reports (CSV) and figures (JPG, PDF) generated. Validate the selection by comparing the produced ion list against the example outputs in the repository's test_data folder.

## Related tools

- **PeakQC** (Executes the auto-tracked ion selection and subsequent peak metric extraction/outlier detection on MS1 data) — https://github.com/pnnl/IonToolPack
- **MZA** (Converts vendor raw data (e.g., Agilent .d, Thermo .raw, Bruker .d) prior to PeakQC processing)
- **IonToolPack** (GUI suite housing PeakQC; accepts Agilent .d, Thermo .raw, Bruker .d, and mzML inputs and provides example input/output files in test_data) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import raw MS files, click “Process” to run PeakQC auto-tracking
```

## Evaluation signals

- The auto-tracked ion list corresponds to the topmost abundant ions in MS1 data ranked by per-sample-group detection frequency
- Auto-tracked ion selection reproduces the example output files in IonToolPack's test_data folder when the example MZA-converted data are run
- Peak metrics (mass error, retention time error, arrival time error) are extracted for all auto-tracked ions in every run of the batch
- CSV reports and JPG/PDF figures (including Isolation Forest error heatmaps) are generated per sample group

## Limitations

- Auto-tracking depends on ions being abundant and frequent within sample groups; it is not tailored to any single omics type but relies on whatever ions dominate the MS1 data in the batch
- Traditional/targeted QC cannot be replaced when specific reference molecules must be monitored — user-specified ions with theoretical values are still supported separately

## Evidence
