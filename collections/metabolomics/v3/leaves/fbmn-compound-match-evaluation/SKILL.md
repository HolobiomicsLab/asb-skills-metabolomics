---
name: fbmn-compound-match-evaluation
description: Use when you have FBMN annotation results with top-matched compounds
  and need to quantify how structurally faithful those matches are to the actually
  analyzed compounds via Tanimoto and MCES scores.
when_to_use_negative:
- When compounds are already confirmed by reference standards or identical spectral
  match — structural similarity scoring adds nothing for identical structures
- When input is not structure-annotated (no SMILES available for either the match
  or the analyzed compound); PubChem Name search fallback may be needed first
- When evaluating spectral-level match quality (cosine scores) rather than structural
  plausibility of library annotations
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: myopic-mces
  role: Computes myopic Maximum Common Edge Subgraph distances between matched and
    analyzed compound SMILES pairs
  repo: https://github.com/AlBi-HHU/myopic-mces
- name: mzmine
  role: Open-source software providing the FBMN feature-based molecular networking
    results and annotation evaluation context
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:69f549b7e7887bcb516c13f45b17fb8dc19a596f1796160ad722c1ccb4107a35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AlBi-HHU/myopic-mces
  tools:
  - myopic-mces
  - mzmine
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# fbmn-compound-match-evaluation

## Summary

Evaluates the quality of FBMN library matches by computing Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits) and myopic MCES distances for the top-10 matched compound pairs, classifying pairs as identical or structurally similar when MCES < 4 — the criterion behind the reported 94% identical-or-similar annotations.

## When to use

Use when you have FBMN annotation results with top-matched compounds and need to quantify how structurally faithful those matches are to the actually analyzed compounds via Tanimoto and MCES scores.

## When NOT to use

- When compounds are already confirmed by reference standards or identical spectral match — structural similarity scoring adds nothing for identical structures
- When input is not structure-annotated (no SMILES available for either the match or the analyzed compound); PubChem Name search fallback may be needed first
- When evaluating spectral-level match quality (cosine scores) rather than structural plausibility of library annotations

## Inputs

- FBMN evaluation results containing top-10 matched compounds per query with SMILES for both matched and analyzed compounds
- CSV file of SMILES pairs formatted as 'index,SMILES1,SMILES2' for myopic-mces

## Outputs

- Pairwise Tanimoto similarity scores for each compound pair
- Myopic MCES distances per pair with computation mode
- Aggregate fraction of identical-or-similar pairs (target: ~94%)
- similarity_scores.csv containing pairwise scores and the aggregate summary

## How to apply

Load the top-10 matched compounds per query from the FBMN evaluation results, then compute pairwise Tanimoto similarity using Morgan fingerprints (radius 2, 2048 bits). For the same pairs, compute myopic MCES distances with the AlBi-HHU/myopic-mces tool, which takes an input CSV of SMILES pairs ('index,SMILES1,SMILES2') and returns 'index,myopic MCES distance,computation time in seconds,computation mode'. Classify each pair as identical or structurally similar when MCES < 4, consistent with the Extended Data Fig. 5 criterion — e.g., MCES = 2 often describes structural isomers differing in the position of one functional group. Aggregate the fraction of identical-or-similar pairs to reproduce the reported 94% value, and write the pairwise Tanimoto and MCES scores plus the aggregate summary to similarity_scores.csv. Use the threshold-bounded mode (default threshold 10) or set --threshold 4 to only resolve exact distances for borderline pairs, and check the computation mode column: mode 1 is an exact proven distance, mode 2 means the distance is above threshold.

## Related tools

- **myopic-mces** (Computes myopic Maximum Common Edge Subgraph distances between matched and analyzed compound SMILES pairs) — https://github.com/AlBi-HHU/myopic-mces
- **mzmine** (Open-source software providing the FBMN feature-based molecular networking results and annotation evaluation context) — https://github.com/mzmine/mzmine

## Examples

```
from myopic_mces import MCES; MCES('CC(=O)OC1=CC=CC=C1C(=O)O', 'CN1C=NC2=C1C(=O)N(C(=O)N2C)C')
```

## Evaluation signals

- Aggregate fraction of identical-or-similar pairs reproduces the reported 94% value
- All pairs with MCES < 4 are classified as identical or structurally similar per the Extended Data Fig. 5 criterion
- similarity_scores.csv contains both pairwise Tanimoto and MCES scores plus the aggregate summary
- myopic-mces output rows report computation mode 1 (exact, proven optimal) for distances below the threshold; mode 2 confirms distance is above threshold
- Tanimoto scores use the specified fingerprint parameters: Morgan radius 2, 2048 bits

## Limitations

- MCES computation may be slow for large molecule sets; the C++ lower-bound implementation (~266× faster) or a solver time limit may be needed, and time-limited runs yield non-exact modes (5: solution found but not proven exact; 6: no solution found, distance -1)
- Exact distance is only calculated if the distance is lower than the threshold (default 10); pairs above the threshold return the threshold value rather than the exact distance
- myopic-mces requires an ILP solver (default COIN_CMD via PuLP) that must be configured separately, since PuLP does not ship with solvers
- MCES = 2 structural isomers may be biologically or chemically distinct despite being classified as 'similar'

## Evidence

- [results] Classify identical/similar pairs by MCES: [evidence span withheld — anchor status: not_found]
- [results] MCES = 2 isomers interpretation: [evidence span withheld — anchor status: not_found]
- [readme] myopic-mces input/output format: "input-file: `index,SMILES1,SMILES2`

output-file: `index,myopic MCES distance,computation time in seconds,computation mode`"
- [readme] Threshold behavior of myopic-mces: "--threshold int         Threshold for the comparison.
                        Exact distance is only calculated if the distance is lower than the threshold."
- [methods] Tanimoto fingerprint parameters: [evidence span withheld — anchor status: not_found]
