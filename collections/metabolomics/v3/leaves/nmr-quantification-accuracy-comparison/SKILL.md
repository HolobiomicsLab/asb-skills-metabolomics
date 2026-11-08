---
name: nmr-quantification-accuracy-comparison
description: Use when simulated 1D NMR spectra with known ground-truth peak parameters
  are available and you need to quantify how a perturbation (e.g., zero-order phase
  distortion) or preprocessing choice degrades peak parameter estimation relative
  to a phased baseline.
when_to_use_negative:
- Input is experimental data without known ground-truth peak parameters — the correlation-with-ground-truth
  metric cannot be computed
- Spectra are low-resolution or non-metabolomics — the article states the current
  SAND version is tuned only for high-resolution metabolomics spectra
- The comparison target is multidimensional or nonuniformly sampled NMR data — SAND
  is not yet adapted to multidimensional applications
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Performs the time-domain spectral decomposition and peak parameter estimation
    on both the phase-distorted and phased simulated spectra
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Runs the SAND implementation with its hybrid Markov chain Monte Carlo optimization
    for both test conditions
- name: NMRbox
  role: Cloud computing environment in which SAND is available and can be executed
    for the comparison
  repo: https://nmrbox.org
- name: NMRPipe
  role: Interface through which SAND can be invoked for the decomposition runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:db60c54d73d9f26501adb42c692bc542979bd7179b0a1ac4cc5417aa73a703ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-quantification-accuracy-comparison

## Summary

Benchmark a spectral deconvolution method's peak parameter estimation accuracy by comparing estimated peak parameters against known ground truth under a perturbed condition (e.g., a 0.2 radian zero-order phase distortion) versus a phased baseline, using correlation with ground truth as the primary metric (SAND benchmark: ~0.9 on highly overlapped simulated data).

## When to use

Use when simulated 1D NMR spectra with known ground-truth peak parameters are available and you need to quantify how a perturbation (e.g., zero-order phase distortion) or preprocessing choice degrades peak parameter estimation relative to a phased baseline.

## When NOT to use

- Input is experimental data without known ground-truth peak parameters — the correlation-with-ground-truth metric cannot be computed
- Spectra are low-resolution or non-metabolomics — the article states the current SAND version is tuned only for high-resolution metabolomics spectra
- The comparison target is multidimensional or nonuniformly sampled NMR data — SAND is not yet adapted to multidimensional applications

## Inputs

- Simulated 1D NMR spectra with known ground-truth peak parameters
- Phase-distorted version of the same spectra (zero-order phase distortion of 0.2 radians applied to all peaks)
- Phased (undistorted) baseline spectra

## Outputs

- Estimated peak parameter tables (peak positions, amplitudes, widths) for each condition
- Correlation of estimated peak parameters with ground truth per condition (distorted vs phased baseline)
- Pass/fail validation against the reference correlation (~0.9) from the benchmark figure/table (Figure S6/Table S3)

## How to apply

Regenerate the simulated 1D NMR spectra using the same ground-truth peak parameters as the reference simulation study, then create a perturbed test set by applying a zero-order phase distortion of 0.2 radians to all peaks. Run the deconvolution method (SAND, implemented in MATLAB, using its hybrid Markov chain Monte Carlo optimization with subsampling in both time and frequency domains and random division of the time-domain data into training and validation sets) identically on both the phase-distorted spectra and the phased (undistorted) spectra. Estimate peak parameters for both conditions and compute the correlation between estimated parameters and ground truth for each. Judge success by reproducing the reported relative performance: accept the result when the perturbed case matches the baseline's correlation, benchmarked against SAND's reported correlation of approximately 0.9 with ground truth on highly overlapped simulated data.

## Related tools

- **SAND** (Performs the time-domain spectral decomposition and peak parameter estimation on both the phase-distorted and phased simulated spectra) — https://github.com/edisonomics/SAND
- **MATLAB** (Runs the SAND implementation with its hybrid Markov chain Monte Carlo optimization for both test conditions)
- **NMRbox** (Cloud computing environment in which SAND is available and can be executed for the comparison) — https://nmrbox.org
- **NMRPipe** (Interface through which SAND can be invoked for the decomposition runs)

## Evaluation signals

- Correlation between estimated peak parameters and ground truth is computed separately for the 0.2 rad phase-distorted and phased conditions
- Phase-distorted correlation should be similar to the phased baseline correlation, reproducing the relative performance of the reference study
- Both correlations should be close to the reported ~0.9 benchmark on highly overlapped simulated data
- Identical SAND settings (same MCMC hybrid optimization, same time/frequency subsampling, same training/validation split procedure) are used for both conditions so the comparison is controlled
- Result should match the reported relative performance in Figure S6/Table S3 of the source paper

## Limitations

- SAND's initial pre-processing by the user via autophasing or manually adjusted phase values is preferred but not always needed — the reliability of fully unphased input is not guaranteed
- The current SAND implementation in MATLAB is slow and not readily parallelizable; a faster C/C++ version is planned
- SAND is tuned only for high-resolution metabolomics spectra; other spectral types are not yet accommodated
- Validation thresholds are anchored to the paper's specific simulation setup (0.2 rad zero-order distortion, highly overlapped simulated data); other distortion magnitudes or spectral complexity are untested

## Evidence
