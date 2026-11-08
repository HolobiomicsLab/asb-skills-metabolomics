---
name: medium-exchange-metabolite-matching
description: Use when building a multi-species community metabolic model in PyCoMo
  and you must define which substrates the community can take up, or link exchange
  metabolites across member models to enable cross-feeding.
when_to_use_negative:
- You only need to analyse an existing community model (exchange feasibility, cross-feeding,
  growth rate prediction) rather than construct one with a defined medium
- Member models use non-identical metabolite identifiers for the same compounds, so
  ID-based matching would silently miss exchange links without prior identifier harmonization
- The task is visualization of an already-built community model — use ScyNet on the
  SBML file instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: PyCoMo
  role: Python 3 package used to load member SBML models, match exchange metabolites,
    set the medium, and generate the compartmentalized community model
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Community models produced are COBRApy models, usable directly with other COBRA
    methods and reloadable from the saved SBML
- name: openCOBRA
  role: File-format standard that generated community models are compliant with
- name: SBML
  role: Input format for member models and output format for the saved community model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:22bc1435f528a8be416472e21f0c7fc655bca239ced7c096a6d4620e3d9aec0f
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
n_claims: 10
---

# medium-exchange-metabolite-matching

## Summary

Configuring the community medium and matching exchange metabolites when building a compartmentalized community metabolic model with PyCoMo, so that shared exchange metabolites in the medium compartment are linked across member models. In the biogas case study this meant matching medium-compartment exchange metabolites by identical identifiers and supplying ethanol and CO2 as the available substrates per Koch et al. (2019).

## When to use

Use when building a multi-species community metabolic model in PyCoMo and you must define which substrates the community can take up, or link exchange metabolites across member models to enable cross-feeding.

## When NOT to use

- You only need to analyse an existing community model (exchange feasibility, cross-feeding, growth rate prediction) rather than construct one with a defined medium
- Member models use non-identical metabolite identifiers for the same compounds, so ID-based matching would silently miss exchange links without prior identifier harmonization
- The task is visualization of an already-built community model — use ScyNet on the SBML file instead

## Inputs

- Three member genome-scale metabolic models in SBML format (D. vulgaris, M. hungatei, M. barkeri from the Koch et al. 2019 deposit)
- Medium definition specifying available exchange metabolites (ethanol and CO2 in the case study)
- Member metabolite identifiers used for medium-compartment exchange matching

## Outputs

- Compartmentalized community metabolic model with all metabolites, reactions, genes and compartments attributable to their member of origin
- community_model.sbml — the community model saved in SBML format, reloadable as a COBRApy model

## How to apply

Load the member genome-scale models (e.g. D. vulgaris, M. hungatei, M. barkeri from SBML files in the Koch et al. 2019 deposit) into PyCoMo, then match medium-compartment exchange metabolites across members by identical metabolite identifiers. Configure the community medium to contain only the intended exchange metabolites — in the biogas case study, ethanol and CO2, as specified by Koch et al. (2019). Generate the compartmentalized community model with PyCoMo, which keeps all metabolites, reactions, genes and compartments attributable to their member of origin and retains the original reaction bounds of the input member models. Save the result in SBML format (e.g. community_model.sbml) so it can be reloaded as a COBRApy model and shared or reused.

## Related tools

- **PyCoMo** (Python 3 package used to load member SBML models, match exchange metabolites, set the medium, and generate the compartmentalized community model) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Community models produced are COBRApy models, usable directly with other COBRA methods and reloadable from the saved SBML)
- **openCOBRA** (File-format standard that generated community models are compliant with)
- **SBML** (Input format for member models and output format for the saved community model)

## Examples

```
pip install pycomo
```

## Evaluation signals

- The saved community_model.sbml reloads as a COBRApy model without errors
- All metabolites, reactions, genes and compartments in the community model remain attributable to their member of origin
- The original reaction bounds of all three input member models are retained in the community model
- The community medium contains exactly ethanol and CO2 as available exchange metabolites, per Koch et al. (2019)
- The generated model is compliant with openCOBRA file formats

## Limitations

- Exchange metabolite matching by identical IDs presumes member models share identifiers; divergent IDs would break matching
- The case study reproduces a previously published simplified biogas community; other media require domain-specific substrate definitions
- The generated structure is tied to fixed growth rate with variable abundance profile, or fixed abundance profile with variable community growth rate — it switches between these two modes only

## Evidence

- [methods] PyCoMo is a python 3 package for the creation and analysis of community metabolic models.: "PyCoMo is a python 3 package for the creation and analysis of community metabolic models."
- [methods] SBML save/load support: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [methods] Community models are COBRApy models: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] Member attribution and reaction bounds retained: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [abstract] openCOBRA compliance: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
