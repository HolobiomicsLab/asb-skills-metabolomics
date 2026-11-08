---
name: zero-order-phase-distortion-application
description: 'Use when validating that a spectral deconvolution method is robust to
  phase errors: add a known zero-order phase distortion (e.g., 0.2 rad) to simulated
  spectra with known ground-truth peak parameters and compare parameter recovery to
  the phased baseline.'
when_to_use_negative:
- Input is experimental rather than simulated data — without ground-truth peak parameters
  there is no reference against which to compute the phase-distortion recovery correlation
- The spectral model requires nonzero-order (first-order) phase handling or nonexponential
  signals, which SAND does not yet natively support
- The spectra are not high-resolution metabolomics 1D spectra; the current SAND version
  is tuned for that data type
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: SAND
  role: Time-domain spectral decomposition of both phase-distorted and phased spectra
    into peak tables using hybrid MCMC optimization with time- and frequency-domain
    subsampling
  repo: github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation environment in which SAND's hybrid Markov chain Monte Carlo
    optimization is executed for the distortion test
- name: NMRbox
  role: Cloud computing environment hosting SAND, used to run the decomposition on
    phase-distorted and phased spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:31b0d8f3855dab78708449cd4c55deec9b484d45e60f938db588326733515c64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# zero-order-phase-distortion-application

## Summary

A robustness-testing procedure that applies a 0.2 radian zero-order phase distortion to all peaks in simulated 1D NMR spectra and compares SAND's peak parameter estimation against results on phased spectra, benchmarked against SAND's reported ~0.9 correlation with ground truth.

## When to use

Use when validating that a spectral deconvolution method is robust to phase errors: add a known zero-order phase distortion (e.g., 0.2 rad) to simulated spectra with known ground-truth peak parameters and compare parameter recovery to the phased baseline.

## When NOT to use

- Input is experimental rather than simulated data — without ground-truth peak parameters there is no reference against which to compute the phase-distortion recovery correlation
- The spectral model requires nonzero-order (first-order) phase handling or nonexponential signals, which SAND does not yet natively support
- The spectra are not high-resolution metabolomics 1D spectra; the current SAND version is tuned for that data type

## Inputs

- Simulated 1D NMR spectra with known ground-truth peak parameters (same ground truth as the paper's simulation study)
- SAND MATLAB implementation available via NMRbox or NMRPipe

## Outputs

- Phase-distorted test set of 1D NMR spectra (0.2 rad zero-order distortion applied to all peaks)
- Peak parameter tables (decomposed peak parameters) for both phase-distorted and phased conditions
- Correlation of estimated peak parameters with ground truth for each condition (cf. Figure S6/Table S3)

## How to apply

First regenerate the simulated 1D NMR spectra using the same ground-truth peak parameters as the original simulation study, establishing the phased baseline. Then apply a zero-order phase distortion of 0.2 radians to all peaks in every spectrum to create the phase-distorted test set. Run SAND (MATLAB implementation, e.g., via NMRbox) identically on both the phase-distorted and phased spectra, using its hybrid Markov chain Monte Carlo optimization with subsampling in both time and frequency domains, and random division of the time-domain data into training and validation sets to avoid overfitting. Compare estimated peak parameters against ground truth for both conditions and compute the correlation with ground truth for each. Accept the result when the phase-distorted case reproduces the relative performance reported in Figure S6/Table S3, benchmarked against SAND's reported correlation of approximately 0.9 with ground truth on highly overlapped simulated data.

## Related tools

- **SAND** (Time-domain spectral decomposition of both phase-distorted and phased spectra into peak tables using hybrid MCMC optimization with time- and frequency-domain subsampling) — github.com/edisonomics/SAND
- **MATLAB** (Implementation environment in which SAND's hybrid Markov chain Monte Carlo optimization is executed for the distortion test)
- **NMRbox** (Cloud computing environment hosting SAND, used to run the decomposition on phase-distorted and phased spectra)

## Evaluation signals

- Correlation between SAND-estimated peak parameters and ground truth on the 0.2 rad phase-distorted spectra is similar to the phased baseline (target: ~0.9, per the paper's highly overlapped simulated data benchmark)
- Phase-distorted result reproduces the relative performance reported in Figure S6/Table S3
- Identical SAND configuration (hybrid MCMC, time/frequency subsampling, random train/validation split of time-domain data) is applied to both conditions so the comparison is controlled
- The same ground-truth peak parameters are used when regenerating both the phased and phase-distorted simulated spectra

## Limitations

- SAND's initial pre-processing via autophasing or manually adjusted phase values is preferred but not always needed; reliability of unphased input was an open question addressed by this test
- SAND's current implementation is a slow MATLAB version, not yet parallelizable in C/C++, which limits throughput for large distortion-test batches
- The current version of SAND is tuned for high-resolution metabolomics spectra; other cases are not yet accommodated

## Evidence

- [results] SAND successfully decomposes simulated spectra with a zero-order phase distortion, and its peak parameter estimation performance is similar to that on phased spectra.: [evidence span withheld — anchor status: not_found]
