---
name: formate-hydrogen-shuttle-loop-identification
description: Use when a community FBA/FVA solution predicts exchange fluxes (e.g.,
  H2 and CO2 uptake) that were absent from the original analysis, suggesting a thermodynamically
  infeasible cycle rather than a biological interaction.
when_to_use_negative:
- The model is a single-organism model rather than a PyCoMo compartmentalized community
  model with member-prefixed metabolites and compartments
- The analysis goal is enumerating elementary flux modes rather than diagnosing a
  specific spurious exchange flux between two community members
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model and provides the
    loopless FVA workflow used to detect the infeasible cycle
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads and simulates the community SBML model for standard and loopless FVA
- name: efmtool
  role: Optional dependency for EFM enumeration / CycleBreaker used with PyCoMo
- name: SBML
  role: File format used to save and load the community model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:494861d6eb6b001d1bf5fe7f318b0105725531486ede5d6690cc2eaf9a86bdb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  - efmtool
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Formate-hydrogen shuttle loop identification

## Summary

A diagnostic workflow for isolating thermodynamically infeasible cross-feeding cycles in PyCoMo-generated community metabolic models, exemplified by the spurious D. vulgaris H2 uptake driven by a formate/H2/CO2 shuttle with M. hungatei in the Koch et al. (2019) biogas community.

## When to use

Use when a community FBA/FVA solution predicts exchange fluxes (e.g., H2 and CO2 uptake) that were absent from the original analysis, suggesting a thermodynamically infeasible cycle rather than a biological interaction.

## When NOT to use

- The model is a single-organism model rather than a PyCoMo compartmentalized community model with member-prefixed metabolites and compartments
- The analysis goal is enumerating elementary flux modes rather than diagnosing a specific spurious exchange flux between two community members

## Inputs

- PyCoMo-generated compartmentalized community metabolic model in SBML format
- Standard FVA flux distribution from the community model
- Loopless FVA flux distribution obtained via the PyCoMo loopless_fva_tutorial

## Outputs

- loop_reactions.csv with annotated loop reactions (reaction ID, member model of origin, compartment, flux direction and magnitude)
- Isolated set of exchange reactions carrying flux in standard FVA but not in loopless FVA
- Explanation of the spurious exchange flux (e.g., D. vulgaris H2 uptake) as a thermodynamically infeasible cycle

## How to apply

Load the compartmentalized biogas community model in SBML format with COBRApy, as produced by PyCoMo. Run the PyCoMo loopless FVA workflow (loopless_fva_tutorial) to obtain a loop-free flux solution. Compare the loopless FVA flux distribution with the standard FVA solution and isolate exchange reactions that carry flux only in the standard solution — here, the formate/H2/CO2 cross-feeding between D. vulgaris and M. hungatei. Trace the full reaction path of the loop, recording each reaction, its member model of origin (D. vulgaris or M. hungatei), its compartment, and flux direction and magnitude. Rationale: D. vulgaris produces formate from H2 and CO2 while M. hungatei converts formate back to H2 and CO2 without requiring any other metabolites, a cycle that violates thermodynamic constraints and inflates exchange predictions. Write the annotated loop reactions to loop_reactions.csv to document the explanation for the spurious uptake.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model and provides the loopless FVA workflow used to detect the infeasible cycle) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads and simulates the community SBML model for standard and loopless FVA)
- **efmtool** (Optional dependency for EFM enumeration / CycleBreaker used with PyCoMo)
- **SBML** (File format used to save and load the community model)

## Examples

```
pip install pycomo[efm]
```

## Evaluation signals

- loop_reactions.csv exists and is populated
- loop_reactions.csv contains the D. vulgaris formate-production reactions and the M. hungatei formate-consumption reactions of the isolated loop
- No reaction of the identified loop carries flux in the loopless FVA solution
- The isolated reactions carry flux in the standard FVA solution but not in the loopless solution (differential flux comparison)
- Each annotated reaction is traceable to its member model of origin (D. vulgaris or M. hungatei) and compartment

## Limitations

- The community model structure is required: metabolites, reactions, genes and compartments must be attributable to their member of origin (memberId_metaboliteId prefixing) for the loop trace to identify member of origin
- Loopless FVA is computationally heavier than standard FVA; the loop must be isolated by differential comparison of the two solutions

## Evidence

- [methods] Loopless FVA isolates the infeasible cycle: [evidence span withheld — anchor status: not_found]
- [methods] PyCoMo models are COBRApy models loadable from SBML: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [results] The finding: formate shuttle cycle between D. vulgaris and M. hungatei: [evidence span withheld — anchor status: not_found]
- [readme] PyCoMo attributes all model parts to their member of origin: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [methods] Optional EFM/CycleBreaker dependency installation: "Installation with optional dependencies for EFM enumeration / CycleBreaker"
