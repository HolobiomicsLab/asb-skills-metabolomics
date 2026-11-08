---
name: graph-regularization-simulation
description: Use when comparing kernel association test power under simulated pathway
  graphs with varied topology and incomplete knowledge, to judge whether graph regularization
  (τ = 1) beats unregularized Laplacian or network-agnostic tests.
when_to_use_negative:
- Datasets with over 1,000 observations, where p-values from the compared methods
  were indistinguishable from one another — power differences only manifest on small
  random subsets (sizes 100–500)
- Studies whose goal is Type I error calibration alone rather than power comparison
  across graph structures and SNR levels
- Pathway graphs already known to be fully connected and accurately annotated, where
  the unregularized normalized Laplacian does not suffer the disconnected-graph performance
  penalty
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used for all simulations, score-test p-value computation, power estimation,
    and figure generation
- name: igraph
  role: Generates the random pathway graphs according to the Barabasi-Albert model
    for the 4 graph-structure scenarios
- name: PaIRKAT
  role: Provides the SimulationFunctions scripts for TypeI and power simulations and
    the regularized kernel score test (τ = 1)
  repo: github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:6f39e9a88c906d7914a0dd262753a07212388322e9b88a122dc7854017981569
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CharlieCarpenter/PaIRKAT
  tools:
  - PaIRKAT
  - R
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 23
---

# graph-regularization-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulation skill for quantifying how regularizing a pathway graph (PaIRKAT's regularized normalized Laplacian) improves kernel association test power versus ignoring network information or using the unregularized normalized Laplacian, across Barabasi-Albert graph structures, levels of pathway knowledge, and signal-to-noise ratios.

## When to use

Use when comparing kernel association test power under simulated pathway graphs with varied topology and incomplete knowledge, to judge whether graph regularization (τ = 1) beats unregularized Laplacian or network-agnostic tests.

## When NOT to use

- Datasets with over 1,000 observations, where p-values from the compared methods were indistinguishable from one another — power differences only manifest on small random subsets (sizes 100–500)
- Studies whose goal is Type I error calibration alone rather than power comparison across graph structures and SNR levels
- Pathway graphs already known to be fully connected and accurately annotated, where the unregularized normalized Laplacian does not suffer the disconnected-graph performance penalty

## Inputs

- Random pathway graphs generated with igraph under the Barabasi-Albert model (4 graph-structure scenarios, including disconnected graphs)
- Simulated phenotype-metabolite data with fixed effects βj = 0.1, or βj ~ N(0.1, 0.1) drawn across a grid of signal-to-noise ratios
- 3 levels of pathway knowledge (correct / incomplete / incorrect pathway information)

## Outputs

- simulation_power_results.csv — empirical power (rejection rate over replicates) for each method × scenario × SNR combination
- Power curves figure: 3 knowledge × 4 structure panels comparing the three methods
- Power-vs-SNR curves showing each method's SNR required to reach 80% power

## How to apply

Clone the PaIRKAT scripts (github.com/CharlieCarpenter/PaIRKAT) and run the SimulationFunctions TypeI and Power scripts in R. Generate random pathway graphs with the igraph package under the Barabasi-Albert model, crossing 4 graph-structure scenarios (including disconnected graphs) with 3 levels of pathway knowledge. Simulate phenotype-metabolite data with fixed effects βj = 0.1; for SNR-varied runs draw βj from N(0.1, 0.1) over a grid of signal-to-noise ratios. For each scenario compute p-values from the score test with regularization parameter τ = 1 for the three competing approaches: PaIRKAT (regularized normalized Laplacian), the unregularized normalized Laplacian, and a kernel test ignoring network information. Estimate empirical power as the rejection rate over replicates for each method × scenario × SNR combination, write the power table to simulation_power_results.csv, and plot the 3 knowledge × 4 structure panel figure and the power-vs-SNR curves.

## Related tools

- **R** (Language used for all simulations, score-test p-value computation, power estimation, and figure generation)
- **igraph** (Generates the random pathway graphs according to the Barabasi-Albert model for the 4 graph-structure scenarios)
- **PaIRKAT** (Provides the SimulationFunctions scripts for TypeI and power simulations and the regularized kernel score test (τ = 1)) — github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- Reproduced power-vs-SNR figure shows PaIRKAT reaching 80% power at SNR ≈ 0.23 versus ≈ 0.60 for the network-ignoring test
- The unregularized normalized Laplacian never reaches 80% power on disconnected graphs, matching the reported performance penalty
- simulation_power_results.csv contains rejection-rate rows for the full 3 knowledge levels × 4 graph structures × SNR grid × 3 methods
- Regularization parameter is fixed at τ = 1 throughout all simulated score tests
- Simulated fixed effects match the specification: βj = 0.1 for fixed-effect runs, βj ~ N(0.1, 0.1) for SNR-varied runs

## Limitations

- The regularization parameter τ is fixed at 1 in all simulations and data analyses, with no sensitivity analysis of τ reported
- Graph structure must be examined before applying regularization — the method is not universally applicable without inspection of the pathway graph
- Unique challenges arise from the sparsity present in many metabolomic pathways, which can greatly hinder performance
- Real-data performance differences were only demonstrated on random subsets of 100–500 subjects; p-values from the compared methods were indistinguishable for datasets with over 1,000 observations
- Software/tool versions for R and igraph are not reported in the paper, only citations

## Evidence

- [discussion] we saw meaningful improvements in power by including pathway information through a graph's regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
