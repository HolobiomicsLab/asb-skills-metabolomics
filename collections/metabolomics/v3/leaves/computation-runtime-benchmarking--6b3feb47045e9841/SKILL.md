---
name: computation-runtime-benchmarking
description: Use when measuring how PyCoMo community model construction, FBA, and
  exchange-metabolite/cross-feeding analysis runtimes scale with community size on
  AGORA SBML models up to 40 members.
when_to_use_negative:
- The goal is accuracy of growth-rate or cross-feeding predictions rather than runtime/scaling
  measurements
- Community models are not built from AGORA SBML member models or the target scale
  exceeds the single-core, up-to-40-member benchmarking scope
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: PyCoMo
  role: Builds the compartmentalized community metabolic models and performs FBA and
    exchange-metabolite/cross-feeding analysis being timed
  repo: https://github.com/univieCUBE/PyCoMo
- name: COBRApy
  role: Underlying modeling framework; the PyCoMo community models are COBRApy models
    used directly for FBA
- name: Python 3
  role: Runtime environment in which PyCoMo is installed via pip and benchmarks are
    executed
- name: SBML
  role: Input format of the AGORA single-organism member models and storage format
    for built community models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae153
    title: 'PyCoMo: a python package for community metabolic model creation and analysis'
  source_content_hash: sha256:179d2e715490f03c9035f02bee742e4214dbc8857fc5da3d58a916c394940caf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PyCoMo
  tools:
  - COBRApy
  - PyCoMo
  - Python 3
  - SBML
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# computation-runtime-benchmarking

## Summary

Benchmark wall-clock runtimes of PyCoMo community metabolic model construction, FBA at equal member abundance, and enumeration of all potential exchange metabolites on AGORA genome-scale models up to 40 members on a single core, capturing how each step scales with community size.

## When to use

Use when measuring how PyCoMo community model construction, FBA, and exchange-metabolite/cross-feeding analysis runtimes scale with community size on AGORA SBML models up to 40 members.

## When NOT to use

- The goal is accuracy of growth-rate or cross-feeding predictions rather than runtime/scaling measurements
- Community models are not built from AGORA SBML member models or the target scale exceeds the single-core, up-to-40-member benchmarking scope

## Inputs

- AGORA genome-scale single-organism SBML metabolic models (up to 40 members)
- Equal community member abundance profile
- Single-core execution environment

## Outputs

- runtime_benchmark.csv with measured construction, FBA, and exchange-metabolite runtimes per community size
- Community metabolic models built with PyCoMo (optionally saved as SBML)

## How to apply

Install PyCoMo via pip (`pip install pycomo`) with COBRApy as the underlying modeling framework, then load AGORA genome-scale single-organism SBML models and build compartmentalized community metabolic models at increasing community sizes up to 40 members, recording wall-clock construction time on a single core (reference: 11.3 min at 40 members). Run FBA on each built community model at an equal abundance profile to predict the maximum community growth rate and record the FBA runtime (reference: 2.3 min at 40 members). Enumerate all feasible exchange metabolites and cross-feeding interactions by exchange-flux analysis over the abundance-profile space, recording the analysis runtime (reference: 31.5 min at 40 members; ~3 min for 10-member cross-feeding enumeration). Collect measured runtimes per community size and operation into runtime_benchmark.csv, and accept the benchmark if the 40-member timings match the reported 11.3, 2.3, and 31.5 min within the same order of magnitude.

## Related tools

- **PyCoMo** (Builds the compartmentalized community metabolic models and performs FBA and exchange-metabolite/cross-feeding analysis being timed) — https://github.com/univieCUBE/PyCoMo
- **COBRApy** (Underlying modeling framework; the PyCoMo community models are COBRApy models used directly for FBA)
- **Python 3** (Runtime environment in which PyCoMo is installed via pip and benchmarks are executed)
- **SBML** (Input format of the AGORA single-organism member models and storage format for built community models)

## Examples

```
pip install pycomo; pycomo -h
```

## Evaluation signals

- 40-member community model construction time matches the reported 11.3 min within the same order of magnitude
- 40-member FBA runtime matches the reported 2.3 min within the same order of magnitude
- 40-member exchange-metabolite calculation matches the reported 31.5 min within the same order of magnitude
- 10-member cross-feeding enumeration completes in approximately 3 min
- Measured runtimes per community size per operation are collected into runtime_benchmark.csv

## Limitations

- Runtimes were benchmarked on a single core, so parallel-hardware setups are not represented
- Benchmark scope covers community sizes up to 40 AGORA members; exchange-metabolite calculation scales quadratically with community size
- Larger SBML models can be slow to process outside the benchmark itself, e.g. ScyNet/cy3sbml import of >10-member models 'can take a couple of minutes, depending on your hardware'

## Evidence

- [results] AGORA up to 40 members runtime table: [evidence span withheld — anchor status: not_found]
- [methods] installation: "PyCoMo can be installed via pip:

    pip install pycomo"
- [methods] COBRApy models: "The community models are COBRApy models and can therefor be directly used by other COBRA methods."
- [readme] SBML saving/loading: "It is also possible to save and load the community models in SBML format, allowing to share and reuse the models built with PyCoMo."
- [abstract] exchange metabolite analysis scope: "(iii) all exchange metabolites and cross-feeding interactions in a community metabolic model independent of the abundance profile"
