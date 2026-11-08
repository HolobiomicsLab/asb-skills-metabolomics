---
name: kinetic-equation-integration-into-flux-calculation
description: Use when running MetaboPAC's kinetic-equations approach on time-course
  relative-abundance (concMatrix/timeVec) data and a subset of kinetic terms for the
  reaction network are known a priori, so fluxes needed for penalty calculation must
  be inferred from metabolite rates of change.
when_to_use_negative:
- The data are not intracellular metabolomics samples in a single compartment — MetaboPAC
  does not consider transport reactions across intracellular compartments in eukaryotic
  systems
- No kinetic terms are known and the underdetermined systems cannot be constrained
  — the pseudoinverse solution requires known kinetic terms
- The goal is flux analysis on measured fluxomics data rather than inferring fluxes
  from time-course relative-abundance data, since MetaboPAC does not assume fluxomics
  data are available
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaboPAC
  role: Framework whose kinetic-equations approach receives the hard-coded kinetic
    equations and produces the flux profiles used in penalty calculation
  repo: https://github.com/gtStyLab/MetaboPAC
- name: calcFluxWithKinetics_generic.m
  role: File edited to include known kinetic equations for flux calculation
  repo: https://github.com/gtStyLab/MetaboPAC
- name: solveRF_KEapproach_generic.m
  role: File edited to include known kinetic equations when identifying response factors
    via the kinetic equations approach
  repo: https://github.com/gtStyLab/MetaboPAC
- name: MATLAB
  role: Execution environment in which generic_data.mat is loaded, S·v = dx/dt is
    solved, and flux_profiles.mat is written
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:48fde7227cadbeaa7f6c905805c7b45280363d562b1caeedd754d7febad37f77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  - calcFluxWithKinetics_generic.m
  - solveRF_KEapproach_generic.m
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kinetic-equation-integration-into-flux-calculation

## Summary

Integrates hard-coded known kinetic rate equations into MetaboPAC's flux-calculation routines (calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m) to convert smoothed concentration-derivative data into reaction rate profiles via the stoichiometric mass balance S·v = dx/dt, solving determined systems exactly and underdetermined systems via the Moore-Penrose pseudoinverse.

## When to use

Use when running MetaboPAC's kinetic-equations approach on time-course relative-abundance (concMatrix/timeVec) data and a subset of kinetic terms for the reaction network are known a priori, so fluxes needed for penalty calculation must be inferred from metabolite rates of change.

## When NOT to use

- The data are not intracellular metabolomics samples in a single compartment — MetaboPAC does not consider transport reactions across intracellular compartments in eukaryotic systems
- No kinetic terms are known and the underdetermined systems cannot be constrained — the pseudoinverse solution requires known kinetic terms
- The goal is flux analysis on measured fluxomics data rather than inferring fluxes from time-course relative-abundance data, since MetaboPAC does not assume fluxomics data are available

## Inputs

- generic_data.mat containing relative abundance matrix (concMatrix) and measurement time vector (timeVec)
- modelInfo_generic: stoichiometric matrix of the system of interest plus known concentration and flux bounds
- Known kinetic equations for the subset of reactions with a priori kinetics
- Response factors (candidate or true RF sets from trueRF/ or solveRF_KEapproach)

## Outputs

- Per-flux flux profiles for each of the four models (stored in flux_profiles.mat)
- Per-metabolite derivative (rate-of-change) matrices
- Fluxes consumed by MetaboPAC's penalty calculation during response-factor optimization

## How to apply

Load generic_data.mat (relative abundance matrix concMatrix, measurement time vector timeVec) and modelInfo_generic containing the stoichiometric matrix plus any known concentration and flux bounds. Smooth the relative-abundance time courses, compute each metabolite's rate of change as the difference in relative abundance between two timepoints divided by the time difference, and divide by the corresponding response factor to obtain the rate of change of absolute concentration. Hard-code the known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m where specified. For each model, solve S·v = dx/dt exactly when the system is determined; for underdetermined models, use the known kinetic terms to constrain the system and apply the Moore-Penrose pseudoinverse to yield a determined system. Store per-metabolite derivative matrices and per-flux flux profiles in flux_profiles.mat, then feed the flux profiles into MetaboPAC's penalty-based optimization over the percKnownKinetics values (e.g., [0 20 40 60 80 100]).

## Related tools

- **MetaboPAC** (Framework whose kinetic-equations approach receives the hard-coded kinetic equations and produces the flux profiles used in penalty calculation) — https://github.com/gtStyLab/MetaboPAC
- **calcFluxWithKinetics_generic.m** (File edited to include known kinetic equations for flux calculation) — https://github.com/gtStyLab/MetaboPAC
- **solveRF_KEapproach_generic.m** (File edited to include known kinetic equations when identifying response factors via the kinetic equations approach) — https://github.com/gtStyLab/MetaboPAC
- **MATLAB** (Execution environment in which generic_data.mat is loaded, S·v = dx/dt is solved, and flux_profiles.mat is written)

## Examples

```
matlab -batch "MetaboPAC_generic_noisy" % after editing modelInfo_generic and hard-coding known kinetic equations into calcFluxWithKinetics_generic.m
```

## Evaluation signals

- Confirm that the computed fluxes reproduce the smoothed derivative profiles through the stoichiometric mass balances (S·v = dx/dt) for each of the four models
- Flux inference should succeed across 3 of 4 models studied, consistent with MetaboPAC's reported results under lower sampling frequency and high noise
- Running with percKnownKinetics = 20 should already give significantly better performance than negative control approaches
- flux_profiles.mat contains per-metabolite derivative matrices and per-flux flux profiles for all four models
- Under the kinetic equations approach, response-factor identification should outperform the biological-heuristics approach as the fraction of known kinetics (percKnownKinetics: 0, 20, 40, 60, 80, 100) increases

## Limitations

- MetaboPAC assumes all metabolites are directly connected via reactions and can only be used for intracellular metabolomics samples in a single compartment; transport reactions across intracellular compartments in eukaryotic systems are not considered
- Response factors are assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption, and allowing nonlinear response factors would be computationally challenging
- High noise levels in the data can lead to violations of mass balances, complicating assessment of biological feasibility of candidate response factors
- Flux calculation accuracy depends on existing/in-development approaches, limiting MetaboPAC inference accuracy; accuracy varied from 40% to nearly 100% depending on network, data quality, and error tolerance
- Missing values (e.g., 10% missingness with imputation) reduce the accuracy of inferred concentrations

## Evidence

- [methods] Flux inference from derivatives: [evidence span withheld — anchor status: not_found]
- [methods] Pseudoinverse solution for underdetermined systems: [evidence span withheld — anchor status: not_found]
