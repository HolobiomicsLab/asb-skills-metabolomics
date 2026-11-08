---
name: regularized-normalized-laplacian-standardization
description: Use when a pathway's connectivity is sparse or disjoint and you need
  a regularized graph representation for a kernel machine score test on metabolomics
  associations.
when_to_use_negative:
- The pathway graph is already dense and well-connected (the regularization targets
  sparse connectivity and disjoint pathways; in simulations the plain normalized Laplacian
  actually hindered performance on disconnected graphs compared with PaIRKAT or ignoring
  pathway information).
- You have not examined the graph structure yet — the paper explicitly says it is
  important to examine the structure of a graph before applying a regularization step.
- The study cohort is large enough that p-values are indistinguishable between methods
  (the paper notes differences were only demonstrable on random subsets of 100–500
  subjects from datasets with over 1,000 observations).
edam_operation: http://edamontology.org/operation_0281
edam_topics:
- http://edamontology.org/topic_3396
- http://edamontology.org/topic_0611
tools:
- name: R
  role: Implementation language in which the PaIRKAT functions, regularized Laplacian
    computation, and score tests are performed.
  repo: github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Used to generate random graphs (Barabasi-Albert model) for the simulation
    studies that evaluate the regularized-Laplacian kernel.
- name: KEGGREST
  role: Provides keggLink to access KEGG pathway/reaction connections that define
    the pathway graph whose Laplacian is regularized.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:9700a7a64a9d6f31fa49d7f6940af94040c1c423a806e93e3907d8b3a873efcc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - KEGGREST
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0281
  edam_topics:
  - http://edamontology.org/topic_3396
  - http://edamontology.org/topic_0611
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# regularized-normalized-laplacian-standardization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

PaIRKAT's graph-kernel regularization step: form the normalized graph Laplacian from a pathway's adjacency matrix and apply a regularization kernel that dampens noisy graph components while preserving biologically relevant structure for pathway association tests.

## When to use

Use when a pathway's connectivity is sparse or disjoint and you need a regularized graph representation for a kernel machine score test on metabolomics associations.

## When NOT to use

- The pathway graph is already dense and well-connected (the regularization targets sparse connectivity and disjoint pathways; in simulations the plain normalized Laplacian actually hindered performance on disconnected graphs compared with PaIRKAT or ignoring pathway information).
- You have not examined the graph structure yet — the paper explicitly says it is important to examine the structure of a graph before applying a regularization step.
- The study cohort is large enough that p-values are indistinguishable between methods (the paper notes differences were only demonstrable on random subsets of 100–500 subjects from datasets with over 1,000 observations).

## Inputs

- p×p adjacency matrix (or igraph/KEGG graph object) representing a KEGG pathway whose edges come from the KEGG reaction database
- KEGG metabolite-to-pathway links retrieved via keggLink from the KEGGREST package
- Regularization parameter τ (set to 1 in all analyses reported)

## Outputs

- Regularized normalized Laplacian matrix (p×p) ready to be plugged into the kernel function of the kernel machine score test
- Pathway-level p-value from the score test using the Satterthwaite approximation of a chi-square distribution

## How to apply

Start from the p×p adjacency matrix of a KEGG pathway, where edges are defined by connections from the KEGG reaction database (metabolites as nodes, reactions as edges). Compute the normalized Laplacian and then apply a regularization kernel to obtain a regularized normalized Laplacian, as described in PaIRKAT Section 2; the regularization dampens noisy aspects of the graph while preserving biologically relevant features, overcoming sparse connectivity and disjoint pathways. Set the regularization parameter τ = 1 for all simulations and data analyses (no sensitivity analysis is reported). The resulting regularized Laplacian is then incorporated into the kernel function of a score test evaluated via a Satterthwaite approximation of a chi-square distribution. Before applying the regularization step, examine the structure of the graph — the article states performance is greatly hindered by the sparsity present in many metabolomic pathways and that the unregularized normalized Laplacian performed worse than PaIRKAT or ignoring pathway information when a graph was disconnected.

## Related tools

- **R** (Implementation language in which the PaIRKAT functions, regularized Laplacian computation, and score tests are performed.) — github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Used to generate random graphs (Barabasi-Albert model) for the simulation studies that evaluate the regularized-Laplacian kernel.)
- **KEGGREST** (Provides keggLink to access KEGG pathway/reaction connections that define the pathway graph whose Laplacian is regularized.)

## Examples

```
Rscript -e "source('PaIRKAT.R'); # compute regularized normalized Laplacian per PaIRKAT workflow with tau=1"
```

## Evaluation signals

- The regularized normalized Laplacian should be a symmetric matrix with dimensions equal to the pathway's metabolite count (p×p).
- Score-test p-values computed with τ = 1 should show improved power relative to using the unregularized normalized Laplacian or ignoring pathway information, especially when the graph is disconnected or sparse.
- In simulations, the method should be robust to incorrect and incomplete pathway knowledge (PaIRKAT's abstract claims robustness to these conditions).
- Compare against the plain normalized Laplacian baseline: on disconnected graphs the unregularized version hindered testing performance, so PaIRKAT's regularized version should not degrade results.
- Reproduce the example workflow from github.com/CharlieCarpenter/PaIRKAT to verify the regularized Laplacian is correctly integrated into the kernel function before the Satterthwaite score test.

## Limitations

- The regularization parameter τ is fixed at 1 in all simulations and data analyses, with no sensitivity analysis reported.
- Regularization functions are applied after graph-structure inspection; the paper warns that sparsity in metabolomic pathways can greatly hinder performance.
- Methodological differences were only demonstrable on downsampled subsets (100–500 subjects), not on the full cohort of over 1,000 observations.
- Derivation details of the regularization functions are deferred to an external reference rather than provided in the article.

## Evidence

- [discussion] Edges from KEGG reaction database: [evidence span withheld — anchor status: not_found]
