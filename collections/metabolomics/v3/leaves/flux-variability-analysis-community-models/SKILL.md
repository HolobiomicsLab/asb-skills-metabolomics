---
name: flux-variability-analysis-community-models
description: Use when you have a PyCoMo-generated community metabolic model (e.g.
  SBML) and need to identify which cross-feeding metabolites and member-to-member
  exchange routes are feasible across the whole space of abundance profiles, rather
  than at a single fixed composition.
when_to_use_negative:
- You want a single flux distribution at a fixed abundance profile — use standard
  FBA instead of variable-abundance FVA
- The model is a single-species (non-community) metabolic model with no member-specific
  exchange compartment
- You need to predict the maximum growth rate at a given abundance profile rather
  than enumerate feasible exchange interactions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model with a structure
    allowing fixed growth rate but variable abundance profile, and can save/load it
    in SBML format
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Runs the flux variability analysis over member-specific exchange reactions
    of the community model
- name: ScyNet
  role: Optionally visualizes the resulting cross-feeding network contextualized with
    FVA flux ranges
  repo: https://github.com/univieCUBE/ScyNet/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:6f73ef46bb79a835f49e44f9012b400f2edcd3b414a68bb5db31907a542a828a
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
yes_rate: 0.9
n_claims: 10
---

# flux-variability-analysis-community-models

## Summary

Enumerates all feasible exchange metabolites and pairwise cross-feeding interactions in a compartmentalized community metabolic model by running COBRApy flux variability analysis (FVA) over member-specific exchange reactions under variable community composition, as enabled by PyCoMo's fixed growth-rate / variable abundance model structure.

## When to use

Use when you have a PyCoMo-generated community metabolic model (e.g. SBML) and need to identify which cross-feeding metabolites and member-to-member exchange routes are feasible across the whole space of abundance profiles, rather than at a single fixed composition.

## When NOT to use

- You want a single flux distribution at a fixed abundance profile — use standard FBA instead of variable-abundance FVA
- The model is a single-species (non-community) metabolic model with no member-specific exchange compartment
- You need to predict the maximum growth rate at a given abundance profile rather than enumerate feasible exchange interactions

## Inputs

- PyCoMo-generated community metabolic model in SBML format (compartmentalized, with shared exchange compartment)
- List of member-specific exchange reactions in the community model

## Outputs

- CSV table (e.g. cross_feeding_fva_results.csv) with one row per member exchange reaction and min/max flux columns
- List of feasible exchange metabolites
- Cross-feeding route table: metabolite exported by one member and consumed by another

## How to apply

Load the previously generated community model from its SBML file with COBRApy, since PyCoMo community models are COBRApy models directly usable by other COBRA methods. Switch the model to the structure with fixed community growth rate but variable abundance profile (member growth rates unconstrained), which is the configuration under which FVA results are independent of any single abundance profile. Run COBRApy FVA over all member-specific exchange reactions of the community model, where every metabolite, reaction and compartment is directly attributable to its member of origin via the memberId_metaboliteId naming scheme. Extract from the FVA min/max intervals the set of feasible exchange metabolites and pairwise cross-feeding routes (metabolites exported by one member and consumed by another). Write per-reaction min/max flux intervals to a CSV (e.g. cross_feeding_fva_results.csv); ranges in this reaction_id/min_flux/max_flux format can also be visualized with ScyNet.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model with a structure allowing fixed growth rate but variable abundance profile, and can save/load it in SBML format) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Runs the flux variability analysis over member-specific exchange reactions of the community model)
- **ScyNet** (Optionally visualizes the resulting cross-feeding network contextualized with FVA flux ranges) — https://github.com/univieCUBE/ScyNet/wiki

## Examples

```
model = cobra.io.read_sbml_model('biogas_community.xml'); fva = cobra.flux_analysis.flux_variability_analysis(model, reaction_list=member_exchange_reactions, fraction_of_optimum=0); fva.to_csv('cross_feeding_fva_results.csv')
```

## Evaluation signals

- Output CSV exists and contains one row per member exchange reaction with min_flux and max_flux columns
- Every reported cross-feeding route is supported by opposing (nonzero) flux intervals on the same metabolite between two members
- Model was in the fixed-community-growth-rate / variable-abundance structure before running FVA, so results are independent of abundance profile
- All exchanges reported as feasible correspond to member-specific exchange reactions present in the loaded SBML model
- For the biogas community, FVA recovers cross-feeding via acetate, H2, CO2, and formate with D. vulgaris feeding M. barkeri and M. hungatei

## Limitations

- FVA must be run in the fixed growth rate / variable abundance structure; results depend on switching the PyCoMo model structure correctly
- Reported interactions are feasible flux ranges, not realized fluxes — they do not indicate which route occurs at a specific abundance profile
- Member models must retain original reaction bounds and memberId_-prefixed IDs for correct attribution of metabolites and exchanges

## Evidence

- [abstract] variable abundance FVA rationale: [evidence span withheld — anchor status: not_found]
- [abstract] FVA demonstrated on biogas community: "we demonstrate PyCoMo’s capability by analysing methane production in a previously published simplified biogas community metabolic model"
- [methods] COBRApy compatibility: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] model structure switch: "simulations under fixed growth rate, but variable abundance profile, or fixed abundance profile and variable community growth rate"
- [methods] SBML reuse: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
