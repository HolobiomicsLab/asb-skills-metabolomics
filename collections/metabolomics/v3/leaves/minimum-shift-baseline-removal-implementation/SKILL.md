---
name: minimum-shift-baseline-removal-implementation
description: Use when implementing or modifying the SAND adaptive binning stage on
  1D high-resolution metabolomics NMR spectra and each binned region (nominal 0.002
  ppm buckets) needs intensity offsets removed before raised-cosine banding and inverse
  Fourier processing.
when_to_use_negative:
- The input is already a feature/peak table produced by SAND decomposition — baseline
  handling at the spectrum level no longer applies
- The spectrum has not yet been adaptively partitioned and you need a global whole-spectrum
  baseline correction rather than per-binned-region offset removal
- The data are not high-resolution metabolomics NMR spectra — SAND's current version
  is tuned for that case and other NMR data types are not yet accommodated
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3792
tools:
- name: SAND
  role: Host pipeline whose adaptive binning stage embeds the minimum-shift baseline
    removal step; test examples and contribution workflow define validation
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation language for the binning/baseline module wrapped under src/
    per SAND's base-function interface conventions
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Environment in which updated code must run and pass the SAND test examples
    before merge
  repo: https://github.com/edisonomics/SAND
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:e5119c320f822a2e3c90fd2703ddf0ba8be8f80481f67abafd61febfbdd11c43
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3792
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# minimum-shift baseline removal implementation

## Summary

Implements the minimum-shift baseline-correction step inside SAND's adaptive frequency-binning preprocessing: within each binned region of a 1D metabolomics NMR spectrum, intensity offsets are corrected before raised-cosine apodization and Hilbert/inverse-FT conversion to per-bin FIDs. It is a prerequisite step because bin-to-FID modeling assumes each bin's intensities are referenced to a corrected baseline.

## When to use

Use when implementing or modifying the SAND adaptive binning stage on 1D high-resolution metabolomics NMR spectra and each binned region (nominal 0.002 ppm buckets) needs intensity offsets removed before raised-cosine banding and inverse Fourier processing.

## When NOT to use

- The input is already a feature/peak table produced by SAND decomposition — baseline handling at the spectrum level no longer applies
- The spectrum has not yet been adaptively partitioned and you need a global whole-spectrum baseline correction rather than per-binned-region offset removal
- The data are not high-resolution metabolomics NMR spectra — SAND's current version is tuned for that case and other NMR data types are not yet accommodated

## Inputs

- 1D frequency-domain NMR spectrum (ppm axis + intensity vector) loaded from SAND repository test examples
- Binned region definitions from adaptive partitioning (0.002 ppm nominal buckets, slackness 0.99)

## Outputs

- Baseline-corrected intensity arrays within each binned region (offsets removed)
- Offset-corrected bin regions ready for raised-cosine banding (0.02 ppm roll-off) and subsequent Hilbert/inverse-FT conversion to per-bin FIDs

## How to apply

Within the adaptive binning pipeline (partition the ppm axis into buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where met), apply minimum-shift baseline removal to the intensities inside each binned region so the region's minimum intensity is shifted to the corrected offset. This step must run on the binned region before the raised-cosine band with 0.02 ppm roll-off is applied at the bin edges, since apodization assumes offset-corrected intensities. Order matters: baseline removal -> raised-cosine banding -> Hilbert transform to form the analytic signal -> inverse Fourier transform to produce a complex per-bin time-domain FID. Following SAND's contribution conventions, wrap the implementation as a callable MATLAB module under src/ (e.g. src/adaptive_binning.m) and run it against the SAND repository test examples; any update is expected to run in the NMRBox environment and pass the tests before merge. No standalone quantitative accuracy metric is reported for the baseline stage in isolation, so correctness is judged by downstream pipeline behavior on the test examples.

## Related tools

- **SAND** (Host pipeline whose adaptive binning stage embeds the minimum-shift baseline removal step; test examples and contribution workflow define validation) — https://github.com/edisonomics/SAND
- **MATLAB** (Implementation language for the binning/baseline module wrapped under src/ per SAND's base-function interface conventions) — https://github.com/edisonomics/SAND
- **NMRbox** (Environment in which updated code must run and pass the SAND test examples before merge) — https://github.com/edisonomics/SAND

## Evaluation signals

- Each binned region's minimum intensity is shifted to the corrected baseline offset after the step (offset removal invariant)
- Downstream raised-cosine banding (0.02 ppm roll-off) and Hilbert/inverse-FT stages still yield a complex per-bin FID for every bin, indicating intensities were valid for windowing
- The SAND repository test examples pass when the module is run in the NMRBox test environment (the paper's merge requirement is that updates 'pass the test before merge')
- The ppm axis and bin partitioning (0.002 ppm nominal, slackness 0.99, three-bin merges) are unchanged by the correction — only intensity offsets are modified

## Limitations

- No quantitative metric is reported for this stage in isolation; validation relies on the SAND test examples passing rather than a standalone accuracy number
- Correction is applied per binned region, so it addresses local offsets within bins and is not designed to fix broad baselines spanning the entire spectrum
- SAND (including this preprocessing) is currently implemented in slow MATLAB and is not yet parallelizable; a C/C++ rewrite is planned
- The current version of SAND is tuned for high-resolution metabolomics spectra; other cases are not yet accommodated

## Evidence

- [full_text] Apply minimum-shift baseline removal within each binned region to correct intensity offsets.: [evidence span withheld — anchor status: not_found]
- [full_text] Apply a Hilbert transform to form the analytic signal and an inverse Fourier transform to convert each binned region into a complex per-bin time-domain FID.: [evidence span withheld — anchor status: not_found]
