---
name: gcxgc-2dtic-preprocessing
description: Use when raw GCxGC-MS chromatograms (NetCDF files) contain baseline drift,
  noise, or retention-time shifts that must be reduced before multivariate analysis
  such as MPCA of 2D-TIC data.
when_to_use_negative:
- The data are already unfolded feature tables or peak lists rather than 2D chromatograms
  — alignment and smoothing operate on two-dimensional Total Intensity Chromatograms
- The analysis goal is supervised modelling from scratch without prior signal enhancement
  — preprocessing (smoothing, baseline correction, peak alignment) is a prerequisite
  step, not the modelling itself
- The input is not two-dimensional gas chromatography data (the package is built for
  2D-TIC from GCxGC, not 1D chromatograms or LC-MS data)
tools:
- name: RGCxGC
  role: Provides read_chrom, baseline_corr, twod_cow/batch_2DCOW, m_prcomp, and unfold_chrom
    for the full preprocessing and multivariate-analysis workflow
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: ptw
  role: Supplies the base functions with1 and with2 that are coupled inside the 2D-COW
    peak-alignment implementation
- name: R
  role: Execution environment for the RGCxGC package and its vignette workflow
- name: RNetCDF
  role: Supports importing the raw chromatogram from a NetCDF file before folding
    into the 2D-TIC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:90f0084d6999c813e322d62aee0876dffa4201bbb981272697532bf49c3fa540
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  repo_url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - R
  - RGCxGC
  - RNetCDF
  - ptw
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# gcxgc-2dtic-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The RGCxGC end-to-end preprocessing pipeline for comprehensive two-dimensional gas chromatography-mass spectrometry (GCxGC-MS): import a NetCDF raw chromatogram, fold it into a 2D-TIC, then apply asymmetric least-squares baseline correction, Whittaker smoothing, and 2D-COW peak alignment before multivariate analysis.

## When to use

Use when raw GCxGC-MS chromatograms (NetCDF files) contain baseline drift, noise, or retention-time shifts that must be reduced before multivariate analysis such as MPCA of 2D-TIC data.

## When NOT to use

- The data are already unfolded feature tables or peak lists rather than 2D chromatograms — alignment and smoothing operate on two-dimensional Total Intensity Chromatograms
- The analysis goal is supervised modelling from scratch without prior signal enhancement — preprocessing (smoothing, baseline correction, peak alignment) is a prerequisite step, not the modelling itself
- The input is not two-dimensional gas chromatography data (the package is built for 2D-TIC from GCxGC, not 1D chromatograms or LC-MS data)

## Inputs

- Raw GCxGC-MS chromatogram in NetCDF format
- 2D-TIC chromatogram objects (e.g. MTBLS08 sample and MTBLS09 reference from the MTBLS579 MetaboLights dataset)
- Preprocessed reference chromatogram (baseline-corrected and smoothed) for 2D-COW alignment

## Outputs

- Two-dimensional Total Intensity Chromatogram (2D-TIC) object
- Baseline-corrected and smoothed chromatogram objects
- Aligned chromatogram object (e.g. saved as aligned_08GB.rds via saveRDS)
- Unfolded chromatogram matrices for downstream supervised models (mixOmics) via unfold_chrom

## How to apply

Import the raw chromatogram from a NetCDF file with read_chrom (setting modulation time, sampling rate, and x/y cut points) and fold it into the two-dimensional Total Intensity Chromatogram (2D-TIC). Apply baseline correction with baseline_corr using the asymmetric least squares algorithm, then reduce noise with quadratic-penalty (Whittaker) smoothing. For peak alignment, call twod_cow (or batch_2DCOW for multiple samples) with a preprocessed reference chromatogram — e.g. aligning the smoothed 08GB sample to the 09GB reference with segments = c(10, 40) and max_warp = c(1, 8); the 2D-COW implementation couples the with1 and with2 base functions from the ptw package. Optionally add the reference back into the batch (add_ref), save the aligned chromatogram with saveRDS (e.g. aligned_08GB.rds), and proceed to multivariate analysis with m_prcomp to reveal differences between sample groups.

## Related tools

- **RGCxGC** (Provides read_chrom, baseline_corr, twod_cow/batch_2DCOW, m_prcomp, and unfold_chrom for the full preprocessing and multivariate-analysis workflow) — https://github.com/DanielQuiroz97/RGCxGC
- **ptw** (Supplies the base functions with1 and with2 that are coupled inside the 2D-COW peak-alignment implementation)
- **R** (Execution environment for the RGCxGC package and its vignette workflow)
- **RNetCDF** (Supports importing the raw chromatogram from a NetCDF file before folding into the 2D-TIC)

## Examples

```
aligned_08GB <- twod_cow(reference = MTBL09_sm2, sample_chroms = MTBLS08_sm2, segments = c(10, 40), max_warp = c(1, 8)); saveRDS(aligned_08GB, "aligned_08GB.rds")
```

## Evaluation signals

- The output file aligned_08GB.rds exists and, when loaded, contains a two-dimensional chromatogram object of the same dimensionality as the input sample chromatogram
- Baseline correction is performed on top of the asymmetric least squares algorithm before smoothing and alignment
- 2D-COW warping parameters match the vignette's values: segments = c(10, 40) and max_warp = c(1, 8), with the 09GB chromatogram as reference and 08GB as sample
- Preprocessed chromatograms can be unfolded via unfold_chrom and fed to MPCA (m_prcomp) without dimension errors
- Scores and loadings can be extracted from the m_prcomp result via the scores and loading functions, confirming the chromatogram matrices retain consistent 2D structure

## Limitations

- The asymmetric least squares baseline correction and Whittaker smoothing links point to specific parametrizations (Eilers 2003/2004); results depend on choosing suitable penalty settings
- 2D-COW alignment requires a preprocessed reference chromatogram (the vignette aligns 08GB against the equivalently preprocessed 09GB), so sample and reference must undergo identical preprocessing
- No changelog is found in the repository, so version-specific behavior changes are not documented

## Evidence

- [methods] 2D-COW parameters from the vignette peak-alignment workflow: [evidence span withheld — anchor status: not_found]
