---
name: sbml-model-conversion-and-validation
description: Use when member models carry fixed lower/upper flux bounds that must
  scale with community member abundance, or when a COBRApy community model must be
  saved/loaded as SBML and remain fully reusable.
when_to_use_negative:
- The member model contains no fixed flux bounds that need to become abundance-dependent
  — the bound-free transformation adds unnecessary dummy metabolites
- The target workflow requires a community model with fixed abundance profile and
  variable growth rate rather than variable abundance — PyCoMo's structure switching
  supports this without re-encoding bounds
- The file must remain a plain member model, not a community model with memberId_-prefixed
  metabolite and compartment IDs as required for ScyNet/PyCoMo community models
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Performs the bound-free linearization of abundance-dependent flux bounds via
    dummy metabolites and saves/loads the community model in SBML format
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the member SBML models, verifies the transformed model reproduces original
    bounds at the reference mass fraction, and runs FBA for validation
- name: Python 3
  role: Runtime environment in which PyCoMo is imported as a package or used via its
    command line interface
- name: SBML
  role: File format for saving and reloading the bound-free community models, enabling
    sharing and reuse
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:e78ca42143351217f3a543a23e70a583b30a2256d39ec03dc6b0d3746766dd81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sbml-model-conversion-and-validation

## Summary

Convert genome-scale member metabolic models into a bound-free, SBML-compliant community form with PyCoMo, where fixed flux bounds become stoichiometric coefficients of dummy metabolites scaled by member abundance, then validate round-trip via SBML export/load in COBRApy.

## When to use

Use when member models carry fixed lower/upper flux bounds that must scale with community member abundance, or when a COBRApy community model must be saved/loaded as SBML and remain fully reusable.

## When NOT to use

- The member model contains no fixed flux bounds that need to become abundance-dependent — the bound-free transformation adds unnecessary dummy metabolites
- The target workflow requires a community model with fixed abundance profile and variable growth rate rather than variable abundance — PyCoMo's structure switching supports this without re-encoding bounds
- The file must remain a plain member model, not a community model with memberId_-prefixed metabolite and compartment IDs as required for ScyNet/PyCoMo community models

## Inputs

- Genome-scale member metabolic model in SBML format (e.g. member_model.xml)
- Reference abundance profile (member mass fractions) for bound scaling
- Python 3 environment with PyCoMo and COBRApy installed

## Outputs

- Bound-free community-structured model exported as SBML (bound_free_member_model.xml)
- Validation check that FBA on the transformed model matches the original model's solution space at the reference abundance profile

## How to apply

Load each genome-scale member model from its SBML file into COBRApy in a Python 3 environment. For every reaction, replace the fixed lower/upper flux bounds by introducing a compartment-specific dummy metabolite whose stoichiometric coefficient encodes the original bound value. Couple each dummy metabolite to a producing or consuming reaction whose flux equals the member's mass fraction, so the effective bound scales with abundance instead of being hard-coded — this keeps the model compliant with the current SBML standard, which does not accommodate variable flux bounds. Verify in COBRApy that the transformed model reproduces the original reaction bounds at the reference mass fraction and that all metabolites, reactions, genes and compartments remain attributable to their member of origin. Export the bound-free community-structured model via PyCoMo's save functionality to bound_free_member_model.xml; the resulting models are COBRApy models and can be directly used by other COBRA methods.

## Related tools

- **PyCoMo** (Performs the bound-free linearization of abundance-dependent flux bounds via dummy metabolites and saves/loads the community model in SBML format) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the member SBML models, verifies the transformed model reproduces original bounds at the reference mass fraction, and runs FBA for validation)
- **Python 3** (Runtime environment in which PyCoMo is imported as a package or used via its command line interface)
- **SBML** (File format for saving and reloading the bound-free community models, enabling sharing and reuse)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- Exported bound_free_member_model.xml loads back via COBRApy/SBML without errors
- FBA on the transformed model matches the original model's solution space at the reference abundance profile
- Transformed model reproduces the original reaction bounds exactly when evaluated at the reference mass fraction
- All metabolites, reactions, genes and compartments remain attributable to their member of origin (memberId_-prefixed IDs)
- Resulting model is compliant with current openCOBRA file formats

## Limitations

- The current SBML standard does not accommodate variable flux bounds, hence the workaround of encoding bounds as stoichiometric coefficients of dummy metabolites
- Loading large community models into downstream tools can be slow — the ScyNet README warns importing community metabolic model SBML files 'can take a couple of minutes, depending on your hardware', especially for >10 members

## Evidence

- [methods] It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo.: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [results] PyCoMo converts each genome-scale model into an equivalent bound-free form where flux bounds become stoichiometric coefficients of newly introduced dummy metabolites.: [evidence span withheld — anchor status: not_found]
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
