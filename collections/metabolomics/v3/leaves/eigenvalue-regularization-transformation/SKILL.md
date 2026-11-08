---
name: eigenvalue-regularization-transformation
description: Use when a KEGG-derived pathway's normalized Laplacian must be regularized
  before being embedded in the kernel function of a semi-parametric kernel machine
  regression, particularly for sparse or disconnected pathway graphs.
when_to_use_negative:
- The graph structure (connectivity/sparse layout) has not been inspected first —
  PaIRKAT explicitly advises examining graph structure before applying a regularization
  step
- The input is raw metabolite intensity data or a phenotype table rather than a pathway
  graph's normalized Laplacian — this skill operates only on the graph spectral matrix
- Pathway information is being deliberately ignored as a comparator, in which case
  no regularized Laplacian should enter the kernel
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language in which the eigendecomposition, eigenvalue transformation r(λ)=1+τλ,
    matrix reconstruction, and saving of regularized_laplacian.rds are performed
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used in PaIRKAT simulations to generate random pathway graphs (Barabasi-Albert
    model) whose Laplacians are subsequently regularized
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:e1a3beacb14e1deecee5e8d2a129a43d63dcee1454f48ab8be0fd943040d7137
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# eigenvalue-regularization-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a linear graph-spectral regularization r(λ)=1+τλ (with τ=1) to the eigenvalues of a pathway's normalized Laplacian in R, reconstructing a 'smoothed' Laplacian that dampens noisy graph components while preserving biologically relevant topology for use in PaIRKAT kernel machine score tests.

## When to use

Use when a KEGG-derived pathway's normalized Laplacian must be regularized before being embedded in the kernel function of a semi-parametric kernel machine regression, particularly for sparse or disconnected pathway graphs.

## When NOT to use

- The graph structure (connectivity/sparse layout) has not been inspected first — PaIRKAT explicitly advises examining graph structure before applying a regularization step
- The input is raw metabolite intensity data or a phenotype table rather than a pathway graph's normalized Laplacian — this skill operates only on the graph spectral matrix
- Pathway information is being deliberately ignored as a comparator, in which case no regularized Laplacian should enter the kernel

## Inputs

- Normalized Laplacian matrix of a pathway graph (e.g., built from KEGG reaction-database edges)
- Regularization parameter τ (set to 1 per PaIRKAT convention)

## Outputs

- regularized_laplacian.rds — symmetric regularized normalized Laplacian with the same dimensionality as the input, to be included in the PaIRKAT kernel function

## How to apply

Load the normalized Laplacian matrix of the pathway graph (edges defined by connections in the KEGG reaction database) in R. Eigendecompose the normalized Laplacian, apply the linear regularization function r(λ)=1+τλ to its eigenvalues with τ=1 — the value fixed in all PaIRKAT simulations and real-data analyses — and reconstruct the regularized normalized Laplacian from the transformed spectrum. Save the result as regularized_laplacian.rds and verify it is symmetric and of the same dimensionality as the input. Examine the graph's structure (connectivity) before regularizing, since sparsity in metabolomic pathways can greatly hinder performance and the regularization step is what overcomes sparse connectivity and disjoint pathways. The regularized Laplacian then enters the kernel function of the semi-parametric kernel regression, with associations tested via a score test.

## Related tools

- **R** (Language in which the eigendecomposition, eigenvalue transformation r(λ)=1+τλ, matrix reconstruction, and saving of regularized_laplacian.rds are performed) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used in PaIRKAT simulations to generate random pathway graphs (Barabasi-Albert model) whose Laplacians are subsequently regularized) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
ed <- eigen(Lnorm); Lreg <- ed$vectors %*% diag(1 + 1*ed$values) %*% t(ed$vectors); stopifnot(isSymmetric(Lreg)); saveRDS(Lreg, 'regularized_laplacian.rds')
```

## Evaluation signals

- Output matrix is symmetric (e.g., isSymmetric() returns TRUE in R)
- Output matrix has the same dimensionality as the input normalized Laplacian
- Eigenvalues of the output equal 1 + τλ (τ=1) applied to the input Laplacian's spectrum
- Result is written to regularized_laplacian.rds and readable back into R without dimension or symmetry errors
- Downstream score-test p-values computed with the regularized Laplacian show improved power relative to the unregularized normalized Laplacian, especially on disconnected graphs

## Limitations

- τ is fixed to 1 in all simulations and analyses with no sensitivity analysis of τ reported
- Graph sparsity in metabolomic pathways is a stated challenge that can greatly hinder performance; the article notes it is important to examine graph structure before applying regularization
- Complete derivation details of the regularization functions are deferred to an external reference, and some equations (including the regularization kernel definition) appear as blank placeholders in the source text

## Evidence

- [abstract] By incorporating a regularized or 'smoothed' graph into a score test, PaIRKAT is capable of providing more powerful tests for associations: [evidence span withheld — anchor status: not_found]
- [discussion] The regularization parameter τ is set to 1 for all simulations.: "The regularization parameter τ is set to 1 for all simulations."
- [discussion] It is important to examine the structure of a graph before applying a regularization step.: "It is important to examine the structure of a graph before applying a regularization step."
- [discussion] we saw meaningful improvements in power by including pathway information through a graph's regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
- [discussion] Using the normalized Laplacian, , hindered testing performance compared to using PaIRKAT or ignoring the pathway information when a graph was disconnected.: "Using the normalized Laplacian, , hindered testing performance compared to using PaIRKAT or ignoring the pathway information when a graph was disconnected."
- [readme] Scripts for PaIRKAT functions with example work flow: "Scripts for PaIRKAT functions with example work flow"
