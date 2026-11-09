---
name: lc-ms-batch-inspection
description: Use when a batch of raw LC-MS (or LC-IMS-MS, DDA/DIA, infusion) files
  needs a fast, omics-agnostic global quality assessment to spot outlier runs or instrument
  drift prior to targeted peak-metric analysis.
when_to_use_negative:
- The goal is targeted, precursor-level QC (mass error, RT/arrival-time error metrics)
  rather than global batch-level inspection — PeakQC's auto-tracked/user-specified
  ion pipeline covers that.
- The input is already a feature table, peak list, or image set — this skill operates
  on raw MS1 signal from individual instrument files.
- The analysis requires resolution of individual chromatographic peaks from the image;
  the log-transformed time-vs-m/z rendering deliberately distorts intensity and does
  not resolve individual peaks.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: Renders the log-transformed time-versus-m/z image per run from MS1 data and
    supports PCA-based global batch quality assessment
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: GUI housing PeakQC; imports raw MS files in supported vendor formats and processes
    them
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:658fb660f7f6f6ef6cfded623af074193faa06157479451962ece2498c431c16
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lc-ms-batch-inspection

## Summary

Batch-level visual inspection of LC-MS runs by rendering each sample as a log-transformed time-versus-m/z image from MS1 data, then running PCA over the batch to flag globally anomalous samples before deeper targeted QC.

## When to use

Use when a batch of raw LC-MS (or LC-IMS-MS, DDA/DIA, infusion) files needs a fast, omics-agnostic global quality assessment to spot outlier runs or instrument drift prior to targeted peak-metric analysis.

## When NOT to use

- The goal is targeted, precursor-level QC (mass error, RT/arrival-time error metrics) rather than global batch-level inspection — PeakQC's auto-tracked/user-specified ion pipeline covers that.
- The input is already a feature table, peak list, or image set — this skill operates on raw MS1 signal from individual instrument files.
- The analysis requires resolution of individual chromatographic peaks from the image; the log-transformed time-vs-m/z rendering deliberately distorts intensity and does not resolve individual peaks.

## Inputs

- Raw MS files per sample in a batch (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML)
- MS1 data for each run

## Outputs

- One JPG time-versus-m/z image per MS run
- PCA scores/plot over the batch of sample images for global quality assessment

## How to apply

For each MS run in the batch, run PeakQC's fast rendering algorithm on the MS1 data to build a two-dimensional time-versus-m/z image, with intensity log-transformed; per Figure 2A individual peaks are not well resolved, but this representation is sufficient to capture global similarity patterns. Import the raw files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) via IonToolPack, then export one JPG image per run and use the resulting per-sample image set as input to PCA across the batch to assess global quality. Judge correctness by confirming every run in the batch yields an image and that samples with known instrument issues cluster as outliers in the PCA scores plot.

## Related tools

- **PeakQC** (Renders the log-transformed time-versus-m/z image per run from MS1 data and supports PCA-based global batch quality assessment) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (GUI housing PeakQC; imports raw MS files in supported vendor formats and processes them) — https://github.com/pnnl/IonToolPack

## Examples

```
# Via IonToolPack GUI: import raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML), click "Process" to render time-vs-m/z images and run PCA-based global QC
# No CLI invocation is documented; the tool is driven through IonToolPack.exe.
```

## Evaluation signals

- Every run in the batch produces exactly one JPG time-versus-m/z image; missing files indicate failed imports or unsupported formats
- Image intensities are log-transformed — expect compressed intensity ranges rather than linear peak heights
- PCA of the batch of images should cluster runs with shared global patterns and separate known-bad runs as outliers, as shown in Figure 2A-type analysis
- The representation must be built from MS1 data only — MS2/fragmentation signals should not appear in the image

## Limitations

- Intensity is log-transformed and thus distorted; individual peaks are not well resolved, so the image is suitable only for capturing global similarity patterns, not detailed peak inspection
- Derived from MS1 data only; not a substitute for detailed peak-metric or MS2 fragmentation QC
- Raw data conversion adds to total execution time for the pipeline, from conversion to error heatmap generation

## Evidence
