---
name: precursor-peak-metric-extraction
description: Use when raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML;
  LC-MS, LC-IMS-MS, DDA/DIA, direct infusion) need automated QC of precursor-ion peak
  metrics without identification-based scoring, e.g. batch-level metabolomics QC.
when_to_use_negative:
- Input is not raw MS data but already identified spectral matches — the article notes
  identification-based QC tools 'do little to pinpoint the specific causes of performance
  deterioration', so this skill is not for scoring identification outputs
- The task is interactive visualization or export of individual chromatograms/MS2
  mirror plots rather than batch QC — the Mirador tool in IonToolPack is the appropriate
  choice
- The goal is unattended, command-line QC across batches over months and years — per
  the discussion this is planned functionality, not yet delivered by the GUI workflow
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3366
tools:
- name: PeakQC
  role: 'Primary tool executing the omics-agnostic QC pipeline: MS1 time-versus-m/z
    image generation with PCA, auto-tracking of abundant ions, peak metric extraction,
    and Isolation Forest outlier detection'
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Handles the raw data conversion step that precedes metric extraction in the
    PeakQC pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:915f2c15679325b24465acbd4130aceea93b6ae948ad1beb60eb9621cace5790
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - MZA
  - PeakQC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3366
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precursor-peak-metric-extraction

## Summary

Automated, omics-agnostic extraction and inspection of precursor-ion peak metrics (errors in mass, retention time, arrival time) from raw MS data using PeakQC, which auto-tracks the topmost abundant MS1 ions or uses user-specified reference ions and flags outliers with Isolation Forest.

## When to use

Use when raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML; LC-MS, LC-IMS-MS, DDA/DIA, direct infusion) need automated QC of precursor-ion peak metrics without identification-based scoring, e.g. batch-level metabolomics QC.

## When NOT to use

- Input is not raw MS data but already identified spectral matches — the article notes identification-based QC tools 'do little to pinpoint the specific causes of performance deterioration', so this skill is not for scoring identification outputs
- The task is interactive visualization or export of individual chromatograms/MS2 mirror plots rather than batch QC — the Mirador tool in IonToolPack is the appropriate choice
- The goal is unattended, command-line QC across batches over months and years — per the discussion this is planned functionality, not yet delivered by the GUI workflow

## Inputs

- Raw MS files in Agilent 'd', Thermo '.raw', Bruker 'd', or mzML format
- Optionally, user-specified target ions with theoretical/reference m/z, retention time, and arrival time values
- Optionally, expected fragment ions and intensities for MS2 diagnostic figures

## Outputs

- CSV metric reports of peak metrics for auto-tracked and user-specified ions
- JPG/PDF ion metric figures per ion
- Isolation Forest error heatmaps flagging outliers per sample group
- Time-versus-m/z images of each MS run with a PCA plot of the batch
- Mobiligrams (ion mobility traces) for user-specified ions when IMS data are present
- Diagnostic figures for fragmentation spectra (MS2) of user-specified ions

## How to apply

Run the PeakQC pipeline inside IonToolPack.exe on a batch of raw MS files: the pipeline first performs raw data conversion (via MZA), then generates a time-versus-m/z image from MS1 data for each run and a PCA plot of the batch. It selects the topmost abundant MS1 ions, most frequent per sample group, as 'auto-tracked' ions, and extracts peak metrics — mass error, retention time error, and arrival time error — for both auto-tracked ions and user-specified ions given as theoretical or reference values. Results are written as CSV reports plus JPG and PDF figures per ion. Outlier detection on individual peak metrics per sample group is performed with the Isolation Forest algorithm (chosen for its linear time complexity and low memory requirement) to produce error heatmaps. If IMS data with AT in the input are supplied, mobiligrams are extracted for user-specified ions; if expected fragment ions and intensities are provided, MS2 diagnostic figures are generated per molecule and per MS run.

## Related tools

- **PeakQC** (Primary tool executing the omics-agnostic QC pipeline: MS1 time-versus-m/z image generation with PCA, auto-tracking of abundant ions, peak metric extraction, and Isolation Forest outlier detection) — https://github.com/pnnl/IonToolPack
- **MZA** (Handles the raw data conversion step that precedes metric extraction in the PeakQC pipeline)

## Evaluation signals

- Output folders contain all expected products: time-versus-m/z images with the batch PCA plot, CSV metric reports, JPG/PDF ion metric figures, and error heatmaps
- Generated outputs match the example input and output files published in the repository's test_data folder
- The auto-tracked ion list corresponds to the topmost abundant MS1 ions that are most frequent per sample group
- Error heatmaps show per-sample-group outlier detection results from Isolation Forest, and the run spans raw data conversion through heatmap generation
- Figures appear in both JPG and PDF for each ion, with CSV reports accompanying each metric set

## Limitations

- Command-line functionality for unattended QC evaluations and large-scale comparisons across batches is planned, not yet available; the current workflow is GUI-driven (IonToolPack.exe)
- No explicit standalone installation instructions (pip/conda/docker) are provided; users must download and decompress IonToolPack.exe from the GitHub Releases section
- MS2 diagnostic figures are only generated for user-specified ions if their expected fragment ions and intensities are included in the input; mobiligrams require AT data and IMS separation in the input

## Evidence
