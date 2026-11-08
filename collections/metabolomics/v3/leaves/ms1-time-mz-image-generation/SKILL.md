---
name: ms1-time-mz-image-generation
description: Use when raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML)
  must be screened for global batch quality via MS1-derived time-versus-m/z images
  and PCA, before or alongside targeted peak-metric QC.
when_to_use_negative:
- The input is already a feature table or peak-metric CSV rather than raw MS data,
  so no MS1 image can be reconstructed
- Only MS2 fragmentation spectra are available without MS1 scans, since the images
  are generated from MS1 data
- The goal is targeted quantitation of known analytes rather than global batch QC
  screening (use PeakQuant-style targeted extraction instead)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakQC
  role: Runs the automated QC pipeline that converts raw data and generates the MS1
    time-versus-m/z images and batch PCA
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Performs the raw data conversion step preceding MS1 image generation
- name: IonToolPack
  role: GUI suite housing PeakQC; user imports raw MS files and clicks 'Process' to
    trigger image generation and PCA
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:daef61ce966991ec9c9ca2c1c2a674ecc3c6eb99c9d76ad9513858460e3c538c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ms1-time-mz-image-generation

## Summary

Generate time-versus-m/z images from MS1 data for each MS run in a batch within the PeakQC/IonToolPack pipeline, then perform PCA on the images for global, omics-agnostic quality assessment of the batch.

## When to use

Use when raw MS files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) must be screened for global batch quality via MS1-derived time-versus-m/z images and PCA, before or alongside targeted peak-metric QC.

## When NOT to use

- The input is already a feature table or peak-metric CSV rather than raw MS data, so no MS1 image can be reconstructed
- Only MS2 fragmentation spectra are available without MS1 scans, since the images are generated from MS1 data
- The goal is targeted quantitation of known analytes rather than global batch QC screening (use PeakQuant-style targeted extraction instead)

## Inputs

- Raw MS1 data files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion acquisitions
- A batch of samples from the same acquisition campaign

## Outputs

- Time-versus-m/z image per MS run/sample, generated from MS1 data
- PCA plot summarizing the batch of time-versus-m/z images
- Output folders organizing images together with CSV metric reports, JPG/PDF ion metric figures, and Isolation Forest error heatmaps

## How to apply

Load the raw MS files into IonToolPack (launch IonToolPack.exe from the downloaded release; no installation required) and click 'Process' to run the PeakQC pipeline from raw data conversion (via MZA) through image generation. For each MS run or sample in the batch, convert the MS1 data into a time-versus-m/z image. Perform PCA across the images of the batch to obtain a global quality assessment, revealing which samples cluster apart as potential outliers before the more detailed per-ion metric extraction (auto-tracked ions, user-specified ions, Isolation Forest error heatmaps). The images and the PCA plot are written into the output folders and can be validated against the example input and output files in the repository's test_data directory.

## Related tools

- **PeakQC** (Runs the automated QC pipeline that converts raw data and generates the MS1 time-versus-m/z images and batch PCA) — https://github.com/pnnl/IonToolPack
- **MZA** (Performs the raw data conversion step preceding MS1 image generation)
- **IonToolPack** (GUI suite housing PeakQC; user imports raw MS files and clicks 'Process' to trigger image generation and PCA) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import Agilent 'd'/Thermo .raw/Bruker 'd'/mzML files, click 'Process' to run PeakQC (raw conversion via MZA -> MS1 time-versus-m/z images -> PCA)
```

## Evaluation signals

- One time-versus-m/z image exists per MS run/sample in the batch, generated from MS1 data
- The output includes a PCA plot of the batch of images alongside the individual images
- The generated outputs match the example input and output files published in the test_data folder of pnnl/IonToolPack
- The run covers the full pipeline span, i.e. 'raw data conversion to error heatmap generation', confirming image generation executed in sequence with the other QC steps

## Limitations

- Images require MS1 data; acquisitions lacking MS1 scans cannot be imaged
- The PCA-based screen is global: it flags outlying samples but does not pinpoint specific causes of performance deterioration (e.g., chromatography, sensitivity, mass calibration), which require the downstream per-ion peak metrics
- The workflow shown in the README is GUI-driven (IonToolPack.exe); command-line/batch operation for automated comparisons across months and years is described as planned functionality

## Evidence

4. See example input and output files in"
