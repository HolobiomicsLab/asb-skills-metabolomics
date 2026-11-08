---
name: constraint-based-metabolic-modeling
description: Use when combining genome-scale member models (SBML) into a community
  model where flux bounds must vary with member abundance, and the SBML/openCOBRA
  format requires bounds to be represented implicitly rather than as variable parameters.
when_to_use_negative:
- The task involves a single static member model with fixed bounds — no abundance
  coupling is needed and the original SBML bounds can be used directly.
- The target format natively supports variable flux bounds, making the dummy-metabolite
  workaround unnecessary.
- The analysis goal is not constraint-based (e.g. kinetic or thermodynamic modeling),
  where COBRApy/openCOBRA FBA semantics do not apply.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: PyCoMo
  role: Converts member models into the bound-free form with dummy-metabolite-encoded
    bounds and exports the community model to SBML
  repo: http://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Loads the SBML member models, performs the bound-free transformation checks,
    and runs FBA to compare solution spaces
- name: Python 3
  role: Runtime environment in which the PyCoMo/COBRApy transformation is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:ccbbc552f99674f8356022bd52343750988d1d50e94373a1ddc8e9a7fc0251de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# constraint-based-metabolic-modeling

## Summary

Builds SBML-compliant community metabolic models in which abundance-dependent flux bounds are encoded as stoichiometric coefficients of dummy metabolites (a 'bound-free' form), so bounds scale with member mass fraction instead of being hard-coded, since the SBML standard cannot represent variable flux bounds.

## When to use

Use when combining genome-scale member models (SBML) into a community model where flux bounds must vary with member abundance, and the SBML/openCOBRA format requires bounds to be represented implicitly rather than as variable parameters.

## When NOT to use

- The task involves a single static member model with fixed bounds — no abundance coupling is needed and the original SBML bounds can be used directly.
- The target format natively supports variable flux bounds, making the dummy-metabolite workaround unnecessary.
- The analysis goal is not constraint-based (e.g. kinetic or thermodynamic modeling), where COBRApy/openCOBRA FBA semantics do not apply.

## Inputs

- Genome-scale member metabolic model in SBML format
- Member mass-fraction (abundance) profile at which original bounds should be reproduced
- Python 3 environment with PyCoMo and COBRApy installed (pip install pycomo)

## Outputs

- bound_free_member_model.xml: bound-free community-structured member model in SBML format
- FBA solution space at the reference abundance profile, matching the original bounded model

## How to apply

Load each genome-scale member model from its SBML file into COBRApy within a Python 3 environment. For every reaction, replace the fixed lower/upper flux bounds by introducing a compartment-specific dummy metabolite whose stoichiometric coefficient encodes the original bound value. Couple each dummy metabolite to a newly introduced producing or consuming reaction whose flux equals the member's mass fraction, so the effective bound scales with abundance rather than being hard-coded. Verify in COBRApy that the transformed model reproduces the original reaction bounds at the reference mass fraction and that all metabolites, reactions, genes and compartments remain attributable to the member of origin. Export the bound-free community-structured model to SBML with PyCoMo's save functionality, yielding bound_free_member_model.xml.

## Related tools

- **PyCoMo** (Converts member models into the bound-free form with dummy-metabolite-encoded bounds and exports the community model to SBML) — http://github.com/univieCUBE/PyCoMo
- **COBRApy** (Loads the SBML member models, performs the bound-free transformation checks, and runs FBA to compare solution spaces)
- **Python 3** (Runtime environment in which the PyCoMo/COBRApy transformation is executed)

## Examples

```
pip install pycomo
```

## Evaluation signals

- The exported bound_free_member_model.xml loads back via COBRApy/SBML without errors.
- FBA on the transformed model matches the original model's solution space at the reference abundance profile.
- The transformed model reproduces the original reaction bounds exactly at the reference mass fraction.
- All metabolites, reactions, genes and compartments remain directly attributable to their member of origin.
- The resulting community model is compliant with current openCOBRA file formats.

## Limitations

- The current SBML standard does not accommodate variable flux bounds, so abundance dependence can only be represented indirectly via dummy metabolites and mass-fraction-coupled reactions.
- PyCoMo community models simulate under fixed growth rate with a variable abundance profile, or fixed abundance profile with variable community growth rate; the model structure assumes one of these regimes.
- Large genome-scale models (>10 members) can be slow to load downstream (e.g. in cy3sbml/Cytoscape).

## Evidence

- [readme] PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
- [readme] The community metabolic models generated by PyCoMo can switch between these two structures and retain the original reaction bounds of the input member models.: "The community metabolic models generated by PyCoMo can switch between these two structures and retain the original reaction bounds of the input member models."
- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [methods] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
