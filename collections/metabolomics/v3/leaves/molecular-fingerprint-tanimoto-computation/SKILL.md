---
name: molecular-fingerprint-tanimoto-computation
description: Use when you have top-matched compound pairs (query SMILES vs. library-hit
  SMILES) from FBMN-style annotations and need to verify that library matches are
  identical or structurally close to the analyzed compounds.
when_to_use_negative:
- You only need spectral (fragmentation) similarity — this skill measures structural
  similarity from SMILES, not MS/MS cosine similarity.
- The matched structures are unavailable (no SMILES and no resolvable structure via
  PubChem Name search) — neither Morgan fingerprints nor MCES can be computed.
- You need exact isomer-level distinction beyond what MCES < 4 captures — positional-isomer
  cases like MCES = 2 are classified as 'similar', not resolved.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: AlBi-HHU/myopic-mces
  role: Computes the myopic Maximum Common Edge Subgraph (MCES) distance between SMILES
    pairs used to classify matches as identical or similar (MCES < 4).
  repo: https://github.com/AlBi-HHU/myopic-mces
- name: mzmine
  role: Open-source MS data processing software underlying the FBMN evaluation workflow
    from which the top-10 matched compounds per query are drawn.
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:9c29deab883900d3143d65f480b21129feb95a1ad0952215f42fa32b02d8bcf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AlBi-HHU/myopic-mces
  tools:
  - AlBi-HHU/myopic-mces
  - mzmine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# molecular-fingerprint-tanimoto-computation

## Summary

Quantifies structural similarity between query and library-matched compounds by computing Tanimoto similarity from Morgan fingerprints (radius 2, 2048 bits) and myopic MCES distances, classifying matches as identical or structurally similar when MCES < 4.

## When to use

Use when you have top-matched compound pairs (query SMILES vs. library-hit SMILES) from FBMN-style annotations and need to verify that library matches are identical or structurally close to the analyzed compounds.

## When NOT to use

- You only need spectral (fragmentation) similarity — this skill measures structural similarity from SMILES, not MS/MS cosine similarity.
- The matched structures are unavailable (no SMILES and no resolvable structure via PubChem Name search) — neither Morgan fingerprints nor MCES can be computed.
- You need exact isomer-level distinction beyond what MCES < 4 captures — positional-isomer cases like MCES = 2 are classified as 'similar', not resolved.

## Inputs

- FBMN evaluation results with top-10 matched compounds per query (SMILES for query and library match)
- CSV of compound pairs in the format `index,SMILES1,SMILES2` for myopic-mces input

## Outputs

- Pairwise Tanimoto similarity scores (Morgan fingerprints, radius 2, 2048 bits)
- Pairwise myopic MCES distances with computation mode flags
- Aggregate summary of the fraction of identical-or-similar pairs (e.g., the 94% value)
- similarity_scores.csv combining all of the above

## How to apply

Load the top-10 matched compounds per query from the FBMN evaluation results and form all compound pairs as SMILES strings. Compute Tanimoto similarity for each pair using Morgan fingerprints with radius 2 and 2048 bits, and compute myopic MCES distances for the same pairs with the AlBi-HHU/myopic-mces tool, which takes an input CSV with columns `index,SMILES1,SMILES2` and returns `index,myopic MCES distance,computation time in seconds,computation mode`. Classify each pair as identical or structurally similar when MCES < 4, consistent with the Extended Data Fig. 5 criterion (e.g., MCES = 2 often describes structural isomers differing in the position of one functional group). Aggregate the fraction of identical-or-similar pairs to reproduce the reported 94% value, and write the pairwise Tanimoto and MCES scores plus the aggregate summary to similarity_scores.csv. When running myopic-mces, prefer the C++ lower-bound implementation (`pip install myopic-mces[cpp]`, ~266× faster) and check the computation mode column: mode 1 is a proven-exact distance, mode 2 means the distance is above threshold, and modes 5/6 indicate time-limit cases whose distances should not be trusted.

## Related tools

- **AlBi-HHU/myopic-mces** (Computes the myopic Maximum Common Edge Subgraph (MCES) distance between SMILES pairs used to classify matches as identical or similar (MCES < 4).) — https://github.com/AlBi-HHU/myopic-mces
- **mzmine** (Open-source MS data processing software underlying the FBMN evaluation workflow from which the top-10 matched compounds per query are drawn.) — https://github.com/mzmine/mzmine

## Examples

```
from myopic_mces import MCES; MCES('CC(=O)OC1=CC=CC=C1C(=O)O', 'CN1C=NC2=C1C(=O)N(C(=O)N2C)C')
```

## Evaluation signals

- similarity_scores.csv contains a Tanimoto value and an MCES distance plus computation mode for every pair in the top-10-per-query input.
- The aggregate fraction of identical-or-similar pairs (MCES < 4) reproduces the reported 94%.
- myopic-mces output lines follow the schema `index,myopic MCES distance,computation time in seconds,computation mode`.
- All MCES entries used in the aggregate carry computation mode 1 (proven exact) or 2 (above threshold); modes 5/6 (time-limit hits) are excluded or flagged.
- Known controls: e.g., structural isomers differing in one functional-group position yield MCES ≈ 2.

## Limitations

- MCES computation is ILP-based (default solver COIN_CMD via PuLP); instances that hit `--solver_time_limit_seconds` return mode 5/6 results whose distances are not proven exact.
- Using `--no_ilp_threshold` can produce longer runtimes and violations of the triangle equation.
- The MCES < 4 'identical or similar' criterion lumps positional structural isomers (MCES = 2) together with identical compounds, so it does not resolve isomer-level differences.

## Evidence

- [methods] Compute Tanimoto similarity for each compound pair using Morgan fingerprints (radius 2, 2048 bits).: [evidence span withheld — anchor status: not_found]
- [results] Of all annotations, 94% were identical or similar, with MCES < 4 indicating identical or structurally similar compounds.: [evidence span withheld — anchor status: not_found]
- [readme] Reference implementation for computing myopic Maximum Common Edge Subgraph (MCES) distances.: [evidence span withheld — anchor status: not_found]
- [readme] myopic-mces input/output CSV schema.: [evidence span withheld — anchor status: external_doc]
- [readme] Time-limit trustworthiness of computed distances.: [evidence span withheld — anchor status: external_doc]
