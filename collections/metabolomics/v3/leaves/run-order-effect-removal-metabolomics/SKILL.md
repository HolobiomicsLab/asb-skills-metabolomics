---
name: run-order-effect-removal-metabolomics
description: Use when a metabolomics peak-intensity matrix (samples x peaks) has only
  one batch or an unknown batch label but an injection order vector is available,
  and run order effects must be removed before biomarker discovery.
when_to_use_negative:
- The input is raw LC-MS data (.d or mzXML files) that has not yet been peak-picked
  with XCMS and annotated with CAMERA — preprocess first.
- A reliable batch label exists and the original WaveICA (F-test based, batch-label
  dependent) is preferred, e.g. multi-batch data where batch-aware correction is available.
- The data contain no QCSs and no injection order information, so neither run order
  drift detection nor QC-RLSC/QC-SVRC fitting is possible.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: WaveICA_2.0
  role: 'R package implementing the method: MODWT decomposition (Haar wavelet), ICA,
    GAM-based selection of batch-related independent components with adjusted R2 >
    0.1, and inverse MODWT reconstruction using injection order only'
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: R
  role: Execution environment; the authors state they only compare methods developed
    in R for metabolomics studies
- name: XCMS (Version 3.2)
  role: Upstream preprocessing of mzXML files to produce the retention time, m/z,
    and peak-intensity matrix the correction is applied to
- name: CAMERA
  role: Peak annotation after XCMS processing, before batch effect removal
- name: MetaboAnalystR
  role: Pipeline into which WaveICA has been incorporated; used for downstream PLS-DA
    biomarker identification with FDR and VIP thresholds
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:8098580e688228e718202f0c38f423a28f31bfb0f32f93d254af361bceb0e448
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - CAMERA
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  - XCMS (Version 3.2)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# run-order-effect-removal-metabolomics

## Summary

Removes run order (intra-batch injection order) intensity drift from untargeted metabolomics peak tables without batch labels, using WaveICA 2.0's MODWT decomposition, ICA, and GAM-based detection of batch-related independent components (adjusted R2 > 0.1), validated against QC-RLSC and QC-SVRC.

## When to use

Use when a metabolomics peak-intensity matrix (samples x peaks) has only one batch or an unknown batch label but an injection order vector is available, and run order effects must be removed before biomarker discovery.

## When NOT to use

- The input is raw LC-MS data (.d or mzXML files) that has not yet been peak-picked with XCMS and annotated with CAMERA — preprocess first.
- A reliable batch label exists and the original WaveICA (F-test based, batch-label dependent) is preferred, e.g. multi-batch data where batch-aware correction is available.
- The data contain no QCSs and no injection order information, so neither run order drift detection nor QC-RLSC/QC-SVRC fitting is possible.

## Inputs

- Peak-intensity matrix from XCMS/CAMERA preprocessing (e.g. 192 subject samples and 25 QCSs, 6,461 peaks for batch 1)
- Injection order vector for all samples
- Optional batch label (not required; WaveICA 2.0 uses injection order only)
- QCS sample annotations identifying quality control samples

## Outputs

- Batch/run-order corrected peak-intensity matrix
- Pairwise Pearson correlation coefficients (PCCs) between QCSs before and after correction (e.g. WaveICA 2.0 PCC 0.973)
- Differential peak counts per method via PLS-DA with FDR < 0.05 and VIP > 1 (e.g. 172 peaks for WaveICA 2.0 vs 56 for QC-RLSC, 68 for QC-SVRC)
- Comparison table (e.g. batch1_comparison_results.csv) of PCCs and differential peak counts per method

## How to apply

Take the XCMS/CAMERA-derived data matrix of retention time, m/z, and peak intensity (e.g. the plasma dataset with 6,461 peaks after discarding peaks detected in less than 80% of biological samples or QCSs) together with the injection order vector. Run WaveICA_2.0 in R using only injection order: apply MODWT decomposition (Haar wavelet) to split signals into frequencies, use ICA to extract independent components, fit a GAM of each independent component against injection order, and remove components with adjusted R2 greater than 0.1 as batch-effect related, then reconstruct via inverse MODWT. For comparison, run QC-RLSC (loess-style smoothing fitted in QCSs) and QC-SVRC (SVM regression with radial basis kernel, gamma grid-searched by ten-fold cross-validation over [2^-3, ..., 2^6]) on the same data. Evaluate correction quality by computing Pearson correlation coefficients between all pairs of QCSs in original vs corrected data, and downstream utility by PLS-DA biomarker counting with FDR < 0.05 and VIP > 1.

## Related tools

- **WaveICA_2.0** (R package implementing the method: MODWT decomposition (Haar wavelet), ICA, GAM-based selection of batch-related independent components with adjusted R2 > 0.1, and inverse MODWT reconstruction using injection order only) — https://github.com/dengkuistat/WaveICA_2.0
- **R** (Execution environment; the authors state they only compare methods developed in R for metabolomics studies)
- **XCMS (Version 3.2)** (Upstream preprocessing of mzXML files to produce the retention time, m/z, and peak-intensity matrix the correction is applied to)
- **CAMERA** (Peak annotation after XCMS processing, before batch effect removal)
- **MetaboAnalystR** (Pipeline into which WaveICA has been incorporated; used for downstream PLS-DA biomarker identification with FDR and VIP thresholds)

## Examples

```
library(devtools); devtools::install_github("dengkuistat/WaveICA_2.0", host="https://api.github.com"); library(WaveICA_2.0); data(data); corrected <- WaveICA2.0(data = data, wf = "haar", J = 2, cutoff = 0.1)
```

## Evaluation signals

- Pearson correlation coefficients between all pairs of QCSs should increase after correction and approach 1; on batch 1, WaveICA 2.0 reached PCC ~0.973 while QC-RLSC and QC-SVRC failed to increase QCS similarity (0.941 and 0.934).
- Differential peak counts between CRC and CE (PLS-DA, FDR < 0.05, VIP > 1) should increase relative to baseline; expected 172 peaks for WaveICA 2.0 vs 56 (QC-RLSC) and 68 (QC-SVRC).
- Average distance of QCSs in PCA score plots should shrink substantially after correction; compare before/after and against competing methods.
- Cumulative RSD curves should show reduced batch-effect variances after correction.
- SVM (radial basis) five-fold cross-validation AUC repeated 100 times should not degrade (e.g. 0.8831 ± 0.0170 for WaveICA 2.0 on batch 1).

## Limitations

- The method requires injection order information; without injection order, run order effects cannot be modeled.
- The batch-related component threshold (adjusted R2 > 0.1) is a study-specific choice; components below this threshold are retained and may leave residual drift.
- On multi-batch data, WaveICA (batch-label aware) gave slightly shorter average distance in QCSs than WaveICA 2.0, so batch-aware correction may be marginally better when labels are available.
- Peak filtering (discard peaks detected in < 80% of biological samples or QCSs) is applied before correction; peaks failing this rule cannot be rescued by the correction.
- The paper's dataset is unnamed with no accession statement, so exact reproduction depends on the packaged data in the WaveICA_2.0 R package.

## Evidence

- [methods] GAM-based detection of batch-related independent components: [evidence span withheld — anchor status: not_found]
