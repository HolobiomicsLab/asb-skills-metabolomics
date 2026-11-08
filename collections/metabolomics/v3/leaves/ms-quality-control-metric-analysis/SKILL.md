---
name: ms-quality-control-metric-analysis
description: 'Use when raw MS runs from a batch need omics-agnostic QC: you want to
  flag anomalous runs by tracking mass error, RT error, or arrival time error for
  auto-tracked or user-specified precursor ions rather than relying on spectral-match
  counts.'
when_to_use_negative:
- When the QC question is cause attribution via identification-based metrics (e.g.,
  why total spectral matches dropped) — the article notes such tools 'do little to
  pinpoint the specific causes of performance deterioration'.
- When only a feature/peak table already exists and no raw MS data is available for
  metric extraction or PCA of the batch.
- 'When per-batch QC needs a single combined score: PeakQC operates per metric and
  per sample group rather than producing one overall QC value.'
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: 'Primary tool: extracts precursor-ion peak metrics, runs Isolation Forest
    outlier detection per metric/sample group, and generates error heatmaps and CSV
    reports.'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Distributes PeakQC via a no-install GUI; raw files are imported and 'Process'
    runs the QC pipeline from raw data conversion to error heatmap generation.
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Data conversion tool feeding raw MS data into the QC pipeline (raw data conversion
    is a step from raw data conversion to error heatmap generation).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:5661fa3b5d4fd846b2590aa1b9344ecc5e653e1927d2c9f7212da543aaf4b5c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ms-quality-control-metric-analysis

## Summary

Extract precursor-ion peak metrics (mass error, retention time error, arrival time error) from raw MS runs with PeakQC and apply Isolation Forest anomaly detection per metric and per sample group to flag outlier MS runs, generating error heatmaps of defective metrics.

## When to use

Use when raw MS runs from a batch need omics-agnostic QC: you want to flag anomalous runs by tracking mass error, RT error, or arrival time error for auto-tracked or user-specified precursor ions rather than relying on spectral-match counts.

## When NOT to use

- When the QC question is cause attribution via identification-based metrics (e.g., why total spectral matches dropped) — the article notes such tools 'do little to pinpoint the specific causes of performance deterioration'.
- When only a feature/peak table already exists and no raw MS data is available for metric extraction or PCA of the batch.
- When per-batch QC needs a single combined score: PeakQC operates per metric and per sample group rather than producing one overall QC value.

## Inputs

- Raw MS data files in Agilent '.d', Thermo '.raw', Bruker '.d', or mzML formats from LC-MS, LC-IMS-MS, direct infusion, or DDA/DIA acquisitions
- User-specified ion targets with theoretical or reference values (m/z, RT, AT, expected fragment ions and intensities)
- Sample group assignments for the batch of MS runs

## Outputs

- CSV reports of extracted precursor-ion peak metrics (mass error, retention time error, arrival time error) per ion and MS run
- Structured CSV outlier table listing anomalous MS runs with metric, sample group, and run identifiers
- Error heatmaps (JPG/PDF) visualizing per-metric deviations of flagged runs
- JPG/PDF diagnostic plots for fragmentation spectra and mobiligrams (ion mobility traces) for user-specified ions

## How to apply

First convert raw vendor files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) and generate time-versus-m/z images from MS1 data for PCA of the batch. Extract peak metrics — errors in mass, retention time, and arrival time — for the topmost abundant MS1 ions used as 'auto-tracked' ions and for any user-specified ions with theoretical/reference values, writing CSV reports and JPG/PDF figures. Then partition runs by sample group and fit an Isolation Forest model separately for each peak metric within each group, exploiting the algorithm's binary-tree splitting, linear time complexity, and low memory requirement. Score every MS run per metric/group, compile flagged anomalous runs into a structured CSV outlier table (metric, sample group, run identifiers), and generate error heatmaps (JPG/PDF) showing the per-metric deviations of the flagged runs. Optionally extract MS2 diagnostic figures and mobiligrams for user-specified ions with expected fragment ions and intensities.

## Related tools

- **PeakQC** (Primary tool: extracts precursor-ion peak metrics, runs Isolation Forest outlier detection per metric/sample group, and generates error heatmaps and CSV reports.) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Distributes PeakQC via a no-install GUI; raw files are imported and 'Process' runs the QC pipeline from raw data conversion to error heatmap generation.) — https://github.com/pnnl/IonToolPack
- **MZA** (Data conversion tool feeding raw MS data into the QC pipeline (raw data conversion is a step from raw data conversion to error heatmap generation).)

## Examples

```
Download IonToolPack.exe from the Releases page, decompress, double-click IonToolPack.exe, import raw MS files (Thermo .raw / mzML), assign sample groups, then click 'Process' to run conversion → peak metrics → Isolation Forest outlier detection → error heatmaps.
```

## Evaluation signals

- The output outlier table must cover every peak-metric × sample-group combination and the flagged anomalous runs must be consistent with the deviations rendered in the generated error heatmaps.
- Peak metrics for both auto-tracked and user-specified ions are extracted and written as CSV reports plus JPG/PDF figures.
- Outlier detection is performed within each sample group (partition-then-fit), not across the whole batch pooled together.
- The pipeline spans all steps from raw data conversion to error heatmap generation; verify no intermediate stage (conversion, metric extraction, heatmap) is skipped.
- Flagged runs should correspond to deviations in mass error, RT error, or AT error rather than to a drop in spectral identifications.

## Limitations

- Anomaly detection is per metric and per sample group — PeakQC does not emit a single combined QC score for a run or batch.
- Identification-based QC tools that only count spectral matches do little to pinpoint specific causes of performance deterioration (chromatography, sensitivity, mass calibration).
- Batch-vs-batch comparisons across months and years require planned command-line functionality for unattended QC evaluations.
- MS2 diagnostic figures and mobiligrams for user-specified ions require expected fragment ions and intensities (and AT in input) to be supplied.

## Evidence
