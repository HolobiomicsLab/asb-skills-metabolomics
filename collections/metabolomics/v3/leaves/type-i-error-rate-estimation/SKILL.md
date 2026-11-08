---
name: type-i-error-rate-estimation
description: Use when validating that a proposed association test (e.g. PaIRKAT on
  simulated metabolite pathway graphs) holds its nominal type I error rate before
  assessing power.
when_to_use_negative:
- The goal is to estimate statistical power or detect sensitivity to alternatives
  — this skill only calibrates the null-hypothesis rejection rate, not power under
  true associations.
- Data are real experimental results (e.g. COPDGene metabolomics) rather than null
  simulations; empirical rejection rates of real data cannot validate calibration.
- The test of interest is not a kernel machine score test and no pathway graph structure
  is involved.
tools:
- name: R
  role: Language used to run the type I error simulation scripts
- name: igraph
  role: Generates random pathway graphs under the Barabasi-Albert model for the null
    simulations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.04.23.440821
    title: 'PaIRKAT: A pathway integrated regression-based kernel association test
      with applications to metabolomics and COPD phenotypes'
  source_content_hash: sha256:8aeb5e1fbca95ec0e6a10dfcd5c6605563ffe8b64fe4319b07a4406212a06ed1
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# type-i-error-rate-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirical type I error rate calibration for a pathway kernel score test: run 1000 null simulations per scenario and verify the proportion of rejections at alpha=0.05 is approximately 0.05.

## When to use

Use when validating that a proposed association test (e.g. PaIRKAT on simulated metabolite pathway graphs) holds its nominal type I error rate before assessing power.

## When NOT to use

- The goal is to estimate statistical power or detect sensitivity to alternatives — this skill only calibrates the null-hypothesis rejection rate, not power under true associations.
- Data are real experimental results (e.g. COPDGene metabolomics) rather than null simulations; empirical rejection rates of real data cannot validate calibration.
- The test of interest is not a kernel machine score test and no pathway graph structure is involved.

## Inputs

- Simulated metabolite data under the null as multivariate normal draws with Danaher joint graphical-model precision matrices
- Pathway graphs (igraph objects) generated via the Barabasi-Albert model for p=15, 30, 45 metabolites, n=160 samples
- R scripts for TypeI simulations from github.com/CharlieCarpenter/PaIRKAT

## Outputs

- Empirical type I error rates per simulation scenario (proportion of rejections over 1000 simulations at alpha=0.05)
- type1_error_rates.csv aggregating error rates across scenarios
- Comparison against Tables 1-4 and S1-S4 of the PaIRKAT paper

## How to apply

Simulate metabolite data under the null (no phenotype association) as multivariate normal draws whose dependence structure follows the Danaher joint graphical-model precision matrix, using graphs generated with igraph under the Barabasi-Albert model, for sample size n=160 and pathway sizes p=15, 30, and 45. For each scenario run 1000 Monte Carlo simulations, applying PaIRKAT (and comparison kernels such as the unregularized normalized Laplacian) at the alpha=0.05 level, including the regularized normalized Laplacian variant with tau=1. Aggregate the proportion of rejections across the 1000 simulations per scenario to estimate empirical type I error rates, mirroring Tables 1-4 and supplementary Tables S1-S4, and write them to type1_error_rates.csv. The test is correctly calibrated only if each scenario's empirical rejection rate is approximately 0.05; results should also be checked against the normalized Laplacian variant (Tables S1-S4).

## Related tools

- **R** (Language used to run the type I error simulation scripts)
- **igraph** (Generates random pathway graphs under the Barabasi-Albert model for the null simulations)

## Examples

```
Rscript run_type1_simulation.R --pathway_size 30 --n 160 --alpha 0.05 --output type1_error_rates.csv
```

## Evaluation signals

- Aggregate the proportion of rejections over 1000 Monte Carlo simulations per scenario; each should be approximately 0.05 (nominal alpha=0.05)
- Replicate the scenarios in Tables 1-4 (and S1-S4 for the normalized Laplacian variant) and compare empirical rates to those tables
- Run each scenario at n=160 with pathway sizes p=15, 30, 45 under the Danaher joint graphical-model precision matrix
- Confirm output matches the schema of type1_error_rates.csv with one empirical rate per scenario

## Limitations

- Simulation settings are specific to n=160 and pathway sizes p=15, 30, 45; other sample sizes or pathway sizes are not covered by the paper's type I error tables
- Regularization parameter tau is fixed to 1 in all simulations, with no sensitivity analysis of tau reported
- Only Barabasi-Albert (igraph) random graphs are used to model pathway structure; other graph topologies are not evaluated for type I error

## Evidence

- [results] Simulation settings are specific to n=160 and pathway sizes p=15, 30, 45; other sample sizes or pathway sizes are not covered by the paper's type I error tables: [evidence span withheld — anchor status: not_found]
- [results] Empirical type I error rates per simulation scenario (proportion of rejections over 1000 simulations at alpha=0.05): [evidence span withheld — anchor status: not_found]
- [results] Aggregate the proportion of rejections over 1000 Monte Carlo simulations per scenario; each should be approximately 0.05 (nominal alpha=0.05): [evidence span withheld — anchor status: not_found]
- [methods] Run each scenario at n=160 with pathway sizes p=15, 30, 45 under the Danaher joint graphical-model precision matrix: [evidence span withheld — anchor status: not_found]
