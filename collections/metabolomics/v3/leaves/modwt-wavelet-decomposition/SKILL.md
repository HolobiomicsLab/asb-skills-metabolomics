---
name: modwt-wavelet-decomposition
description: Use when an LC–MS peak intensity matrix shows batch or run-order effects
  (e.g. intensity drift over injection order) and batch labels are missing or there
  is only one batch, so effect removal must rely on injection order.
when_to_use_negative:
- The dataset lacks injection order metadata — WaveICA 2.0's GAM step uses injection
  order as its only covariate, so no batch-related components can be detected.
- The input is already a batch-corrected matrix or the goal is downstream biomarker
  identification (PLS-DA VIP > 1, FDR < 0.05) rather than batch-effect removal.
- Peak intensities have not been filtered (peaks detected in less than 80% of biological
  samples or QCSs should first be discarded).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: waveslim
  role: computes the MODWT decomposition (and inverse) of the peak intensity matrix
    via the Mallat pyramid algorithm with the Haar wavelet
- name: ica
  role: applies independent component analysis to each MODWT wavelet component to
    extract candidate batch-effect-related independent components
- name: mgcv
  role: fits the generalized additive model (thin plate regression spline) of each
    independent component on injection order, selecting components with adjusted R2
    > 0.1 for removal
- name: WaveICA_2.0
  role: R package implementing the full pipeline and providing the Amide_data example
    dataset and injection order vector
  repo: github.com/dengkuistat/WaveICA_2.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:d10ac3c4f82861e697307bfcb6a0ca835cd2f0579642a4b19dd1e6ae70239bc3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - WaveICA_2.0
  - ica
  - mgcv
  - waveslim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# modwt-wavelet-decomposition

## Summary

Decompose an untargeted metabolomics peak intensity matrix into frequency components using the maximal overlap discrete wavelet transform (MODWT) with the Haar wavelet, as the first stage of the WaveICA 2.0 batch-effect-removal pipeline (followed by ICA, GAM filtering, and inverse MODWT reconstruction).

## When to use

Use when an LC–MS peak intensity matrix shows batch or run-order effects (e.g. intensity drift over injection order) and batch labels are missing or there is only one batch, so effect removal must rely on injection order.

## When NOT to use

- The dataset lacks injection order metadata — WaveICA 2.0's GAM step uses injection order as its only covariate, so no batch-related components can be detected.
- The input is already a batch-corrected matrix or the goal is downstream biomarker identification (PLS-DA VIP > 1, FDR < 0.05) rather than batch-effect removal.
- Peak intensities have not been filtered (peaks detected in less than 80% of biological samples or QCSs should first be discarded).

## Inputs

- peak intensity data matrix from XCMS/CAMERA preprocessing (retention time, m/z, peak intensity)
- injection order vector for all samples
- example Amide_data metabolomics dataset shipped with the WaveICA_2.0 R package

## Outputs

- MODWT wavelet frequency components of the intensity matrix (per decomposition level)
- independent components per wavelet component with GAM adjusted R-squared values
- batch-effect-corrected intensity matrix (e.g. amide_data_waveica2_corrected.csv)

## How to apply

Load the peak intensity data matrix (e.g. the Amide_data example from the WaveICA_2.0 R package) together with its injection order vector. Compute the MODWT of the intensity matrix with the Haar wavelet via the Mallat pyramid algorithm implemented in the waveslim package, splitting the signal into frequency components that separate slow batch-related trends from fast biological variation. Apply independent component analysis (ica package) to each wavelet component, then fit a generalized additive model (mgcv, thin plate regression spline, y ~ injection order) with parameters chosen by generalized cross-validation; remove independent components whose adjusted R-squared exceeds the cutoff of 0.1 as batch-effect-related. Reconstruct the corrected data matrix with the inverse MODWT and save it (e.g. as amide_data_waveica2_corrected.csv).

## Related tools

- **waveslim** (computes the MODWT decomposition (and inverse) of the peak intensity matrix via the Mallat pyramid algorithm with the Haar wavelet)
- **ica** (applies independent component analysis to each MODWT wavelet component to extract candidate batch-effect-related independent components)
- **mgcv** (fits the generalized additive model (thin plate regression spline) of each independent component on injection order, selecting components with adjusted R2 > 0.1 for removal)
- **WaveICA_2.0** (R package implementing the full pipeline and providing the Amide_data example dataset and injection order vector) — github.com/dengkuistat/WaveICA_2.0

## Examples

```
devtools::install_github("dengkuistat/WaveICA_2.0", host="https://api.github.com")
```

## Evaluation signals

- Pearson correlation coefficients between any two quality-control samples (QCSs) in the corrected matrix approach 1, as reported in the paper.
- Corrected peak intensities no longer show intensity drift over injection order (e.g. peak M122T352, m/z 122.081, RT 352.052).
- Cumulative RSD curves show reduced batch-effect variance relative to the original data.
- Average distance of QCSs in PCA score plots decreases markedly after correction (WaveICA 2.0 performed similarly to WaveICA, with WaveICA slightly shorter in QCSs).
- Inverse MODWT reconstruction returns a matrix of the same dimensions as the input, and removed components are exactly those with GAM adjusted R-squared > 0.1.

## Limitations

- Requires injection order information as input; without it the GAM-based detection of batch-related components cannot run.
- Performance is close to but not identical to the original WaveICA: the average distance in QCSs was slightly shorter for WaveICA than WaveICA 2.0.
- The adjusted R-squared cutoff of 0.1 is a study-specific threshold for flagging batch-effect-related components.

## Evidence

- [other] MODWT with Haar wavelet via Mallat pyramid in waveslim: [evidence span withheld — anchor status: not_found]
- [methods] Adjusted R2 > 0.1 cutoff for batch-related components: [evidence span withheld — anchor status: not_found]
