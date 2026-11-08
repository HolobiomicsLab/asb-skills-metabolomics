---
name: residual-based-peak-parameter-initialization
description: Use when initializing peak parameters for SAND's per-bin time-domain
  decomposition of 1D NMR spectra so the hybrid optimizer starts from data-driven
  seeds rather than arbitrary guesses.
when_to_use_negative:
- The spectrum is already a peak/feature table or another program's deconvolution
  output — no time-domain residuals exist to sweep
- You need SAND's second-round exponential + Gaussian re-optimization of already-identified
  peaks; that refinement is separate from residual-based seeding
- A nonexponential or multidimensional NMR dataset is being analyzed; the paper states
  SAND is currently tuned only for 1D high-resolution metabolomics spectra
edam_topics:
- http://edamontology.org/topic_3176
tools:
- name: SAND
  role: Provides the residual-based initialization and per-bin decomposition loop
    that the skill implements
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment where SAND (and thus the initialization step) is run via
    a single terminal command
  repo: https://nmrbox.org
- name: NMRPipe
  role: Command-line interface through which SAND, including the initialization stage,
    can be invoked
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:85337b212824c073f5296ef1be1fc05cbdc3a5ba1b30a31385364345553df8b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3176
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# residual-based-peak-parameter-initialization

## Summary

Residual-based peak-parameter initialization in SAND sweeps a 1,000-point frequency grid on each binned FID's residual spectrum to seed amplitude, frequency, decay, and phase starting values before hybrid MCMC optimization.

## When to use

Use when initializing peak parameters for SAND's per-bin time-domain decomposition of 1D NMR spectra so the hybrid optimizer starts from data-driven seeds rather than arbitrary guesses.

## When NOT to use

- The spectrum is already a peak/feature table or another program's deconvolution output — no time-domain residuals exist to sweep
- You need SAND's second-round exponential + Gaussian re-optimization of already-identified peaks; that refinement is separate from residual-based seeding
- A nonexponential or multidimensional NMR dataset is being analyzed; the paper states SAND is currently tuned only for 1D high-resolution metabolomics spectra

## Inputs

- Binned 1D NMR FID (time-domain data) for one spectral bin
- Current fitted model parameters for the bin (to compute residuals)
- Predefined parameter ranges for amplitude, frequency, decay, and phase

## Outputs

- Initial peak-parameter table entries (amplitude, frequency, decay, phase) per candidate peak
- Updated residual spectrum after seeding new peaks

## How to apply

For each spectral bin, first subtract the model currently fitted to that bin and compute the residual spectrum. Sweep a 1,000-point grid across the residual's frequency axis to locate candidate new peaks. At each candidate frequency, seed the four SAND parameters (amplitude, frequency, decay, phase) from the residual's local shape rather than random guesses. Hand these initialized parameters to the hybrid optimizer's Metropolis-sampling MCMC stage, where perturbations are bounded to 10% of the predefined parameter range; this keeps the search near data-supported values. Cap the seeded peaks at the stop-criteria limit of 7 peaks per bin and continue the iterative loop until a model-complexity criterion is met.

## Related tools

- **SAND** (Provides the residual-based initialization and per-bin decomposition loop that the skill implements) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud environment where SAND (and thus the initialization step) is run via a single terminal command) — https://nmrbox.org
- **NMRPipe** (Command-line interface through which SAND, including the initialization stage, can be invoked)

## Examples

```
peaks identified by SAND subject to a second round of optimization using an exponential + Gaussian model
```

## Evaluation signals

- Peak table contains the amplitude/frequency/decay/phase fields and a nonempty peak list consistent with the ≤7-peak-per-bin constraint
- Initial seeds lie within the predefined parameter ranges, and MCMC perturbations stay bounded to 10% of those ranges
- FID is split into training and validation subsets before optimization, and validation-set misfit is monitored to detect overfitting
- Frequency sweep uses the 1,000-point grid on the residual, not the full spectrum
- Output correlation with ground truth on overlapped simulated spectra or spiked urine reaches the ~0.9 level reported for SAND

## Limitations

- Tuned only for high-resolution 1D metabolomics spectra; other data types are not yet accommodated
- The MATLAB implementation is slow and not parallelizable; C/C++ replacement is planned
- Does not natively handle nonexponential signals, and multidimensional adaptation is only an aspirational goal
- Autophasing or manual phase adjustment is preferred though not always strictly required

## Evidence

- [methods] Residual-based peak-parameter initialization via 1,000-point frequency sweep: [evidence span withheld — anchor status: not_found]
- [methods] Perturbation bound for MCMC parameter updates: [evidence span withheld — anchor status: not_found]
- [methods] Per-bin peak-count stop criterion: [evidence span withheld — anchor status: not_found]
