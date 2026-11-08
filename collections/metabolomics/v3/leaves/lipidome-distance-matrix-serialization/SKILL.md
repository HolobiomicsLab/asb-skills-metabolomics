---
name: lipidome-distance-matrix-serialization
description: Use when a multi-lipidome dataset must be reduced to a single pairwise
  distance matrix relating all lipidomes structurally, as the basis for hierarchical
  clustering or QC of large-scale lipidomics studies.
when_to_use_negative:
- When only a single lipidome is analyzed — no pairwise lipidome distances exist to
  compute; a per-lipidome structural space model is built instead.
- When the goal is identifying lipids that distinguish study variables — use the feature
  analysis module (SFS) rather than the lipidome distance matrix.
- When analysis state must be persisted between sessions — the README states it is
  not possible to store or load an analysis.
edam_operation: http://edamontology.org/operation_0505
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: LipidSpace
  role: Standalone workstation application that builds per-lipidome structural space
    models, computes the pairwise Hausdorff LipidomeDistanceMatrix, and clusters it
    into a dendrogram
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: Header-only Lanczos implementation used to compute only the first n' < n (default
    7) principal components of the lipid structural distance matrix
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Linear algebra library used for fast matrix multiplication during the PCA
    and distance computations
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:936be0c6e74baae67edfbdc3ce06f8e88f4f4038922e3daf482479b5b3e70edb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - OpenBLAS
  - lambda-lanczos
  edam_operation: http://edamontology.org/operation_0505
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lipidome-distance-matrix-serialization

## Summary

Computes and serializes the global m x m LipidomeDistanceMatrix by applying pairwise Hausdorff distance computations between all lipidomes over a shared structural space (PCA via Lanczos with 7 principal components), which is then clustered hierarchically into a dendrogram.

## When to use

Use when a multi-lipidome dataset must be reduced to a single pairwise distance matrix relating all lipidomes structurally, as the basis for hierarchical clustering or QC of large-scale lipidomics studies.

## When NOT to use

- When only a single lipidome is analyzed — no pairwise lipidome distances exist to compute; a per-lipidome structural space model is built instead.
- When the goal is identifying lipids that distinguish study variables — use the feature analysis module (SFS) rather than the lipidome distance matrix.
- When analysis state must be persisted between sessions — the README states it is not possible to store or load an analysis.

## Inputs

- Multi-lipidome lipidomics dataset (csv or xlsx, flat or pivot table layout; lipid species rows or columns with samples transposed)
- Lipidomics data in mzTab-M format
- Bundled example JSON dataset (examples/Rest/Plasma-Singapore-Short.json) submitted via POST to /lipidspace/v1/pca

## Outputs

- m x m symmetric LipidomeDistanceMatrix with zero diagonal
- Interactive dendrogram of the global lipidome distance model, exportable as PDF

## How to apply

Load the multi-lipidome dataset (csv/xlsx flat or pivot tables, mzTab-M, or the bundled examples/Rest/Plasma-Singapore-Short.json, optionally via POST to the /lipidspace/v1/pca endpoint) and assemble the union set of distinct lipid species across all lipidomes. For each lipidome, compute PCA on the n x n lipid structural distance matrix (maximum common subgraph via LaWeCSE plus Jaccard index) using the Lanczos method with 7 principal components and openBLAS-accelerated linear algebra. Normalize each lipidome's abundances with respect to the variance of the first PC and add them to the model, then compute all pairwise Hausdorff distances between the m lipidomes, yielding a symmetric m x m matrix with zero diagonal. Cluster this matrix with agglomerative hierarchical clustering using unweighted average linkage by default (single and complete linkage selectable in the menu Analysis → Clustering strategy) and render the dendrogram, exportable as PDF. Rationale: quantitative information normalized to the first PC prevents the structural comparison from being dominated by low-abundant lipid species.

## Related tools

- **LipidSpace** (Standalone workstation application that builds per-lipidome structural space models, computes the pairwise Hausdorff LipidomeDistanceMatrix, and clusters it into a dendrogram) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Header-only Lanczos implementation used to compute only the first n' < n (default 7) principal components of the lipid structural distance matrix) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Linear algebra library used for fast matrix multiplication during the PCA and distance computations) — https://www.openblas.net/

## Examples

```
curl -X POST --data-binary @examples/Rest/Plasma-Singapore-Short.json http://localhost:8080/lipidspace/v1/pca
```

## Evaluation signals

- The LipidomeDistanceMatrix is symmetric with dist(A,B) = dist(B,A) and a zero diagonal.
- Matrix values match the reference REST response for the example dataset, e.g. 1.13403 between the first two lipidomes of Plasma-Singapore-Short.json.
- PCA is performed with the default 7 principal components via the Lanczos method; changing components or abundances handling (quantities switchable off) changes the matrix.
- Default clustering is unweighted average linkage; switching to single or complete linkage must alter the dendrogram but not the underlying distance matrix.
- Dendrogram export produces a PDF of the hierarchical clustering of the m lipidomes.

## Limitations

- The unbound distance metric mode (range 0 to infinity) gives more accurate results but reduces the visibility of the lipid spaces since distances may become very big.
- Comparing all fatty acyl chain combinations when sn-positions are unspecified (Ignore lipid sn-positions mode) is more accurate but decreases performance.
- Analyses cannot be saved or loaded; the README notes this feature is currently not possible.

## Evidence

- [methods] PCA on the lipid distance matrix: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [methods] Abundance normalization and pairwise lipidome distance computation: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes"
- [readme] Selectable linkage strategies for the dendrogram: "The user can switch between single linkage, unweighted average, and complete linkage clustering in the menu → Analysis → Clustering strategy."
- [methods] OpenBLAS acceleration: "We are utilizing the openBLAS [https://www.openblas.net/] library for fast matrix multiplication"
