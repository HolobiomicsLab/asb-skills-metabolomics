---
name: time-domain-spectral-decomposition
description: Use when 1D NMR spectra (simulated, mixture, or biofluid such as urine)
  contain heavily overlapping peaks that frustrate frequency-domain peak picking,
  and the goal is automated peak-table extraction and metabolite quantification against
  ground truth.
when_to_use_negative:
- The spectra are multidimensional or nonuniformly sampled — SAND is not yet adapted
  to multidimensional applications, though time-domain subsampling makes extension
  possible in principle
- The signals are nonexponential — SAND does not yet handle nonexponential signals
  natively (only a second-round exponential + Gaussian refinement exists)
- A fast, parallelizable production pipeline is required — the current MATLAB implementation
  is slow; a C/C++ version is intended but not yet available
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Primary software performing automated time-domain modeling and decomposition
    of 1D NMR spectra into peak tables
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud computing environment hosting SAND; SAND can be run via NMRbox resources
    initiated by a single command, and updates must pass NMRBox environment tests
- name: NMRPipe
  role: Interface through which SAND can currently be used alongside NMRbox resources
- name: MATLAB
  role: Current (slow) implementation language of SAND's decomposition workflow
- name: C/C++
  role: Planned faster, parallelizable reimplementation of SAND not yet completed
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:6163c4cf0af674ddc85116e4728b0b82ab48dfccdc8e9dad8f7d64e6a043e2a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - C/C++
  - MATLAB
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# time-domain-spectral-decomposition

## Summary

Automated decomposition of 1D NMR spectra into peak tables by fitting exponentially damped sinusoids directly in the time domain using SAND's hybrid Markov chain Monte Carlo optimization, enabling metabolite quantification even on highly overlapped spectra.

## When to use

Use when 1D NMR spectra (simulated, mixture, or biofluid such as urine) contain heavily overlapping peaks that frustrate frequency-domain peak picking, and the goal is automated peak-table extraction and metabolite quantification against ground truth.

## When NOT to use

- The spectra are multidimensional or nonuniformly sampled — SAND is not yet adapted to multidimensional applications, though time-domain subsampling makes extension possible in principle
- The signals are nonexponential — SAND does not yet handle nonexponential signals natively (only a second-round exponential + Gaussian refinement exists)
- A fast, parallelizable production pipeline is required — the current MATLAB implementation is slow; a C/C++ version is intended but not yet available

## Inputs

- 1D NMR spectra (time-domain FIDs or frequency-domain spectra), simulated or experimental
- Simulated benchmark spectra with known ground-truth peak parameters (e.g., ~100 peaks in a 1.50 ppm region, random concentrations, exponentially damped sinusoids with lambda 1-15)
- Optionally phase-distorted spectra (e.g., zero-order phase distortion of 0.2 radians applied to all peaks)

## Outputs

- Peak tables with fitted peak parameters (position, amplitude, linewidth/decay) per spectrum
- Estimated compound concentrations with correlation and linear-regression slope (k) against ground truth
- Clustered peak annotations via correlation networks (on average 74% of peaks per compound recovered in single clusters)

## How to apply

Feed each spectrum through SAND's time-domain modeling workflow, which uses hybrid optimization with Markov chain Monte Carlo methods and subsampling in both time and frequency domains, while randomly dividing the time-domain data into training and validation sets to avoid overfitting. Optionally apply an initial pre-processing step with autophasing or manually adjusted phase values (preferred but not strictly required — SAND tolerates distortions such as a 0.2 radian zero-order phase shift). For peaks identified in the first pass, run a second round of optimization using an exponential + Gaussian model to refine peak parameters. Extract the resulting peak tables and, for quantification, map fitted peaks to compounds of origin and compute Pearson correlation and linear-regression slope against known concentrations — on highly overlapped simulated benchmarks (~100 peaks over a 1.50 ppm region, lambda 1–15) expect correlation ~0.9 and slope k ≈ 0.9. SAND can be run locally, on HPC, or via NMRPipe/NMRbox initiated from a single terminal command.

## Related tools

- **SAND** (Primary software performing automated time-domain modeling and decomposition of 1D NMR spectra into peak tables) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud computing environment hosting SAND; SAND can be run via NMRbox resources initiated by a single command, and updates must pass NMRBox environment tests)
- **NMRPipe** (Interface through which SAND can currently be used alongside NMRbox resources)
- **MATLAB** (Current (slow) implementation language of SAND's decomposition workflow)
- **C/C++** (Planned faster, parallelizable reimplementation of SAND not yet completed)

## Examples

```
SAND run via NMRbox/NMRPipe is initiated 'via a single command from a terminal window or shell script'; install locally from the repo with: git clone https://github.com/edisonomics/SAND && cd SAND && bash NMRBox_ADMIN_versions/SAND_V7/install_SAND.sh
```

## Evaluation signals

- On the highly overlapped simulated benchmark (~100 peaks, 1.50 ppm region, lambda 1-15), Pearson correlation of SAND-estimated vs. ground-truth concentrations reaches approximately 0.9 and linear-regression slope k ≈ 0.9; flag runs falling below this threshold as out-of-spec
- Adding a 0.2 radian zero-order phase distortion to all peaks should leave peak-parameter estimation performance similar to the phased (undistorted) baseline, reproducing Figure S6/Table S3 relative performance
- Automated annotation with correlation networks recovers on average 74% of peaks per compound in single clusters
- Time-domain training/validation random split is active (guard against overfitting) and peak-splitting behavior around ~7.3 and ~7.95 ppm is recorded on the simulated benchmark
- Peak tables are produced for both simulated and experimental datasets using the same workflow (urine and worm datasets tested)

## Limitations

- SAND is currently implemented in slow MATLAB and is not readily parallelizable; a faster C/C++ version is intended but not yet delivered
- The current version of SAND is tuned for high-resolution metabolomics spectra; other cases are not yet accommodated
- SAND does not yet handle nonexponential signals natively and is not yet adapted to multidimensional applications
- Initial pre-processing by autophasing or manually adjusted phase values is preferred but not always needed — reliability of completely unphased input should be verified per dataset
- Reproducible quantification workflows via NMRbox and the Network for Advanced NMR (NAN) are aspirational/in progress, not yet fully realized

## Evidence
