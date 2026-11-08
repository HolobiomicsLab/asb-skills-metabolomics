---
name: community-model-structure-reconstruction
description: Use when individual genome-scale SBML member models must be combined
  into a single reusable community metabolic model whose flux bounds depend on member
  abundance, and the current SBML standard cannot encode variable bounds directly.
when_to_use_negative:
- When flux bounds are fixed and independent of abundance — a plain merged SBML model
  without bound-free linearization suffices
- When the goal is only visualization of an existing community model — ScyNet operates
  on already-built PyCoMo SBML community models rather than reconstructing them
- When the input is not per-member SBML genome-scale models (e.g. already merged community
  models without member-attributable IDs)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model, performs the bound-free
    transformation, and saves/loads the community model in SBML format
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Hosts the resulting community models so they can be directly used by other
    COBRA methods and verified to reproduce original reaction bounds
- name: SBML
  role: Exchange format for saving and reloading the bound-free community model (bound_free_member_model.xml)
- name: Python 3
  role: Runtime environment in which PyCoMo and COBRApy perform the model reconstruction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:8682bd4772f248f2710cb0dc2309045ad93669c0a5a9aa9dd80d8ea34730b207
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
yes_rate: 1.0
n_claims: 11
---

# community-model-structure-reconstruction

## Summary

Reconstruct a genome-scale, compartmentalized community metabolic model with PyCoMo by merging member SBML models into an openCOBRA/COBRApy-compliant community structure, using bound-free linearization (dummy metabolites encoding flux bounds) so that effective bounds scale with member abundance while remaining SBML-expressible.

## When to use

Use when individual genome-scale SBML member models must be combined into a single reusable community metabolic model whose flux bounds depend on member abundance, and the current SBML standard cannot encode variable bounds directly.

## When NOT to use

- When flux bounds are fixed and independent of abundance — a plain merged SBML model without bound-free linearization suffices
- When the goal is only visualization of an existing community model — ScyNet operates on already-built PyCoMo SBML community models rather than reconstructing them
- When the input is not per-member SBML genome-scale models (e.g. already merged community models without member-attributable IDs)

## Inputs

- Genome-scale member metabolic models in SBML format
- Reference abundance profile / mass fractions for community members

## Outputs

- Bound-free community-structured model exported as SBML (e.g. bound_free_member_model.xml), loadable via COBRApy
- A COBRApy community model with metabolites, reactions, genes and compartments attributable to their member of origin

## How to apply

Load each genome-scale member model from its SBML file into COBRApy in a Python 3 environment. For each reaction, replace fixed lower/upper flux bounds by introducing a compartment-specific dummy metabolite whose stoichiometric coefficient encodes the original bound value; couple each dummy metabolite to a producing or consuming reaction whose flux equals the member's mass fraction, so the effective bound scales with abundance instead of being hard-coded. Verify in COBRApy that the transformed model reproduces the original bounds at the reference mass fraction and that all metabolites, reactions, genes and compartments remain attributable to their member of origin. Export the community model with PyCoMo's save functionality to SBML (e.g. bound_free_member_model.xml); PyCoMo community models switch between fixed-growth-rate/variable-abundance and fixed-abundance/variable-growth-rate structures while retaining the original reaction bounds of the input member models.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model, performs the bound-free transformation, and saves/loads the community model in SBML format) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Hosts the resulting community models so they can be directly used by other COBRA methods and verified to reproduce original reaction bounds)
- **SBML** (Exchange format for saving and reloading the bound-free community model (bound_free_member_model.xml))
- **Python 3** (Runtime environment in which PyCoMo and COBRApy perform the model reconstruction)

## Examples

```
pip install pycomo
```

## Evaluation signals

- The exported SBML file loads back via COBRApy/SBML without errors
- FBA on the transformed (bound-free) model matches the original model's solution space at the reference abundance profile
- The transformed model reproduces the original reaction bounds at the reference mass fraction
- All metabolites, reactions, genes and compartments in the community model are attributable to the member of origin (memberId_ prefixed IDs)

## Limitations

- The current SBML standard does not accommodate variable flux bounds, which is why the bound-free dummy-metabolite workaround is required
- The bound-free structure encodes bounds via stoichiometric coefficients of dummy metabolites coupled to reactions whose fluxes equal the member's mass fraction; correctness holds only at the reference abundance profile and must be re-validated if abundance profiles change

## Evidence

- [full_text] PyCoMo converts each genome-scale model into an equivalent bound-free form where flux bounds become stoichiometric coefficients of newly introduced dummy metabolites: [evidence span withheld — anchor status: not_found]
- [methods] It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo.: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [methods] PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
