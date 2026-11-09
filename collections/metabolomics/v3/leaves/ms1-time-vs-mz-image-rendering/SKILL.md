---
name: ms1-time-vs-mz-image-rendering
description: Use when assessing global quality of a batch of untargeted MS runs (LC-MS,
  LC-IMS-MS, direct infusion, DDA/DIA) before detailed peak-metric QC, and you need
  a fast, identification-free representation per sample.
when_to_use_negative:
- You need well-resolved individual peak detail or peak-shape metrics — the image
  rendering deliberately distorts intensity (log-transform) and does not resolve individual
  peaks; use PeakQC's auto-tracked ion peak-metric extraction instead.
- Data are identification-based (spectral-match) QC targets, e.g. lipid identification
  rates from LIQUID — this rendering is identification-free and does not use spectral
  matches.
- The run is already a processed feature table rather than raw MS1 data — PeakQC requires
  raw vendor or mzML input.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: Executes the fast rendering algorithm on MS1 data to build the time-versus-m/z
    image and export JPG images for PCA
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Standalone Windows GUI (IonToolPack.exe) that houses PeakQC and imports the
    raw MS files; no installation required
  repo: https://github.com/pnnl/IonToolPack/releases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:829a6d60dc7e1b628ef9803ed6c4078cb036e03a9ed9919aecb2cf8662c8c11e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ms1-time-vs-mz-image-rendering

## Summary

Render each MS run in a batch as a two-dimensional time-versus-m/z image from MS1 data so that a PCA across the batch of images reveals global sample similarity and outliers in an omics-agnostic way.

## When to use

Use when assessing global quality of a batch of untargeted MS runs (LC-MS, LC-IMS-MS, direct infusion, DDA/DIA) before detailed peak-metric QC, and you need a fast, identification-free representation per sample.

## When NOT to use

- You need well-resolved individual peak detail or peak-shape metrics — the image rendering deliberately distorts intensity (log-transform) and does not resolve individual peaks; use PeakQC's auto-tracked ion peak-metric extraction instead.
- Data are identification-based (spectral-match) QC targets, e.g. lipid identification rates from LIQUID — this rendering is identification-free and does not use spectral matches.
- The run is already a processed feature table rather than raw MS1 data — PeakQC requires raw vendor or mzML input.

## Inputs

- Raw MS data files (Agilent 'd' directories, Thermo '.raw', Bruker 'd', mzML)
- MS1 data from each run in the batch (one or more samples; LC-MS, LC-IMS-MS, direct infusion, DDA/DIA with or without fragmentation)

## Outputs

- One JPG time-versus-m/z image per MS run/sample
- Per-sample image set used as input for PCA of the batch of samples

## How to apply

Import the raw MS files into IonToolPack's PeakQC tool, which reads Agilent 'd', Thermo '.raw', Bruker 'd', and mzML. Run PeakQC's fast rendering algorithm on the MS1 data of each run, building a two-dimensional time-versus-m/z image (one image per sample in the batch). The intensity is log-transformed for this representation; individual peaks are intentionally not well resolved because the image is meant to capture global similarity patterns, not peak detail. Export one JPG image per run, then use the per-sample image set downstream for PCA of the batch to flag runs whose global pattern deviates from the group. Rationale: identification-based QC tools do little to pinpoint the causes of performance deterioration, whereas the time-vs-m/z image gives a bias-free, omics-agnostic global view of instrument behavior (chromatography, sensitivity, ion-mobility behavior) without needing spectral matches.

## Related tools

- **PeakQC** (Executes the fast rendering algorithm on MS1 data to build the time-versus-m/z image and export JPG images for PCA) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Standalone Windows GUI (IonToolPack.exe) that houses PeakQC and imports the raw MS files; no installation required) — https://github.com/pnnl/IonToolPack/releases

## Examples

```
1. Download the latest IonToolPack.exe from https://github.com/pnnl/IonToolPack/releases, decompress it
2. Double click IonToolPack.exe
3. Import raw MS files and click "Process" to generate the time-vs-m/z JPGs for PCA
```

## Evaluation signals

- One JPG image is produced per MS run in the batch (1:1 sample-to-image correspondence).
- Each image is a 2D time-versus-m/z plot generated from MS1 data only, not MS2.
- Intensities in the image are log-transformed (distorted) — a log-scaled intensity axis confirms the correct rendering mode.
- The exported image set feeds a PCA across the batch; PCA output should reflect global similarity grouping of samples rather than peak-level metrics.
- PCA-identified outlier runs should correspond to samples with visibly deviating global patterns in their images (e.g. abnormal chromatographic or infusion profiles).

## Limitations

- The intensity is log-transformed and therefore distorted, and individual peaks are not well resolved in the time-vs-m/z representation — it is sufficient only for capturing global similarity patterns.
- PeakQC's rendering supports a fixed set of input formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML); other vendor formats must be converted first.
- The graphical-interface workflow (download IonToolPack.exe, decompress, double-click) is Windows-oriented; unattended command-line batch QC across months and years was listed as planned functionality.

## Evidence
