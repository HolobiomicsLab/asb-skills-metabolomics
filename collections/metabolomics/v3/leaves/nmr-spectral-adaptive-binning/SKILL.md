---
name: nmr-spectral-adaptive-binning
description: Use when a 1D frequency-domain (ppm-axis) metabolomics NMR spectrum must
  be partitioned into frequency bins and converted into per-bin time-domain FIDs for
  time-domain deconvolution or peak-table generation.
when_to_use_negative:
- The input is already a peak table or feature table — SAND's adaptive binning operates
  on raw 1D frequency-domain spectra before time-domain decomposition
- The data are not high-resolution metabolomics spectra — the article states the current
  version of SAND is tuned for high-resolution metabolomics spectra and other cases
  are not yet accommodated
- The data are multidimensional or nonuniformly sampled — adaptation of SAND to multidimensional
  applications remains an unexplored goal
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3382
tools:
- name: SAND
  role: Parent pipeline whose adaptive binning stage this skill implements; runs on
    NMRBox with test examples used for validation
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation language for the slow but functional adaptive-binning module
    (src/adaptive_binning.m); slated for replacement by a parallelizable C/C++ version
- name: NMRBox
  role: Cloud environment where updated binning code must run and pass the test examples
    before merge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:af981143005f819e61211b9b45774b0088c39da0c758c5e7fdb93c2ad23df3fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRBox
  - SAND
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# nmr-spectral-adaptive-binning

## Summary

Adaptive frequency-domain binning stage of the SAND (Spectral Automated NMR Decomposition) pipeline: partitions a 1D metabolomics NMR spectrum into frequency bins and converts each binned region into a complex per-bin time-domain FID via raised-cosine apodization, Hilbert transform, and inverse Fourier transform, preparing bins for downstream time-domain modeling.

## When to use

Use when a 1D frequency-domain (ppm-axis) metabolomics NMR spectrum must be partitioned into frequency bins and converted into per-bin time-domain FIDs for time-domain deconvolution or peak-table generation.

## When NOT to use

- The input is already a peak table or feature table — SAND's adaptive binning operates on raw 1D frequency-domain spectra before time-domain decomposition
- The data are not high-resolution metabolomics spectra — the article states the current version of SAND is tuned for high-resolution metabolomics spectra and other cases are not yet accommodated
- The data are multidimensional or nonuniformly sampled — adaptation of SAND to multidimensional applications remains an unexplored goal

## Inputs

- 1D frequency-domain NMR spectrum from the SAND repository test examples, with a mapped ppm axis
- Binning parameters: nominal bucket size (0.002 ppm), slackness criterion (0.99), raised-cosine roll-off width (0.02 ppm)

## Outputs

- A set of frequency bins partitioned from the ppm axis (with adjacent bins merged in groups of three where the slackness criterion is met)
- A complex per-bin time-domain FID for each binned region, produced via Hilbert transform and inverse Fourier transform
- A callable MATLAB module (src/adaptive_binning.m) exposing the binning stage with SAND base-function interface conventions

## How to apply

Load the 1D frequency-domain NMR spectrum and map its ppm axis, then partition the spectrum adaptively into buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where the criterion is met. Apply minimum-shift baseline removal within each binned region to correct intensity offsets before windowing. Apply a raised-cosine band with 0.02 ppm roll-off at the edges of each bin region to suppress spectral leakage between bins. Finally, apply a Hilbert transform to form the analytic signal (generating the imaginary data) and an inverse Fourier transform to convert each binned region into a complex per-bin time-domain FID. In the SAND codebase, this is packaged as a callable MATLAB module (e.g. src/adaptive_binning.m) following the interface conventions of the SAND base functions, and changes are validated by running the repository test examples in the NMRBox test harness.

## Related tools

- **SAND** (Parent pipeline whose adaptive binning stage this skill implements; runs on NMRBox with test examples used for validation) — https://github.com/edisonomics/SAND
- **MATLAB** (Implementation language for the slow but functional adaptive-binning module (src/adaptive_binning.m); slated for replacement by a parallelizable C/C++ version)
- **NMRBox** (Cloud environment where updated binning code must run and pass the test examples before merge)

## Evaluation signals

- The repository test examples in the SAND test harness pass after the binning module is updated (the merge requirement is that updates 'pass the test before merge')
- Bin boundaries respect the 0.002 ppm nominal bucket size with adjacent bins merged only in groups of three where the slackness criterion of 0.99 is met
- Each binned region is baseline-corrected by minimum-shift removal before raised-cosine windowing, and edge roll-off is 0.02 ppm
- The output for each bin is a complex time-domain FID whose length corresponds to the binned region after Hilbert-based analytic-signal formation and inverse Fourier transform
- The module exposes the SAND base-function interface conventions so it can be invoked within the SAND/NMRPipe workflow

## Limitations

- No quantitative metric is reported for the adaptive binning stage in isolation; validation relies on the SAND test examples passing rather than a binned-output accuracy measure
- SAND (and hence this binning stage) is currently implemented in slow MATLAB and is not yet readily parallelizable; a faster C/C++ version is planned
- The current version of SAND is tuned for high-resolution metabolomics spectra; other spectrum types are not yet accommodated
- Multidimensional extension is aspirational — the binning stage is designed for 1D spectra only

## Evidence

- [other] The band-selected bin is processed by Hilbert transform to generate imaginary data and then inverse Fourier-processed to produce an FID for each bin.: [evidence span withheld — anchor status: not_found]
- [discussion] We intend to replace the slow MATLAB implementation with a faster, more readily parallelizable version implemented in C/C++: [evidence span withheld — anchor status: not_found]
