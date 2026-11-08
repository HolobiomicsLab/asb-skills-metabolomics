---
name: matlab-signal-processing-module-development
description: Use when implementing or modifying a spectral preprocessing stage of
  SAND in MATLAB — e.g., reconstructing adaptive binning that converts 1D frequency-domain
  NMR spectra into per-bin time-domain FIDs — and it must pass SAND's NMRBox tests
  before merge.
when_to_use_negative:
- The goal is raw performance or parallelism — the article notes the current MATLAB
  implementation is slow and the team intends to replace it with a faster, parallelizable
  C/C++ version
- The input is already a peak/feature table — this skill operates on 1D frequency-domain
  NMR spectra prior to time-domain decomposition
- The spectra are not high-resolution metabolomics spectra — SAND's current version
  is tuned only for high-resolution metabolomics spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MATLAB
  role: Implementation language for the signal-processing module (adaptive binning,
    raised-cosine banding, Hilbert transform, inverse FT), following SAND base-function
    conventions
- name: SAND
  role: Framework whose base-function interface conventions the module must follow,
    and whose repository test examples serve as the validation harness
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Environment in which updates must be run and tested before merge; test server
    is requested and code is copied into /usr/software/sand
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:3500b9609133cd8d79ee59cccaaaad215cfd80655af9b535d8845517b0b560ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# matlab-signal-processing-module-development

## Summary

Wraps NMR spectral preprocessing stages (e.g., SAND's adaptive frequency binning with raised-cosine banding, Hilbert transform, and per-bin inverse Fourier transform) as a callable MATLAB module, following the conventions of the SAND base functions and its NMRBox test harness.

## When to use

Use when implementing or modifying a spectral preprocessing stage of SAND in MATLAB — e.g., reconstructing adaptive binning that converts 1D frequency-domain NMR spectra into per-bin time-domain FIDs — and it must pass SAND's NMRBox tests before merge.

## When NOT to use

- The goal is raw performance or parallelism — the article notes the current MATLAB implementation is slow and the team intends to replace it with a faster, parallelizable C/C++ version
- The input is already a peak/feature table — this skill operates on 1D frequency-domain NMR spectra prior to time-domain decomposition
- The spectra are not high-resolution metabolomics spectra — SAND's current version is tuned only for high-resolution metabolomics spectra

## Inputs

- 1D frequency-domain NMR spectrum with ppm axis (from SAND repository test examples)
- Binning parameters: nominal bin size (0.002 ppm), slackness criterion (0.99), raised-cosine roll-off (0.02 ppm)

## Outputs

- Callable MATLAB module (e.g., src/adaptive_binning.m) conforming to SAND base-function interfaces
- Complex per-bin time-domain FID data, one FID per adaptively binned spectral region

## How to apply

Load a 1D frequency-domain NMR spectrum from the SAND repository test examples and map its ppm axis. Partition the spectrum adaptively into buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where the criterion is met, then apply minimum-shift baseline removal within each binned region. Apply a raised-cosine band with 0.02 ppm roll-off at bin edges to suppress spectral leakage, then apply a Hilbert transform to form the analytic signal and an inverse Fourier transform to convert each binned region into a complex per-bin time-domain FID. Save the code as a callable MATLAB module (e.g., src/adaptive_binning.m) following the interface conventions of the SAND base functions. Before merging, run the module on the repository test examples in the SAND test harness — request a test NMRBox server, copy updated code into /usr/software/sand, and run the test examples.

## Related tools

- **MATLAB** (Implementation language for the signal-processing module (adaptive binning, raised-cosine banding, Hilbert transform, inverse FT), following SAND base-function conventions)
- **SAND** (Framework whose base-function interface conventions the module must follow, and whose repository test examples serve as the validation harness) — https://github.com/edisonomics/SAND
- **NMRbox** (Environment in which updates must be run and tested before merge; test server is requested and code is copied into /usr/software/sand)

## Examples

```
matlab -batch "fid = adaptive_binning(spectrum, ppm, 'binSize', 0.002, 'slackness', 0.99, 'rolloff', 0.02);"
```

## Evaluation signals

- Module passes the SAND test examples in the test harness (the paper's stated merge requirement is that updates 'pass the test before merge')
- Output FIDs are complex, time-domain, and produced one per binned spectral region
- Bin boundaries respect the nominal 0.002 ppm size with the 0.99 slackness criterion and three-bin merging where applicable
- Raised-cosine roll-off of 0.02 ppm is applied at the edges of each bin region, suppressing spectral leakage
- Module interface matches the conventions of the SAND base functions and runs in the NMRBox environment (code in /usr/software/sand)

## Limitations

- The MATLAB implementation is slow and not readily parallelizable; the authors intend to replace it with a C/C++ version
- No quantitative metric is reported for this preprocessing stage in isolation — validation is only that the test examples pass
- The current SAND version is tuned for high-resolution metabolomics spectra; other cases are not yet accommodated
- Updates must run in the NMRBox environment and pass tests before merge, which constrains local-only development workflows

## Evidence

- [other] The band-selected bin is processed by Hilbert transform to generate imaginary data and then inverse Fourier-processed to produce an FID for each bin.: [evidence span withheld — anchor status: not_found]
