---
name: response-factor-normalization-of-abundances
description: Use when time-course metabolomics data provide only relative abundances
  (no fluxomics measurements) but a stoichiometric matrix and time vector exist, and
  flux profiles are needed for MetaboPAC's penalty calculation.
when_to_use_negative:
- 'Eukaryotic multi-compartment systems: MetaboPAC does not take into consideration
  transport reactions across different intracellular compartments and only applies
  to intracellular metabolomics samples in a single compartment'
- Datasets with high noise or heavy missingness that violate mass balances — high
  noise levels complicate assessment of biological feasibility and 10% missingness
  with imputation reduces inferred-concentration accuracy
- Cases where response factors are known to be strongly concentration-dependent (nonlinear
  ionization/suppression), since MetaboPAC assumes constant response factors
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboPAC
  role: Framework whose penalty calculation consumes the inferred flux profiles; the
    generic version (MetaboPAC_generic.m) orchestrates the RF solve and flux computation
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for loading generic_data.mat, editing calcFluxWithKinetics_generic.m
    / solveRF_KEapproach_generic.m, and solving S·v = dx/dt via exact solve or Moore-Penrose
    pseudoinverse
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:210e9c7b1cfba2c1c697c29117a4578b66091642881af1f0f63ba845f6d43f2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# response-factor-normalization-of-abundances

## Summary

A MetaboPAC sub-procedure that infers metabolic flux profiles from time-course relative-abundance GC-MS data by converting smoothed rates of change of relative abundance into rates of change of absolute concentration via response factors, then solving the stoichiometric mass-balance system S·v = dx/dt.

## When to use

Use when time-course metabolomics data provide only relative abundances (no fluxomics measurements) but a stoichiometric matrix and time vector exist, and flux profiles are needed for MetaboPAC's penalty calculation.

## When NOT to use

- Eukaryotic multi-compartment systems: MetaboPAC does not take into consideration transport reactions across different intracellular compartments and only applies to intracellular metabolomics samples in a single compartment
- Datasets with high noise or heavy missingness that violate mass balances — high noise levels complicate assessment of biological feasibility and 10% missingness with imputation reduces inferred-concentration accuracy
- Cases where response factors are known to be strongly concentration-dependent (nonlinear ionization/suppression), since MetaboPAC assumes constant response factors

## Inputs

- generic_data.mat containing the relative abundance matrix (concMatrix) and measurement time vector (timeVec)
- modelInfo_generic: stoichiometric matrix of the system of interest plus any known concentration and flux bounds
- Known kinetic equations to hard-code into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m
- Putative response factors (e.g. sampled uniformly or log-uniformly between 1 and 1000 in trueRF replicate sets)

## Outputs

- Per-metabolite derivative matrices (dx/dt of absolute concentrations, derived from smoothed relative-abundance differences divided by response factors)
- Per-flux flux profiles for the four models solving S·v = dx/dt, stored in flux_profiles.mat

## How to apply

Load generic_data.mat in MATLAB, reading the relative abundance matrix (concMatrix) and the measurement time vector (timeVec), then pull the stoichiometric matrix plus any known concentration and flux bounds from modelInfo_generic. Hard-code any known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m as specified for the MetaboPAC_generic framework. Smooth each relative-abundance time course and compute the concentration derivative as the rate of change of relative abundance between two timepoints divided by the time difference, then divided by that metabolite's putative response factor (RF), yielding dx/dt for the absolute concentration. For each of the four models, solve S·v = dx/dt exactly when the system is determined, and for underdetermined systems apply the Moore-Penrose pseudoinverse using known kinetic terms to reduce the system to a determined one. Store the per-metabolite derivative matrices and per-flux flux profiles (e.g. in flux_profiles.mat) so they can feed the penalty functions in MetaboPAC's optimization over response-factor sets.

## Related tools

- **MetaboPAC** (Framework whose penalty calculation consumes the inferred flux profiles; the generic version (MetaboPAC_generic.m) orchestrates the RF solve and flux computation) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for loading generic_data.mat, editing calcFluxWithKinetics_generic.m / solveRF_KEapproach_generic.m, and solving S·v = dx/dt via exact solve or Moore-Penrose pseudoinverse)

## Examples

```
load('generic_data.mat'); % concMatrix, timeVec
edit modelInfo_generic % add stoichiometric matrix + bounds
run MetaboPAC_generic_noisy.m % solves S·v = dx/dt for flux profiles
```

## Evaluation signals

- Mass-balance check: confirm the computed fluxes reproduce the smoothed derivative profiles through the stoichiometric mass balances (S·v = dx/dt) for each of the four models
- Consistency with MetaboPAC's reported result that inference is significant in 3 of 4 models studied, even under lower sampling frequency (15 timepoints) and high noise (CoV 15)
- Flux solutions must respect the concentration and flux bounds supplied in modelInfo_generic
- Underdetermined models must be rendered determined via the Moore-Penrose pseudoinverse with known kinetic terms (percKnownKinetics e.g. 20% and above yields significant performance vs negative controls)
- Flux profiles are persisted per model (flux_profiles.mat) so the penalty functions can evaluate candidate response-factor sets consistently across repetitions 1:20

## Limitations

- Response factors were assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption
- Flux calculation accuracy depends on existing/in-development approaches and limits MetaboPAC inference accuracy
- High noise levels in the data can lead to violations of mass balances and complicate assessment of biological feasibility of response factors
- MetaboPAC assumes all metabolites are directly connected via reactions and only applies to intracellular metabolomics samples in a single compartment; transport reactions in eukaryotes are not considered
- Multiple sets of response factors can have similar or smaller penalty values than the true ones, suggesting the penalties need refinement

## Evidence
