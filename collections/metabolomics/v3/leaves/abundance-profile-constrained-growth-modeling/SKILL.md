---
name: abundance-profile-constrained-growth-modeling
description: Use when you need to predict community growth or feasible compositions
  at a given abundance profile, or enumerate all cross-feeding interactions independent
  of abundance, from single-organism genome-scale SBML models.
when_to_use_negative:
- The input is not a genome-scale single-organism SBML model but raw omics data or
  a pre-built community model without member-prefixed IDs and a shared exchange compartment
- The goal is single-species (non-community) metabolic analysis, where PyCoMo's compartmentalized
  community structure and exchange-compartment machinery add no benefit
- A visualization of an existing community model is needed instead — ScyNet network
  reduction/contextualization, not model construction, is the right tool
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Generates the compartmentalized community metabolic model from member SBML
    models and performs FBA and exchange-metabolite/cross-feeding analysis
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Underlying modeling framework; community models are COBRApy models directly
    usable by other COBRA methods
- name: efmtool
  role: Optional dependency for EFM enumeration installed via 'pip install pycomo[efm]'
- name: ScyNet
  role: Downstream Cytoscape visualization of the PyCoMo-built community network reduced
    to members and exchange metabolites
  repo: univieCUBE/ScyNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:7cbc0b1d4f286474fef76c411b1c3361e15a8869cb66c8af6915e9ecb12c89f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - ScyNet
  - efmtool
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# abundance-profile-constrained-growth-modeling

## Summary

Build and simulate compartmentalized community metabolic models with PyCoMo/COBRApy that switch between fixed growth rate with variable abundance profile and fixed abundance profile with variable community growth rate, enabling FBA growth-rate prediction and enumeration of exchange metabolites and cross-feeding interactions.

## When to use

Use when you need to predict community growth or feasible compositions at a given abundance profile, or enumerate all cross-feeding interactions independent of abundance, from single-organism genome-scale SBML models.

## When NOT to use

- The input is not a genome-scale single-organism SBML model but raw omics data or a pre-built community model without member-prefixed IDs and a shared exchange compartment
- The goal is single-species (non-community) metabolic analysis, where PyCoMo's compartmentalized community structure and exchange-compartment machinery add no benefit
- A visualization of an existing community model is needed instead — ScyNet network reduction/contextualization, not model construction, is the right tool

## Inputs

- Single-organism genome-scale metabolic models in SBML format (e.g., AGORA models)
- Community member abundance profile (e.g., equal abundance across members)
- Target community growth rate (for fixed-growth/variable-abundance simulations)

## Outputs

- Compartmentalized community metabolic COBRApy model (reusable, saved as SBML)
- Maximum community growth rate at the given abundance profile
- Feasible community compositions at a given growth rate
- List of all exchange metabolites and cross-feeding interactions, independent of the abundance profile

## How to apply

Install PyCoMo via pip (optionally with the efm flag for efmtool-based EFM enumeration) and load AGORA single-organism SBML models as member models. Build a compartmentalized community metabolic model in which all metabolites, reactions, genes, and compartments remain attributable to their member of origin, so the model can switch between fixed-growth/variable-abundance and fixed-abundance/variable-growth structures while retaining the original reaction bounds. Run FBA at an equal member abundance profile (e.g., AGORA communities of up to 40 members) to compute the maximum community growth rate, and use exchange-flux analysis to enumerate all feasible exchange metabolites and cross-feeding interactions over the whole space of growth rate and abundance profiles. Reference runtimes on a single core at 40 members are 11.3 min for model construction, 2.3 min for FBA, and 31.5 min for exchange-metabolite enumeration (quadratic scaling for the latter); models can be saved/loaded in SBML format for reuse.

## Related tools

- **PyCoMo** (Generates the compartmentalized community metabolic model from member SBML models and performs FBA and exchange-metabolite/cross-feeding analysis) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Underlying modeling framework; community models are COBRApy models directly usable by other COBRA methods)
- **efmtool** (Optional dependency for EFM enumeration installed via 'pip install pycomo[efm]')
- **ScyNet** (Downstream Cytoscape visualization of the PyCoMo-built community network reduced to members and exchange metabolites) — univieCUBE/ScyNet

## Examples

```
pip install pycomo
```

## Evaluation signals

- At 40 AGORA members on a single core, runtimes match the reported 11.3 min (construction), 2.3 min (FBA), and 31.5 min (exchange metabolites) within the same order of magnitude
- Exchange-metabolite enumeration runtime scales quadratically with community size (≈3 min at 10 members)
- Every metabolite, reaction, gene, and compartment in the community model is attributable to its member of origin (member-prefixed IDs), and original reaction bounds are retained
- The built community model loads as a COBRApy model and FBA returns a positive maximum community growth rate at the equal-abundance profile
- The community model round-trips through SBML save/load without loss of the shared exchange compartment or member attributions

## Limitations

- Cross-feeding enumeration does not scale linearly: it scales quadratically with community size (31.5 min at 40 members on a single core)
- Building community models from large sets of genome-scale members is time-consuming (11.3 min for 40 members on a single core)
- PyCoMo is a python 3 package; environments must support Python 3 and COBRApy-compatible openCOBRA file formats

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [methods] The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles.: "The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles."
- [methods] PyCoMo can be installed via pip:

    pip install pycomo: "PyCoMo can be installed via pip:

    pip install pycomo"
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods. It is also possible to save and load the community models in SBML format: "The community models are COBRApy models and can therefor be directly used by other COBRA methods. It is also possible to save and load the community models in SBML format"
- [results] The runtime of the calculation of all potential exchange metabolites scales quadratically with community size: [evidence span withheld — anchor status: not_found]
