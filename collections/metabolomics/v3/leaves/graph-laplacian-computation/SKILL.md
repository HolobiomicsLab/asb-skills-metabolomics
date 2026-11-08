---
name: graph-laplacian-computation
description: Use when a pathway is represented as a 0/1 symmetric adjacency matrix
  and you need the regularized normalized Laplacian (D + τI)^(-1/2) L (D + τI)^(-1/2),
  τ = 1, for kernel regression.
when_to_use_negative:
- The input matrix is not a symmetric 0/1 adjacency matrix of pathway connections
  (e.g., it is weighted, directed, or a feature/intensity table)
- The graph structure has not been examined first — the authors state it is important
  to examine graph structure before applying regularization
- The analysis does not embed the Laplacian into a kernel machine score test (e.g.,
  plain network visualization or clustering)
tools:
- name: R
  role: Implementation environment for building D, forming L = D − A, applying the
    regularized normalization, eigen-verification, and saving the .rds output; all
    PaIRKAT simulations were done in R
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generation of random Barabasi-Albert pathway graphs whose Laplacians are computed
    for simulation-based validation of the regularization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:c2a01a6fcf83ad9c9b1b8cfe900b03bac6c69f595c317c3ffa183acb3c11b05b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  repo_url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - R
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# graph-laplacian-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the regularized normalized graph Laplacian from a pathway's p×p adjacency matrix in R, then feed it into the PaIRKAT kernel machine score test so pathway topography (e.g., KEGG reaction edges) informs association testing.

## When to use

Use when a pathway is represented as a 0/1 symmetric adjacency matrix and you need the regularized normalized Laplacian (D + τI)^(-1/2) L (D + τI)^(-1/2), τ = 1, for kernel regression.

## When NOT to use

- The input matrix is not a symmetric 0/1 adjacency matrix of pathway connections (e.g., it is weighted, directed, or a feature/intensity table)
- The graph structure has not been examined first — the authors state it is important to examine graph structure before applying regularization
- The analysis does not embed the Laplacian into a kernel machine score test (e.g., plain network visualization or clustering)

## Inputs

- p×p symmetric 0/1 adjacency matrix A for a pathway (edges defined by KEGG reaction database connections)
- Regularization parameter τ (fixed to 1 in PaIRKAT simulations and analyses)

## Outputs

- Regularized normalized Laplacian matrix saved as normalized_laplacian.rds
- Eigen-decomposition check confirming symmetry and positive semidefiniteness (all eigenvalues ≥ 0)

## How to apply

Load the p×p adjacency matrix A in R and validate it is symmetric with 0/1 entries; edges come from pathway connections such as those in the KEGG reaction database. Build the degree matrix D as the diagonal matrix of row sums of A and form the unnormalized Laplacian L = D − A. Apply the I-based standardization to obtain the regularized normalized Laplacian (D + τI)^(-1/2) L (D + τI)^(-1/2) with the regularization parameter τ = 1, the value used throughout PaIRKAT simulations and data analyses. Verify the result is symmetric and positive semidefinite via eigen-decomposition in R (all eigenvalues ≥ 0) before using it in the kernel function of the score test (Satterthwaite chi-square approximation). Save the matrix as normalized_laplacian.rds for downstream PaIRKAT testing.

## Related tools

- **R** (Implementation environment for building D, forming L = D − A, applying the regularized normalization, eigen-verification, and saving the .rds output; all PaIRKAT simulations were done in R) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generation of random Barabasi-Albert pathway graphs whose Laplacians are computed for simulation-based validation of the regularization)

## Examples

```
A <- as.matrix(readRDS("adjacency.rds")); D <- diag(rowSums(A)); L <- D - A; tau <- 1; Lreg <- (solve(sqrtm(D + tau*diag(nrow(A))))) %*% L %*% (solve(sqrtm(D + tau*diag(nrow(A))))); saveRDS(Lreg, "normalized_laplacian.rds")
```

## Evaluation signals

- Adjacency matrix A is symmetric with 0/1 entries before any computation
- Degree matrix D equals diag(rowSums(A)); unnormalized L = D − A has row sums equal to 0
- Regularized normalized Laplacian is symmetric and all eigenvalues ≥ 0 (positive semidefinite) via eigen-decomposition in R
- τ = 1 was used, matching the value used in all PaIRKAT simulations and data analyses
- Output persists as normalized_laplacian.rds and can be loaded back as a p×p matrix

## Limitations

- The regularization parameter τ was fixed to 1 in all simulations with no sensitivity analysis of τ reported
- The unregularized normalized Laplacian hindered testing performance on disconnected graphs, so graph structure must be examined before regularization
- Sparsity present in many metabolomic pathways can greatly hinder performance

## Evidence

- [discussion] we saw meaningful improvements in power by including pathway information through a graph's regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
- [discussion] Edges in a pathway's graph were defined by connections within a pathway from the KEGG reaction database.: [evidence span withheld — anchor status: not_found]
