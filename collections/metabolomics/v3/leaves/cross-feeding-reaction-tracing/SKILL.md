---
name: cross-feeding-reaction-tracing
description: Use when a community metabolic model (e.g. PyCoMo SBML biogas community
  model) predicts cross-feeding exchanges — such as spurious D. vulgaris H2 uptake
  — that were absent from the original member-model analysis, suggesting an internal
  thermodynamically infeasible cycle.
when_to_use_negative:
- The model is a single-member (non-community) metabolic model with no cross-feeding
  exchange reactions to trace
- The discrepancy under investigation is caused by differing model versions or bounds
  rather than internal cycles — fluxes will already agree between standard and loopless
  FVA
- EFM enumeration with efmtool is required for exhaustive cycle enumeration rather
  than isolating one specific loop behind a known spurious exchange
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: PyCoMo
  role: Provides the loopless FVA workflow (loopless_fva_tutorial) used to obtain
    a loop-free flux distribution on the community model
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the SBML community model and runs standard FVA whose fluxes are contrasted
    with the loopless solution
- name: Python 3
  role: Runtime environment for running PyCoMo and COBRApy analysis scripts
- name: SBML
  role: File format in which the community model is saved and loaded for reuse
- name: efmtool
  role: Optional dependency of PyCoMo for EFM enumeration / CycleBreaker used in cycle
    analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:337071f9f963411837e451fb9ffe8eeb8579afbab775ded88697f7f321197411
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-feeding-reaction-tracing

## Summary

Isolates thermodynamically infeasible cross-feeding cycles in PyCoMo-generated community metabolic models by comparing standard FVA with loopless FVA flux distributions, then tracing and annotating every reaction in the spurious loop to its member model of origin and compartment.

## When to use

Use when a community metabolic model (e.g. PyCoMo SBML biogas community model) predicts cross-feeding exchanges — such as spurious D. vulgaris H2 uptake — that were absent from the original member-model analysis, suggesting an internal thermodynamically infeasible cycle.

## When NOT to use

- The model is a single-member (non-community) metabolic model with no cross-feeding exchange reactions to trace
- The discrepancy under investigation is caused by differing model versions or bounds rather than internal cycles — fluxes will already agree between standard and loopless FVA
- EFM enumeration with efmtool is required for exhaustive cycle enumeration rather than isolating one specific loop behind a known spurious exchange

## Inputs

- PyCoMo-generated compartmentalized community metabolic model in SBML format
- Standard FVA flux distribution over the community model (COBRApy)
- Loopless FVA solution obtained via the PyCoMo loopless_fva_tutorial workflow

## Outputs

- loop_reactions.csv: structured record of the isolated loop reactions with member model of origin, compartment, flux direction, and flux magnitude
- Set of exchange reactions carrying flux in standard FVA but zero in loopless FVA (the infeasible cycle)

## How to apply

Load the compartmentalized community model in SBML format with COBRApy and run PyCoMo's loopless FVA workflow (loopless_fva_tutorial) to obtain a loop-free flux solution. Compare the loopless FVA flux distribution against the standard FVA solution and isolate exchange reactions that carry flux only in the standard solution — e.g. the formate/H2/CO2 cross-feeding between D. vulgaris and M. hungatei. Trace the full reaction path of the identified loop, recording each reaction, its member model of origin (D. vulgaris or M. hungatei), its compartment, and its flux direction and magnitude. The rationale is that a cycle in which D. vulgaris produces formate from H2 and CO2 while M. hungatei converts formate back into H2 and CO2 without consuming any other metabolite violates thermodynamic constraints and explains the spurious uptake. Write the annotated loop reactions to a structured record (loop_reactions.csv) documenting how the spurious exchange is explained.

## Related tools

- **PyCoMo** (Provides the loopless FVA workflow (loopless_fva_tutorial) used to obtain a loop-free flux distribution on the community model) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the SBML community model and runs standard FVA whose fluxes are contrasted with the loopless solution)
- **Python 3** (Runtime environment for running PyCoMo and COBRApy analysis scripts)
- **SBML** (File format in which the community model is saved and loaded for reuse)
- **efmtool** (Optional dependency of PyCoMo for EFM enumeration / CycleBreaker used in cycle analysis)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('community_model.xml'); from pycomo.helper import loopless_fva_tutorial
```

## Evaluation signals

- loop_reactions.csv exists and is populated
- loop_reactions.csv contains the D. vulgaris formate-production reactions of the isolated loop
- loop_reactions.csv contains the M. hungatei formate-consumption reactions of the isolated loop
- No reaction of the traced loop carries flux in the loopless FVA solution
- Each annotated reaction is attributable to its member of origin and compartment, consistent with PyCoMo's fully attributable model structure

## Limitations

- The method requires the optional efm dependency (pip install pycomo[efm] or install efmtool) for EFM enumeration / CycleBreaker; without it cycle-breaking functionality is unavailable
- Loopless FVA removes internal loops from flux solutions but only highlights exchanges that vanish looplessly; reactions that carry flux in both solutions may still participate in other thermodynamic issues
- Community models must retain PyCoMo's naming conventions (member-prefixed metabolite and compartment IDs) for member-of-origin attribution to work

## Evidence

- [other] The spurious D. vulgaris H2 uptake is caused by a thermodynamically infeasible cycle in which D. vulgaris produces formate from H2 and CO2, and M. hungatei converts the formate back into H2 and CO2 without requiring any other metabolites.: [evidence span withheld — anchor status: not_found]
- [methods] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [methods] Installation with optional dependencies for EFM enumeration / CycleBreaker: "Installation with optional dependencies for EFM enumeration / CycleBreaker"
- [readme] It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo.: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] Install PyCoMo via pip with the efm flag:: "Install PyCoMo via pip with the efm flag:"
