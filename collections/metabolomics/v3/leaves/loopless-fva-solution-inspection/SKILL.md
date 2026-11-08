---
name: loopless-fva-solution-inspection
description: Use when a PyCoMo/COBRApy community model (SBML) predicts exchange or
  cross-feeding fluxes absent from the original member-model analysis and you suspect
  an internal loop; compare standard vs. loopless FVA to flag them.
when_to_use_negative:
- When the model has no internal cycles or no exchange flux disappears under loopless
  constraints — the comparison will be uninformative
- When you need loop-free flux prediction itself rather than diagnosing why a standard
  FVA solution carries spurious flux — use PyCoMo's max growth rate or discretized
  growth workflows instead
- When the model is not a PyCoMo community model with member-prefixed IDs and a shared
  exchange compartment (medium / shared_compartment_id)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
tools:
- name: PyCoMo
  role: Provides the loopless FVA workflow (loopless_fva_tutorial) and generates the
    compartmentalized community model; optional efm extra enables EFM enumeration
    / CycleBreaker loop analysis
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the SBML community model and runs standard FVA for the comparison against
    the loopless solution
- name: efmtool
  role: Optional dependency (pip install pycomo[efm]) for EFM enumeration / CycleBreaker
    to enumerate elementary flux cycles
  repo: http://github.com/univieCUBE/PyCoMo
- name: Python 3
  role: Runtime for PyCoMo and COBRApy workflow scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:30e74061d4d2fafbd2cea85ef5f74ac6427916bd1b5897758b511add797cb9ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - efmtool
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Loopless FVA solution inspection

## Summary

Runs PyCoMo's loopless FVA on a compartmentalized community metabolic model and contrasts it with standard FVA to isolate exchange reactions whose flux exists only because of a thermodynamically infeasible cycle (e.g. the spurious D. vulgaris H2 uptake in the biogas community model).

## When to use

Use when a PyCoMo/COBRApy community model (SBML) predicts exchange or cross-feeding fluxes absent from the original member-model analysis and you suspect an internal loop; compare standard vs. loopless FVA to flag them.

## When NOT to use

- When the model has no internal cycles or no exchange flux disappears under loopless constraints — the comparison will be uninformative
- When you need loop-free flux prediction itself rather than diagnosing why a standard FVA solution carries spurious flux — use PyCoMo's max growth rate or discretized growth workflows instead
- When the model is not a PyCoMo community model with member-prefixed IDs and a shared exchange compartment (medium / shared_compartment_id)

## Inputs

- PyCoMo-generated compartmentalized community metabolic model in SBML format
- COBRApy-readable model object (community models are COBRApy models)
- Optional FVA flux files (reaction_id, min_flux, max_flux columns) for contextualization

## Outputs

- loop_reactions.csv: structured record of loop reactions with reaction ID, member model of origin, compartment, flux direction and magnitude
- Set of exchange reactions carrying flux in standard FVA but not in loopless FVA (loop-dependent fluxes)
- Annotated explanation of the thermodynamically infeasible cycle (e.g. D. vulgaris formate production / M. hungatei formate consumption)

## How to apply

Load the PyCoMo-generated compartmentalized community model in SBML format with COBRApy and run the loopless FVA workflow (PyCoMo loopless_fva_tutorial) to obtain a solution free of internal loops. Run standard FVA in parallel and compute the set difference: exchange reactions carrying flux in standard FVA but zero flux in the loopless solution are loop-dependent candidates. For each candidate, trace the full reaction path of the loop, recording each participating reaction, its member model of origin (e.g. D. vulgaris or M. hungatei), its compartment, and its flux direction and magnitude; for the biogas case this surfaces the formate/H2/CO2 cross-feeding cycle in which D. vulgaris produces formate from H2 and CO2 and M. hungatei converts formate back to H2 and CO2. Write the annotated loop reactions to a structured record (loop_reactions.csv) documenting how the spurious uptake is explained by the thermodynamically infeasible cycle. Optionally cross-check with EFM enumeration via PyCoMo's efmtool/CycleBreaker dependencies (pip install pycomo[efm]).

## Related tools

- **PyCoMo** (Provides the loopless FVA workflow (loopless_fva_tutorial) and generates the compartmentalized community model; optional efm extra enables EFM enumeration / CycleBreaker loop analysis) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the SBML community model and runs standard FVA for the comparison against the loopless solution)
- **efmtool** (Optional dependency (pip install pycomo[efm]) for EFM enumeration / CycleBreaker to enumerate elementary flux cycles) — http://github.com/univieCUBE/PyCoMo
- **Python 3** (Runtime for PyCoMo and COBRApy workflow scripts)

## Examples

```
pip install pycomo[efm]
```

## Evaluation signals

- loop_reactions.csv exists and contains both the D. vulgaris formate-production reactions and the M. hungatei formate-consumption reactions of the isolated loop
- No reaction in the recorded loop carries flux in the loopless FVA solution (loop-free invariant)
- Every recorded loop reaction carries flux in the standard FVA solution, and the spurious exchange flux (D. vulgaris H2 uptake) is present only in the standard solution
- Each loop reaction is annotated with its member model of origin and compartment, matching the PyCoMo memberId_metaboliteId / memberId_compartmentId conventions
- The loop closes mass-balanced: no external metabolite beyond the shared exchange compartment is required to sustain the cycle

## Limitations

- Loopless FVA can be computationally expensive on genome-scale community models, especially with many members
- The community model must retain the original member reaction bounds and member-attributable IDs as PyCoMo guarantees; models in other formats may not be traceable
- PyCoMo-generated models are compliant with openCOBRA formats; non-compliant SBML files cannot be loaded without the required member-prefixed IDs and a shared exchange compartment (medium or shared_compartment_id parameter)

## Evidence

- [methods] PyCoMo loopless FVA workflow: [evidence span withheld — anchor status: not_found]
- [methods] COBRApy compatibility: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [methods] SBML save/load: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [methods] EFM dependencies: "Installation with optional dependencies for EFM enumeration / CycleBreaker"
- [readme] PyCoMo purpose: "PyCoMo is a python 3 package for the creation and analysis of community metabolic models."
- [abstract] Cross-feeding prediction: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
