---
name: matlab-nonlinear-least-squares-scripting
description: Use when you have relative-abundance time-course metabolomics data (concMatrix
  + timeVec) for a metabolic network and need to infer response factors / absolute
  concentrations without chemical standards via the kinetic equation (KE) approach.
when_to_use_negative:
- Data are not intracellular, single-compartment metabolomics samples from a directly
  connected metabolic network (MetaboPAC assumes all metabolites are connected via
  reactions and ignores inter-compartment transport in eukaryotes).
- High noise or substantial missingness (e.g. 10% values imputed) where mass balances
  are violated, since the optimizer's feasibility assessment depends on mass-balance
  consistency.
- Metabolites whose response factors are strongly nonlinear with concentration (ion
  suppression / limited linear dynamic range), since the optimization assumes constant
  response factors.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboPAC
  role: Computational framework whose KE-approach MATLAB scripts (solveRF_KEapproach_generic.m,
    calcFluxWithKinetics_generic.m) run the nonlinear optimization for response factors
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Language/environment in which the MetaboPAC_*.m optimization scripts and GenData
    data generators are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:8ea5a8ed506e5dce32942e481faeb6aaf2e15fa68fd419d77f63b5e08a1ccecc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# matlab-nonlinear-least-squares-scripting

## Summary

MATLAB-based optimization scripting in MetaboPAC that solves for response factors (and thus absolute metabolite concentrations) from relative abundance time-course GC-MS data by minimizing penalties grounded in mass balances and known kinetic equations.

## When to use

Use when you have relative-abundance time-course metabolomics data (concMatrix + timeVec) for a metabolic network and need to infer response factors / absolute concentrations without chemical standards via the kinetic equation (KE) approach.

## When NOT to use

- Data are not intracellular, single-compartment metabolomics samples from a directly connected metabolic network (MetaboPAC assumes all metabolites are connected via reactions and ignores inter-compartment transport in eukaryotes).
- High noise or substantial missingness (e.g. 10% values imputed) where mass balances are violated, since the optimizer's feasibility assessment depends on mass-balance consistency.
- Metabolites whose response factors are strongly nonlinear with concentration (ion suppression / limited linear dynamic range), since the optimization assumes constant response factors.

## Inputs

- generic_data.mat with relative abundance data (concMatrix) and measurement times (timeVec)
- modelInfo_generic stoichiometric matrix with known concentration and flux bounds
- Known kinetic equations hard-coded into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m
- Penalty functions defined in the penalty_functions folder
- trueRF: randomly generated true response factors (20 replicate sets per system) sampled uniform [1, 1000]

## Outputs

- Inferred response factors and inferred absolute concentrations per metabolite
- Accuracy as the fraction of inferred concentrations within log2 error tolerances (log2(1.1), log2(1.3))
- Median-aggregated results across repetitions [1:20] for each percKnownKinetics setting (e.g. saved as keapproach_noiseless_results.mat)

## How to apply

Prepare generic_data.mat containing the relative abundance matrix (concMatrix) and measurement times (timeVec), edit modelInfo_generic with the system's stoichiometric matrix and known concentration/flux bounds, and hard-code known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m. Run the MetaboPAC_*.m scripts over seeded repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]; the optimizer seeks the response-factor set (true values sampled from uniform [1, 1000]) that minimizes penalty functions defined in penalty_functions, evaluating fluxes through the kinetic equations. Aggregate inferred response factors and concentrations across the 20 seeded repetitions via the median to obtain per-model absolute concentrations. Judge success by log2 error tolerances: on noiseless data the KE approach should place 100% of response factors within log2(1.1) for synthetic models and 100% within log2(1.3) (with >90% within log2(1.1)) for the E. coli and S. cerevisiae biological models.

## Related tools

- **MetaboPAC** (Computational framework whose KE-approach MATLAB scripts (solveRF_KEapproach_generic.m, calcFluxWithKinetics_generic.m) run the nonlinear optimization for response factors) — gtStyLab/MetaboPAC
- **MATLAB** (Language/environment in which the MetaboPAC_*.m optimization scripts and GenData data generators are executed)

## Examples

```
addpath(genpath('.')); MetaboPAC_1; % run in MATLAB with repetitions [1:20] and percKnownKinetics [0 20 40 60 80 100]
```

## Evaluation signals

- On noiseless synthetic-model data, 100% of inferred response factors fall within log2(1.1) error of the true values.
- On noiseless biological-model data (E. coli, S. cerevisiae), 100% of inferred response factors fall within log2(1.3) and >90% within log2(1.1).
- Median aggregation over the 20 seeded repetitions [1:20] is applied before computing per-model absolute concentrations.
- The script grid is complete: percKnownKinetics values [0 20 40 60 80 100] are covered for each of the four models.
- Data generation matches the manuscript protocol: noiseless data from the first cell of GenData/driver_genDatasets*.m; noise-added data from the second cell with timepoints [50 15] and CoV [5 15].

## Limitations

- MetaboPAC assumes all metabolites are directly connected via reactions and only applies to intracellular, single-compartment samples; transport reactions across eukaryotic compartments are not considered.
- Response factors are assumed constant with concentration; linear dynamic range limits or ion suppression can violate this assumption, and allowing nonlinear response factors would be computationally challenging.
- Multiple response-factor sets can have similar or smaller penalty values than the true ones, indicating the penalties need refinement and solutions may not be unique.
- Inference accuracy is limited by the accuracy of the underlying flux-calculation approaches, and accuracy varies widely (40% to nearly 100%) with network, data quality, and error tolerance.
- Missing values (e.g. 10% of the dataset imputed) reduce the accuracy of inferred concentrations.

## Evidence

- [results] Noiseless KE accuracy thresholds: [evidence span withheld — anchor status: not_found]
