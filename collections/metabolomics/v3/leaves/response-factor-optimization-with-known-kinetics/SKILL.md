---
name: response-factor-optimization-with-known-kinetics
description: Use when metabolomics time-course data are relative abundances only,
  a stoichiometric model with known kinetic equations for some metabolites is available,
  and absolute concentrations must be inferred standard-free.
when_to_use_negative:
- The metabolomics sample spans multiple compartments or includes transport reactions
  — MetaboPAC does not consider transport across intracellular compartments in eukaryotic
  systems
- No kinetic equations or biological penalty information exist for any metabolite,
  leaving the kinetic-equation step with nothing to fix
- Response factors are strongly concentration-dependent (nonlinear ion suppression),
  since RFs are assumed constant with concentration
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: MetaboPAC
  role: Computational framework that combines the kinetic-equations approach and the
    optimization (heuristic penalty-function) approach to infer absolute concentrations
    from relative abundances
  repo: gtStyLab/MetaboPAC
- name: MetaboPAC_generic.m
  role: Generalized driver script in which known kinetic equations are hard-coded
    before running the optimization
  repo: gtStyLab/MetaboPAC
- name: solveRF_KEapproach_generic.m
  role: Solves response factors via the kinetic equations approach for metabolites
    with known kinetics before the remainder are optimized
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for MetaboPAC scripts, GenData simulation, and penalty-function
    optimization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:9bf978e0fe469f12c7cfc5fd32a4995c32bfa166de0c4f6cf3968fb9d53fd9bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  - MetaboPAC_generic.m
  - solveRF_KEapproach_generic.m
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Response-factor optimization with known kinetics

## Summary

A MetaboPAC strategy that fixes the response factors of metabolites whose kinetic equations are fully known (via the kinetic-equations approach) and then heuristically optimizes the remaining response factors against mass balances and biological penalty functions, improving absolute-concentration inference from relative-abundance GC-MS time-course data.

## When to use

Use when metabolomics time-course data are relative abundances only, a stoichiometric model with known kinetic equations for some metabolites is available, and absolute concentrations must be inferred standard-free.

## When NOT to use

- The metabolomics sample spans multiple compartments or includes transport reactions — MetaboPAC does not consider transport across intracellular compartments in eukaryotic systems
- No kinetic equations or biological penalty information exist for any metabolite, leaving the kinetic-equation step with nothing to fix
- Response factors are strongly concentration-dependent (nonlinear ion suppression), since RFs are assumed constant with concentration

## Inputs

- generic_data.mat containing relative-abundance time-course matrix (concMatrix) and timeVec
- Stoichiometric matrix and known concentration/flux bounds in modelInfo_generic
- Known kinetic equations hard-coded in calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m
- Penalty functions in the penalty_functions folder
- Randomly sampled true response factors from trueRF (20 replicate sets per system)

## Outputs

- Inferred absolute metabolite concentrations (response factors converting relative abundances)
- Per-condition accuracy metrics: fraction of inferred concentrations within log2(1.3) error, split by determined vs. underdetermined metabolites
- Penalty scores for candidate response-factor sets
- Figures generated from the plots folder

## How to apply

Load relative-abundance data (concMatrix) and measurement times (timeVec) into generic_data.mat, and supply the stoichiometric matrix plus known concentration/flux bounds in modelInfo_generic. Hard-code the known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m; the kinetic-equations approach first solves response factors for metabolites whose terms are at least partly known. For the remaining 'underdetermined' metabolites, MetaboPAC's optimization approach searches over randomly sampled response factors (true RFs were drawn uniform or log-uniform between 1 and 1000, 20 replicate sets per system) and scores candidate RF vectors with penalty functions placed in the penalty_functions folder, enforcing mass balances of the metabolic system. Sweep percKnownKinetics across [0 20 40 60 80 100] over repetitions [1:20] (e.g. with MetaboPAC_*.m, MetaboPAC_*_noisy.m at timepoints [50 15] and CoV [5 15], and MetaboPAC_*_log.m at percKnownKinetics = 100), then judge accuracy as the fraction of inferred absolute concentrations matching true concentrations within log2(1.3) error, split by determined vs. underdetermined metabolites.

## Related tools

- **MetaboPAC** (Computational framework that combines the kinetic-equations approach and the optimization (heuristic penalty-function) approach to infer absolute concentrations from relative abundances) — gtStyLab/MetaboPAC
- **MetaboPAC_generic.m** (Generalized driver script in which known kinetic equations are hard-coded before running the optimization) — gtStyLab/MetaboPAC
- **solveRF_KEapproach_generic.m** (Solves response factors via the kinetic equations approach for metabolites with known kinetics before the remainder are optimized) — gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for MetaboPAC scripts, GenData simulation, and penalty-function optimization)

## Examples

```
MATLAB: % edit modelInfo_generic, calcFluxWithKinetics_generic.m, solveRF_KEapproach_generic.m, penalty_functions; then: MetaboPAC_generic % with percKnownKinetics in [0 20 40 60 80 100], repetitions [1:20]
```

## Evaluation signals

- Run the full sweep (repetitions [1:20], percKnownKinetics [0 20 40 60 80 100]) and check the combined approach reproduces published noiseless benchmarks: E. coli ~77% and S. cerevisiae ~52% within log2(1.3) at 80% known kinetics
- Verify accuracy is reported separately for determined vs. underdetermined metabolites, with underdetermined metabolites ≈83% at 20% known kinetics
- Confirm candidate RF sets are scored by the penalty functions and mass balances; multiple RF sets with penalties similar to or smaller than the true RFs indicate the penalties need refinement
- Check RFs are drawn between 1 and 1000 (uniform for MetaboPAC_*.m, log-uniform for MetaboPAC_*_log.m) across 20 replicate sets per system

## Limitations

- MetaboPAC only applies to intracellular metabolomics samples in a single compartment and assumes all metabolites are directly connected via reactions; eukaryotic transport reactions are not modeled
- Response factors were assumed constant with concentration; nonlinear response (ion suppression, limited linear dynamic range) would violate this and is computationally challenging to allow
- Inference accuracy is limited by the accuracy of the flux-calculation approach and by missing values (e.g. 10% missingness with imputation reduces accuracy)
- Multiple RF sets can achieve similar or smaller penalties than the true ones, so penalty functions may need refinement
- Prediction accuracy varies widely (40% to nearly 100%) depending on network, data quality, and error tolerance

## Evidence

- [results] Accuracy benchmark for the combined approach at 80% known kinetics: [evidence span withheld — anchor status: not_found]
