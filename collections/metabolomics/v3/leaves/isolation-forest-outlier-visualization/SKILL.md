---
name: isolation-forest-outlier-visualization
description: Use when a batch of MS runs (metabolomics, lipidomics, proteomics, or
  multiomics) needs automated per-metric outlier detection and you must visually pinpoint
  which runs and which ion metrics (mass, RT, arrival time errors, etc.) are defective.
when_to_use_negative:
- Input is already an identification-based QC summary (e.g., total number of spectral
  matches) and the goal is to interpret identification counts rather than inspect
  specific precursor-ion metrics — the article notes identification-based tools 'do
  little to pinpoint the specific causes of performance deterioration'.
- Data does not contain the precursor-ion peak metrics needed (no extracted mass,
  RT, or arrival time errors), so Isolation Forest has no per-metric features to score.
- The question is general batch quality assessment rather than pinpointing defective
  runs/metrics — PeakQC's PCA-on-MS1 step is the appropriate prior step in that case.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Performs per-sample-group Isolation Forest outlier detection on individual
    peak metrics of precursor ions and generates the error heatmaps; shipped within
    the IonToolPack GUI as the PeakQC tab.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:ed6f5609e58243640fa97917e9ea0b178cbe8578cc82d58e903572a28e779372
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - PeakQC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# isolation-forest-outlier-visualization

## Summary

A PeakQC pipeline step that applies the Isolation Forest algorithm to flag outlier peak metrics for individual precursor ions per sample group, then renders the flagged defective MS runs and their defective metrics as error heatmaps exported as JPG and PDF files alongside CSV reports.

## When to use

Use when a batch of MS runs (metabolomics, lipidomics, proteomics, or multiomics) needs automated per-metric outlier detection and you must visually pinpoint which runs and which ion metrics (mass, RT, arrival time errors, etc.) are defective.

## When NOT to use

- Input is already an identification-based QC summary (e.g., total number of spectral matches) and the goal is to interpret identification counts rather than inspect specific precursor-ion metrics — the article notes identification-based tools 'do little to pinpoint the specific causes of performance deterioration'.
- Data does not contain the precursor-ion peak metrics needed (no extracted mass, RT, or arrival time errors), so Isolation Forest has no per-metric features to score.
- The question is general batch quality assessment rather than pinpointing defective runs/metrics — PeakQC's PCA-on-MS1 step is the appropriate prior step in that case.

## Inputs

- Table of peak metrics (CSV) extracted per MS run for auto-tracked and user-specified precursor ions, with per-metric values such as mass error, retention time error, arrival time error
- Sample group assignments for each MS run in the batch
- Raw MS data (Agilent '.d', Thermo '.raw', Bruker '.d', or mzML) from infusion, LC-MS, or LC-IMS-MS acquisitions in DDA or DIA mode

## Outputs

- Error heatmap figures (JPG and PDF) where rows are defective MS runs and columns are defective ions/metrics, sized proportionally to the number of issues detected
- CSV reports flagging anomalous peak metrics per sample group

## How to apply

Run Isolation Forest on the table of peak metrics extracted for auto-tracked and user-specified ions, operating per sample group and per individual peak metric; the algorithm detects anomalies using binary trees, which keeps runtime linear and memory low, making it suitable for large batches. For each metric flagged as anomalous, PeakQC records the defective MS runs and the defective ions/metrics. Arrange flagged defective runs as rows and defective ions/metrics as columns into a matrix whose size scales with the number of issues detected, and render that matrix as error heatmap figures highlighting which runs and which metrics are outlying. Export the heatmaps as both JPG and PDF files together with the CSV reports generated in earlier PeakQC steps. The Isolation Forest step is intentionally omics-agnostic: the same metric-outlier detection works for infusion, LC-MS, LC-IMS-MS, and DDA/DIA data once peak metrics (e.g., mass error, retention time error, arrival time error) are extracted.

## Related tools

- **PeakQC** (Performs per-sample-group Isolation Forest outlier detection on individual peak metrics of precursor ions and generates the error heatmaps; shipped within the IonToolPack GUI as the PeakQC tab.) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # open PeakQC tab, import raw MS files (e.g. Thermo .raw, mzML), click "Process" to run Isolation Forest outlier detection and generate error heatmaps
```

## Evaluation signals

- Heatmap files exist in both JPG and PDF formats in the output directory alongside the CSV reports.
- Each rendered heatmap visually encodes only the defective MS runs and their corresponding defective metrics, as determined by the Isolation Forest outliers.
- Heatmap dimensions are proportional to the number of issues detected — rows match the count of flagged runs and columns match the count of flagged ions/metrics.
- Flagged metrics correspond to precursors tracked either as auto-tracked ions (topmost abundant ions per sample group) or as user-specified targets with theoretical/reference values.
- Total execution time is consistent with the Isolation Forest step's linear time complexity and low memory requirement when run on large batches.

## Limitations

- Error heatmaps display only defective runs and their defective metrics, so clean samples are not shown — a heatmap with few or no rows means no anomalies were flagged, not that the run set is empty.
- The skill depends on prior extraction of peak metrics for auto-tracked or user-specified ions; without a user-specified ion list PeakQC relies on topmost abundant MS1 ions as auto-tracked ions.
- Results are produced per sample group, so comparisons across different sample groups require separate outlier detections.

## Evidence

- [results] Outlier detection and heatmap rendering per sample group using Isolation Forest binary trees with linear time complexity and low memory requirement.: [evidence span withheld — anchor status: not_found]
- [results] Heatmaps display defective runs and metrics; heatmap size scales with number of issues.: [evidence span withheld — anchor status: not_found]
