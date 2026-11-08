---
name: batch-effect-correction-execution
description: Use when a quantitated MRM abundance table (e.g., MAT-file or xlsx output)
  contains QC samples such as batch-pooled QCs (BQCs) and abundance trends with injection
  order must be removed across batches.
when_to_use_negative:
- The sample files contain no QC samples — the article states batch-effect correction
  is conducted only if quality control (QC) samples are included in the sample files
- The goal is raw peak detection and quantitation rather than post-quantitation drift
  removal (use MRMQuant's peak detection/deconvolution steps instead)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Performs the automated QC-RLSC batch-effect correction on quantitated MRM
    abundances when QC samples are present
  repo: https://github.com/kslynn128171/MRMQuant
- name: MRMkit
  role: Source of the demonstration data set (3793 samples, 33 batches, 295 compounds,
    389 BQCs) and comparison tool whose default correction settings were benchmarked
  repo: https://github.com/MRMkit/MRMkit
- name: MATLAB
  role: Platform for MRMQuant; the corrected results are loaded from/saved as a binary
    MATLAB MAT-file
- name: Microsoft Excel
  role: Format for saving the corrected abundance matrix (xlsx)
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:f829b930e7a827bafca9719d7f84c332b32fa70065110b9d70864d6a3cba0a37
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  - MRMkit
  - Microsoft Excel
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# batch-effect-correction-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute QC-RLSC batch-effect correction on MRM-quantitated abundance matrices using MRMQuant's four-step LOESS-based procedure, to remove systematic technical drift across injection batches while preserving original abundance magnitudes.

## When to use

Use when a quantitated MRM abundance table (e.g., MAT-file or xlsx output) contains QC samples such as batch-pooled QCs (BQCs) and abundance trends with injection order must be removed across batches.

## When NOT to use

- The sample files contain no QC samples — the article states batch-effect correction is conducted only if quality control (QC) samples are included in the sample files
- The goal is raw peak detection and quantitation rather than post-quantitation drift removal (use MRMQuant's peak detection/deconvolution steps instead)

## Inputs

- MRMQuant quantitation output (binary MATLAB MAT-file or xlsx Excel file) containing abundances for all compounds and samples
- Sample annotation identifying QC samples (e.g., batch-pooled QCs/BQCs) and acquisition order within each batch

## Outputs

- Corrected abundance matrix for all compounds and samples (e.g., batch_corrected_abundances.xlsx)
- BQC-vs-injection-order regression fit before and after correction (slope and R2) for validation

## How to apply

Load the MRMQuant-quantitated abundances and identify QC samples within each batch (e.g., the 389 batch-pooled QCs in the MRMkit data set of 3793 samples in 33 batches, 295 compounds). For each compound, fit a LOESS curve to the compound's abundance in QC samples across injection order, then use cubic-spline interpolation/extrapolation of that LOESS curve to estimate the trend at every non-QC sample injection point (samples ordered by acquisition). Divide each sample's raw abundance by the interpolated QC trend to compute the correction ratio, then restore the corrected values to the original magnitude scale by multiplying the ratio by the reference sample's abundance. Save the corrected abundance matrix (e.g., as batch_corrected_abundances.xlsx) and validate that the regression of BQC abundances against injection order flattens.

## Related tools

- **MRMQuant** (Performs the automated QC-RLSC batch-effect correction on quantitated MRM abundances when QC samples are present) — https://github.com/kslynn128171/MRMQuant
- **MRMkit** (Source of the demonstration data set (3793 samples, 33 batches, 295 compounds, 389 BQCs) and comparison tool whose default correction settings were benchmarked) — https://github.com/MRMkit/MRMkit
- **MATLAB** (Platform for MRMQuant; the corrected results are loaded from/saved as a binary MATLAB MAT-file)
- **Microsoft Excel** (Format for saving the corrected abundance matrix (xlsx))

## Evaluation signals

- Regression of BQC abundances against injection order flattens after correction: reported change from slope 0.0004 with R2 = 0.3 to slope ~0.0 with R2 = 1.44 × 10–4
- Corrected abundances remain on the original magnitude scale (restored via ratios multiplied by the reference sample's abundance), comparable to raw values
- LOESS trend is fitted only to QC samples (e.g., 389 BQCs) within each batch, not to test samples
- All compounds and samples in the input matrix are present in the corrected output matrix

## Limitations

- Wall-clock processing time can be misleading: MRMQuant took around 5 h vs 4 h for MRMKit on the MRMkit data set, but the actual computation time is stated to be comparable
- LOESS interpolation/extrapolation depends on samples being ordered by acquisition time; samples without nearby QC coverage may rely on extrapolation of the cubic spline

## Evidence

- [abstract] fit a LOESS curve to the compound's abundance in the QC samples: "a LOESS curve is fitted to its abundance in the QC samples"
- [abstract] batch-effect correction is conducted if quality control (QC) samples are included in the sample files: "batch-effect correction is conducted if quality control (QC) samples are included in the sample files"
- [results] BQC slope/R2 before and after correction: "the slope of the regression line of the BQC samples (red dots) was 0.0004 with R2 = 0.3 before the correction but was very close to a flat line (slope ∼0.0) with R2 = 1.44 × 10–4"
- [results] processing time comparison: "Although the processing time of around 5 h may seem longer than that (4 h) of the MRMKit, the actual computation time should be comparable"
- [results] MRMkit dataset dimensions: "the data set consists of 3793 samples in 33 batches, each with 295 compounds. A total of 389 batch-pooled QCs (BQCs)"
- [abstract] results saved as xlsx: "The quantitation results are saved in a Microsoft Excel file in xlsx format"
