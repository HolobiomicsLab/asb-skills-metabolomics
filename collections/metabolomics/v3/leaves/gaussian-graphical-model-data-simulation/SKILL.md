---
name: gaussian-graphical-model-data-simulation
description: Use when evaluating a kernel or pathway-graph association test via Monte
  Carlo — you need synthetic correlated data whose covariance encodes a pathway graph
  under null or alternative hypotheses.
when_to_use_negative:
- You need real observed metabolomics data rather than synthetic null/alternative
  draws (e.g., the COPDGene analysis with 1,113 FEV1/FVC and 1,065 percent emphysema
  complete cases)
- The dependence structure of your data is not plausibly represented by a Gaussian
  graphical model precision matrix (e.g., non-continuous or heavily discrete traits)
- You require sensitivity analysis over the regularization parameter τ, which the
  paper fixed at 1 in all simulations without reporting sensitivity
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: R
  role: Language in which all simulation scripts are implemented and executed
  repo: https://github.com/CharlieCarpenter/PaIRKAT
- name: igraph
  role: Generates random pathway graphs under the Barabasi-Albert model for the simulated
    scenarios
- name: PaIRKAT scripts
  role: SimulationFunctions repository scripts run the Type I error and power simulations
  repo: https://github.com/CharlieCarpenter/PaIRKAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:d882c569b90276feb765a3aa91fbfdb0e0efbcdc88414bfa2ea6c79deebc04df
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
  - PaIRKAT scripts
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Gaussian graphical model data simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates metabolomics-style omics data as multivariate normal draws whose dependence structure follows a Danaher joint graphical-model precision matrix, built on random graphs generated with igraph under the Barabasi-Albert model, to evaluate kernel association tests such as PaIRKAT.

## When to use

Use when evaluating a kernel or pathway-graph association test via Monte Carlo — you need synthetic correlated data whose covariance encodes a pathway graph under null or alternative hypotheses.

## When NOT to use

- You need real observed metabolomics data rather than synthetic null/alternative draws (e.g., the COPDGene analysis with 1,113 FEV1/FVC and 1,065 percent emphysema complete cases)
- The dependence structure of your data is not plausibly represented by a Gaussian graphical model precision matrix (e.g., non-continuous or heavily discrete traits)
- You require sensitivity analysis over the regularization parameter τ, which the paper fixed at 1 in all simulations without reporting sensitivity

## Inputs

- Pathway graph specification (random graph parameters for igraph's Barabasi-Albert model)
- Precision matrix from a Danaher joint graphical model encoding metabolite dependence structure
- Simulation design parameters: sample size n=160, pathway sizes p=15/30/45, number of Monte Carlo replicates (1000), alpha=0.05

## Outputs

- Simulated multivariate normal metabolite datasets per scenario (null and alternative)
- Empirical type I error rates per scenario (target ~0.05), replicating Tables 1–4 and S1–S4
- Empirical power estimates for alternative-hypothesis scenarios
- Aggregated results table such as type1_error_rates.csv

## How to apply

First generate a random pathway graph using the igraph package according to the Barabasi-Albert model, mirroring the sparse, scale-free topology of metabolomic pathways. Derive the joint graphical-model precision matrix (Danaher framework) from the graph so that dependence between simulated metabolites follows pathway edges. Draw multivariate normal data under the null (no phenotype association) at n=160 with pathway sizes p=15, 30, and 45, and under alternatives for power studies. Run 1000 Monte Carlo simulations per scenario, applying the test under evaluation (e.g., PaIRKAT score test with regularization parameter τ = 1, plus regularized and unregularized Laplacian kernel comparisons) at alpha=0.05. Aggregate the proportion of rejections across simulations to estimate empirical type I error rates (should be ~0.05, as reported in Tables 1–4 and S1–S4) or power, and write the aggregated rates to a CSV (e.g., type1_error_rates.csv).

## Related tools

- **R** (Language in which all simulation scripts are implemented and executed) — https://github.com/CharlieCarpenter/PaIRKAT
- **igraph** (Generates random pathway graphs under the Barabasi-Albert model for the simulated scenarios)
- **PaIRKAT scripts** (SimulationFunctions repository scripts run the Type I error and power simulations) — https://github.com/CharlieCarpenter/PaIRKAT

## Evaluation signals

- Empirical type I error rate in each scenario is approximately 0.05 at alpha=0.05, matching the paper's Tables 1–4 (and S1–S4 for the normalized Laplacian variant)
- Each scenario is based on 1000 Monte Carlo simulations, not fewer
- Simulated covariance/precision structure reflects the generated graph (Danaher joint graphical-model precision matrix), not independent draws
- Scenario grid covers n=160 with p=15, 30, and 45 metabolites, including regularized and unregularized Laplacian kernel comparisons
- Aggregated rates are written to type1_error_rates.csv and match the nominal level within Monte Carlo error

## Limitations

- Regularization parameter τ is fixed to 1 in all simulations, with no sensitivity analysis of τ reported
- Graph sparsity in metabolomic pathways can greatly hinder performance; examine graph structure before applying regularization
- The method's robustness must be assessed against incorrect and incomplete pathway knowledge, since simulated pathways may not reflect true biology

## Evidence
