---
name: r-matrix-linear-algebra
description: Use when a PaIRKAT-style kernel machine association test needs a pathway's
  normalized Laplacian regularized via its eigenspectrum (τ=1) to dampen noisy graph
  components before building the kernel matrix, e.g. on KEGG metabolite pathway graphs.
when_to_use_negative:
- The pathway graph structure has not been examined first — the authors note 'It is
  important to examine the structure of a graph before applying a regularization step,'
  so regularization is not universally applicable without inspection
- The pathway graph is disconnected and you intend to use the raw, unregularized normalized
  Laplacian instead — the paper shows this 'hindered testing performance compared
  to using PaIRKAT or ignoring the pathway information when a graph was disconnected'
- You need a different regularization function than the linear r(λ)=1+τλ; PaIRKAT
  defers derivation details of alternative regularizations to an external reference
  and uses τ=1 throughout
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Language for the eigendecomposition of the normalized Laplacian, eigenvalue
    transformation r(λ)=1+τλ with τ=1, matrix reconstruction, and saving regularized_laplacian.rds
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used to generate pathway/random graphs (e.g. Barabasi-Albert model) whose
    normalized Laplacians undergo this regularization in simulations
- name: KEGGREST
  role: Accesses KEGG pathway graphs (keggLink) whose normalized Laplacians are the
    input matrices for this regularization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:13f3ea2caf2f1591b6c8edaca44e32eca5894a6de57d8e49b0e22f8db6336ab0
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
  - KEGGREST
  - R
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# r-matrix-linear-algebra

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

R-based spectral matrix operation from PaIRKAT: eigendecompose a pathway's normalized Laplacian, apply the linear regularization r(λ)=1+τλ with τ=1 to the eigenvalues, and reconstruct the regularized Laplacian used inside the kernel function of the semi-parametric kernel regression score test.

## When to use

Use when a PaIRKAT-style kernel machine association test needs a pathway's normalized Laplacian regularized via its eigenspectrum (τ=1) to dampen noisy graph components before building the kernel matrix, e.g. on KEGG metabolite pathway graphs.

## When NOT to use

- The pathway graph structure has not been examined first — the authors note 'It is important to examine the structure of a graph before applying a regularization step,' so regularization is not universally applicable without inspection
- The pathway graph is disconnected and you intend to use the raw, unregularized normalized Laplacian instead — the paper shows this 'hindered testing performance compared to using PaIRKAT or ignoring the pathway information when a graph was disconnected'
- You need a different regularization function than the linear r(λ)=1+τλ; PaIRKAT defers derivation details of alternative regularizations to an external reference and uses τ=1 throughout

## Inputs

- Normalized Laplacian matrix of a pathway graph (e.g. metabolite pathway edges defined by connections from the KEGG reaction database)
- Regularization parameter τ (fixed at 1 per PaIRKAT convention)

## Outputs

- regularized_laplacian.rds — symmetric regularized normalized Laplacian with the same dimensionality as the input
- Regularized graph subsequently embedded in the PaIRKAT kernel matrix for score testing

## How to apply

Load the pathway's normalized Laplacian matrix in R, then eigendecompose it (since the normalized Laplacian is symmetric, use a symmetric eigen solver). Transform each eigenvalue λ by the linear function r(λ)=1+τλ with τ=1 — the value used for all simulations and the COPDGene score-test analyses — leaving eigenvectors unchanged, and reconstruct the regularized normalized Laplacian from the transformed spectrum. This 'smooths' the graph by dampening high-eigenvalue (noisy, poorly connected) components while preserving biologically relevant structure. Save the result as regularized_laplacian.rds for downstream inclusion in the PaIRKAT kernel. Before regularizing, examine the graph's structure: the paper stresses this inspection step because sparse connectivity and disconnected graphs are a known challenge. The regularized Laplacian then enters the kernel function of the semi-parametric kernel regression, and associations are tested via a score test.

## Related tools

- **R** (Language for the eigendecomposition of the normalized Laplacian, eigenvalue transformation r(λ)=1+τλ with τ=1, matrix reconstruction, and saving regularized_laplacian.rds) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used to generate pathway/random graphs (e.g. Barabasi-Albert model) whose normalized Laplacians undergo this regularization in simulations)
- **KEGGREST** (Accesses KEGG pathway graphs (keggLink) whose normalized Laplacians are the input matrices for this regularization)

## Examples

```
eig <- eigen(L, symmetric=TRUE); L_reg <- with(eig, vectors %*% diag(1 + values) %*% t(vectors)); saveRDS(L_reg, "regularized_laplacian.rds")
```

## Evaluation signals

- Output matrix is symmetric (verify with isSymmetric() in R), as required by the workflow
- Output has the same dimensionality as the input normalized Laplacian
- Eigenvectors are unchanged from the input decomposition; every eigenvalue λ is mapped to 1+τλ with τ=1 (i.e. values in the [1, 1+2] range for a normalized Laplacian spectrum), so noisy high-λ components are relatively dampened
- Result is written as regularized_laplacian.rds and matches the PaIRKAT example workflow at github.com/CharlieCarpenter/PaIRKAT
- Downstream score-test p-values with τ=1 show power improvement over the unregularized normalized Laplacian, particularly on disconnected graphs and in downsampled subsets (100–500 subjects)

## Limitations

- The regularization parameter τ is fixed to 1 in all simulations and data analyses, and no sensitivity analysis of τ is reported
- Graph sparsity in metabolomic pathways can greatly hinder performance; graph structure must be examined before applying regularization
- Mathematical derivation details of the regularization functions are deferred to an external reference rather than provided in the paper
- Performance differences versus alternatives were only demonstrated on random subsets of 100–500 subjects; on datasets with over 1,000 observations, P-values from compared methods were indistinguishable

## Evidence

- [discussion] we saw meaningful improvements in power by including pathway information through a graph's regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
