---
name: qc-based-signal-correction
description: Use when a metabolomics peak-intensity matrix has repeatedly injected
  pooled QCSs interspersed with subject samples and intensities drift with injection
  order or batch, and a batch label is available.
when_to_use_negative:
- There is only one batch of data or the batch label is unknown — the article shows
  QC-RLSC and QC-SVRC failed to increase QCS similarity on single-batch data while
  WaveICA 2.0 (injection-order-only) succeeded.
- No repeatedly injected pooled QCSs are present in the run — these regression correctors
  require QCSs to fit the drift curves/models.
- The input is already batch-corrected or you need within-batch drift removal without
  batch information, where the article's own comparisons favor WaveICA 2.0.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: QC-RLSC
  role: Loess-style smoothing curve fitted in QCSs across run order to correct intensity
    drift
- name: QC-SVRC
  role: SVM regression with radial basis kernel, gamma tuned by ten-fold cross-validation
    grid search, fitted on QCSs
- name: WaveICA_2.0
  role: Baseline method used to benchmark QC-RLSC/QC-SVRC performance and supply the
    packaged plasma dataset
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: R
  role: Implementation environment for the compared batch-effect removal methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:87fa2815e7be67c6f3627347165bb639b24fb7fcd63bb882a0586ad94560f18f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - QC-RLSC
  - QC-SVRC
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# qc-based-signal-correction

## Summary

Corrects intensity drift and run-order/batch effects in untargeted metabolomics peak tables by fitting regression models to repeated quality control samples (QCSs) and applying the fitted curves/models to subject samples. This card covers the loess-style QC-RLSC and SVM-based QC-SVRC variants benchmarked against WaveICA 2.0.

## When to use

Use when a metabolomics peak-intensity matrix has repeatedly injected pooled QCSs interspersed with subject samples and intensities drift with injection order or batch, and a batch label is available.

## When NOT to use

- There is only one batch of data or the batch label is unknown — the article shows QC-RLSC and QC-SVRC failed to increase QCS similarity on single-batch data while WaveICA 2.0 (injection-order-only) succeeded.
- No repeatedly injected pooled QCSs are present in the run — these regression correctors require QCSs to fit the drift curves/models.
- The input is already batch-corrected or you need within-batch drift removal without batch information, where the article's own comparisons favor WaveICA 2.0.

## Inputs

- Peak-intensity data matrix from XCMS/CAMERA preprocessing (retention time, m/z, peak intensity per sample)
- Injection order vector for all samples
- QCS labels identifying which rows are pooled quality control samples
- Batch labels (when multiple batches are present)

## Outputs

- Batch/run-order-corrected peak-intensity matrix for all samples
- Per-peak fitted correction curves or regression models (loess fit for QC-RLSC; RBF-kernel SVR models for QC-SVRC)
- Evaluation table of mean QCS pair Pearson correlation coefficients per method

## How to apply

Extract the QCS peak-intensity rows and the injection-order vector from the preprocessed peak matrix (e.g., XCMS/CAMERA output). For QC-RLSC, fit a loess-style smoothing curve per peak to the QCS intensities across run order and interpolate the correction to subject samples. For QC-SVRC, fit a support vector regression with a radial basis kernel per peak, selecting gamma by grid search over {2^-3,...,2^6} with ten-fold cross-validation on the QCSs. Apply the fitted correction to subject-sample intensities, then evaluate by computing Pearson correlation coefficients between all pairs of QCSs (higher is better, approaching 1) and by downstream PLS-DA biomarker discovery with FDR < 0.05 and VIP > 1. On the paper's single-batch (batch 1, 192 subject samples + 25 QCSs, 6,461 peaks) plasma data, QC-RLSC reached QCS PCC 0.941 (56 differential peaks) and QC-SVRC 0.934 (68 peaks) versus 0.973 (172 peaks) for WaveICA 2.0, so expect these regression-based correctors to underperform wavelet-ICA methods when batch labels are absent or only one batch exists.

## Related tools

- **QC-RLSC** (Loess-style smoothing curve fitted in QCSs across run order to correct intensity drift)
- **QC-SVRC** (SVM regression with radial basis kernel, gamma tuned by ten-fold cross-validation grid search, fitted on QCSs)
- **WaveICA_2.0** (Baseline method used to benchmark QC-RLSC/QC-SVRC performance and supply the packaged plasma dataset) — https://github.com/dengkuistat/WaveICA_2.0
- **R** (Implementation environment for the compared batch-effect removal methods)

## Examples

```
Rscript run_batch1_comparison.R --method QC-RLSC --data batch1_plasma.csv --qcs 25 --peaks 6461
```

## Evaluation signals

- Mean Pearson correlation coefficient between all pairs of QCSs should increase after correction (approaching 1); on batch 1 reference data QC-RLSC yields 0.941 and QC-SVRC 0.934 vs 0.916 for raw data.
- Downstream differential-peak count (PLS-DA, FDR < 0.05, VIP > 1) should be comparable or improved: 56 peaks (QC-RLSC) and 68 peaks (QC-SVRC) vs 172 for WaveICA 2.0 on batch 1.
- Predictive SVM (RBF, five-fold CV repeated 100 times) AUC on identified biomarkers should remain in the ~0.85–0.90 range (QC-RLSC 0.8585 ± 0.0201, QC-SVRC 0.8476 ± 0.0203).
- PCA score plots of corrected data should show QCSs and subject samples gathering more tightly; average intra-QCS distance should decrease relative to the original data.
- Cumulative RSD curves should show reduced variance attributable to run order after correction.

## Limitations

- On single-batch data, QC-RLSC and QC-SVRC failed to increase QCS similarity — only WaveICA 2.0 efficiently improved QCS PCCs — so these methods are not recommended when batch information is unavailable.
- Both methods require a batch label / run-order design with adequate repeated QCS coverage to fit the correction models.
- QC-SVRC hyperparameters (gamma grid 2^-3 to 2^6) must be tuned by cross-validation, adding computational cost and sensitivity to QCS count.

## Evidence

- [results] QC-RLSC and QC-SVRC failed to increase the similarity of quality control samples, while only WaveICA 2.0 efficiently improved the Pearson correlation coefficients between pairs of QCSs: [evidence span withheld — anchor status: not_found]
- [methods] QC-SVRC (SVM regression with radial basis kernel, grid search with ten-fold cross-validation over gamma in [2^-3, 2^-2, 2^-1, 1, 2^1, 2^2, 2^3, 2^4, 2^5, 2^6]): [evidence span withheld — anchor status: not_found]
- [methods] Peaks with an FDR value less than 0.05 and variable importance in the projection (VIP) of PLS-DA greater than 1 are considered as potential biomarkers.: [evidence span withheld — anchor status: not_found]
