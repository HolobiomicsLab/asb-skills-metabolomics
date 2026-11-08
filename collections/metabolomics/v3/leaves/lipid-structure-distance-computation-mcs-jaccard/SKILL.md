---
name: lipid-structure-distance-computation-mcs-jaccard
description: Use when pairwise structural distances or similarities between lipid
  species must be derived from their molecular structures (e.g., to build a structural
  space PCA model, an arc-shaped lipid ordering, or lipidome-level similarity), rather
  than from exact name matches or abundances.
when_to_use_negative:
- The analysis only needs comparisons of lipid species by exact name/identifier matches
  across homogeneous lipidomes — structural distance computation is unnecessary overhead.
- Quantitative abundance patterns, not molecular structure, should drive the comparison
  — note that quantitative information is not added to the structural comparison models,
  which can be influenced by low-abundance lipid species.
- Only one fatty acyl chain class or single-chain lipids without headgroup/FA comparison
  semantics are involved, where LaWeCSE headgroup and sn-order-aware FA comparison
  rules do not apply.
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0092
tools:
- name: LipidSpace
  role: Standalone tool that performs the MCS + Jaccard pairwise distance computation
    and builds the structural space PCA models; menu option 'Analysis → Ignore lipid
    sn-positions' toggles combinatorial fatty acyl comparison.
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin (Goslin)
  role: Parses and standardizes lipid names before structural comparison, resolving
    headgroups and fatty acyl/sn-position annotations.
  repo: https://github.com/lifs-tools/cppgoslin
- name: lambda-lanczos
  role: Header-only Lanczos implementation used to run PCA on the distance matrix,
    computing only the first n' < n principal components.
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Library for fast matrix multiplication used in the PCA on the distance matrix.
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:0cc48a4d17677721003bf806477444c515b9ec1ebde1b755be6e91217038c993
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LIPID MAPS
  - LipidSpace
  - OpenBLAS
  - cppgoslin (Goslin)
  - lambda-lanczos
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lipid-structure-distance-computation-mcs-jaccard

## Summary

Computes pairwise structural distances between lipid species by combining the maximum common subgraph (MCS/LaWeCSE) approach with the Jaccard index, yielding a distance = 1 - similarity in [0, 1] that underlies LipidSpace's structural space PCA models and lipidome similarity computations.

## When to use

Use when pairwise structural distances or similarities between lipid species must be derived from their molecular structures (e.g., to build a structural space PCA model, an arc-shaped lipid ordering, or lipidome-level similarity), rather than from exact name matches or abundances.

## When NOT to use

- The analysis only needs comparisons of lipid species by exact name/identifier matches across homogeneous lipidomes — structural distance computation is unnecessary overhead.
- Quantitative abundance patterns, not molecular structure, should drive the comparison — note that quantitative information is not added to the structural comparison models, which can be influenced by low-abundance lipid species.
- Only one fatty acyl chain class or single-chain lipids without headgroup/FA comparison semantics are involved, where LaWeCSE headgroup and sn-order-aware FA comparison rules do not apply.

## Inputs

- Lipid species names/identifiers (e.g., PC 12:0/[12-24,26]:0 series, PC 18:0_16:1) parsed from csv/xlsx/mzTab-M tables or retrieved from LIPID MAPS / SwissLipids
- Molecular structures of lipids at the highest structural resolution level from LIPID MAPS / SwissLipids
- Optionally lipid abundance values (for downstream Hausdorff lipidome distances)

## Outputs

- An n x n pairwise structural distance matrix with entries distance = 1 - Jaccard similarity, bounded in [0, 1]
- A structural space model: PCA (first 7 principal components by default) computed on the distance matrix via the Lanczos method
- Per-lipid coordinates on the first/second principal components (e.g., pc_series_structural_space.csv for a 14-lipid PC series)

## How to apply

Parse and standardize all lipid names first with the C++ Goslin library (cppgoslin) so that headgroups, fatty acyl chain lengths, double-bond positions, and sn-positions are resolved at the highest structural resolution available from LIPID MAPS/SwissLipids. Then compute the n x n pairwise distance matrix by applying the maximum common subgraph (MCS) approach — in LipidSpace implemented as LaWeCSE for headgroups plus fatty acyl comparison — combined with the Jaccard index, defining distance = 1 - similarity so values are bounded between 0 (identical lipids) and 1. By default, fatty acyl chains are compared in sn-order (first FA of lipid 1 vs first FA of lipid 2, second vs second); when sn-positions are unspecified (e.g., PC 18:0_16:1), optionally activate 'Ignore lipid sn-positions' to compare all FA combinations and keep the lowest distance, trading performance for accuracy. Feed the resulting distance matrix into a PCA using the lambda-lanczos implementation of the Lanczos method (default 7 principal components, OpenBLAS-accelerated matrix multiplication) to obtain the structural space model; lipid abundances, normalized to the variance of the first PC, can be added afterwards for lipidome-level (Hausdorff) distances.

## Related tools

- **LipidSpace** (Standalone tool that performs the MCS + Jaccard pairwise distance computation and builds the structural space PCA models; menu option 'Analysis → Ignore lipid sn-positions' toggles combinatorial fatty acyl comparison.) — https://github.com/lifs-tools/lipidspace
- **cppgoslin (Goslin)** (Parses and standardizes lipid names before structural comparison, resolving headgroups and fatty acyl/sn-position annotations.) — https://github.com/lifs-tools/cppgoslin
- **lambda-lanczos** (Header-only Lanczos implementation used to run PCA on the distance matrix, computing only the first n' < n principal components.) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Library for fast matrix multiplication used in the PCA on the distance matrix.) — https://www.openblas.net/

## Examples

```
unzip LipidSpace.zip && ./LipidSpace.sh  # then File → Import example dataset; toggle Analysis → Ignore lipid sn-positions if sn-positions are unspecified
```

## Evaluation signals

- Identical lipid structures must yield a distance of exactly 0, and all pairwise distances must fall within [0, 1] under the default bound distance metric.
- For a homologous lipid series (e.g., 14 diacylglycerophosphocholines PC 12:0/[12-24,26]:0), the first/second PC coordinates must reproduce the sequential arc-like ordering reported in the paper (Supporting Figure S7), both isolated and when embedded among ~500 background lipids.
- Lipid names must be successfully parsed/standardized by Goslin before distance computation; unparsable names indicate the structural comparison cannot proceed.
- With sn-positions unspecified (e.g., PC 18:0_16:1), results computed with 'Ignore lipid sn-positions' activated should be more accurate (lowest distance over all FA combinations) than the default sn-order-aware comparison.

## Limitations

- Default mode compares fatty acyl chains in sn-order (first with first, second with second); when sn-positions are unspecified this can be inaccurate, and the combinatorial alternative decreases performance.
- Structural comparison models do not include quantitative information, so they can be heavily influenced by low-abundant lipid species that do or do not appear across samples.
- The optional 'unbound distance metric' (0 to infinity) provides more accurate results but reduces the visibility of the lipid spaces since distances may become very big.

## Evidence

- [intro] MCS + Jaccard definition of similarity: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] Goslin used to standardize names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] PCA on the distance matrix with Lanczos: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [readme] Default sn-order-aware fatty acyl comparison: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid, the second FA of the first lipid with the second FA of the second"
- [readme] Bound distance metric range: "the distance is a value that ranges between 0 (both lipids are identical) and 1"
