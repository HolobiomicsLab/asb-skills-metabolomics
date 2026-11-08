---
name: flux-bound-encoding-via-dummy-metabolites
description: Use when building community metabolic models where member flux bounds
  must scale with each member's abundance, and the SBML format's inability to represent
  variable flux bounds would otherwise force hard-coded bounds.
when_to_use_negative:
- If flux bounds never need to vary with abundance (a fixed abundance profile suffices),
  hard-coded bounds in the original SBML are simpler and adequate
- If the target model format supports variable flux bounds natively, the bound-free
  linearization adds unnecessary dummy metabolites and reactions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Performs the bound-free linearization, converts member models into the community
    structure with dummy metabolites encoding flux bounds, and exports the result
    to SBML
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the member SBML models, validates that the transformed model reproduces
    original bounds, and runs FBA checks on the bound-free model
- name: SBML
  role: File format used to load input member models and save the bound-free community
    model for sharing and reuse
- name: Python 3
  role: Language environment in which PyCoMo and COBRApy execute the transformation
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:762ba6502cfa30602c96da19cf92c6e0826dd6f4de9e8e7b2504ae6f8ba772b4
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
yes_rate: 0.875
n_claims: 8
---

# flux-bound-encoding-via-dummy-metabolites

## Summary

PyCoMo converts each genome-scale member model into an equivalent bound-free form in which fixed lower/upper flux bounds are encoded as stoichiometric coefficients of compartment-specific dummy metabolites, coupled to reactions whose fluxes equal the member's mass fraction. This lets abundance-dependent bounds remain SBML-compliant while making effective flux limits scale dynamically with community composition.

## When to use

Use when building community metabolic models where member flux bounds must scale with each member's abundance, and the SBML format's inability to represent variable flux bounds would otherwise force hard-coded bounds.

## When NOT to use

- If flux bounds never need to vary with abundance (a fixed abundance profile suffices), hard-coded bounds in the original SBML are simpler and adequate
- If the target model format supports variable flux bounds natively, the bound-free linearization adds unnecessary dummy metabolites and reactions

## Inputs

- Genome-scale member metabolic model in SBML format
- Reference mass fraction (abundance profile) for validation
- Python 3 environment with PyCoMo and COBRApy installed (pip install pycomo)

## Outputs

- Bound-free member model (e.g. bound_free_member_model.xml) in SBML format with flux bounds encoded as dummy-metabolite stoichiometry
- Community-structured model whose fluxes are attributable to each member of origin

## How to apply

Load each genome-scale member model from its SBML file into COBRApy within a Python 3 environment. For every reaction, replace the fixed lower/upper flux bounds by introducing a compartment-specific dummy metabolite whose stoichiometric coefficient encodes the original bound value. Couple each dummy metabolite to a newly introduced producing or consuming reaction whose flux equals the member's mass fraction, so the effective bound scales with abundance rather than being hard-coded. Verify in COBRApy that the transformed model reproduces the original reaction bounds at the reference mass fraction and that all metabolites, reactions, genes and compartments remain attributable to their member of origin. Export the bound-free community-structured model to SBML with PyCoMo's save functionality.

## Related tools

- **PyCoMo** (Performs the bound-free linearization, converts member models into the community structure with dummy metabolites encoding flux bounds, and exports the result to SBML) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the member SBML models, validates that the transformed model reproduces original bounds, and runs FBA checks on the bound-free model)
- **SBML** (File format used to load input member models and save the bound-free community model for sharing and reuse)
- **Python 3** (Language environment in which PyCoMo and COBRApy execute the transformation workflow)

## Examples

```
import pycomo; member_models = pycomo.load_member_models(['bound_free_member_model.xml']); community = pycomo.CommunityModel(member_models, 'community'); community.save('bound_free_community_model.xml')
```

## Evaluation signals

- FBA on the transformed model at the reference abundance profile reproduces the original model's solution space
- The exported bound_free_member_model.xml loads back via COBRApy/SBML without errors
- Effective reaction bounds at the reference mass fraction match the original hard-coded lower/upper bounds
- All metabolites, reactions, genes and compartments remain attributable to their member of origin
- Changing the member's mass fraction proportionally scales the effective bounds (the defining behavior of the dummy-metabolite coupling)

## Limitations

- The approach exists because the current SBML standard does not accommodate variable flux bounds; it is a workaround, not a native representation
- The model structure assumes simulations under either fixed growth rate with variable abundance profile, or fixed abundance profile with variable community growth rate
- Correct re-loading requires the PyCoMo naming conventions (member-prefixed metabolite and compartment IDs, shared exchange compartment) to be preserved

## Evidence

- [abstract] bound-free conversion: [evidence span withheld — anchor status: not_found]
- [methods] SBML save/load: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [readme] COBRApy models: "The community metabolic models generated by PyCoMo can switch between these two structures and retain the original reaction bounds of the input member models."
- [readme] structure flexibility: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
