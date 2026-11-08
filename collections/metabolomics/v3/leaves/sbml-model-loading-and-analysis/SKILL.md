---
name: sbml-model-loading-and-analysis
description: Use when a compartmentalized community metabolic model (SBML from PyCoMo,
  e.g. the biogas community of D. vulgaris, M. hungatei, M. barkeri) must be reloaded
  and analysed for exchange fluxes or cross-feeding via COBRApy.
when_to_use_negative:
- The goal is a single optimal flux distribution for a fixed abundance profile (FBA),
  not the full space of feasible exchange metabolites and cross-feeding interactions
- The input is raw member genome-scale models not yet combined — use PyCoMo model
  creation first, then load the resulting SBML
- Only visual exploration of the community network is needed (use ScyNet on the SBML
  instead of scripting FVA)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model in SBML format and
    provides the fixed growth rate / variable abundance structure used for the analysis
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the SBML community model and runs FVA over member-specific exchange
    reactions
- name: Python 3
  role: Runtime environment in which PyCoMo and COBRApy are scripted (pip install
    pycomo)
- name: ScyNet
  role: Optional visualization of the reduced community network contextualized with
    FVA flux ranges (reaction_id, min_flux, max_flux tab-separated file)
  repo: http://github.com/univieCUBE/PyCoMo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:a8b85b51b793b01ea287fae11fa72a19ccca29aff1586be22872c763553ec9b9
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# sbml-model-loading-and-analysis

## Summary

Load a PyCoMo-generated, COBRApy-compatible community metabolic model from SBML and analyse it with COBRApy methods (e.g. FVA) to enumerate feasible exchange metabolites and cross-feeding interactions independent of the abundance profile.

## When to use

Use when a compartmentalized community metabolic model (SBML from PyCoMo, e.g. the biogas community of D. vulgaris, M. hungatei, M. barkeri) must be reloaded and analysed for exchange fluxes or cross-feeding via COBRApy.

## When NOT to use

- The goal is a single optimal flux distribution for a fixed abundance profile (FBA), not the full space of feasible exchange metabolites and cross-feeding interactions
- The input is raw member genome-scale models not yet combined — use PyCoMo model creation first, then load the resulting SBML
- Only visual exploration of the community network is needed (use ScyNet on the SBML instead of scripting FVA)

## Inputs

- Community metabolic model in SBML format (PyCoMo-generated, with shared medium/exchange compartment and member-prefixed IDs)
- List or automatic detection of member-specific exchange reactions

## Outputs

- CSV of FVA intervals per exchange reaction (e.g. cross_feeding_fva_results.csv with reaction_id, min_flux, max_flux)
- List of feasible exchange metabolites (e.g. acetate, H2, CO2, formate)
- Pairwise cross-feeding route table (exporter member, metabolite, consumer member)

## How to apply

Load the previously generated community model from its SBML file with COBRApy, which PyCoMo models directly support. Switch the model to the fixed community growth rate structure so the abundance profile is variable and member growth rates are unconstrained. Run COBRApy flux variability analysis (FVA) over all member-specific exchange reactions. Extract from the FVA min/max flux intervals the feasible exchange metabolites and pairwise cross-feeding routes (a metabolite exported by one member and consumed by another), e.g. acetate, H2, CO2, and formate in the biogas community. Write results (e.g. cross_feeding_fva_results.csv) with one row per exchange reaction and min/max flux columns.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model in SBML format and provides the fixed growth rate / variable abundance structure used for the analysis) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the SBML community model and runs FVA over member-specific exchange reactions)
- **Python 3** (Runtime environment in which PyCoMo and COBRApy are scripted (pip install pycomo))
- **ScyNet** (Optional visualization of the reduced community network contextualized with FVA flux ranges (reaction_id, min_flux, max_flux tab-separated file)) — http://github.com/univieCUBE/PyCoMo

## Examples

```
import cobra; model = cobra.io.read_sbml_model('biogas_community.xml'); fva = cobra.flux_analysis.flux_variability_analysis(model, reaction_list=[r for r in model.reactions if 'EX_' in r.id]); fva.to_csv('cross_feeding_fva_results.csv')
```

## Evaluation signals

- Output CSV exists and contains one row per member exchange reaction with min_flux and max_flux columns
- Every reported cross-feeding route is supported by opposing flux intervals on the same metabolite between two members (exporter min/max vs consumer min/max)
- Loaded model retains member-attributable metabolites, reactions, genes and compartments as guaranteed by the PyCoMo SBML format
- Exchange metabolites found (acetate, H2, CO2, formate) match the expected biogas-community routes: D. vulgaris → M. barkeri (acetate, H2) and D. vulgaris → M. hungatei (formate, H2)

## Limitations

- FVA results are only meaningful once the model is switched to the fixed growth rate structure with variable abundance profile; using the wrong structure changes what fluxes are feasible
- SBML files intended for reuse/visualization must follow PyCoMo's ID conventions (memberId_metaboliteId, memberId_compartmentId, shared 'medium' compartment or shared_compartment_id parameter) or downstream tools like ScyNet cannot parse them
- Analysis covers the whole space of growth rate and abundance profiles; it does not give a single predicted flux state for a specific abundance

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [readme] It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo.: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [abstract] (iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
- [methods] PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
- [readme] A tab separated file with three columns needs to be provided: reaction_id, min_flux, max_flux.: [evidence span withheld — anchor status: external_doc]
