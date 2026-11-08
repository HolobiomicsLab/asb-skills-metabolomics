---
name: large-cohort-mrm-dataset-processing
description: Use when a large LC-MS/MS MRM dataset (e.g., thousands of samples spanning
  multiple batches with batch-pooled QCs/BQCs) must be peak-detected, quantitated,
  and batch-corrected, with the batch effect verified via BQC regression on injection
  order.
when_to_use_negative:
- The dataset is a small set of technical replicates without QC samples spanning batches
  — batch-effect correction is only conducted if QC samples are included in the sample
  files
- The goal is inter-tool comparison of coeluted/noisy peak handling rather than whole-cohort
  quantitation (the article shows MRMkit and Skyline had problematic quantitation
  of coeluted peaks; use a benchmarking workflow instead)
- Input is already a quantitated feature/abundance table rather than raw or converted
  MRM chromatogram files
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: 'Primary tool: automated peak detection, quantitation, and QC-RLSC batch-effect
    correction of large MRM cohorts'
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard
  role: msconvert converts vendor raw MRM data to the mzML/MAT-compatible formats
    MRMQuant loads
- name: MRMkit
  role: Source of the large-scale benchmark dataset (3793 samples, 33 batches) and
    comparison tool (~4 h processing vs ~5 h)
  repo: https://github.com/MRMkit/MRMkit
- name: LOESS
  role: Curve fitted to compound abundance in QC samples underlying the QC-RLSC correction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:ded16542a4113405a04866b337673914cb658af4950f22a76884356e6749ec0a
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
  - LOESS
  - MRMQuant
  - MRMkit
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# large-cohort-mrm-dataset-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processes large-scale targeted metabolomics MRM cohorts (thousands of samples across many batches) by loading MRM sample and method files, running automated peak detection and quantitation, and applying QC-RLSC batch-effect correction, as demonstrated in MRMQuant on the 3793-sample MRMkit benchmark.

## When to use

Use when a large LC-MS/MS MRM dataset (e.g., thousands of samples spanning multiple batches with batch-pooled QCs/BQCs) must be peak-detected, quantitated, and batch-corrected, with the batch effect verified via BQC regression on injection order.

## When NOT to use

- The dataset is a small set of technical replicates without QC samples spanning batches — batch-effect correction is only conducted if QC samples are included in the sample files
- The goal is inter-tool comparison of coeluted/noisy peak handling rather than whole-cohort quantitation (the article shows MRMkit and Skyline had problematic quantitation of coeluted peaks; use a benchmarking workflow instead)
- Input is already a quantitated feature/abundance table rather than raw or converted MRM chromatogram files

## Inputs

- MRM sample files (mzML or converted vendor raw data via ProteoWizard msconvert)
- Method file with targeted compound information (e.g., 295 compounds, RTs, transitions)
- Batch-pooled QC (BQC) samples with injection order (e.g., 389 BQCs across 33 batches)

## Outputs

- Quantitated compound abundances per sample (pre-correction)
- Batch-corrected abundances after QC-RLSC
- BQC regression slope and R2 values before and after correction (e.g., batch_correction_results.csv)
- Quantitation results saved as a Microsoft Excel xlsx file

## How to apply

Convert vendor raw data to mzML using msconvert in ProteoWizard, then load the MRM sample files and a method file describing the targeted compounds into MRMQuant (a binary MATLAB MAT-file is generated for loading). Run peak detection — MATLAB's built-in 'findpeaks' locates peak tips, with dynamic signal smoothing and automatic deconvolution of coeluted peaks — to quantitate the targeted compounds in every sample. If QC samples are present, run MRMQuant's QC-RLSC batch-effect correction, in which a LOESS curve is fitted to each compound's abundance in the QC samples and saturated peaks are automatically excluded from the regression for unreliable abundances. For a multi-batch cohort like the MRMkit data set (3793 samples, 33 batches, 295 compounds, 389 BQCs), fit a regression of BQC abundances against injection order before and after correction and record slope and R2. Success is judged by the post-correction BQC line flattening (slope ~0.0 vs 0.0004 pre-correction) and R2 dropping from 0.3 to 1.44 × 10–4. Results are exported as a Microsoft Excel xlsx file (or compiled into a CSV such as batch_correction_results.csv).

## Related tools

- **MRMQuant** (Primary tool: automated peak detection, quantitation, and QC-RLSC batch-effect correction of large MRM cohorts) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard** (msconvert converts vendor raw MRM data to the mzML/MAT-compatible formats MRMQuant loads)
- **MRMkit** (Source of the large-scale benchmark dataset (3793 samples, 33 batches) and comparison tool (~4 h processing vs ~5 h)) — https://github.com/MRMkit/MRMkit
- **LOESS** (Curve fitted to compound abundance in QC samples underlying the QC-RLSC correction)

## Evaluation signals

- Pre-correction BQC regression vs injection order reproduces slope 0.0004 with R2 = 0.3
- Post-correction BQC regression is near-flat: slope ~0.0 with R2 = 1.44 × 10–4
- Saturated peaks are absent from the correction regression (automatically excluded)
- Sample/batch/compound counts reconcile with the input (e.g., 3793 samples, 33 batches, 295 compounds, 389 BQCs)
- Final quantitation results are delivered as a complete xlsx file with minimal manual correction required

## Limitations

- Batch-effect correction requires QC samples (BQCs) included in the sample files; without them the correction step is skipped
- Processing time scales with cohort size (~5 h for the MRMkit data set, nominally longer than MRMKit's ~4 h though actual computation time is comparable)
- Manual inspection and correction of quantitated peaks may still be required; users can skip points in a regression plot by unchecking ion chromatograms that do not fit the curve
- MRMkit's GitHub repository is archived; development has moved to SLINGhub/MRMhub

## Evidence
