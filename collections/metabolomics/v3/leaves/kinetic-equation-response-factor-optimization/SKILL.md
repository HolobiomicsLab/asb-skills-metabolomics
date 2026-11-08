---
name: kinetic-equation-response-factor-optimization
description: Use when time-course relative abundance metabolomics data (concMatrix
  + timeVec) are available for a metabolic network whose kinetic equations are at
  least partially known, and absolute concentrations or response factors must be inferred
  without chemical standards.
when_to_use_negative:
- Data are from extracellular or multi-compartment eukaryotic samples — MetaboPAC
  only applies to intracellular metabolomics in a single compartment and does not
  model transport reactions across intracellular compartments
- The metabolic network has no known kinetic equations and no additional biological
  constraints can be supplied (percKnownKinetics = 0 performance degrades relative
  to negative controls)
- Concentrations span a range where response factors are non-linear (ion suppression
  / limited linear dynamic range), since response factors are assumed constant with
  concentration
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetaboPAC
  role: Computational framework implementing the kinetic equation (KE) approach that
    optimizes response factors against metabolic mass balances; run via MetaboPAC_*.m
    or MetaboPAC_generic.m
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Environment in which MetaboPAC scripts, data generation (driver_genDatasets*.m),
    and the response-factor optimization are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:c7eefd49dd646ef5f1734790aca582b56d9658ab859d97963f344e6d9872a160
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# kinetic-equation-response-factor-optimization

## Summary

Optimization-based inference of GC-MS response factors (and thus absolute metabolite concentrations) from relative abundance time-course data by exploiting metabolic mass balances and known kinetic equations, as implemented in MetaboPAC's kinetic equation (KE) approach.

## When to use

Use when time-course relative abundance metabolomics data (concMatrix + timeVec) are available for a metabolic network whose kinetic equations are at least partially known, and absolute concentrations or response factors must be inferred without chemical standards.

## When NOT to use

- Data are from extracellular or multi-compartment eukaryotic samples — MetaboPAC only applies to intracellular metabolomics in a single compartment and does not model transport reactions across intracellular compartments
- The metabolic network has no known kinetic equations and no additional biological constraints can be supplied (percKnownKinetics = 0 performance degrades relative to negative controls)
- Concentrations span a range where response factors are non-linear (ion suppression / limited linear dynamic range), since response factors are assumed constant with concentration

## Inputs

- generic_data.mat with relative abundance data (concMatrix) and measurement times (timeVec)
- Stoichiometric matrix and known concentration/flux bounds (modelInfo_generic)
- Known kinetic equations hard-coded into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m
- Penalty functions defined in the penalty_functions folder
- Noiseless time-course data generated via the first cell of GenData/driver_genDatasets*.m

## Outputs

- Inferred response factors per metabolite per repetition
- Inferred absolute concentrations aggregated across repetitions via the median (saved as keapproach_noiseless_results.mat)
- Accuracy metrics as the fraction of inferred concentrations within log2 error tolerances (log2(1.1), log2(1.3))

## How to apply

Prepare generic_data.mat (or model-specific data) containing the relative abundance matrix concMatrix and sampling times timeVec, and supply the stoichiometric matrix plus any known concentration/flux bounds in modelInfo. Hard-code the known kinetic equations into calcFluxWithKinetics and solveRF_KEapproach, then run MetaboPAC over seeded repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100] to vary how much kinetic information is assumed known a priori. The optimizer searches for the set of response factors that best satisfies the mass balances of the metabolic system, scoring candidate response-factor sets with penalty functions; aggregate across the 20 repetitions via the median to obtain per-model inferred absolute concentrations. Judge success by the fraction of inferred concentrations within log2 error tolerances: on noiseless data, 100% of response factors should fall within log2(1.1) for synthetic models and within log2(1.3) (>90% within log2(1.1)) for the E. coli and S. cerevisiae biological models.

## Related tools

- **MetaboPAC** (Computational framework implementing the kinetic equation (KE) approach that optimizes response factors against metabolic mass balances; run via MetaboPAC_*.m or MetaboPAC_generic.m) — gtStyLab/MetaboPAC
- **MATLAB** (Environment in which MetaboPAC scripts, data generation (driver_genDatasets*.m), and the response-factor optimization are executed)

## Examples

```
MATLAB: run MetaboPAC_*.m scripts with repetitions = 1:20 and percKnownKinetics = [0 20 40 60 80 100] after generating data via the first cell of GenData/driver_genDatasets*.m
```

## Evaluation signals

- On noiseless synthetic model data, 100% of predicted response factors fall within log2(1.1) error of the true values
- On noiseless biological model data (E. coli, S. cerevisiae), 100% of inferred concentrations fall within log2(1.3) and >90% within log2(1.1)
- Median aggregation over the 20 seeded repetitions ([1:20]) yields per-model concentrations consistent across runs
- Performance at percKnownKinetics = 20 should exceed negative control approaches, matching the reported finding that MetaboPAC performs significantly better when 20% of kinetic terms are known
- Output file keapproach_noiseless_results.mat is produced with per-model inferred response factors and concentrations

## Limitations

- Response factors are assumed constant with concentration; linear dynamic range limits or ion suppression can violate this assumption, and allowing non-linear response factors would be computationally challenging
- High noise levels can violate mass balances and complicate assessment of biological feasibility of candidate response factors
- Multiple sets of response factors can have similar or smaller penalty values than the true ones, suggesting the penalties need refinement
- Missing values (~10% with imputation) reduce accuracy of inferred concentrations
- Inference accuracy is bounded by the accuracy of the underlying flux-calculation approaches

## Evidence

- [results] 100% of response factors within log2(1.1) error for the two synthetic models: [evidence span withheld — anchor status: not_found]
