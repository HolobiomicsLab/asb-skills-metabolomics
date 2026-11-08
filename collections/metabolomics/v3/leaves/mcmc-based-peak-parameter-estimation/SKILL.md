---
name: mcmc-based-peak-parameter-estimation
description: Use when 1D NMR spectra (especially highly overlapped metabolomics spectra)
  must be deconvoluted into peak tables with accurate peak parameters rather than
  fitted in the frequency domain.
when_to_use_negative:
- 'Input spectra are low-resolution or non-metabolomics: SAND''s current version is
  tuned for high-resolution metabolomics spectra'
- 'Signals are nonexponential: SAND does not yet handle nonexponential signals natively
  (exponential + Gaussian model only)'
- 'Data are multidimensional or nonuniformly sampled: adaptation to multidimensional
  applications is a future goal, not implemented'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Primary software performing hybrid MCMC optimization of time-domain spectra
    with time/frequency subsampling and training/validation splitting to produce peak
    tables
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation language of the current (slow) SAND version used to run the
    MCMC-based decomposition
- name: NMRbox
  role: Cloud computing environment where SAND is hosted and run via NMRPipe with
    a single terminal command
- name: NMRPipe
  role: Interface through which SAND is invoked on NMRbox resources
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:25b640ecf584d38abd607b5adc634203501c791c1e7c6401ae39e09b198530de
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRPipe
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# mcmc-based-peak-parameter-estimation

## Summary

Estimate NMR peak parameters by fitting time-domain FID data with SAND's hybrid Markov chain Monte Carlo optimization, which subsamples both time-domain and frequency-domain points and randomly splits the time-domain data into training and validation sets to avoid overfitting.

## When to use

Use when 1D NMR spectra (especially highly overlapped metabolomics spectra) must be deconvoluted into peak tables with accurate peak parameters rather than fitted in the frequency domain.

## When NOT to use

- Input spectra are low-resolution or non-metabolomics: SAND's current version is tuned for high-resolution metabolomics spectra
- Signals are nonexponential: SAND does not yet handle nonexponential signals natively (exponential + Gaussian model only)
- Data are multidimensional or nonuniformly sampled: adaptation to multidimensional applications is a future goal, not implemented

## Inputs

- Raw time-domain 1D NMR FID data (simulated or experimental, e.g. urine or worm metabolomics datasets)
- Optionally phased spectra (autophasing or manually adjusted phase values as initial pre-processing)
- NMRbox environment or local/HPC installation with SAND installed (install_SAND.sh under NMRBox_ADMIN_versions/SAND_V7)

## Outputs

- Peak tables (peak parameter estimates: frequency, amplitude, linewidth/phase) decomposed from each 1D spectrum
- Per-peak parameter estimates validated against ground truth (correlation ~0.9 on highly overlapped simulated data)

## How to apply

Run SAND (MATLAB implementation, runnable via NMRPipe on NMRbox or locally/on HPC) on the raw time-domain FID data. SAND uses hybrid optimization with Markov chain Monte Carlo methods, subsampling in both time and frequency domains, and randomly divides the time-domain data into training and validation sets to help avoid overfitting. Peaks identified in the first pass are subject to a second round of optimization using an exponential + Gaussian model. Initial pre-processing by the user (autophasing or manually adjusted phase values) is preferred but not always needed; the simulated 0.2 radian zero-order phase distortion test showed similar peak parameter estimation to phased spectra. Compare estimated peak parameters against ground truth (or expected metabolite peak lists) and compute correlations; accept results when correlation with ground truth is approximately 0.9 on highly overlapped simulated data, as reported for simulated datasets, a two-compound mixture, and a spiked urine sample.

## Related tools

- **SAND** (Primary software performing hybrid MCMC optimization of time-domain spectra with time/frequency subsampling and training/validation splitting to produce peak tables) — https://github.com/edisonomics/SAND
- **MATLAB** (Implementation language of the current (slow) SAND version used to run the MCMC-based decomposition)
- **NMRbox** (Cloud computing environment where SAND is hosted and run via NMRPipe with a single terminal command)
- **NMRPipe** (Interface through which SAND is invoked on NMRbox resources)

## Examples

```
Run SAND on NMRbox via NMRPipe, e.g.: SAND <input.fid> --output peaks.csv   (analysis is initiated via a single command from a terminal window or shell script per the paper; see also tutorial/ in github.com/edisonomics/SAND)
```

## Evaluation signals

- Correlation between estimated peak parameters and ground truth is approximately 0.9 on highly overlapped simulated data
- Phase-distorted inputs (e.g. 0.2 radian zero-order phase distortion on all peaks) yield peak parameter estimation similar to phased spectra (Figure S6/Table S3)
- Time-domain data are split into disjoint training and validation sets; validation-set fit does not show overfitting relative to training set
- Output is a per-spectrum peak table (feature table) with identified peaks subject to a second exponential + Gaussian optimization round
- On average 74% of peaks per compound can be recovered in single clusters via automated annotation with correlation networks

## Limitations

- Current MATLAB implementation is slow and not parallelizable; a C/C++ replacement is planned
- Tuned only for high-resolution metabolomics spectra; other spectral types not yet accommodated
- Does not yet handle nonexponential signals natively
- Autophasing or manual phase adjustment is preferred but not always required, so reliability of unphased input is only partially verified
- Multidimensional extension and NAN reproducible-quantification workflows are aspirational / in progress

## Evidence
