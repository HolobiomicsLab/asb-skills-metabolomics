---
name: snr-threshold-peak-filtering
description: Use when a raw MALDI mass spectrum has been peak-detected (e.g. via CWT
  ridges from MassSpecWavelet) and the resulting peak list contains noise-level peaks
  that must be dropped before writing a final filtered peak table.
when_to_use_negative:
- The input is raw, unprocessed spectral intensity data rather than an already peak-detected
  list — filtering should follow peak detection, not replace it.
- The spectrum has been processed with subMALDI's own peak detection algorithms rather
  than MassSpecWavelet CWT ridges, where no SNR.Th flags are produced.
- The goal is peak alignment or normalization of replicate spectra to an m/z grid
  rather than noise-peak removal.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: subMALDI
  role: Loads and holds the raw spectrum as an accessible data frame (readSpec) so
    the SNR flag-and-remove filtering can operate on plain m/z/intensity columns
  repo: https://github.com/wesleyburr/subMaldi
- name: MassSpecWavelet
  role: Provides cwt, getLocalMaximumCWT, getRidge, and identifyMajorPeaks (with SNR.Th
    = 3) that generate the SNR flags used by the filtering step
- name: R
  role: Programming environment in which the whole CWT-then-filter workflow runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:e49891a1f10c34aeb36799c8ea9df9e6f1f38218ae60e1e8bb037cdff1a5c560
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# SNR-threshold peak filtering

## Summary

A fine-grained filtering step that removes noise-level peaks from a mass-spectrometry peak list by retaining only peaks whose signal-to-noise ratio exceeds a threshold (SNR.Th = 3 in the subMALDI/MassSpecWavelet vignette workflow), applied after CWT-based peak detection on raw MALDI spectra.

## When to use

Use when a raw MALDI mass spectrum has been peak-detected (e.g. via CWT ridges from MassSpecWavelet) and the resulting peak list contains noise-level peaks that must be dropped before writing a final filtered peak table.

## When NOT to use

- The input is raw, unprocessed spectral intensity data rather than an already peak-detected list — filtering should follow peak detection, not replace it.
- The spectrum has been processed with subMALDI's own peak detection algorithms rather than MassSpecWavelet CWT ridges, where no SNR.Th flags are produced.
- The goal is peak alignment or normalization of replicate spectra to an m/z grid rather than noise-peak removal.

## Inputs

- Peak list (m/z and intensity values) produced by MassSpecWavelet identifyMajorPeaks on a subMALDI-readSpec data frame of the bsline raw spectrum
- SNR values per peak from the CWT coefficient matrix (scales 1-40)

## Outputs

- Filtered peak table with m/z and intensity columns, written to peaks_bsline_cwt.csv, containing only peaks with SNR above the threshold (SNR.Th = 3)

## How to apply

After running the continuous wavelet transform (scales 1-40), getLocalMaximumCWT, getRidge, and identifyMajorPeaks with SNR.Th = 3 on a raw spectrum loaded via subMALDI's readSpec, apply the vignette's flag-and-remove filtering: peaks flagged by the SNR criterion (below SNR.Th = 3) are dropped from the peak list. This step is compatible with subMALDI's data-frame structure because the package does not mask data into S3/S4 objects, so m/z and intensity columns remain directly accessible for filtering. The rationale is to reduce the peak count to only meaningful signals above the noise floor, following the MassSpecWavelet major-peak identification behaviour bundled with subMALDI's dependencies.

## Related tools

- **subMALDI** (Loads and holds the raw spectrum as an accessible data frame (readSpec) so the SNR flag-and-remove filtering can operate on plain m/z/intensity columns) — https://github.com/wesleyburr/subMaldi
- **MassSpecWavelet** (Provides cwt, getLocalMaximumCWT, getRidge, and identifyMajorPeaks (with SNR.Th = 3) that generate the SNR flags used by the filtering step)
- **R** (Programming environment in which the whole CWT-then-filter workflow runs)

## Examples

```
R> spec <- subMaldi::readSpec("bsline"); coe <- MassSpecWavelet::cwt(spec$intensity, scales = 1:40); major <- MassSpecWavelet::identifyMajorPeaks(spec$mz, coe, ridges, SNR.Th = 3)
```

## Evaluation signals

- The output CSV (peaks_bsline_cwt.csv) contains a filtered peak table whose row count is reduced relative to the pre-filter peak list.
- Only peaks exceeding SNR.Th = 3 are retained in the output table.
- The filtering behaviour matches the vignette's flag-and-remove pattern on the bsline raw spectrum.
- The output preserves the subMALDI data-frame structure: m/z values in the first column with intensities adjacent.
- The irregular baseline was corrected and peak count reduced in a single processing step, as demonstrated in the vignette.

## Limitations

- subMALDI's README documents no specific peak-detection algorithm names or SNR thresholds of its own — the SNR.Th = 3 value comes from the MassSpecWavelet vignette workflow, so it may need retuning for other spectra.
- No software version numbers are stated for subMALDI, R, MassSpecWavelet, or any dependency, so reproducibility of the exact filtering behaviour depends on the 1.0-2 source install from the GitHub releases folder.
- BioConductor installation of MassSpecWavelet 'doesn't always seem to work' per the README; installing subMALDI with dependencies = TRUE from source is the recommended fallback.

## Evidence

- [intro] subMALDI data objects are designed to be compatible with functions from other R packages, and user-defined functions are easier to implement and test: "As a result of the package's design, subMALDI data objects are compatible with functions from a wide variety of other R packages, and user-defined functions are easier to implement and test."
- [abstract] subMALDI keeps m/z and intensity data accessible after each processing step: "permits organization, pre-processing (smoothing,
baseline correction, peak detection), and normalization of spectral data sets without masking
into S3 or S4 class objects"
- [readme] README instructs installing MassSpecWavelet dependency from BioConductor: "BiocManager::install("MassSpecWavelet")"
- [readme] README install of subMALDI from source: [evidence span withheld — anchor status: external_doc]
