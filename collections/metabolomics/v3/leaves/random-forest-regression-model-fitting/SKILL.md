---
name: random-forest-regression-model-fitting
description: Use when a large-scale LC-MS metabolomics or proteomics feature table
  contains repeatedly injected pooled QC samples interleaved with biological samples
  (e.g. QC after every 5–8 injections) and per-feature intensities drift with injection
  order or vary across batches.
when_to_use_negative:
- The dataset has no repeated pooled QC injections interleaved with real samples (use
  the QC-free shiftCor_dQC Combat route instead, as statTarget notes 'Combat is also
  provided for QC free datasets').
- Input is already corrected or aggregated per-batch and per-feature drift has already
  been modeled.
- A feature has more than 20 percent missing values or fails the 80-percent non-zero
  rule, so there is insufficient QC signal to fit a stable RF regression.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: random forest
  role: Ensemble learner (ntree = 500) fitted on QC-sample intensities vs injection
    order to predict per-feature correction factors for unwanted signal drift and
    batch variation.
  repo: https://github.com/13479776/statTarget
- name: statTarget
  role: 'R package implementing the QC-RFSC correction: shiftCor(samPeno, samFile,
    Frule = 0.8, MLmethod = ''QCRFSC'', QCspan = 0, imputeM = ''KNN'') wraps the RF
    fit, 80% filter, KNN imputation and correction-factor application.'
  repo: https://github.com/13479776/statTarget
- name: XCMS
  role: Upstream peak-abundance extraction producing the feature table whose drift
    QC-RFSC corrects.
- name: Skyline
  role: Upstream extraction of tryptic-peptide transition abundances in SRM proteomics
    data (Dataset 3) fed to QC-RFSC.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:2df826be9cbb87dac8d5b0dafdbf3f58907286ef3102c1bb63edf8278fca3a63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - Skyline
  - XCMS
  - random forest
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# random-forest-regression-model-fitting

## Summary

Fit an ensemble of 500 random forest regression trees on QC-sample intensities versus injection order to learn unwanted signal drift and batch variation, then use the fitted model to predict per-feature correction factors that are applied to every real sample in LC-MS or SRM proteomics data.

## When to use

Use when a large-scale LC-MS metabolomics or proteomics feature table contains repeatedly injected pooled QC samples interleaved with biological samples (e.g. QC after every 5–8 injections) and per-feature intensities drift with injection order or vary across batches.

## When NOT to use

- The dataset has no repeated pooled QC injections interleaved with real samples (use the QC-free shiftCor_dQC Combat route instead, as statTarget notes 'Combat is also provided for QC free datasets').
- Input is already corrected or aggregated per-batch and per-feature drift has already been modeled.
- A feature has more than 20 percent missing values or fails the 80-percent non-zero rule, so there is insufficient QC signal to fit a stable RF regression.

## Inputs

- Feature intensity table ordered by injection sequence (XCMS/MZmine2/Skyline export, e.g. statTarget MTBLS79 profile from package extdata) with biological samples and pooled QC injections
- Sample metadata (samPeno/samFile) marking injection order and which rows are QCs
- List of QC-sample intensities per feature for model fitting

## Outputs

- Corrected full data table shift_all_cor.csv (QC + biological samples)
- Corrected QC-only table shift_QC_cor.csv
- Corrected sample-only table shift_sample_cor.csv
- Per-feature correction factors F_ij applied to each injection-order position

## How to apply

For each feature j, build a training set from the pooled QC samples only, using QC intensity as the response and injection order k_ij as the predictor, and fit a random forest ensemble with ntree = 500 (R randomForest package, as implemented in statTarget's shiftCor with MLmethod = 'QCRFSC'). The fitted regression evaluates each sample's injection-order position to produce a correction factor F_ij; divide the raw intensity x_ij by F_ij to obtain the corrected value x'_ij for both QC and biological samples (eq 1–2 of the statTarget paper). Before fitting, apply the 80-percent rule (Frule = 0.8) to keep features with non-zero values in at least 80 percent of samples and remove features with more than 20 percent missing values, imputing residual NAs with KNN. The rationale is that pooled QCs replicate the same mixture repeatedly, so RF's ensemble averaging (shown to outperform Loess and SVR on R-square at median/first-quartile) captures nonlinear, feature-specific drift that linear regression cannot, while generalizing to neighboring real samples rather than only the QCs themselves.

## Related tools

- **random forest** (Ensemble learner (ntree = 500) fitted on QC-sample intensities vs injection order to predict per-feature correction factors for unwanted signal drift and batch variation.) — https://github.com/13479776/statTarget
- **statTarget** (R package implementing the QC-RFSC correction: shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = 'QCRFSC', QCspan = 0, imputeM = 'KNN') wraps the RF fit, 80% filter, KNN imputation and correction-factor application.) — https://github.com/13479776/statTarget
- **XCMS** (Upstream peak-abundance extraction producing the feature table whose drift QC-RFSC corrects.)
- **Skyline** (Upstream extraction of tryptic-peptide transition abundances in SRM proteomics data (Dataset 3) fed to QC-RFSC.)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = 'QCRFSC', QCspan = 0, imputeM = 'KNN')
```

## Evaluation signals

- Compare relative standard deviation (RSD) distribution of corrected QC peaks against raw data: proportion of peaks with RSD < 30 percent should rise markedly (43.9% → 86.2% in dataset 1; 52.1% → 90.9% in dataset 2).
- Peaks within 15 percent RSDs should increase roughly 12.7-fold (1.97% → 25.1%) after correction in dataset 1.
- RF prediction quality: R-square at median and first quartile should be high across datasets, consistent with 'RF, QRF and Loess outperformed with high predictive accuracy or R-square at median and first quartile value'.
- Corrected output files shift_all_cor.csv, shift_QC_cor.csv and shift_sample_cor.csv must exist and contain the same feature count as the input after the 80% / <20% missing filter.
- Corrected data must integrate QC and real samples across batches into one table without residual between-batch offsets.

## Limitations

- Requires interleaved pooled QC injections (every 5–8 samples in the cited datasets); with no QC design the QC-RFSC route is not applicable.
- Performance is tied to QC injection frequency and randomization of subject running order; unordered or non-randomized batches reduce correction fidelity.
- The method corrects intensity drift at the feature level but does not replace upstream feature extraction (XCMS/Skyline) or downstream statistical analysis (statAnalysis with glog and Pareto scaling).
- Mac users must install XQuartz and may need R 3.3.0 + RGtk2 2.20.31 for the statTarget GUI, as noted in the README.

## Evidence

- [methods] Correction factor applied via feature-level equations: [evidence span withheld — anchor status: not_found]
- [results] RSD improvement is the validation metric: [evidence span withheld — anchor status: not_found]
