---
name: markov-chain-monte-carlo-optimization
description: Use when quantifying overlapped 1D NMR metabolomics spectra (e.g. ~100
  peaks in a 1.50 ppm region) from time-domain FID data and peak tables must be derived
  automatically without manual deconvolution.
when_to_use_negative:
- Input spectra are nonexponential in signal shape — SAND is tuned for high-resolution
  metabolomics spectra and does not yet handle nonexponential signals natively.
- Data are multidimensional or nonuniformly sampled in a way that exceeds current
  1D scope — SAND's ultimate goal is to adapt to multidimensional applications, but
  that is not yet realized.
- A peak table already exists from prior deconvolution and only needs aggregation
  or annotation.
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: SAND
  role: Software implementing the hybrid MCMC time-domain decomposition that produces
    peak tables from 1D NMR spectra
  repo: github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud computing environment where SAND is hosted and run via a single terminal
    command
  repo: https://nmrbox.org
- name: MATLAB
  role: Current (slow) implementation language of the SAND MCMC optimizer; slated
    for replacement by C/C++
- name: C/C++
  role: Planned parallelizable implementation of the SAND optimizer
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:e8f28856bb18964b1309640320f6b345cdfcccbb8622d92f77d60e035a86bc66
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C/C++
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# markov-chain-monte-carlo-optimization

## Summary

Time-domain modeling skill in which SAND fits 1D NMR spectra by hybrid Metropolis-sampled Markov chain Monte Carlo plus interior-point refinement, iteratively per spectral bin to build a peak-parameter table.

## When to use

Use when quantifying overlapped 1D NMR metabolomics spectra (e.g. ~100 peaks in a 1.50 ppm region) from time-domain FID data and peak tables must be derived automatically without manual deconvolution.

## When NOT to use

- Input spectra are nonexponential in signal shape — SAND is tuned for high-resolution metabolomics spectra and does not yet handle nonexponential signals natively.
- Data are multidimensional or nonuniformly sampled in a way that exceeds current 1D scope — SAND's ultimate goal is to adapt to multidimensional applications, but that is not yet realized.
- A peak table already exists from prior deconvolution and only needs aggregation or annotation.

## Inputs

- 1D NMR time-domain FID data (simulated exponentially damped sinusoids or experimental urine/worm spectra)
- Predefined peak-parameter ranges (amplitude, frequency, decay, phase) per spectral bin
- Optionally pre-phased spectra (autophasing or manually adjusted phase values preferred but not always needed)

## Outputs

- Per-bin peak-parameter table (peak_table.csv) with amplitude, frequency, decay, and phase fields
- Metabolite concentration estimates correlated against ground truth (~0.9 Pearson correlation on the simulated benchmark)

## How to apply

Load the input 1D NMR spectrum and randomly divide its time-domain data into training and validation subsets to avoid overfitting. For each spectral bin, initialize peak parameters by residual-based estimation from a 1,000-point frequency sweep, then run hybrid optimization: the Metropolis-sampling MCMC inner step perturbs one randomly selected parameter at a time, bounding each perturbation to 10% of the predefined parameter range. Apply interior-point local refinement of the sampled parameters every 100 iterations (interval refinement of the sampled parameters is applied at intervals during the default 2,000 iterations) at synthetic temperature 80, with subsampling in both the time and frequency domains. Terminate the per-bin loop when one of the model-complexity stop criteria is met, including a maximum of 7 peaks per bin, then emit the per-peak table of amplitude, frequency, decay, and phase parameters. Rationale: the random training/validation subsampling plus stochastic MCMC search keeps the fit robust on highly overlapped synthetic and experimental (urine, worm) datasets where deterministic least squares fails.

## Related tools

- **SAND** (Software implementing the hybrid MCMC time-domain decomposition that produces peak tables from 1D NMR spectra) — github.com/edisonomics/SAND
- **NMRbox** (Cloud computing environment where SAND is hosted and run via a single terminal command) — https://nmrbox.org
- **MATLAB** (Current (slow) implementation language of the SAND MCMC optimizer; slated for replacement by C/C++)
- **C/C++** (Planned parallelizable implementation of the SAND optimizer)

## Examples

```
sand --input fid.dat --output peak_table.csv
```

## Evaluation signals

- Verify peak_table.csv exists and contains nonempty amplitude, frequency, decay, and phase fields per bin.
- Check the peak list obeys the model-complexity stop criterion: no more than 7 peaks per spectral bin.
- On the simulated benchmark (~100 peaks, 1.50 ppm window, lambda 1-15), confirm Pearson correlation ≈ 0.9 and slope k ≈ 0.9 against ground truth; flag runs below threshold.
- Confirm peaks identified by SAND survive the second-round exponential + Gaussian model optimization without discarding overlapped compound signals (e.g. around 7.3 and 7.95 ppm).
- Confirm the training/validation time-domain subsampling was applied (random split) rather than fitting on all points, to avoid overfitting.

## Limitations

- SAND is tuned for high-resolution metabolomics spectra; other data types are not yet accommodated.
- SAND does not yet handle nonexponential signals natively, though expansion is planned.
- The current MATLAB implementation is slow and not readily parallelizable; a faster C/C++ version is planned but not delivered.
- Multidimensional (nD) applications remain an 'intriguing possibility' — not yet implemented.
- Initial pre-processing by the user using either autophasing or manually adjusted phase values is preferred but not always needed, so unphased input may yield unreliable fits.

## Evidence

- [results] Within the per-bin iterative loop, the hybrid optimizer's inner (MCMC) step selects one parameter at random, perturbs it, and bounds each perturbation to 10% of the predefined parameter range.: [evidence span withheld — anchor status: not_found]
