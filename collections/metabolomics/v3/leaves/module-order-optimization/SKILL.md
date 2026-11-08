---
name: module-order-optimization
description: Use when antiSMASH GenBank (.gbk) annotations of PKS/NRPS clusters need
  to be parsed into collinear gene blocks and their module order optimised before
  visualisation with RAIChU.
when_to_use_negative:
- Input is not antiSMASH GenBank output (e.g. raw nucleotide FASTA without BGC annotation)
  — RAIChU's parser expects annotated cluster features
- Building biosynthetic PKS/NRPS systems from scratch rather than reordering existing
  antiSMASH predictions
- RiPP, terpene, or other non-modular compound classes where module-order rules for
  PKS/NRPS starter/elongation/termination do not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Parses antiSMASH GenBank files, predicts modules via its 7 domain/module-assignment
    rules, and applies the ordering ruleset with O(n!) factorial search and pruning
    to minimise broken modules
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of the BGC detection and annotation GenBank output that is ingested
    by RAIChU for module ordering
- name: Python
  role: Execution environment for RAIChU's pip-installable package used in the reordering
    pipeline
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:60fee5560269cbd1a8ffafcfaae72bac88791018825142029619be234c9c51cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# module-order-optimization

## Summary

Reconstructs and optimises the module ordering of PKS/NRPS biosynthetic clusters loaded from antiSMASH GenBank output in RAIChU, applying RAIChU's strict ordering ruleset (one starter, elongation modules, optional terminator) and an O(n!) factorial search with pruning to minimise broken (split) modules.

## When to use

Use when antiSMASH GenBank (.gbk) annotations of PKS/NRPS clusters need to be parsed into collinear gene blocks and their module order optimised before visualisation with RAIChU.

## When NOT to use

- Input is not antiSMASH GenBank output (e.g. raw nucleotide FASTA without BGC annotation) — RAIChU's parser expects annotated cluster features
- Building biosynthetic PKS/NRPS systems from scratch rather than reordering existing antiSMASH predictions
- RiPP, terpene, or other non-modular compound classes where module-order rules for PKS/NRPS starter/elongation/termination do not apply

## Inputs

- antiSMASH GenBank (.gbk) output files of biosynthetic gene clusters (e.g. from the MIBiG database)
- Cluster annotations containing collinear gene blocks

## Outputs

- module_order: optimised module ordering per cluster (one file per input GenBank)
- broken-modules report listing modules left split after reordering

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip. Load antiSMASH GenBank files (e.g. MIBiG BGCs) with RAIChU's GenBank parser and extract collinear gene blocks from the cluster annotations. Predict modules within each gene block using the 7 domain/module-assignment rules in RAIChU's module-prediction logic. Optimise block order with RAIChU's ordering ruleset — exactly one starter module first, then elongation modules, optionally one termination module — applying the O(n!) factorial-search with pruning for clusters containing starter or termination blocks and for clusters with more than 5 blocks, to minimise the number of broken (split) modules. Serialize the optimised module ordering to module_order and write a broken-modules report listing modules left split after reordering.

## Related tools

- **RAIChU** (Parses antiSMASH GenBank files, predicts modules via its 7 domain/module-assignment rules, and applies the ordering ruleset with O(n!) factorial search and pruning to minimise broken modules) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of the BGC detection and annotation GenBank output that is ingested by RAIChU for module ordering)
- **Python** (Execution environment for RAIChU's pip-installable package used in the reordering pipeline) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- module_order output files are produced for every input GenBank without unexpected processing errors
- A broken-modules report is generated for each input listing any modules left split after reordering
- Every optimised ordering satisfies RAIChU's ruleset: exactly one starter module first, any number of elongation modules, optionally one termination module last
- The number of broken (split) modules after reordering is minimal for the candidate block permutations explored
- Pipeline validation over MIBiG antiSMASH GenBank files completes without errors, matching the article's approach of running RAIChU on the entire MIBiG database to catch general processing errors

## Limitations

- The O(n!) factorial search with pruning is only applied for clusters containing starter or termination blocks and for clusters with more than 5 blocks; other cases rely on the base ordering ruleset
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output
- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted; users must specify reaction targets
- Programmatic shortcuts (e.g. for cationic cascade reactions in terpenes) can lead to simplified representations of reaction intermediates

## Evidence

- [other] RAIChU's parsing/assembly mechanism imposes a strict module-ordering ruleset on loaded clusters: exactly one starter module first, then any number of elongation modules, optionally ending with one termination module.: [evidence span withheld — anchor status: not_found]
- [readme] Installing RAIChU requires a pinned Python version: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [discussion] Validation approach on the MIBiG database: "we ran RAIChU on the entire MIBiG database [36] to catch general processing errors."
- [abstract] antiSMASH output is the intended ingestion source: "allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool"
- [results] MIBiG is the reference dataset of verified BGCs: "the MIBiG database: a database of experimentally verified biosynthetic gene clusters for which both DNA sequence and the associated natural product are known"
