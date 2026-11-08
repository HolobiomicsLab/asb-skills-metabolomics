---
name: reference-ion-metric-comparison
description: Use when precursor-ion peak metrics have already been extracted and you
  need to package them as CSV reports and JPG/PDF figures that visualize each ion
  metric across a batch of MS runs for operator inspection.
when_to_use_negative:
- Peak metrics have not yet been extracted — run PeakQC's metric-extraction step first;
  this skill only packages and presents already-extracted metrics
- The goal is global batch quality assessment via PCA on time-versus-m/z MS1 images
  rather than per-ion metric inspection and tracking
- The task is identification-based QC (e.g., counting spectral matches) rather than
  precursor-ion peak-metric tracking, since identification-based QC does little to
  pinpoint specific causes of performance deterioration
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakQC
  role: 'Primary tool: its reporting module writes the CSV summary reports and its
    plotting module renders the JPG/PDF metric figures for auto-tracked and user-specified
    ions'
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:e7d8c1c5a42c6f3776b0aaccd6cb9b551496f2b1a38b8ca8f5587d57ed2bcad1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - PeakQC
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# reference-ion-metric-comparison

## Summary

Package extracted precursor-ion peak metrics (mass, retention time, and arrival time errors) for both auto-tracked and user-specified reference ions into per-ion/per-metric CSV reports and JPG/PDF figures using PeakQC's reporting and plotting modules, so an instrument operator can visually inspect and track QC across a batch of MS runs.

## When to use

Use when precursor-ion peak metrics have already been extracted and you need to package them as CSV reports and JPG/PDF figures that visualize each ion metric across a batch of MS runs for operator inspection.

## When NOT to use

- Peak metrics have not yet been extracted — run PeakQC's metric-extraction step first; this skill only packages and presents already-extracted metrics
- The goal is global batch quality assessment via PCA on time-versus-m/z MS1 images rather than per-ion metric inspection and tracking
- The task is identification-based QC (e.g., counting spectral matches) rather than precursor-ion peak-metric tracking, since identification-based QC does little to pinpoint specific causes of performance deterioration

## Inputs

- Extracted precursor-ion peak metrics table (mass error, retention time error, arrival time error) from the upstream metric-extraction step
- List of auto-tracked ions (topmost abundant MS1 ions most frequent per sample group)
- User-specified ions with theoretical or reference values (m/z, retention time, and optionally arrival time / expected fragment ions and intensities)
- Raw MS files in supported formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion acquisitions

## Outputs

- Per-ion and per-metric CSV summary reports
- JPG figures visualizing each ion metric across the batch of MS runs
- PDF figures visualizing each ion metric across the batch of MS runs
- Navigable output folders structured per ion and per metric

## How to apply

Load the peak-metrics table (mass error, retention time error, arrival time error) produced by PeakQC's metric-extraction step with pandas, then merge the metrics computed for auto-tracked ions (the topmost abundant MS1 ions most frequent per sample group) with those for user-specified ions given by theoretical or reference values. Call PeakQC's reporting module to write per-ion and per-metric CSV summary reports, and its plotting module to render JPG and PDF figures visualizing each ion metric across the batch of MS runs. Organize all CSV reports and JPG/PDF figures into navigable output folders structured per ion and per metric so the operator can navigate from an ion to its tracked metrics. This step follows PeakQC's core QC logic: automated extraction and inspection of peak metrics of precursor ions, so the reports are the human-facing artifact for tracking instrument performance across runs.

## Related tools

- **PeakQC** (Primary tool: its reporting module writes the CSV summary reports and its plotting module renders the JPG/PDF metric figures for auto-tracked and user-specified ions) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe → PeakQC tab: import raw MS files (Thermo .raw / mzML), specify user ions with theoretical values, click "Process"; outputs land in per-ion/per-metric folders as CSV reports and JPG/PDF figures
```

## Evaluation signals

- Every auto-tracked ion and every user-specified ion has both a CSV report and a rendered JPG/PDF figure in the output folders
- Figures correctly display the tracked metric values (mass error, retention time error, arrival time error) — verified by qualitative visual QC check
- Output folders are structured per ion and per metric, so navigating from an ion reaches all of its metric reports and figures
- The union of auto-tracked and user-specified ions in the reports matches the ions extracted upstream (no ion silently dropped in the merge)

## Limitations

- Reporting covers only precursor-ion peak metrics (mass, retention time, arrival time errors); diagnostic plots for fragmentation spectra and mobiligrams are separate PeakQC outputs
- Arrival-time metrics and mobiligrams are only available when arrival time (AT) is included in the input and the MS data contains IMS separation
- Traditional manual inspection of such spectra/reports is time-consuming and subjective, which is why PeakQC pairs reporting with automated outlier detection (Isolation Forest) — visual inspection alone should not be the sole QC gate

## Evidence
