---
name: stoichiometric-mass-balance-flux-solving
description: Use when time-course relative-abundance metabolomics data (concMatrix/timeVec)
  and a stoichiometric matrix are available but fluxomics measurements are not, and
  flux profiles v must be reconstructed for penalty-based optimization of response
  factors.
when_to_use_negative:
- Eukaryotic systems requiring transport reactions across intracellular compartments
  — MetaboPAC does not take these into consideration and applies only to intracellular
  metabolomics samples in a single compartment
- Single-timepoint data with no time-course structure — concentration derivatives
  cannot be computed as differences between timepoints
- Datasets with high noise levels that violate stoichiometric mass balances, complicating
  assessment of biological feasibility of response factors
edam_topics:
- http://edamontology.org/topic_3368
tools:
- name: MetaboPAC
  role: Computational framework whose flux-inference routine solves S·v = dx/dt to
    obtain the flux profiles needed for penalty calculation; run via MetaboPAC_generic.m
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment for loading generic_data.mat, editing modelInfo_generic,
    and running the MetaboPAC_* scripts
- name: calcFluxWithKinetics_generic.m
  role: Repository file edited to include known kinetic equations used to constrain
    the underdetermined flux-solving systems
  repo: gtStyLab/MetaboPAC
- name: solveRF_KEapproach_generic.m
  role: Repository file edited to include known kinetic equations for the response-factor
    identification approach that supplies denominators for derivative scaling
  repo: gtStyLab/MetaboPAC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:90480d296aeb656304e5d96a760b0b698711f13f85167919662adddd1238bf29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  - calcFluxWithKinetics_generic.m
  - solveRF_KEapproach_generic.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# stoichiometric-mass-balance-flux-solving

## Summary

Infers metabolic flux profiles from time-course relative-abundance GC-MS metabolomics data by computing smoothed metabolite concentration derivatives and solving the stoichiometric mass balance S·v = dx/dt, exactly for determined systems and via the Moore-Penrose pseudoinverse with known kinetic terms for underdetermined systems. These flux profiles feed MetaboPAC's penalty calculation for inferring absolute concentrations without chemical standards.

## When to use

Use when time-course relative-abundance metabolomics data (concMatrix/timeVec) and a stoichiometric matrix are available but fluxomics measurements are not, and flux profiles v must be reconstructed for penalty-based optimization of response factors.

## When NOT to use

- Eukaryotic systems requiring transport reactions across intracellular compartments — MetaboPAC does not take these into consideration and applies only to intracellular metabolomics samples in a single compartment
- Single-timepoint data with no time-course structure — concentration derivatives cannot be computed as differences between timepoints
- Datasets with high noise levels that violate stoichiometric mass balances, complicating assessment of biological feasibility of response factors

## Inputs

- generic_data.mat containing the relative abundance matrix (concMatrix) and measurement time vector (timeVec)
- modelInfo_generic: stoichiometric matrix of the system of interest plus known concentration and flux bounds
- Known kinetic equations hard-coded into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m
- Response factors (rate-of-change denominators) from the kinetic equations approach

## Outputs

- Per-flux flux profiles for each of the four models satisfying S·v = dx/dt
- Per-metabolite derivative matrices (rate of change of absolute concentration)
- flux_profiles.mat storing derivative and flux profiles for the four models

## How to apply

Load generic_data.mat in MATLAB to read the relative abundance matrix (concMatrix) and the measurement time vector (timeVec), and pull the stoichiometric matrix plus known concentration/flux bounds from modelInfo_generic. Smooth the relative-abundance time courses and compute each metabolite's rate of change as the difference in relative abundance between two timepoints divided by the time difference, then divide that rate by the corresponding response factor to obtain the rate of change of absolute concentration dx/dt. Hard-code any known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m, since MetaboPAC requires these where specified. For each of the four models, solve S·v = dx/dt exactly when the system is determined; when it is underdetermined, append known kinetic terms to convert it into a determined system and apply the Moore-Penrose pseudoinverse. Store the per-metabolite derivative matrices and per-flux flux profiles in flux_profiles.mat for downstream penalty computation.

## Related tools

- **MetaboPAC** (Computational framework whose flux-inference routine solves S·v = dx/dt to obtain the flux profiles needed for penalty calculation; run via MetaboPAC_generic.m) — gtStyLab/MetaboPAC
- **MATLAB** (Execution environment for loading generic_data.mat, editing modelInfo_generic, and running the MetaboPAC_* scripts)
- **calcFluxWithKinetics_generic.m** (Repository file edited to include known kinetic equations used to constrain the underdetermined flux-solving systems) — gtStyLab/MetaboPAC
- **solveRF_KEapproach_generic.m** (Repository file edited to include known kinetic equations for the response-factor identification approach that supplies denominators for derivative scaling) — gtStyLab/MetaboPAC

## Examples

```
load('generic_data.mat'); MetaboPAC_generic_noisy % concMatrix + timeVec loaded; kinetic equations pre-edited in calcFluxWithKinetics_generic.m
```

## Evaluation signals

- Computed fluxes must reproduce the smoothed derivative profiles through the stoichiometric mass balances for each of the four models
- For determined systems, the solution of S·v = dx/dt should be exact (zero residual), consistent with MetaboPAC's reported successful inference in 3 of 4 models studied
- For underdetermined models, verify that adding known kinetic terms converts the system to a determined one before applying the Moore-Penrose pseudoinverse
- flux_profiles.mat must contain per-metabolite derivative matrices and per-flux flux profiles for all four models
- Sensitivity checks should replicate MetaboPAC's design grid: percKnownKinetics values of [0 20 40 60 80 100], repetitions [1:20], timepoints [50 15], and CoV [5 15]

## Limitations

- MetaboPAC assumes all metabolites are directly connected via reactions and only applies to intracellular metabolomics samples in a single compartment
- Transport reactions across intracellular compartments in eukaryotic systems are not considered
- Flux calculation accuracy depends on existing or in-development approaches, which limits the accuracy of MetaboPAC's inferences
- High noise levels in the data can lead to violations of the mass balances and complicate assessment of the biological feasibility of response factors
- Response factors were assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption

## Evidence
