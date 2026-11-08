---
name: pk-nrp-module-prediction
description: Use when antiSMASH GenBank output for PKS, NRPS, or hybrid PKS/NRPS clusters
  must be parsed into modules and an optimised module ordering (minimising broken/split
  modules) is required.
when_to_use_negative:
- Input is not antiSMASH GenBank output for modular PKS/NRPS clusters — RAIChU handles
  RiPPs and terpenes differently, with tailoring/cyclisation reactions orderable rather
  than starter-elongation-termination modules
- The goal is predicting mature natural product structures from raw antiSMASH output
  — RAIChU generates natural product scaffolds, not maturated products
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Provides the GenBank parser, the 7 domain/module-assignment rules, and the
    module-ordering ruleset (factorial search with pruning) used to build and optimise
    cluster module order
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of the BGC detection and annotation GenBank results that are uploaded,
    parsed, and edited as input to the loader
- name: Python
  role: Execution environment (3.9.12) and scripting interface; RAIChU is installed
    as a pip package for integration into Python pipelines
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:e823be36ef5e7e044a4a5b9914ec46034d0382643e412fff302ee35f8994af0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pk-nrp-module-prediction

## Summary

Reconstructs RAIChU's ingestion of antiSMASH GenBank (.gbk) output into PKS/NRPS cluster representations, including collinear gene-block extraction, module prediction via 7 domain-assignment rules, and module-order optimisation under RAIChU's strict starter–elongation–termination ruleset.

## When to use

Use when antiSMASH GenBank output for PKS, NRPS, or hybrid PKS/NRPS clusters must be parsed into modules and an optimised module ordering (minimising broken/split modules) is required.

## When NOT to use

- Input is not antiSMASH GenBank output for modular PKS/NRPS clusters — RAIChU handles RiPPs and terpenes differently, with tailoring/cyclisation reactions orderable rather than starter-elongation-termination modules
- The goal is predicting mature natural product structures from raw antiSMASH output — RAIChU generates natural product scaffolds, not maturated products

## Inputs

- antiSMASH GenBank (.gbk) output files of PKS, NRPS, or hybrid PKS/NRPS biosynthetic gene clusters (e.g. from the MIBiG database)

## Outputs

- module_order serialization (one file per input GenBank) containing the optimised module ordering
- broken-modules report listing any modules left split after reordering

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip (pip install raichu). Load the antiSMASH GenBank files (e.g. MIBiG BGCs) with RAIChU's GenBank parser and extract collinear gene blocks from the cluster annotations. Predict modules within each gene block by iterating over the 7 domain/module-assignment rules implemented in RAIChU's module-prediction logic. Optimise block order with RAIChU's ordering ruleset — which requires exactly one starter module first, any number of elongation modules, and optionally one termination module last — applying the O(n!) factorial search with pruning for clusters containing starter or termination blocks and for clusters with more than 5 blocks, to minimise split modules. Serialize the optimised ordering (one output per input GenBank) and emit a broken-modules report for any modules left split after reordering.

## Related tools

- **RAIChU** (Provides the GenBank parser, the 7 domain/module-assignment rules, and the module-ordering ruleset (factorial search with pruning) used to build and optimise cluster module order) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of the BGC detection and annotation GenBank results that are uploaded, parsed, and edited as input to the loader)
- **Python** (Execution environment (3.9.12) and scripting interface; RAIChU is installed as a pip package for integration into Python pipelines) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- module_order and a broken-modules report are produced for each input GenBank without unexpected processing errors
- Every optimised ordering satisfies the strict ruleset: exactly one starter module first, any number of elongation modules, optionally one termination module last
- The broken-modules report shows the number of split modules was minimised by the ordering search
- Pipeline validated end-to-end over MIBiG antiSMASH GenBank files as RAIChU did to catch general processing errors

## Limitations

- Clusters whose gene blocks cannot be arranged under the strict starter/elongation/termination ruleset yield split (broken) modules recorded in the report
- Validation relied on manual curation of only a few select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- If a newer Python version is required, the biopython_independent branch instructions must be used instead of the standard pip install

## Evidence

- [other] RAIChU's parsing/assembly mechanism imposes a strict module-ordering ruleset on loaded clusters: exactly one starter module first, then any number of elongation modules, optionally ending with one termination module.: [evidence span withheld — anchor status: not_found]
- [abstract] allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool, or to build biosynthetic PKS/NRPS systems from scratch: "allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool, or to build biosynthetic PKS/NRPS systems from scratch"
- [methods] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [discussion] For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database [36] to catch general processing errors.: "we ran RAIChU on the entire MIBiG database [36] to catch general processing errors"
