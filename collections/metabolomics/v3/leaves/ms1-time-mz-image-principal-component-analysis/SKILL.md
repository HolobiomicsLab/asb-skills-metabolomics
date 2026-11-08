---
name: ms1-time-mz-image-principal-component-analysis
description: Use when you have a batch of MS runs (Thermo .raw, Agilent .d, Bruker
  .d, or mzML; LC-MS, LC-IMS-MS, or direct infusion) and need an omics-agnostic, MS1-based
  global quality check to flag defective runs or group-separating trends before deeper
  metrics analysis.
when_to_use_negative:
- You need to diagnose the specific cause of poor performance (e.g., chromatography,
  sensitivity, mass calibration) — identification-based or metric-specific QC is needed
  since PCA flags outliers without pinpointing causes
- Input is already processed peak metrics or a feature table rather than raw MS data
  — PeakQC reads raw instrument formats, not precomputed QC tables
- Only a single run is available with no batch context — PCA-based batch QC is designed
  to compare a batch of runs, not assess one run in isolation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: PeakQC
  role: 'Core tool: generates time-versus-m/z images from MS1 data and performs the
    batch PCA for global quality assessment; part of IonToolPack'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI suite housing PeakQC; imports raw MS files in multiple vendor formats
    and runs the 'Process' step that produces the QC outputs
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:dab09bcb66bf2021b3855ae0a28659e26b522adc599b1ad591d4a0239b8419d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ms1-time-mz-image-principal-component-analysis

## Summary

PeakQC-style batch QC in which each MS run is represented by a time-versus-m/z image built from MS1 data and principal component analysis of those images is used to globally assess data quality, quickly spotting compromised or outlier runs in a batch.

## When to use

Use when you have a batch of MS runs (Thermo .raw, Agilent .d, Bruker .d, or mzML; LC-MS, LC-IMS-MS, or direct infusion) and need an omics-agnostic, MS1-based global quality check to flag defective runs or group-separating trends before deeper metrics analysis.

## When NOT to use

- You need to diagnose the specific cause of poor performance (e.g., chromatography, sensitivity, mass calibration) — identification-based or metric-specific QC is needed since PCA flags outliers without pinpointing causes
- Input is already processed peak metrics or a feature table rather than raw MS data — PeakQC reads raw instrument formats, not precomputed QC tables
- Only a single run is available with no batch context — PCA-based batch QC is designed to compare a batch of runs, not assess one run in isolation

## Inputs

- Raw MS files for a batch of runs in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format (LC-MS, LC-IMS-MS, DDA/DIA with or without fragmentation, or direct infusion)
- MS1 data within those raw files (basis for the time-versus-m/z images)
- Optional user-specified sample groups for coloring the PCA

## Outputs

- Initial PCA figure (PDF) of the batch, with samples/runs colored by group
- Time-versus-m/z image representations of each MS run (intermediate)
- Complementary QC outputs: CSV reports and JPG/PDF diagnostic figures for peak metrics of auto-tracked and user-specified ions

## How to apply

Load the batch of raw MS files into PeakQC (available within the IonToolPack GUI, which reads Agilent 'd', Thermo '.raw', Bruker 'd', and mzML and requires no installation) and run the processing step that converts MS1 data into a time-versus-m/z image per MS run. PeakQC's QC evaluation module then performs PCA on these image representations, projecting each sample onto the leading principal components. Color PCA points by automatically assigned or user-specified sample groups so batch outliers and group-separating trends are visible. Export the initial PCA figure as a PDF for later QC inspection alongside the JPG/PDF diagnostic figures and CSV reports. Follow up flagged outliers with PeakQC's detailed assessment: peak-metric extraction for auto-tracked (topmost abundant MS1) or user-specified ions and Isolation Forest anomaly detection per sample group, since PCA identifies compromised runs but does not pinpoint whether the cause is chromatography, sensitivity, or mass calibration.

## Related tools

- **PeakQC** (Core tool: generates time-versus-m/z images from MS1 data and performs the batch PCA for global quality assessment; part of IonToolPack) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI suite housing PeakQC; imports raw MS files in multiple vendor formats and runs the 'Process' step that produces the QC outputs) — https://github.com/pnnl/IonToolPack

## Examples

```
Download IonToolPack.exe from https://github.com/pnnl/IonToolPack/releases, decompress, double-click, import the batch of raw MS files (Thermo .raw/Agilent .d/Bruker .d/mzML) in the PeakQC tab, and click "Process"
```

## Evaluation signals

- A PCA figure (PDF) is produced in which each point corresponds to one MS run in the batch and points are colored by assigned or user-specified sample groups
- The number of points in the PCA matches the number of MS runs loaded, confirming every run was converted to a time-versus-m/z image from MS1 data
- Known defective/compromised runs (e.g., those with instrument performance issues) appear as outliers or drive group separation on the leading principal components, as illustrated in the paper's Figure 2
- The PCA output is accompanied by the expected companion QC artifacts: CSV peak-metric reports and JPG/PDF diagnostic figures for auto-tracked and user-specified ions
- Isolation Forest error heatmaps on individual peak metrics per sample group corroborate the outliers seen in the PCA view

## Limitations

- PCA flags compromised runs but does little to pinpoint specific causes of performance deterioration (e.g., chromatography, sensitivity, mass calibration) — additional PeakQC metric extraction is needed for diagnosis
- Requires raw MS data in supported vendor formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML); conversion from raw data is part of the pipeline's total execution time
- PeakQC is distributed as a Windows executable (IonToolPack.exe) with no pip/conda/docker installation path documented, and no changelog is available

## Evidence
