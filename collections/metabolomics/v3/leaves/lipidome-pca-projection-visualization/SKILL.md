---
name: lipidome-pca-projection-visualization
description: 'Use when lipidome samples must be placed into a shared structural space:
  an n x n lipid-species distance matrix is available and per-lipidome coordinates
  plus reportable projection figures are the deliverable.'
when_to_use_negative:
- The input is already per-lipidome coordinates or a finished PCA model rather than
  a structural distance matrix to be decomposed
- The analysis goal is feature/variable selection between study variables, which is
  handled by the feature analysis module with sequential forward selection rather
  than PCA projection
- Only a single lipidome is of interest and no cross-lipidome structural-space comparison
  is intended
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: LipidSpace
  role: Workstation application that performs the PCA on the distance matrix, projects
    lipidomes via PC1/PC2, exports lipid_space_pca_projection.csv, and produces the
    ready-to-use result figures
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: Header-only C++ Lanczos-algorithm implementation used to compute only the
    first n' < n principal components of the distance matrix
  repo: https://github.com/mrcdr/lambda-lanczos
- name: OpenBLAS
  role: Library for fast matrix multiplication underlying the PCA computation
  repo: https://www.openblas.net/
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names so the species entering the distance matrix
    and the projection output are consistently identified
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:9bb608ef5a2dc921d03b09afe4697d791683713c785f2077a4d11cfa27b0d9b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LipidSpace
  - OpenBLAS
  - lambda-lanczos
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lipidome-pca-projection-visualization

## Summary

Project lipidomes into the global lipid space model by running PCA on the pairwise lipid-structure distance matrix with the lambda-lanczos implementation, then deliver PC1/PC2 coordinates and ready-to-use result figures (Figure 1F) as exportable output.

## When to use

Use when lipidome samples must be placed into a shared structural space: an n x n lipid-species distance matrix is available and per-lipidome coordinates plus reportable projection figures are the deliverable.

## When NOT to use

- The input is already per-lipidome coordinates or a finished PCA model rather than a structural distance matrix to be decomposed
- The analysis goal is feature/variable selection between study variables, which is handled by the feature analysis module with sequential forward selection rather than PCA projection
- Only a single lipidome is of interest and no cross-lipidome structural-space comparison is intended

## Inputs

- n x n pairwise distance matrix between lipid species (from LaWeCSE/Jaccard structural comparison)
- Imported lipidomics table (csv/xlsx flat or pivot format, or mzTab-M) parsed with Goslin
- Per-lipidome lipid abundances normalized to the variance of the first PC

## Outputs

- Global lipid space model with the default 7 principal components
- Per-lipidome PC1/PC2 (x/y) Cartesian coordinates
- lipid_space_pca_projection.csv containing lipid names, intensities, and X/Y values
- Ready-to-use result figures from the statistics module (Figure 1F)

## How to apply

Compute the complete pairwise structural distance matrix between distinct lipid species using the maximum common subgraph (LaWeCSE) approach combined with the Jaccard index, then run a principal component analysis on that complete matrix. Use the lambda-lanczos header-only library to compute only the first n' < n principal components (7 by default), accelerating the matrix operations with OpenBLAS. Project each lipidome into the model using the first and second principal components as x and y Cartesian coordinates, adding lipid abundances normalized to the variance of the first PC before applying pairwise Hausdorff distance computations between lipidomes. Export per-lipidome lipid names, intensities, and X/Y values to lipid_space_pca_projection.csv, and package the outcomes as ready-to-use result figures via the statistics module (Figure 1F).

## Related tools

- **LipidSpace** (Workstation application that performs the PCA on the distance matrix, projects lipidomes via PC1/PC2, exports lipid_space_pca_projection.csv, and produces the ready-to-use result figures) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Header-only C++ Lanczos-algorithm implementation used to compute only the first n' < n principal components of the distance matrix) — https://github.com/mrcdr/lambda-lanczos
- **OpenBLAS** (Library for fast matrix multiplication underlying the PCA computation) — https://www.openblas.net/
- **Goslin (cppgoslin)** (Parses and standardizes lipid names so the species entering the distance matrix and the projection output are consistently identified) — https://github.com/lifs-tools/cppgoslin

## Examples

```
LambdaLanczos<double> engine(mv_mul, n, true, 7); vector<double> eigenvalues; vector<vector<double>> eigenvectors; engine.run(eigenvalues, eigenvectors);
```

## Evaluation signals

- The Lanczos run resolves the default seven principal components from the n x n distance matrix
- The PCA plot maps PC1 onto the x axis and PC2 onto the y axis of the lipid space model
- lipid_space_pca_projection.csv is produced containing, per lipidome, lipid names, intensities, and X/Y values
- Lipid abundances added to the model are normalized with respect to the variance of the first PC
- Result figures are exportable/packaged as ready-to-use output (Figure 1F) for user reporting

## Limitations

- Only the first n' < n principal components are computed (7 by default), so projections use PC1/PC2 rather than a full spectral decomposition
- The projection is built on the structural distance matrix, and quantitative information is only added afterward normalized to the first PC's variance, so low-abundant species shape the underlying structural model
- LipidSpace is designed as a workstation application for a regular office computer; large matrix operations rely on OpenBLAS for speed
- Distance metric choice affects visibility: the unbound distance metric gives more accurate results but reduces visibility of the lipid spaces because distances may become very big

## Evidence

- [methods] PCA via Lanczos: "an implementation of the Lanczos method17 [https://github.com/mrcdr/lambda-lanczos] to compute only the first n’ < n principal components"
- [methods] PCA on complete matrix: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [methods] Abundance normalization and Hausdorff: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model."
- [methods] OpenBLAS: "We are utilizing the openBLAS [https://www.openblas.net/] library for fast matrix multiplication"
- [intro] Statistics module figures: "The statistics module produces ready-to-use result figures"
- [readme] Distance metric caveat: "This mode provides more accurate results but reduces the visibility of the lipid spaces since the distances may become very big."
