---
name: barabasi-albert-random-graph-generation
description: Use when simulation studies need synthetic pathway graphs emulating KEGG
  metabolite network topology — e.g., generating Barabási-Albert graphs of size 15,
  30, or 45 nodes to evaluate kernel association tests like PaIRKAT.
when_to_use_negative:
- The analysis requires real pathway connectivity — build graphs from the KEGG reaction
  database via the KEGGREST keggLink function instead of synthetic BA graphs.
- An observed/empirical metabolite network already exists — then the need is graph-structure
  inspection and regularization, not graph generation.
- The downstream task is not a graph-based kernel association simulation — these BA
  graphs specifically emulate metabolomic pathway networks of 15-45 metabolites for
  PaIRKAT score-test evaluation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Environment in which all PaIRKAT simulations and random pathway graph generation
    are executed
- name: igraph
  role: Generates random pathway graphs according to the Barabasi-Albert model for
    the simulation scenarios
- name: PaIRKAT SimulationFunctions
  role: Hosts the TypeI and Power simulation scripts that consume the generated BA
    pathway graphs in Monte Carlo replicates
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:bbdf7fab0abd4042f108d5bffb1dcc6cd8a4adccf65c0fdeadacdb3531cdc352
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
  - PaIRKAT SimulationFunctions
  - R
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# barabasi-albert-random-graph-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates undirected scale-free random pathway graphs in R using igraph's Barabási-Albert model, emulating metabolomic pathway topology (e.g., KEGG-like hub structures) for PaIRKAT's Type I error and power simulation studies.

## When to use

Use when simulation studies need synthetic pathway graphs emulating KEGG metabolite network topology — e.g., generating Barabási-Albert graphs of size 15, 30, or 45 nodes to evaluate kernel association tests like PaIRKAT.

## When NOT to use

- The analysis requires real pathway connectivity — build graphs from the KEGG reaction database via the KEGGREST keggLink function instead of synthetic BA graphs.
- An observed/empirical metabolite network already exists — then the need is graph-structure inspection and regularization, not graph generation.
- The downstream task is not a graph-based kernel association simulation — these BA graphs specifically emulate metabolomic pathway networks of 15-45 metabolites for PaIRKAT score-test evaluation.

## Inputs

- Simulation design parameters: pathway sizes p = 15, 30, 45; 4 graph-structure scenarios x 3 pathway-knowledge levels; sample size n = 160
- R environment with the igraph package installed
- PaIRKAT SimulationFunctions scripts (TypeI and Power) from github.com/CharlieCarpenter/PaIRKAT

## Outputs

- Undirected Barabási-Albert igraph graph objects / adjacency matrices emulating metabolomic pathway graphs (scale-free hub structure, connected or disconnected per scenario)
- Graph inputs consumed by 1000-replicate Monte Carlo simulations yielding type1_error_rates.csv (Tables 1-4, S1-S4) and simulation_power_results.csv (Figures 1-2 power curves)

## How to apply

In R, load the igraph package and generate undirected random graphs via the Barabási-Albert preferential-attachment model (sample_pa), setting node counts to the PaIRKAT pathway-size scenarios p = 15, 30, and 45, and varying attachment parameters to realize the 4 graph-structure scenarios crossed with 3 pathway-knowledge levels. Record whether each generated graph is connected, because the paper shows the unregularized normalized Laplacian underperforms on disconnected graphs while PaIRKAT's regularized kernel is robust to incomplete pathway knowledge. Feed the graphs, together with multivariate-normal metabolite draws (Danaher joint graphical-model precision matrix, n = 160), into the SimulationFunctions TypeI and Power scripts from github.com/CharlieCarpenter/PaIRKAT, running 1000 Monte Carlo replicates per scenario at alpha = 0.05 for type I error and sweeping SNR for power curves with the score-test regularization parameter tau = 1. The rationale is that BA graphs reproduce the sparse, hub-dominated topology of real KEGG pathway graphs (edges defined by the KEGG reaction database), letting simulations stress-test power gains from graph regularization.

## Related tools

- **R** (Environment in which all PaIRKAT simulations and random pathway graph generation are executed)
- **igraph** (Generates random pathway graphs according to the Barabasi-Albert model for the simulation scenarios)
- **PaIRKAT SimulationFunctions** (Hosts the TypeI and Power simulation scripts that consume the generated BA pathway graphs in Monte Carlo replicates) — https://github.com/CharlieCarpenter/PaIRKAT

## Examples

```
library(igraph); g <- sample_pa(n = 30, m = 1, directed = FALSE)
```

## Evaluation signals

- Each generated graph's node count matches a pathway-size scenario (p = 15, 30, or 45) and its degree distribution is scale-free with hub nodes characteristic of the Barabasi-Albert model.
- Connectivity status is recorded for every generated graph; disconnected graphs must be present across the 4 structure scenarios since they drive the Laplacian-vs-PaIRKAT performance contrast.
- Feeding generated graphs into the TypeI simulation with 1000 replicates at alpha = 0.05 yields empirical type I error rates of approximately 0.05 per scenario (Tables 1-4 / S1-S4 pattern).
- Power simulations reproduce Figure 1's 3 knowledge x 4 structure panels and Figure 2's SNR curves, with PaIRKAT reaching ~80% power at SNR ~ 0.23 versus ~ 0.60 when ignoring network information.
- Downstream score tests use tau = 1, the paper's fixed regularization parameter, so results remain comparable to the published simulation tables.

## Limitations

- BA attachment parameters and the precise definitions of the 4 graph-structure scenarios are not fully specified in the article text, complicating exact reproduction.
- Software versions for R and igraph are not reported (only citation numbers), so graph generation may vary across igraph releases.
- BA graphs are only a proxy for real KEGG pathway topology whose sparse connectivity can greatly hinder performance; the paper advises examining graph structure before applying regularization.

## Evidence

- [discussion] Random pathway graphs are generated with igraph according to the Barabasi-Albert model: "Random graphs were generated using the igraph(42) package according to the Barabasi-Albert model(43)"
- [discussion] All PaIRKAT simulations are executed in R: "All simulations were accomplished using R(41)."
- [discussion] Disconnected generated graphs penalize the unregularized normalized Laplacian but not PaIRKAT: "Using the normalized Laplacian, , hindered testing performance compared to using PaIRKAT or ignoring the pathway information when a graph was disconnected."
- [discussion] Sparse real pathway topology motivates stress-testing with generated graph structures: "Unique challenges arose from the sparsity present in many metabolomic pathways which can greatly hinder performance."
- [readme] The repository README locates the TypeI and Power simulation scripts that consume generated graphs: "TypeI and Power simulation scripts"
