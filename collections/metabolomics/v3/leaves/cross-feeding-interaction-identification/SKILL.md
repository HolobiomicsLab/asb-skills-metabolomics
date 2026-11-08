---
name: cross-feeding-interaction-identification
description: Use when a PyCoMo-generated community metabolic model (SBML) must be
  queried for which metabolites members can exchange and which member-to-member cross-feeding
  routes are feasible, independent of a specific abundance profile.
when_to_use_negative:
- When the goal is a single flux distribution at a fixed abundance profile (FBA) rather
  than the space of feasible interactions — use standard COBRApy FBA instead
- When the input models are not community member models compartmentalized by PyCoMo,
  since member attribution and the shared exchange compartment structure are required
  to map fluxes to individual members
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0602
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model and switches it
    between fixed growth rate / variable abundance and fixed abundance / variable
    growth rate structures; supports SBML save/load
  repo: http://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Executes flux variability analysis on the PyCoMo community model, which is
    natively a COBRApy model
- name: ScyNet
  role: 'Optional downstream visualization: contextualizes the reduced community network
    edges with FVA flux ranges supplied as reaction_id/min_flux/max_flux files'
  repo: https://github.com/univieCUBE/ScyNet/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:49a85e9fc92c01b1a767bc8bb21d70588c3699f34b68dd2f40204d2a89065e06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - ScyNet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-feeding-interaction-identification

## Summary

Identify all feasible exchange metabolites and pairwise cross-feeding interactions in a compartmentalized community metabolic model (e.g. the D. vulgaris / M. hungatei / M. barkeri biogas community) by running FVA with COBRApy over member-specific exchange reactions of a PyCoMo model set to fixed growth rate and variable abundance profile.

## When to use

Use when a PyCoMo-generated community metabolic model (SBML) must be queried for which metabolites members can exchange and which member-to-member cross-feeding routes are feasible, independent of a specific abundance profile.

## When NOT to use

- When the goal is a single flux distribution at a fixed abundance profile (FBA) rather than the space of feasible interactions — use standard COBRApy FBA instead
- When the input models are not community member models compartmentalized by PyCoMo, since member attribution and the shared exchange compartment structure are required to map fluxes to individual members

## Inputs

- PyCoMo community metabolic model saved in SBML format (compartmentalized, member-prefixed IDs, shared 'medium' exchange compartment)
- List of member-specific exchange reactions of the community model
- Python 3 environment with PyCoMo and COBRApy installed

## Outputs

- CSV (e.g. cross_feeding_fva_results.csv) with one row per member exchange reaction and min/max flux columns
- List of feasible exchange metabolites independent of the abundance profile
- Cross-feeding route table (metabolite exported by one member and consumed by another)

## How to apply

Load the previously generated PyCoMo community model from its SBML file with COBRApy, then switch the model to the fixed community growth rate structure so the abundance profile is variable and member growth rates are unconstrained — this is the PyCoMo mode designed to probe 'all feasible exchange metabolites and cross-feeding interactions... for the whole space of growth rate and abundance profiles'. Run COBRApy flux variability analysis over all member-specific exchange reactions, since PyCoMo community models are COBRApy models directly usable by other COBRA methods. From the returned min/max flux intervals, extract exchange metabolites with nonzero feasible flux and derive cross-feeding routes by pairing a metabolite exported (nonzero lower or upper flux) by one member with consumption (opposing flux interval) of the same metabolite by another member. In the demonstrated biogas community this recovers acetate and H2 from D. vulgaris to M. barkeri (acetoclastic and hydrogenotrophic methanogenesis) and formate and H2 to M. hungatei (hydrogenotrophic pathway), with cross-feeding via acetate, H2, CO2, and formate. Write per-reaction min/max fluxes to a CSV (e.g. cross_feeding_fva_results.csv); the same tab-separated reaction_id/min_flux/max_flux format is what ScyNet consumes for FVA visualization.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model and switches it between fixed growth rate / variable abundance and fixed abundance / variable growth rate structures; supports SBML save/load) — http://github.com/univieCUBE/PyCoMo
- **COBRApy** (Executes flux variability analysis on the PyCoMo community model, which is natively a COBRApy model)
- **ScyNet** (Optional downstream visualization: contextualizes the reduced community network edges with FVA flux ranges supplied as reaction_id/min_flux/max_flux files) — https://github.com/univieCUBE/ScyNet/wiki

## Examples

```
import cobra; model = cobra.io.read_sbml_model('biogas_community.xml'); fva = cobra.flux_analysis.flux_variability_analysis(model, reaction_list=[r.id for r in model.exchanges]); fva.to_csv('cross_feeding_fva_results.csv')
```

## Evaluation signals

- Output CSV exists and contains one row per member-specific exchange reaction with min and max flux columns
- Every reported cross-feeding route is supported by opposing (export vs. uptake) flux intervals on the same metabolite between two members
- Recovered exchange metabolites are independent of any chosen abundance profile (fixed growth rate, variable abundance structure was active)
- For the biogas community, expected routes include acetate+H2 from D. vulgaris to M. barkeri and formate+H2 to M. hungatei, with cross-feeding via acetate, H2, CO2, and formate
- Model retains original member reaction bounds after structure switching (PyCoMo models 'retain the original reaction bounds of the input member models')

## Limitations

- Results describe feasibility across the whole space of growth rate and abundance profiles, not fluxes at a specific community composition
- Community model must be regenerated/reloaded via PyCoMo-compatible SBML with member-prefixed metabolite and compartment IDs and a detectable shared exchange compartment ('medium' or a shared_compartment_id parameter) for member attribution to work

## Evidence

- [abstract] PyCoMo predicts all exchange metabolites and cross-feeding interactions independent of the abundance profile: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
- [methods] The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles: "The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles."
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods: "The community models are COBRApy models and can therefor be directly used by other COBRA methods. It is also possible to save and load the community models in SBML format"
- [other] FVA flux files for ScyNet require reaction_id, min_flux, max_flux columns, matching the cross_feeding_fva_results.csv schema: [evidence span withheld — anchor status: external_doc]
- [abstract] The demonstrated application is methane production analysis in a simplified biogas community metabolic model: "we demonstrate PyCoMo’s capability by analysing methane production in a previously published simplified biogas community metabolic model"
