---
name: absolute-concentration-inference-from-relative-abundance
description: Use when you have relative abundance time-course metabolomics data (concMatrix
  + timeVec) from a single-compartment intracellular metabolic system, and you need
  standard-free absolute concentrations by leveraging mass balances and known kinetics.
when_to_use_negative:
- Data are extracellular or multi-compartment (e.g. eukaryotic systems with transport
  reactions across compartments) — MetaboPAC only applies to intracellular samples
  in a single compartment
- Response factors vary with concentration (violated linear dynamic range / ion suppression),
  since MetaboPAC assumes constant response factors
- The data have substantial missingness — 10% missing values with imputation already
  reduced inferred-concentration accuracy
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboPAC
  role: Computational framework that infers response factors and absolute concentrations
    from relative abundance time-course GC-MS data using kinetic equations and biological-heuristic
    penalties
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Runtime environment for the MetaboPAC_*.m scripts, GenData/driver_genDatasets*.m
    data generators, and result aggregation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:3ed6c5ca941e4505df6f805db80e84d4729ce07d83eb5243adf6c9372793c66e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# absolute-concentration-inference-from-relative-abundance

## Summary

MetaboPAC infers absolute metabolite concentrations from relative abundance time-course GC-MS metabolomics data by fitting response factors using metabolic mass balances, either via a kinetic-equation (KE) approach or biological-heuristic penalties, without requiring chemical standards.

## When to use

Use when you have relative abundance time-course metabolomics data (concMatrix + timeVec) from a single-compartment intracellular metabolic system, and you need standard-free absolute concentrations by leveraging mass balances and known kinetics.

## When NOT to use

- Data are extracellular or multi-compartment (e.g. eukaryotic systems with transport reactions across compartments) — MetaboPAC only applies to intracellular samples in a single compartment
- Response factors vary with concentration (violated linear dynamic range / ion suppression), since MetaboPAC assumes constant response factors
- The data have substantial missingness — 10% missing values with imputation already reduced inferred-concentration accuracy

## Inputs

- generic_data.mat with relative abundance data (concMatrix) and measurement times (timeVec)
- Stoichiometric matrix of the metabolic system plus known concentration and flux bounds (modelInfo_generic)
- Known kinetic equations for reactions (hard-coded in calcFluxWithKinetics_generic.m / solveRF_KEapproach_generic.m)
- Penalty functions in penalty_functions folder

## Outputs

- Inferred response factors per metabolite
- Inferred absolute concentrations per metabolite (per-model, median-aggregated over repetitions)
- Accuracy as fraction of inferred concentrations within log2 error tolerance (e.g. keapproach_noiseless_results.mat)

## How to apply

Prepare generic_data.mat containing the relative abundance matrix (concMatrix) and measurement times (timeVec), then edit modelInfo_generic with the stoichiometric matrix and any known concentration/flux bounds, and hard-code known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m. Run MetaboPAC over seeded repetitions [1:20] with percKnownKinetics values of [0 20 40 60 80 100]; the KE approach solves for response factors so that mass balances hold, using penalty functions in the penalty_functions folder to score biological feasibility. Aggregate inferred response factors and concentrations across the 20 seeded repetitions via the median to obtain per-metabolite absolute concentrations. Evaluate accuracy as the fraction of inferred concentrations within log2 error tolerances — log2(1.1) for synthetic models and log2(1.3) for biological (E. coli, S. cerevisiae) models; noiseless KE inference should reach 100% within these tolerances.

## Related tools

- **MetaboPAC** (Computational framework that infers response factors and absolute concentrations from relative abundance time-course GC-MS data using kinetic equations and biological-heuristic penalties) — gtStyLab/MetaboPAC
- **MATLAB** (Runtime environment for the MetaboPAC_*.m scripts, GenData/driver_genDatasets*.m data generators, and result aggregation)

## Examples

```
matlab -batch "run('GenData/driver_genDatasets_model1.m'); MetaboPAC_model1" % repetitions [1:20], percKnownKinetics [0 20 40 60 80 100]
```

## Evaluation signals

- On noiseless synthetic models, 100% of inferred response factors fall within log2(1.1) error
- On noiseless biological models (E. coli, S. cerevisiae), 100% within log2(1.3) and >90% within log2(1.1)
- Median aggregation over the 20 seeded repetitions ([1:20]) yields stable per-model absolute concentrations
- MetaboPAC outperforms negative control approaches when at least 20% of kinetic terms are known a priori
- Under low sampling frequency (15 timepoints) and high noise (CoV 15), significant concentration inference is still achieved in 3 of 4 models

## Limitations

- Only applicable to intracellular metabolomics samples in a single compartment; transport reactions across eukaryotic compartments are not modeled
- Response factors are assumed constant with concentration; nonlinear response (linear dynamic range limits, ion suppression) violates this and would be computationally challenging to allow
- Inference accuracy is limited by the accuracy of the underlying flux calculation approaches
- High noise can violate mass balances and complicate assessment of biological feasibility; 10% missing values with imputation reduces accuracy
- Multiple response-factor sets had similar or smaller penalty values than the true ones, indicating penalties need refinement; accuracy varies from 40% to nearly 100% depending on network, data quality, and tolerance

## Evidence

- [abstract] KE approach infers response factors: "we developed and characterized MetaboPAC, a computational framework that uses two approaches—one based on kinetic equations and another using biological heuristics"
- [readme] Run parameters for repetitions and percKnownKinetics: "Run MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]."
- [readme] generic_data.mat input format: "generic_data.mat should include relative abundance data (named concMatrix) and information about when each datapoint was measured (timeVec)."
- [discussion] Scope limitation to single compartment: "it can only be used for intracellular metabolomics samples in a single compartment"
- [discussion] Constant response factor assumption: "the response factors were assumed to be constant with concentration, but this assumption will not always hold"
