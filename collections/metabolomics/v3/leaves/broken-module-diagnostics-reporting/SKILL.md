---
name: broken-module-diagnostics-reporting
description: Use when antiSMASH GenBank (.gbk) annotations of PKS/NRPS clusters have
  been reordered by RAIChU's ordering ruleset and you need to enumerate modules left
  split across non-adjacent gene blocks.
when_to_use_negative:
- The cluster was assembled from scratch in RAIChU rather than loaded from antiSMASH
  GenBank output, so no reordering of gene blocks (and hence no broken modules) applies
- You need chemical structure validation (scaffold correctness or fingerprint comparison)
  rather than module-ordering diagnostics
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0780
tools:
- name: RAIChU
  role: Parses antiSMASH GenBank output, predicts and orders modules, and emits the
    broken-modules report
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of the BGC GenBank annotations that are ingested and reordered
- name: Python
  role: Runtime for the RAIChU pip package and pipeline scripts (Python 3.9.12)
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:c2b4e39bd6467ec739bd5cf8d6e3c82f093858d39350fdcf344744d0ddb6da8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Broken-module diagnostics reporting

## Summary

A diagnostics step in RAIChU's antiSMASH GenBank loading pipeline that reports which biosynthetic modules remain broken (split across reordered gene blocks) after applying the module-ordering ruleset, so users can identify clusters where collinear order could not be fully restored.

## When to use

Use when antiSMASH GenBank (.gbk) annotations of PKS/NRPS clusters have been reordered by RAIChU's ordering ruleset and you need to enumerate modules left split across non-adjacent gene blocks.

## When NOT to use

- The cluster was assembled from scratch in RAIChU rather than loaded from antiSMASH GenBank output, so no reordering of gene blocks (and hence no broken modules) applies
- You need chemical structure validation (scaffold correctness or fingerprint comparison) rather than module-ordering diagnostics

## Inputs

- antiSMASH GenBank (.gbk) files of biosynthetic gene clusters (e.g. MIBiG entries)
- Module predictions per gene block from RAIChU's 7 domain/module-assignment rules
- Optimised block ordering produced by RAIChU's ordering ruleset

## Outputs

- Broken-modules report listing any modules left split (non-adjacent) after reordering
- module_order serialisation (one file per input GenBank) of the optimised module ordering

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip. Load the antiSMASH GenBank files (e.g. MIBiG BGCs) with the RAIChU GenBank parser and extract collinear gene blocks from cluster annotations. Predict modules per gene block using RAIChU's 7 domain/module-assignment rules, then optimise block order with the ordering ruleset (O(n!) factorial search with pruning, applied when starter or termination blocks are present or when a cluster has more than 5 blocks) to minimise split modules. After serialising the optimised ordering (module_order, one file per input GenBank), write a broken-modules report listing any modules left split after reordering — these flag clusters whose biological collinearity could not be recovered by the ruleset.

## Related tools

- **RAIChU** (Parses antiSMASH GenBank output, predicts and orders modules, and emits the broken-modules report) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of the BGC GenBank annotations that are ingested and reordered)
- **Python** (Runtime for the RAIChU pip package and pipeline scripts (Python 3.9.12)) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- For every input GenBank file, both a module_order file and a broken-modules report are produced with no unexpected processing errors
- The broken-modules report accounts for all modules in each cluster: a module is either intact in the ordering or explicitly listed as broken
- Reordered cluster satisfies the strict ruleset: exactly one starter module first, any number of elongation modules, optionally one termination module last
- The factorial-search-with-pruning path was applied for clusters with starter/termination blocks or more than 5 blocks, and reported split-module counts are minimal among evaluated orderings
- Cross-check against the MIBiG validation run: running the pipeline over MIBiG antiSMASH GenBank files completes without errors

## Limitations

- Split modules may persist when the input gene order is not collinear with module order; the report only flags them, it does not repair them
- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets, so diagnostics cover module ordering only
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so downstream structural comparison may diverge from characterised products

## Evidence

- [other] RAIChU's parsing/assembly mechanism imposes a strict module-ordering ruleset on loaded clusters: exactly one starter module first, then any number of elongation modules, optionally ending with one termination module.: [evidence span withheld — anchor status: not_found]
- [other] Serialize the optimised module ordering to module_order (one file per input GenBank) and write a broken-modules report listing any modules left split after reordering.: [evidence span withheld — anchor status: not_found]
- [other] Optimise the block order using the RAIChU ordering ruleset, applying the O(n!) factorial-search with pruning for clusters containing starter or termination blocks and for clusters with more than 5 blocks, to minimise the number of broken (split) modules.: [evidence span withheld — anchor status: not_found]
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda"
- [discussion] For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database [36] to catch general processing errors.: "we ran RAIChU on the entire MIBiG database [36] to catch general processing errors"
