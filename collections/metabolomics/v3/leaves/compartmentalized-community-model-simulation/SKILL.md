---
name: compartmentalized-community-model-simulation
description: Use when you need to simulate a multi-species metabolic community with
  member-attributable compartments, predict feasible community compositions or maximum
  growth rates at a given abundance profile, and enumerate cross-feeding interactions
  independent of abundance.
when_to_use_negative:
- The task concerns a single-species metabolic model with no community members or
  shared exchange compartment — PyCoMo's compartmentalized community structure and
  cross-feeding analysis are not applicable.
- You need kinetic or dynamic simulations of metabolite concentrations rather than
  flux-based steady-state predictions of growth rates, compositions, or exchange fluxes.
- Input SBML files do not follow PyCoMo's required naming conventions (memberId_metaboliteId,
  memberId_compartmentId) and cannot be regenerated with PyCoMo, so member attribution
  and the shared medium compartment cannot be resolved.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: PyCoMo
  role: Generates and switches the compartmentalized community model structure (fixed
    growth rate/variable abundance or vice versa), attributes all entities to member
    models, and supports SBML save/load and exchange/cross-feeding analysis
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Runs flux variability analysis directly on the PyCoMo community model to enumerate
    feasible exchange metabolites and cross-feeding interactions
  repo: https://github.com/univieCUBE/PyCoMo
- name: Python 3
  role: Runtime environment in which PyCoMo is imported and the FVA/cross-feeding
    workflow is scripted
  repo: https://github.com/univieCUBE/PyCoMo
- name: ScyNet
  role: Cytoscape app that visualizes the reduced community network (members plus
    exchange metabolites) contextualized with FBA or FVA flux data
  repo: https://github.com/univieCUBE/ScyNet
- name: SBML
  role: File format used to save and reload PyCoMo community models so they can be
    shared and reused
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:bf05455f5480abdd575ee5b6fe0bc137910cc056f0284e2813edc90d070a8585
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  - ScyNet
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# compartmentalized-community-model-simulation

## Summary

Build and analyse compartmentalized community metabolic models with PyCoMo, then interrogate exchange metabolites and cross-feeding interactions via COBRApy methods such as FVA over variable abundance profiles, as demonstrated on the three-species biogas community (D. vulgaris, M. hungatei, M. barkeri).

## When to use

Use when you need to simulate a multi-species metabolic community with member-attributable compartments, predict feasible community compositions or maximum growth rates at a given abundance profile, and enumerate cross-feeding interactions independent of abundance.

## When NOT to use

- The task concerns a single-species metabolic model with no community members or shared exchange compartment — PyCoMo's compartmentalized community structure and cross-feeding analysis are not applicable.
- You need kinetic or dynamic simulations of metabolite concentrations rather than flux-based steady-state predictions of growth rates, compositions, or exchange fluxes.
- Input SBML files do not follow PyCoMo's required naming conventions (memberId_metaboliteId, memberId_compartmentId) and cannot be regenerated with PyCoMo, so member attribution and the shared medium compartment cannot be resolved.

## Inputs

- Member genome-scale metabolic models (SBML, openCOBRA-compliant)
- Previously generated PyCoMo community model in SBML format
- Abundance profile or community growth rate (whichever is held fixed)
- FVA flux range table with reaction_id, min_flux, max_flux columns (for ScyNet contextualization)

## Outputs

- Compartmentalized community metabolic model (COBRApy/SBML, reusable)
- Predicted maximum community growth rate at a given abundance profile
- Feasible community compositions at a given growth rate
- Feasible exchange metabolites and cross-feeding interaction table (e.g. cross_feeding_fva_results.csv with min/max flux per member exchange reaction)

## How to apply

Install PyCoMo (pip install pycomo) and construct a compartmentalized community model from member genome-scale models; the structure keeps all metabolites, reactions, genes and compartments attributable to their member of origin and preserves original reaction bounds. Switch between the two model structures as needed: fixed community growth rate with variable abundance profile, or fixed abundance profile with variable community growth rate. Load or save the model in SBML format (openCOBRA-compliant) for reuse; because community models are COBRApy models, run COBRApy methods such as FVA directly on member-specific exchange reactions. Extract feasible exchange metabolites and pairwise cross-feeding routes (metabolites exported by one member and consumed by another) from the FVA intervals — e.g. in the biogas community, D. vulgaris supplies acetate and H2 to M. barkeri and formate and H2 to M. hungatei, with cross-feeding via acetate, H2, CO2 and formate. Optionally visualize reduced networks with flux data (FBA single values or FVA min/max ranges) using the ScyNet Cytoscape app, which expects a shared exchange compartment named 'medium' or a 'shared_compartment_id' parameter and member-prefixed IDs.

## Related tools

- **PyCoMo** (Generates and switches the compartmentalized community model structure (fixed growth rate/variable abundance or vice versa), attributes all entities to member models, and supports SBML save/load and exchange/cross-feeding analysis) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Runs flux variability analysis directly on the PyCoMo community model to enumerate feasible exchange metabolites and cross-feeding interactions) — https://github.com/univieCUBE/PyCoMo
- **Python 3** (Runtime environment in which PyCoMo is imported and the FVA/cross-feeding workflow is scripted) — https://github.com/univieCUBE/PyCoMo
- **ScyNet** (Cytoscape app that visualizes the reduced community network (members plus exchange metabolites) contextualized with FBA or FVA flux data) — https://github.com/univieCUBE/ScyNet
- **SBML** (File format used to save and reload PyCoMo community models so they can be shared and reused)

## Examples

```
pip install pycomo; import cobra; model = cobra.io.read_sbml_model('biogas_community.xml'); fva = cobra.flux_analysis.flux_variability_analysis(model, reaction_list=model.exchanges)
```

## Evaluation signals

- The output CSV (e.g. cross_feeding_fva_results.csv) exists and contains one row per member exchange reaction with min/max flux columns.
- Every reported cross-feeding route is supported by opposing flux intervals (export by one member, uptake by another) on the same metabolite between two members.
- The community model loads as a COBRApy model from SBML and retains the original reaction bounds of the input member models.
- The shared exchange compartment is detectable either as a compartment named 'medium' or via the 'shared_compartment_id' parameter, and metabolite/compartment IDs carry the memberId_ prefix required for member attribution.
- Predicted maximum growth rate or feasible compositions reproduce expected results when abundance profile or growth rate is fixed, matching the two model structures PyCoMo supports.

## Limitations

- Analyses cover flux-based steady-state predictions (growth rates, compositions, exchanges, cross-feeding); the package does not provide kinetic community dynamics.
- PyCoMo community models must switch between fixed-growth-rate/variable-abundance and fixed-abundance/variable-growth-rate structures; results are structure-dependent, and cross-feeding enumeration is intended to be independent of the abundance profile.
- ScyNet visualization requires specific SBML formatting (shared 'medium' compartment or shared_compartment_id parameter, memberId_-prefixed IDs) and loads slowly for large models (>10 members).

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [readme] The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions.: "The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles."
- [methods] It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo.: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] A parameter `shared_compartment_id` which is set to the name of the shared exchange compartment.: [evidence span withheld — anchor status: external_doc]
