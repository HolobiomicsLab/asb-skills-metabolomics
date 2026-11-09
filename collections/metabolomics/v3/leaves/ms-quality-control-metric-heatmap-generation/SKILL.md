---
name: ms-quality-control-metric-heatmap-generation
description: Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion)
  has been processed through PeakQC's Isolation Forest anomaly detection on peak metrics
  and you need a compact visual report of which runs and which ions/metrics are defective.
when_to_use_negative:
- Input is already a feature table or quantified abundance matrix rather than raw
  MS runs needing QC metric visualization
- You need QC based on identification counts or spectral library matching instead
  of precursor-ion peak metrics — PeakQC explicitly avoids identification-based QC
  because a decrease in spectral matches could be caused by a variety of reasons
- The run data lack sample-group structure needed for PeakQC's per-sample-group outlier
  detection
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakQC
  role: Software suite that performs the Isolation Forest outlier detection on peak
    metrics and generates the error heatmaps as its final pipeline step
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Executable packaging PeakQC (with Mirador, TandemMatch, PeakQuant, Comparador
    tabs) that must be launched to run the QC pipeline and obtain the heatmaps
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:b076f2686ba13e51bf304e4fa7fe381d82af67aa95f11df6f7c95b6545828371
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ms-quality-control-metric-heatmap-generation

## Summary

Generates error heatmaps that display only the defective MS runs and their corresponding defective metrics, as determined by the Isolation Forest outlier detection on individual peak metrics per sample group in PeakQC.

## When to use

Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion) has been processed through PeakQC's Isolation Forest anomaly detection on peak metrics and you need a compact visual report of which runs and which ions/metrics are defective.

## When NOT to use

- Input is already a feature table or quantified abundance matrix rather than raw MS runs needing QC metric visualization
- You need QC based on identification counts or spectral library matching instead of precursor-ion peak metrics — PeakQC explicitly avoids identification-based QC because a decrease in spectral matches could be caused by a variety of reasons
- The run data lack sample-group structure needed for PeakQC's per-sample-group outlier detection

## Inputs

- Outlier-detection results table from the Isolation Forest step (peak metrics per sample/run with anomaly labels)
- Raw MS data (Agilent '.d', Thermo '.raw', Bruker '.d', mzML) previously converted within PeakQC's pipeline

## Outputs

- Error heatmap figures highlighting defective MS runs and their corresponding defective metrics
- Heatmap exports in JPG and PDF formats

## How to apply

Load the outlier-detection results produced by the Isolation Forest step, which uses binary trees with linear time complexity and low memory requirements to flag anomalies in individual peak metrics (e.g., mass error, retention time error, arrival time error) per sample group. Arrange only the defective MS runs and their corresponding defective metrics into a matrix — rows are defective runs, columns are the flagged ions/metrics — so that heatmap size is proportional to the number of issues detected. Render this matrix as heatmap figures that visually encode which runs and which metrics are outlying. Export the heatmaps as JPG and PDF image files alongside the other generated CSV and figure reports. This step is the final one of PeakQC's pipeline, following raw data conversion, MS1 time-versus-m/z image generation, auto-tracked ion selection, and peak metric extraction for auto-tracked and user-specified ions.

## Related tools

- **PeakQC** (Software suite that performs the Isolation Forest outlier detection on peak metrics and generates the error heatmaps as its final pipeline step) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Executable packaging PeakQC (with Mirador, TandemMatch, PeakQuant, Comparador tabs) that must be launched to run the QC pipeline and obtain the heatmaps) — https://github.com/pnnl/IonToolPack

## Examples

```
Download and run IonToolPack.exe; import raw MS files and click 'Process' to generate error heatmaps in JPG/PDF
```

## Evaluation signals

- Each heatmap file exists in both JPG and PDF format in the output directory
- Heatmap visually encodes the anomalous runs and their problematic metrics as determined by the Isolation Forest outliers — rows correspond to defective runs and columns to defective ions/metrics
- Heatmap dimensions (rows × columns) are proportional to the number of issues detected, with only defective runs shown (no all-good runs)
- Flagged metrics correspond to the peak metrics actually extracted (mass error, retention time error, arrival time error) and are consistent with the sample-group outlier-detection results

## Limitations

- Heatmaps only show defective runs and defective metrics — a clean, issue-free batch yields no or minimal heatmap content, so absence of a heatmap is not itself an error signal
- The Isolation Forest detects anomalies per sample group; runs from a group with too few samples may produce unreliable anomaly labels
- PeakQC is omics-agnostic but requires precursor-ion peak metrics (errors in mass, retention time, arrival time); it does not pinpoint causes of performance deterioration via identification counts, which is deliberate — identification-based tools 'do little to pinpoint the specific causes of performance deterioration'

## Evidence

- [results] PeakQC's final step generates error heatmaps: [evidence span withheld — anchor status: not_found]
