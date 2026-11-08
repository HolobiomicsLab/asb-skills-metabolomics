---
name: isolation-forest-anomaly-detection
description: Use when a batch of raw MS runs (Agilent 'd', Thermo '.raw', Bruker 'd',
  or mzML) has had precursor-ion peak metrics extracted and individual anomalous runs
  per metric and per sample group must be identified and visualized.
when_to_use_negative:
- When the goal is a single combined QC score for the batch — PeakQC's anomaly detection
  operates per metric/per group rather than producing a combined QC score
- 'When input is still raw vendor spectra and peak metrics have not yet been extracted
  (run the full PeakQC pipeline first: raw data conversion, MS1 time-versus-m/z images,
  PCA, metric extraction)'
- When identification-based QC is sufficient, e.g. counting spectral matches in proteomics;
  such tools do little to pinpoint specific causes of performance deterioration, which
  is the gap this skill addresses
tools:
- name: PeakQC
  role: Implements the per-metric/per-sample-group Isolation Forest outlier detection
    and generates the error heatmaps within its automated QC pipeline
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Upstream raw data conversion step that feeds the PeakQC QC pipeline whose
    final step is Isolation Forest error heatmap generation
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:722b2dbb7af36fdb3a24e65fa0f0c62e7770c293d64a23ee914547129acd8092
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - MZA
  - PeakQC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# isolation-forest-anomaly-detection

## Summary

Anomaly detection of MS run quality in which PeakQC fits an Isolation Forest model separately for each precursor-ion peak metric (mass error, retention time error, arrival time error) within each sample group, flagging runs whose metric values are outliers relative to their group.

## When to use

Use when a batch of raw MS runs (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) has had precursor-ion peak metrics extracted and individual anomalous runs per metric and per sample group must be identified and visualized.

## When NOT to use

- When the goal is a single combined QC score for the batch — PeakQC's anomaly detection operates per metric/per group rather than producing a combined QC score
- When input is still raw vendor spectra and peak metrics have not yet been extracted (run the full PeakQC pipeline first: raw data conversion, MS1 time-versus-m/z images, PCA, metric extraction)
- When identification-based QC is sufficient, e.g. counting spectral matches in proteomics; such tools do little to pinpoint specific causes of performance deterioration, which is the gap this skill addresses

## Inputs

- CSV reports of extracted precursor-ion peak metrics (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions
- Sample group assignments for each MS run in the batch
- Raw MS files from which metrics were derived (Agilent 'd', Thermo '.raw', Bruker 'd', mzML; LC-MS, LC-IMS-MS, DDA/DIA, direct infusion)

## Outputs

- Structured outlier table (CSV) with metric, sample group, and run identifiers for every flagged anomalous run
- Error heatmaps (JPG/PDF) visualizing the per-metric deviations of the flagged runs

## How to apply

Load the CSV reports of extracted peak metrics (mass error, retention time error, arrival time error) computed for both auto-tracked and user-specified ions, then partition the MS runs by sample group so detection is performed within each group. Fit an Isolation Forest model separately for each peak metric within each sample group; the algorithm detects anomalies using binary-tree splitting, which gives linear time complexity and a low memory requirement. Score every MS run per metric/group and label runs whose metric value is anomalous relative to the group. Compile all flagged runs into a structured outlier table (CSV) containing metric, sample group, and run identifiers, then generate error heatmaps (JPG/PDF) visualizing the per-metric deviations of the flagged runs. Rationale: per-metric/per-group modeling avoids conflating different QC causes (chromatography, sensitivity, mass calibration) into one score, and supports omics-agnostic QC across metabolomics, lipidomics, and proteomics data.

## Related tools

- **PeakQC** (Implements the per-metric/per-sample-group Isolation Forest outlier detection and generates the error heatmaps within its automated QC pipeline) — https://github.com/pnnl/IonToolPack
- **MZA** (Upstream raw data conversion step that feeds the PeakQC QC pipeline whose final step is Isolation Forest error heatmap generation)

## Evaluation signals

- The output outlier table must cover every peak-metric × sample-group combination in the batch
- The flagged anomalous runs must be consistent with the deviations rendered in the generated error heatmaps
- Each flagged run is anomalous for a specific metric relative to its own sample group, not relative to the whole batch
- Detection completes with linear time complexity and low memory requirement, as expected of the binary-tree Isolation Forest implementation
- Heatmaps are produced in both JPG and PDF alongside the CSV outlier table in the output folders

## Limitations

- Detection is per metric and per sample group; no combined QC score is produced, so cross-metric aggregation is left to the analyst
- User-specified ions require expected theoretical or reference values to be included in the input for meaningful metric errors
- Detection quality depends on the sample-group partitioning; anomalous assessment is relative to the group, so mis-assigned groups can mask or fabricate outliers

## Evidence
