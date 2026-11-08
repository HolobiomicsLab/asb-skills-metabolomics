---
name: lanczos-pca-dimensionality-reduction
description: Use when you have a large symmetric distance/similarity matrix (e.g.
  lipid-vs-lipid structural distances) and need only the few principal components
  for projection or visualization.
when_to_use_negative:
- You need the full eigenspectrum or more than a handful of leading components — the
  Lanczos method is tuned to computing only the first n' < n principal components.
- The matrix is not symmetric/Hermitian — lambda-lanczos explicitly calculates eigenvalues/eigenvectors
  of symmetric (Hermitian) matrices.
- The input is raw abundance data rather than a distance matrix — LipidSpace builds
  the structural distance matrix (MCS + Jaccard) before PCA, so PCA directly on raw
  intensities is out of scope for this skill.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: lambda-lanczos
  role: Header-only C++ library that computes the first n' < n principal components
    of the symmetric n x n lipid distance matrix via the Lanczos method; LipidSpace
    embeds it for its PCA step.
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Provides fast matrix multiplication for the distance-matrix operations feeding
    the Lanczos PCA.
  repo: https://www.openblas.net/
- name: LipidSpace
  role: Workstation application whose statistics module wraps the Lanczos PCA, projects
    each lipidome onto PC1/PC2, and exports the results as CSV and ready-to-use figures.
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:cf2f463f75e266f24c7768d052ae7ab49015560aa203e3c698c1c492df806232
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mrcdr/lambda-lanczos
  tools:
  - Goslin
  - LipidSpace
  - OpenBLAS
  - lambda-lanczos
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lanczos-pca-dimensionality-reduction

## Summary

Computes a principal component analysis on an n x n pairwise structural distance matrix using a header-only Lanczos-method implementation (lambda-lanczos), so that only the first n' < n components (default 7) are extracted instead of a full eigendecomposition.

## When to use

Use when you have a large symmetric distance/similarity matrix (e.g. lipid-vs-lipid structural distances) and need only the leading few principal components for projection or visualization.

## When NOT to use

- You need the full eigenspectrum or more than a handful of leading components — the Lanczos method is tuned to computing only the first n' < n principal components.
- The matrix is not symmetric/Hermitian — lambda-lanczos explicitly calculates eigenvalues/eigenvectors of symmetric (Hermitian) matrices.
- The input is raw abundance data rather than a distance matrix — LipidSpace builds the structural distance matrix (MCS + Jaccard) before PCA, so PCA directly on raw intensities is out of scope for this skill.

## Inputs

- n x n pairwise distance matrix between lipid species (from maximum common subgraph + Jaccard index comparison of molecular structures)
- lipidome abundance table (lipid names and intensities per sample, imported from csv/xlsx or mzTab-M)
- number of principal components to compute (default 7 in LipidSpace)

## Outputs

- eigenvalues/eigenvectors of the first n' principal components of the distance matrix
- lipidome PCA coordinates mapped onto PC1 (x) and PC2 (y)
- exported projection table lipid_space_pca_projection.csv (per-lipidome lipid names, intensities, and X/Y values)
- PCA scatter figure (Figure 1F) as ready-to-use reportable output

## How to apply

First build the n x n distance matrix — in LipidSpace this comes from pairwise lipid-species structural distances computed via the maximum common subgraph (LaWeCSE) approach combined with the Jaccard index. Then run PCA on the complete matrix using the lambda-lanczos implementation of the Lanczos method, which computes only the first n' < n principal components; LipidSpace defaults to 7 principal components. Use OpenBLAS for fast matrix multiplication on the distance matrix (the adaptive matrix-vector-multiplication routine of lambda-lanczos can also be wired to any matrix library, e.g. Eigen). Project each observation (lipidome) into the model by taking, by default, the first and second principal components as x and y Cartesian coordinates, and export the per-lipidome lipid names, intensities, and X/Y values (e.g. to lipid_space_pca_projection.csv).

## Related tools

- **lambda-lanczos** (Header-only C++ library that computes the first n' < n principal components of the symmetric n x n lipid distance matrix via the Lanczos method; LipidSpace embeds it for its PCA step.) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Provides fast matrix multiplication for the distance-matrix operations feeding the Lanczos PCA.) — https://www.openblas.net/
- **LipidSpace** (Workstation application whose statistics module wraps the Lanczos PCA, projects each lipidome onto PC1/PC2, and exports the results as CSV and ready-to-use figures.) — https://github.com/lifs-tools/lipidspace

## Examples

```
LambdaLanczos<double> engine(mv_mul, n, true, 1); engine.run(eigenvalues, eigenvectors);
```

## Evaluation signals

- The model resolves the default seven principal components from the n x n distance matrix.
- The PCA plot maps PC1 and PC2 onto the x and y axes as specified in the methods.
- Each lipidome row in lipid_space_pca_projection.csv carries lipid names, intensities, and X/Y coordinates consistent with the first two computed eigenvectors.
- Because the input matrix is symmetric with non-negative Jaccard-based distances, eigenvalues of the leading components should be ordered by decreasing variance and coordinates bounded by the [0,1] distance range of the underlying metric.
- lambda-lanczos returns eigenpairs for a symmetric matrix, so recomputing with the same matrix-vector-multiplication routine should reproduce the same leading eigenvalues.

## Limitations

- Only the first n' < n principal components are computed; the method is not intended for full-spectrum decomposition.
- Applicable to symmetric (Hermitian) matrices only, per the lambda-lanczos design.
- In LipidSpace the resulting lipid space is heavily shaped by structural (not quantitative) information; abundances are normalized to the variance of the first PC only afterwards, and low-abundance lipids can influence structural comparison models.

## Evidence

- [methods] PCA on distance matrix via Lanczos: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [methods] only first n' components computed: "an implementation of the Lanczos method17 [https://github.com/mrcdr/lambda-lanczos] to compute only the first n’ < n principal components"
- [methods] OpenBLAS used for fast matrix operations: "We are utilizing the openBLAS [https://www.openblas.net/] library for fast matrix multiplication"
- [readme] lambda-lanczos scope: symmetric matrix eigenpairs: "**Lambda Lanczos** calculates the smallest or largest eigenvalue and
the corresponding eigenvector of a symmetric (Hermitian) matrix."
- [methods] abundances normalized against first-PC variance after model construction: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model."
