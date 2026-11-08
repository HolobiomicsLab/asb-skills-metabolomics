---
name: stoichiometric-mass-balance-constraint-modeling
description: Use when you have relative-abundance (not absolute) time-course metabolomics
  data for an intracellular metabolic network with a known stoichiometric matrix and
  need to infer response factors / absolute concentrations without chemical standards.
when_to_use_negative:
- When metabolites span multiple intracellular compartments (e.g. eukaryotic transport
  reactions) — MetaboPAC only applies to intracellular metabolomics samples in a single
  compartment and does not consider transport reactions across intracellular compartments.
- When response factors vary with concentration — the method assumes response factors
  are constant with concentration, which may not hold outside the linear dynamic range
  or under ion suppression.
- When the dataset is missing many values — missing values (e.g. 10% missingness with
  imputation) reduce the accuracy of inferred concentrations.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboPAC
  role: Computational framework that uses mass balances of cellular metabolism to
    score candidate response-factor sets; run via MetaboPAC_*.m, MetaboPAC_*_noisy.m,
    MetaboPAC_*_log.m, or MetaboPAC_generic.m
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for MetaboPAC scripts, data generation drivers, and
    penalty-function evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:511eeae01baadbb7b32704ac6095e450aaa68a01051e9b9c9efc3caeaf523ae6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# stoichiometric-mass-balance-constraint-modeling

## Summary

Uses stoichiometric mass balances of metabolic reaction networks, encoded in a stoichiometric matrix, to constrain and score candidate response-factor sets so that simulated concentrations from relative-abundance time-course GC-MS data remain biologically feasible.

## When to use

Use when you have relative-abundance (not absolute) time-course metabolomics data for an intracellular metabolic network with a known stoichiometric matrix and need to infer response factors / absolute concentrations without chemical standards.

## When NOT to use

- When metabolites span multiple intracellular compartments (e.g. eukaryotic transport reactions) — MetaboPAC only applies to intracellular metabolomics samples in a single compartment and does not consider transport reactions across intracellular compartments.
- When response factors vary with concentration — the method assumes response factors are constant with concentration, which may not hold outside the linear dynamic range or under ion suppression.
- When the dataset is missing many values — missing values (e.g. 10% missingness with imputation) reduce the accuracy of inferred concentrations.

## Inputs

- Relative abundance time-course GC-MS data (concMatrix) with measurement times (timeVec) in generic_data.mat
- Stoichiometric matrix of the metabolic system plus known concentration and flux bounds (modelInfo_*.m)
- Known kinetic equations (hard-coded into calcFluxWithKinetics_*.m and solveRF_KEapproach_*.m)
- Candidate response-factor sets sampled uniformly or log-uniformly between 1 and 1000 (trueRF folder)
- Penalty functions for the heuristic optimization approach (penalty_functions folder)

## Outputs

- Inferred response factors for each metabolite
- Simulated absolute metabolite concentrations over the time course
- Accuracy metrics: fraction of inferred absolute concentrations within log2(1.3) error of true concentrations, separated by determined vs underdetermined metabolites

## How to apply

Build or load the stoichiometric matrix of the system in modelInfo (per-system .m files, or modelInfo_generic), together with any known concentration and flux bounds; MetaboPAC assumes all metabolites are directly connected via reactions in a single compartment. For each candidate set of response factors (drawn from a uniform or log-uniform distribution between 1 and 1000, 20 replicate sets per system), convert measured relative abundances (concMatrix in generic_data.mat with timeVec) into simulated absolute concentrations and check whether the resulting fluxes satisfy the mass balances — in the kinetic-equation approach fluxes come from known kinetic equations via calcFluxWithKinetics*.m, while the heuristic approach scores candidates with penalty functions from the penalty_functions folder. Run MetaboPAC_*.m over repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100], letting the kinetic-equation step fix response factors of metabolites with known kinetics before heuristic optimization of the remainder. Judge feasibility by the fraction of inferred absolute concentrations matching true concentrations within a log2(1.3) error range, reported separately for determined and underdetermined metabolites.

## Related tools

- **MetaboPAC** (Computational framework that uses mass balances of cellular metabolism to score candidate response-factor sets; run via MetaboPAC_*.m, MetaboPAC_*_noisy.m, MetaboPAC_*_log.m, or MetaboPAC_generic.m) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for MetaboPAC scripts, data generation drivers, and penalty-function evaluation)

## Examples

```
Run MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]
```

## Evaluation signals

- Inferred absolute concentrations for determined metabolites should match true concentrations within the log2(1.3) error range; on noiseless data at 80% known kinetic terms the combined approach reaches ~77% accuracy for E. coli and ~52% for S. cerevisiae.
- At 20% known kinetic terms, determined metabolites should reach ≈50% accuracy and underdetermined metabolites ≈83% within the log2(1.3) error range.
- MetaboPAC results should outperform negative control approaches when 20% of kinetic terms are known a priori.
- The synthetic model with 80% known kinetic terms should exceed 90% accuracy on noiseless data.
- Multiple candidate response-factor sets should NOT achieve smaller penalty values than the true set — if they do, penalties need refinement.

## Limitations

- Applies only to intracellular metabolomics samples in a single compartment; transport reactions across intracellular compartments in eukaryotic systems are not considered.
- Response factors are assumed constant with concentration, an assumption that will not always hold (linear dynamic range, ion suppression).
- Flux calculation accuracy depends on existing/in-development kinetic approaches and thus limits MetaboPAC inference accuracy.
- High noise levels can violate mass balances and complicate assessment of biological feasibility of candidate response factors.
- Missing values (even 10% with imputation) reduce the accuracy of inferred concentrations.
- Multiple sets of response factors can have similar or smaller penalty values than the true ones, indicating the penalties need refinement.
- Prediction accuracy varies widely (40% to nearly 100%) depending on the metabolic network, data quality, and error tolerance.

## Evidence

- [results] combined approach accuracy: [evidence span withheld — anchor status: not_found]
