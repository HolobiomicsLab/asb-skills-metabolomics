---
name: adjacency-matrix-symmetry-validation
description: Use when a metabolite/pathway graph is supplied as an adjacency matrix
  that must feed into Laplacian construction, since the regularized normalized Laplacian
  and downstream kernel are only valid for symmetric, undirected graphs.
when_to_use_negative:
- 'The graph is directed or weighted: the validation assumes undirected connectivity
  with 0/1 adjacency, so a non-symmetric or weighted matrix violates the Laplacian
  construction assumptions.'
- 'Graph structure has not been examined: PaIRKAT''s authors state it is important
  to examine the structure of a graph before applying a regularization step, so blind
  application to an uninspected graph is out of scope.'
- The goal is association testing itself rather than graph/Laplacian preprocessing
  — this skill covers only the adjacency-to-Laplacian validation stage, not the score
  test with Satterthwaite approximation.
tools:
- name: R
  role: Implementation language for loading A, checking symmetry and 0/1 entries,
    computing D and L, and running the eigen-decomposition PSD check
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used in PaIRKAT to generate random graphs (Barabasi-Albert model) for simulation-based
    validation of the Laplacian workflow
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:7ee6f8bcbc3b7b9e3a4437e729a6c79a1350e5941272181e63ec8154b437f8a0
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
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# adjacency-matrix-symmetry-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that a pathway's p×p adjacency matrix is symmetric with 0/1 entries before it is converted into the degree matrix, graph Laplacian, and the regularized normalized Laplacian (D + τI)^(-1/2) L (D + τI)^(-1/2) used in PaIRKAT's kernel machine score test.

## When to use

Use when a metabolite/pathway graph is supplied as an adjacency matrix that must feed into Laplacian construction, since the regularized normalized Laplacian and downstream kernel are only valid for symmetric, undirected graphs.

## When NOT to use

- The graph is directed or weighted: the validation assumes undirected connectivity with 0/1 adjacency, so a non-symmetric or weighted matrix violates the Laplacian construction assumptions.
- Graph structure has not been examined: PaIRKAT's authors state it is important to examine the structure of a graph before applying a regularization step, so blind application to an uninspected graph is out of scope.
- The goal is association testing itself rather than graph/Laplacian preprocessing — this skill covers only the adjacency-to-Laplacian validation stage, not the score test with Satterthwaite approximation.

## Inputs

- p×p adjacency matrix A of a pathway graph (0/1 entries, one row/column per metabolite node)
- regularization parameter τ (τ = 1 per PaIRKAT default)
- pathway membership/edge definitions, e.g. edges drawn from the KEGG reaction database

## Outputs

- Validated symmetric 0/1 adjacency matrix
- Degree matrix D (diagonal of A's row sums)
- Unnormalized graph Laplacian L = D − A
- Regularized normalized Laplacian saved as normalized_laplacian.rds, verified symmetric and positive semidefinite

## How to apply

Load the p×p adjacency matrix A in R and assert that it is symmetric and contains only 0/1 entries before any further processing. Compute the degree matrix D as the diagonal matrix of A's row sums, form the unnormalized Laplacian L = D − A, then apply the regularization (D + τI)^(-1/2) L (D + τI)^(-1/2) with τ = 1, the value PaIRKAT used for all simulations and data analyses. Verify the resulting matrix is symmetric and positive semidefinite via eigen-decomposition in R (all eigenvalues ≥ 0) before saving it (e.g., as normalized_laplacian.rds). Inspection of the graph's structure (e.g., connected vs. disconnected components) should precede regularization because PaIRKAT's authors stress graph structure must be examined before the regularization step.

## Related tools

- **R** (Implementation language for loading A, checking symmetry and 0/1 entries, computing D and L, and running the eigen-decomposition PSD check) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used in PaIRKAT to generate random graphs (Barabasi-Albert model) for simulation-based validation of the Laplacian workflow) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
A <- as.matrix(readRDS('pathway_adj.rds')); stopifnot(isSymmetric(A), all(A %in% c(0,1))); L <- diag(rowSums(A)) - A
```

## Evaluation signals

- isSymmetric(A) returns TRUE in R and all(A %in% c(0,1)) holds before Laplacian construction
- Degree matrix D equals diag(rowSums(A)); row sums of L = D − A are zero
- Eigen-decomposition of the regularized normalized Laplacian yields all eigenvalues ≥ 0 (positive semidefiniteness)
- The regularized normalized Laplacian matrix is itself symmetric
- Downstream PaIRKAT score tests (Satterthwaite chi-square approximation) run without errors using τ = 1

## Limitations

- Regularization parameter τ was fixed to 1 in all PaIRKAT simulations and data analyses with no sensitivity analysis reported.
- Graph sparsity in metabolomic pathways is a stated challenge that can greatly hinder performance; the unregularized normalized Laplacian hindered testing performance on disconnected graphs.
- Mathematical formulas (including the regularization kernel definition) appear as blank placeholders in the source document text, so exact equation details must be recovered from the cited regularization reference.

## Evidence

- [discussion] Regularization parameter τ fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported: "The regularization parameter τ is set to 1 for all simulations."
- [discussion] Graph structure must be examined before applying regularization: "It is important to examine the structure of a graph before applying a regularization step."
- [discussion] Regularized Laplacian improved testing power in PaIRKAT: [evidence span withheld — anchor status: not_found]
- [readme] PaIRKAT example workflow with source code is available in the repository: "Scripts for PaIRKAT functions with example work flow"
