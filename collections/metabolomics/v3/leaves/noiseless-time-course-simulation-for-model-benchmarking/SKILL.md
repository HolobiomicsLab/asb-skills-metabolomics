---
name: noiseless-time-course-simulation-for-model-benchmarking
description: Use when establishing an upper-bound accuracy baseline for methods that
  infer absolute concentrations from relative abundance time-course data, before testing
  noisy or sparse regimes.
when_to_use_negative:
- When the goal is to assess robustness to measurement noise or sparse sampling —
  use MetaboPAC_*_noisy.m with timepoints [50 15] and CoV [5 15] instead
- When the dataset contains real experimental metabolomics measurements with missing
  values or instrument noise rather than simulated noiseless profiles
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboPAC
  role: Computational framework whose kinetic-equation (KE) approach is benchmarked
    on the noiseless simulated data
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Environment for running driver_genDatasets*.m to generate noiseless data and
    MetaboPAC_*.m to run inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:8d49e13fd1e103e128224f6e031cd8ab19189f8d8819048b97bb433b7fb29e67
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# noiseless-time-course-simulation-for-model-benchmarking

## Summary

Generate noiseless in silico time-course metabolomics data for benchmarking response-factor/concentration inference by running the first cell of GenData/driver_genDatasets*.m in MATLAB across the paper's two synthetic and two biological models, then evaluating inferred absolute concentrations against log2 error tolerances.

## When to use

Use when establishing an upper-bound accuracy baseline for methods that infer absolute concentrations from relative abundance time-course data, before testing noisy or sparse regimes.

## When NOT to use

- When the goal is to assess robustness to measurement noise or sparse sampling — use MetaboPAC_*_noisy.m with timepoints [50 15] and CoV [5 15] instead
- When the dataset contains real experimental metabolomics measurements with missing values or instrument noise rather than simulated noiseless profiles

## Inputs

- GenData/driver_genDatasets*.m MATLAB driver scripts (first cell for noiseless generation)
- Four in silico model definitions (two synthetic, two biological: E. coli and S. cerevisiae) with stoichiometric matrices and kinetic equations
- trueRF replicate sets (20 replicate sets per system of randomly generated true response factors, uniform between 1 and 1000)
- modelInfo files (stoichiometric matrix, concentration and flux bounds)

## Outputs

- Noiseless time-course relative abundance data (concMatrix) with measurement times (timeVec)
- Aggregated per-model inferred absolute concentrations and response factors (keapproach_noiseless_results.mat)
- Accuracy fractions within log2(1.1) and log2(1.3) error tolerances per model

## How to apply

Run the first cell of each GenData/driver_genDatasets*.m file in MATLAB to simulate noiseless time-course data (relative abundance concMatrix plus timeVec) for the four in silico systems, with true response factors sampled from a uniform distribution between 1 and 1000. Then execute MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100], aggregating inferred response factors across seeded repetitions via the median to obtain per-model absolute concentrations. Score accuracy as the fraction of inferred concentrations within log2 error tolerances: log2(1.1) for the synthetic models and log2(1.3) for the E. coli and S. cerevisiae models. Correct application reproduces the noiseless benchmark: 100% of response factors within log2(1.1) for synthetic models and 100% within log2(1.3) for biological models; save outputs to keapproach_noiseless_results.mat.

## Related tools

- **MetaboPAC** (Computational framework whose kinetic-equation (KE) approach is benchmarked on the noiseless simulated data) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Environment for running driver_genDatasets*.m to generate noiseless data and MetaboPAC_*.m to run inference)

## Examples

```
matlab -batch "run('GenData/driver_genDatasets*.m'); MetaboPAC_*.m % repetitions [1:20], percKnownKinetics [0 20 40 60 80 100]"
```

## Evaluation signals

- Synthetic models reach 100% of inferred concentrations within log2(1.1) error on noiseless data
- Biological models (E. coli, S. cerevisiae) reach 100% within log2(1.3) with >90% within log2(1.1)
- Results are stable across the 20 seeded repetitions [1:20] when aggregated via median
- True response factors were sampled uniform between 1 and 1000, matching MetaboPAC_*.m conditions (not the log-uniform MetaboPAC_*_log.m setting)
- Outputs saved as keapproach_noiseless_results.mat

## Limitations

- Noiseless results are an idealized upper bound; under lower sampling frequency and high noise MetaboPAC only infers concentrations significantly in 3 of 4 models, and accuracy ranges from 40% to nearly 100% depending on network and data quality
- The simulation assumes constant response factors with concentration, which may not hold when the linear dynamic range or ion suppression is violated
- Applicability limited to intracellular metabolomics in a single compartment; transport reactions between intracellular compartments are not modeled

## Evidence

- [results] On noiseless data, the kinetic equation approach predicted 100% of response factors within log2(1.1) error for the two synthetic models: [evidence span withheld — anchor status: not_found]
