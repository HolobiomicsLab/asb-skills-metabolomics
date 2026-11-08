---
name: pathway-graph-topology-analysis
description: Use when testing phenotype–pathway associations from metabolomics data
  where metabolites can be mapped to a pathway graph (e.g., KEGG reaction edges) and
  the graph's topography should inform the kernel.
when_to_use_negative:
- There is no pathway/graph annotation linking metabolites — PaIRKAT requires a pathway
  adjacency matrix such as KEGG reaction-derived edges; without it the kernel cannot
  incorporate topography.
- The pathway graph structure has not been inspected first — the authors state it
  is important to examine graph structure before applying regularization, and sparse
  or poorly characterized graphs can hinder performance.
- The cohort is large (>1,000 observations) and the goal is to distinguish between
  competing methods — p-values from compared methods were indistinguishable at that
  sample size.
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the PaIRKAT functions, simulations, and example
    workflow
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Computes the normalized Laplacian of the pathway graph that is regularized
    and fed into the kernel
- name: KEGGREST
  role: Accesses KEGG pathway links used to map metabolites to pathway graphs
- name: impute (R package)
  role: k-nearest neighbors imputation (k = 10) of missing metabolite values before
    pathway kernel testing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:aa062c60831025a95648d830ed9fc585ec3d30bbbe65a0a0552beaa8f1809218
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
  - impute (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pathway-graph-topology-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

PaIRKAT incorporates pathway graph topography into semiparametric kernel machine regression by feeding the regularized (smoothed) normalized Laplacian of the pathway graph into the kernel function, yielding more powerful score tests of phenotype–pathway associations in metabolomics data.

## When to use

Use when testing phenotype–pathway associations from metabolomics data where metabolites can be mapped to a pathway graph (e.g., KEGG reaction edges) and the graph's topography should inform the kernel.

## When NOT to use

- There is no pathway/graph annotation linking metabolites — PaIRKAT requires a pathway adjacency matrix such as KEGG reaction-derived edges; without it the kernel cannot incorporate topography.
- The pathway graph structure has not been inspected first — the authors state it is important to examine graph structure before applying regularization, and sparse or poorly characterized graphs can hinder performance.
- The cohort is large (>1,000 observations) and the goal is to distinguish between competing methods — p-values from compared methods were indistinguishable at that sample size.

## Inputs

- Pathway graph adjacency matrix (edges from the KEGG reaction database)
- Metabolite intensity matrix (natural log transformed and standardized, kNN-imputed with k = 10, run-day median normalized, covariate-adjusted partial residuals)
- Covariate matrix X (e.g., white blood cell count, eosinophil/lymphocyte/monocyte/neutrophil percentages, hemoglobin)
- Phenotype vector Y (e.g., log FEV1/FVC ratio or percent emphysema)
- Regularization parameter τ (set to 1) and kernel bandwidth ρ (median pairwise distance)

## Outputs

- Regularized normalized Laplacian of the pathway graph
- Gaussian kernel matrix on smoothed pathway metabolite features
- Per-pathway kernel score test statistic and p-value for the variance component

## How to apply

Load the pathway graph adjacency matrix (edges defined from the KEGG reaction database) together with metabolite, covariate, and phenotype matrices. Compute the normalized Laplacian of the pathway graph with igraph in R and apply the regularization kernel with τ = 1 to obtain the regularized normalized Laplacian, which smooths metabolite features over the pathway structure and overcomes sparse connectivity and disjoint pathways. Project the pathway metabolite features through this regularized Laplacian and build a Gaussian (radial basis) kernel on the smoothed features, setting the bandwidth ρ to the median pairwise distance. Fit the semi-parametric kernel machine model g(Y) = Xβ + h(Z) + ε, estimating covariate effects β under the null and treating h(Z) as a random effect with covariance proportional to the kernel matrix, then compute the variance-component score test statistic and p-value per pathway as described in Section 2 of the paper. Examine the graph's structure before regularizing, since sparsity and disconnectedness strongly affect performance.

## Related tools

- **R** (Implementation language for the PaIRKAT functions, simulations, and example workflow) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Computes the normalized Laplacian of the pathway graph that is regularized and fed into the kernel)
- **KEGGREST** (Accesses KEGG pathway links used to map metabolites to pathway graphs)
- **impute (R package)** (k-nearest neighbors imputation (k = 10) of missing metabolite values before pathway kernel testing)

## Examples

```
g <- graph_from_adjacency_matrix(A, mode='undirected'); L_norm <- laplacian_matrix(g, normalized=TRUE)  # then regularize with tau = 1 before building the Gaussian kernel
```

## Evaluation signals

- Running the example workflow data with τ = 1 must produce a score test statistic and a p-value for each pathway.
- Per-pathway p-values should qualitatively show improved testing power versus ignoring pathway information or using the unregularized normalized Laplacian.
- On disconnected graphs, the unregularized normalized Laplacian should underperform PaIRKAT; if it does not, the regularization step was likely misapplied.
- The Gaussian kernel must be built on features projected through the regularized Laplacian with ρ set to the median pairwise distance, not on raw metabolite intensities.
- The semi-parametric fit should recover covariate effects β under the null with h(Z) covariance proportional to the kernel matrix before the score test is computed.

## Limitations

- The regularization parameter τ was fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported.
- Sparse connectivity in metabolomic pathways can greatly hinder performance; graph structure must be examined before applying regularization, so the method is not universally applicable without inspection.
- The filtered dataset did not contain every metabolite in the selected KEGG pathways, so some analyzed pathways had fewer than 10 metabolites.
- Real-data performance differences were demonstrated only on random subsets of 100–500 subjects; p-values were indistinguishable for datasets with over 1,000 observations.
- In the source document, mathematical equations (including the regularization kernel definition) appear as blank placeholders and derivations are deferred to an external reference.

## Evidence

- [discussion] Edges in a pathway's graph were defined by connections within a pathway from the KEGG reaction database.: [evidence span withheld — anchor status: not_found]
