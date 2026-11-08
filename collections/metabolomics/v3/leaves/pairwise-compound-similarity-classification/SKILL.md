---
name: pairwise-compound-similarity-classification
description: Use when you have FBMN/MSnLib annotation matches (top-10 matched compounds
  per query) as SMILES pairs and need to quantify and threshold their structural similarity
  via Tanimoto and MCES scores.
when_to_use_negative:
- Compounds lack SMILES/structure information and only names are available (structure
  must first be resolved, e.g. via PubChem Name search)
- The goal is spectral similarity scoring (e.g. cosine/gnps cosine in FBMN) rather
  than chemical-structure comparison of annotation matches
- You need exact 3D conformer comparison rather than myopic MCES distances on 2D molecular
  graphs
tools:
- name: myopic-mces
  role: Computes the myopic Maximum Common Edge Subgraph distances used for the MCES
    < 4 identical-or-similar classification; takes index,SMILES1,SMILES2 CSV input
  repo: https://github.com/AlBi-HHU/myopic-mces
- name: mzmine
  role: Open-source MS data processing software underlying the FBMN evaluation workflow
    from which the top-10 matched compounds per query are taken
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:b9a786c4d70e0bcbefb9f6c6dcded4ea330185e7457e626e69bf4ddd35b688c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AlBi-HHU/myopic-mces
  tools:
  - myopic-mces
  - mzmine
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pairwise-compound-similarity-classification

## Summary

Classifies pairs of annotated compounds as identical or structurally similar by computing Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits) and myopic maximum common edge subgraph (MCES) distances, using the MCES < 4 criterion from the MCEBIO FBMN evaluation to reproduce the reported 94% identical-or-similar fraction.

## When to use

Use when you have FBMN/MSnLib annotation matches (top-10 matched compounds per query) as SMILES pairs and need to quantify and threshold their structural similarity via Tanimoto and MCES scores.

## When NOT to use

- Compounds lack SMILES/structure information and only names are available (structure must first be resolved, e.g. via PubChem Name search)
- The goal is spectral similarity scoring (e.g. cosine/gnps cosine in FBMN) rather than chemical-structure comparison of annotation matches
- You need exact 3D conformer comparison rather than myopic MCES distances on 2D molecular graphs

## Inputs

- FBMN evaluation results containing top-10 matched compounds per query compound (SMILES structures)
- CSV input file for MCES computation with one comparison per line: index,SMILES1,SMILES2

## Outputs

- similarity_scores.csv with pairwise Tanimoto similarities and myopic MCES distances per compound pair
- Aggregate summary: fraction of identical-or-similar pairs (reported value 94%, MCES < 4 criterion)

## How to apply

Load the top-10 matched compounds per query from the FBMN evaluation results and form all compound pairs. Compute Tanimoto similarity for each pair from Morgan fingerprints with radius 2 and 2048 bits. Compute the myopic MCES distance for each pair with the AlBi-HHU/myopic-mces tool, which takes a CSV input file of rows 'index,SMILES1,SMILES2' and returns 'index,myopic MCES distance,computation time in seconds,computation mode'. Classify a pair as identical or structurally similar when MCES < 4, consistent with the Extended Data Fig. 5 criterion (e.g., MCES = 2 often describes structural isomers differing in the position of one functional group). Aggregate the fraction of identical-or-similar pairs to reproduce the reported 94% value, and write the pairwise Tanimoto and MCES scores plus the aggregate summary to similarity_scores.csv.

## Related tools

- **myopic-mces** (Computes the myopic Maximum Common Edge Subgraph distances used for the MCES < 4 identical-or-similar classification; takes index,SMILES1,SMILES2 CSV input) — https://github.com/AlBi-HHU/myopic-mces
- **mzmine** (Open-source MS data processing software underlying the FBMN evaluation workflow from which the top-10 matched compounds per query are taken) — https://github.com/mzmine/mzmine

## Examples

```
from myopic_mces import MCES; MCES('CC(=O)OC1=CC=CC=C1C(=O)O', 'CN1C=NC2=C1C(=O)N(C(=O)N2C)C')
```

## Evaluation signals

- similarity_scores.csv contains one row per compound pair with both a Tanimoto value (0–1) and a myopic MCES distance
- Per-pair MCES computation mode is checked (mode 1 = exact distance proven optimal; mode 2 = distance above threshold; modes 5/6 indicate a solver time limit was hit and the distance cannot be trusted)
- Aggregate fraction of pairs with MCES < 4 reproduces the reported 94% identical-or-similar value
- Structural isomer pairs (e.g. differing in the position of one functional group) yield low MCES values around 2
- MCES input/output CSVs follow the documented schema 'index,myopic MCES distance,computation time in seconds,computation mode'

## Limitations

- Exact MCES distance is only guaranteed when the ILP solver proves optimality; with --solver_time_limit_seconds, modes 5/6 return unproven upper bounds or -1, and this distinction is only reliable for the COIN_CMD and CPLEX_PY solvers
- The default --threshold of 10 means exact distance is only calculated below the threshold; pairs above it return the threshold value instead of the exact distance
- RDKit warnings may appear during parsing; a solver (default COIN_CMD) must be installed separately since PuLP does not ship with solvers

## Evidence

- [readme] MCES tool input/output format: "input-file: `index,SMILES1,SMILES2`

output-file: `index,myopic MCES distance,computation time in seconds,computation mode`"
- [readme] MCES reference implementation description: [evidence span withheld — anchor status: external_doc]
- [readme] Exactness of returned distances depends on computation mode: [evidence span withheld — anchor status: external_doc]
- [readme] CLI usage of myopic-mces: "Download via pip and execute:
```bash
pip install myopic-mces
myopic_mces input-file [output-file]"
