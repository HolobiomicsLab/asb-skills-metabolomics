---
name: defective-run-metrics-matrix-plotting
description: Use when finalizing an automated MS QC pipeline for a batch of samples
  and you must visualize which specific MS runs and which precursor-ion peak metrics
  were flagged anomalous, so instrument operators can pinpoint the failing runs and
  failing metrics at a glance.
when_to_use_negative:
- The batch contains no Isolation Forest outliers — PeakQC's heatmap is sized to the
  number of issues detected, so with zero defective runs there is nothing meaningful
  to plot; rely on the CSV metric reports and PCA of MS1 time-versus-m/z images instead.
- You need an interactive or global overview of all runs (defective and non-defective)
  — this heatmap intentionally displays only the defective runs and their defective
  metrics, not the complete run×metric matrix.
- Input is already a curated feature table with curated QC decisions — this skill
  is a reporting/visualization step within the PeakQC automated pipeline, not a manual
  curation tool.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: PeakQC
  role: Performs the Isolation Forest outlier detection on individual peak metrics
    per sample group and generates the resulting error heatmaps as JPG and PDF files
    in its final pipeline step.
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Standalone Windows executable housing PeakQC; reads raw MS files (Agilent
    'd', Thermo '.raw', Bruker 'd', mzML) with no installation and runs the QC pipeline
    through a tabbed GUI.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:7f4389460dde6f4172c774d228268cdebabd8d6e4ca7b342bd934f8ac1c0ac7f
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# defective-run-metrics-matrix-plotting

## Summary

Renders the outlier-detection results of PeakQC's Isolation Forest step as error heatmaps whose rows are defective MS runs and columns are their defective peak metrics (mass, retention time, arrival time errors), sized to the number of issues found and exported as JPG and PDF reports.

## When to use

Use when finalizing an automated MS QC pipeline for a batch of samples and you must visualize which specific MS runs and which precursor-ion peak metrics were flagged anomalous, so instrument operators can pinpoint the failing runs and failing metrics at a glance.

## When NOT to use

- The batch contains no Isolation Forest outliers — PeakQC's heatmap is sized to the number of issues detected, so with zero defective runs there is nothing meaningful to plot; rely on the CSV metric reports and PCA of MS1 time-versus-m/z images instead.
- You need an interactive or global overview of all runs (defective and non-defective) — this heatmap intentionally displays only the defective runs and their defective metrics, not the complete run×metric matrix.
- Input is already a curated feature table with curated QC decisions — this skill is a reporting/visualization step within the PeakQC automated pipeline, not a manual curation tool.

## Inputs

- Outlier-detection results table from PeakQC's Isolation Forest step, flagging anomalous peak metrics (mass error, RT error, arrival time error) per sample group and per MS run
- PeakQC processed batch of MS runs (Agilent 'd', Thermo '.raw', Bruker 'd', mzML; LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion)

## Outputs

- Error heatmap images in JPG format highlighting defective MS runs and their corresponding defective peak metrics
- Error heatmap images in PDF format (same content, vector export)
- Companion CSV reports of extracted peak metrics from the same pipeline run

## How to apply

Load the results table produced by PeakQC's Isolation Forest outlier detection, which labels per-sample-group anomalies in individual peak metrics (e.g., errors in mass, retention time, arrival time) for auto-tracked or user-specified precursor ions. Arrange the defective MS runs and their corresponding defective metrics into a matrix, then render it as an error heatmap whose size scales with the number of issues detected — rows are defective runs, columns are defective ions/metrics. PeakQC generates these heatmaps as the final step of its pipeline (after raw data conversion, MS1 time-versus-m/z images for PCA, peak-metric extraction, and MS2 diagnostic figures) and exports them alongside the CSV reports as both JPG and PDF figures. Correct application means every anomalous run/metric pair from the Isolation Forest output appears in the heatmap, and the output files exist in both JPG and PDF formats in the report directory. Because the heatmap deliberately shows only defective entries rather than the full run×metric table, absence of a run from the heatmap implies it was not flagged anomalous, not that it was omitted.

## Related tools

- **PeakQC** (Performs the Isolation Forest outlier detection on individual peak metrics per sample group and generates the resulting error heatmaps as JPG and PDF files in its final pipeline step.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Standalone Windows executable housing PeakQC; reads raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) with no installation and runs the QC pipeline through a tabbed GUI.) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import raw MS files (e.g. Thermo .raw / mzML), click 'Process' to run PeakQC's full pipeline through error heatmap generation
```

## Evaluation signals

- Each anomalous run/metric pair from the Isolation Forest output is present as a highlighted cell in the rendered heatmap.
- Heatmap files exist in both JPG and PDF formats in the report output directory alongside the CSV reports.
- Heatmap dimensions match the number of defective runs (rows) and defective ions/metrics (columns) detected in the batch.
- Heatmap content is restricted to defective entries — no non-flagged runs or metrics appear in the matrix.
- Metrics plotted correspond to the peak metrics extracted by PeakQC (errors in mass, retention time, arrival time) for auto-tracked or user-specified ions.

## Limitations

- Heatmaps display only the defective MS runs and their corresponding defective metrics, so the plot is empty or trivial when the batch is clean; the full run×metric matrix is not shown.
- Heatmap size scales with the number of issues detected, which can make very large batches with many outliers produce unwieldy plots.
- Depends on upstream steps (raw data conversion, MS1 time-versus-m/z PCA images, peak-metric extraction for auto-tracked or user-specified ions) having run correctly; errors there propagate to the heatmap.
- PeakQC distribution details (e.g., standalone pip/conda install paths) are not documented in the article or README; usage is via the IonToolPack.exe executable.

## Evidence

- [results] Error heatmap generation: [evidence span withheld — anchor status: not_found]
- [results] Isolation Forest detection: [evidence span withheld — anchor status: not_found]
- [readme] IonToolPack usage: [evidence span withheld — anchor status: not_found]
