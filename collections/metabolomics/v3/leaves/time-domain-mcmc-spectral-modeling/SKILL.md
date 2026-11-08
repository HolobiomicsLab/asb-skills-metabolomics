---
name: time-domain-mcmc-spectral-modeling
description: Use when you need to deconvolute high-resolution 1D NMR spectra (e.g.,
  urine metabolomics FIDs/spectra) into peak feature tables and quantify metabolite
  amplitudes, especially with overlapping peaks.
when_to_use_negative:
- Input spectra are not high-resolution metabolomics 1D NMR spectra — the current
  version of SAND is tuned for high-resolution metabolomics spectra
- Signals are nonexponential — SAND does not yet handle nonexponential signals natively
- Data are multidimensional — SAND is not yet adapted to multidimensional applications
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: SAND
  role: Core software that performs the time-domain MCMC deconvolution of 1D NMR spectra
    into peak tables
  repo: github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud computing environment where SAND is hosted and analysis is initiated
    via a single command from a terminal window or shell script
- name: NMRPipe
  role: Interface through which SAND can currently be used, leveraging NMRbox resources
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:c6e987faab27d85e058f03e70f1233810ab86140806cf0fa526f13d674913b6b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# time-domain-mcmc-spectral-modeling

## Summary

Automated time-domain modeling of 1D NMR spectra via SAND, which uses hybrid Markov chain Monte Carlo optimization with time- and frequency-domain subsampling to deconvolute spectra into peak tables for metabolite quantification.

## When to use

Use when you need to deconvolute high-resolution 1D NMR spectra (e.g., urine metabolomics FIDs/spectra) into peak feature tables and quantify metabolite amplitudes, especially with overlapping peaks.

## When NOT to use

- Input spectra are not high-resolution metabolomics 1D NMR spectra — the current version of SAND is tuned for high-resolution metabolomics spectra
- Signals are nonexponential — SAND does not yet handle nonexponential signals natively
- Data are multidimensional — SAND is not yet adapted to multidimensional applications

## Inputs

- 1D NMR spectra (high-resolution metabolomics, e.g., urine spectra) or their time-domain data
- Optionally phase-corrected spectra (autophasing or manual phase values)

## Outputs

- Peak tables (deconvoluted peak features with amplitudes)
- Compound-level quantification (clustered peaks with correlation networks)

## How to apply

Run SAND (via NMRbox or NMRPipe) on the 1D NMR spectrum: SAND automatically processes and deconvolutes it into a peak table using hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains, and randomly dividing the time-domain data into training and validation sets to avoid overfitting. Optionally perform initial pre-processing (autophasing or manually adjusted phase values) beforehand, which is preferred but not always needed. Peaks identified by SAND can be subject to a second round of optimization using an exponential + Gaussian model. Peaks of the same compound can be grouped via automated annotation with correlation networks, which recovers on average 74% of peaks per compound in single clusters. Quantification accuracy was demonstrated as a correlation of ~0.9 with ground truth on highly overlapped simulated data, a two-compound mixture, and a spiked urine sample.

## Related tools

- **SAND** (Core software that performs the time-domain MCMC deconvolution of 1D NMR spectra into peak tables) — github.com/edisonomics/SAND
- **NMRbox** (Cloud computing environment where SAND is hosted and analysis is initiated via a single command from a terminal window or shell script)
- **NMRPipe** (Interface through which SAND can currently be used, leveraging NMRbox resources)

## Evaluation signals

- Quantified peak amplitudes should show a correlation of ~0.9 with ground truth on spiked/overlapped test cases (e.g., spiked urine)
- Spike-in compound amplitudes vs. spike-in volume should yield Pearson correlations close to 1 (e.g., 0–80 μL five-compound mixture in urine)
- Compounds present only in the matrix (urine-only peaks) should show no systematic trend across a spike-in series
- On average ~74% of peaks per compound should be recovered in single clusters by the correlation-network annotation
- Training/validation split behavior: overfitting is controlled by random division of time-domain data into training and validation sets

## Limitations

- Current implementation is slow MATLAB code; a faster, more readily parallelizable C/C++ version is intended but not yet available
- SAND is tuned only for high-resolution metabolomics spectra; other cases are not yet accommodated
- SAND does not yet handle nonexponential signals natively
- Not yet adapted to multidimensional applications
- Autophasing or manual phase adjustment is preferred but not always required — reliability of unphased input is unverified
- Reproducible quantification workflows via NMRbox and the Network for Advanced NMR (NAN) are still aspirational/in progress

## Evidence
