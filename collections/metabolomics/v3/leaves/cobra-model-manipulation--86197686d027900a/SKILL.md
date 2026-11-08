---
name: cobra-model-manipulation
description: Use when input consists of genome-scale metabolic models (e.g. SBML files
  from Koch et al. 2019 deposits) that must be combined into a compartmentalized community
  model, queried, or saved/reloaded in SBML format for COBRA analysis.
when_to_use_negative:
- The task is single-species FBA on an existing model that needs no community compartmentalization
  — plain COBRApy suffices without PyCoMo model generation.
- Member SBML models use inconsistent metabolite identifiers for shared exchange metabolites,
  so identical-ID matching would fail to link cross-feeding exchanges.
- The goal is only network visualization of an already-built community model — ScyNet
  with cy3sbml handles that without regenerating the model.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: PyCoMo
  role: Generates compartmentalized community metabolic models from member SBML models,
    openCOBRA-compliant, and saves/loads them in SBML format
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: 'Format/analysis backbone: the community models produced are COBRApy models
    usable directly by other COBRA methods'
- name: Python 3
  role: Runtime environment; PyCoMo is a python 3 package installed via pip
- name: SBML
  role: Input member model format and output format for saving/reloading community
    models
- name: openCOBRA
  role: File-format compliance target for the generated community models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:054f7d4d2ab6f53eaef78a3e865b4028568833759a6d55d035887af31e4a505c
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
yes_rate: 0.9375
n_claims: 16
---

# cobra-model-manipulation

## Summary

Construct, transform, and save constraint-based metabolic models (including compartmentalized community models) using COBRApy-compatible tooling such as PyCoMo, loading SBML member models, generating openCOBRA-compliant community models, and reloading saved SBML for downstream COBRA methods.

## When to use

Use when input consists of genome-scale metabolic models (e.g. SBML files from Koch et al. 2019 deposits) that must be combined into a compartmentalized community model, queried, or saved/reloaded in SBML format for COBRA analysis.

## When NOT to use

- The task is single-species FBA on an existing model that needs no community compartmentalization — plain COBRApy suffices without PyCoMo model generation.
- Member SBML models use inconsistent metabolite identifiers for shared exchange metabolites, so identical-ID matching would fail to link cross-feeding exchanges.
- The goal is only network visualization of an already-built community model — ScyNet with cy3sbml handles that without regenerating the model.

## Inputs

- Member genome-scale metabolic models in SBML format (e.g. from the Koch et al. 2019 biogas community deposit)
- Medium specification as a list of available exchange metabolites (e.g. ethanol and CO2)
- Member metabolite identifiers for exchange metabolite matching

## Outputs

- Compartmentalized community metabolic model as a COBRApy model, with member-of-origin attribution and original reaction bounds preserved
- Community model saved in SBML format (e.g. community_model.sbml)

## How to apply

Install PyCoMo (pip install pycomo), then load the member genome-scale SBML models (e.g. D. vulgaris, M. hungatei, M. barkeri) into PyCoMo. Match medium-compartment exchange metabolites across members by identical metabolite identifiers, and configure the community medium to the desired exchange metabolites (e.g. ethanol and CO2 per Koch et al. 2019). Generate the compartmentalized community model, which keeps all metabolites, reactions, genes and compartments attributable to their member of origin and retains the original reaction bounds of the input member models. Save the result as SBML (community_model.sbml); because the generated models are COBRApy models, they can be reloaded and used directly by other COBRA methods such as FBA, loopless FVA, or max growth rate computation.

## Related tools

- **PyCoMo** (Generates compartmentalized community metabolic models from member SBML models, openCOBRA-compliant, and saves/loads them in SBML format) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Format/analysis backbone: the community models produced are COBRApy models usable directly by other COBRA methods)
- **Python 3** (Runtime environment; PyCoMo is a python 3 package installed via pip)
- **SBML** (Input member model format and output format for saving/reloading community models)
- **openCOBRA** (File-format compliance target for the generated community models)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- The saved community_model.sbml reloads as a COBRApy model without errors.
- All metabolites, reactions, genes and compartments in the community model remain attributable to their member of origin.
- The original reaction bounds of all three input member models are retained in the community model.
- Exchange metabolites in the medium compartment match across member models by identical identifiers.
- The community model is openCOBRA-compliant and supports simulations under fixed growth rate with variable abundance profile (and vice versa).

## Limitations

- Exchange metabolite matching relies on identical metabolite IDs between member models; inconsistent ID conventions across depositions will break cross-feeding linkage.
- Installation of optional EFM enumeration / CycleBreaker dependencies requires extra flags (pip install pycomo[efm] or installing efmtool).
- The PyCoMo GitHub repository shows no changelog ('No changelog found'), so version-specific behavior must be checked directly.

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [methods] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods. It is also possible to save and load the community models in SBML format"
- [readme] PyCoMo community metabolic models are fully reusable.: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [readme] PyCoMo can be installed via pip: "PyCoMo can be installed via pip:
``npip install pycomo
"
