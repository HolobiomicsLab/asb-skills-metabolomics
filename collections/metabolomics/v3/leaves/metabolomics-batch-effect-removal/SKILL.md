---
name: metabolomics-batch-effect-removal
description: Use when you have a preprocessed LC-MS peak intensity matrix (e.g., XCMS/CAMERA
  output) with a per-sample injection order vector, but batch labels are missing or
  there is only one batch, and batch/drift effects must be removed.
when_to_use_negative:
- Input is raw LC-MS data (.d, mzXML) rather than an XCMS/CAMERA-preprocessed peak
  intensity matrix — peak detection, 80%-detection filtering, and annotation must
  happen first
- Injection order metadata is unavailable, since WaveICA 2.0 uses only injection order
  information for batch effect removal
- The data already consists of only QC samples or has no injection-order-dependent
  structure to model
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WaveICA_2.0
  role: R package implementing the full method (MODWT -> ICA -> GAM -> inverse MODWT);
    ships the Amide_data example dataset and injection order
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: waveslim
  role: computes the MODWT decomposition with the Haar wavelet via the Mallat pyramid
    algorithm, and the inverse MODWT reconstruction
- name: ica
  role: independent component analysis on each wavelet component to extract candidate
    batch-effect-related independent components
- name: mgcv
  role: fits the thin plate regression spline GAM of each independent component on
    injection order with GCV-selected parameters, yielding adjusted R-squared for
    the 0.1 cutoff
- name: MetaboAnalystR
  role: downstream pipeline in which WaveICA is incorporated; used for PLS-DA biomarker
    identification and evaluation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:6dc2c1bc977eb10c65d8975563796f7b31fc1aa5b3af9c7bec8192a3c83402b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  - ica
  - mgcv
  - waveslim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolomics-batch-effect-removal

## Summary

Removes inter-batch and intra-batch (run order) effects from untargeted metabolomics peak intensity matrices without batch labels, using WaveICA 2.0: MODWT decomposition, ICA, GAM-based screening against injection order, and inverse MODWT reconstruction.

## When to use

Use when you have a preprocessed LC-MS peak intensity matrix (e.g., XCMS/CAMERA output) with a per-sample injection order vector, but batch labels are missing or there is only one batch, and batch/drift effects must be removed.

## When NOT to use

- Input is raw LC-MS data (.d, mzXML) rather than an XCMS/CAMERA-preprocessed peak intensity matrix — peak detection, 80%-detection filtering, and annotation must happen first
- Injection order metadata is unavailable, since WaveICA 2.0 uses only injection order information for batch effect removal
- The data already consists of only QC samples or has no injection-order-dependent structure to model

## Inputs

- Preprocessed metabolomics peak intensity matrix (samples x peaks, e.g., XCMS/CAMERA output from mzXML-converted LC-MS data)
- Per-sample injection order vector
- Optional: batch label vector and QCS flags for benchmarking against label-dependent methods

## Outputs

- Batch-effect-corrected peak intensity matrix (same dimensions as input)
- Corrected data exported as CSV (e.g., amide_data_waveica2_corrected.csv)
- Diagnostics: average distance of QCSs and subject samples in PCA score plots, average PCCs between QCS pairs, cumulative RSD curves

## How to apply

Take the metabolite peak intensity matrix (e.g., 642 samples x 6,461 peaks) plus the injection order vector as the only covariate — no batch label is used. Decompose the matrix into frequency components with the maximal overlap discrete wavelet transform (MODWT) using the Haar wavelet (waveslim, Mallat pyramid algorithm). Apply independent component analysis (ica package) to each wavelet component to extract candidate batch-effect-related independent components, then fit a generalized additive model (mgcv) with a thin plate regression spline of each independent component on injection order, with smoothing parameters chosen by generalized cross-validation. Remove components whose GAM fit yields adjusted R-squared > 0.1 (the paper's cutoff, replacing WaveICA's F-test P < 0.05 on batch labels), and reconstruct the corrected data matrix via inverse MODWT. Compare against baseline methods such as WaveICA (batch-label-dependent) using QCS average distance, subject-sample average distance, average Pearson correlation coefficients (PCCs) between QCS pairs, biomarker counts (FDR < 0.05, VIP > 1 from PLS-DA), and SVM AUC.

## Related tools

- **WaveICA_2.0** (R package implementing the full method (MODWT -> ICA -> GAM -> inverse MODWT); ships the Amide_data example dataset and injection order) — https://github.com/dengkuistat/WaveICA_2.0
- **waveslim** (computes the MODWT decomposition with the Haar wavelet via the Mallat pyramid algorithm, and the inverse MODWT reconstruction)
- **ica** (independent component analysis on each wavelet component to extract candidate batch-effect-related independent components)
- **mgcv** (fits the thin plate regression spline GAM of each independent component on injection order with GCV-selected parameters, yielding adjusted R-squared for the 0.1 cutoff)
- **MetaboAnalystR** (downstream pipeline in which WaveICA is incorporated; used for PLS-DA biomarker identification and evaluation)

## Examples

```
devtools::install_github("dengkuistat/WaveICA_2.0",host="https://api.github.com")
```

## Evaluation signals

- Pearson correlation coefficients between any two QCSs approach 1 in the corrected matrix (the paper reports aPCC 0.973 for WaveICA 2.0 vs 0.941 QC-RLSC and 0.934 QC-SVRC)
- Corrected peak intensities no longer show drift over injection order; batch structure and within-batch intensity drift (e.g., peak M122T352) are visibly removed
- Average distance of QCSs and of subject samples in PCA score plots is greatly reduced versus original data (supplementary row: 'WaveICA 2.0 9.154 34.261 0.973 172 0.8831 ± 0.0170')
- Corrected matrix has the same dimensions as the input peak matrix, and the reconstructed data are obtainable via inverse MODWT without dropping samples
- Downstream benchmark (Table 1 pattern): biomarker count (FDR < 0.05, VIP > 1) and five-fold CV SVM AUC (repeated 100 times) for CRC vs CE remain comparable to the batch-label-dependent WaveICA baseline (e.g., 172 biomarkers, AUC 0.8831 ± 0.0170)

## Limitations

- The original WaveICA requires a batch label, so batch-label-dependent comparisons are impossible when there is only one batch of data or the batch label is unknown — this is exactly the gap WaveICA 2.0 fills, but direct label-aware benchmarking may not be feasible on such data
- The adjusted R-squared cutoff of 0.1 for declaring independent components batch-effect-related is a study-specific choice; no explicit sensitivity analysis or limitation discussion is provided in the paper's discussion section
- The dataset used is unnamed and has no accession or availability statement beyond the data bundled in the WaveICA_2.0 package

## Evidence

- [readme] WaveICA 2.0 method is an improved version of WaveICA method to remove inter-batch and intra-batch effects for untargeted metabolomics data without using batch information.: "WaveICA 2.0 method is an improved version of WaveICA method to remove inter-batch and intra-batch effects for untargeted metabolomics data without using batch information."
- [methods] GAM detects batch-related independent components: "the generalized additive model (GAM) is used to examine the nonlinear associations between the independent component"
- [methods] adjusted R2 cutoff: [evidence span withheld — anchor status: not_found]
- [intro] motivating gap: "the WaveICA method needs a batch label, which cannot be used when there is only one batch of data or the batch label is unknown."
- [results] QCS similarity outcome: "the PCCs between any two QCSs approached 1 in the data processed by WaveICA 2.0 and WaveICA"
