---
name: pdf-figure-export-of-qc-diagnostics
description: Use when a PeakQC QC run has completed over a batch of MS runs (Thermo
  .raw, Agilent/Bruker 'd', mzML) and the resulting PCA projection and diagnostic
  figures need to be saved as PDF/JPG reports for QC inspection.
when_to_use_negative:
- The goal is real-time, unattended QC monitoring across batches over months and years
  rather than static, human-inspectable figure exports (planned command-line functionality
  for large-scale comparisons is described as future work)
- The user needs interactive, customizable visualization (e.g., adjustable m/z, RT,
  or arrival time ranges and tolerances) rather than the fixed PeakQC diagnostic exports
  — Mirador in IonToolPack is the interactive option
- Input is a feature table or already-summarized QC metrics rather than raw MS data
  from which PeakQC generates its images and figures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Generates the PCA figure, error heatmaps, and JPG/PDF diagnostic plots that
    are exported for QC inspection
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI suite housing PeakQC; supports raw MS data visualization and export (PDF,
    CSV) including XICs, XIM heatmaps, and MS/MS mirror plots
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:8a041c4497a0cf75c70ef7f378d61f663b8646cf5fa43160fc94b81018988276
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pdf-figure-export-of-qc-diagnostics

## Summary

Export PeakQC's batch PCA figure and per-ion diagnostic plots (JPG and PDF) as static image files so that compromised MS runs can be inspected, archived, and compared offline across a batch.

## When to use

Use when a PeakQC QC run has completed over a batch of MS runs (Thermo .raw, Agilent/Bruker 'd', mzML) and the resulting PCA projection and diagnostic figures need to be saved as PDF/JPG reports for QC inspection.

## When NOT to use

- The goal is real-time, unattended QC monitoring across batches over months and years rather than static, human-inspectable figure exports (planned command-line functionality for large-scale comparisons is described as future work)
- The user needs interactive, customizable visualization (e.g., adjustable m/z, RT, or arrival time ranges and tolerances) rather than the fixed PeakQC diagnostic exports — Mirador in IonToolPack is the interactive option
- Input is a feature table or already-summarized QC metrics rather than raw MS data from which PeakQC generates its images and figures

## Inputs

- PeakQC PCA projection of time-versus-m/z images for each MS run in a batch, with sample group assignments
- Extracted peak metrics for auto-tracked and user-specified ions (mass error, RT error, arrival time error)
- Isolation Forest outlier-detection results per sample group
- MS2 signal extractions for user-specified ions with expected fragment ions and intensities
- Mobiligrams (ion mobility traces) for user-specified ions when arrival time is in the input and the data contain IMS separation

## Outputs

- PDF figure of the initial PCA of the batch (samples colored by group)
- JPG and PDF diagnostic figures (error heatmaps, fragmentation-spectrum plots, mobiligrams)
- CSV metric reports accompanying the figures

## How to apply

After PeakQC generates a time-versus-m/z image per MS run from MS1 data and runs PCA of the batch, project each sample onto the leading principal components and export the initial PCA figure as a PDF, coloring points by automatically assigned or user-specified sample groups so outliers and group-separating trends are visible. Alongside the PCA export, PeakQC writes CSV reports and JPG and PDF figures covering extracted peak metrics (mass, retention time, and arrival time errors) for auto-tracked and user-specified ions, Isolation Forest error heatmaps, MS/MS diagnostic plots, and mobiligrams when IMS separation is present. Keep the PDF exports as the archival/inspectable QC record alongside the raw machine-readable CSVs; the IonToolPack GUI also lets the user export visualizations (e.g., XICs, XIM heatmaps, MS/MS mirror plots) to PDF or CSV. The step sits at the end of the pipeline, after raw data conversion through error heatmap generation, so exported figures reflect the full set of computed metrics.

## Related tools

- **PeakQC** (Generates the PCA figure, error heatmaps, and JPG/PDF diagnostic plots that are exported for QC inspection) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI suite housing PeakQC; supports raw MS data visualization and export (PDF, CSV) including XICs, XIM heatmaps, and MS/MS mirror plots) — https://github.com/pnnl/IonToolPack

## Examples

```
Double click IonToolPack.exe; import raw MS files; click "Process"; collect the exported PCA PDF plus JPG/PDF diagnostic figures and CSV reports from the output folder
```

## Evaluation signals

- An initial PCA PDF exists and shows samples colored by automatically assigned or user-specified groups, with defective runs visually separable as in Figure 2 of the paper
- Both JPG and PDF versions of the diagnostic figures are present next to CSV reports for auto-tracked and user-specified ions
- Error heatmaps reflect Isolation Forest outlier detection per sample group and agree with runs flagged as compromised in the PCA view
- The exported figures cover the whole pipeline window, from raw data conversion through error heatmap generation (PeakQC reports total execution time over all steps)
- Example input and output files in the IonToolPack test_data directory can be used to check expected figure/report structure

## Limitations

- Figure export is a static snapshot; large-scale, unattended QC comparisons across batches over months and years require planned command-line functionality not yet available
- MS2 diagnostic figures for user-specified ions are only generated if expected fragment ions and intensities are included in the input; mobiligrams require arrival time in the input and IMS separation in the data
- IonToolPack must be run via the Windows executable (IonToolPack.exe); no pip/conda/docker installation instructions are provided

## Evidence

- [other] Project each sample/run onto the leading principal components and color the points by sample group: [evidence span withheld — anchor status: not_found]
- [other] Export the initial PCA figure as a PDF file: [evidence span withheld — anchor status: not_found]
