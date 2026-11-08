---
name: compartmentalized-model-generation
description: Use when multiple genome-scale SBML member models must be merged into
  a single reusable community metabolic model with member-of-origin compartmentalization
  and openCOBRA/COBRApy compliance for growth-rate or abundance-profile simulations.
when_to_use_negative:
- You only have a single-organism metabolic model and no community members to merge
  — PyCoMo community model generation requires multiple member models
- You need to perform flux analysis or visualize an already-built community model
  rather than create one — use COBRApy methods or ScyNet for analysis/visualization
  of the existing SBML community model
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: PyCoMo
  role: Core Python 3 package that generates the compartmentalized community metabolic
    model from member SBML models, matching exchange metabolites and retaining original
    reaction bounds
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Framework in which the generated community models are represented; community
    models can be directly used by other COBRA methods and reloaded from SBML
- name: Python 3
  role: Runtime environment in which PyCoMo is installed and the community model generation
    is executed
- name: SBML
  role: 'Input and output file format: member models are loaded from SBML and the
    generated community model is saved/loaded in SBML for sharing and reuse'
- name: openCOBRA
  role: File-format standard with which the PyCoMo-generated community models are
    compliant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:9c84daa1194aec246409e46216bf28a4f1bd6a2b38d5a4d598e7969d2c6b50e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  - openCOBRA
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# compartmentalized-model-generation

## Summary

Generate a genome-scale compartmentalized community metabolic model by loading member genome-scale SBML models (e.g. D. vulgaris, M. hungatei, M. barkeri from Koch et al. 2019) into PyCoMo, which attributes all metabolites, reactions, genes and compartments to their member of origin and retains original reaction bounds.

## When to use

Use when multiple genome-scale SBML member models must be merged into a single reusable community metabolic model with member-of-origin compartmentalization and openCOBRA/COBRApy compliance for growth-rate or abundance-profile simulations.

## When NOT to use

- You only have a single-organism metabolic model and no community members to merge — PyCoMo community model generation requires multiple member models
- You need to perform flux analysis or visualize an already-built community model rather than create one — use COBRApy methods or ScyNet for analysis/visualization of the existing SBML community model

## Inputs

- Member genome-scale metabolic models in SBML format (e.g. D. vulgaris, M. hungatei, M. barkeri models from the Koch et al. 2019 deposit)
- Community medium specification (set of available exchange metabolites, e.g. ethanol and CO2)

## Outputs

- Compartmentalized community metabolic model (COBRApy-compatible, openCOBRA-compliant) with member-of-origin attribution and preserved original reaction bounds
- Saved community model in SBML format (community_model.sbml), reloadable for reuse and downstream analysis

## How to apply

Install PyCoMo via pip (pip install pycomo) and load the member genome-scale SBML models with COBRApy/PyCoMo. Match medium-compartment exchange metabolites across member models by identical metabolite identifiers, and configure the community medium to the target exchange metabolites (e.g. ethanol and CO2 as in Koch et al. 2019 for the three-species biogas community). Generate the compartmentalized community model, which keeps every metabolite, reaction, gene and compartment attributable to its member of origin and retains the original reaction bounds of the input member models. Save the result in SBML format (e.g. community_model.sbml) so it can be reloaded as a COBRApy model and reused, shared, or analyzed for exchange metabolites and cross-feeding interactions.

## Related tools

- **PyCoMo** (Core Python 3 package that generates the compartmentalized community metabolic model from member SBML models, matching exchange metabolites and retaining original reaction bounds) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Framework in which the generated community models are represented; community models can be directly used by other COBRA methods and reloaded from SBML)
- **Python 3** (Runtime environment in which PyCoMo is installed and the community model generation is executed)
- **SBML** (Input and output file format: member models are loaded from SBML and the generated community model is saved/loaded in SBML for sharing and reuse)
- **openCOBRA** (File-format standard with which the PyCoMo-generated community models are compliant)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- The saved community_model.sbml reloads as a COBRApy model without errors
- All metabolites, reactions, genes and compartments in the community model remain attributable to their member of origin
- The original reaction bounds of all three input member models are retained in the community model
- Exchange metabolites in the medium compartment are matched by identical identifiers across member models, and the medium contains the configured exchange metabolites (ethanol and CO2)
- The generated model is compliant with openCOBRA file formats

## Limitations

- Medium-compartment exchange metabolites are matched across member models only by identical metabolite identifiers, so models using different IDs for the same metabolite will not be merged on those exchanges
- Community models are built to switch between fixed growth rate with variable abundance profile and fixed abundance profile with variable growth rate; they do not inherently model other coupling schemes
- ScyNet visualization of the resulting SBML requires specific format conventions (a shared exchange compartment named 'medium' or a shared_compartment_id parameter, and memberId_-prefixed metabolite and compartment IDs), which the model must fulfill

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [methods] community models save and load in SBML: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [methods] community models are COBRApy models: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] README: member attribution and original bounds: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [readme] README: retain original reaction bounds: "The community metabolic models generated by PyCoMo can switch between these two structures and retain the original reaction bounds of the input member models."
