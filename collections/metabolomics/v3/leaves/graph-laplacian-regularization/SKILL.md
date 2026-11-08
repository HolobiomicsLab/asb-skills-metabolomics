---
name: graph-laplacian-regularization
description: Use when testing phenotype–pathway associations on metabolite features
  mapped onto a sparse or disconnected pathway graph (e.g. KEGG pathways), where ignoring
  topography or using the unregularized normalized Laplacian loses testing power.
when_to_use_negative:
- Full-cohort analyses with over 1,000 observations, where the paper reports p-values
  from the compared methods were indistinguishable — subsample to 100–500 to expose
  method differences
- Pathways that are well-connected rather than sparse or disjoint, where regularization
  may add little over the plain normalized Laplacian
- When graph structure has not been examined first — the authors caution it is important
  to inspect the graph before applying a regularization step
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the Laplacian computation, regularization kernel,
    and kernel machine score test
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Builds the pathway graph and computes the normalized Laplacian; also used
    to generate random Barabasi-Albert graphs in simulations
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: KEGGREST
  role: Fetches KEGG human pathway membership (keggLink) and reaction-database edges
    used to define pathway graphs
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:dcffc9bfbcfda7890da0d7e63e2b4a6f4f5f78d293f806cd03078b5941878134
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# graph-laplacian-regularization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Smooths pathway metabolite features over a biological network by applying a regularization kernel to the graph's normalized Laplacian (τ = 1), so that pathway topography enters a kernel machine score test directly instead of being ignored or added as noise.

## When to use

Use when testing phenotype–pathway associations on metabolite features mapped onto a sparse or disconnected pathway graph (e.g. KEGG pathways), where ignoring topography or using the unregularized normalized Laplacian loses testing power.

## When NOT to use

- Full-cohort analyses with over 1,000 observations, where the paper reports p-values from the compared methods were indistinguishable — subsample to 100–500 to expose method differences
- Pathways that are well-connected rather than sparse or disjoint, where regularization may add little over the plain normalized Laplacian
- When graph structure has not been examined first — the authors caution it is important to inspect the graph before applying a regularization step

## Inputs

- Pathway graph adjacency/edge list (e.g. KEGG pathway edges from the KEGG reaction database) loaded in igraph
- Metabolite feature matrix restricted to metabolites in the pathway
- Covariate matrix X and phenotype vector Y (e.g. log FEV1/FVC ratio or percent emphysema from COPDGene)
- Regularization parameter τ (set to 1)

## Outputs

- Regularized normalized Laplacian matrix used as the pathway kernel
- Kernel machine score test statistic and p-value per pathway
- Per-pathway power comparison against the unregularized Laplacian and topography-ignored baselines

## How to apply

Build the pathway graph in R with igraph, defining edges by connections within the pathway from the KEGG reaction database, then compute the normalized Laplacian and apply the regularization kernel with τ = 1 to obtain the regularized (smoothed) normalized Laplacian. Project the pathway metabolite features through the regularized Laplacian, which dampens noisy components while preserving biologically relevant signals, and construct a Gaussian (radial basis) kernel on the smoothed features with bandwidth ρ set to the median pairwise distance. In the semi-parametric model g(Y)=Xβ+h(Z)+ϵ, treat h(Z) as a random effect with covariance proportional to this kernel and compute the kernel score test statistic and p-value as described in Section 2. Before regularizing, examine the graph structure, since sparse, disconnected graphs are exactly where the unregularized normalized Laplacian hinders performance and regularization helps.

## Related tools

- **R** (Implementation language for the Laplacian computation, regularization kernel, and kernel machine score test) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Builds the pathway graph and computes the normalized Laplacian; also used to generate random Barabasi-Albert graphs in simulations) — https://github.com/CharlieCarpenter/PaIRKAT
- **KEGGREST** (Fetches KEGG human pathway membership (keggLink) and reaction-database edges used to define pathway graphs)

## Examples

```
g <- graph_from_data_frame(kegg_edges); L <- laplacian_matrix(g, normalization = "symmetric")  # then apply regularization kernel with tau = 1 before building the Gaussian kernel
```

## Evaluation signals

- With τ = 1 on the example workflow data, a score test statistic and p-value are produced per pathway (the paper sets no numeric p-value threshold; acceptance is qualitative agreement with improved testing power)
- On the 28 KEGG pathways against log FEV1/FVC, the regularized-Laplacian test shows improved power over ignoring topography or the plain normalized Laplacian for the majority of pathways (reported 18/28 FEV1/FVC, 17/28 emphysema)
- On disconnected pathway graphs, the regularized kernel does not show the degraded performance attributed to the unregularized normalized Laplacian
- Averaged p-values over 100 random subsets of sizes 100–500 separate the methods; verify the comparison table has 28 pathway rows for both phenotypes
- The smoothed feature projection preserves signal rather than injecting noise: regularized-Laplacian power should meet or exceed the unregularized Laplacian baseline

## Limitations

- The regularization parameter τ was fixed at 1 in all simulations and data analyses, with no sensitivity analysis of τ reported
- Mathematical details of the regularization kernel appear as blank placeholders in the source text; derivation details are deferred to an external reference
- Sparse connectivity in metabolomic pathways can greatly hinder performance, so graph structure must be examined before applying regularization
- The filtered dataset did not contain every metabolite in the selected KEGG pathways, so some analyzed pathways had fewer than 10 metabolites
- Real-data performance differences were only demonstrated on downsampled subsets (100–500 subjects), not the full cohort

## Evidence

- [discussion] Graph edges from KEGG reactions: [evidence span withheld — anchor status: not_found]
