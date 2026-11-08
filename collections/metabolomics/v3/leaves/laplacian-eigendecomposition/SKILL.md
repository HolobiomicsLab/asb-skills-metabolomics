---
name: laplacian-eigendecomposition
description: Use when a pathway's normalized Laplacian must be smoothed before kernel
  construction — e.g., sparse or disconnected metabolite pathway graphs (KEGG reaction-derived
  edges) feeding a PaIRKAT semi-parametric kernel machine score test.
when_to_use_negative:
- The graph structure has not been examined first — the article stresses it is important
  to examine the structure of a graph before applying a regularization step, so regularization
  is not universally applicable without inspection
- The input is a raw metabolite intensity table or cohort phenotype file rather than
  a pathway graph Laplacian — preprocessing (run-day median scaling, kNN imputation
  with k=10, ln-transform) and regression on covariates come first
- Working with datasets of over 1,000 observations, where P-values from methods with
  and without pathway regularization were indistinguishable and the regularization
  offers no demonstrated benefit
tools:
- name: R
  role: Language in which the normalized Laplacian is eigendecomposed and the regularized
    Laplacian is reconstructed and saved as .rds
  repo: github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used to generate the pathway graphs (Barabasi-Albert random graphs) whose
    Laplacians are regularized in the simulation studies
  repo: github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:4495b063fd9d4e7b64c8f2ef900bede079f0de47c528d4c9c2690b781ace3bb7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - R
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# laplacian-eigendecomposition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Spectrally regularizes a pathway graph's normalized Laplacian by eigendecomposing it in R and applying the linear function r(λ)=1+τλ (τ=1) to the eigenvalues, producing a regularized 'smoothed' Laplacian that dampens noisy graph components before its use in the PaIRKAT kernel score test.

## When to use

Use when a pathway's normalized Laplacian must be smoothed before kernel construction — e.g., sparse or disconnected metabolite pathway graphs (KEGG reaction-derived edges) feeding a PaIRKAT semi-parametric kernel machine score test.

## When NOT to use

- The graph structure has not been examined first — the article stresses it is important to examine the structure of a graph before applying a regularization step, so regularization is not universally applicable without inspection
- The input is a raw metabolite intensity table or cohort phenotype file rather than a pathway graph Laplacian — preprocessing (run-day median scaling, kNN imputation with k=10, ln-transform) and regression on covariates come first
- Working with datasets of over 1,000 observations, where P-values from methods with and without pathway regularization were indistinguishable and the regularization offers no demonstrated benefit

## Inputs

- Normalized Laplacian matrix of a pathway graph (e.g., metabolite pathway with edges defined by KEGG reaction database connections)
- Regularization parameter τ (set to 1 per the article)

## Outputs

- regularized_laplacian.rds — the regularized normalized Laplacian, symmetric and of the same dimensionality as the input, for use in the PaIRKAT kernel score test

## How to apply

Load the normalized Laplacian matrix of the pathway graph in R (graphs whose edges come from the KEGG reaction database) and eigendecompose it. Apply the linear regularization r(λ)=1+τλ to the eigenvalues with τ=1 — the value used for all simulations and real-data analyses in the paper — keeping eigenvectors fixed, and reconstruct the regularized normalized Laplacian from the transformed spectrum. Save the result as regularized_laplacian.rds. This 'smoothed' graph is then included through the kernel function of the semi-parametric kernel regression, with association tested by a score test; the rationale is to dampen noisy graph aspects while preserving biologically relevant features, which especially helps sparse or disconnected pathways where the unregularized normalized Laplacian hindered testing performance.

## Related tools

- **R** (Language in which the normalized Laplacian is eigendecomposed and the regularized Laplacian is reconstructed and saved as .rds) — github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used to generate the pathway graphs (Barabasi-Albert random graphs) whose Laplacians are regularized in the simulation studies) — github.com/CharlieCarpenter/PaIRKAT

## Examples

```
e <- eigen(L, symmetric=TRUE); L.reg <- e$vectors %*% diag(1 + 1*e$values) %*% t(e$vectors); saveRDS(L.reg, "regularized_laplacian.rds")
```

## Evaluation signals

- Output matrix regularized_laplacian.rds is symmetric
- Output matrix has the same dimensionality as the input normalized Laplacian
- Spectrum check: each input eigenvalue λ maps to 1+τλ with τ=1 (eigenvalues 0→1, 2→3); eigenvectors are unchanged
- Downstream check: regularized Laplacian enters the kernel function and score test P-values are computed with τ=1, matching the paper's setting
- On disconnected pathway graphs, performance should not degrade as it does when using the unregularized normalized Laplacian

## Limitations

- The regularization parameter τ is fixed at 1 in all simulations and data analyses, and no sensitivity analysis of τ is reported
- Sparse connectivity in many metabolomic pathways is a stated challenge that can greatly hinder performance, and graph structure must be examined before applying regularization
- Methodological performance differences were demonstrated only on random subsets of 100–500 subjects; on datasets with over 1,000 observations, P-values across compared methods were indistinguishable
- The article defers derivation details of the regularization functions to an external reference rather than providing them

## Evidence
