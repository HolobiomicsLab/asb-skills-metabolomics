---
name: statistical-power-curve-estimation
description: Use when comparing association-test power between graph-regularized kernel
  tests (PaIRKAT, regularized normalized Laplacian), an unregularized normalized Laplacian
  kernel, and a kernel test ignoring network information across signal-to-noise-ratio
  levels.
when_to_use_negative:
- When the analysis goal is a single association test on real data rather than a simulation-based
  power comparison across methods and scenarios
- When the input graph is fully connected and complete pathway knowledge is available
  — the article shows method differences were indistinguishable for datasets with
  over 1,000 observations
- When type-I error calibration (not power) is the target — PaIRKAT's TypeI simulation
  scripts cover that separately
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for running simulation scripts and computing score-test p-values
- name: igraph
  role: Generates random pathway graphs according to the Barabasi-Albert model for
    the 4 graph-structure scenarios
- name: PaIRKAT
  role: Source of the score-test functions and TypeI/Power simulation scripts used
    to produce power curves
  repo: github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:e4bb6d88916f7d0ca46f2f3198cef52617bd655830effacb4623b6f1631d8c58
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# statistical-power-curve-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate empirical power of PaIRKAT versus network-ignoring and unregularized-Laplacian score tests by simulating Barabasi-Albert pathway graphs with igraph in R across signal-to-noise ratios and pathway-knowledge/structure scenarios, then plotting rejection rates as power curves.

## When to use

Use when comparing association-test power between graph-regularized kernel tests (PaIRKAT, regularized normalized Laplacian), an unregularized normalized Laplacian kernel, and a kernel test ignoring network information across signal-to-noise-ratio levels.

## When NOT to use

- When the analysis goal is a single association test on real data rather than a simulation-based power comparison across methods and scenarios
- When the input graph is fully connected and complete pathway knowledge is available — the article shows method differences were indistinguishable for datasets with over 1,000 observations
- When type-I error calibration (not power) is the target — PaIRKAT's TypeI simulation scripts cover that separately

## Inputs

- Random pathway graphs generated with igraph according to the Barabasi-Albert model (4 structure scenarios)
- Simulated metabolite/phenotype data with fixed effects βj = 0.1 or βj drawn from N(0.1, 0.1)
- Pathway knowledge scenarios (3 levels: full, partial/incorrect, none)
- Signal-to-noise ratio grid

## Outputs

- simulation_power_results.csv (empirical power table for each method × scenario × SNR)
- Figure 1 power curves: 3 knowledge × 4 structure panels
- Figure 2 power-vs-SNR curves for PaIRKAT, unregularized normalized Laplacian, and network-ignoring kernel test

## How to apply

Run the simulation scripts from the PaIRKAT GitHub repository (github.com/CharlieCarpenter/PaIRKAT) in R. Generate random pathway graphs with igraph according to the Barabasi-Albert model, covering the 4 graph-structure scenarios crossed with 3 levels of pathway knowledge. Simulate metabolite/phenotype data with fixed effects βj = 0.1, and for SNR-varied runs draw βj from N(0.1, 0.1) across a grid of signal-to-noise ratios. Compute p-values from the score test with regularization parameter τ = 1 for the three competing approaches: PaIRKAT (regularized normalized Laplacian), the unregularized normalized Laplacian, and a kernel test ignoring network information. Estimate empirical power as the rejection rate over simulation replicates for each method × scenario × SNR combination, write the table to simulation_power_results.csv, and plot Figure 1 power curves (3 knowledge × 4 structure panels) and Figure 2 power-vs-SNR curves. Rationale: the paper shows PaIRKAT reaches 80% power at SNR ≈ 0.23 versus ≈ 0.60 for ignoring network information, and the unregularized normalized Laplacian hindered testing performance on disconnected graphs.

## Related tools

- **R** (Environment for running simulation scripts and computing score-test p-values)
- **igraph** (Generates random pathway graphs according to the Barabasi-Albert model for the 4 graph-structure scenarios)
- **PaIRKAT** (Source of the score-test functions and TypeI/Power simulation scripts used to produce power curves) — github.com/CharlieCarpenter/PaIRKAT

## Examples

```
Rscript SimulationFunctions/Power.R  # run from github.com/CharlieCarpenter/PaIRKAT to produce simulation_power_results.csv and Figures 1–2
```

## Evaluation signals

- PaIRKAT reaches approximately 80% power at a signal-to-noise ratio around 0.23
- The network-ignoring kernel test requires an SNR of about 0.60 to reach 80% power
- The unregularized normalized Laplacian never reaches 80% power on disconnected graphs
- Empirical power table is written to simulation_power_results.csv with one row per method × scenario × SNR
- Figure 1 contains 3 knowledge × 4 structure = 12 power panels

## Limitations

- The regularization parameter τ is set to 1 for all simulations, with no sensitivity analysis of τ reported in the paper
- Software/tool versions not reported for R or igraph (only citation numbers given)
- Method performance differences in real data were only demonstrated on downsampled subsets (sizes 100–500), not the full cohort
- The filtered dataset did not contain every metabolite within the 28 KEGG pathways selected, so some analyzed pathways had fewer than 10 metabolites

## Evidence

- [other] PaIRKAT achieved approximately 80% power at a signal-to-noise ratio around 0.23: [evidence span withheld — anchor status: not_found]
