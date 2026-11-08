---
name: batch-pca-quality-assessment
description: Use when a batch of raw MS files (Agilent 'd', Thermo '.raw', Bruker
  'd', or mzML) needs a rapid, label-free global quality overview to flag outlier
  runs or drift across samples before investing in targeted peak-metric QC.
when_to_use_negative:
- The analysis goal is targeted quantitation of specific analytes rather than global
  batch quality screening (PeakQuant is the better-suited tool in the same pack)
- Input is already a processed feature table or CSV list rather than raw MS data (Comparador
  handles feature-list comparison; PeakQC requires raw vendor or mzML input)
- Only a single isolated MS run is available with no batch context, since the PCA
  assessment is defined over a batch of samples
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PeakQC
  role: 'Executes the omics-agnostic QC pipeline: builds MS1 time-versus-m/z images
    per run, runs PCA for global batch assessment, then auto-tracks topmost abundant
    ions and applies Isolation Forest outlier detection on peak metrics'
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Performs the raw-data conversion step that feeds vendor files (Agilent 'd',
    Thermo '.raw', Bruker 'd') into the PeakQC MS1 imaging/PCA pipeline
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:dba732cc7095a2afa4887148e019fb70f17a27a3f920a78863ceee7e8fd71d76
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# batch-pca-quality-assessment

## Summary

A global, omics-agnostic batch QC step from PeakQC (IonToolPack) that converts each MS run in a batch into a time-versus-m/z image from MS1 data and applies PCA to expose outlying samples and instrument drift before any targeted metric extraction.

## When to use

Use when a batch of raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) needs a rapid, label-free global quality overview to flag outlier runs or drift across samples before investing in targeted peak-metric QC.

## When NOT to use

- The analysis goal is targeted quantitation of specific analytes rather than global batch quality screening (PeakQuant is the better-suited tool in the same pack)
- Input is already a processed feature table or CSV list rather than raw MS data (Comparador handles feature-list comparison; PeakQC requires raw vendor or mzML input)
- Only a single isolated MS run is available with no batch context, since the PCA assessment is defined over a batch of samples

## Inputs

- A batch of raw MS files in vendor formats supported by IonToolPack: Agilent 'd', Thermo '.raw', Bruker 'd', or mzML
- Acquisition types: LC-MS, LC-IMS-MS, direct infusion, with/without fragmentation spectra in DDA or DIA mode
- Optional user-specified ion target lists (theoretical/reference m/z, RT, AT values) for the subsequent detailed QC tier

## Outputs

- Time-versus-m/z images generated from MS1 data for every MS run/sample in the batch
- PCA score plot of the batch for global quality assessment
- CSV peak-metric reports and JPG/PDF ion-metric figures (for auto-tracked and user-specified ions)
- Isolation Forest error heatmaps flagging per-sample-group anomalies
- Diagnostic fragmentation-spectrum figures and mobiligrams (when IMS data and expected fragments are supplied)

## How to apply

Load the batch of raw MS files into IonToolPack.exe (the PeakQC tab) and click 'Process'; the pipeline first converts vendor raw data (via the MZA conversion step) so each MS run or sample in the batch is represented by a time-versus-m/z image generated from the MS1 data. PeakQC then computes a PCA on these MS1-derived images to assess global batch quality, visualizing the score plot alongside the image gallery so runs that separate from the main cluster indicate anomalous acquisition quality. This PCA pass is deliberately omics-agnostic and does not require identified features or ion target lists, making it the first-line screen; only after inspecting the PCA does one proceed to the detailed tier, where the topmost abundant MS1 ions are auto-tracked (or user-specified reference ions) and peak metrics (mass, RT, arrival time errors) are extracted with Isolation Forest outlier heatmaps. Validate the run by checking that the time-versus-m/z images, the PCA plot of the batch, CSV metric reports, and JPG/PDF figures all appear in the output folders, and compare against the example input/output files in the pnnl/IonToolPack test_data directory.

## Related tools

- **PeakQC** (Executes the omics-agnostic QC pipeline: builds MS1 time-versus-m/z images per run, runs PCA for global batch assessment, then auto-tracks topmost abundant ions and applies Isolation Forest outlier detection on peak metrics) — https://github.com/pnnl/IonToolPack
- **MZA** (Performs the raw-data conversion step that feeds vendor files (Agilent 'd', Thermo '.raw', Bruker 'd') into the PeakQC MS1 imaging/PCA pipeline) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe (PeakQC tab) -> import raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) -> click "Process"
```

## Evaluation signals

- Every MS run in the batch yields a corresponding time-versus-m/z image derived from its MS1 data — image count should equal sample count
- A PCA score plot covering the whole batch is present in the output folder; outlier runs visibly separate from the main cluster
- The full output set (images + PCA plot, CSV metric reports, JPG/PDF ion metric figures, Isolation Forest error heatmaps) is present in navigable output folders
- Generated outputs match the example input and output files published in the repository's test_data folder when run on the bundled example data
- The run completes all stages from raw data conversion through error heatmap generation without manual inspection steps

## Limitations

- Currently delivered as a GUI workflow (IonToolPack.exe, Windows executable downloaded from Releases); unattended command-line QC for large-scale cross-batch comparisons spanning months and years is planned functionality, not yet available
- The PCA assessment is global and unsupervised — it flags that a run is anomalous but, like identification-based QC, does not by itself pinpoint the specific cause of performance deterioration (e.g., chromatography, sensitivity, mass calibration); the subsequent peak-metric/Isolation Forest tier is needed for cause localization
- Requires vendor-convertible raw data; processed feature tables or identification result files are not valid input

## Evidence

- [results] each MS run or sample in the batch is represented by a time-versus-m/z image generated from the MS1 data: "each MS run or sample in the batch is represented by a time-versus-m/z image generated from the MS1 data."
- [readme] PeakQC: Automated quality control pipeline by PCA analysis on MS1 data: "PeakQC: Automated quality control pipeline by PCA analysis on MS1 data for global quality assessment, and detailed assessment with comprehensive metrics extraction and outlier detection"
