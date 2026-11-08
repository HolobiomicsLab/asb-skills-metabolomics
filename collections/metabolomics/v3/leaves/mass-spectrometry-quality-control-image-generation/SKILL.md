---
name: mass-spectrometry-quality-control-image-generation
description: Use when performing batch-level quality assessment of untargeted LC-MS,
  LC-IMS-MS, or direct-infusion MS1 data (Agilent 'd', Thermo '.raw', Bruker 'd',
  or mzML) and you need a per-sample visual/PCA representation before extracting detailed
  peak metrics.
when_to_use_negative:
- When you already need detailed per-ion peak metrics (mass error, retention time
  error, arrival time error) — PeakQC's downstream auto-tracked/user-specified ion
  assessment is the appropriate step, not image generation.
- When the input is a pre-processed feature table or already-converted image — this
  skill operates on raw MS1 instrument files, not on CSV feature lists.
- When individual peak resolution is required — the article explicitly notes that
  'individual peaks are not well resolved' in this representation.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: Implements the fast rendering of MS1 data into time-versus-m/z images for
    each run; part of IonToolPack
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI wrapper that imports raw MS files, provides the 'Process' button, and
    houses PeakQC
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:4be12e41127812efba1097d40deb6fd6236d114837881c23a2f2a4a0ad0b5bb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectrometry-quality-control-image-generation

## Summary

Generates log-transformed time-versus-m/z images from MS1 data for each MS run using PeakQC, enabling PCA-based global quality assessment across a batch. This is the first, omics-agnostic stage of PeakQC's automated quality control pipeline.

## When to use

Use when performing batch-level quality assessment of untargeted LC-MS, LC-IMS-MS, or direct-infusion MS1 data (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) and you need a per-sample visual/PCA representation before extracting detailed peak metrics.

## When NOT to use

- When you already need detailed per-ion peak metrics (mass error, retention time error, arrival time error) — PeakQC's downstream auto-tracked/user-specified ion assessment is the appropriate step, not image generation.
- When the input is a pre-processed feature table or already-converted image — this skill operates on raw MS1 instrument files, not on CSV feature lists.
- When individual peak resolution is required — the article explicitly notes that 'individual peaks are not well resolved' in this representation.

## Inputs

- Raw MS files per run (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, or direct-infusion acquisitions, with/without fragmentation spectra (DDA or DIA)
- A batch of multiple MS runs from the same study for cross-sample comparison

## Outputs

- One time-versus-m/z JPG image per MS run (log-transformed MS1 intensity representation)
- A per-sample image set used as input for PCA-based global quality assessment of the batch

## How to apply

Import raw MS files into PeakQC via IonToolPack (supported formats: Agilent 'd', Thermo '.raw', Bruker 'd', and mzML; acquisitions may be LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion). For each MS run, run PeakQC's fast rendering algorithm on the MS1 data to build the two-dimensional time-versus-m/z image. The intensity is log-transformed to compress the wide dynamic range; individual peaks are not well resolved in this representation, but the log-transformed image is sufficient to capture global similarity patterns across samples. Export one JPG image per run, producing the per-sample image set used downstream for PCA of the batch. Judge correctness by checking that every sample yields a JPG, that the images appear as coherent 2D m/z-vs-RT landscapes (not sparse point plots), and that PCA of the resulting images groups replicates and separates outliers consistent with known batch quality.

## Related tools

- **PeakQC** (Implements the fast rendering of MS1 data into time-versus-m/z images for each run; part of IonToolPack) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI wrapper that imports raw MS files, provides the 'Process' button, and houses PeakQC) — https://github.com/pnnl/IonToolPack

## Examples

```
1. Download latest IonToolPack.exe from Releases and decompress 2. Double click IonToolPack.exe 3. Import raw MS files and click "Process" 4. See example input and output files in test_data.
```

## Evaluation signals

- A JPG file is produced for every input MS run with no missing samples.
- Each image shows a continuous 2D time-versus-m/z intensity landscape (log-transformed) rather than sparse peak annotations.
- PCA on the image set clusters technical replicates together and places instrument-quality outliers apart, consistent with the batch's known QC state.
- The image set is generated from MS1 data only; no MS2 fragmentation spectra are needed for this step.
- Runtime scales linearly enough that batch-level PCA can be run on typical instrument batches without manual per-file parameter tuning.

## Limitations

- The intensity is log-transformed and thus distorted; individual peaks are not well resolved in the time-versus-m/z image.
- The representation is intentionally coarse — it captures only global similarity patterns, not per-ion quantitative QC.
- PeakQC is omics-agnostic but the article notes existing QC tools were previously proteomics-focused; metabolomics/lipidomics users must rely on PeakQC's own metrics extraction for detailed assessment.
- No standalone pip/conda installation instructions are documented; the tool is distributed as an IonToolPack.exe release.

## Evidence
