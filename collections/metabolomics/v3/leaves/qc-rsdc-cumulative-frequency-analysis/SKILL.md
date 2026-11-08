---
name: qc-rsdc-cumulative-frequency-analysis
description: Use when assessing whether signal drift correction (e.g. QC-RFSC, QC-RLSC,
  QC-SVR) improved data precision on an MS-based metabolomics or proteomics feature
  table with repeated pooled-QC injections, by quantifying RSD% cumulative frequency
  shifts.
when_to_use_negative:
- The dataset has no repeated pooled-QC injections interleaved among study samples,
  so RSD% cannot be computed for QC samples.
- The goal is to evaluate a correction method on a single-sample-per-batch design
  without QC replicates, where RSD-based cumulative frequency analysis is not applicable.
- Input data are not MS-based feature tables (e.g. non-omics imaging or sequencing
  counts), as the RSD thresholds (15%, 30%) are calibrated for metabolomics/proteomics
  peak precision.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: R package providing shiftCor (QC-RFSC) to generate the corrected feature tables
    whose RSD distributions are compared, and statAnalysis for the associated PCA
  repo: https://github.com/13479776/statTarget
- name: random forest
  role: Core regression algorithm inside QC-RFSC (QCRFSC, ntree = 500) used to model
    and remove drift per feature; its effect is quantified by the cumulative RSD analysis
- name: Skyline
  role: Source of the proteomics peak-abundance table (174 transitions / 69 proteins)
    on which the cumulative frequency analysis is performed; converted to statTarget
    input via transX
- name: XCMS
  role: Source of metabolomics peak-abundance tables (datasets 1 and 2) for the cumulative
    RSD comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:7e5cacecb72a32ac4e2f230521285563a5ec7ca786f138081fef7b5a7f01c8f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - Skyline
  - XCMS
  - random forest
  - statTarget
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# QC-RFSC Cumulative Frequency Analysis

## Summary

Evaluates the effectiveness of QC-based random forest signal correction (QC-RFSC) in removing signal drift and inter-batch variation from metabolomics and proteomics feature tables by comparing cumulative relative standard deviation (RSD%) distributions of QC samples before and after correction, against QC-RLSC and QC-SVR baselines.

## When to use

Use when assessing whether signal drift correction (e.g. QC-RFSC, QC-RLSC, QC-SVR) improved data precision on an MS-based metabolomics or proteomics feature table with repeated pooled-QC injections, by quantifying RSD% cumulative frequency shifts.

## When NOT to use

- The dataset has no repeated pooled-QC injections interleaved among study samples, so RSD% cannot be computed for QC samples.
- The goal is to evaluate a correction method on a single-sample-per-batch design without QC replicates, where RSD-based cumulative frequency analysis is not applicable.
- Input data are not MS-based feature tables (e.g. non-omics imaging or sequencing counts), as the RSD thresholds (15%, 30%) are calibrated for metabolomics/proteomics peak precision.

## Inputs

- Raw MS feature table (e.g. XCMS or Skyline peak-abundance output) with pooled-QC samples interleaved across batches
- Corrected feature tables from QC-RFSC, QC-RLSC, and QC-SVR correction algorithms
- Sample annotation / injection-order metadata linking each run to batch and QC status

## Outputs

- Cumulative frequency distribution curves of feature RSD% for raw and corrected data
- Per-threshold summary statistics (e.g. proportion of features with RSD < 15%, RSD < 30%) enabling cross-method comparison of precision gain

## How to apply

Compute the relative standard deviation (RSD%) of each feature measured in pooled QC samples for the raw data matrix and for each corrected matrix (QC-RFSC, QC-RLSC, QC-SVR). Build cumulative frequency distribution curves over the RSD% range and report the proportion of features falling below key thresholds (e.g. 15% and 30% RSD). In the proteomics dataset (174 transitions / 69 proteins, SRM), QC-RFSC raised the share of peaks with RSD < 30% from 47.7% (raw) to 87.9%, versus 68.4% for QC-RLSC and 61.9% for QC-SVR; in metabolomics dataset 1 the share rose from 43.9% to 86.2%, and in dataset 2 from 52.1% to 90.9%. Apply the same statistic to corrected outputs produced by the statTarget shiftCor function (Frule = 0.8, MLmethod = 'QCRFSC', ntree = 500, imputeM = 'KNN') so that raw vs. corrected curves are directly comparable. Use the resulting cumulative-frequency curves and the reported threshold-crossing proportions as the primary precision metric for judging whether the correction removed unwanted variation.

## Related tools

- **statTarget** (R package providing shiftCor (QC-RFSC) to generate the corrected feature tables whose RSD distributions are compared, and statAnalysis for the associated PCA) — https://github.com/13479776/statTarget
- **random forest** (Core regression algorithm inside QC-RFSC (QCRFSC, ntree = 500) used to model and remove drift per feature; its effect is quantified by the cumulative RSD analysis)
- **Skyline** (Source of the proteomics peak-abundance table (174 transitions / 69 proteins) on which the cumulative frequency analysis is performed; converted to statTarget input via transX)
- **XCMS** (Source of metabolomics peak-abundance tables (datasets 1 and 2) for the cumulative RSD comparison)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Proportion of features with RSD < 30% should increase after correction relative to raw data (e.g. 43.9% → 86.2% in dataset 1; 52.1% → 90.9% in dataset 2; 47.7% → 87.9% in proteomics dataset 3).
- QC-RFSC should outperform both QC-RLSC and QC-SVR on the same dataset at the 30% RSD threshold (e.g. 87.9% vs 68.4% vs 61.9% in proteomics).
- Cumulative frequency curves for corrected data must lie above the raw-data curve across the RSD% range, indicating systematic precision gain rather than isolated improvements.
- In raw proteomics data the share of features with RSD < 15% was 1.97%, so any corrected result claiming a large gain should be checked against this baseline.
- Frule = 0.8 filtering (80% rule) must be applied before RSD computation so that low-quality features are not counted in the cumulative frequency statistics.

## Limitations

- The evaluation depends on the presence of repeated pooled-QC injections at regular intervals (every 5 samples in dataset 1, every 6 in dataset 2, every 8 in dataset 3); without them the RSD% baseline cannot be established.
- The 15% and 30% RSD thresholds are calibration points specific to this study's metabolomics and proteomics datasets and may not generalise to other omics modalities or acquisition platforms.
- Cumulative frequency analysis measures precision (repeatability) of QC features only; it does not by itself demonstrate improved statistical accuracy for between-group comparisons — PCA score plots are reported alongside for that purpose.
- Performance figures were obtained on real samples processed with specific settings (Frule = 0.8, KNN imputation, ntree = 500); different parameter choices may shift the reported proportions.
- The method assumes the correction algorithm itself (QC-RFSC vs QC-RLSC vs QC-SVR) is the only difference between compared curves; differences in upstream preprocessing (e.g. ProteoWizard mzXML conversion, XCMS vs Skyline parameterisation) could confound the comparison.

## Evidence

- [results] QC-RFSC increased the proportion of proteomics peaks with RSDs less than 30% to 87.9%, outperforming QC-RLSC (68.4%) and QC-SVR (47.7%): [evidence span withheld — anchor status: not_found]
- [results] the proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method.: [evidence span withheld — anchor status: not_found]
