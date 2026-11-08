---
name: pathway-graph-kernel-smoothing
description: Use when metabolites map onto a pathway graph (e.g., KEGG edges from
  the KEGG reaction database) that is sparse or disconnected, and you want to incorporate
  that graph topology into a kernel machine association test via a regularized normalized
  Laplacian.
when_to_use_negative:
- When the pathway graph is well-connected and dense — regularization targets sparsity
  and disconnected components, and the article notes it is important to examine graph
  structure before applying a regularization step
- When you have no metabolite-to-pathway mapping (e.g., metabolites lack KEGG/HMDB
  identifiers), since the Laplacian cannot be constructed
- When the goal is standard per-metabolite differential analysis rather than pathway-level
  kernel association testing
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation language for eigendecomposition of the normalized Laplacian,
    spectrum-based regularization, kernel construction, and score testing
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generation of random pathway graphs (Barabasi-Albert model) used to evaluate
    the regularization in simulation studies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:5d9edcc0b4cb313d5931a0d84ec181b4fb2e9f15d7e03dda519526fc8376817a
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
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pathway-graph-kernel-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Graph kernel regularization in the PaIRKAT sense: eigendecompose a pathway's normalized Laplacian, apply the linear regularization r(λ)=1+τλ with τ=1 to its eigenvalues, and pass the reconstructed regularized Laplacian into the kernel function of a semi-parametric kernel machine score test, dampening noisy graph components while preserving biologically relevant structure.

## When to use

Use when metabolites map onto a pathway graph (e.g., KEGG edges from the KEGG reaction database) that is sparse or disconnected, and you want to incorporate that graph topology into a kernel machine association test via a regularized normalized Laplacian.

## When NOT to use

- When the pathway graph is well-connected and dense — regularization targets sparsity and disconnected components, and the article notes it is important to examine graph structure before applying a regularization step
- When you have no metabolite-to-pathway mapping (e.g., metabolites lack KEGG/HMDB identifiers), since the Laplacian cannot be constructed
- When the goal is standard per-metabolite differential analysis rather than pathway-level kernel association testing

## Inputs

- Normalized Laplacian matrix of a pathway graph (e.g., built from KEGG pathway metabolites with edges from the KEGG reaction database)
- Regularization parameter τ (set to 1 in PaIRKAT)
- Pathway-mapped metabolomics data (run-day median normalized, KNN-imputed with k = 10, log-transformed and standardized) with covariates for the score test

## Outputs

- Regularized normalized Laplacian matrix (saved as regularized_laplacian.rds in the reference workflow)
- Score-test p-values for pathway-phenotype association using the regularized kernel

## How to apply

Load the pathway's normalized Laplacian matrix in R and eigendecompose it (the normalized Laplacian is symmetric PSD, so a symmetric eigendecomposition applies). Apply the linear regularization function r(λ)=1+τλ to the eigenvalues with τ=1, the value PaIRKAT fixed for all simulations and real-data analyses, then reconstruct the regularized normalized Laplacian from the transformed spectrum. Verify the reconstructed matrix is symmetric with the same dimensionality as the input. Feed the regularized Laplacian into the kernel function of the semi-parametric kernel regression and test pathway-phenotype association with a score test (P-values were calculated with τ = 1 in the COPDGene analyses of FEV1/FVC ratio and percent emphysema). Rationale: the regularization dampens noisy components of sparse/disjoint pathway graphs — using the unregularized normalized Laplacian hindered testing performance on disconnected graphs.

## Related tools

- **R** (Implementation language for eigendecomposition of the normalized Laplacian, spectrum-based regularization, kernel construction, and score testing) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generation of random pathway graphs (Barabasi-Albert model) used to evaluate the regularization in simulation studies)

## Examples

```
L <- readRDS('laplacian.rds'); e <- eigen(L, symmetric=TRUE); Lreg <- e$vectors %*% diag(1 + 1*e$values) %*% t(e$vectors); saveRDS(Lreg, 'regularized_laplacian.rds')
```

## Evaluation signals

- Output matrix is symmetric and has the same dimensionality as the input normalized Laplacian
- Eigenvalues transformed as r(λ)=1+λ with τ=1, so all regularized eigenvalues are ≥ 1 and shifted upward relative to the Laplacian spectrum
- Spectral reconstruction error: regenerating the matrix from transformed eigenpairs reproduces the regularized Laplacian to numerical tolerance
- On disconnected pathway graphs, the regularized-kernel score test should outperform (lower p-values / higher power than) the unregularized normalized Laplacian, per the simulation comparisons
- For large cohorts (>1,000 observations) p-values across methods converge; power differences are expected only on small subsets (100–500 subjects)

## Limitations

- The regularization parameter τ was fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported
- Graph sparsity in metabolomic pathways can greatly hinder performance, and graph structure should be examined before applying regularization — the method is not universally applicable without inspection
- Power improvements over competing methods were demonstrated on downsampled subsets of 100–500 subjects, not the full COPDGene cohort where p-values were indistinguishable

## Evidence

- [discussion] we saw meaningful improvements in power by including pathway information through a graph's regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
