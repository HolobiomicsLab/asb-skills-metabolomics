---
name: monte-carlo-simulation-study-design
description: Use when validating that a proposed statistical test (e.g., a kernel
  score test) maintains nominal size, by estimating empirical type I error rates via
  Monte Carlo replication under simulated null data.
when_to_use_negative:
- The goal is power evaluation under an alternative association rather than type I
  error calibration under the null — the paper separates these into distinct Power
  and TypeI simulation scripts
- The analysis target is the real COPDGene metabolomics data application rather than
  synthetic null-data calibration
- Pathway graphs come from real KEGG reaction-database structure and must first be
  inspected for disconnectedness before choosing a regularization step
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Environment in which all null-data simulations, PaIRKAT score tests, and comparison
    kernel tests are executed
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates the random pathway graphs according to the Barabasi-Albert model
    used in each simulation scenario
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:c1484a60cb74c76caf9003255228d2c3a2678b58bb06958a76b10cef92a53ac9
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
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Monte Carlo simulation study design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design and execute null-hypothesis Monte Carlo simulations to verify that PaIRKAT's kernel score test maintains its nominal type I error rate (~0.05) across graph-structured scenarios, mirroring Tables 1-4 and S1-S4 of the paper.

## When to use

Use when validating that a proposed statistical test (e.g., a kernel score test) maintains nominal size, by estimating empirical type I error rates via Monte Carlo replication under simulated null data.

## When NOT to use

- The goal is power evaluation under an alternative association rather than type I error calibration under the null — the paper separates these into distinct Power and TypeI simulation scripts
- The analysis target is the real COPDGene metabolomics data application rather than synthetic null-data calibration
- Pathway graphs come from real KEGG reaction-database structure and must first be inspected for disconnectedness before choosing a regularization step

## Inputs

- Simulation R scripts from github.com/CharlieCarpenter/PaIRKAT (SimulationFunctions: TypeI and Power scripts)
- Random pathway graphs generated with igraph under the Barabasi-Albert model
- Null multivariate normal metabolite data with Danaher joint graphical-model precision matrix (n=160; p=15, 30, 45)

## Outputs

- type1_error_rates.csv with per-scenario empirical type I error rates aggregated over 1000 Monte Carlo simulations
- Scenario-wise rejection proportions comparable to Tables 1-4 and supplementary Tables S1-S4

## How to apply

Generate random pathway graphs with the igraph package under the Barabasi-Albert model, then simulate metabolite data under the null (no phenotype association) as multivariate normal draws whose dependence structure follows the Danaher joint graphical-model precision matrix, using sample size n=160 and pathway sizes p=15, 30, and 45. For each of the 12 scenario combinations, run 1000 Monte Carlo simulations, applying PaIRKAT and the comparison kernel tests (regularized and unregularized Laplacian variants) at alpha=0.05 under the null. Use the regularization parameter tau=1, as fixed throughout the paper's simulations and data analyses. Aggregate the proportion of rejections across the 1000 replicates per scenario to estimate empirical type I error rates, and write them to type1_error_rates.csv. Verify each scenario's empirical rate is approximately 0.05, matching the nominal level reported in Tables 1-4 (and S1-S4 for the normalized Laplacian variant).

## Related tools

- **R** (Environment in which all null-data simulations, PaIRKAT score tests, and comparison kernel tests are executed) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates the random pathway graphs according to the Barabasi-Albert model used in each simulation scenario) — https://github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- Each scenario's aggregated empirical rejection proportion over 1000 Monte Carlo simulations is approximately 0.05 (the nominal alpha level)
- Rates cover all 12 pathway information scenarios with pathway sizes p=15, 30, and 45, matching the layout of Tables 1-4 and S1-S4
- Both PaIRKAT and the comparison kernel tests (regularized and unregularized Laplacian variants) are evaluated under the same null data and alpha=0.05
- Output CSV (type1_error_rates.csv) contains one empirical rate per scenario with no missing scenario cells

## Limitations

- The regularization parameter tau is fixed at 1 in all simulations, with no sensitivity analysis of tau reported
- With 1000 Monte Carlo replicates per scenario, empirical rates near 0.05 carry non-trivial Monte Carlo sampling error
- Graph structure must be examined before applying regularization, so simulation conclusions do not transfer blindly to arbitrarily structured pathways

## Evidence
