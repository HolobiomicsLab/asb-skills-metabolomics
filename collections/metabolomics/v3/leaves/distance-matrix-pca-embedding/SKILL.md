---
name: distance-matrix-pca-embedding
description: 'Use when you must reproduce or validate a LipidSpace-style structural
  lipid space: a set of lipid species (e.g. a homologous PC series) needs a pairwise
  structural distance matrix embedded by PCA, or an arc-like sequential PC1/PC2 ordering
  must be verified.'
when_to_use_negative:
- The input is already a feature/abundance table and quantitative information should
  shape the model - the article notes quantitative information is not added to these
  structural comparison models, so low-abundant species are weighted equally.
- Lipid names cannot be parsed into fully resolved structures (e.g. sum-composition
  level species without headgroup/acyl detail), since the MCS comparison needs molecular
  graphs.
- You need exact name-identifier matching between lipidomes rather than structural
  similarity - LipidSpace explicitly targets the structural-comparison use case instead.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidSpace
  role: Reference implementation of the MCS + Jaccard distance computation, Lanczos
    PCA structural space, and background embedding logic
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin (Goslin)
  role: Parses and standardizes lipid names into structured representations used to
    build the molecular graphs for comparison
  repo: https://github.com/lifs-tools/cppgoslin
- name: lambda-lanczos
  role: Header-only Lanczos eigensolver used to compute only the first n' < n principal
    components of the distance matrix
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Fast matrix multiplication backend for the distance-matrix and PCA linear
    algebra
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:82ff7d63a60fc98caf51fc6f78410f381e8820af14e8aa0bd4497c19a0a2f8ae
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# distance-matrix-pca-embedding

## Summary

Reproduces LipidSpace's structural space model: pairwise lipid structural distances are computed via maximum common subgraph (MCS/LaWeCSE headgroups plus sn-order-aware fatty acyl comparison) combined with the Jaccard index, then embedded by PCA on the n x n distance matrix using the Lanczos method.

## When to use

Use when you must reproduce or validate a LipidSpace-style structural lipid space: a set of lipid species (e.g. a homologous PC series) needs a pairwise structural distance matrix embedded by PCA, or an arc-like sequential PC1/PC2 ordering must be verified.

## When NOT to use

- The input is already a feature/abundance table and quantitative information should shape the model - the article notes quantitative information is not added to these structural comparison models, so low-abundant species are weighted equally.
- Lipid names cannot be parsed into fully resolved structures (e.g. sum-composition level species without headgroup/acyl detail), since the MCS comparison needs molecular graphs.
- You need exact name-identifier matching between lipidomes rather than structural similarity - LipidSpace explicitly targets the structural-comparison use case instead.

## Inputs

- Lipid species names (e.g. PC 12:0/[12-24,26]:0 series) parsed and standardized with Goslin/cppgoslin
- Molecular structures of the lipids retrieved from LIPID MAPS or SwissLipids at highest structural resolution
- Optional background lipid set (e.g. ~500 lipids across different categories) for embedded-model testing

## Outputs

- n x n pairwise structural distance matrix (bound metric: values in [0,1], 0 for identical lipids)
- Structural space model: PC coordinates for each lipid, projected onto the first and second principal components
- Coordinate table for the target lipid series (e.g. pc_series_structural_space.csv) for isolated and embedded models

## How to apply

Retrieve molecular structures at the highest structural resolution level (e.g. from LIPID MAPS or SwissLipids) and parse/standardize lipid names with Goslin (cppgoslin). Compute the n x n pairwise distance matrix using the maximum common subgraph approach (LaWeCSE for headgroups, sn-order-aware fatty acyl comparison) combined with the Jaccard index, with distance = 1 - similarity; by default the first fatty acyl of one lipid is compared to the first fatty acyl of the other, in positional order, and the metric is bound to [0, 1]. Run a principal component analysis on the complete distance matrix using the lambda-lanczos implementation of the Lanczos method (default: only the first 7 principal components, n' < n) with OpenBLAS-accelerated matrix multiplication. To test embedding robustness, embed the target lipids in a background set of ~500 lipids from different categories, recompute the global distance matrix, repeat the PCA, and project everything onto the first and second principal components. Verify that the target lipids preserve their relative ordering in PC1/PC2 in both the isolated and embedded models, allowing only slight distortion.

## Related tools

- **LipidSpace** (Reference implementation of the MCS + Jaccard distance computation, Lanczos PCA structural space, and background embedding logic) — https://github.com/lifs-tools/lipidspace
- **cppgoslin (Goslin)** (Parses and standardizes lipid names into structured representations used to build the molecular graphs for comparison) — https://github.com/lifs-tools/cppgoslin
- **lambda-lanczos** (Header-only Lanczos eigensolver used to compute only the first n' < n principal components of the distance matrix) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Fast matrix multiplication backend for the distance-matrix and PCA linear algebra) — https://www.openblas.net/

## Examples

```
LambdaLanczos<double> engine(mv_mul, n, true, 1); engine.run(eigenvalues, eigenvectors); // largest eigenpairs of the distance matrix via lambda-lanczos
```

## Evaluation signals

- Distance matrix is symmetric, has a zero diagonal, and with the default bound metric all values lie in [0,1] with 0 only for identical lipids.
- The target lipid series (e.g. the 14 PC 12:0/[12-24,26]:0 species) preserve their sequential positioning in an arc along PC1/PC2 in both the isolated and the 500-lipid-embedded model, allowing slight distortion.
- The recovered ordering matches the published ordering (e.g. Supporting Figure S7 of the LipidSpace paper).
- PCA uses the Lanczos method with the default 7 principal components (n' < n), not a full eigendecomposition.
- Sn-order-aware comparison: with sn-positions specified, FA1-FA1/FA2-FA2 pairing is used; activating 'Ignore lipid sn-positions' should lower distances for unspecified-position species.

## Limitations

- By default the distance metric is bound to [0,1]; the unbound mode is more accurate but distances may become very big and reduce the visibility of the lipid spaces.
- Default FA comparison is positional (FA1 vs FA1, FA2 vs FA2); when sn-positions are unspecified (e.g. PC 18:0_16:1) the all-combinations mode gives more accurate but slower results (decreased performance).
- Structural-space models exclude quantitative information, so they are heavily influenced by low abundant lipid species that do or do not appear across samples.
- LipidSpace cannot currently store/load an analysis, so a reproduced pipeline must persist intermediate matrices and coordinates itself.

## Evidence

- [intro] MCS + Jaccard distance: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] Lanczos PCA: "an implementation of the Lanczos method17 [https://github.com/mrcdr/lambda-lanczos] to compute only the first n’ < n principal components"
- [methods] OpenBLAS: "We are utilizing the openBLAS [https://www.openblas.net/] library for fast matrix multiplication"
- [methods] Goslin parsing: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [readme] Bound distance metric: "As default, LipidSpace is using a bound distance metric to compare the structure of any two lipids. That means that the distance is a value that ranges between 0 (both lipids are identical) and 1."
- [readme] sn-position comparison mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
