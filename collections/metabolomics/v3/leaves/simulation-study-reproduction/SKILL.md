---
name: simulation-study-reproduction
description: Use when a task requires reproducing or validating published kernel-association-test
  power results, specifically PaIRKAT's power-vs-SNR curves over graph-structure and
  pathway-knowledge scenarios.
when_to_use_negative:
- The goal is applying PaIRKAT to real metabolomics data (e.g. COPDGene KEGG pathways)
  rather than reproducing the simulation power study.
- Datasets exceed 1,000 observations, where the paper reports p-values from the compared
  methods were indistinguishable, so no power separation would appear.
- The pathway graph is fully connected with complete pathway knowledge, making the
  disconnected-graph scenarios central to the comparison moot.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Language in which all simulation scripts (TypeI and Power) are executed to
    produce empirical power estimates and power-curve figures
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates the random pathway graphs according to the Barabasi-Albert model
    for the 4 graph-structure scenarios crossed with 3 levels of pathway knowledge
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:540d2cb3310e9d52a58e8c579715e9ec3d100e4d94d4418c1f7acc29601415a1
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# simulation-study-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the PaIRKAT simulation study in R, regenerating empirical power curves that compare the regularized normalized Laplacian kernel test against ignoring graph topography and against the unregularized normalized Laplacian across pathway structures, pathway-knowledge levels, and SNR grids.

## When to use

Use when a task requires reproducing or validating published kernel-association-test power results, specifically PaIRKAT's power-vs-SNR curves over graph-structure and pathway-knowledge scenarios.

## When NOT to use

- The goal is applying PaIRKAT to real metabolomics data (e.g. COPDGene KEGG pathways) rather than reproducing the simulation power study.
- Datasets exceed 1,000 observations, where the paper reports p-values from the compared methods were indistinguishable, so no power separation would appear.
- The pathway graph is fully connected with complete pathway knowledge, making the disconnected-graph scenarios central to the comparison moot.

## Inputs

- PaIRKAT GitHub repository simulation scripts (SimulationFunctions: TypeI and Power scripts)
- igraph-generated random pathway graphs (Barabasi-Albert model) for 4 structure scenarios x 3 pathway-knowledge levels
- Simulated phenotype-metabolite data with beta_j = 0.1 (fixed) or beta_j ~ N(0.1, 0.1) across an SNR grid
- Regularization parameter setting tau = 1

## Outputs

- simulation_power_results.csv (empirical power table over method x scenario x SNR)
- Figure 1 PNG: power curves arranged as 3 knowledge-level x 4 structure panels
- Figure 2 PNG: power-vs-SNR curves for the three competing methods
- Empirical power estimates (rejection rates over replicates) per method/scenario/SNR

## How to apply

Clone the PaIRKAT repository (github.com/CharlieCarpenter/PaIRKAT) and run the SimulationFunctions scripts (TypeI and Power) in R, using igraph to generate random pathway graphs via the Barabasi-Albert model covering 4 graph-structure scenarios crossed with 3 levels of pathway knowledge. Simulate phenotype-metabolite data with fixed effects beta_j = 0.1, and for SNR-varied runs draw beta_j from N(0.1, 0.1) across a grid of signal-to-noise ratios. For every scenario x method x SNR cell, compute p-values from the score test with regularization parameter tau = 1 for three competitors: PaIRKAT (regularized normalized Laplacian), the unregularized normalized Laplacian, and a kernel test ignoring network information. Estimate empirical power as the rejection rate over simulation replicates, write the results to simulation_power_results.csv, and plot the Figure 1 power curves (3 knowledge x 4 structure panels) and Figure 2 power-vs-SNR curves as PNGs.

## Related tools

- **R** (Language in which all simulation scripts (TypeI and Power) are executed to produce empirical power estimates and power-curve figures) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates the random pathway graphs according to the Barabasi-Albert model for the 4 graph-structure scenarios crossed with 3 levels of pathway knowledge) — https://github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- Reproduced Figure 2 must show PaIRKAT reaching 80% power at SNR ~ 0.23 versus ~ 0.60 for ignoring network information.
- The unregularized normalized Laplacian curve must fail to reach 80% power on disconnected graphs, matching the paper's reported performance.
- simulation_power_results.csv is produced with empirical power (rejection-rate) entries for every method x scenario x SNR combination.
- Figure 1 renders as 12 panels (3 pathway-knowledge levels x 4 graph-structure scenarios) and Figure 2 as power-vs-SNR curves, both exported as PNGs.
- Score-test p-values in every run must be computed with the regularization parameter fixed at tau = 1, as in the paper.

## Limitations

- The regularization parameter tau is fixed to 1 in all simulations and data analyses, with no sensitivity analysis of tau reported.
- Software/tool versions for R and igraph are not reported (only citation numbers), so exact reproducibility may depend on version drift.
- The unregularized normalized Laplacian hindered testing performance on disconnected graphs, so that competitor behaves degenerately under some scenarios.
- P-values from the compared methods are indistinguishable for datasets with over 1,000 observations; differences were demonstrated via subsets of 100-500 subjects.

## Evidence

- [discussion] power improvements from regularized normalized Laplacian: [evidence span withheld — anchor status: not_found]
