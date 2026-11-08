---
name: genbank-feature-parsing
description: Use when the input is antiSMASH GenBank (.gbk) output of a PKS/NRPS BGC
  (e.g. MIBiG entries) and the goal is to load cluster features into RAIChU for module
  prediction and visualisation.
when_to_use_negative:
- The input is not antiSMASH GenBank output but raw sequence or other annotation formats
  — RAIChU's parser targets antiSMASH .gbk results specifically
- The goal is predicting regio- and stereochemistry of tailoring enzymes — RAIChU
  requires users to specify reaction targets manually
- The goal is producing mature natural products directly from raw antiSMASH output
  — RAIChU generates scaffolds, not matured products
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Provides the GenBank parser and the module-ordering ruleset that consume antiSMASH
    .gbk output and produce ordered module representations for visualisation
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of the BGC detection and annotation GenBank output that RAIChU parses
    as input
- name: Python
  role: Execution environment (3.9.12 via conda) in which RAIChU is installed via
    pip and invoked in pipelines
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:71d3f5a765f7c62f9cf77575a10ab10b6e5ebf33d2e2e660d854419a77a19486
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# genbank-feature-parsing

## Summary

Ingest antiSMASH-generated GenBank (.gbk) output of biosynthetic gene clusters using RAIChU's GenBank parser so that cluster features can be processed, module-ordered, and visualised as biosynthetic pathways.

## When to use

Use when the input is antiSMASH GenBank (.gbk) output of a PKS/NRPS BGC (e.g. MIBiG entries) and the goal is to load cluster features into RAIChU for module prediction and visualisation.

## When NOT to use

- The input is not antiSMASH GenBank output but raw sequence or other annotation formats — RAIChU's parser targets antiSMASH .gbk results specifically
- The goal is predicting regio- and stereochemistry of tailoring enzymes — RAIChU requires users to specify reaction targets manually
- The goal is producing mature natural products directly from raw antiSMASH output — RAIChU generates scaffolds, not matured products

## Inputs

- antiSMASH GenBank (.gbk) files of biosynthetic gene clusters (e.g. MIBiG BGCs)
- Cluster annotations containing collinear gene blocks

## Outputs

- Parsed cluster objects ready for module prediction and visualisation
- module_order file per input GenBank (optimised module ordering)
- Broken-modules report listing modules left split after reordering

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip, as the parser depends on this environment (a biopython_independent branch exists for newer Python versions). Feed antiSMASH GenBank files (one per BGC) to RAIChU's GenBank parser, which extracts collinear gene blocks from the cluster annotations. The parser/assembly mechanism then imposes a strict module-ordering ruleset: exactly one starter module first, then any number of elongation modules, optionally ending with one termination module. Module prediction proceeds by iterating over the 7 domain/module-assignment rules implemented in RAIChU's module-prediction logic, and block order can be optimised with the O(n!) factorial-search with pruning for clusters containing starter or termination blocks or more than 5 blocks. The result is a serialised module_order per input GenBank file plus a broken-modules report listing modules left split after reordering.

## Related tools

- **RAIChU** (Provides the GenBank parser and the module-ordering ruleset that consume antiSMASH .gbk output and produce ordered module representations for visualisation) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of the BGC detection and annotation GenBank output that RAIChU parses as input)
- **Python** (Execution environment (3.9.12 via conda) in which RAIChU is installed via pip and invoked in pipelines) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- A module_order file is produced for each input GenBank file without unexpected processing errors
- A broken-modules report is generated for each input, listing any modules left split after reordering
- Parsed cluster satisfies the ordering ruleset: exactly one starter module first, any number of elongation modules, at most one termination module last
- Pipeline runs end-to-end over the MIBiG antiSMASH GenBank validation set without errors, mirroring the paper's validation approach

## Limitations

- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may differ in fingerprint similarity
- Requires Python 3.9.12 for the main branch; newer Python versions need the biopython_independent branch
- Module ordering search scales as O(n!) factorial search with pruning, applied when clusters contain starter or termination blocks or more than 5 blocks
- No changelog or archival Zenodo DOI for the code is provided in the repository documentation

## Evidence

- [other] RAIChU's parsing/assembly mechanism imposes a strict module-ordering ruleset on loaded clusters: exactly one starter module first, then any number of elongation modules, optionally ending with one termination module.: [evidence span withheld — anchor status: not_found]
- [abstract] allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool: "allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool"
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [discussion] For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database: "For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database [36] to catch general processing errors."
