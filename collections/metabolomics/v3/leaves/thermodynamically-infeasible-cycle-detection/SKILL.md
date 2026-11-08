---
name: thermodynamically-infeasible-cycle-detection
description: Use when a compartmentalized community metabolic model (SBML, PyCoMo/COBRApy)
  predicts exchange fluxes or cross-feeding interactions absent from a reference analysis,
  suggesting a thermodynamically infeasible cycle.
when_to_use_negative:
- The model is a single-organism model with no community cross-feeding compartment,
  so no inter-member formate/H2/CO2 loops can form
- The discrepancy between model prediction and reference analysis is due to differing
  model content or bounds, not internal loops — loopless FVA will not change the exchange
  fluxes
- Only maximum growth rate or feasible community composition is needed, not exchange/cross-feeding
  flux feasibility
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model and provides the
    loopless FVA workflow (loopless_fva_tutorial) and CycleBreaker helper used to
    isolate infeasible cycles
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the SBML community model and runs the standard and loopless FVA simulations
    whose flux distributions are compared
- name: efmtool
  role: Optional dependency for EFM enumeration supporting cycle analysis, installed
    via 'pip install pycomo[efm]'
- name: SBML
  role: File format in which the community model is saved and loaded for reuse in
    the loop analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:9483c222d45b3509d7fe1e69ade2b99ab92bfc91966a07c0d3c455b1bfcb8515
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# thermodynamically-infeasible-cycle-detection

## Summary

Detects thermodynamically infeasible internal cycles in PyCoMo-generated community metabolic models by contrasting standard FVA with loopless FVA (COBRApy), then traces the full reaction path of the isolated loop to explain spurious cross-feeding predictions such as the D. vulgaris H2 uptake artifact.

## When to use

Use when a compartmentalized community metabolic model (SBML, PyCoMo/COBRApy) predicts exchange fluxes or cross-feeding interactions absent from a reference analysis, suggesting a thermodynamically infeasible cycle.

## When NOT to use

- The model is a single-organism model with no community cross-feeding compartment, so no inter-member formate/H2/CO2 loops can form
- The discrepancy between model prediction and reference analysis is due to differing model content or bounds, not internal loops — loopless FVA will not change the exchange fluxes
- Only maximum growth rate or feasible community composition is needed, not exchange/cross-feeding flux feasibility

## Inputs

- PyCoMo-generated compartmentalized community metabolic model in SBML format
- Standard FVA flux distribution (or FVA flux-range file with reaction_id, min_flux, max_flux)
- Loopless FVA solution from the PyCoMo loopless_fva_tutorial workflow

## Outputs

- loop_reactions.csv: structured record of loop reactions with member model of origin (e.g. D. vulgaris or M. hungatei), compartment, flux direction, and flux magnitude
- Explanation of the spurious exchange prediction (e.g. D. vulgaris H2 uptake) as a thermodynamically infeasible cycle

## How to apply

Load the compartmentalized community model in SBML format with COBRApy, as produced by PyCoMo. Run the PyCoMo loopless FVA workflow (loopless_fva_tutorial) to obtain a flux distribution free of internal loops, and run standard FVA on the same model. Compare the two solutions and isolate exchange reactions carrying flux in the standard FVA solution but not in the loopless one — e.g. the formate/H2/CO2 cross-feeding between D. vulgaris and M. hungatei. Trace the full reaction path of the identified loop, recording each reaction, its member model of origin, its compartment, and its flux direction and magnitude; EFM enumeration via efmtool (installed with 'pip install pycomo[efm]') and the PyCoMo CycleBreaker helper can support cycle analysis. Write the annotated loop reactions to a structured record (loop_reactions.csv) documenting how the spurious uptake is explained by the cycle.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model and provides the loopless FVA workflow (loopless_fva_tutorial) and CycleBreaker helper used to isolate infeasible cycles) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the SBML community model and runs the standard and loopless FVA simulations whose flux distributions are compared)
- **efmtool** (Optional dependency for EFM enumeration supporting cycle analysis, installed via 'pip install pycomo[efm]')
- **SBML** (File format in which the community model is saved and loaded for reuse in the loop analysis)

## Examples

```
pip install pycomo[efm]
```

## Evaluation signals

- loop_reactions.csv exists and contains both the D. vulgaris formate-production reactions and the M. hungatei formate-consumption reactions of the isolated loop
- No reaction of the isolated loop carries flux in the loopless FVA solution
- The loop cycle closes stoichiometrically: formate produced by D. vulgaris from H2 and CO2 is converted back into H2 and CO2 by M. hungatei without requiring any other metabolites
- The identified exchange reactions (e.g. D. vulgaris H2 uptake) carry flux in standard FVA but zero flux in loopless FVA
- Every loop reaction is attributable to its member model of origin and compartment via the PyCoMo member-prefixed IDs

## Limitations

- The artifact arises from the PyCoMo community model structure; the original analysis (Koch et al. 2019) did not predict the spurious D. vulgaris H2 uptake, so the cycle is an artifact of the compartmentalized community construction, not of the underlying member models
- Loopless FVA removes loop flux but the model retains original reaction bounds of the input member models, so cycle-prone stoichiometry persists and must be checked each time
- EFM enumeration via efmtool is an optional dependency and must be installed explicitly ('pip install pycomo[efm]'); without it, cycle enumeration support is unavailable

## Evidence

- [methods] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [methods] loopless FVA tutorial reference: "   loopless_fva_tutorial"
- [methods] optional dependencies for EFM enumeration: "Installation with optional dependencies for EFM enumeration / CycleBreaker"
- [methods] SBML save/load support: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] FVA flux file format for flux contextualization: [evidence span withheld — anchor status: external_doc]
