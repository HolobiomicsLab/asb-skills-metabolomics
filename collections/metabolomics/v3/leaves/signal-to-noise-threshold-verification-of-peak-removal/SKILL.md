---
name: signal-to-noise-threshold-verification-of-peak-removal
description: Use when a baseline-corrected MALDI-MS spectrum has been peak-detected
  in subMALDI using an SNR_thresh plus segment size n, and you must confirm every
  sub-threshold peak is absent from the filtered output.
when_to_use_negative:
- The spectrum has not yet been baseline-corrected — the SNR comparison is defined
  against the baseline-corrected intensity values, so running it on raw spectra will
  misclassify noise as signal
- The goal is peak alignment to the theoretical m/z grid or normalization of replicates
  — these are separate downstream subMALDI workflow steps, not the SNR threshold verification
  step
- The input is already a peak table with no intensity trace — verification requires
  re-scanning the corrected spectrum, so a peaks-only object cannot be checked
edam_operation: http://edamontology.org/operation_3220
tools:
- name: subMALDI
  role: Provides the SNR-based peak detection (per-segment median absolute deviation
    noise, SNR_thresh cutoff) and the slope-based endpoint filtering used to discard
    noise-shaped candidates from the baseline-corrected spectrum
  repo: https://github.com/wesleyburr/subMaldi
- name: R
  role: Programming environment in which subMALDI's peak detection and filtering functions
    are executed on m/z/intensity data frames
- name: MassSpecWavelet
  role: BioConductor dependency that must be installed before subMALDI, whose peak-detection
    routines support the workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02694
    title: 'subMALDI: an open framework R package for processing irregularly-spaced
      mass spectrometry data'
  source_content_hash: sha256:ce5902d8779212403c1106aece6ed67957e3de91b6182802fc511d3a3618e9f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wesleyburr/subMaldi
  tools:
  - MassSpecWavelet
  - R
  - subMALDI
  edam_operation: http://edamontology.org/operation_3220
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# signal-to-noise-threshold-verification-of-peak-removal

## Summary

Verifies that subMALDI's SNR-based peak detection correctly discarded sub-threshold features: peaks whose intensity divided by per-segment noise (median absolute deviation) falls below SNR_thresh must be absent from the peak-filtered output, with slope-based endpoint filtering applied to remove remaining noise-shaped candidates.

## When to use

Use when a baseline-corrected MALDI-MS spectrum has been peak-detected in subMALDI using an SNR_thresh plus segment size n, and you must confirm every sub-threshold peak is absent from the filtered output.

## When NOT to use

- The spectrum has not yet been baseline-corrected — the SNR comparison is defined against the baseline-corrected intensity values, so running it on raw spectra will misclassify noise as signal
- The goal is peak alignment to the theoretical m/z grid or normalization of replicates — these are separate downstream subMALDI workflow steps, not the SNR threshold verification step
- The input is already a peak table with no intensity trace — verification requires re-scanning the corrected spectrum, so a peaks-only object cannot be checked

## Inputs

- Baseline-corrected spectrum from subMALDI's baseline correction step, as a data frame of m/z and intensity values
- User-specified SNR threshold (SNR_thresh) for peak detection
- User-specified segment size (n) used for per-segment noise estimation

## Outputs

- Peak-filtered spectrum (data frame of m/z and intensity values) in which all peaks below SNR_thresh have been removed
- Verification record confirming each sub-threshold peak in the baseline-corrected spectrum is absent from the filtered output

## How to apply

Load the baseline-corrected spectrum as a plain data frame of m/z and intensity values (subMALDI deliberately avoids S3/S4 masking so the data stay accessible). Run subMALDI's peak detection with the user-specified SNR_thresh and segment size n: the spectrum is divided into segments, noise is estimated as the median absolute deviation per segment, and any peak whose intensity divided by segment noise falls below SNR_thresh is discarded. Apply subMALDI's slope-based endpoint filtering to the flagged candidates to remove features lacking the characteristic shape of real peaks. Then compare the filtered output against the baseline-corrected input and confirm each sub-threshold peak found in the corrected data is absent from the peak-filtered result before saving it for downstream alignment and normalization.

## Related tools

- **subMALDI** (Provides the SNR-based peak detection (per-segment median absolute deviation noise, SNR_thresh cutoff) and the slope-based endpoint filtering used to discard noise-shaped candidates from the baseline-corrected spectrum) — https://github.com/wesleyburr/subMaldi
- **R** (Programming environment in which subMALDI's peak detection and filtering functions are executed on m/z/intensity data frames)
- **MassSpecWavelet** (BioConductor dependency that must be installed before subMALDI, whose peak-detection routines support the workflow)

## Evaluation signals

- Every peak in the baseline-corrected spectrum whose intensity divided by segment noise falls below SNR_thresh is confirmed absent from the peak-filtered output
- Peaks above SNR_thresh that also pass the slope-based endpoint shape test are retained in the output
- The filtered output remains a plain m/z/intensity data frame (no S3/S4 masking), keeping the data compatible with other R packages
- The verified filtered spectrum is saved and carries forward intact into the downstream alignment and normalization steps of the subMALDI workflow

## Limitations

- The article's abstract does not name the specific peak detection algorithm variants, so the exact function name for SNR detection must be confirmed from the package vignettes
- No software version numbers are stated for subMALDI, R, or its dependencies, so reproducibility of the filtering behavior depends on the release installed
- No DOI or data availability statement is given for the datasets used in the workflow example, so the verification cannot be reproduced from a fixed published dataset

## Evidence

- [abstract] one of several smoothing filters, baseline correction methods, and peak detection algorithms included in the package: [evidence span withheld — anchor status: external_doc]
- [abstract] without masking into S3 or S4 class objects, keeping m/z and intensity data accessible after each processing step: "without masking
into S3 or S4 class objects"
- [readme] subMALDI is an open framework package for the R programming environment that permits organization, pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data sets.: "subMALDI is an open framework package for the R programming environment that permits organization,  pre-processing (smoothing, baseline correction, peak detection), and normalization of spectral data"
- [readme] Refer to the vignette on workflow for a brief summary of the package functionality: "Refer to the [vignette on workflow](https://wesleyburr.github.io/subMaldi/articles/subMALDI_workflow.html)
for a brief summary of the package functionality"
