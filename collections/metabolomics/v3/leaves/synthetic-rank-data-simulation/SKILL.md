---
name: synthetic-rank-data-simulation
description: Use when you need to benchmark or validate a MaRR-type reproducibility
  estimator against a ground-truth simulation, e.g., before applying it to MS-Metabolomics
  replicate data.
when_to_use_negative:
- Input is real MS-Metabolomics replicate data with unknown ground truth — use the
  MaRR procedure directly (marr package) rather than a simulation harness
- You only need reproducibility summaries (features per sample pair, sample pairs
  per feature) from the Marr() function rather than estimator validation against known
  pi_1
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Language in which the simulation scripts (SIM_B1.R etc.), MaRR functions,
    and plotting scripts are written
  repo: http://bioconductor.org/packages/marr
- name: marr
  role: Maximum Rank Reproducibility estimator package whose functions are exercised
    on simulated rank-pair data
  repo: http://bioconductor.org/packages/marr
- name: devtools
  role: Used to install MSPrep and development dependencies (install_github) in the
    simulation environment
- name: xtable
  role: Generates LaTeX tables from simulation/real-data R objects in the evaluation
    repository
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:7a9254c98fd6b5f777339dd03bb83cad89624e23a0f09bc7836664297e3e9f20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - marr
  - xtable
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# synthetic-rank-data-simulation

## Summary

Generates synthetic rank-pair datasets under a known reproducible-proportion setting (e.g., pi_1 = 0.35) to validate the MaRR maximum-rank reproducibility estimator, bias, and mFDR calibration in R.

## When to use

Use when you need to benchmark or validate a MaRR-type reproducibility estimator against a ground-truth simulation, e.g., before applying it to MS-Metabolomics replicate data.

## When NOT to use

- Input is real MS-Metabolomics replicate data with unknown ground truth — use the MaRR procedure directly (marr package) rather than a simulation harness
- You only need reproducibility summaries (features per sample pair, sample pairs per feature) from the Marr() function rather than estimator validation against known pi_1

## Inputs

- Simulation design parameters (M metabolites, true reproducible proportion pi_1, e.g., 0.35, nominal FDR alpha)
- Sourced MaRR functions from source_MaRR.R (folder 'src' of Ghoshlab/marr_paper_evaluations)
- R environment (R >= 4.0) with devtools for package installation

## Outputs

- Rdata object of simulation results (e.g., B1_sim_results.Rdata) containing estimated reproducible proportion, mFDR curve, and rejection threshold N-hat
- Validation histogram/plot in SIMPLOT_B style
- Bias and empirical FDR diagnostics versus known ground-truth pi_1

## How to apply

Write R simulation scripts mirroring the paper's SIM_B1.R -> B1_sim_results.Rdata convention: simulate a synthetic rank-pair dataset under the ideal setting with a known true reproducible proportion (e.g., pi_1 = 0.35), source the MaRR functions (source_MaRR.R) from the 'src' folder of the marr_paper_evaluations repository, and run the estimator (maximal rank statistic, empirical survival function of ranks, MSE(lambda)-based estimator with tuning parameter lambda = 0.9). Add the mFDR estimator mFDR-hat(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) and compute the rejection threshold N-hat at nominal FDR level alpha. Save the estimated reproducible proportion, mFDR curve, and rejection threshold as an .Rdata object, then generate a SIMPLOT_B-style validation plot. Validate by checking that bias (estimated minus true pi_1) is close to zero and empirical FDR at N-hat is controlled at the nominal alpha level, consistent with the paper's findings that bias values are close to zero and FDR is effectively controlled.

## Related tools

- **R** (Language in which the simulation scripts (SIM_B1.R etc.), MaRR functions, and plotting scripts are written) — http://bioconductor.org/packages/marr
- **marr** (Maximum Rank Reproducibility estimator package whose functions are exercised on simulated rank-pair data) — http://bioconductor.org/packages/marr
- **devtools** (Used to install MSPrep and development dependencies (install_github) in the simulation environment)
- **xtable** (Generates LaTeX tables from simulation/real-data R objects in the evaluation repository)

## Examples

```
Rscript SIM_B1.R  # sources src/source_MaRR.R, simulates rank pairs with pi_1=0.35, saves B1_sim_results.Rdata
```

## Evaluation signals

- Bias (estimated minus true reproducible proportion) is close to zero for the simulated pi_1
- Empirical FDR at the rejection threshold N-hat is controlled at the nominal level alpha
- Estimated reproducible proportion matches the simulation ground truth (e.g., pi_1 = 0.35) within expected Monte Carlo error
- mFDR curve follows the analytic form mFDR-hat(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) over the rejection region (0, l)
- Output object naming follows the SIM_B1.R -> B1_sim_results.Rdata convention and SIMPLOT_B rendering completes without error

## Limitations

- True signals may fall in undetectable regions due to overlap between reproducible and irreproducible components, so separation-limited simulations underestimate difficulty
- MaRR tends to have lower power for detecting reproducible metabolites at smaller values of the proportion of reproducible metabolites
- FDR control was validated for gradual reduction in correlation between replicate pairs for less highly ranked signals; other dependence structures may behave differently

## Evidence

- [methods] Run simulation scripts to generate R objects: "The scripts 'SIM_B1.R',.. , 'SIM_C1.R',..., 'SIM_D4.R' can generate the corresponding R objects 'B1_sim_results.Rdata',..,'C1_sim_results.Rdata',...,'D4_sim_results.Rdata'"
- [methods] Source MaRR functions: "The MaRR functions ('source_MaRR.R') can be sourced from the folder 'src'"
- [abstract] Bias close to zero: "Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero"
- [abstract] FDR control under gradual correlation reduction: "the MaRR procedure effectively controls the False Discovery Rate (FDR) when there was a gradual reduction in correlation between replicate pairs for less highly ranked signals"
- [abstract] Power limitation at small pi_1: "the MaRR procedure tends to have high power for detecting reproducible metabolites in most situations except for smaller values of proportion of reproducible metabolites"
- [other] Simulation plot scripts: "3 R scripts ('SIMPLOT_A45.R', 'SIMPLOT_A99.R' and 'SIMPLOT_B.R') that generate the plots for 3 simulation designs"
