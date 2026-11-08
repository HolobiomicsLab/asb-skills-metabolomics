---
name: structural-ordering-robustness-comparison-across-embeddings
description: Use when you must confirm that relative positions of a chosen lipid series
  in a graph-based structural space remain stable after recomputation within a larger
  background lipidome, e.g. reproducing the arc-like PC ordering.
when_to_use_negative:
- The analysis goal is quantitative (abundance-based) comparison of lipidomes — abundances
  are deliberately excluded from structural space models, so this skill cannot validate
  intensity-driven conclusions
- Lipid names cannot be resolved to full structures (e.g. species without defined
  fatty acyl composition), since MCS/Jaccard distances require structural resolution
  from LIPID MAPS/SwissLipids
- You need a standalone numeric pipeline output rather than an interactive check —
  LipidSpace currently cannot store or load an analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidSpace
  role: Standalone workstation application that computes the MCS/Jaccard distance
    matrix and the PCA structural space models to be compared
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names so each species is mapped to its structural
    representation before distance computation
  repo: https://github.com/lifs-tools/cppgoslin
- name: lambda-lanczos
  role: Header-only C++ Lanczos implementation used to compute only the first n' <
    n principal components of the distance matrix in both models
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Provides fast matrix multiplication underlying the distance/PCA computations
  repo: https://www.openblas.net/
- name: LIPID MAPS
  role: Source database for the molecular structures of the lipid species being compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:ae8b47e7004e765b7b9acbcbec1a586e22f8a257da81ad6c4ea6ef197d265baf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LIPID MAPS
  - LipidSpace
  - OpenBLAS
  - lambda-lanczos
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# structural-ordering-robustness-comparison-across-embeddings

## Summary

Verifies that the relative spatial ordering of a defined lipid subset in a structural space (built from maximum-common-subgraph + Jaccard distances and Lanczos PCA) is preserved when the subset is embedded in a larger background lipidome. In LipidSpace, the 14 PC 12:0/[12−24,26]:0 species retain their arc-like sequential arrangement in both the isolated and the embedded model.

## When to use

Use when you must confirm that relative positions of a chosen lipid series in a graph-based structural space remain stable after recomputation within a larger background lipidome, e.g. reproducing the arc-like PC ordering.

## When NOT to use

- The analysis goal is quantitative (abundance-based) comparison of lipidomes — abundances are deliberately excluded from structural space models, so this skill cannot validate intensity-driven conclusions
- Lipid names cannot be resolved to full structures (e.g. species without defined fatty acyl composition), since MCS/Jaccard distances require structural resolution from LIPID MAPS/SwissLipids
- You need a standalone numeric pipeline output rather than an interactive check — LipidSpace currently cannot store or load an analysis

## Inputs

- Lipid species names of the target subset (e.g. 14 PC 12:0/[12−24,26]:0 species) parsed via Goslin
- Molecular structures retrieved from LIPID MAPS (or SwissLipids) at the highest structural resolution level
- Pairwise structural distance matrix (distance = 1 − Jaccard similarity of the maximum common subgraph)
- Background lipidome set (~500 lipids from different categories) for the embedded model

## Outputs

- First/second PC coordinates of the subset lipids from both the isolated and embedded structural space models (saved as pc_series_structural_space.csv)
- Qualitative ordering comparison (arc-like sequential positioning) against the published reference ordering
- Recomputed global distance matrix and PCA model for the embedded configuration

## How to apply

First build the isolated structural model: parse the lipid names (e.g. the 14 diacylglycerophosphocholines PC 12:0/[12−24,26]:0) with Goslin, fetch molecular structures from LIPID MAPS, and compute the full n×n pairwise distance matrix as 1 − Jaccard similarity of the maximum common subgraph (LaWeCSE for headgroups, sn-order-aware fatty acyl comparison). Run a PCA on this distance matrix with the Lanczos method (default 7 principal components, OpenBLAS-accelerated linear algebra) and project each lipid onto the first and second principal components. Then embed the same lipids in a large background set (~500 lipids from different categories), recompute the global pairwise distance matrix, and repeat the Lanczos PCA. Extract PC1/PC2 coordinates of the subset from both models and check that the sequential ordering — here the arc shape — is preserved, allowing slight distortion, against the published ordering (Supporting Figure S7).

## Related tools

- **LipidSpace** (Standalone workstation application that computes the MCS/Jaccard distance matrix and the PCA structural space models to be compared) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid names so each species is mapped to its structural representation before distance computation) — https://github.com/lifs-tools/cppgoslin
- **lambda-lanczos** (Header-only C++ Lanczos implementation used to compute only the first n' < n principal components of the distance matrix in both models) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Provides fast matrix multiplication underlying the distance/PCA computations) — https://www.openblas.net/
- **LIPID MAPS** (Source database for the molecular structures of the lipid species being compared)

## Examples

```
LipidSpace.sh  # then: Analysis → Ignore lipid sn-positions (if needed), recompute structural space with and without the ~500-lipid background, and compare PC1/PC2 ordering
```

## Evaluation signals

- The 14 PC-series lipids preserve their arc-like sequential positioning in both the isolated and the embedded structural space model, allowing slight distortion
- The extracted ordering matches the published reference ordering (Supporting Figure S7)
- All pairwise distance values are bounded in [0, 1] under the default bound distance metric (0 = identical lipids)
- The same sn-order-aware fatty acyl comparison mode is used for both the isolated and the embedded distance matrix, so ordering differences reflect embedding and not a metric change
- Both models are computed with the same PCA settings (default 7 principal components via Lanczos/OpenBLAS), making coordinates directly comparable

## Limitations

- When sn-positions are unspecified (e.g. PC 18:0_16:1), the more accurate all-combinations FA comparison mode decreases performance; the default first-FA/second-FA alignment is faster but less accurate
- Switching to the unbound distance metric (0 to infinity) gives more accurate results but reduces the visibility of the lipid spaces because distances may become very big
- Ordering agreement is judged qualitatively with 'slight distortion' tolerated when embedded, so borderline reordering near the arc's ends may be ambiguous
- LipidSpace analyses cannot currently be saved or loaded, so the isolated and embedded model comparisons must be completed within one session

## Evidence

- [intro] MCS/Jaccard basis: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] PCA on the distance matrix: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method"
- [readme] Default fatty acyl comparison mode: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
- [readme] Unbound metric caveat: "This mode provides more accurate results but reduces the visibility of the lipid spaces since the distances may become very big."
