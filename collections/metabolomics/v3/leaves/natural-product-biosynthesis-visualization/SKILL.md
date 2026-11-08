---
name: natural-product-biosynthesis-visualization
description: Use when you have a PKS, NRPS, RiPP, or terpene BGC (e.g., an antiSMASH
  prediction or MIBiG entry) and need a module-resolution pathway diagram with drawn
  intermediates and products, rather than only a scaffold prediction.
when_to_use_negative:
- You need automated prediction of tailoring enzyme regio- and stereochemistry — RAIChU
  requires users to specify their reaction targets
- You expect a fully matured natural product structure directly from raw antiSMASH
  output — RAIChU generates natural product scaffolds, not mature natural products
- You only need BGC detection/scaffold prediction without pathway visualization —
  antiSMASH or PRISM suffice, since they predict NP scaffolds but do not supply detailed
  pathway visualizations
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Core renderer producing automated, high-quality 'spaghetti diagram' visualizations
    of PKS/NRPS/hybrid biosynthetic pathways, with random cluster generation and antiSMASH
    result import
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of the depicted elongation and tailoring reaction
    intermediates during rendering
- name: Python
  role: Runtime environment (3.9.12 conda env) and scripting interface via the pip
    package for pipeline integration
  repo: https://pypi.org/project/raichu
- name: antiSMASH
  role: Source of BGC predictions that can be uploaded and edited into RAIChU for
    visualization
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:deebf819843b7b8b33bb2d6c8bca5bf71edee0682ae55956ec6e2d42cbf6a721
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
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# natural-product-biosynthesis-visualization

## Summary

Automated generation of 'spaghetti diagram' visualizations of PKS, NRPS, and hybrid PKS/NRPS biosynthetic pathways at module resolution using RAIChU, with PIKAChU enforcing chemical correctness of elongation and tailoring reaction intermediates.

## When to use

Use when you have a PKS, NRPS, RiPP, or terpene BGC (e.g., an antiSMASH prediction or MIBiG entry) and need a module-resolution pathway diagram with drawn intermediates and products, rather than only a scaffold prediction.

## When NOT to use

- You need automated prediction of tailoring enzyme regio- and stereochemistry — RAIChU requires users to specify their reaction targets
- You expect a fully matured natural product structure directly from raw antiSMASH output — RAIChU generates natural product scaffolds, not mature natural products
- You only need BGC detection/scaffold prediction without pathway visualization — antiSMASH or PRISM suffice, since they predict NP scaffolds but do not supply detailed pathway visualizations

## Inputs

- PKS/NRPS BGC definitions built from scratch in RAIChU
- antiSMASH BGC detection and annotation results
- MIBiG database entries (experimentally verified BGCs with linked SMILES, optionally from PubChem or NP Atlas)
- Randomly generated modular cluster architectures (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS) with 0–11 elongation modules
- User-specified tailoring enzyme reaction targets (regio- and stereochemistry cannot be predicted)

## Outputs

- Spaghetti diagram pathway visualizations at module resolution (separate .svg files and animated .gif files per validation set)
- Visual biosynthetic models with highlighted potential tailoring reaction sites on molecules
- Per-cluster correctness/readability tallies (e.g., validation_metrics.csv)
- Structured error/mismatch logs from batch MIBiG processing (e.g., JSON log)

## How to apply

Create a conda environment with Python 3.9.12 and install the tool with 'pip install raichu'. Build the biosynthetic model either from scratch, by uploading/editing antiSMASH results, or by random cluster generation for validation; for tailoring steps, note that regio- and stereochemistry cannot be predicted, so reaction targets must be specified by the user. Render the spaghetti diagram, relying on PIKAChU, which enforces chemical correctness of the depicted elongation and tailoring intermediates; for terpene cationic cascades a summarized pathway view (initial and final intermediates only) is available. Validate correctness by checking that the depicted structures match the encoded module/domain composition, and readability by checking for legible bond angles and absence of text/structure overlap; benchmark expectations are 100% drawing correctness and ~97.62–97.66% readability on randomly generated sets, with ~2.73 s average drawing time per PKS cluster.

## Related tools

- **RAIChU** (Core renderer producing automated, high-quality 'spaghetti diagram' visualizations of PKS/NRPS/hybrid biosynthetic pathways, with random cluster generation and antiSMASH result import) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of the depicted elongation and tailoring reaction intermediates during rendering)
- **Python** (Runtime environment (3.9.12 conda env) and scripting interface via the pip package for pipeline integration) — https://pypi.org/project/raichu
- **antiSMASH** (Source of BGC predictions that can be uploaded and edited into RAIChU for visualization)

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Depicted structures match the encoded module/domain composition (drawing correctness target: 100%)
- Diagrams have legible bond angles and no text/structure overlap (readability target: ~97.62% on random sets)
- Batch MIBiG run completes with no unexpected processing errors; the colistin A entry is flagged as pointing to the wrong PubChem entry
- Drawing speed is on the order of seconds per cluster (reported average 2.73 s per PKS cluster, 9.62 s NRPS, 5.21 s NRPS/PKS hybrid)
- All expected rows/records present in outputs (e.g., 5000 validation clusters, 5 types x 10 module counts x 10 BGCs timing rows with finite, non-negative durations)

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot currently be predicted; users must specify their reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (e.g., observed in the CMC-thuggacins)
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, leading to simplified representations; a summarized pathway view shows only initial and final intermediates
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity

## Evidence

- [abstract] RAIChU produces depictions of biosynthetic transformations: "we developed RAIChU (Reaction Analysis through Illustrating Chemical Units), which produces depictions of biosynthetic transformations of PKS, NRPS, and hybrid PKS/NRPS systems"
- [results] PIKAChU enforces chemical correctness: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"
- [results] Tailoring reaction target specification: "As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets"
- [methods] Installation via conda and pip: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [readme] Tailoring site highlighting feature: "A unique feature of RAIChU is its capability to detect and visually highlight potential sites for these tailoring reactions on molecules"
