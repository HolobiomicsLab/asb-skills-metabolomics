---
name: unsupervised-dimensionality-reduction-of-spectral-images
description: Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion,
  DDA/DIA) needs rapid, identification-free quality assessment and you want outliers
  to emerge from an unsupervised projection rather than per-run metrics.
when_to_use_negative:
- Input is already a processed feature table or identification-based metrics (e.g.,
  spectral match counts) rather than raw MS1 data — PeakQC's PCA works on time-vs-m/z
  images from MS1, not on annotation tables.
- The goal is pinpointing the specific cause of performance deterioration (chromatography
  vs sensitivity vs mass calibration) — identification-based/PCA views flag outliers
  but PeakQC's detailed metrics extraction and error heatmaps are needed for cause
  diagnosis.
- A single run needs detailed per-ion inspection rather than batch-level comparison
  — use PeakQC's tracked-ion metrics or Mirador visualization instead of the batch
  PCA.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakQC
  role: Generates time-versus-m/z images from MS1 data and runs the PCA of the batch,
    coloring samples by groups; part of the IonToolPack GUI
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Executable suite housing PeakQC; imports raw MS files in multiple vendor formats
    and runs the QC pipeline via a 'Process' click
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:4f3f4c96c58b2a9ac4caf147c3daf8c08f1f73a69f44d49cd8f31af8b661fbce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# unsupervised-dimensionality-reduction-of-spectral-images

## Summary

Perform batch-level PCA on time-versus-m/z images generated from MS1 data of each MS run, so that compromised or outlier runs are visually separated along leading principal components and can be quickly spotted in untargeted MS quality control.

## When to use

Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion, DDA/DIA) needs rapid, identification-free quality assessment and you want outliers to emerge from an unsupervised projection rather than per-run metrics.

## When NOT to use

- Input is already a processed feature table or identification-based metrics (e.g., spectral match counts) rather than raw MS1 data — PeakQC's PCA works on time-vs-m/z images from MS1, not on annotation tables.
- The goal is pinpointing the specific cause of performance deterioration (chromatography vs sensitivity vs mass calibration) — identification-based/PCA views flag outliers but PeakQC's detailed metrics extraction and error heatmaps are needed for cause diagnosis.
- A single run needs detailed per-ion inspection rather than batch-level comparison — use PeakQC's tracked-ion metrics or Mirador visualization instead of the batch PCA.

## Inputs

- Raw MS files per run (Agilent '.d', Thermo '.raw', Bruker '.d', or mzML) for the batch
- Time-versus-m/z images generated from MS1 data by PeakQC's prior processing step
- Optional user-specified sample groups for coloring PCA points

## Outputs

- PCA projection of the batch with each run as a point on leading principal components, colored by group
- PCA figure exported as PDF for QC inspection
- Outlier/batch-trend visual assessment used together with error heatmaps and CSV metric reports

## How to apply

First, for each MS run in the batch, generate a time-versus-m/z image from the MS1 data as part of the PeakQC processing step; this image representation is omics-agnostic and works for infusion, LC, and ion mobility (IMS) acquisitions with or without fragmentation spectra. Next, run principal component analysis on the collection of image representations using PeakQC's QC evaluation module within IonToolPack, which reads Agilent 'd', Thermo '.raw', Bruker 'd', and mzML files. Project each sample/run onto the leading principal components and color the points by automatically assigned or user-specified sample groups so that batch outliers or group-separating trends become visible. Export the resulting PCA figure as a PDF to inspect alongside PeakQC's other JPG/PDF diagnostic figures (error heatmaps from Isolation Forest, chromatograms, mobiligrams) and CSV metric reports. Rationale: PCA on whole-run MS1 images avoids dependence on spectral identifications, whose decline can stem from many causes (chromatography, sensitivity, mass calibration) and thus fails to isolate deteriorating runs.

## Related tools

- **PeakQC** (Generates time-versus-m/z images from MS1 data and runs the PCA of the batch, coloring samples by groups; part of the IonToolPack GUI) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Executable suite housing PeakQC; imports raw MS files in multiple vendor formats and runs the QC pipeline via a 'Process' click) — https://github.com/pnnl/IonToolPack

## Examples

```
Download IonToolPack from Releases, decompress, double-click IonToolPack.exe, import raw MS files for the batch, click "Process" — PeakQC then builds MS1 time-vs-m/z images and emits the PCA figure as PDF.
```

## Evaluation signals

- A PDF PCA figure is produced in which each MS run in the batch appears as one point on the leading principal components.
- Compromised/outlier runs visibly separate from the main cluster in the PCA view, matching the defective-run examples shown in Figure 2 of the paper.
- Points are colored by automatically assigned or user-specified sample groups, and group-separating trends are interpretable.
- The PCA figure is consistent with PeakQC's complementary outputs (Isolation Forest error heatmaps, tracked-ion CSV reports) for the same batch — outliers should not contradict across views.
- The images feeding the PCA were generated from MS1 data only, confirming the assessment is identification-free and omics-agnostic.

## Limitations

- The PCA view flags outliers but does not pinpoint the specific cause of performance deterioration (e.g., chromatography, sensitivity, mass calibration); complementary metric extraction is needed.
- The initial PCA figure is a visual, subjective QC aid; automated cross-batch comparison over months and years is only planned via command-line functionality.
- Requires the prior PeakQC step that converts raw data and generates MS1 images, so raw data conversion time contributes to total pipeline execution time.

## Evidence

- [results] group coloring: [evidence span withheld — anchor status: not_found]
