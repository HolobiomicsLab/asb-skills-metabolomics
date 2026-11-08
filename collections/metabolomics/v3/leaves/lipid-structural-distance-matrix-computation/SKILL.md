---
name: lipid-structural-distance-matrix-computation
description: Use when lipid species names have been parsed/standardized (e.g., via
  Goslin) and a pairwise structural distance/similarity matrix between all distinct
  lipid species is required for downstream PCA, lipidome comparison, or dendrogram
  clustering.
when_to_use_negative:
- The input is already a quantitative feature matrix and the goal is sample-level
  statistics — quantitative information is deliberately not added to this structural
  comparison model (abundances are only added downstream, normalized to the variance
  of the first PC)
- Lipid names are non-standardized or from heterogeneous dialects — running structural
  comparison without prior Goslin parsing/standardization yields invalid FA-chain
  comparisons
- Only exact species-name matching across lipidomes is needed — the article positions
  this skill precisely for the case where exact-name matching is insufficient, e.g.,
  heterogeneous lipidomes
tools:
- name: LipidSpace
  role: Implements the maximum common subgraph (LaWeCSE) + Jaccard distance computation
    and assembles the n×n lipid structural distance matrix feeding the PCA model
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin (Goslin)
  role: Parses and standardizes lipid names (unifying dialects) prior to structural
    FA-chain comparison
  repo: https://github.com/lifs-tools/cppgoslin
- name: lambda-lanczos
  role: Consumes the distance matrix downstream to compute the first n' < n principal
    components (eigenpairs) of the structural space model
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Provides fast matrix multiplication for the linear-algebra operations on the
    distance matrix
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:a09edf5a4379373f18080f4e70ca1a6ddc40bf4cdb8ace0600b104e356720c4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - LipidSpace
  - OpenBLAS
  - cppgoslin (Goslin)
  - lambda-lanczos
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# lipid-structural-distance-matrix-computation

## Summary

Computes a pairwise structural distance matrix between lipid species by combining the maximum common subgraph (LaWeCSE) approach with the Jaccard index, forming the basis for LipidSpace's structural space models and lipidome similarity computations.

## When to use

Use when lipid species names have been parsed/standardized (e.g., via Goslin) and a pairwise structural distance/similarity matrix between all distinct lipid species is required for downstream PCA, lipidome comparison, or dendrogram clustering.

## When NOT to use

- The input is already a quantitative feature matrix and the goal is sample-level statistics — quantitative information is deliberately not added to this structural comparison model (abundances are only added downstream, normalized to the variance of the first PC)
- Lipid names are non-standardized or from heterogeneous dialects — running structural comparison without prior Goslin parsing/standardization yields invalid FA-chain comparisons
- Only exact species-name matching across lipidomes is needed — the article positions this skill precisely for the case where exact-name matching is insufficient, e.g., heterogeneous lipidomes

## Inputs

- Standardized lipid species names (parsed/normalized via Goslin/cppgoslin)
- Lipidomics data table (csv or xlsx, flat or pivot format, or mzTab-M) from which the distinct lipid species are extracted
- Configuration options: sn-position handling (positional vs. all-combinations), bound vs. unbound distance metric

## Outputs

- n×n pairwise structural distance matrix between distinct lipid species
- Input matrix for the global lipid space PCA model (default 7 principal components) and for lipidome dendrogram/clustering views

## How to apply

First parse and standardize all lipid names with the Goslin C++ library (cppgoslin) so species are directly comparable. Then, for every pair of distinct lipid species, apply the maximum common subgraph approach (LaWeCSE) in combination with the Jaccard index to determine structural similarity based on molecular structure, converting this into a distance. By default the comparison is positional — the first fatty acyl (FA) chain of one lipid vs. the first FA of the other, the second FA vs. the second FA, etc. — producing a bounded distance between 0 (identical lipids) and 1. If sn-positions are unspecified (e.g., PC 18:0_16:1), optionally activate 'Ignore lipid sn-positions' to compare all FA combinations and take the lowest distance (more accurate, slower); optionally switch to an unbound distance metric ranging from 0 to infinity. Assemble all pairwise distances into an n×n distance matrix (n = number of distinct lipid species), which is then ready for PCA via the lambda-lanczos implementation with OpenBLAS-backed matrix operations.

## Related tools

- **LipidSpace** (Implements the maximum common subgraph (LaWeCSE) + Jaccard distance computation and assembles the n×n lipid structural distance matrix feeding the PCA model) — https://github.com/lifs-tools/lipidspace
- **cppgoslin (Goslin)** (Parses and standardizes lipid names (unifying dialects) prior to structural FA-chain comparison) — https://github.com/lifs-tools/cppgoslin
- **lambda-lanczos** (Consumes the distance matrix downstream to compute the first n' < n principal components (eigenpairs) of the structural space model) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Provides fast matrix multiplication for the linear-algebra operations on the distance matrix) — https://www.openblas.net/

## Evaluation signals

- All pairwise distances fall in the bounded range [0, 1] (0 = identical lipids) when the default bound metric is active
- The resulting matrix is square n×n for the n distinct lipid species in the dataset and is symmetric with zero diagonal
- Double bond comparisons are correct per the article's dedicated 'Correctness of Double Bond Comparisons' validation criterion
- A PCA run on the matrix (lambda-lanczos) resolves the default seven principal components, with PC1/PC2 mapping onto the x/y axes of the projection plot
- With 'Ignore lipid sn-positions' enabled, the per-pair distance equals the minimum across all FA-chain combinations and matches distances of lipids with unspecified sn-positions (e.g., PC 18:0_16:1)

## Limitations

- Default positional FA comparison (first FA vs. first FA, etc.) is less accurate when sn-positions are unspecified; the all-combinations mode improves accuracy but decreases performance
- The unbound distance metric (0 to infinity) yields more accurate results but reduces the visibility of lipid spaces since distances may become very large
- Structural distances ignore quantitative abundances; low-abundant species influence the model unless abundances are added downstream, normalized to the variance of the first PC

## Evidence

- [intro] we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [readme] In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
- [readme] As default, LipidSpace is using a bound distance metric to compare the structure of any two lipids. That means that the distance is a value that ranges between 0 (both lipids are identical) and 1.: "the distance is a value that ranges between 0 (both lipids are identical) and 1"
- [methods] It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
