---
name: barabasi-albert-network-generation
description: Use when simulating pathway graph structures (e.g., metabolite networks)
  for kernel machine test power or type I error studies, or when evaluating robustness
  to incorrect/incomplete pathway edges via controlled edge permutation.
when_to_use_negative:
- When real, biologically derived pathway graphs are available (e.g., KEGG reaction
  database edges for the 28 KEGG pathways) — use those instead of random Barabasi-Albert
  graphs
- When the analysis does not involve graph-structured pathway information (e.g., single-metabolite
  tests or unregularized kernel tests)
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0151
tools:
- name: R
  role: Implementation environment for the simulation framework and PaIRKAT score
    test
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates Barabasi-Albert model random pathway graphs used in the simulations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:06e25d419a88cf6d0bd5154627de701546c8c1aefceb3cfc8721d47b640c5db2
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0151
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# barabasi-albert-network-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate Barabasi-Albert model random pathway graphs with the R igraph package to serve as simulated metabolite pathway structures, as used in PaIRKAT's simulation studies for pathway sizes p = 15, 30, and 45 metabolites.

## When to use

Use when simulating pathway graph structures (e.g., metabolite networks) for kernel machine test power or type I error studies, or when evaluating robustness to incorrect/incomplete pathway edges via controlled edge permutation.

## When NOT to use

- When real, biologically derived pathway graphs are available (e.g., KEGG reaction database edges for the 28 KEGG pathways) — use those instead of random Barabasi-Albert graphs
- When the analysis does not involve graph-structured pathway information (e.g., single-metabolite tests or unregularized kernel tests)

## Inputs

- Pathway size p (number of metabolites; e.g., 15, 30, or 45)
- Fraction of edges to be made incorrect (e.g., 10%, 40%, 70% mismatch)
- Simulated pathway-structured outcome/omics data (from the PaIRKAT simulation framework)

## Outputs

- Barabasi-Albert random pathway graphs (igraph objects) for each pathway size
- Edge-permuted pathway graphs at each mismatch level with edge count preserved
- Empirical power tables, e.g., power_mismatch_by_size.csv with rows per (pathway size, mismatch level)

## How to apply

Use the igraph package in R to generate Barabasi-Albert model random graphs at each target pathway size (p = 15, 30, 45 metabolites), mirroring the PaIRKAT simulation design. To study sensitivity to pathway mismatch, permute a fixed fraction of edges (e.g., 10%, 40%, or 70% incorrect edges) while keeping the number of edges constant, so only connection correctness varies. These graphs supply the pathway structure for computing the regularized normalized Laplacian used in the PaIRKAT kernel machine score test (τ = 1). Aggregate results across simulation replicates into empirical power or type I error estimates per (pathway size, mismatch level) combination.

## Related tools

- **R** (Implementation environment for the simulation framework and PaIRKAT score test) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates Barabasi-Albert model random pathway graphs used in the simulations)

## Examples

```
library(igraph); g <- barabasi.game(n = 45, directed = FALSE, m = 1) # pathway-size graph for PaIRKAT simulation
```

## Evaluation signals

- Generated graphs have the requested number of metabolites (p = 15, 30, 45) and valid Barabasi-Albert topology in igraph
- Edge permutation preserves the total number of edges; only connection correctness changes
- Empirical power values all fall within [0, 1] for every (pathway size, mismatch level) cell
- Power decreases monotonically as the proportion of incorrect edges grows from 10% to 70%, with perfect structure giving the most power
- At ~10% mismatch in small pathways, power is comparable to the perfect-structure setting, matching the paper's qualitative claims

## Limitations

- Graph sparsity in metabolomic pathways can greatly hinder performance; examine graph structure before applying regularization
- The regularization parameter τ was fixed at 1 in all simulations with no sensitivity analysis reported
- Findings are based on simulated Barabasi-Albert graphs, not necessarily realistic metabolite pathway topologies
- The paper does not report software/tool versions for R or igraph

## Evidence

- [discussion] Random graphs were generated using the igraph(42) package according to the Barabasi-Albert model(43): "Random graphs were generated using the igraph(42) package according to the Barabasi-Albert model(43)"
- [abstract] Our simulation studies illustrate the robustness of this method to incorrect and incomplete pathway knowledge: "Our simulation studies illustrate the robustness of this method to incorrect and incomplete pathway knowledge"
- [discussion] All simulations were accomplished using R(41).: "All simulations were accomplished using R(41)."
- [readme] # SimulationFunctions
TypeI and Power simulation scripts: "# SimulationFunctions
TypeI and Power simulation scripts"
