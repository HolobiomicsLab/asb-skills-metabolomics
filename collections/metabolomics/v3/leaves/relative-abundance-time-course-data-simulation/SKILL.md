---
name: relative-abundance-time-course-data-simulation
description: Use when benchmarking an inference method like MetaboPAC that needs ground-truth
  absolute concentrations — simulate relative-abundance time-course GC-MS-style data
  from a known metabolic model with controlled noise, sampling frequency, and response-factor
  distributions.
when_to_use_negative:
- MetaboPAC's own scope is intracellular, single-compartment systems with all metabolites
  connected via reactions — eukaryotic transport reactions across intracellular compartments
  are not modeled, so such systems should not be simulated with these scripts
- Do not use these simulation scripts when the goal is processing real experimental
  GC-MS/LC-MS data rather than in silico benchmarking with known ground truth
- Assumes response factors constant with concentration; if the target system exhibits
  nonlinear response or ion suppression effects, simulated data will not capture that
  behavior
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboPAC
  role: Simulation scripts (driver_genDatasets*.m, MetaboPAC_* variants) ship in the
    MetaboPAC repo; generated datasets feed directly into MetaboPAC benchmarking runs
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for driver_genDatasets*.m data generation scripts and
    downstream MetaboPAC_* runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:ed4b2755b149fdac92e49e038f467e03d1765228e7d225062a0f26514e36e090
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Relative-abundance time-course data simulation

## Summary

Generates in silico noiseless and noise-added relative-abundance time-course metabolomics datasets from known metabolic systems with randomly sampled true response factors, providing ground truth to benchmark MetaboPAC's standard-free absolute-concentration inference. README-based: MATLAB scripts and generated .mat datasets. All steps, files, and parameters are grounded in the README and article methods; EDAM operation inferred conservatively from simulation context.

## When to use

Use when benchmarking an inference method like MetaboPAC that needs ground-truth absolute concentrations — simulate relative-abundance time-course GC-MS-style data from a known metabolic model with controlled noise, sampling frequency, and response-factor distributions.

## When NOT to use

- MetaboPAC's own scope is intracellular, single-compartment systems with all metabolites connected via reactions — eukaryotic transport reactions across intracellular compartments are not modeled, so such systems should not be simulated with these scripts
- Do not use these simulation scripts when the goal is processing real experimental GC-MS/LC-MS data rather than in silico benchmarking with known ground truth
- Assumes response factors constant with concentration; if the target system exhibits nonlinear response or ion suppression effects, simulated data will not capture that behavior

## Inputs

- MATLAB scripts driver_genDatasets*.m (GenData folder) for each in silico system
- modelInfo files specifying each metabolic system's stoichiometry (modelInfo folder)
- Randomly generated true response factor sets (trueRF folder; 20 replicate sets per system, uniform 1–1000 or log-uniform)

## Outputs

- Simulated noiseless relative-abundance time-course datasets (.mat) with true response factors known
- Noise-added datasets at multiple sampling frequencies (e.g. timepoints [50 15]) and CoV levels (e.g. [5 15])
- Log-uniform-response-factor noiseless datasets for the MetaboPAC_*_log.m variants
- Ground-truth absolute concentrations and response factors for accuracy evaluation within log2(1.3) error

## How to apply

Run the first cell of GenData/driver_genDatasets*.m in MATLAB to generate noiseless relative-abundance datasets, and the second cell to generate noise-added variants with controlled sampling frequencies (e.g. timepoints [50 15]) and coefficient-of-variation levels (e.g. CoV [5 15]). True response factors are randomly sampled for 20 replicate sets per system (uniform between 1 and 1000, or log-uniform between 1 and 1000 for the _log variants), so that relative abundances are derived from true concentrations multiplied by known response factors — allowing accuracy of inferred absolute concentrations to be scored within a log2(1.3) error range. Iterate over four in silico systems whose model information lives in the modelInfo folder; downstream MetaboPAC runs expect repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100]. Choose the sampling/noise grid deliberately: the paper stresses that lower sampling frequency and higher noise degrade inference, so the simulation grid should span the regimes you want to characterize.

## Related tools

- **MetaboPAC** (Simulation scripts (driver_genDatasets*.m, MetaboPAC_* variants) ship in the MetaboPAC repo; generated datasets feed directly into MetaboPAC benchmarking runs) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for driver_genDatasets*.m data generation scripts and downstream MetaboPAC_* runs)

## Examples

```
matlab -batch "run('GenData/driver_genDatasets_synthetic.m')" % then run MetaboPAC_*.m with percKnownKinetics [0 20 40 60 80 100]
```

## Evaluation signals

- Noiseless dataset generated from first cell of driver_genDatasets*.m must reproduce relative abundances consistent with known true concentrations and true response factors drawn from the trueRF sets
- Noise-added runs must sweep the documented grid: timepoints [50 15], CoV [5 15], repetitions [1:20], percKnownKinetics [0 20 40 60 80 100]
- Each system should have 20 replicate response-factor sets available for MetaboPAC to identify (matching trueRF folder structure)
- Downstream MetaboPAC accuracy on simulated data should be scored as fraction of inferred absolute concentrations within log2(1.3) of truth
- Log-variant datasets must use response factors drawn log-uniform between 1 and 1000, distinguishable from the uniform 1–1000 sets used in MetaboPAC_*.m

## Limitations

- Simulation assumes response factors constant with concentration — nonlinear response or ion suppression regimes are not captured
- Results depend on the four specific in silico systems in the modelInfo folder; extension to new networks requires editing modelInfo files
- High noise in the simulated data can violate mass balances and complicate downstream feasibility assessment (article discussion)
- MetaboPAC applied to simulated data cannot model eukaryotic intracellular transport reactions across compartments

## Evidence
