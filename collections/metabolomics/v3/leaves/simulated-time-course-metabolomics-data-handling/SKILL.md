---
name: simulated-time-course-metabolomics-data-handling
description: Use when benchmarking inference of absolute concentrations from relative
  abundance time-course GC-MS data and ground-truth response factors are needed that
  cannot be measured experimentally.
when_to_use_negative:
- The input is real experimental GC-MS metabolomics data with measured (rather than
  simulated) response factors—use the generic MetaboPAC workflow with generic_data.mat
  instead
- The system of interest is a eukaryotic, multi-compartment model with transport reactions,
  which MetaboPAC's simulated data generation does not represent
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetaboPAC
  role: Computational framework whose benchmarking consumes the simulated time-course
    datasets; simulation scripts live in its GenData, modelInfo, and trueRF folders
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for the driver_genDatasets*.m scripts that generate
    the noiseless and noise-added simulated datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:bc031baa5989dd910af8f09389fbf8f958eed4c6fdaa0732b3ff8527ee2d698a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# simulated-time-course-metabolomics-data-handling

## Summary

Generates and handles simulated relative-abundance time-course metabolomics datasets for the four metabolic models in MetaboPAC, using MATLAB driver scripts to create noiseless and noise-added data with specified sampling frequencies, coefficients of variation, and randomly sampled true response factors.

## When to use

Use when benchmarking inference of absolute concentrations from relative abundance time-course GC-MS data and ground-truth response factors are needed that cannot be measured experimentally.

## When NOT to use

- The input is real experimental GC-MS metabolomics data with measured (rather than simulated) response factors—use the generic MetaboPAC workflow with generic_data.mat instead
- The system of interest is a eukaryotic, multi-compartment model with transport reactions, which MetaboPAC's simulated data generation does not represent

## Inputs

- GenData/driver_genDatasets*.m MATLAB driver scripts for the four metabolic models
- modelInfo files with stoichiometric matrices and concentration/flux bounds per system
- Randomly generated true response factor sets from trueRF folder (20 replicate sets per system)
- Simulation parameters: percKnownKinetics values [0 20 40 60 80 100], timepoints [50 15], CoV [5 15]

## Outputs

- Noiseless relative-abundance time-course datasets (concMatrix with associated timeVec)
- Noise-added time-course datasets at varying sampling frequencies and noise levels
- Per-model simulated datasets supporting 20 repetitions for benchmarking inferred vs. true response factors

## How to apply

Generate in silico time-course data by running the first cell of the GenData/driver_genDatasets*.m MATLAB scripts to produce noiseless relative-abundance datasets for each of the four metabolic models. For noise robustness testing, run the second cell of the same driver scripts to generate noise-added data at sampling frequencies of [50 15] timepoints and coefficients of variation CoV [5 15], repeating across repetitions [1:20]. True response factors are randomly sampled per the experimental design—either uniformly (MetaboPAC_*.m scenario) or log-uniformly between 1 and 1000 (MetaboPAC_*_log.m scenario)—and stored in the trueRF folder as 20 replicate sets per system. Support each dataset with the stoichiometric matrices and concentration/flux bounds held in modelInfo, and consistent time vectors, so downstream MetaboPAC runs at percKnownKinetics values [0 20 40 60 80 100] can compare inferred response factors against known ground truth.

## Related tools

- **MetaboPAC** (Computational framework whose benchmarking consumes the simulated time-course datasets; simulation scripts live in its GenData, modelInfo, and trueRF folders) — gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for the driver_genDatasets*.m scripts that generate the noiseless and noise-added simulated datasets)

## Examples

```
MATLAB: run("GenData/driver_genDatasets_model1.m") first cell for noiseless data, then MetaboPAC_1.m with percKnownKinetics = [0 20 40 60 80 100]
```

## Evaluation signals

- Each of the four metabolic models yields its own simulated dataset, and each system has 20 replicate trueRF sets from which MetaboPAC tries to identify the true response factors
- Noise-added datasets are generated at exactly the manuscript's parameter grid: timepoints [50 15] and CoV [5 15]
- Noiseless datasets reproduce the setting where the heuristic optimization approach should outperform negative controls when at least 20% of kinetic terms are known a priori (Figure S5-S6 trend)
- concMatrix and timeVec structures are correctly named and shaped so MetaboPAC_*.m can consume them directly
- Uniformly vs. log-uniformly sampled true response factors fall in the range 1 to 1000 as specified for the MetaboPAC_*.m and MetaboPAC_*_log.m scenarios

## Limitations

- Simulated data assume response factors are constant with concentration; linear dynamic range or ion suppression effects are not modeled
- Simulated systems represent single-compartment intracellular metabolism with all metabolites directly connected via reactions, excluding eukaryotic transport reactions
- Noise-added data with high noise levels can violate mass balances, complicating downstream feasibility assessment

## Evidence
