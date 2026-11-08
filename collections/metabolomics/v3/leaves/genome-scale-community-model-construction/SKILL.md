---
name: genome-scale-community-model-construction
description: Use when the input is a set of genome-scale single-organism SBML metabolic
  models (e.g. AGORA members) and the goal is a reusable, member-attributable community
  metabolic model for FBA, exchange-metabolite, or cross-feeding analysis.
when_to_use_negative:
- The input models are not genome-scale single-organism metabolic models in SBML format,
  or the goal is dynamic/timestep-based community simulation rather than constraint-based
  (FBA/FVA) analysis.
- A coarse reduced interaction network visualization is the end goal only — the reduced
  network can be produced downstream with ScyNet/cy3sbml, but building a full PyCoMo
  model first is unnecessary overhead if no flux-based analysis is planned.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: 'Primary package: generates compartmentalized community metabolic models from
    single-organism SBML models and enables FBA, exchange-metabolite, and cross-feeding
    analysis'
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Underlying modeling framework; community models are COBRApy models directly
    usable by other COBRA methods for FBA/FVA
- name: SBML
  role: Input format for single-organism member models and the save/load format for
    built community models
- name: Python 3
  role: Implementation and runtime environment; PyCoMo is a python 3 package installable
    via pip and usable via CLI ('pycomo -h') or import
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:f8afd6426e12e36d4f3b69d718ed1950cd499f06bdebdd8792f50c68f19a65cc
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# genome-scale-community-model-construction

## Summary

Build compartmentalized genome-scale community metabolic models from single-organism SBML models using PyCoMo, which wraps COBRApy and outputs openCOBRA-compliant models that support fixed growth rate/variable abundance or fixed abundance/variable growth rate simulations.

## When to use

Use when the input is a set of genome-scale single-organism SBML metabolic models (e.g. AGORA members) and the goal is a reusable, member-attributable community metabolic model for FBA, exchange-metabolite, or cross-feeding analysis.

## When NOT to use

- The input models are not genome-scale single-organism metabolic models in SBML format, or the goal is dynamic/timestep-based community simulation rather than constraint-based (FBA/FVA) analysis.
- A coarse reduced interaction network visualization is the end goal only — the reduced network can be produced downstream with ScyNet/cy3sbml, but building a full PyCoMo model first is unnecessary overhead if no flux-based analysis is planned.

## Inputs

- Genome-scale single-organism metabolic models in SBML format (e.g. AGORA genome-scale models)
- Community member abundance profile (e.g. equal abundance)
- Target community size (number of member models, e.g. 10 or 40)

## Outputs

- Compartmentalized community metabolic model as a COBRApy/openCOBRA-compliant model object
- Community model saved in SBML format for sharing and reuse
- Predicted maximum community growth rate at a given abundance profile (via FBA)
- Set of all feasible exchange metabolites and cross-feeding interactions independent of abundance profile
- Runtime benchmark measurements per operation (e.g. runtime_benchmark.csv)

## How to apply

Install PyCoMo via 'pip install pycomo' (with COBRApy as the underlying framework) and load the single-organism SBML genome-scale models as community members. Build the compartmentalized community model with PyCoMo, which places member metabolites/reactions/genes/compartments in member-prefixed compartments and a shared exchange compartment, so every entity remains attributable to its member of origin and the original reaction bounds are retained. Verify the model is a COBRApy model compliant with current openCOBRA file formats, then either simulate at a fixed abundance profile to predict the maximum community growth rate via FBA, or switch to the structure allowing fixed growth rate with variable abundance profile. Optionally save/load the model in SBML for reuse, or enumerate all feasible exchange metabolites and cross-feeding interactions over the abundance-profile space. Benchmarking reference points on AGORA models on a single core: construction of a 40-member community takes about 11.3 min, FBA at equal abundance 2.3 min, and full exchange-metabolite enumeration 31.5 min (approximately 3 min for a 10-member community), with exchange-metabolite enumeration scaling quadratically with community size.

## Related tools

- **PyCoMo** (Primary package: generates compartmentalized community metabolic models from single-organism SBML models and enables FBA, exchange-metabolite, and cross-feeding analysis) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Underlying modeling framework; community models are COBRApy models directly usable by other COBRA methods for FBA/FVA)
- **SBML** (Input format for single-organism member models and the save/load format for built community models)
- **Python 3** (Implementation and runtime environment; PyCoMo is a python 3 package installable via pip and usable via CLI ('pycomo -h') or import)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- The constructed community model loads as a COBRApy model and all metabolites, reactions, genes and compartments are directly attributable to their member of origin.
- The built model retains the original reaction bounds of the input member models and can switch between fixed-growth-rate/variable-abundance and fixed-abundance/variable-growth-rate structures.
- FBA at an equal abundance profile on a 40-member AGORA community completes in the same order of magnitude as the reported 2.3 min, and model construction near the reported 11.3 min on a single core.
- Exchange-metabolite/cross-feeding enumeration on a 40-member community completes near 31.5 min, and approximately 3 min for 10 members, scaling quadratically with community size.
- The saved community model exports to and reloads from SBML without loss of member attribution.

## Limitations

- Runtime scales quadratically with community size for the calculation of all potential exchange metabolites, so very large communities become expensive to analyse for cross-feeding.
- Models built with PyCoMo must place all exchange metabolites in a shared exchange compartment and prefix member metabolite and compartment IDs (memberId_metaboliteId) for downstream tools such as ScyNet to attribute them correctly.
- No changelog was found for the repository, so version-specific behavior changes may be undocumented.

## Evidence

- [abstract] PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats.: "PyCoMo is a python package for quick and easy generation of genome-scale compartmentalized community metabolic models that are compliant with current openCOBRA file formats."
- [readme] The community models are COBRApy models and can therefor be directly used by other COBRA methods.: "The community models are COBRApy models and can therefor be directly used by other COBRA methods. It is also possible to save and load the community models in SBML format, allowing to share and reuse"
- [readme] PyCoMo community metabolic models are fully reusable.: "As also all metabolites, reactions, genes and compartments are directly attributable to their member of origin, PyCoMo community metabolic models are fully reusable."
- [methods] PyCoMo can be installed via pip: pip install pycomo: "PyCoMo can be installed via pip:

    pip install pycomo"
- [methods] PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile: "PyCoMo generates compartmentalized community metabolic models with a structure allowing simulations under fixed growth rate, but variable abundance profile"
