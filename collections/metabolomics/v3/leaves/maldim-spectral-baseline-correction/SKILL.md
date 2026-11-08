---
name: maldim-spectral-baseline-correction
description: Use when raw MALDI spectra (m/z + intensity data frames) show an irregular
  baseline and you need combined baseline correction plus peak detection without masking
  data into S3/S4 objects.
when_to_use_negative:
- Input is not raw spectral data but an already peak-picked or feature-aligned table
  — subMALDI's pre-processing (smoothing, baseline correction, peak detection) targets
  raw spectra
- The spectra are regularly spaced and already processed into another R framework's
  S3/S4 class objects that you do not want to unmask
- You only need normalization or replicate alignment to an m/z grid, not baseline
  correction or peak detection
tools:
- name: subMALDI
  role: Loads and holds raw MALDI spectra as plain data frames of m/z and intensity,
    keeping them accessible between processing steps; installed from source as subMALDI-1.0-2.tar.gz
  repo: https://github.com/wesleyburr/subMaldi
- name: MassSpecWavelet
  role: BioConductor dependency supplying cwt, getLocalMaximumCWT, getRidge, and identifyMajorPeaks
    for the CWT-based baseline correction and peak detection
- name: R
  role: Programming environment in which the subMALDI + MassSpecWavelet pipeline runs;
    installed alongside CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer,
    signal
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:60b12c459d02e93654a8efb3b9da5089335d182c8da7b839b4e4b4f158f742dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# maldim-spectral-baseline-correction

## Summary

Corrects the irregular baseline of raw MALDI mass spectra and performs peak detection in one step by combining subMALDI's plain data-frame spectral structure with the MassSpecWavelet continuous wavelet transform (CWT) pipeline, then filtering peaks by signal-to-noise ratio.

## When to use

Use when raw MALDI spectra (m/z + intensity data frames) show an irregular baseline and you need combined baseline correction plus peak detection without masking data into S3/S4 objects.

## When NOT to use

- Input is not raw spectral data but an already peak-picked or feature-aligned table — subMALDI's pre-processing (smoothing, baseline correction, peak detection) targets raw spectra
- The spectra are regularly spaced and already processed into another R framework's S3/S4 class objects that you do not want to unmask
- You only need normalization or replicate alignment to an m/z grid, not baseline correction or peak detection

## Inputs

- Raw MALDI mass spectrum as a data frame of m/z and intensity values (e.g. subMALDI's bundled bsline sample, read via readSpec)

## Outputs

- Baseline-corrected spectrum with an irregular baseline removed
- Filtered peak list (m/z and intensity values) written to CSV, e.g. peaks_bsline_cwt.csv, retaining only peaks with SNR above SNR.Th = 3

## How to apply

Load the bundled raw spectrum (e.g. the bsline sample) with subMALDI's readSpec into a data frame of m/z and intensity values. Apply the continuous wavelet transform cwt over a scales vector of 1–40, extract local maxima with getLocalMaximumCWT, and trace ridges with getRidge — all from BioConductor's MassSpecWavelet, which subMALDI's README lists as a required dependency. Call identifyMajorPeaks with SNR.Th = 3 to select major peaks from the CWT ridges, then apply the vignette's flag-and-remove filtering to drop noise-level peaks flagged by the SNR criterion. Rationale: subMALDI deliberately keeps m/z and intensity data accessible as plain data frames after each processing step, so external R package algorithms such as MassSpecWavelet's CWT operate directly on the vectors without any S3/S4 class conversion.

## Related tools

- **subMALDI** (Loads and holds raw MALDI spectra as plain data frames of m/z and intensity, keeping them accessible between processing steps; installed from source as subMALDI-1.0-2.tar.gz) — https://github.com/wesleyburr/subMaldi
- **MassSpecWavelet** (BioConductor dependency supplying cwt, getLocalMaximumCWT, getRidge, and identifyMajorPeaks for the CWT-based baseline correction and peak detection)
- **R** (Programming environment in which the subMALDI + MassSpecWavelet pipeline runs; installed alongside CRAN packages tidyr, ggplot2, ggpmisc, reshape2, RColorBrewer, signal)

## Examples

```
BiocManager::install("MassSpecWavelet"); coef <- cwt(raw_spec$intensity, scales = 1:40); peaks <- identifyMajorPeaks(raw_spec$intensity, coef, ridges, SNR.Th = 3)
```

## Evaluation signals

- The output CSV (e.g. peaks_bsline_cwt.csv) is a peak table whose row count is reduced relative to the pre-filter peak list
- Only peaks exceeding SNR.Th = 3 are retained, matching the vignette's flag-and-remove behaviour
- Plotted output shows the irregular baseline of the raw bsline spectrum was corrected before peak picking
- Data remains a plain data frame (m/z first column, intensities adjacent) at every step — no S3/S4 masking occurred
- MassSpecWavelet was installed via BiocManager before running, since it is a BioConductor (not CRAN) dependency

## Limitations

- The article states no specific baseline correction method names or parameter values beyond those in the vignette (scales 1–40, SNR.Th = 3)
- No software version numbers are stated for subMALDI, R, or dependencies, and no changelog/version history is provided
- The README warns the BioConductor install sequence does not always work; installing from source with dependencies = TRUE is suggested as the workaround
- subMALDI v1.0-2 must be installed from a downloaded source tarball (type = "source"), not from CRAN

## Evidence

- [abstract] permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets without masking into S3 or S4 class objects: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [intro] As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test.: "As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [readme] BiocManager::install("MassSpecWavelet"): "and then install the package MassSpecWavelet:

    BiocManager::install("MassSpecWavelet")"
- [readme] subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets.: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
