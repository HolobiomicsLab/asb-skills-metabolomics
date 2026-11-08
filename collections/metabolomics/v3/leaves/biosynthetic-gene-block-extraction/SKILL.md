---
name: biosynthetic-gene-block-extraction
description: Use when the input is antiSMASH GenBank (.gbk) output for a biosynthetic
  gene cluster (e.g. from MIBiG) and collinear gene blocks with their module/domain
  composition and a valid module ordering must be extracted for downstream PKS/NRPS
  visualisation.
when_to_use_negative:
- The cluster is not modular (e.g. RiPP, terpene, or other compound classes) — RAIChU's
  starter/elongation/termination module-ordering ruleset targets PKS, NRPS, and hybrid
  PKS/NRPS systems
- Input is not antiSMASH GenBank output but raw nucleotide/FASTA sequence without
  BGC annotations — no collinear gene blocks can be extracted without prior detection/annotation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Provides the GenBank parser, the 7 domain/module-assignment rules, and the
    module-ordering ruleset used to extract and assemble collinear gene blocks
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of the GenBank (.gbk) BGC detection and annotation output that is ingested
    by the RAIChU parser
- name: Python
  role: Scripting environment in which RAIChU is integrated (pip package) to run the
    block-extraction workflow
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:66670adee2a0a52cdea279596f1baa969b51c9fea9014b19c494e9f7607cd07e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# biosynthetic-gene-block-extraction

## Summary

Extract and assemble collinear gene blocks from antiSMASH GenBank (.gbk) output using RAIChU's GenBank parser and module-prediction logic, so that PKS/NRPS clusters become ordered module sequences ready for visualisation as biosynthetic clusters.

## When to use

Use when the input is antiSMASH GenBank (.gbk) output for a biosynthetic gene cluster (e.g. from MIBiG) and collinear gene blocks with their module/domain composition and a valid module ordering must be extracted for downstream PKS/NRPS visualisation.

## When NOT to use

- The cluster is not modular (e.g. RiPP, terpene, or other compound classes) — RAIChU's starter/elongation/termination module-ordering ruleset targets PKS, NRPS, and hybrid PKS/NRPS systems
- Input is not antiSMASH GenBank output but raw nucleotide/FASTA sequence without BGC annotations — no collinear gene blocks can be extracted without prior detection/annotation

## Inputs

- antiSMASH GenBank (.gbk) output files for biosynthetic gene clusters (e.g. MIBiG BGC records)
- Cluster annotations containing collinear gene blocks

## Outputs

- module_order files (one per input GenBank) capturing the optimised module ordering
- Broken-modules report listing modules left split after reordering
- Predicted module/domain assignments per gene block

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip, since RAIChU ships as a Python package for scripting in pipelines. Load the antiSMASH GenBank files with the RAIChU GenBank parser and extract the collinear gene blocks from the cluster annotations. For each gene block, predict modules by iterating over the 7 domain/module-assignment rules implemented in RAIChU's module-prediction logic. Optimise the block order using RAIChU's ordering ruleset — a factorial (O(n!)) search with pruning applied when starter or termination blocks are present or when a cluster has more than 5 blocks — to minimise broken (split) modules, respecting the ruleset that exactly one starter module must come first, followed by any number of elongation modules and optionally one termination module. Serialise the resulting ordering to module_order (one file per input GenBank) and produce a broken-modules report listing any modules left split after reordering.

## Related tools

- **RAIChU** (Provides the GenBank parser, the 7 domain/module-assignment rules, and the module-ordering ruleset used to extract and assemble collinear gene blocks) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of the GenBank (.gbk) BGC detection and annotation output that is ingested by the RAIChU parser)
- **Python** (Scripting environment in which RAIChU is integrated (pip package) to run the block-extraction workflow) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- module_order output file is produced for every input antiSMASH GenBank without unexpected processing errors (validated by running over the MIBiG database)
- Ordered cluster satisfies RAIChU's strict ruleset: exactly one starter module first, any number of elongation modules, at most one optional termination module
- Broken-modules report is generated and the number of split modules after reordering is minimised relative to naive block orderings
- Factorial-order optimisation with pruning is triggered for clusters with starter/termination blocks or more than 5 blocks, as specified by the ordering ruleset
- Predicted module assignments per gene block are consistent with RAIChU's 7 domain/module-assignment rules

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted; users must specify reaction targets
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- Validation against MIBiG relied on manual curation of only a few select examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs
- The README specifies Python 3.9.12; newer Python versions require the separate biopython_independent branch

## Evidence

- [abstract] allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool: "allowing users to upload and edit results from antiSMASH, a widely used BGC detection and annotation tool"
- [results] Running RAIChU on antiSMASH predictions of real-world BGCs from the MIBiG database: "we additionally tested RAIChU on antiSMASH predictions of real-world BGCs from the MIBiG database"
- [results] Running RAIChU on the entire MIBiG database to check for errors: "First, we ran RAIChU on the entire database to ensure no unexpected errors occurred"
- [readme] Create conda environment with Python 3.9.12 and install raichu via pip: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] RAIChU can be installed as a Python pip package for scripting purposes: "RAIChU can be installed as a Python pip package for scripting purposes"
