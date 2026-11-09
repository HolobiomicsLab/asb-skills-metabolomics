---
name: mces-structural-similarity-scoring
description: 'Use when evaluating FBMN or library-matching annotation quality: you
  have compound pairs (e.g., top-10 matches per query with SMILES) and need structural
  similarity scores to judge whether matches are identical, structural isomers, or
  dissimilar.'
when_to_use_negative:
- Input is raw MS/MS spectra or feature tables rather than resolved compound pairs
  with SMILES structures — spectrum-level metrics like cosine similarity are the appropriate
  choice instead
- You need a single scalar spectral similarity measure; MCES is a graph-based structural
  distance on molecular graphs, not a fragmentation-similarity metric
- Only identical-match confirmation is needed without structural comparison, since
  MCES computation carries nontrivial runtime cost for large pair sets
edam_topics: []
tools:
- name: AlBi-HHU/myopic-mces
  role: Reference implementation for computing myopic Maximum Common Edge Subgraph
    (MCES) distances between compound SMILES pairs
  repo: https://github.com/AlBi-HHU/myopic-mces
- name: mzmine
  role: Open-source software underlying the high-throughput MS data processing pipeline
    whose FBMN evaluation results provide the compound matches being scored
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:5d13c332826150a10a21de731fbc8f72ebc416460c47622f7447fe3b64d358d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AlBi-HHU/myopic-mces
  tools:
  - AlBi-HHU/myopic-mces
  - mzmine
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mces-structural-similarity-scoring

## Summary

Quantifies how similar matched compounds are by combining Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits) with myopic MCES distances, using the MSnLib criterion that MCES < 4 marks identical or structurally similar compounds.

## When to use

Use when evaluating FBMN or library-matching annotation quality: you have compound pairs (e.g., top-10 matches per query with SMILES) and need structural similarity scores to judge whether matches are identical, structural isomers, or dissimilar.

## When NOT to use

- Input is raw MS/MS spectra or feature tables rather than resolved compound pairs with SMILES structures — spectrum-level metrics like cosine similarity are the appropriate choice instead
- You need a single scalar spectral similarity measure; MCES is a graph-based structural distance on molecular graphs, not a fragmentation-similarity metric
- Only identical-match confirmation is needed without structural comparison, since MCES computation carries nontrivial runtime cost for large pair sets

## Inputs

- Top-10 matched compounds per query from FBMN evaluation results, with SMILES structures
- Compound pairs formatted as CSV lines 'index,SMILES1,SMILES2' for myopic-mces input

## Outputs

- Pairwise Tanimoto similarity scores (Morgan radius 2, 2048-bit fingerprints)
- Pairwise myopic MCES distances with computation modes
- Aggregate fraction of identical-or-similar pairs (target: 94%)
- similarity_scores.csv containing pairwise scores plus aggregate summary

## How to apply

Load the top-10 matched compounds per query from the FBMN evaluation results and their SMILES. Compute Tanimoto similarity for each compound pair using Morgan fingerprints with radius 2 and 2048 bits. Compute myopic MCES distances for the same pairs with the myopic-mces tool (default COIN_CMD ILP solver; distance threshold logic means exact distances are only computed below the threshold). Classify each pair as identical or structurally similar when MCES < 4, consistent with the Extended Data Fig. 5 criterion — e.g., MCES = 2 often describes structural isomers differing in the position of one functional group. Aggregate the fraction of identical-or-similar pairs (MSnLib reports 94%) and write pairwise Tanimoto and MCES scores plus the aggregate summary to similarity_scores.csv.

## Related tools

- **AlBi-HHU/myopic-mces** (Reference implementation for computing myopic Maximum Common Edge Subgraph (MCES) distances between compound SMILES pairs) — https://github.com/AlBi-HHU/myopic-mces
- **mzmine** (Open-source software underlying the high-throughput MS data processing pipeline whose FBMN evaluation results provide the compound matches being scored) — https://github.com/mzmine/mzmine

## Examples

```
from myopic_mces import MCES; MCES('CC(=O)OC1=CC=CC=C1C(=O)O', 'CN1C=NC2=C1C(=O)N(C(=O)N2C)C')
```

## Evaluation signals

- Aggregate fraction of pairs classified as identical-or-similar (MCES < 4) reproduces the reported 94% value
- similarity_scores.csv exists and contains both pairwise Tanimoto and MCES columns plus the aggregate summary row
- MCES computation modes indicate trustworthy distances (mode 1 = exact/proven optimal; modes 5/6 with time limits or distance -1 are not exact)
- Structural isomers differing in one functional-group position score around MCES = 2, sanity-checking the < 4 cutoff
- Tanimoto values are consistent with 2048-bit Morgan radius-2 fingerprints and fall in [0, 1]

## Limitations

- By default exact MCES distance is only calculated if it falls below the threshold (default 10); with --solver_time_limit_seconds, instances hitting the limit return mode 5 (not proven exact) or mode 6 (distance -1), and the distinction is only reliable for COIN_CMD and CPLEX_PY solvers
- Using --no_ilp_threshold results in longer runtimes and potential violations of the triangle equation
- The Python lower bounds are slow; the C++ implementation (~266× faster) requires pip install myopic-mces[cpp]
- PuLP does not ship with solvers anymore, so an ILP solver (default COIN_CMD via pip install pulp[cbc]) must be configured before running myopic-mces

## Evidence

- [results] 94% identical or similar criterion: [evidence span withheld — anchor status: not_found]
- [results] MCES = 2 structural isomers: "MCES = 2 often describes structural isomers differing in the position of one functional group"
- [readme] myopic-mces CLI usage: "pip install myopic-mces
myopic_mces input-file [output-file]"
- [readme] myopic-mces input format: [evidence span withheld — anchor status: external_doc]
- [readme] ILP solver requirement: [evidence span withheld — anchor status: external_doc]
