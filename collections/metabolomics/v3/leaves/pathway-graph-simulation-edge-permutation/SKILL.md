---
name: pathway-graph-simulation-edge-permutation
description: Use when you need to evaluate robustness or statistical power of pathway-structured
  association tests (e.g. kernel machine tests on Laplacian-derived pathway kernels)
  under controlled amounts of incorrect or incomplete pathway graph knowledge.
when_to_use_negative:
- The goal is a real-data pathway analysis on measured (not simulated) pathway knowledge
  — this skill fabricates synthetic graphs for benchmarking, not for biological inference.
- Pathway graphs are already correct and only incomplete coverage (missing metabolites/edges)
  is of interest rather than incorrectly rewired edges; a different incompleteness
  simulation would be appropriate.
- The analysis pipeline does not consume a graph or pathway structure at all (e.g.
  univariate metabolite association testing); permuting edges has no target to perturb.
tools:
- name: R
  role: Environment in which the simulation scripts are written and executed
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates Barabasi-Albert random graphs used as simulated pathway structures
    and supports edge permutation to introduce incorrect edges
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:106c1372c0054ed11a7b8628435d9a11d2d486243681fa60c551934f33fe2d3a
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathway-graph-simulation-edge-permutation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate Barabasi-Albert random pathway graphs with igraph and permute a controlled fraction of their edges (keeping edge count fixed) to simulate incorrect pathway knowledge, so that power of graph-regularized tests like PaIRKAT can be measured as a function of pathway mismatch.

## When to use

Use when you need to evaluate robustness or statistical power of pathway-structured association tests (e.g. kernel machine tests on Laplacian-derived pathway kernels) under controlled amounts of incorrect or incomplete pathway graph knowledge.

## When NOT to use

- The goal is a real-data pathway analysis on measured (not simulated) pathway knowledge — this skill fabricates synthetic graphs for benchmarking, not for biological inference.
- Pathway graphs are already correct and only incomplete coverage (missing metabolites/edges) is of interest rather than incorrectly rewired edges; a different incompleteness simulation would be appropriate.
- The analysis pipeline does not consume a graph or pathway structure at all (e.g. univariate metabolite association testing); permuting edges has no target to perturb.

## Inputs

- Pathway size parameters (e.g. 15, 30, 45 metabolites per simulated pathway)
- Simulated metabolomic feature/covariate data on which the association test is run
- Fraction of edges to permute (e.g. 10%, 40%, 70% incorrect edges)
- Number of simulation replicates for empirical power estimation

## Outputs

- Perturbed pathway graph objects (Barabasi-Albert graphs with a fixed edge count and a controlled proportion of incorrect edges)
- Empirical power estimates per (pathway size, mismatch level) combination, e.g. a table such as power_mismatch_by_size.csv with power values in [0, 1]

## How to apply

Build pathway graphs with a scale-free topology using igraph's Barabasi-Albert model, as done in the PaIRKAT simulations, at several pathway sizes (e.g. p = 15, 30, 45 metabolites). To induce mismatch, permute a chosen fraction of the edges — 10%, 40%, or 70% incorrect edges as benchmarked in the paper — replacing true connections with incorrect ones while keeping the total number of edges constant so that only edge correctness, not graph density, varies. Pair each perturbed graph with simulated metabolite/covariate data and run the association test under evaluation (PaIRKAT's score test with regularization parameter τ = 1 fixed throughout). Aggregate replicate outcomes into empirical power estimates and tabulate them by (pathway size, mismatch level); expect a monotonic decline in power as the incorrect-edge proportion rises, with perfect structure giving the most power and larger pathways retaining more power than small ones.

## Related tools

- **R** (Environment in which the simulation scripts are written and executed) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates Barabasi-Albert random graphs used as simulated pathway structures and supports edge permutation to introduce incorrect edges)

## Examples

```
Rscript SimulationFunctions/Power.R  # generate Barabasi-Albert pathways, permute 10/40/70% of edges, run PaIRKAT score test with τ = 1, and tabulate power
```

## Evaluation signals

- The number of edges in each perturbed graph equals the number in the original Barabasi-Albert graph — permutation must not change graph density.
- For each declared mismatch level (10%, 40%, 70%), the realized fraction of incorrect edges matches the target within simulation tolerance.
- Estimated power values across replicates fall in [0, 1] and decrease monotonically as the incorrect-edge proportion increases from 10% to 70% at a fixed pathway size.
- At roughly 10% mismatch, empirical power is comparable to the perfect-structure setting in small pathways, matching the paper's reported robustness claim.
- The output table contains one row per (pathway size, mismatch level) combination with numeric power for each cell.

## Limitations

- Results are specific to the Barabasi-Albert (scale-free) graph model and pathway sizes (15, 30, 45) chosen; other graph structures or sizes may behave differently.
- Robustness conclusions come from synthetic graphs with deliberately permuted edges; real pathway databases (e.g. KEGG-derived) have sparsity and incompleteness that differ from a fixed-fraction edge permutation.
- The regularization parameter τ is fixed at 1 throughout the simulations, so the power curves do not assess sensitivity to τ.

## Evidence
