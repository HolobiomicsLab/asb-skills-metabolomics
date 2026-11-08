---
name: kernel-machine-score-test-power-estimation
description: Use when assessing how much kernel-association testing power depends
  on pathway graph fidelity — i.e., simulating Barabasi-Albert pathway graphs, permuting
  a known fraction of edges, and estimating empirical power of the PaIRKAT score test.
when_to_use_negative:
- Real-data analysis on the full COPDGene cohort (>1,000 observations) — the paper
  shows method p-value differences were indistinguishable for datasets with over 1,000
  observations; performance differences were demonstrated via random subsets of 100–500
  subjects, so power simulations on small subsets, not full-cohort testing, are the
  appropriate setting.
- The pathway graph is disconnected or sparse and has not been examined first — the
  paper states it is important to examine graph structure before applying regularization,
  and the unregularized normalized Laplacian hindered performance on disconnected
  graphs.
- The goal is real metabolomics data processing (run-day median normalization, kNN
  imputation with k = 10, covariate adjustment) rather than simulation-based power
  assessment.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language in which all simulations and the PaIRKAT score test are implemented
  repo: github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates Barabasi-Albert random pathway graphs and supports edge permutation
    for the mismatch scenarios
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:02800e68821734d98262283fefa288671105a07e58d8355566690f4578dc8e2d
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# kernel-machine-score-test-power-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirical power estimation for the PaIRKAT kernel machine score test (τ = 1) under simulated pathway-structured outcomes, quantifying how testing power degrades as incorrect (permuted) pathway edges are introduced at 10%, 40%, and 70% mismatch levels across pathway sizes p = 15, 30, and 45.

## When to use

Use when assessing how much kernel-association testing power depends on pathway graph fidelity — i.e., simulating Barabasi-Albert pathway graphs, permuting a known fraction of edges, and estimating empirical power of the PaIRKAT score test.

## When NOT to use

- Real-data analysis on the full COPDGene cohort (>1,000 observations) — the paper shows method p-value differences were indistinguishable for datasets with over 1,000 observations; performance differences were demonstrated via random subsets of 100–500 subjects, so power simulations on small subsets, not full-cohort testing, are the appropriate setting.
- The pathway graph is disconnected or sparse and has not been examined first — the paper states it is important to examine graph structure before applying regularization, and the unregularized normalized Laplacian hindered performance on disconnected graphs.
- The goal is real metabolomics data processing (run-day median normalization, kNN imputation with k = 10, covariate adjustment) rather than simulation-based power assessment.

## Inputs

- Simulated pathway-structured outcome and metabolomics data matrices from the PaIRKAT simulation framework (R)
- Barabasi-Albert random pathway graphs (igraph) with p = 15, 30, or 45 metabolite nodes
- Pathway graph edge lists before and after partial edge permutation (10%, 40%, 70% incorrect edges)

## Outputs

- power_mismatch_by_size.csv with one row per (pathway size, mismatch level) combination and columns for pathway size, mismatch level, and estimated power in [0, 1]
- Empirical power estimates per (mismatch level × pathway size) aggregated over simulation replicates

## How to apply

Load the PaIRKAT simulation framework (R code from github.com/CharlieCarpenter/PaIRKAT, SimulationFunctions with TypeI and Power scripts) and generate Barabasi-Albert random pathway graphs of p = 15, 30, and 45 metabolites using the igraph package, as in the original simulations. For each mismatch level (10%, 40%, 70% incorrect edges), permute the corresponding fraction of edges while keeping the edge count constant so only correctness of connections varies. Run the PaIRKAT kernel machine score test with the regularization parameter fixed at τ = 1 — the value used throughout the paper — on simulated pathway-structured outcomes, estimating empirical power over repeated replicates for every (pathway size, mismatch level) combination. Aggregate replicates into a table (rows = pathway size 15/30/45, columns/rows = mismatch level 10%/40%/70%) and write it to power_mismatch_by_size.csv. Expected qualitative pattern: power at ~10% mismatch is comparable to perfect structure in small pathways, and power decreases monotonically as incorrect-edge proportion grows toward 40% and 70%, with larger pathways retaining higher power.

## Related tools

- **R** (Language in which all simulations and the PaIRKAT score test are implemented) — github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates Barabasi-Albert random pathway graphs and supports edge permutation for the mismatch scenarios)

## Examples

```
library(igraph); g <- barabasi.game(30, directed=FALSE)
```

## Evaluation signals

- power_mismatch_by_size.csv exists, parses as CSV, and contains exactly one row per (pathway size, mismatch level) combination: 3 sizes × 3 mismatch levels.
- Every estimated power value is numeric and within [0, 1].
- Monotonicity check: empirical power at 10% mismatch ≥ power at 40% ≥ power at 70% for each pathway size, matching the paper's finding that power decreases as more incorrect edges are introduced.
- Small-pathway check: power at ~10% mismatch is qualitatively comparable to the perfect-structure setting for p = 15.
- Score test configuration matches the paper: regularization parameter τ = 1, and random graphs follow the Barabasi-Albert model as generated with igraph.

## Limitations

- The regularization parameter τ is fixed to 1 in all simulations and data analyses, with no sensitivity analysis of τ reported.
- Software/tool versions are not reported for R or igraph (only citation numbers), so exact reproducibility may vary.
- Graph sparsity in metabolomic pathways is a stated challenge that can greatly hinder performance, and graph structure must be examined before applying regularization.
- The unregularized normalized Laplacian hindered testing performance on disconnected graphs, so mismatch-permutation results assume appropriately regularized kernels.

## Evidence
