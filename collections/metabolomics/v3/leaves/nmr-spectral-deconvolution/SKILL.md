---
name: nmr-spectral-deconvolution
description: Use when 1D NMR spectra (e.g., urine metabolomics FIDs) have heavily
  overlapping peaks that bin-based integration cannot resolve, and the goal is an
  automated peak table for quantification, spike-in linearity checks, or correlation-network
  annotation.
when_to_use_negative:
- Input is nonexponential signal data or multidimensional / nonuniformly sampled NMR
  — SAND does not yet handle these natively
- Spectra are not high-resolution metabolomics data — the current version is tuned
  only for high-resolution metabolomics spectra
- A feature table already exists (e.g., peaks already deconvolved and quantified)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: SAND
  role: Core software performing the hybrid MCMC + interior-point time-domain decomposition
    into peak tables
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment hosting SAND; updates are run in the NMRBox environment
    and must pass tests before merge
- name: NMRPipe
  role: Interface through which SAND is invoked on NMRbox resources via a single terminal
    command or shell script
- name: MATLAB
  role: Current (slow) implementation language of SAND
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:9ce9ae7a0807426b929dce96703383be0824c69c06ea1246938a95e86fdb578e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-spectral-deconvolution

## Summary

Decompose 1D NMR spectra into per-peak parameter tables (amplitude, frequency, decay, phase) using SAND's hybrid MCMC + interior-point time-domain modeling, enabling automated metabolite quantification even on highly overlapped spectra (~0.9 correlation with ground truth).

## When to use

Use when 1D NMR spectra (e.g., urine metabolomics FIDs) have heavily overlapping peaks that bin-based integration cannot resolve, and the goal is an automated peak table for quantification, spike-in linearity checks, or correlation-network annotation.

## When NOT to use

- Input is nonexponential signal data or multidimensional / nonuniformly sampled NMR — SAND does not yet handle these natively
- Spectra are not high-resolution metabolomics data — the current version is tuned only for high-resolution metabolomics spectra
- A feature table already exists (e.g., peaks already deconvolved and quantified)

## Inputs

- 1D NMR spectra / FID time-domain data (e.g., high-resolution metabolomics spectra from urine or worm datasets)
- Predefined spectral bin boundaries and parameter ranges per bin

## Outputs

- peak_table.csv with amplitude, frequency, decay, and phase per peak, constrained to ≤7 peaks per bin
- Feature/peak tables suitable for downstream quantification (e.g., spike-in volume regression) and correlation-network annotation

## How to apply

Pre-process each spectrum with autophasing or manually adjusted phase values (preferred but not always needed), then run SAND, which randomly divides the time-domain data into training and validation sets to avoid overfitting. For each spectral bin, initialize peak parameters by residual-based estimation with a 1,000-point frequency sweep, then optimize via Metropolis-sampling MCMC with subsampling in both time and frequency domains at synthetic temperature 80; the inner MCMC step perturbs one parameter at random, bounding each perturbation to 10% of its predefined parameter range. Apply interior-point local refinement every 100 iterations, iterating up to the default 2,000 iterations, and enforce stop criteria including a maximum of 7 peaks per bin. Optionally subject identified peaks to a second round of optimization using an exponential + Gaussian model. Emit a per-peak table of amplitude, frequency, decay, and phase; SAND runs locally, on HPC, or via NMRPipe and NMRbox (single terminal command), with a MATLAB implementation (C/C++ rewrite planned).

## Related tools

- **SAND** (Core software performing the hybrid MCMC + interior-point time-domain decomposition into peak tables) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud environment hosting SAND; updates are run in the NMRBox environment and must pass tests before merge)
- **NMRPipe** (Interface through which SAND is invoked on NMRbox resources via a single terminal command or shell script)
- **MATLAB** (Current (slow) implementation language of SAND)

## Evaluation signals

- peak_table.csv exists, contains amplitude/frequency/decay/phase fields, and holds a nonempty peak list consistent with the ≤7-peak-per-bin constraint
- On highly overlapped simulated data, two-compound mixtures, or spiked urine, quantified amplitudes correlate ~0.9 with ground truth
- Spike-in urine series (0–80 μL): SAND-quantified peak amplitudes of spiked compounds correlate close to 1 with volume, urine-only peaks show no systematic trend, and glucose amplitude-vs-volume regression has a clearly nonzero intercept
- Training/validation split of the time-domain data confirms no overfitting during optimization

## Limitations

- Current SAND implementation is slow MATLAB code and not yet parallelizable (C/C++ version intended)
- Tuned only for high-resolution metabolomics spectra; other cases are not yet accommodated
- Does not natively handle nonexponential signals; multidimensional adaptation is a future goal
- Autophasing or manual phase adjustment is preferred but not always needed, so unphased input reliability is unverified

## Evidence
