---
name: lc-ms-intensity-matrix-reconstruction
description: Use when an XCMS-derived peak intensity matrix (retention time, m/z,
  peak intensity) shows batch or injection-order drift (visible intensity drift over
  run order, dispersed QCSs) and the corrected intensity matrix must be rebuilt without
  batch labels.
when_to_use_negative:
- The task still requires per-peak annotation or feature detection — this skill operates
  on an already-preprocessed XCMS intensity matrix, not raw mzXML/.d files.
- The analysis goal is biomarker discovery (PLS-DA, FDR < 0.05, VIP > 1) rather than
  batch-effect correction of the intensity matrix.
- A batch label is available and the original WaveICA (F-test, P < 0.05 threshold)
  is preferred, since it achieved a slightly shorter average distance among QCSs in
  PCA.
tools:
- name: WaveICA_2.0
  role: R package that implements MODWT decomposition, ICA-based component screening,
    GAM testing, and inverse MODWT reconstruction; also ships the Amide_data example
    dataset
  repo: github.com/dengkuistat/WaveICA_2.0
- name: waveslim
  role: Computes the MODWT decomposition (Haar wavelet, Mallat pyramid algorithm)
    and its inverse for reconstruction
- name: ica
  role: Performs independent component analysis on each wavelet component to extract
    candidate batch-effect-related components
- name: mgcv
  role: Fits generalized additive models with thin plate regression splines to test
    each independent component against injection order (GCV-selected parameters, adjusted
    R2 cutoff 0.1)
- name: XCMS
  role: Upstream preprocessing that generates the input data matrix of retention time,
    m/z values, and peak intensity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:875606206979c38f48fe27affbd620c3c9d5396e210b8497d58f1dc1c1d57978
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - WaveICA_2.0
  - XCMS
  - ica
  - mgcv
  - waveslim
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# lc-ms-intensity-matrix-reconstruction

## Summary

Reconstructs a batch-effect-corrected LC-MS peak intensity matrix by inverting a MODWT (maximal overlap discrete wavelet transform) decomposition after removing batch-related independent components, as implemented in the WaveICA_2.0 R package for untargeted metabolomics data.

## When to use

Use when an XCMS-derived peak intensity matrix (retention time, m/z, peak intensity) shows batch or injection-order drift (visible intensity drift over run order, dispersed QCSs) and the corrected intensity matrix must be rebuilt without batch labels.

## When NOT to use

- The task still requires per-peak annotation or feature detection — this skill operates on an already-preprocessed XCMS intensity matrix, not raw mzXML/.d files.
- The analysis goal is biomarker discovery (PLS-DA, FDR < 0.05, VIP > 1) rather than batch-effect correction of the intensity matrix.
- A batch label is available and the original WaveICA (F-test, P < 0.05 threshold) is preferred, since it achieved a slightly shorter average distance among QCSs in PCA.

## Inputs

- LC-MS peak intensity data matrix from XCMS preprocessing (rows: samples/QCSs, columns: metabolite peaks with retention time and m/z values)
- Injection order vector for all samples
- Quality-control sample (QCS) labels for validation

## Outputs

- Batch-effect-corrected intensity data matrix (e.g., amide_data_waveica2_corrected.csv)
- Set of removed batch-effect-related independent components (adjusted R2 > 0.1)

## How to apply

Load the peak intensity matrix (e.g., the Amide_data example from the WaveICA_2.0 package) and its injection order vector. Decompose the matrix into frequency components using the MODWT with the Haar wavelet computed via the Mallat pyramid algorithm in the waveslim package. Apply ICA (ica package) to each wavelet component, then fit a generalized additive model (mgcv package) with a thin plate regression spline modeling each independent component on injection order with parameters selected by generalized cross-validation; remove components whose adjusted R-squared exceeds the 0.1 cutoff as batch-effect-related. Finally reconstruct the corrected data matrix with the inverse MODWT and save it (e.g., as amide_data_waveica2_corrected.csv).

## Related tools

- **WaveICA_2.0** (R package that implements MODWT decomposition, ICA-based component screening, GAM testing, and inverse MODWT reconstruction; also ships the Amide_data example dataset) — github.com/dengkuistat/WaveICA_2.0
- **waveslim** (Computes the MODWT decomposition (Haar wavelet, Mallat pyramid algorithm) and its inverse for reconstruction)
- **ica** (Performs independent component analysis on each wavelet component to extract candidate batch-effect-related components)
- **mgcv** (Fits generalized additive models with thin plate regression splines to test each independent component against injection order (GCV-selected parameters, adjusted R2 cutoff 0.1))
- **XCMS** (Upstream preprocessing that generates the input data matrix of retention time, m/z values, and peak intensity)

## Examples

```
devtools::install_github("dengkuistat/WaveICA_2.0", host="https://api.github.com")
```

## Evaluation signals

- Pearson correlation coefficients between any two QCSs in the corrected matrix approach 1 (as reported in the paper for both WaveICA 2.0 and WaveICA).
- Corrected intensities no longer show intensity drift over injection order (no within-batch or run-order trends).
- QCSs and subject samples gather together in PCA score plots; average distance among QCSs is greatly reduced relative to the original data.
- Cumulative RSD curves show reduced batch-effect variances compared with the original data.
- Corrected matrix has the same dimensions as the input peak intensity matrix (e.g., written to amide_data_waveica2_corrected.csv).

## Limitations

- The original WaveICA requires a batch label and cannot be applied when there is only one batch of data or the batch label is unknown; WaveICA 2.0 was developed to cover that gap.
- Average distance in QCSs was slightly shorter for WaveICA than WaveICA 2.0, so reconstruction quality may be marginally lower than batch-label-based alternatives.
- The paper provides no explicit limitations section, no software version details for WaveICA 2.0 in the discussion, and the metabolomics dataset used is unnamed with no accession or availability statement.

## Evidence

- [readme] WaveICA 2.0 method is an improved version of WaveICA method to remove inter-batch and intra-batch effects for untargeted metabolomics data without using batch information.: "WaveICA 2.0 method is an improved version of WaveICA method to remove inter-batch and intra-batch effects for untargeted metabolomics data without using batch information."
- [methods] MODWT decomposition and GAM-based component removal: [evidence span withheld — anchor status: not_found]
- [methods] adjusted R2 cutoff for batch-related components: [evidence span withheld — anchor status: not_found]
- [methods] GAM models independent components on injection order: "the generalized additive model (GAM) is used to examine the nonlinear associations between the independent component"
- [results] QCS similarity validation signal: "the PCCs between any two QCSs approached 1 in the data processed by WaveICA 2.0 and WaveICA"
- [results] WaveICA 2.0 uses only injection order, not batch labels: "the WaveICA 2.0 method does not use batch information and only use injection order information for batch effects removal"
