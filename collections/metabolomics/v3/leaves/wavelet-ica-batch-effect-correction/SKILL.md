---
name: wavelet-ica-batch-effect-correction
description: Use when an untargeted LC-MS metabolomics feature table shows batch effects
  or within-batch intensity drift, and batch labels are unavailable or there is only
  one batch — only injection order information is required.
when_to_use_negative:
- Input is raw MS data (.d, mzXML) or a retention-time/m/z list rather than an already-processed
  peak intensity matrix — preprocess with XCMS/CAMERA first
- You have reliable batch labels and multiple batches and want the original WaveICA
  (F-test, P < 0.05) approach instead — though WaveICA 2.0 gives similar performance
  on multi-batch data
- The data contain no QC samples and you need QC-based signal correction such as QC-RLSC/QC-SVRC
  fitted on QCSs
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: WaveICA_2.0
  role: R package implementing MODWT + ICA + GAM-based batch/run-order effect removal;
    also ships the plasma metabolomics dataset used in the paper
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: R
  role: Runtime environment; the method is developed and compared exclusively as R
    packages
- name: XCMS (Version 3.2)
  role: Upstream preprocessing of mzXML files to generate the peak intensity matrix
    WaveICA 2.0 corrects
- name: CAMERA
  role: Peak annotation after XCMS processing prior to batch effect removal
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:330ac927546e638b22dce76d5f062039a646431d85c36cda27f8012d393f91a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - CAMERA
  - R
  - WaveICA_2.0
  - XCMS (Version 3.2)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# wavelet-ica-batch-effect-correction

## Summary

WaveICA 2.0 removes inter-batch and run-order (intra-batch) effects from untargeted metabolomics peak tables using injection order instead of batch labels, via MODWT wavelet decomposition, ICA, and GAM-based selection of batch-related independent components.

## When to use

Use when an untargeted LC-MS metabolomics feature table shows batch effects or within-batch intensity drift, and batch labels are unavailable or there is only one batch — only injection order information is required.

## When NOT to use

- Input is raw MS data (.d, mzXML) or a retention-time/m/z list rather than an already-processed peak intensity matrix — preprocess with XCMS/CAMERA first
- You have reliable batch labels and multiple batches and want the original WaveICA (F-test, P < 0.05) approach instead — though WaveICA 2.0 gives similar performance on multi-batch data
- The data contain no QC samples and you need QC-based signal correction such as QC-RLSC/QC-SVRC fitted on QCSs

## Inputs

- Untargeted metabolomics peak intensity matrix (samples x peaks, e.g. 6,461 peaks) from XCMS/CAMERA preprocessing
- Sample injection order vector (used in place of batch labels)
- Optionally, quality control sample (QCS) annotations for validation

## Outputs

- Batch/run-order-corrected peak intensity matrix of the same dimensions
- Validation metrics: pairwise Pearson correlation coefficients between QCSs, cumulative RSD curves, PCA score plots
- Downstream biomarker lists (FDR < 0.05, PLS-DA VIP > 1) and SVM cross-validation AUCs

## How to apply

Input is a peak intensity matrix (e.g., 6,461 peaks from an LC-MS run preprocessed with XCMS and annotated with CAMERA) plus the sample injection order vector. Apply a MODWT (Haar wavelet) decomposition to split the data into frequency components, run ICA on the components, then fit a generalized additive model (GAM) of each independent component against injection order; remove components with adjusted R2 > 0.1 as batch-effect related, and reconstruct the corrected data by inverse MODWT. Corrected data can then feed downstream biomarker discovery (PLS-DA with FDR < 0.05 and VIP > 1) and validation by five-fold cross-validated SVM (RBF kernel, 100 repeats, AUC). Validate correction by comparing Pearson correlation coefficients between pairs of quality control samples (QCSs) before and after correction; successful correction drives QCS PCC toward 1 (≈0.973 in the paper's batch-1 test vs. 0.934–0.941 for QC-RLSC/QC-SVRC).

## Related tools

- **WaveICA_2.0** (R package implementing MODWT + ICA + GAM-based batch/run-order effect removal; also ships the plasma metabolomics dataset used in the paper) — https://github.com/dengkuistat/WaveICA_2.0
- **R** (Runtime environment; the method is developed and compared exclusively as R packages)
- **XCMS (Version 3.2)** (Upstream preprocessing of mzXML files to generate the peak intensity matrix WaveICA 2.0 corrects)
- **CAMERA** (Peak annotation after XCMS processing prior to batch effect removal)

## Examples

```
devtools::install_github("dengkuistat/WaveICA_2.0",host="https://api.github.com"); corrected <- WaveICA_2.0(data = peak_matrix, injection_order = order, wavelet = "haar")
```

## Evaluation signals

- Pearson correlation coefficients between all pairs of QCSs should approach 1 after correction (batch-1 benchmark: ≈0.973 for WaveICA 2.0 vs. 0.941 QC-RLSC / 0.934 QC-SVRC)
- PCA score plots should show QCSs and subject samples gathered together, and average pairwise distances in QCSs should shrink relative to original data
- Cumulative RSD curves should show reduced batch-effect variances relative to the original data
- Differential peak count under FDR < 0.05 and VIP > 1 should increase substantially (172 peaks vs. 56–68 for competing methods on batch 1)
- GAM fits of retained independent components against injection order should have adjusted R2 ≤ 0.1 (removed components exceeded this threshold)

## Limitations

- Average distance in QCSs can be slightly shorter for the original WaveICA than for WaveICA 2.0 on multi-batch data, i.e. the label-free variant may be marginally less tight on QC clustering
- Batch effects are inevitable in metabolomics studies even with careful experimental design, so correction quality depends on QCS coverage and injection-order spacing
- The method does not use batch labels, so performance relies on the GAM/injection-order association detection (adjusted R2 > 0.1) correctly identifying batch-related components

## Evidence

- [methods] GAM threshold for batch-related components: [evidence span withheld — anchor status: not_found]
