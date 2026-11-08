---
name: cross-feeding-exchange-metabolite-enumeration
description: Use when a genome-scale community metabolic model (e.g. AGORA SBML member
  models) needs systematic enumeration of which metabolites members can exchange and
  which cross-feeding interactions are feasible, without fixing an abundance profile.
when_to_use_negative:
- You need abundance-profile-specific exchange predictions at a fixed growth rate
  only (use PyCoMo's max growth rate / FBA prediction on the community model instead)
- Input is a single-organism model or the analysis does not require member-to-member
  exchange/cross-feeding interactions
- You only want to visualize an already-built community model without re-enumerating
  exchanges (use ScyNet on the existing PyCoMo-generated SBML)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: PyCoMo
  role: Builds the compartmentalized community metabolic model and performs the enumeration
    of all feasible exchange metabolites and cross-feeding interactions
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Underlying modeling framework; PyCoMo community models are COBRApy models
    directly usable by other COBRA methods such as FBA and FVA
- name: SBML
  role: File format for saving/loading and sharing the community models built with
    PyCoMo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:1751786bbe7ed74766744be65682fea209bc25c054b2fda39f2717cbea890827
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
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# cross-feeding-exchange-metabolite-enumeration

## Summary

Enumerates all feasible exchange metabolites and cross-feeding interactions in a compartmentalized community metabolic model built with PyCoMo, independent of the abundance profile, by exchange-flux analysis over the full space of growth-rate and abundance profiles.

## When to use

Use when a genome-scale community metabolic model (e.g. AGORA SBML member models) needs systematic enumeration of which metabolites members can exchange and which cross-feeding interactions are feasible, without fixing an abundance profile.

## When NOT to use

- You need abundance-profile-specific exchange predictions at a fixed growth rate only (use PyCoMo's max growth rate / FBA prediction on the community model instead)
- Input is a single-organism model or the analysis does not require member-to-member exchange/cross-feeding interactions
- You only want to visualize an already-built community model without re-enumerating exchanges (use ScyNet on the existing PyCoMo-generated SBML)

## Inputs

- Genome-scale single-organism SBML metabolic models (e.g. AGORA models) as community members
- Equal-abundance or user-defined community member abundance profile
- Optional FBA flux vector or FVA flux ranges as tab-separated files (reaction_id, flux; or reaction_id, min_flux, max_flux) for downstream contextualization

## Outputs

- Set of all feasible exchange metabolites in the shared medium compartment
- Enumerated cross-feeding interactions between community members
- Optionally an SBML community model reusable via PyCoMo save/load
- runtime_benchmark.csv table when the enumeration is timed across community sizes

## How to apply

Install PyCoMo via pip ('pip install pycomo', optionally 'pip install pycomo[efm]' to include efmtool dependencies) and load genome-scale single-organism SBML models with COBRApy. Build a compartmentalized community metabolic model in which all metabolites, reactions, genes and compartments remain attributable to their member of origin and a shared medium compartment holds the exchange metabolites. Run PyCoMo's analysis of feasible exchange metabolites and cross-feeding interactions over the whole space of growth rate and abundance profiles, so results are independent of any single abundance profile. The output models are COBRApy models, so downstream COBRA methods (FBA, FVA, loopless FVA) can be applied directly. Expect the enumeration to scale quadratically with community size — at 40 AGORA members it took 31.5 min versus about 3 min at 10 members on a single core — so budget runtime accordingly. Models can be saved/reloaded in SBML format, and flux data (FBA single values or FVA ranges in tab-separated files with columns reaction_id/flux or reaction_id/min_flux/max_flux) can be used to contextualize interactions.

## Related tools

- **PyCoMo** (Builds the compartmentalized community metabolic model and performs the enumeration of all feasible exchange metabolites and cross-feeding interactions) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Underlying modeling framework; PyCoMo community models are COBRApy models directly usable by other COBRA methods such as FBA and FVA)
- **SBML** (File format for saving/loading and sharing the community models built with PyCoMo)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- The enumeration result is independent of the chosen abundance profile (predictions hold across the whole space of growth-rate and abundance profiles)
- Runtime scaling with community size is roughly quadratic: ~3 min at 10 members and 31.5 min at 40 AGORA members on a single core is the reference (same order of magnitude accepted)
- All metabolites, reactions, genes and compartments in the output community model are directly attributable to their member of origin, and a shared exchange/medium compartment contains the exchange metabolites
- The community model retains the original reaction bounds of the input member models and can be reloaded losslessly from SBML
- Flux contextualization files parse with the required columns (reaction_id/flux for FBA; reaction_id/min_flux/max_flux for FVA)

## Limitations

- Enumeration runtime scales quadratically with community size, making very large communities expensive on a single core
- Enumeration must be repeated or reinterpreted if a specific abundance-profile-conditioned exchange prediction is required, since the default analysis is abundance-profile-independent
- The optional EFM enumeration / CycleBreaker features require extra installation flags ('pip install pycomo[efm]' or efmtool) and are not installed by default

## Evidence

- [abstract] PyCoMo predicts all exchange metabolites and cross-feeding interactions independent of the abundance profile: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
- [methods] Community models analyzed over the whole profile space: "The community models can be analysed with PyCoMo to predict all feasible exchange metabolites and cross-feeding interactions, for the whole space of growth rate and abundance profiles."
- [other] Runtime scaling of the enumeration: [evidence span withheld — anchor status: not_found]
- [readme] Installation of PyCoMo with optional EFM dependencies: "PyCoMo can be installed via pip:

    pip install pycomo"
