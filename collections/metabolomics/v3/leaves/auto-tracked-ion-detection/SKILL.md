---
name: auto-tracked-ion-detection
description: Use when running omics-agnostic MS quality control on batch MS1 data
  (mzML or Agilent .d, Thermo .raw, Bruker .d inputs) and no theoretical/reference
  ion list is available to specify as tracking targets.
when_to_use_negative:
- Theoretical or reference ion values (m/z, RT, expected fragments/intensities) are
  already available — PeakQC supports user-specified ion targets instead of auto-tracking
- The QC goal is MS2-focused, e.g. diagnostic figures for expected fragment ions or
  mobiligrams for IMS-separated user-specified ions — those require user-specified
  inputs, not auto-tracked ions
- Data is already a curated feature/peak table rather than raw or MZA-converted MS1
  scans
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Omics-agnostic QC tool that finds the topmost abundant, per-group-frequent
    ions in MS1 data and uses them as auto-tracked ions for peak metric extraction
    and outlier detection
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Data conversion tool used to convert vendor raw files to a common format for
    use in the PeakQC pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:461d58bbac64b3fbb316a5ed0021c924d462cc89d32b4ba279fa82ee5fd23430
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# auto-tracked-ion-detection

## Summary

PeakQC's mechanism for automatically selecting 'auto-tracked' ions from MS1 data — the topmost abundant ions that are most frequent per sample group — so that quality control can proceed without user-specified target ions.

## When to use

Use when running omics-agnostic MS quality control on batch MS1 data (mzML or Agilent .d, Thermo .raw, Bruker .d inputs) and no theoretical/reference ion list is available to specify as tracking targets.

## When NOT to use

- Theoretical or reference ion values (m/z, RT, expected fragments/intensities) are already available — PeakQC supports user-specified ion targets instead of auto-tracking
- The QC goal is MS2-focused, e.g. diagnostic figures for expected fragment ions or mobiligrams for IMS-separated user-specified ions — those require user-specified inputs, not auto-tracked ions
- Data is already a curated feature/peak table rather than raw or MZA-converted MS1 scans

## Inputs

- Batch of MS1-containing raw MS files (Agilent .d, Thermo .raw, Bruker .d, or mzML), or MZA-converted MS data from the repository's test_data folder
- Sample group assignments for the runs in the batch

## Outputs

- Auto-tracked ion list (autotracked_ions.csv) of the topmost abundant, per-group-frequent MS1 ions
- Peak metrics (mass error, retention time error, arrival time error) and CSV/JPG/PDF reports computed for the auto-tracked ions

## How to apply

Load the batch of raw or MZA-converted MS files into the PeakQC tool within IonToolPack (a GUI executable with no installation step). Parse the MS1 scans of each run and compile per-ion intensity observations across the whole batch. Group the runs by sample group and tally how frequently each ion is detected within each group. Rank ions by abundance and per-group detection frequency, and select the topmost abundant, group-frequent ions as the auto-tracked ion set. These ions then serve as the targets for the downstream QC steps (peak metric extraction for mass, retention time, and arrival time errors, and Isolation Forest outlier detection). Validate the selection by comparing the produced ion list against the example input/output files in the IonToolPack test_data folder.

## Related tools

- **PeakQC** (Omics-agnostic QC tool that finds the topmost abundant, per-group-frequent ions in MS1 data and uses them as auto-tracked ions for peak metric extraction and outlier detection) — https://github.com/pnnl/IonToolPack
- **MZA** (Data conversion tool used to convert vendor raw files to a common format for use in the PeakQC pipeline)

## Examples

```
Download IonToolPack.exe from Releases, double-click to launch the GUI, select the PeakQC tab, import raw MS files (Agilent .d / Thermo .raw / Bruker .d / mzML), and click "Process".
```

## Evaluation signals

- The auto-tracked ion list reproduces the example output files provided in the repository's test_data folder
- Selected ions are the topmost abundant ions observed in the MS1 scans, not random or low-abundance features
- Selected ions are more frequent per sample group than ions excluded from the auto-tracked set
- Downstream PeakQC reports (CSV) and figures (JPG/PDF) are generated for every auto-tracked ion and run
- The pipeline runs end-to-end from raw data conversion through error heatmap generation without user-supplied target ions

## Limitations

- Auto-tracking is data-driven, so the selected ion set depends on the batch composition and may differ between batches; it does not rely on theoretical/reference values
- Requires MS1 data with detectable abundant ions; direct infusion and LC/IMS acquisition types are supported, but the tracked ions are only as informative as the group frequency/abundance ranking
- IonToolPack.exe is a Windows GUI executable with no pip/conda/docker installation path documented, and no changelog is provided

## Evidence

- [results] The software finds the topmost abundant ions in the MS1 data which are more frequent per sample group and uses them as “auto-tracked” ions.: "The software finds the topmost abundant ions in the MS1 data which are more frequent per sample group and uses them as “auto-tracked” ions."
- [results] Peak metrics are then extracted for both auto-tracked and user-specified ions (i.e., theoretical or reference values), and various reports (CSV) and figures (JPG and PDF) are generated.: "Peak metrics are then extracted for both auto-tracked and user-specified ions (i.e., theoretical or reference values), and various reports (CSV) and figures (JPG and PDF) are generated."
- [readme] Supported formats include Agilent 'd', Thermo '.raw', Bruker 'd', and mzML, and for different types of MS acquisition methods:: "Supported formats include Agilent 'd', Thermo '.raw', Bruker 'd', and mzML, and for different types of MS acquisition methods:"
- [readme] See example input and output files in test_data.: "See example input and output files in test_data."
- [results] Raw data conversion is a step in the pipeline: [evidence span withheld — anchor status: not_found]
