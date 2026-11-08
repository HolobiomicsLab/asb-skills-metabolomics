---
name: precursor-ion-peak-metrics-analysis
description: Use when QC'ing a batch of LC-MS/LC-IMS-MS or infusion runs (Agilent
  '.d', Thermo '.raw', Bruker '.d', mzML) by profiling precursor-ion errors and flagging
  anomalous runs per metric and sample group rather than eyeballing spectra manually.
when_to_use_negative:
- When the goal is a single combined QC score for the whole batch — PeakQC performs
  outlier detection on individual peak metrics per sample group and does not collapse
  them into one aggregate metric.
- When the data are already summarized feature tables (e.g., from external processing
  software) rather than raw vendor/mzML files; PeakQC expects raw MS runs it can convert
  and parse itself.
- When identification-based QC (e.g., counting spectral matches in proteomics) is
  sufficient — the paper notes such tools 'do little to pinpoint the specific causes
  of performance deterioration, because a decrease in the total number of spectral
  matches could be caused by a variety of reasons'.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: 'Primary implementation: extracts precursor-ion peak metrics, runs per-metric
    per-group Isolation Forest outlier detection, and generates error heatmaps.'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Graphical wrapper that reads raw MS files and hosts PeakQC's QC pipeline (PCA
    on MS1 plus metrics extraction and outlier detection).
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:aa2c34d165b710fa302faf15ff457ea474c00194a0bf491f733148110cac9664
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precursor-ion-peak-metrics-analysis

## Summary

Automated extraction and inspection of precursor-ion peak metrics (mass error, retention time error, arrival time error) in MS runs, with per-metric per-sample-group Isolation Forest outlier detection and error heatmap reporting as implemented in PeakQC.

## When to use

Use when QC'ing a batch of LC-MS/LC-IMS-MS or infusion runs (Agilent '.d', Thermo '.raw', Bruker '.d', mzML) by profiling precursor-ion errors and flagging anomalous runs per metric and sample group rather than eyeballing spectra manually.

## When NOT to use

- When the goal is a single combined QC score for the whole batch — PeakQC performs outlier detection on individual peak metrics per sample group and does not collapse them into one aggregate metric.
- When the data are already summarized feature tables (e.g., from external processing software) rather than raw vendor/mzML files; PeakQC expects raw MS runs it can convert and parse itself.
- When identification-based QC (e.g., counting spectral matches in proteomics) is sufficient — the paper notes such tools 'do little to pinpoint the specific causes of performance deterioration, because a decrease in the total number of spectral matches could be caused by a variety of reasons'.

## Inputs

- Raw MS runs in Agilent '.d', Thermo '.raw', Bruker '.d', or mzML format
- Batch of samples organized into defined sample groups (e.g., QC, control, treatment batches)
- User-specified ion list with theoretical/reference m/z, retention time, and (for IMS) arrival-time values (optional; auto-tracked ions derived from MS1 data otherwise)

## Outputs

- CSV reports of extracted peak metrics (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions
- Outlier table (CSV) listing anomalous MS runs with metric, sample group, and run identifiers
- Error heatmaps (JPG and PDF) visualizing per-metric deviations of flagged runs
- Time-versus-m/z images from MS1 data used for PCA of the batch

## How to apply

1. Convert/import raw vendor files into PeakQC (IonToolPack) which reads Agilent '.d', Thermo '.raw', Bruker '.d', and mzML across LC-MS, LC-IMS-MS, DDA/DIA with or without fragmentation, and direct infusion acquisitions. 2. Determine 'auto-tracked' ions as the topmost abundant ions in the MS1 data that are most frequent per sample group, and/or supply user-specified ions with theoretical or reference m/z, RT, and arrival-time values. 3. Extract peak metrics for both auto-tracked and user-specified ions — errors in mass, retention time, and arrival time — and emit CSV reports plus JPG/PDF figures. 4. Partition runs by sample group and fit an Isolation Forest model separately for each peak metric within each group; the algorithm splits on binary trees, has linear time complexity and low memory requirement, scoring every run per metric/group and labeling anomalous runs. 5. Compile flagged runs into an outlier table (CSV) with metric, sample group, and run identifiers, and generate error heatmaps (JPG/PDF) that visualize the per-metric deviations of the flagged runs. The per-metric/per-group design exists because anomaly detection operates locally rather than producing a combined QC score, so mis-calibration, chromatographic drift, or sensitivity loss each surface in their own metric.

## Related tools

- **PeakQC** (Primary implementation: extracts precursor-ion peak metrics, runs per-metric per-group Isolation Forest outlier detection, and generates error heatmaps.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Graphical wrapper that reads raw MS files and hosts PeakQC's QC pipeline (PCA on MS1 plus metrics extraction and outlier detection).) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import raw MS files (Agilent '.d', Thermo '.raw', Bruker '.d', mzML) and click “Process” to run PeakQC's PCA + metrics extraction + Isolation Forest outlier detection
```

## Evaluation signals

- The outlier table must cover every peak-metric × sample-group combination — mass error, retention time error, arrival time error for each group — with no combination missing.
- Every run flagged as anomalous in the outlier CSV must be consistent with the deviations rendered in the generated error heatmaps (JPG/PDF).
- Metrics must be extracted for both auto-tracked ions (topmost abundant, most frequent per sample group in MS1) and any user-specified ions with theoretical/reference values.
- For IMS acquisitions, mobiligrams (ion mobility traces) must appear for user-specified ions when arrival time is in the input and the MS data contain IMS separation.
- Heatmaps and CSV reports should exist in both JPG and PDF figure formats as reported outputs of the pipeline.

## Limitations

- Outlier detection is performed on individual peak metrics separately for each sample group using Isolation Forest, so it does not produce a combined QC score across metrics.
- Peak metrics require either auto-tracked ions (derived from most-frequent topmost abundant MS1 ions per group) or user-specified ions with theoretical or reference values; neither input means no metrics.
- Arrival-time (ion mobility) metrics and mobiligrams are only available when arrival time is included in the input and the MS data contain IMS separation.
- Execution time spans the whole chain from raw data conversion to error heatmap generation, so conversion cost is part of the QC pipeline.

## Evidence
