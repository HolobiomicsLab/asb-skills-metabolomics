---
name: pks-nrps-module-assembly-representation
description: Use when you need automated, module-resolution visual depictions of PKS/NRPS
  or hybrid cluster chemistry — e.g. drawing elongation and tailoring reactions from
  antiSMASH output or randomly generated BGCs — or when benchmarking drawing performance
  per module type.
when_to_use_negative:
- You need mature natural products predicted directly from raw antiSMASH output —
  RAIChU generates scaffolds, and predicted vs. real-world structures may not have
  high fingerprinting similarity
- You need automatic prediction of regio- and stereochemistry of tailoring enzymes
  — users must specify reaction targets
- You only need NP scaffold prediction without pathway visualisation — antiSMASH or
  PRISM alone suffice
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: 'Primary tool: builds PKS/NRPS/hybrid cluster models and renders module-resolution
    spaghetti diagrams; installable as a Python pip package for scripting'
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of BGC predictions that can be uploaded and edited in RAIChU as input
    for visualisation
- name: PIKAChU
  role: Underlying reaction engine enforcing chemical correctness of RAIChU's depicted
    reaction intermediates
- name: Python
  role: Scripting environment for pip-installed RAIChU, random BGC generation, timing
    with time.perf_counter(), and pandas aggregation
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:08e5c9b3c840a8ab7d0ea6ef6135a27545a26f311c6f484e64f3c45552cda0b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pks-nrps-module-assembly-representation

## Summary

Method for automatically rendering 'spaghetti diagrams' of PKS, NRPS, and hybrid PKS/NRPS biosynthetic pathways at module resolution using RAIChU, including benchmarking rendering speed on randomly generated 1–10 module BGCs.

## When to use

Use when you need automated, module-resolution visual depictions of PKS/NRPS or hybrid cluster chemistry — e.g. drawing elongation and tailoring reactions from antiSMASH output or randomly generated BGCs — or when benchmarking drawing performance per module type.

## When NOT to use

- You need mature natural products predicted directly from raw antiSMASH output — RAIChU generates scaffolds, and predicted vs. real-world structures may not have high fingerprinting similarity
- You need automatic prediction of regio- and stereochemistry of tailoring enzymes — users must specify reaction targets
- You only need NP scaffold prediction without pathway visualisation — antiSMASH or PRISM alone suffice

## Inputs

- antiSMASH BGC detection/annotation results (uploadable into RAIChU)
- Randomly generated PKS/NRPS cluster specifications (1–10 modules, 5 module types: NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS)
- User-specified tailoring reaction targets (regio-/stereochemistry cannot be predicted)

## Outputs

- Spaghetti diagrams of biosynthetic pathways at module resolution (SVG per cluster; GIF animations per validation set)
- Timing table of drawing seconds per cluster (e.g. drawing_time_results.csv with mean and standard deviation per module type and module count)

## How to apply

Install RAIChU in a conda environment pinned to Python 3.9.12 and install via pip ('pip install raichu'); for newer Python, use the biopython_independent branch. Build or import cluster definitions — either upload and edit antiSMASH results, construct biosynthetic PKS/NRPS systems from scratch, or use RAIChU's cluster generation functionality to randomly generate BGCs for each of the 5 module types (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS). Call RAIChU's drawing functionality to produce spaghetti diagrams (SVG files, with GIF animations for validation sets), optionally timing the rendering with time.perf_counter() and aggregating per-module-type means/standard deviations with pandas. RAIChU enforces chemical correctness of reaction intermediates via PIKAChU and implements 34 prevalent tailoring reactions, so diagrams depict fully maturated product pathways at module resolution rather than SMILES concatenation.

## Related tools

- **RAIChU** (Primary tool: builds PKS/NRPS/hybrid cluster models and renders module-resolution spaghetti diagrams; installable as a Python pip package for scripting) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of BGC predictions that can be uploaded and edited in RAIChU as input for visualisation)
- **PIKAChU** (Underlying reaction engine enforcing chemical correctness of RAIChU's depicted reaction intermediates)
- **Python** (Scripting environment for pip-installed RAIChU, random BGC generation, timing with time.perf_counter(), and pandas aggregation) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Diagram correctness: 100% cluster drawing correctness and ≥97.6% drawing readability on randomly generated cluster sets (paper reports 100% accuracy / 97.62% readability, 97.66% overall readability)
- Speed ordering invariant: mean drawing time is fastest for PKS clusters (~2.73 s/cluster), then NRPS/PKS hybrids (5.21 s), then NRPS (9.62 s)
- Completeness check: 5 module types × 10 module counts × 10 BGCs = 500 timing rows, all non-negative and finite
- Structural sanity: generated intermediates are chemically correct per PIKAChU reaction enforcement; assess bond angles and overlap as in the paper's manual assessment

## Limitations

- Cationic cascade reactions in terpenes require programmatic shortcuts, leading to simplified representations of reaction intermediates; a summarized pathway view shows only initial and final intermediates
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (e.g. the CMC-thuggacins)
- Python 3.9.12 is required for the main branch; newer Python needs the biopython_independent branch
- Speed and readability figures come from randomly generated systems and select manually curated MIBiG examples, not automated validation of all MIBiG BGCs

## Evidence

- [results] RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster: "RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster"
- [results] Random generation for validation: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs"
- [methods] Install raichu via pip: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] PIKAChU chemical correctness: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"
- [readme] README: speed assessment scope: "RAIChU is a chemoinformatics tool designed to illuminate the intricate processes of scaffold diversification in natural product chemistry through the implementation of 34 tailoring reactions."
