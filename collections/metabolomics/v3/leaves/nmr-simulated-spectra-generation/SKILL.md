---
name: nmr-simulated-spectra-generation
description: Use when you need controlled benchmark spectra with known ground truth
  — e.g., to validate SAND quantification (~0.9 correlation), cluster recovery (~74%),
  or robustness to 0.2 rad phase distortion.
when_to_use_negative:
- Input is already experimental 1D NMR data (urine, worm datasets) intended for direct
  SAND decomposition rather than controlled benchmarking
- You need spectra of NMR data types SAND is not tuned for — the current version targets
  high-resolution metabolomics spectra
- Multidimensional (nD) or nonuniformly sampled acquisitions are required — SAND is
  not yet adapted to multidimensional applications
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: SAND
  role: Consumes the simulated spectra to produce fitted peak tables for comparison
    against ground truth
  repo: github.com/edisonomics/SAND
- name: MATLAB
  role: Environment used to generate the simulated spectra and run SAND's hybrid MCMC
    decomposition on them
- name: NMRbox
  role: Cloud environment where simulated-data workflows with SAND can be run
  repo: https://nmrbox.org
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:25fe49b3dfd44b99e8ed3b69db3fb2b2e3fc546cc62ff159bdd79a70bd743ec5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C/C++
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nmr-simulated-spectra-generation

## Summary

Generate simulated 1D NMR spectra with known ground-truth peak parameters (highly overlapped synthetic peaks or multi-compound mixtures) to benchmark SAND's time-domain decomposition against ground truth.

## When to use

Use when you need controlled benchmark spectra with known ground truth — e.g., to validate SAND quantification (~0.9 correlation), cluster recovery (~74%), or robustness to 0.2 rad phase distortion.

## When NOT to use

- Input is already experimental 1D NMR data (urine, worm datasets) intended for direct SAND decomposition rather than controlled benchmarking
- You need spectra of NMR data types SAND is not tuned for — the current version targets high-resolution metabolomics spectra
- Multidimensional (nD) or nonuniformly sampled acquisitions are required — SAND is not yet adapted to multidimensional applications

## Inputs

- Ground-truth peak parameter definitions (~100 peaks over a 1.50 ppm region with random concentrations and lambda 1–15)
- 18 predefined compound groups with fixed within-group relative amplitudes
- Random seeds / sampling ranges for concentrations (max down to 20%)
- Optional zero-order phase distortion value (0.2 radians)

## Outputs

- Simulated 1D NMR spectra (e.g., a benchmark set of heavily overlapped spectra; 50 mixture spectra)
- Ground-truth peak tables mapping each peak to its compound of origin and true concentration
- Phase-distorted test spectra (all peaks shifted by 0.2 rad) paired with un-phased baselines

## How to apply

For the overlapped benchmark, synthesize ~100 exponentially damped sinusoid peaks within a 1.50 ppm region, randomly sampling concentrations and damping factors lambda in the range 1–15, so that peaks are heavily overlapped. For mixture-based simulations, generate spectra (e.g., 50 samples) from 18 predefined compound groups, holding within-group relative amplitudes fixed while sampling each compound's concentration from its maximum down to 20%. For robustness tests, regenerate spectra with the same ground-truth peak parameters and optionally apply a zero-order phase distortion of 0.2 radians to all peaks. Store the ground-truth peak tables (position, amplitude, lambda, compound of origin) so SAND's fitted peak tables can be mapped back to compounds, and compute correlation, regression slope, or per-compound cluster recovery against them.

## Related tools

- **SAND** (Consumes the simulated spectra to produce fitted peak tables for comparison against ground truth) — github.com/edisonomics/SAND
- **MATLAB** (Environment used to generate the simulated spectra and run SAND's hybrid MCMC decomposition on them)
- **NMRbox** (Cloud environment where simulated-data workflows with SAND can be run) — https://nmrbox.org

## Evaluation signals

- On the highly overlapped simulated benchmark, SAND-estimated concentrations should reach Pearson correlation ≈0.88–0.9 and linear-regression slope k ≈0.9 with ground truth
- Across the 50-spectra / 18-compound-group mixture set, mean fraction of ground-truth peaks recovered in a single cluster should be ≈74% (never below 60% per the reported range)
- Phase-distorted spectra (0.2 rad) should yield peak parameter estimates similar in correlation to the phased baseline (Figure S6/Table S3 behavior)
- Every fitted SAND peak must map to a ground-truth compound of origin via the stored ground-truth peak table
- Peak-splitting artifacts near ~7.3 and ~7.95 ppm should be observed and recorded when comparing fitted vs. ground-truth peaks

## Limitations

- Simulated spectra assume exponentially damped sinusoid signals; SAND does not yet handle nonexponential signals natively
- Ground truth only exists by construction — conclusions transfer to experimental spectra only insofar as simulations match high-resolution metabolomics conditions
- Phase handling is asymmetric: autophasing or manual phase adjustment is preferred before analysis, though 'not always needed'

## Evidence

- [other] create ~100 peaks within a 1.50 ppm region, with randomly sampled concentrations and exponentially damped sinusoid parameters (lambda in the range 1-15): [evidence span withheld — anchor status: not_found]
- [other] holding within-group relative amplitudes fixed while sampling each compound's concentration from its maximum down to 20%: [evidence span withheld — anchor status: not_found]
