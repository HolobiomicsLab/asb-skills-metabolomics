---
name: scientific-figure-export-jpg-pdf
description: Use when a mass-spectrometry QC pipeline (PeakQC/IonToolPack) must render
  per-sample or per-metric diagnostic plots to image files for reporting or archival,
  or when heatmap-style error summaries of anomalous MS runs and metrics are needed
  alongside CSV outputs.
when_to_use_negative:
- When the input is already a QC report or figure set rather than raw or converted
  MS data — PeakQC operates on raw instrument files ('d', '.raw', 'd', mzML) after
  MZA conversion, not on pre-rendered images.
- When a vector-only archival format is required without a raster copy; the tool's
  native export is JPG plus PDF, not arbitrary image formats.
- When QC depends on spectral-library identification scores (e.g., identification-based
  tools that only report total matches); PeakQC deliberately avoids identification-centric
  QC so it cannot diagnose specific causes such as chromatography, sensitivity, or
  mass calibration from that angle.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Generates error heatmaps and diagnostic plots from QC results and exports
    them as JPG and PDF files
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI wrapper used to load raw MS files and execute PeakQC's pipeline, which
    triggers figure generation and export
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:63feb765280ab408f1abe768bf7a3511840cd5932627b53dd90c47ee536d548b
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
yes_rate: 0.7
n_claims: 10
---

# scientific-figure-export-jpg-pdf

## Summary

Export PeakQC's QC diagnostic figures (error heatmaps, extracted ion chromatograms, mobiligrams, MS/MS mirror plots, time-versus-m/z images) as raster JPG and vector PDF files alongside CSV reports, so quality-control results can be visually inspected, archived, and compared across MS batches.

## When to use

Use when a mass-spectrometry QC pipeline (PeakQC/IonToolPack) must render per-sample or per-metric diagnostic plots to image files for reporting or archival, or when heatmap-style error summaries of anomalous MS runs and metrics are needed alongside CSV outputs.

## When NOT to use

- When the input is already a QC report or figure set rather than raw or converted MS data — PeakQC operates on raw instrument files ('d', '.raw', 'd', mzML) after MZA conversion, not on pre-rendered images.
- When a vector-only archival format is required without a raster copy; the tool's native export is JPG plus PDF, not arbitrary image formats.
- When QC depends on spectral-library identification scores (e.g., identification-based tools that only report total matches); PeakQC deliberately avoids identification-centric QC so it cannot diagnose specific causes such as chromatography, sensitivity, or mass calibration from that angle.

## Inputs

- Outlier-detection results table from the Isolation Forest anomaly-detection step (peak metrics per sample/run with anomaly labels)
- Time-versus-m/z images generated from MS1 data for the batch of samples
- Extracted ion chromatogram (XIC) and ion mobility trace (mobiligram) data for auto-tracked and user-specified ions
- MS2 fragmentation spectra for user-specified ions with expected fragment ions and intensities
- Raw MS files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format (upstream, via MZA conversion)

## Outputs

- Error heatmaps (JPG and PDF) displaying defective MS runs and their corresponding defective metrics, sized by the number of issues detected
- Diagnostic figures for fragmentation spectra (JPG and PDF)
- Time-versus-m/z images, extracted ion chromatograms, and mobiligrams (JPG and PDF)
- Associated CSV reports (peak metrics, outlier flags) exported alongside the figures

## How to apply

Run PeakQC's QC pipeline so that peak metrics (mass error, retention time error, arrival time error) and Isolation Forest outlier labels are already computed; then invoke its figure-export routine to render the derived plots — including the final error heatmap in which rows are defective MS runs and columns are the defective ions/metrics flagged as issues — as JPG and PDF image files. Heatmap dimensions scale with the number of issues detected, so the export automatically accommodates variable numbers of defective runs and metrics. The same export also handles time-versus-m/z MS1 images for PCA of the batch, extracted ion chromatograms, mobiligrams (ion mobility traces), and MS/MS diagnostic figures for user-specified ions with expected fragments. Validate the export by confirming each figure exists in both JPG and PDF format and that heatmap content visibly encodes only the anomalous runs and their problematic metrics rather than all runs.

## Related tools

- **PeakQC** (Generates error heatmaps and diagnostic plots from QC results and exports them as JPG and PDF files) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI wrapper used to load raw MS files and execute PeakQC's pipeline, which triggers figure generation and export) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe → PeakQC tab → Import raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) → click “Process” to generate CSV reports and JPG/PDF figures including error heatmaps
```

## Evaluation signals

- Each exported figure exists in both JPG and PDF format in the output directory.
- Error heatmap row count equals the number of defective MS runs, and column count equals the number of defective ions/metrics flagged by Isolation Forest.
- Heatmap dimensions change in proportion to the number of issues detected (rows/columns not fixed).
- Heatmap content encodes only the anomalous runs and their problematic metrics, not all runs in the batch.
- CSV reports (peak metrics per sample/run, outlier labels) are present alongside the JPG/PDF files.

## Limitations

- Error heatmaps display only defective MS runs and their defective metrics; they do not show the full batch, limiting context about overall batch quality.
- Figure content depends on upstream Isolation Forest anomaly detection; if the outlier model flags false positives or misses anomalies, the exported heatmap will misrepresent run quality.
- Export requires IonToolPack.exe to be downloaded and run with raw MS files imported — it cannot be invoked on already-processed CSVs or figures alone.
- Only JPG and PDF image formats are produced; other formats (PNG, SVG, TIFF) are not mentioned in the article or README.

## Evidence

- [results] PeakQC's final step generates error heatmaps that display only the defective MS runs and their corresponding defective metrics, with heatmap size proportional to the number of issues detected: [evidence span withheld — anchor status: not_found]
- [results] rows = defective runs, columns = defective ions/metrics: [evidence span withheld — anchor status: not_found]
- [results] Export the heatmaps as JPG and PDF image files alongside the other generated reports.: [evidence span withheld — anchor status: not_found]
