---
name: interior-point-local-refinement
description: Decision support for tuning or implementing SAND's per-bin decomposition
  when sampled MCMC peak parameters need periodic deterministic polishing before emitting
  a peak table.
when_to_use_negative:
- Spectra are multidimensional or nonuniformly sampled 1D datasets outside SAND's
  current 1D time-domain scope
- Input signals are nonexponential or high-resolution metabolomics spectra requiring
  the exponential+Gaussian second-round optimization rather than the base hybrid loop
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Hosts the hybrid MCMC + interior-point optimization loop that performs the
    periodic local refinement of peak parameters
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation language of the current SAND hybrid optimizer containing the
    interior-point refinement step
- name: NMRbox
  role: Cloud environment where SAND (and its interior-point refinement loop) is deployed
    and tested before merge
  repo: https://nmrbox.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:62f1709e130b37853a8e95c3f20f079648251c6a8069605166864c6b50a07aba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# interior-point-local-refinement

## Summary

Periodic local optimization step inside SAND's hybrid MCMC peak-decomposition loop, in which an interior-point method refines the sampled peak parameters (amplitude, frequency, decay, phase) of a binned 1D NMR FID at intervals during up to 2,000 iterations.

## When to use

Decision support for tuning or implementing SAND's per-bin decomposition when sampled MCMC peak parameters need periodic deterministic polishing before emitting a peak table.

## When NOT to use

- Spectra are multidimensional or nonuniformly sampled 1D datasets outside SAND's current 1D time-domain scope
- Input signals are nonexponential or high-resolution metabolomics spectra requiring the exponential+Gaussian second-round optimization rather than the base hybrid loop

## Inputs

- Binned 1D NMR time-domain (FID) data with random training/validation split
- Initial peak parameters per bin from 1,000-point residual-based frequency sweep
- Predefined parameter ranges for amplitude, frequency, decay, and phase

## Outputs

- Locally refined peak parameter set per bin (amplitude, frequency, decay, phase)
- Converged peak_table.csv entries constrained to ≤7 peaks per bin

## How to apply

Within SAND's iterative per-bin loop, run the hybrid optimizer in which the MCMC (Metropolis) step randomly selects one parameter, perturbs it, and bounds each perturbation to 10% of the predefined parameter range at synthetic temperature 80 with subsampling in both time and frequency domains. Apply the interior-point local refinement at regular intervals — 100 iterations — of the default 2,000-iteration optimization, using it to converge the sampled parameters toward a local optimum of the time-domain model. Treat the refinement as complementary to, not a replacement for, the stochastic global search: MCMC explores parameter space while interior-point tightening drives the current peak set to best fit the training subset of the time-domain data. Continue iterating until one of the stop criteria is met, including the maximum of 7 peaks per bin. Emit the per-peak table of amplitude, frequency, decay, and phase for that bin.

## Related tools

- **SAND** (Hosts the hybrid MCMC + interior-point optimization loop that performs the periodic local refinement of peak parameters) — https://github.com/edisonomics/SAND
- **MATLAB** (Implementation language of the current SAND hybrid optimizer containing the interior-point refinement step)
- **NMRbox** (Cloud environment where SAND (and its interior-point refinement loop) is deployed and tested before merge) — https://nmrbox.org

## Evaluation signals

- peak_table.csv exists and contains amplitude, frequency, decay, and phase fields
- Peak list per bin is nonempty and consistent with the ≤7-peak-per-bin constraint
- Refinement iterations do not exceed 2,000 (default) with interior-point polishing at 100-iteration intervals
- Parameter perturbations remain within 10% of the predefined parameter range
- Correlation with ground truth on simulated overlapped datasets or spiked urine is around 0.9

## Limitations

- Current SAND is a slow MATLAB implementation not yet parallelizable in C/C++
- Tuned only for high-resolution metabolomics spectra; other cases not yet accommodated
- Does not yet handle nonexponential signals natively or multidimensional applications
- Autophasing or manual phase adjustment is preferred but not always needed — reliability of unphased input is unverified

## Evidence
