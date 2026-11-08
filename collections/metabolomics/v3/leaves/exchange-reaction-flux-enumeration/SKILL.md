---
name: exchange-reaction-flux-enumeration
description: Use when a multi-species community metabolic model (e.g. SBML from PyCoMo)
  must be interrogated for which metabolites members can exchange and which donor-consumer
  cross-feeding routes are feasible, independent of a specific abundance profile.
when_to_use_negative:
- The goal is a single flux distribution for one growth condition — flux balance analysis
  (FBA), not enumeration of all feasible exchanges via FVA, is appropriate
- The input is a single-organism (non-community) metabolic model without member-specific
  exchange reactions or a shared exchange compartment
- A fixed abundance profile with variable community growth rate is required instead
  — use the alternative PyCoMo model structure
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Creates the compartmentalized community metabolic model with a structure allowing
    fixed growth rate but variable abundance profile, and can analyse feasible exchange
    metabolites and cross-feeding interactions
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Runs flux variability analysis on the PyCoMo community model, which is a COBRApy
    model directly usable by other COBRA methods
- name: ScyNet
  role: Optional visualization of FVA-derived flux ranges on the reduced community
    network via a reaction_id/min_flux/max_flux tab-separated file
  repo: https://github.com/univieCUBE/ScyNet/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:e529ad1daa72ca3c40cb8f0ffa37ab0f9fcc88bffc7b3fc6df5edd5e324474b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - ScyNet
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# exchange-reaction-flux-enumeration

## Summary

Enumerates all feasible exchange metabolites and pairwise cross-feeding interactions in a compartmentalized community metabolic model by running flux variability analysis (FVA) over member-specific exchange reactions under a fixed-growth-rate, variable-abundance model structure built with PyCoMo and analysed with COBRApy.

## When to use

Use when a multi-species community metabolic model (e.g. SBML from PyCoMo) must be interrogated for which metabolites members can exchange and which donor-consumer cross-feeding routes are feasible, independent of a specific abundance profile.

## When NOT to use

- The goal is a single flux distribution for one growth condition — flux balance analysis (FBA), not enumeration of all feasible exchanges via FVA, is appropriate
- The input is a single-organism (non-community) metabolic model without member-specific exchange reactions or a shared exchange compartment
- A fixed abundance profile with variable community growth rate is required instead — use the alternative PyCoMo model structure

## Inputs

- PyCoMo-generated community metabolic model in SBML format (member-prefixed metabolite and compartment IDs, shared exchange compartment such as 'medium')
- List of member-specific exchange reactions of the community model
- Python 3 environment with PyCoMo and COBRApy installed

## Outputs

- CSV (e.g. cross_feeding_fva_results.csv) with one row per member exchange reaction and min/max flux columns
- Table of pairwise cross-feeding interactions (metabolite exported by one member and consumed by another)
- List of feasible exchange metabolites; optionally a tab-separated reaction_id/min_flux/max_flux file for ScyNet contextualization

## How to apply

Load the previously generated community model from its SBML file with COBRApy, then switch the PyCoMo model to the structure with fixed community growth rate but variable abundance profile so member growth rates are unconstrained. Run COBRApy flux variability analysis (optionally loopless, as offered by PyCoMo) over all member-specific exchange reactions of the community model. From the resulting min/max flux intervals, extract the set of feasible exchange metabolites and derive pairwise cross-feeding routes where one member exports and another consumes the same metabolite. Write the exchange metabolite list and cross-feeding route table (e.g. cross_feeding_fva_results.csv) with a row per member exchange reaction and min/max flux columns; such tab-separated reaction_id/min_flux/max_flux files can also feed ScyNet for visualizing the reduced community network.

## Related tools

- **PyCoMo** (Creates the compartmentalized community metabolic model with a structure allowing fixed growth rate but variable abundance profile, and can analyse feasible exchange metabolites and cross-feeding interactions) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Runs flux variability analysis on the PyCoMo community model, which is a COBRApy model directly usable by other COBRA methods)
- **ScyNet** (Optional visualization of FVA-derived flux ranges on the reduced community network via a reaction_id/min_flux/max_flux tab-separated file) — https://github.com/univieCUBE/ScyNet/wiki

## Examples

```
import cobra; model = cobra.io.read_sbml_model('biogas_community.xml'); fva = cobra.flux_analysis.flux_variability_analysis(model, reaction_list=[r for r in model.reactions if 'exchange' in r.id]); fva.to_csv('cross_feeding_fva_results.csv')
```

## Evaluation signals

- The output CSV exists and contains a row per member exchange reaction with min/max flux columns
- Every reported cross-feeding route is supported by opposing flux intervals on the same metabolite between two members
- Extracted exchange metabolites are independent of the abundance profile, consistent with the fixed-growth-rate, variable-abundance model structure
- Flux intervals fall within the original reaction bounds retained from the input member models
- For the three-species biogas community (D. vulgaris, M. hungatei, M. barkeri), acetate, H2, CO2, and formate surface as cross-fed metabolites

## Limitations

- The enumeration covers feasibility over the whole space of growth rate and abundance profiles; it does not give fluxes for a specific abundance profile
- Requires the shared exchange compartment to be identifiable (a compartment named 'medium' or a shared_compartment_id parameter) for downstream tools like ScyNet
- Thermodynamically infeasible cycles can inflate feasible intervals; PyCoMo offers loopless FVA as a mitigation

## Evidence

- [abstract] PyCoMo models predict all exchange metabolites independent of abundance: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
- [methods] PyCoMo model structure allows variable abundance at fixed growth rate: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
- [methods] PyCoMo community models are COBRApy models: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] SBML save/load of community models: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [other] ScyNet FVA flux file format: [evidence span withheld — anchor status: external_doc]
