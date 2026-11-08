---
name: pseudoinverse-underdetermined-system-computation
description: Use when time-course relative-abundance metabolomics data must be converted
  into flux profiles for penalty calculation, and the stoichiometric system S·v =
  dx/dt is underdetermined without fluxomics measurements.
when_to_use_negative:
- Data are from eukaryotic systems where inter-compartment transport reactions matter
  — MetaboPAC does not take transport reactions across intracellular compartments
  into consideration
- Samples are extracellular or multi-compartment — the approach applies only to intracellular
  metabolomics samples in a single compartment
- Flux profiles are already available from fluxomics measurements, making inference
  from concentration derivatives unnecessary
edam_operation: http://edamontology.org/operation_3926
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: MetaboPAC
  role: Framework whose generic scripts (MetaboPAC_generic.m, calcFluxWithKinetics_generic.m,
    solveRF_KEapproach_generic.m) perform the determined-system and pseudoinverse
    flux calculations
  repo: gtStyLab/MetaboPAC
- name: MATLAB
  role: Environment in which the MetaboPAC_generic scripts solve S·v = dx/dt and compute
    the Moore-Penrose pseudoinverse
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:fc2c587615c10f229eb0b1251d2c611c239885b4df179604c83ab79ab6f88f11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/gtStyLab/MetaboPAC
  tools:
  - MATLAB
  - MetaboPAC
  edam_operation: http://edamontology.org/operation_3926
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pseudoinverse-underdetermined-system-computation

## Summary

Solves the stoichiometric mass-balance system S·v = dx/dt to infer metabolic flux profiles needed by MetaboPAC's penalty calculation: exactly for determined systems and via the Moore-Penrose pseudoinverse (with known kinetic terms constraining the system) for underdetermined models.

## When to use

Use when time-course relative-abundance metabolomics data must be converted into flux profiles for penalty calculation, and the stoichiometric system S·v = dx/dt is underdetermined without fluxomics measurements.

## When NOT to use

- Data are from eukaryotic systems where inter-compartment transport reactions matter — MetaboPAC does not take transport reactions across intracellular compartments into consideration
- Samples are extracellular or multi-compartment — the approach applies only to intracellular metabolomics samples in a single compartment
- Flux profiles are already available from fluxomics measurements, making inference from concentration derivatives unnecessary

## Inputs

- generic_data.mat containing relative abundance matrix (concMatrix) and measurement time vector (timeVec)
- Stoichiometric matrix plus known concentration and flux bounds from modelInfo_generic
- Known kinetic equations (hard-coded into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m)
- Putative response factors (e.g., from trueRF replicate sets)

## Outputs

- Per-metabolite rate-of-change (derivative) matrices
- Per-flux flux profiles for each of the four models, e.g., stored in flux_profiles.mat

## How to apply

Load the relative abundance matrix (concMatrix) and measurement times (timeVec) from generic_data.mat, pull the stoichiometric matrix and concentration/flux bounds from modelInfo_generic, and smooth the relative-abundance time courses. Compute each metabolite's rate of change as the difference in relative abundance between two timepoints divided by the time difference, then divide that rate by the corresponding response factor to obtain the rate of change of absolute concentration. For each of the four models, solve S·v = dx/dt exactly when the system is determined; when underdetermined, hard-code known kinetic equations into calcFluxWithKinetics_generic.m and solveRF_KEapproach_generic.m and apply the Moore-Penrose pseudoinverse to yield a determined system. Store the per-metabolite derivative matrices and per-flux flux profiles (e.g., in flux_profiles.mat) for use in MetaboPAC's optimization penalty calculation.

## Related tools

- **MetaboPAC** (Framework whose generic scripts (MetaboPAC_generic.m, calcFluxWithKinetics_generic.m, solveRF_KEapproach_generic.m) perform the determined-system and pseudoinverse flux calculations) — gtStyLab/MetaboPAC
- **MATLAB** (Environment in which the MetaboPAC_generic scripts solve S·v = dx/dt and compute the Moore-Penrose pseudoinverse)

## Examples

```
load('generic_data.mat'); [modelInfo] = modelInfo_generic; v = pinv(S_withKinetics)*dxdt % solve S·v = dx/dt via Moore-Penrose pseudoinverse in calcFluxWithKinetics_generic.m
```

## Evaluation signals

- Confirm that the computed fluxes reproduce the smoothed derivative profiles through the stoichiometric mass balances for each of the four models
- Check consistency with MetaboPAC's reported flux inference across 3 of 4 models studied
- Verify the per-metabolite derivative matrices equal smoothed relative-abundance differences divided by time deltas and then by the corresponding response factors
- For underdetermined models, verify the pseudoinverse solution yields a determined system only after known kinetic terms are incorporated

## Limitations

- Flux calculation accuracy depends on existing/in-development approaches, which limits MetaboPAC inference accuracy
- High noise levels in the data can lead to violations of mass balances and complicate the assessment of biological feasibility
- Response factors were assumed constant with concentration; nonlinear response (ion suppression, limited dynamic range) violates this assumption, and it is not evident which metabolites would need such allowances
- MetaboPAC does not take into consideration transport reactions across different intracellular compartments in eukaryotic systems

## Evidence

- [full_text] MetaboPAC infers fluxes by computing each metabolite's rate of change from the difference in relative abundance between two timepoints divided by the time difference: [evidence span withheld — anchor status: not_found]
